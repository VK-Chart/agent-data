# VK CHART AI - DEVELOPER IMPLEMENTATION GUIDE

**Для:** Oleksiy Perepelytsya (Backend Developer)  
**От:** Vitaly Kravets  
**Дата:** 2024-12-02  
**Версия:** 1.0

---

## 📋 TABLE OF CONTENTS

1. [Quick Start Overview](#1-quick-start-overview)
2. [System Architecture](#2-system-architecture)
3. [Implementation Roadmap](#3-implementation-roadmap)
4. [Laravel Backend Implementation](#4-laravel-backend-implementation)
5. [Python LangChain Microservice](#5-python-langchain-microservice)
6. [MongoDB Atlas Setup](#6-mongodb-atlas-setup)
7. [Integration & Testing](#7-integration--testing)
8. [Code Examples](#8-code-examples)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. QUICK START OVERVIEW

### 1.1 Что мы строим?

**VK Chart AI** - система автоматического скоринга судов и генерации персонализированных офферов для freight brokers.

**Основная задача:**
1. Laravel получает данные о судне и грузе
2. Python microservice (LangChain) вычисляет scoring по 7 критериям (P1-P7)
3. AI генерирует персонализированное сообщение для shipowner
4. Laravel отправляет оффер и учится на corrections пользователя

### 1.2 Tech Stack

```
FRONTEND:
├── React (Inertia.js) - UI для review и corrections

BACKEND:
├── Laravel 11 - основной backend, orchestration
├── MySQL - relational data (vessels, cargoes, companies)
├── MongoDB Atlas - vector search, embeddings, knowledge bases
├── RabbitMQ - async queue для scoring requests
└── Redis - caching, session management

AI MICROSERVICE (Python):
├── FastAPI - REST API endpoints
├── LangChain - chains, prompts, agents
├── LangGraph - state management, workflows
├── LangSmith - observability, tracing, debugging
└── OpenAI API (Claude/GPT) - LLM calls
```

### 1.3 Key Documents для reference

**MUST READ перед началом:**
1. `VKCHART_AI_ARCHITECTURE.md` - полная архитектура системы
2. `MASTER_SCORING_SYSTEM.txt` - single source of truth для scoring
3. `LANG_STACK_INTEGRATION_MODEL.md` - 5-layer архитектура AI
4. `C4_FILES_CLASSIFICATION_REPORT.md` - где какие файлы

**Для конкретных задач:**
- Rules files (P1-P7) - алгоритмы scoring
- Knowledge Base files - данные для RAG
- Prompts files - инструкции для AI

---

## 2. SYSTEM ARCHITECTURE

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                          FRONTEND                                │
│                     React + Inertia.js                          │
│  - Review Scores        - Manual Corrections                    │
│  - Send Offers          - Learning Interface                    │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                      LARAVEL BACKEND                             │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   API Layer  │  │ Queue Worker │  │ Learning     │         │
│  │              │  │              │  │ Module       │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼───────┐        │
│  │           Service Layer (Business Logic)            │        │
│  └─────────────────────────┬──────────────────────────┘        │
│                            │                                     │
│  ┌────────────┬────────────┴────────────┬────────────┐         │
│  │   MySQL    │      RabbitMQ           │   Redis    │         │
│  │ (relational│      (queues)           │  (cache)   │         │
│  └────────────┴─────────────────────────┴────────────┘         │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 │ HTTP REST API
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│              PYTHON LANGCHAIN MICROSERVICE                       │
│                        (FastAPI)                                 │
│                                                                  │
│  ┌──────────────────────────────────────────────────┐          │
│  │           Layer 1: API Interface                  │          │
│  │  - /score endpoint    - /generate-message        │          │
│  └─────────────────────┬─────────────────────────────┘          │
│                        │                                         │
│  ┌─────────────────────▼─────────────────────────────┐          │
│  │      Layer 2: State Management (LangGraph)        │          │
│  │  - ScoringAgent State    - MessageAgent State     │          │
│  └─────────────────────┬─────────────────────────────┘          │
│                        │                                         │
│  ┌─────────────────────▼─────────────────────────────┐          │
│  │      Layer 3: Chain Orchestration (LangChain)     │          │
│  │  - Progressive Scoring Chains (P1→P7)             │          │
│  │  - Message Generation Chains                      │          │
│  └─────────────────────┬─────────────────────────────┘          │
│                        │                                         │
│  ┌─────────────────────▼─────────────────────────────┐          │
│  │      Layer 4: RAG & Knowledge (MongoDB Atlas)     │          │
│  │  - Vector Search     - Knowledge Retrieval        │          │
│  └─────────────────────┬─────────────────────────────┘          │
│                        │                                         │
│  ┌─────────────────────▼─────────────────────────────┐          │
│  │      Layer 5: Observability (LangSmith)           │          │
│  │  - Tracing    - Debugging    - Analytics          │          │
│  └───────────────────────────────────────────────────┘          │
│                                                                  │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                      MONGODB ATLAS                               │
│  - Vector Search (embeddings)                                   │
│  - Knowledge Bases (trade patterns, port restrictions)          │
│  - Historical Scores (learning data)                            │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Flow для Scoring Request

```
1. USER INITIATES SCORING
   Frontend → Laravel Controller
   
2. LARAVEL PREPARES DATA
   - Get vessel data from MySQL
   - Get cargo data from MySQL
   - Get company/vessel comments
   - Add context (snapshot, preferences)
   
3. LARAVEL → RABBITMQ
   Dispatch scoring job to queue
   
4. QUEUE WORKER PICKS UP JOB
   Worker → Python Microservice HTTP Request
   
5. PYTHON MICROSERVICE PROCESSES
   a) FastAPI receives request
   b) LangGraph manages state
   c) LangChain runs progressive scoring:
      - P1: Proximity (distance calculation)
      - P1A: Regional Patterns (RAG lookup)
      - P2: Regional Preferences (parse comments)
      - P3: Cargo Preferences (parse comments)
      - P4: Last Ports (history lookup)
      - P5: Intake (complex calculation)
      - P6: OpenArea Comments (NLP parsing)
      - P7: Readiness/ETA (time calculation)
   d) MongoDB Atlas for RAG queries
   e) LangSmith traces everything
   
6. PYTHON RETURNS RESULTS
   {
     "scores": {
       "P1": 18, "P1A": 12, "P2": 10, ...
     },
     "base_score": 85,
     "final_score": 88,
     "reasoning": {
       "P1": "Vessel is 250nm from loading port...",
       ...
     },
     "blocking_conditions": null
   }
   
7. LARAVEL STORES RESULTS
   - Save scores to MySQL
   - Update vessel-cargo match status
   
8. USER REVIEWS & CORRECTS (if needed)
   - Frontend shows scores + reasoning
   - User can adjust scores
   - Corrections sent back to Python for learning
```

---

## 3. IMPLEMENTATION ROADMAP

### 3.1 Phase 1: Foundation (Week 1-2)

**Goal:** Set up basic infrastructure and communication between components

#### Laravel Tasks:
- [ ] Create scoring service architecture
- [ ] Set up RabbitMQ connection and queues
- [ ] Create API controller for scoring requests
- [ ] Design database schema for scores storage
- [ ] Create base models (VesselCargoScore, ScoreHistory)

#### Python Tasks:
- [ ] Set up FastAPI project structure
- [ ] Install LangChain, LangGraph, LangSmith
- [ ] Create /health endpoint
- [ ] Create /score endpoint (stub)
- [ ] Test basic Laravel → Python communication

#### DevOps Tasks:
- [ ] Set up MongoDB Atlas cluster
- [ ] Configure RabbitMQ server
- [ ] Set up Redis for caching
- [ ] Configure environment variables

**Success Criteria:**
✅ Laravel can send request to Python
✅ Python can respond with dummy data
✅ Data is queued in RabbitMQ
✅ MongoDB connection works

---

### 3.2 Phase 2: P1 Scoring (Week 3)

**Goal:** Implement first working scoring criterion (Proximity)

#### Tasks:
- [ ] Python: Implement P1 proximity scoring logic
- [ ] Python: Read `proximity_scoring_matrix.txt`
- [ ] Python: Calculate distance between vessel and loading port
- [ ] Laravel: Prepare vessel location data
- [ ] Laravel: Store P1 scores in database
- [ ] Test end-to-end P1 scoring

**Files to use:**
- `proximity_scoring_matrix.txt` - P1 scoring rules
- `vessel_location_determination.txt` - how to determine vessel position
- `vessel_size_classification.txt` - vessel size categories

**Success Criteria:**
✅ P1 score calculated correctly for test cases
✅ Score saved to database
✅ Reasoning is clear and accurate

---

### 3.3 Phase 3: RAG Setup + P1A (Week 4)

**Goal:** Set up RAG and implement P1A (Regional Patterns)

#### Tasks:
- [ ] MongoDB: Create vector indexes
- [ ] Python: Implement embeddings generation
- [ ] Python: Import knowledge bases to MongoDB
- [ ] Python: Implement RAG retrieval for P1A
- [ ] Python: P1A scoring logic
- [ ] Test RAG queries

**Files to use:**
- `REGIONAL_TRADE_PATTERNS_KB.txt` - knowledge base
- `P1A_REGIONAL_SCORING_RULES.txt` - P1A algorithm

**Success Criteria:**
✅ RAG returns relevant trade patterns
✅ P1A score calculated with RAG context
✅ Vector search works efficiently

---

### 3.4 Phase 4: Comments Parsing (P2, P3, P6) (Week 5-6)

**Goal:** Implement AI-powered comment parsing

#### Tasks:
- [ ] Python: Implement comment parsing logic
- [ ] Python: Extract preferences from comments
- [ ] Python: P2 Regional Preferences scoring
- [ ] Python: P3 Cargo Preferences scoring
- [ ] Python: P6 OpenArea Comments scoring
- [ ] Test with real comment examples

**Files to use:**
- `COMMENTS_PROCESSING_RULES.txt` - parsing rules
- `COMMENTS_PROCESSING_PROMPTS.txt` - AI prompts
- `COMMENTS_PROCESSING_EXAMPLES.txt` - test examples

**Success Criteria:**
✅ Comments parsed accurately
✅ Preferences extracted correctly
✅ P2, P3, P6 scores calculated

---

### 3.5 Phase 5: Remaining Scoring (P4, P5, P7) (Week 7)

**Goal:** Complete all scoring criteria

#### P4 Tasks:
- [ ] Laravel: Query vessel port history
- [ ] Python: Calculate familiarity scores
- [ ] Test P4 scoring

#### P5 Tasks:
- [ ] Python: Implement intake calculator
- [ ] Python: Read port restrictions
- [ ] Test P5 with different scenarios

#### P7 Tasks:
- [ ] Python: Implement ETA calculation
- [ ] Python: Compare with laycan dates
- [ ] Test P7 timing logic

**Files to use:**
- `intake_calculator_formula.txt` - P5 formulas
- `port_restrictions_map.txt` - port data
- `READINESSETA_TO_LOADING_PORT_SCORING.txt` - P7 logic

**Success Criteria:**
✅ All P1-P7 scores calculated
✅ Final combined score accurate
✅ All test cases pass

---

### 3.6 Phase 6: Message Generation (Week 8)

**Goal:** Implement AI offer text generation

#### Tasks:
- [ ] Python: Create MessageAgent
- [ ] Python: Load message templates
- [ ] Python: Personalization logic
- [ ] Laravel: Send offers via email
- [ ] Test different message types

**Files to use:**
- `emailsexamples.txt` - message templates
- `ABBREVIATIONS_KNOWLEDGE_BASE.txt` - terminology

**Success Criteria:**
✅ Personalized offers generated
✅ Messages sent successfully
✅ Style matches examples

---

### 3.7 Phase 7: Learning & Corrections (Week 9-10)

**Goal:** Implement behavioral cloning

#### Tasks:
- [ ] Laravel: UI for manual corrections
- [ ] Laravel: Store correction history
- [ ] Python: Learning endpoint
- [ ] Python: Update LangSmith datasets
- [ ] Test correction workflow

**Success Criteria:**
✅ Users can correct scores
✅ Corrections stored
✅ AI improves over time

---

### 3.8 Phase 8: Testing & Optimization (Week 11-12)

**Goal:** Production readiness

#### Tasks:
- [ ] Comprehensive testing
- [ ] Performance optimization
- [ ] Error handling
- [ ] Documentation
- [ ] Deployment preparation

---

## 4. LARAVEL BACKEND IMPLEMENTATION

### 4.1 Database Schema

```php
// Migration: vessel_cargo_scores
Schema::create('vessel_cargo_scores', function (Blueprint $table) {
    $table->id();
    $table->foreignId('vessel_id')->constrained();
    $table->foreignId('cargo_id')->constrained();
    $table->foreignId('scored_by_user_id')->nullable()->constrained('users');
    
    // Individual scores
    $table->decimal('score_p1', 5, 2)->nullable(); // Proximity
    $table->decimal('score_p1a', 5, 2)->nullable(); // Regional Patterns
    $table->decimal('score_p2', 5, 2)->nullable(); // Regional Preferences
    $table->decimal('score_p3', 5, 2)->nullable(); // Cargo Preferences
    $table->decimal('score_p4', 5, 2)->nullable(); // Last Ports
    $table->decimal('score_p5', 5, 2)->nullable(); // Intake
    $table->decimal('score_p6', 5, 2)->nullable(); // OpenArea Comments
    $table->decimal('score_p7', 5, 2)->nullable(); // Readiness/ETA
    
    // Combined scores
    $table->decimal('base_score', 5, 2); // Sum of P1-P7
    $table->decimal('final_score', 5, 2); // With P6 modifier
    
    // Reasoning (JSON)
    $table->json('reasoning')->nullable();
    
    // Blocking
    $table->json('blocking_conditions')->nullable();
    $table->boolean('is_blocked')->default(false);
    
    // Metadata
    $table->enum('score_version', ['ai_v1', 'manual_override'])->default('ai_v1');
    $table->timestamp('scored_at');
    $table->timestamps();
    
    // Indexes
    $table->index(['vessel_id', 'cargo_id', 'scored_at']);
    $table->index('final_score');
});

// Migration: score_corrections
Schema::create('score_corrections', function (Blueprint $table) {
    $table->id();
    $table->foreignId('vessel_cargo_score_id')->constrained();
    $table->foreignId('user_id')->constrained();
    
    $table->string('criterion'); // P1, P2, etc.
    $table->decimal('original_score', 5, 2);
    $table->decimal('corrected_score', 5, 2);
    $table->text('user_reasoning')->nullable();
    
    $table->boolean('sent_to_ai')->default(false);
    $table->timestamp('sent_to_ai_at')->nullable();
    
    $table->timestamps();
});
```

### 4.2 Service Architecture

```php
// app/Services/ScoringService.php

namespace App\Services;

use App\Models\Vessel;
use App\Models\Cargo;
use App\Models\VesselCargoScore;
use App\Jobs\ScoreVesselCargoJob;
use Illuminate\Support\Facades\Http;

class ScoringService
{
    protected $pythonApiUrl;
    
    public function __construct()
    {
        $this->pythonApiUrl = config('services.langchain.url');
    }
    
    /**
     * Queue scoring request
     */
    public function queueScoring(Vessel $vessel, Cargo $cargo): void
    {
        ScoreVesselCargoJob::dispatch($vessel->id, $cargo->id);
    }
    
    /**
     * Prepare data for Python microservice
     */
    public function prepareScoringData(Vessel $vessel, Cargo $cargo): array
    {
        return [
            'vessel' => [
                'id' => $vessel->id,
                'name' => $vessel->name,
                'dwt' => $vessel->dwt,
                'draft_summer' => $vessel->draft_summer,
                'loa' => $vessel->loa,
                'beam' => $vessel->beam,
                'built_year' => $vessel->built_year,
                'flag' => $vessel->flag,
                'gear' => $vessel->has_gear,
                'ice_class' => $vessel->ice_class,
                
                // Position
                'open_area' => $vessel->snapshot?->open_area,
                'open_port' => $vessel->snapshot?->open_port,
                'open_date_from' => $vessel->snapshot?->open_date_from,
                'destination' => $vessel->snapshot?->destination,
                'current_area' => $vessel->snapshot?->current_area,
                
                // Draft & Cargo status
                'current_draft' => $vessel->snapshot?->current_draft,
                'cargo_export_map' => $vessel->snapshot?->cargo_export_map,
                
                // Comments
                'vessel_comment' => $vessel->comment,
                'company_comment' => $vessel->company?->comment,
                
                // Preferences
                'regional_preferences' => $this->getRegionalPreferences($vessel),
                'cargo_preferences' => $this->getCargoPreferences($vessel),
                
                // History
                'port_history' => $this->getPortHistory($vessel),
            ],
            
            'cargo' => [
                'id' => $cargo->id,
                'cargo_type' => $cargo->cargo_type,
                'cargo_group' => $cargo->cargo_group,
                'quantity' => $cargo->quantity,
                'stowage_factor' => $cargo->stowage_factor,
                
                // Ports
                'loading_port' => $cargo->loading_port,
                'loading_port_country' => $cargo->loading_port_country,
                'loading_port_region' => $cargo->loading_port_region,
                'discharge_port' => $cargo->discharge_port,
                'discharge_port_country' => $cargo->discharge_port_country,
                
                // Dates
                'laycan_start' => $cargo->laycan_start,
                'laycan_end' => $cargo->laycan_end,
                
                // Port restrictions
                'loading_port_restrictions' => $this->getPortRestrictions($cargo->loading_port),
                'discharge_port_restrictions' => $this->getPortRestrictions($cargo->discharge_port),
            ],
            
            'context' => [
                'request_id' => uniqid('score_', true),
                'timestamp' => now()->toIso8601String(),
                'user_id' => auth()->id(),
            ],
        ];
    }
    
    /**
     * Send scoring request to Python microservice
     */
    public function requestScoring(array $data): array
    {
        try {
            $response = Http::timeout(120)
                ->retry(3, 1000)
                ->post("{$this->pythonApiUrl}/api/v1/score", $data);
            
            if ($response->successful()) {
                return $response->json();
            }
            
            throw new \Exception("Scoring API returned: " . $response->status());
            
        } catch (\Exception $e) {
            \Log::error('Scoring API Error', [
                'error' => $e->getMessage(),
                'vessel_id' => $data['vessel']['id'],
                'cargo_id' => $data['cargo']['id'],
            ]);
            
            throw $e;
        }
    }
    
    /**
     * Store scoring results
     */
    public function storeScores(int $vesselId, int $cargoId, array $results): VesselCargoScore
    {
        return VesselCargoScore::create([
            'vessel_id' => $vesselId,
            'cargo_id' => $cargoId,
            'scored_by_user_id' => auth()->id(),
            
            'score_p1' => $results['scores']['P1'],
            'score_p1a' => $results['scores']['P1A'],
            'score_p2' => $results['scores']['P2'],
            'score_p3' => $results['scores']['P3'],
            'score_p4' => $results['scores']['P4'],
            'score_p5' => $results['scores']['P5'],
            'score_p6' => $results['scores']['P6'],
            'score_p7' => $results['scores']['P7'],
            
            'base_score' => $results['base_score'],
            'final_score' => $results['final_score'],
            
            'reasoning' => $results['reasoning'],
            'blocking_conditions' => $results['blocking_conditions'],
            'is_blocked' => !empty($results['blocking_conditions']),
            
            'scored_at' => now(),
        ]);
    }
    
    // Helper methods
    protected function getRegionalPreferences(Vessel $vessel): array
    {
        // Load from vessel settings or snapshot
        // Return structured preferences
    }
    
    protected function getCargoPreferences(Vessel $vessel): array
    {
        // Load from vessel settings
    }
    
    protected function getPortHistory(Vessel $vessel): array
    {
        // Query port calls history
    }
    
    protected function getPortRestrictions(string $port): array
    {
        // Load from port_restrictions table
    }
}
```

### 4.3 Queue Job

```php
// app/Jobs/ScoreVesselCargoJob.php

namespace App\Jobs;

use App\Models\Vessel;
use App\Models\Cargo;
use App\Services\ScoringService;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ScoreVesselCargoJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public $timeout = 180; // 3 minutes
    public $tries = 3;
    
    protected $vesselId;
    protected $cargoId;
    
    public function __construct(int $vesselId, int $cargoId)
    {
        $this->vesselId = $vesselId;
        $this->cargoId = $cargoId;
    }
    
    public function handle(ScoringService $scoringService): void
    {
        $vessel = Vessel::with(['company', 'snapshot'])->findOrFail($this->vesselId);
        $cargo = Cargo::findOrFail($this->cargoId);
        
        // Prepare data
        $data = $scoringService->prepareScoringData($vessel, $cargo);
        
        // Request scoring from Python
        $results = $scoringService->requestScoring($data);
        
        // Store results
        $scoringService->storeScores($this->vesselId, $this->cargoId, $results);
        
        \Log::info('Scoring completed', [
            'vessel_id' => $this->vesselId,
            'cargo_id' => $this->cargoId,
            'final_score' => $results['final_score'],
        ]);
    }
    
    public function failed(\Throwable $exception): void
    {
        \Log::error('Scoring job failed', [
            'vessel_id' => $this->vesselId,
            'cargo_id' => $this->cargoId,
            'error' => $exception->getMessage(),
        ]);
        
        // Notify admin or retry logic
    }
}
```

### 4.4 API Controller

```php
// app/Http/Controllers/Api/ScoringController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Vessel;
use App\Models\Cargo;
use App\Services\ScoringService;
use Illuminate\Http\Request;

class ScoringController extends Controller
{
    protected $scoringService;
    
    public function __construct(ScoringService $scoringService)
    {
        $this->scoringService = $scoringService;
    }
    
    /**
     * Queue scoring for vessel-cargo pair
     */
    public function score(Request $request)
    {
        $validated = $request->validate([
            'vessel_id' => 'required|exists:vessels,id',
            'cargo_id' => 'required|exists:cargoes,id',
        ]);
        
        $vessel = Vessel::findOrFail($validated['vessel_id']);
        $cargo = Cargo::findOrFail($validated['cargo_id']);
        
        $this->scoringService->queueScoring($vessel, $cargo);
        
        return response()->json([
            'message' => 'Scoring queued successfully',
            'vessel_id' => $vessel->id,
            'cargo_id' => $cargo->id,
        ]);
    }
    
    /**
     * Get scoring results
     */
    public function getScore(Request $request)
    {
        $validated = $request->validate([
            'vessel_id' => 'required|exists:vessels,id',
            'cargo_id' => 'required|exists:cargoes,id',
        ]);
        
        $score = VesselCargoScore::where('vessel_id', $validated['vessel_id'])
            ->where('cargo_id', $validated['cargo_id'])
            ->latest('scored_at')
            ->first();
        
        if (!$score) {
            return response()->json([
                'message' => 'Score not found',
            ], 404);
        }
        
        return response()->json($score);
    }
    
    /**
     * Submit correction
     */
    public function correct(Request $request)
    {
        $validated = $request->validate([
            'score_id' => 'required|exists:vessel_cargo_scores,id',
            'criterion' => 'required|in:P1,P1A,P2,P3,P4,P5,P6,P7',
            'corrected_score' => 'required|numeric|min:0|max:100',
            'reasoning' => 'nullable|string',
        ]);
        
        $score = VesselCargoScore::findOrFail($validated['score_id']);
        
        $correction = $score->corrections()->create([
            'user_id' => auth()->id(),
            'criterion' => $validated['criterion'],
            'original_score' => $score->{"score_" . strtolower($validated['criterion'])},
            'corrected_score' => $validated['corrected_score'],
            'user_reasoning' => $validated['reasoning'],
        ]);
        
        // Send to Python for learning
        $this->scoringService->sendCorrectionToAI($correction);
        
        return response()->json([
            'message' => 'Correction saved',
            'correction' => $correction,
        ]);
    }
}
```

---

## 5. PYTHON LANGCHAIN MICROSERVICE

### 5.1 Project Structure

```
langchain-service/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app
│   ├── config.py               # Configuration
│   ├── models/                 # Pydantic models
│   │   ├── __init__.py
│   │   ├── scoring_request.py
│   │   └── scoring_response.py
│   ├── agents/                 # LangGraph agents
│   │   ├── __init__.py
│   │   ├── scoring_agent.py   # Main scoring agent
│   │   └── message_agent.py   # Message generation
│   ├── chains/                 # LangChain chains
│   │   ├── __init__.py
│   │   ├── p1_proximity.py
│   │   ├── p1a_regional.py
│   │   ├── p2_preferences.py
│   │   └── ... (P3-P7)
│   ├── rag/                    # RAG components
│   │   ├── __init__.py
│   │   ├── embeddings.py
│   │   ├── vector_store.py
│   │   └── retrievers.py
│   ├── prompts/                # Prompt templates
│   │   ├── __init__.py
│   │   ├── scoring_prompts.py
│   │   └── message_prompts.py
│   ├── utils/                  # Utilities
│   │   ├── __init__.py
│   │   ├── distance_calc.py
│   │   ├── intake_calc.py
│   │   └── parsers.py
│   └── knowledge/              # Knowledge base loaders
│       ├── __init__.py
│       ├── loader.py
│       └── schemas.py
├── knowledge_bases/            # KB files from project
│   ├── REGIONAL_TRADE_PATTERNS_KB.txt
│   ├── port_restrictions_map.txt
│   └── ... (other KB files)
├── rules/                      # Rules files from project
│   ├── COMMENTS_PROCESSING_RULES.txt
│   ├── P1A_REGIONAL_SCORING_RULES.txt
│   └── ... (other rules files)
├── prompts/                    # Prompts files from project
│   ├── COMMENTS_PROCESSING_PROMPTS.txt
│   └── ... (other prompts files)
├── tests/
│   ├── test_scoring.py
│   └── test_rag.py
├── requirements.txt
├── .env
└── docker-compose.yml
```

### 5.2 FastAPI Setup

```python
# app/main.py

from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import Dict, Any, Optional
import logging

from app.agents.scoring_agent import ScoringAgent
from app.models.scoring_request import ScoringRequest
from app.models.scoring_response import ScoringResponse
from app.config import settings

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Initialize FastAPI
app = FastAPI(
    title="VK Chart AI - Scoring Service",
    version="1.0.0",
    description="AI-powered vessel-cargo scoring microservice"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Initialize Scoring Agent (singleton)
scoring_agent = None

@app.on_event("startup")
async def startup_event():
    """Initialize components on startup"""
    global scoring_agent
    logger.info("Initializing Scoring Agent...")
    scoring_agent = ScoringAgent()
    await scoring_agent.initialize()
    logger.info("Scoring Agent ready")

@app.get("/")
async def root():
    """Health check"""
    return {
        "status": "ok",
        "service": "VK Chart AI Scoring Service",
        "version": "1.0.0"
    }

@app.get("/health")
async def health():
    """Detailed health check"""
    return {
        "status": "healthy",
        "components": {
            "scoring_agent": scoring_agent is not None,
            "mongodb": await scoring_agent.check_mongodb_connection(),
            "langsmith": settings.LANGSMITH_API_KEY is not None,
        }
    }

@app.post("/api/v1/score", response_model=ScoringResponse)
async def score_vessel_cargo(request: ScoringRequest):
    """
    Main scoring endpoint
    
    Receives vessel and cargo data from Laravel,
    calculates P1-P7 scores, returns results
    """
    try:
        logger.info(f"Scoring request: vessel={request.vessel.id}, cargo={request.cargo.id}")
        
        # Run scoring through agent
        result = await scoring_agent.score(request)
        
        logger.info(f"Scoring complete: final_score={result.final_score}")
        
        return result
        
    except Exception as e:
        logger.error(f"Scoring error: {str(e)}", exc_info=True)
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/api/v1/generate-message")
async def generate_message(request: Dict[str, Any]):
    """Generate personalized offer message"""
    try:
        # TODO: Implement message generation
        return {"message": "Message generation not yet implemented"}
    except Exception as e:
        logger.error(f"Message generation error: {str(e)}")
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/api/v1/learn")
async def learn_from_correction(correction: Dict[str, Any]):
    """Receive user correction for learning"""
    try:
        # TODO: Implement learning logic
        logger.info(f"Correction received: {correction}")
        return {"status": "correction_saved"}
    except Exception as e:
        logger.error(f"Learning error: {str(e)}")
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 5.3 Pydantic Models

```python
# app/models/scoring_request.py

from pydantic import BaseModel, Field
from typing import Optional, Dict, List
from datetime import date, datetime

class VesselPosition(BaseModel):
    open_area: Optional[str] = None
    open_port: Optional[str] = None
    open_date_from: Optional[date] = None
    destination: Optional[str] = None
    current_area: Optional[str] = None

class VesselData(BaseModel):
    id: int
    name: str
    dwt: float
    draft_summer: float
    loa: float
    beam: float
    built_year: int
    flag: str
    gear: bool
    ice_class: Optional[str] = None
    
    # Position
    open_area: Optional[str] = None
    open_port: Optional[str] = None
    open_date_from: Optional[date] = None
    destination: Optional[str] = None
    current_area: Optional[str] = None
    
    # Status
    current_draft: Optional[float] = None
    cargo_export_map: Optional[str] = None
    
    # Comments
    vessel_comment: Optional[str] = None
    company_comment: Optional[str] = None
    
    # Preferences
    regional_preferences: Optional[Dict] = None
    cargo_preferences: Optional[Dict] = None
    
    # History
    port_history: Optional[List[Dict]] = None

class CargoData(BaseModel):
    id: int
    cargo_type: str
    cargo_group: Optional[str] = None
    quantity: float
    stowage_factor: Optional[float] = None
    
    # Ports
    loading_port: str
    loading_port_country: str
    loading_port_region: str
    discharge_port: str
    discharge_port_country: str
    
    # Dates
    laycan_start: date
    laycan_end: date
    
    # Restrictions
    loading_port_restrictions: Optional[Dict] = None
    discharge_port_restrictions: Optional[Dict] = None

class ContextData(BaseModel):
    request_id: str
    timestamp: datetime
    user_id: Optional[int] = None

class ScoringRequest(BaseModel):
    vessel: VesselData
    cargo: CargoData
    context: ContextData

# app/models/scoring_response.py

from pydantic import BaseModel
from typing import Dict, Optional, List

class IndividualScores(BaseModel):
    P1: float = Field(..., ge=0, le=20)
    P1A: float = Field(..., ge=0, le=15)
    P2: float = Field(..., ge=0, le=15)
    P3: float = Field(..., ge=0, le=15)
    P4: float = Field(..., ge=0, le=10)
    P5: float = Field(..., ge=0, le=15)
    P6: float = Field(..., ge=-50, le=25)
    P7: float = Field(..., ge=0, le=10)

class ScoringResponse(BaseModel):
    scores: IndividualScores
    base_score: float  # Sum of P1-P7 (excluding P6)
    final_score: float  # Base + P6 modifier, clamped [0, 100]
    
    reasoning: Dict[str, str]  # Explanation for each criterion
    
    blocking_conditions: Optional[List[str]] = None
    
    metadata: Dict = {
        "model_version": "v1",
        "processing_time_ms": 0,
    }
```

### 5.4 Scoring Agent (LangGraph)

```python
# app/agents/scoring_agent.py

from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage
import logging

from app.chains.p1_proximity import P1ProximityChain
from app.chains.p1a_regional import P1ARegionalChain
# ... import other chains
from app.rag.vector_store import VectorStoreManager
from app.config import settings

logger = logging.getLogger(__name__)

class ScoringState(TypedDict):
    """State for scoring workflow"""
    vessel_data: dict
    cargo_data: dict
    context: dict
    
    # Scores
    score_p1: float
    score_p1a: float
    score_p2: float
    score_p3: float
    score_p4: float
    score_p5: float
    score_p6: float
    score_p7: float
    
    # Reasoning
    reasoning: dict
    
    # Blocking
    blocking_conditions: list
    is_blocked: bool

class ScoringAgent:
    """Main scoring agent using LangGraph for state management"""
    
    def __init__(self):
        self.llm = ChatOpenAI(
            model=settings.OPENAI_MODEL,
            temperature=0,
            api_key=settings.OPENAI_API_KEY
        )
        
        # Initialize chains
        self.p1_chain = P1ProximityChain(self.llm)
        self.p1a_chain = P1ARegionalChain(self.llm)
        # ... initialize other chains
        
        # Initialize RAG
        self.vector_store = None
        
        # Build graph
        self.graph = self._build_graph()
    
    async def initialize(self):
        """Initialize components (call on startup)"""
        # Initialize MongoDB vector store
        self.vector_store = VectorStoreManager()
        await self.vector_store.connect()
        
        # Initialize chains with vector store
        await self.p1a_chain.initialize(self.vector_store)
        
        logger.info("Scoring Agent initialized")
    
    def _build_graph(self) -> StateGraph:
        """Build LangGraph workflow"""
        workflow = StateGraph(ScoringState)
        
        # Add nodes for each criterion
        workflow.add_node("p1", self._score_p1)
        workflow.add_node("p1a", self._score_p1a)
        workflow.add_node("p2", self._score_p2)
        workflow.add_node("p3", self._score_p3)
        workflow.add_node("p4", self._score_p4)
        workflow.add_node("p5", self._score_p5)
        workflow.add_node("p6", self._score_p6)
        workflow.add_node("p7", self._score_p7)
        workflow.add_node("finalize", self._finalize_scores)
        
        # Define edges (progressive scoring)
        workflow.set_entry_point("p1")
        workflow.add_edge("p1", "p1a")
        workflow.add_edge("p1a", "p2")
        workflow.add_edge("p2", "p3")
        workflow.add_edge("p3", "p4")
        workflow.add_edge("p4", "p5")
        workflow.add_edge("p5", "p6")
        workflow.add_edge("p6", "p7")
        workflow.add_edge("p7", "finalize")
        workflow.add_edge("finalize", END)
        
        return workflow.compile()
    
    async def score(self, request: ScoringRequest) -> ScoringResponse:
        """Run scoring workflow"""
        
        # Initialize state
        initial_state = {
            "vessel_data": request.vessel.dict(),
            "cargo_data": request.cargo.dict(),
            "context": request.context.dict(),
            "reasoning": {},
            "blocking_conditions": [],
            "is_blocked": False,
        }
        
        # Run graph
        final_state = await self.graph.ainvoke(initial_state)
        
        # Build response
        response = ScoringResponse(
            scores=IndividualScores(
                P1=final_state["score_p1"],
                P1A=final_state["score_p1a"],
                P2=final_state["score_p2"],
                P3=final_state["score_p3"],
                P4=final_state["score_p4"],
                P5=final_state["score_p5"],
                P6=final_state["score_p6"],
                P7=final_state["score_p7"],
            ),
            base_score=final_state["base_score"],
            final_score=final_state["final_score"],
            reasoning=final_state["reasoning"],
            blocking_conditions=final_state["blocking_conditions"] if final_state["is_blocked"] else None,
        )
        
        return response
    
    async def _score_p1(self, state: ScoringState) -> ScoringState:
        """Score P1: Proximity"""
        logger.info("Scoring P1: Proximity")
        
        result = await self.p1_chain.score(
            vessel=state["vessel_data"],
            cargo=state["cargo_data"]
        )
        
        state["score_p1"] = result["score"]
        state["reasoning"]["P1"] = result["reasoning"]
        
        return state
    
    async def _score_p1a(self, state: ScoringState) -> ScoringState:
        """Score P1A: Regional Patterns (with RAG)"""
        logger.info("Scoring P1A: Regional Patterns")
        
        result = await self.p1a_chain.score(
            vessel=state["vessel_data"],
            cargo=state["cargo_data"]
        )
        
        state["score_p1a"] = result["score"]
        state["reasoning"]["P1A"] = result["reasoning"]
        
        return state
    
    # ... similar methods for P2-P7
    
    async def _finalize_scores(self, state: ScoringState) -> ScoringState:
        """Calculate final combined scores"""
        
        # Base score (P1 + P1A + P2 + P3 + P4 + P5 + P7)
        base_score = (
            state["score_p1"] +
            state["score_p1a"] +
            state["score_p2"] +
            state["score_p3"] +
            state["score_p4"] +
            state["score_p5"] +
            state["score_p7"]
        )
        
        # Final score (base + P6 modifier, clamped [0, 100])
        final_score = max(0, min(100, base_score + state["score_p6"]))
        
        state["base_score"] = base_score
        state["final_score"] = final_score
        
        # Check blocking conditions
        if state["score_p6"] <= -40:
            state["is_blocked"] = True
            state["blocking_conditions"].append("P6 score too negative (strong conflict)")
        
        return state
    
    async def check_mongodb_connection(self) -> bool:
        """Health check for MongoDB"""
        try:
            return await self.vector_store.is_connected()
        except:
            return False
```

### 5.5 Example Chain: P1 Proximity

```python
# app/chains/p1_proximity.py

from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import JsonOutputParser
from typing import Dict
import math

class P1ProximityChain:
    """Chain for P1: Proximity scoring"""
    
    def __init__(self, llm: ChatOpenAI):
        self.llm = llm
        self.output_parser = JsonOutputParser()
        
        # Load proximity scoring matrix from file
        self.scoring_matrix = self._load_scoring_matrix()
    
    def _load_scoring_matrix(self) -> Dict:
        """Load proximity_scoring_matrix.txt"""
        # TODO: Parse proximity_scoring_matrix.txt
        # Return structured scoring rules
        return {}
    
    async def score(self, vessel: Dict, cargo: Dict) -> Dict:
        """Calculate P1 proximity score"""
        
        # 1. Determine vessel location
        vessel_location = self._determine_vessel_location(vessel)
        
        # 2. Get loading port
        loading_port = cargo["loading_port"]
        loading_region = cargo["loading_port_region"]
        
        # 3. Calculate distance
        distance_nm = self._calculate_distance(vessel_location, loading_port)
        
        # 4. Determine vessel size category
        vessel_size = self._get_vessel_size_category(vessel["dwt"])
        
        # 5. Look up score from matrix
        internal_score = self._lookup_score_from_matrix(
            vessel_location, 
            loading_region, 
            distance_nm,
            vessel_size
        )
        
        # 6. Scale internal score [0-50] to P1 range [0-20]
        p1_score = self._scale_score(internal_score, 0, 50, 0, 20)
        
        # 7. Generate reasoning
        reasoning = self._generate_reasoning(
            vessel_location,
            loading_port,
            distance_nm,
            internal_score,
            p1_score
        )
        
        return {
            "score": p1_score,
            "reasoning": reasoning,
            "metadata": {
                "distance_nm": distance_nm,
                "vessel_location": vessel_location,
                "internal_score": internal_score,
            }
        }
    
    def _determine_vessel_location(self, vessel: Dict) -> str:
        """
        Determine vessel's current location
        
        Priority:
        1. Open Information (open_area, open_port)
        2. Destination
        3. Current Area
        4. Unknown
        
        See: vessel_location_determination.txt
        """
        if vessel.get("open_area"):
            return vessel["open_area"]
        elif vessel.get("open_port"):
            return vessel["open_port"]
        elif vessel.get("destination"):
            # Validate it's a real location (not "to order", "TBN", etc.)
            dest = vessel["destination"]
            if not self._is_non_geographic(dest):
                return dest
        elif vessel.get("current_area"):
            return vessel["current_area"]
        
        return "Unknown"
    
    def _is_non_geographic(self, text: str) -> bool:
        """Check if text is non-geographic (orders, TBN, etc.)"""
        non_geo_keywords = [
            "to order", "for orders", "awaiting orders",
            "tbn", "tba", "to be nominated",
            "crew", "inco", "awaiting instructions",
            "owners option"
        ]
        return any(kw in text.lower() for kw in non_geo_keywords)
    
    def _calculate_distance(self, from_location: str, to_location: str) -> float:
        """
        Calculate nautical miles between two locations
        
        Options:
        1. Use CERDIseadistance.xlsx lookup
        2. Use VesselFinder API (with caching)
        3. Use coordinate-based calculation
        """
        # TODO: Implement distance calculation
        # For now, return dummy value
        return 250.0
    
    def _get_vessel_size_category(self, dwt: float) -> str:
        """
        Get vessel size category
        
        See: vessel_size_classification.txt
        """
        if dwt <= 4000:
            return "SC"  # Small Coaster
        elif dwt <= 17000:
            return "C"   # Coaster
        elif dwt <= 24000:
            return "SH"  # Small Handy
        elif dwt <= 40000:
            return "H"   # Handy
        elif dwt <= 50000:
            return "SS"  # Small Supramax
        elif dwt <= 65000:
            return "SM"  # Supramax
        else:
            return "PM"  # Panamax
    
    def _lookup_score_from_matrix(
        self, 
        vessel_location: str, 
        loading_region: str,
        distance_nm: float,
        vessel_size: str
    ) -> float:
        """
        Look up internal score from proximity matrix
        
        See: proximity_scoring_matrix.txt for full matrix
        """
        # TODO: Implement matrix lookup
        # This is simplified - real implementation needs full matrix
        
        # Basic logic:
        if distance_nm < 100:
            return 50  # Same port/very close
        elif distance_nm < 300:
            return 40  # Same region
        elif distance_nm < 600:
            return 30  # Nearby region
        elif distance_nm < 1000:
            return 20  # Medium distance
        else:
            return 10  # Far distance
    
    def _scale_score(
        self, 
        value: float, 
        from_min: float, 
        from_max: float, 
        to_min: float, 
        to_max: float
    ) -> float:
        """Scale score from one range to another"""
        scaled = ((value - from_min) / (from_max - from_min)) * (to_max - to_min) + to_min
        return round(max(to_min, min(to_max, scaled)), 2)
    
    def _generate_reasoning(
        self,
        vessel_location: str,
        loading_port: str,
        distance_nm: float,
        internal_score: float,
        p1_score: float
    ) -> str:
        """Generate human-readable reasoning"""
        return (
            f"Vessel is currently at {vessel_location}, "
            f"approximately {distance_nm:.0f} nautical miles from loading port {loading_port}. "
            f"This distance results in an internal proximity score of {internal_score}/50, "
            f"which scales to P1 score of {p1_score}/20."
        )
```

### 5.6 RAG Setup

```python
# app/rag/vector_store.py

from motor.motor_asyncio import AsyncIOMotorClient
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from typing import List, Dict
import logging

from app.config import settings

logger = logging.getLogger(__name__)

class VectorStoreManager:
    """Manage MongoDB Atlas Vector Search"""
    
    def __init__(self):
        self.client = None
        self.db = None
        self.embeddings = OpenAIEmbeddings(
            model="text-embedding-3-small",
            api_key=settings.OPENAI_API_KEY
        )
        self.vector_stores = {}
    
    async def connect(self):
        """Connect to MongoDB Atlas"""
        try:
            self.client = AsyncIOMotorClient(settings.MONGODB_URI)
            self.db = self.client[settings.MONGODB_DB_NAME]
            
            # Test connection
            await self.client.admin.command('ping')
            
            logger.info("Connected to MongoDB Atlas")
            
            # Initialize vector stores for different collections
            await self._initialize_vector_stores()
            
        except Exception as e:
            logger.error(f"MongoDB connection error: {e}")
            raise
    
    async def _initialize_vector_stores(self):
        """Initialize vector stores for different knowledge bases"""
        
        collections = [
            "regional_trade_patterns",
            "port_restrictions",
            "vessel_comments",
            "cargo_patterns",
        ]
        
        for collection_name in collections:
            self.vector_stores[collection_name] = MongoDBAtlasVectorSearch(
                collection=self.db[collection_name],
                embedding=self.embeddings,
                index_name="vector_index",
                text_key="text",
                embedding_key="embedding",
            )
            
            logger.info(f"Initialized vector store: {collection_name}")
    
    async def similarity_search(
        self, 
        collection: str, 
        query: str, 
        k: int = 5,
        filters: Dict = None
    ) -> List[Dict]:
        """
        Perform similarity search
        
        Args:
            collection: Collection name
            query: Search query
            k: Number of results
            filters: Optional MongoDB filters
        """
        if collection not in self.vector_stores:
            raise ValueError(f"Unknown collection: {collection}")
        
        vector_store = self.vector_stores[collection]
        
        # Perform search
        results = await vector_store.asimilarity_search(
            query=query,
            k=k,
            pre_filter=filters
        )
        
        return [
            {
                "text": doc.page_content,
                "metadata": doc.metadata,
                "score": doc.metadata.get("score", 0)
            }
            for doc in results
        ]
    
    async def is_connected(self) -> bool:
        """Check if connected to MongoDB"""
        try:
            await self.client.admin.command('ping')
            return True
        except:
            return False
```

---

## 6. MONGODB ATLAS SETUP

### 6.1 Collections Structure

```javascript
// Collection: regional_trade_patterns
{
  _id: ObjectId,
  text: "Coasters from Black Sea to Marmara Sea with grain are very common...",
  embedding: [0.123, -0.456, ...],  // 1536 dimensions
  metadata: {
    vessel_size: "C",  // Coaster
    origin_region: "Black Sea",
    destination_region: "Marmara Sea",
    cargo_type: "grain",
    frequency: "very_common"
  }
}

// Collection: port_restrictions
{
  _id: ObjectId,
  text: "Istanbul: Max draft 12.5m, max LOA 220m, no beam restriction...",
  embedding: [...],
  metadata: {
    port_name: "Istanbul",
    country: "Turkey",
    max_draft: 12.5,
    max_loa: 220,
    max_beam: null,
    gear_required: false
  }
}

// Collection: vessel_comments
{
  _id: ObjectId,
  vessel_id: 12345,
  text: "Vessel prefers Black Sea to Mediterranean trades, no Egypt",
  embedding: [...],
  metadata: {
    vessel_id: 12345,
    comment_type: "preferences",
    extracted_preferences: {
      regions: ["Black Sea", "Mediterranean"],
      avoid: ["Egypt"]
    }
  }
}

// Collection: scoring_history
{
  _id: ObjectId,
  vessel_id: 12345,
  cargo_id: 67890,
  scores: {
    P1: 18, P1A: 12, ...
  },
  final_score: 85,
  reasoning: {...},
  scored_at: ISODate(...),
  scored_by: "ai_v1",
  corrected: false
}

// Collection: corrections
{
  _id: ObjectId,
  vessel_cargo_score_id: "...",
  original_score: 12,
  corrected_score: 15,
  criterion: "P1A",
  user_reasoning: "Vessel frequently operates this route",
  user_id: 123,
  created_at: ISODate(...)
}
```

### 6.2 Vector Indexes

```javascript
// Create vector index for regional_trade_patterns
db.regional_trade_patterns.createSearchIndex({
  name: "vector_index",
  type: "vectorSearch",
  definition: {
    fields: [
      {
        type: "vector",
        path: "embedding",
        numDimensions: 1536,
        similarity: "cosine"
      },
      {
        type: "filter",
        path: "metadata.vessel_size"
      },
      {
        type: "filter",
        path: "metadata.origin_region"
      }
    ]
  }
});

// Similar indexes for other collections...
```

### 6.3 Data Import Script

```python
# scripts/import_knowledge_bases.py

import asyncio
from motor.motor_asyncio import AsyncIOMotorClient
from langchain_openai import OpenAIEmbeddings
from typing import List, Dict
import re

class KnowledgeBaseImporter:
    """Import knowledge base files to MongoDB with embeddings"""
    
    def __init__(self, mongodb_uri: str, db_name: str):
        self.client = AsyncIOMotorClient(mongodb_uri)
        self.db = self.client[db_name]
        self.embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
    
    async def import_regional_trade_patterns(self, filepath: str):
        """Import REGIONAL_TRADE_PATTERNS_KB.txt"""
        
        with open(filepath, 'r', encoding='utf-8') as f:
            content = f.read()
        
        # Parse file into chunks
        chunks = self._parse_trade_patterns(content)
        
        # Generate embeddings
        texts = [chunk["text"] for chunk in chunks]
        embeddings = await self.embeddings.aembed_documents(texts)
        
        # Prepare documents
        documents = []
        for chunk, embedding in zip(chunks, embeddings):
            doc = {
                "text": chunk["text"],
                "embedding": embedding,
                "metadata": chunk["metadata"]
            }
            documents.append(doc)
        
        # Insert to MongoDB
        collection = self.db["regional_trade_patterns"]
        await collection.delete_many({})  # Clear existing
        result = await collection.insert_many(documents)
        
        print(f"Imported {len(documents)} trade patterns")
    
    def _parse_trade_patterns(self, content: str) -> List[Dict]:
        """Parse trade patterns file into structured chunks"""
        chunks = []
        
        # TODO: Implement proper parsing based on file structure
        # For now, simple paragraph splitting
        paragraphs = content.split('\n\n')
        
        for para in paragraphs:
            if len(para.strip()) < 50:
                continue
            
            # Extract metadata from text
            metadata = self._extract_trade_pattern_metadata(para)
            
            chunks.append({
                "text": para.strip(),
                "metadata": metadata
            })
        
        return chunks
    
    def _extract_trade_pattern_metadata(self, text: str) -> Dict:
        """Extract metadata from trade pattern text"""
        metadata = {}
        
        # Extract vessel size
        if "coaster" in text.lower():
            metadata["vessel_size"] = "C"
        elif "handy" in text.lower():
            metadata["vessel_size"] = "H"
        
        # Extract regions
        regions = ["Black Sea", "Mediterranean", "Marmara", "East Med"]
        for region in regions:
            if region.lower() in text.lower():
                metadata.setdefault("regions", []).append(region)
        
        return metadata
    
    # Similar methods for other knowledge bases...
    
async def main():
    importer = KnowledgeBaseImporter(
        mongodb_uri="mongodb+srv://...",
        db_name="vkchart_ai"
    )
    
    # Import all knowledge bases
    await importer.import_regional_trade_patterns(
        "../knowledge_bases/REGIONAL_TRADE_PATTERNS_KB.txt"
    )
    # ... import other files

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 7. INTEGRATION & TESTING

### 7.1 Environment Setup

```bash
# Laravel .env
LANGCHAIN_API_URL=http://localhost:8000
LANGCHAIN_API_TIMEOUT=120
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_QUEUE=scoring_queue
MONGODB_URI=mongodb+srv://...
REDIS_HOST=localhost
REDIS_PORT=6379

# Python .env
OPENAI_API_KEY=sk-...
LANGSMITH_API_KEY=ls-...
LANGSMITH_PROJECT=vkchart-ai-prod
MONGODB_URI=mongodb+srv://...
MONGODB_DB_NAME=vkchart_ai
ALLOWED_ORIGINS=http://localhost,https://vkchart.com
```

### 7.2 Testing Strategy

```python
# tests/test_scoring.py

import pytest
from app.agents.scoring_agent import ScoringAgent
from app.models.scoring_request import ScoringRequest, VesselData, CargoData, ContextData
from datetime import date, datetime

@pytest.fixture
async def scoring_agent():
    agent = ScoringAgent()
    await agent.initialize()
    return agent

@pytest.mark.asyncio
async def test_p1_proximity_same_region(scoring_agent):
    """Test P1 scoring for vessel in same region as loading port"""
    
    request = ScoringRequest(
        vessel=VesselData(
            id=1,
            name="Test Vessel",
            dwt=8500,  # Coaster
            draft_summer=6.5,
            loa=115,
            beam=16,
            built_year=2010,
            flag="Malta",
            gear=True,
            open_area="Black Sea",
            open_port="Constanta",
        ),
        cargo=CargoData(
            id=1,
            cargo_type="wheat",
            quantity=5000,
            loading_port="Odessa",
            loading_port_country="Ukraine",
            loading_port_region="Black Sea",
            discharge_port="Istanbul",
            discharge_port_country="Turkey",
            laycan_start=date(2024, 12, 15),
            laycan_end=date(2024, 12, 20),
        ),
        context=ContextData(
            request_id="test_001",
            timestamp=datetime.now(),
        )
    )
    
    result = await scoring_agent.score(request)
    
    # Assert P1 score is high (vessel in same region)
    assert result.scores.P1 >= 15
    assert "Black Sea" in result.reasoning["P1"]

@pytest.mark.asyncio
async def test_full_scoring_workflow(scoring_agent):
    """Test complete scoring workflow P1-P7"""
    
    # TODO: Create comprehensive test case
    # with all vessel and cargo data
    
    result = await scoring_agent.score(request)
    
    # Assertions
    assert 0 <= result.final_score <= 100
    assert len(result.reasoning) == 8  # P1-P7 + summary
    assert result.base_score == sum([
        result.scores.P1,
        result.scores.P1A,
        result.scores.P2,
        result.scores.P3,
        result.scores.P4,
        result.scores.P5,
        result.scores.P7,
    ])

# ... more tests
```

### 7.3 Integration Testing

```php
// tests/Feature/ScoringIntegrationTest.php

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\Vessel;
use App\Models\Cargo;
use App\Services\ScoringService;
use Illuminate\Support\Facades\Queue;
use Illuminate\Support\Facades\Http;

class ScoringIntegrationTest extends TestCase
{
    public function test_scoring_workflow_end_to_end()
    {
        Queue::fake();
        
        $vessel = Vessel::factory()->create([
            'dwt' => 8500,
            'open_area' => 'Black Sea',
        ]);
        
        $cargo = Cargo::factory()->create([
            'cargo_type' => 'wheat',
            'loading_port' => 'Odessa',
        ]);
        
        // Mock Python API response
        Http::fake([
            '*/api/v1/score' => Http::response([
                'scores' => [
                    'P1' => 18,
                    'P1A' => 12,
                    'P2' => 10,
                    'P3' => 12,
                    'P4' => 8,
                    'P5' => 14,
                    'P6' => 5,
                    'P7' => 9,
                ],
                'base_score' => 83,
                'final_score' => 88,
                'reasoning' => [...],
                'blocking_conditions' => null,
            ], 200),
        ]);
        
        // Request scoring
        $service = app(ScoringService::class);
        $service->queueScoring($vessel, $cargo);
        
        // Assert job was dispatched
        Queue::assertPushed(ScoreVesselCargoJob::class);
        
        // Process job
        $job = new ScoreVesselCargoJob($vessel->id, $cargo->id);
        $job->handle($service);
        
        // Assert score was saved
        $this->assertDatabaseHas('vessel_cargo_scores', [
            'vessel_id' => $vessel->id,
            'cargo_id' => $cargo->id,
            'final_score' => 88,
        ]);
    }
}
```

---

## 8. CODE EXAMPLES

### 8.1 Example: Manual Correction Flow

```javascript
// Frontend (React)
const handleCorrection = async (criterion, newScore, reasoning) => {
  const response = await fetch('/api/scoring/correct', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      score_id: scoreId,
      criterion: criterion,
      corrected_score: newScore,
      reasoning: reasoning,
    }),
  });
  
  if (response.ok) {
    toast.success('Correction saved and sent to AI for learning');
    refreshScores();
  }
};
```

```php
// Laravel Backend
class ScoringService {
    public function sendCorrectionToAI(ScoreCorrection $correction)
    {
        $score = $correction->vesselCargoScore;
        
        Http::post("{$this->pythonApiUrl}/api/v1/learn", [
            'correction_id' => $correction->id,
            'vessel_id' => $score->vessel_id,
            'cargo_id' => $score->cargo_id,
            'criterion' => $correction->criterion,
            'original_score' => $correction->original_score,
            'corrected_score' => $correction->corrected_score,
            'user_reasoning' => $correction->user_reasoning,
            'vessel_data' => [...],
            'cargo_data' => [...],
        ]);
        
        $correction->update([
            'sent_to_ai' => true,
            'sent_to_ai_at' => now(),
        ]);
    }
}
```

```python
# Python Microservice
@app.post("/api/v1/learn")
async def learn_from_correction(correction: Dict[str, Any]):
    """Store correction in LangSmith for behavioral cloning"""
    
    from langsmith import Client
    
    client = Client()
    
    # Create dataset example
    client.create_example(
        dataset_name="scoring_corrections",
        inputs={
            "vessel_data": correction["vessel_data"],
            "cargo_data": correction["cargo_data"],
            "criterion": correction["criterion"],
        },
        outputs={
            "score": correction["corrected_score"],
            "reasoning": correction["user_reasoning"],
        },
        metadata={
            "correction_id": correction["correction_id"],
            "original_score": correction["original_score"],
        }
    )
    
    logger.info(f"Correction stored in LangSmith: {correction['correction_id']}")
    
    return {"status": "learned"}
```

### 8.2 Example: RAG Query for P1A

```python
# In P1ARegionalChain

async def score(self, vessel: Dict, cargo: Dict) -> Dict:
    """Score P1A with RAG lookup"""
    
    # Build query for RAG
    query = self._build_query(vessel, cargo)
    
    # Query: "Coaster vessel size C from Black Sea to Marmara Sea with grain cargo"
    
    # Retrieve relevant trade patterns from MongoDB
    patterns = await self.vector_store.similarity_search(
        collection="regional_trade_patterns",
        query=query,
        k=5,
        filters={
            "metadata.vessel_size": vessel_size,
            "metadata.origin_region": vessel_region,
        }
    )
    
    # Build context from retrieved patterns
    context = "\n\n".join([p["text"] for p in patterns])
    
    # Use LLM to score based on context
    prompt = ChatPromptTemplate.from_template("""
    You are a maritime freight expert evaluating vessel-cargo compatibility.
    
    Based on the following trade patterns from our knowledge base:
    
    {context}
    
    Score how well this voyage fits typical regional patterns:
    - Vessel: {vessel_size} ({vessel_dwt} DWT)
    - Origin: {origin_region}
    - Destination: {destination_region}
    - Cargo: {cargo_type}
    
    Provide a score from 0-15 where:
    - 15: Very common and logical trade pattern
    - 10: Acceptable pattern
    - 5: Unusual but possible
    - 0: Very rare or illogical
    
    Return JSON with score and reasoning.
    """)
    
    response = await self.llm.ainvoke(
        prompt.format(
            context=context,
            vessel_size=vessel_size,
            vessel_dwt=vessel["dwt"],
            origin_region=vessel_region,
            destination_region=cargo_region,
            cargo_type=cargo["cargo_type"],
        )
    )
    
    result = json.loads(response.content)
    
    return {
        "score": result["score"],
        "reasoning": result["reasoning"],
    }
```

---

## 9. TROUBLESHOOTING

### Common Issues

**Issue: MongoDB connection timeout**
```python
# Solution: Check MongoDB Atlas IP whitelist
# Add your server IP to Network Access in Atlas dashboard
# Or use 0.0.0.0/0 for development (not recommended for production)
```

**Issue: RabbitMQ queue not processing**
```bash
# Check queue status
rabbitmqctl list_queues

# Restart queue worker
php artisan queue:restart

# Check worker logs
tail -f storage/logs/laravel.log
```

**Issue: LangSmith traces not appearing**
```python
# Verify environment variables
print(os.getenv("LANGSMITH_API_KEY"))
print(os.getenv("LANGSMITH_PROJECT"))

# Check LangSmith client initialization
from langsmith import Client
client = Client()
print(client.info())
```

**Issue: Embeddings generation slow**
```python
# Use batch processing
texts = [chunk["text"] for chunk in chunks]
embeddings = await self.embeddings.aembed_documents(texts)  # Batch

# Instead of:
# for chunk in chunks:
#     embedding = await self.embeddings.aembed_query(chunk["text"])  # One by one
```

---

## 10. NEXT STEPS FOR OLEKSIY

### Week 1 Tasks:
1. ✅ Read this guide completely
2. ✅ Review key documentation files:
   - VKCHART_AI_ARCHITECTURE.md
   - MASTER_SCORING_SYSTEM.txt
   - LANG_STACK_INTEGRATION_MODEL.md
3. ✅ Set up development environment:
   - Install Python dependencies
   - Set up MongoDB Atlas account
   - Configure RabbitMQ
   - Set up LangSmith account
4. ✅ Create basic Laravel API endpoint
5. ✅ Create basic Python FastAPI /score stub
6. ✅ Test Laravel → Python communication

### Questions?
Contact Vitaly with any questions about:
- Architecture decisions
- Business logic clarifications
- Scoring rules interpretation
- Priority of features

---

**Document Version:** 1.0  
**Last Updated:** 2024-12-02  
**Status:** Ready for Implementation

**Good luck, Oleksiy! 🚀**
