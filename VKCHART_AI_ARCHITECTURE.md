# 🏗️ VK CHART AI — ПОЛНАЯ АРХИТЕКТУРА СИСТЕМЫ

**Версия**: 2.0  
**Дата**: 2024-12-02  
**Автор**: Claude + Vitaly  
**Статус**: In Progress
**Обновление v2.0**: Интеграция Lang* Stack (LangGraph, LangSmith, LangServe)

---

## 📋 ОГЛАВЛЕНИЕ

1. [Executive Summary](#1-executive-summary)
2. [High-Level Architecture](#2-high-level-architecture)
3. [Компоненты системы](#3-компоненты-системы)
4. [Data Flow](#4-data-flow)
5. [API Contracts](#5-api-contracts)
6. [Database Schema](#6-database-schema)
7. [AI Agents Design](#7-ai-agents-design)
8. [Knowledge Base Structure](#8-knowledge-base-structure)
9. [Normalization System](#9-normalization-system)
10. [Learning Mechanism](#10-learning-mechanism)
11. [Technology Stack](#11-technology-stack)
12. [Implementation Phases](#12-implementation-phases)
13. [Risks & Mitigations](#13-risks--mitigations)
14. [LangGraph & LangSmith Integration](#14-langgraph--langsmith-integration)

---

## 1. EXECUTIVE SUMMARY

### 🎯 Цель системы
Автоматизация vessel-cargo matching и генерации персонализированных офферов для морских фрахтовых брокеров.

### 🔑 Ключевые решения

| Аспект | Решение | Обоснование |
|--------|---------|-------------|
| **Архитектура** | Laravel + Python microservice | Использует существующую инфраструктуру |
| **Очереди** | RabbitMQ | Уже используется в проекте |
| **AI Agents** | 2 агента (Scoring + Message) | Разделение ответственности |
| **LLM** | Claude API (primary) | Лучший для reasoning и текстов |
| **Vector Search** | MongoDB Atlas | Уже настроен |
| **Knowledge** | Hybrid (Rules + RAG) | Баланс скорости и гибкости |
| **★ State Management** | **LangGraph** | **Workflow orchestration, conditional routing** |
| **★ Observability** | **LangSmith** | **Tracing, evaluation, monitoring** |
| **★ API Deployment** | **LangServe** | **FastAPI streaming endpoints** |

### 📊 Метрики успеха
- Точность скоринга: >85% совпадение с экспертной оценкой
- Время скоринга: <5 сек на одно судно
- Качество текстов: >90% одобрение без правок

---

## 2. HIGH-LEVEL ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              VK CHARTS SYSTEM                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                        LARAVEL APPLICATION                            │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │   Vessels   │  │   Cargos    │  │  Companies  │  │    UI/API   │  │   │
│  │  │   Module    │  │   Module    │  │   Module    │  │   Gateway   │  │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  │   │
│  │         │                │                │                │         │   │
│  │         └────────────────┴────────────────┴────────────────┘         │   │
│  │                                   │                                   │   │
│  │                    ┌──────────────▼──────────────┐                   │   │
│  │                    │     EVENT DISPATCHER        │                   │   │
│  │                    │  (Cargo/Vessel/Comment)     │                   │   │
│  │                    └──────────────┬──────────────┘                   │   │
│  └───────────────────────────────────┼──────────────────────────────────┘   │
│                                      │                                       │
│                                      ▼                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                          RABBITMQ                                      │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │  │
│  │  │  scoring    │  │  messages   │  │  learning   │  │  normalize  │   │  │
│  │  │   queue     │  │   queue     │  │   queue     │  │   queue     │   │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘   │  │
│  └─────────┼────────────────┼────────────────┼────────────────┼──────────┘  │
│            │                │                │                │             │
│            ▼                ▼                ▼                ▼             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                     PYTHON AI SERVICE (FastAPI)                       │   │
│  │                                                                       │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │   │
│  │  │  SCORING AGENT  │  │  MESSAGE AGENT  │  │ NORMALIZER      │       │   │
│  │  │  ┌───────────┐  │  │  ┌───────────┐  │  │ ┌───────────┐   │       │   │
│  │  │  │ P1-P7     │  │  │  │ Email     │  │  │ │ Rules     │   │       │   │
│  │  │  │ Criteria  │  │  │  │ Templates │  │  │ │ Engine    │   │       │   │
│  │  │  └───────────┘  │  │  └───────────┘  │  │ └───────────┘   │       │   │
│  │  │  ┌───────────┐  │  │  ┌───────────┐  │  │ ┌───────────┐   │       │   │
│  │  │  │ Tools     │  │  │  │ Personal- │  │  │ │ LLM       │   │       │   │
│  │  │  │ (intake,  │  │  │  │ ization   │  │  │ │ Fallback  │   │       │   │
│  │  │  │  distance)│  │  │  │ Engine    │  │  │ └───────────┘   │       │   │
│  │  │  └───────────┘  │  │  └───────────┘  │  │ ┌───────────┐   │       │   │
│  │  │  ┌───────────┐  │  │  ┌───────────┐  │  │ │ Auto-Add  │   │       │   │
│  │  │  │ RAG       │  │  │  │ LangChain │  │  │ │ Aliases   │   │       │   │
│  │  │  │ Retriever │  │  │  │ Chain     │  │  │ └───────────┘   │       │   │
│  │  │  └───────────┘  │  │  └───────────┘  │  └─────────────────┘       │   │
│  │  └─────────────────┘  └─────────────────┘                            │   │
│  │                                                                       │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │   │
│  │  │                    LEARNING MODULE                               │ │   │
│  │  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │ │   │
│  │  │  │ Correction│  │ Pattern   │  │ Feedback  │  │ Weight    │    │ │   │
│  │  │  │ Collector │  │ Extractor │  │ Analyzer  │  │ Adjuster  │    │ │   │
│  │  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘    │ │   │
│  │  └─────────────────────────────────────────────────────────────────┘ │   │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│                                      ▼                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        MONGODB ATLAS                                   │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │  │
│  │  │  scoring_   │  │  knowledge_ │  │  learning_  │  │  aliases_   │   │  │
│  │  │  results    │  │  embeddings │  │  patterns   │  │  auto       │   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                    │  │
│  │  │  message_   │  │  distances_ │  │  normalize_ │                    │  │
│  │  │  history    │  │  cache      │  │  logs       │                    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         EXTERNAL SERVICES                              │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                    │  │
│  │  │  Claude API │  │  OpenAI API │  │  Gmail API  │                    │  │
│  │  │  (Primary)  │  │  (Backup)   │  │  (Send)     │                    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. КОМПОНЕНТЫ СИСТЕМЫ

### 3.1 Laravel Application (Existing)

**Роль**: Оркестратор, UI, Data Gateway

**Новые компоненты для добавления**:

```php
// app/Events/
CargoUpdatedEvent.php          // Триггер при изменении груза
VesselPositionChangedEvent.php // Триггер при изменении позиции
CommentsUpdatedEvent.php       // Триггер при изменении комментов
ScoringRequestedEvent.php      // Запрос на скоринг

// app/Jobs/
RequestScoringJob.php          // Отправка в RabbitMQ
ProcessScoringResultJob.php    // Получение результатов
RequestMessageGenerationJob.php // Генерация текстов

// app/Services/
AIServiceClient.php            // HTTP клиент к Python
ScoringResultProcessor.php     // Обработка результатов
TriggerManager.php             // Управление триггерами

// app/Models/
ScoringResult.php              // Результаты скоринга
LearningFeedback.php           // Обратная связь
NormalizationLog.php           // Логи нормализации
```

### 3.2 Python AI Service

**Роль**: AI-логика, скоринг, генерация текстов

**Структура проекта**:

```
vkchart-ai-service/
├── app/
│   ├── __init__.py
│   ├── main.py                    # FastAPI entry point
│   ├── config.py                  # Configuration
│   │
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── scoring_agent.py       # Scoring Agent
│   │   ├── message_agent.py       # Message Agent
│   │   └── base_agent.py          # Base class
│   │
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── intake_calculator.py   # P5 Intake calculation
│   │   ├── distance_calculator.py # Distance estimation
│   │   ├── port_restrictions.py   # Port data lookup
│   │   ├── comments_parser.py     # Comments extraction
│   │   └── eta_calculator.py      # ETA/Readiness calc
│   │
│   ├── normalizers/
│   │   ├── __init__.py
│   │   ├── port_normalizer.py     # Port normalization
│   │   ├── region_normalizer.py   # Region normalization
│   │   ├── cargo_normalizer.py    # Cargo type normalization
│   │   ├── rules_engine.py        # Fast rules-based
│   │   └── llm_fallback.py        # LLM fallback
│   │
│   ├── knowledge/
│   │   ├── __init__.py
│   │   ├── rag_retriever.py       # RAG implementation
│   │   ├── embeddings.py          # Embedding generation
│   │   └── knowledge_loader.py    # Load YAML/TXT files
│   │
│   ├── learning/
│   │   ├── __init__.py
│   │   ├── feedback_collector.py  # Collect corrections
│   │   ├── pattern_extractor.py   # Extract patterns
│   │   └── weight_adjuster.py     # Adjust scoring weights
│   │
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── scoring.py             # Scoring request/response
│   │   ├── message.py             # Message request/response
│   │   └── common.py              # Shared schemas
│   │
│   ├── workers/
│   │   ├── __init__.py
│   │   ├── scoring_worker.py      # RabbitMQ consumer
│   │   ├── message_worker.py      # RabbitMQ consumer
│   │   └── learning_worker.py     # RabbitMQ consumer
│   │
│   └── utils/
│       ├── __init__.py
│       ├── llm_client.py          # LLM abstraction layer
│       ├── mongo_client.py        # MongoDB connection
│       └── logger.py              # Logging
│
├── knowledge_base/
│   ├── proximity_scoring_matrix.yaml
│   ├── regional_trade_patterns.yaml
│   ├── intake_calculator_formula.yaml
│   ├── vessel_size_classification.yaml
│   ├── port_restrictions_map.yaml
│   ├── comments_processing_guide.yaml
│   ├── open_area_comments_scoring.yaml
│   ├── readiness_eta_scoring.yaml
│   └── email_templates/
│       ├── case_a_strong_match.txt
│       ├── case_b_not_ideal.txt
│       ├── reminder_1.txt
│       └── whatsapp_templates.txt
│
├── aliases/
│   ├── ports_aliases.yaml         # Port name aliases
│   ├── regions_aliases.yaml       # Region aliases
│   ├── cargo_aliases.yaml         # Cargo type aliases
│   └── auto_added/                # Auto-added by system
│       └── pending_review.yaml
│
├── tests/
│   ├── test_scoring_agent.py
│   ├── test_message_agent.py
│   ├── test_normalizers.py
│   └── test_tools.py
│
├── requirements.txt
├── Dockerfile
└── docker-compose.yaml
```

### 3.3 RabbitMQ Queues

| Queue Name | Producer | Consumer | Purpose |
|------------|----------|----------|---------|
| `vkchart.scoring.requests` | Laravel | Python | Scoring requests |
| `vkchart.scoring.results` | Python | Laravel | Scoring results |
| `vkchart.messages.requests` | Laravel | Python | Message generation |
| `vkchart.messages.results` | Python | Laravel | Generated messages |
| `vkchart.learning.feedback` | Laravel | Python | User corrections |
| `vkchart.normalize.requests` | Laravel | Python | Normalization requests |
| `vkchart.normalize.logs` | Python | Laravel | Auto-added aliases |

### 3.4 MongoDB Collections

```javascript
// Database: vkchart_ai

// 1. Scoring Results (transactional)
db.scoring_results {
  _id: ObjectId,
  vessel_id: Number,
  cargo_id: Number,
  
  scores: {
    p1_proximity: { score: Number, max: 20, reasoning: String },
    p1a_patterns: { score: Number, max: 15, reasoning: String },
    p2_regional: { score: Number, max: 15, reasoning: String },
    p3_cargo: { score: Number, max: 15, reasoning: String },
    p4_last_ports: { score: Number, max: 10, reasoning: String },
    p5_intake: { score: Number, max: 15, reasoning: String },
    p6_open_area: { score: Number, range: [-50, 25], reasoning: String },
    p7_readiness: { score: Number, max: 10, reasoning: String }
  },
  
  final_score: Number,
  classification: String,  // "excellent" | "good" | "acceptable" | "marginal" | "poor"
  is_blocked: Boolean,
  block_reason: String,
  
  reasoning_summary: String,
  strengths: [String],
  weaknesses: [String],
  recommendation: String,
  
  snapshot: {
    vessel: Object,  // Frozen vessel data
    cargo: Object,   // Frozen cargo data
    company: Object  // Frozen company data
  },
  
  metadata: {
    model_used: String,
    processing_time_ms: Number,
    knowledge_files_used: [String],
    similar_patterns_found: Number
  },
  
  created_at: Date,
  updated_at: Date,
  version: Number
}

// 2. Knowledge Embeddings (for RAG)
db.knowledge_embeddings {
  _id: ObjectId,
  source_file: String,
  chunk_id: String,
  chunk_text: String,
  embedding: [Number],  // Vector 1536 dimensions
  metadata: {
    criterion: String,  // "P1" | "P2" | etc.
    category: String,   // "rule" | "example" | "pattern"
    keywords: [String]
  },
  created_at: Date
}

// Index for vector search
db.knowledge_embeddings.createIndex(
  { embedding: "vectorSearch" },
  { 
    name: "vector_index",
    vectorSearchOptions: {
      dimensions: 1536,
      similarity: "cosine"
    }
  }
)

// 3. Learning Patterns
db.learning_patterns {
  _id: ObjectId,
  pattern_type: String,  // "score_correction" | "text_edit" | "alias_add"
  
  context: {
    vessel_type: String,
    cargo_type: String,
    region_from: String,
    region_to: String,
    season: String
  },
  
  original: {
    score: Number,
    reasoning: String
  },
  
  corrected: {
    score: Number,
    reasoning: String,
    user_explanation: String
  },
  
  confidence: Number,  // How many times confirmed
  applied_count: Number,
  
  created_by: String,
  created_at: Date,
  last_applied: Date
}

// 4. Auto-Added Aliases (pending review)
db.aliases_auto {
  _id: ObjectId,
  entity_type: String,  // "port" | "region" | "cargo"
  
  original_value: String,
  normalized_value: String,
  
  source: String,  // "llm_fallback" | "fuzzy_match"
  confidence: Number,
  
  context: {
    vessel_id: Number,
    cargo_id: Number,
    field_name: String
  },
  
  status: String,  // "pending" | "approved" | "rejected"
  reviewed_by: String,
  reviewed_at: Date,
  rejection_reason: String,
  
  created_at: Date
}

// 5. Message History
db.message_history {
  _id: ObjectId,
  vessel_id: Number,
  cargo_id: Number,
  company_id: Number,
  person_id: Number,
  
  message_type: String,  // "offer" | "reminder_1" | "reminder_2" | "whatsapp"
  channel: String,  // "email" | "whatsapp" | "teams"
  
  generated_text: String,
  edited_text: String,
  was_edited: Boolean,
  
  scoring_result_id: ObjectId,
  
  personalization: {
    tone: String,
    language: String,
    highlights_used: [String]
  },
  
  sent_at: Date,
  response_received: Boolean,
  response_category: String,
  
  created_at: Date
}

// 6. Distances Cache
db.distances_cache {
  _id: ObjectId,
  port_from: String,  // UNLOCODE
  port_to: String,    // UNLOCODE
  distance_nm: Number,
  route_type: String,  // "direct" | "via_suez" | "via_gibraltar"
  source: String,      // "calculated" | "seadistances" | "manual"
  created_at: Date,
  expires_at: Date
}

// 7. Normalization Logs
db.normalization_logs {
  _id: ObjectId,
  entity_type: String,
  original_value: String,
  normalized_value: String,
  method: String,  // "exact_match" | "fuzzy" | "llm"
  confidence: Number,
  processing_time_ms: Number,
  created_at: Date
}
```

---

## 4. DATA FLOW

### 4.1 Scoring Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SCORING DATA FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────┘

[1] TRIGGER
     │
     ▼
┌─────────────┐    Event: CargoUpdated / VesselPositionChanged
│   LARAVEL   │    
│  (Trigger)  │────────────────────────────────────────────────────┐
└─────────────┘                                                     │
     │                                                              │
     │ [2] Prepare Snapshot                                         │
     ▼                                                              │
┌─────────────────────────────────────────────────────────────┐    │
│  SNAPSHOT DATA                                               │    │
│  {                                                           │    │
│    vessel: { id, name, dwt, flag, open_area, open_date,     │    │
│              comments, position, destination, eta }          │    │
│    cargo: { id, type, quantity, load_port, disch_port,      │    │
│             laycan_start, laycan_end, rate }                 │    │
│    company: { id, name, comments, preferences }              │    │
│    person: { id, name, comments, comm_style }                │    │
│  }                                                           │    │
└─────────────────────────────────────────────────────────────┘    │
     │                                                              │
     │ [3] Publish to Queue                                         │
     ▼                                                              │
┌─────────────┐                                                     │
│  RabbitMQ   │  Queue: vkchart.scoring.requests                   │
│   Queue     │  Message: { snapshot, request_id, priority }       │
└─────────────┘                                                     │
     │                                                              │
     │ [4] Consume                                                  │
     ▼                                                              │
┌─────────────────────────────────────────────────────────────┐    │
│              PYTHON SCORING WORKER                           │    │
│                                                              │    │
│  ┌─────────────────────────────────────────────────────┐    │    │
│  │ [4.1] NORMALIZATION                                  │    │    │
│  │                                                      │    │    │
│  │  Input: open_area = "cta"                           │    │    │
│  │    ↓                                                │    │    │
│  │  Rules Engine: Check aliases_ports.yaml             │    │    │
│  │    ↓ (found? → return)                              │    │    │
│  │  Fuzzy Match: Levenshtein distance                  │    │    │
│  │    ↓ (confidence > 0.85? → return)                  │    │    │
│  │  LLM Fallback: "What port is 'cta'?"               │    │    │
│  │    ↓                                                │    │    │
│  │  Output: "ROCTA" (Constanta)                        │    │    │
│  │    ↓                                                │    │    │
│  │  Auto-add to pending aliases                        │    │    │
│  └─────────────────────────────────────────────────────┘    │    │
│                         │                                    │    │
│                         ▼                                    │    │
│  ┌─────────────────────────────────────────────────────┐    │    │
│  │ [4.2] SCORING AGENT                                  │    │    │
│  │                                                      │    │    │
│  │  for criterion in [P1, P1A, P2, P3, P4, P5, P6, P7]:│    │    │
│  │    │                                                │    │    │
│  │    ├─► RAG: Retrieve relevant knowledge             │    │    │
│  │    │   (proximity_matrix, trade_patterns, etc.)     │    │    │
│  │    │                                                │    │    │
│  │    ├─► Tools: Call if needed                        │    │    │
│  │    │   - calculate_intake(vessel, port)             │    │    │
│  │    │   - get_distance(port_a, port_b)               │    │    │
│  │    │   - parse_comments(text)                       │    │    │
│  │    │                                                │    │    │
│  │    ├─► Learning: Check similar patterns             │    │    │
│  │    │   db.learning_patterns.find(context)           │    │    │
│  │    │                                                │    │    │
│  │    └─► LLM: Generate score + reasoning              │    │    │
│  │        Claude: "Based on ... score is X because..." │    │    │
│  │                                                      │    │    │
│  │  Calculate final_score = sum(P1..P5,P7) + P6        │    │    │
│  │  Apply blocking conditions                          │    │    │
│  │  Generate reasoning_summary                         │    │    │
│  └─────────────────────────────────────────────────────┘    │    │
│                         │                                    │    │
│                         ▼                                    │    │
│  [4.3] Save to MongoDB: db.scoring_results.insert()         │    │
│                         │                                    │    │
└─────────────────────────┼────────────────────────────────────┘    │
                          │                                         │
                          │ [5] Publish Result                      │
                          ▼                                         │
                   ┌─────────────┐                                  │
                   │  RabbitMQ   │  Queue: vkchart.scoring.results │
                   │   Queue     │  Message: { result, request_id } │
                   └─────────────┘                                  │
                          │                                         │
                          │ [6] Consume & Update                    │
                          ▼                                         │
                   ┌─────────────┐                                  │
                   │   LARAVEL   │  Update vessel_cargo_statuses   │
                   │  (Result)   │  Broadcast to UI (Pusher)       │
                   └─────────────┘◄─────────────────────────────────┘
```

### 4.2 Message Generation Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MESSAGE GENERATION DATA FLOW                           │
└─────────────────────────────────────────────────────────────────────────────┘

[1] User clicks "Generate Offer" or System triggers auto-generation
     │
     ▼
┌─────────────┐
│   LARAVEL   │  Fetch: scoring_result + company + person data
└─────────────┘
     │
     │ [2] Build Message Request
     ▼
┌─────────────────────────────────────────────────────────────┐
│  MESSAGE REQUEST                                             │
│  {                                                           │
│    scoring_result_id: ObjectId,                             │
│    message_type: "offer" | "reminder_1" | "whatsapp",       │
│    channel: "email" | "whatsapp",                           │
│    person: {                                                 │
│      name, nationality, language, comm_style,               │
│      timezone, previous_messages: [...]                     │
│    },                                                        │
│    company: { name, relationship_history },                 │
│    cargo_summary: String,                                    │
│    vessel_highlights: [...],                                 │
│    today_contact: Boolean  // Писали ли сегодня?            │
│  }                                                           │
└─────────────────────────────────────────────────────────────┘
     │
     │ [3] Publish to Queue
     ▼
┌─────────────┐
│  RabbitMQ   │  Queue: vkchart.messages.requests
└─────────────┘
     │
     │ [4] Consume
     ▼
┌─────────────────────────────────────────────────────────────┐
│              PYTHON MESSAGE WORKER                           │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ [4.1] LOAD CONTEXT                                   │    │
│  │                                                      │    │
│  │  - Fetch scoring_result from MongoDB                 │    │
│  │  - Load appropriate template                         │    │
│  │  - Load previous messages with this person           │    │
│  │  - Load person's communication preferences           │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ [4.2] MESSAGE AGENT                                  │    │
│  │                                                      │    │
│  │  System Prompt:                                      │    │
│  │  "You are Vitaly, a maritime freight broker.        │    │
│  │   Write personalized, professional messages.        │    │
│  │   Match the tone to the recipient's style.          │    │
│  │   Use scoring insights to highlight key points."    │    │
│  │                                                      │    │
│  │  Context Injection:                                  │    │
│  │  - Scoring reasoning (strengths, weaknesses)        │    │
│  │  - Company relationship history                     │    │
│  │  - Person's preferred communication style           │    │
│  │  - Previous message thread (if reminder)            │    │
│  │  - Today's contact flag (skip greeting if true)     │    │
│  │                                                      │    │
│  │  Template Selection:                                 │    │
│  │  - Score ≥ 85: case_a_strong_match.txt              │    │
│  │  - Score 70-84: case_a_standard.txt                 │    │
│  │  - Score 60-69: case_b_not_ideal.txt                │    │
│  │  - Reminder: reminder_{n}.txt                       │    │
│  │                                                      │    │
│  │  LLM Generation:                                     │    │
│  │  Claude → Personalized message text                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  [4.3] Save to MongoDB: db.message_history.insert()         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
     │
     │ [5] Return to Laravel
     ▼
┌─────────────┐
│   LARAVEL   │  Show in UI for approval
│     UI      │  User can edit → Save edited version
└─────────────┘
     │
     │ [6] On Send
     ▼
┌─────────────┐
│   LARAVEL   │  Send via Gmail API / WhatsApp API
│   Sender    │  Update message_history (sent_at)
└─────────────┘
```

---

## 5. API CONTRACTS

### 5.1 Internal API (Laravel ↔ Python)

```yaml
# FastAPI Endpoints

# ═══════════════════════════════════════════════════════════════
# SCORING ENDPOINTS
# ═══════════════════════════════════════════════════════════════

POST /api/v1/scoring/calculate:
  description: "Calculate scoring for vessel-cargo pair"
  request:
    content-type: application/json
    body:
      vessel:
        id: integer
        name: string
        imo: string
        dwt: integer
        flag: string
        built_year: integer
        open_area: string
        open_port: string
        open_date: date
        destination: string
        eta: datetime
        current_position:
          lat: float
          lon: float
        comments: string
        last_ports: 
          - port: string
            percentage: float
      cargo:
        id: integer
        type: string
        quantity: integer
        stowage_factor: float
        load_port: string
        load_region: string
        disch_port: string
        disch_region: string
        laycan_start: date
        laycan_end: date
        loading_rate: integer
      company:
        id: integer
        name: string
        comments: string
        regional_preferences:
          - region: string
            preference: string  # specializes|prefers|can_work|neutral|avoids|cannot
        cargo_preferences:
          - cargo_type: string
            preference: string
      person:
        id: integer
        name: string
        comments: string
      request_id: string
      priority: string  # high|normal|low
      
  response:
    200:
      scoring_result_id: string  # MongoDB ObjectId
      request_id: string
      final_score: integer
      classification: string
      is_blocked: boolean
      block_reason: string
      scores:
        p1_proximity:
          score: integer
          max_score: 20
          reasoning: string
        p1a_patterns:
          score: integer
          max_score: 15
          reasoning: string
        # ... all P1-P7
      reasoning_summary: string
      strengths: [string]
      weaknesses: [string]
      recommendation: string
      metadata:
        processing_time_ms: integer
        model_used: string
        
GET /api/v1/scoring/{scoring_result_id}:
  description: "Get scoring result by ID"
  response:
    200: ScoringResult

# ═══════════════════════════════════════════════════════════════
# MESSAGE ENDPOINTS
# ═══════════════════════════════════════════════════════════════

POST /api/v1/messages/generate:
  description: "Generate message based on scoring"
  request:
    scoring_result_id: string
    message_type: string  # offer|reminder_1|reminder_2|reminder_3|whatsapp
    channel: string  # email|whatsapp|teams
    person:
      id: integer
      name: string
      nationality: string
      language: string
      comm_style: string  # formal|informal|brief
      timezone: string
    previous_messages: 
      - text: string
        sent_at: datetime
        response: string
    today_contact: boolean
    
  response:
    200:
      message_id: string
      generated_text: string
      subject: string  # for emails
      personalization_notes: string
      
# ═══════════════════════════════════════════════════════════════
# NORMALIZATION ENDPOINTS  
# ═══════════════════════════════════════════════════════════════

POST /api/v1/normalize:
  description: "Normalize entity value"
  request:
    entity_type: string  # port|region|cargo
    value: string
    context:
      vessel_id: integer
      cargo_id: integer
      
  response:
    200:
      original: string
      normalized: string
      confidence: float
      method: string  # exact|fuzzy|llm
      auto_added: boolean

GET /api/v1/normalize/pending:
  description: "Get pending alias approvals"
  response:
    200:
      items:
        - id: string
          entity_type: string
          original: string
          normalized: string
          confidence: float
          created_at: datetime

POST /api/v1/normalize/review/{id}:
  description: "Approve or reject auto-added alias"
  request:
    action: string  # approve|reject
    rejection_reason: string  # if reject
    correct_value: string  # if reject with correction
    
# ═══════════════════════════════════════════════════════════════
# LEARNING ENDPOINTS
# ═══════════════════════════════════════════════════════════════

POST /api/v1/learning/feedback:
  description: "Submit user correction"
  request:
    scoring_result_id: string
    feedback_type: string  # score_correction|text_edit
    original_value: any
    corrected_value: any
    user_explanation: string
    user_id: integer
    
  response:
    200:
      pattern_id: string
      similar_patterns_found: integer
      
GET /api/v1/learning/stats:
  description: "Get learning statistics"
  response:
    200:
      total_corrections: integer
      accuracy_improvement: float
      patterns_by_criterion:
        p1: integer
        p2: integer
        # ...

# ═══════════════════════════════════════════════════════════════
# HEALTH & MONITORING
# ═══════════════════════════════════════════════════════════════

GET /health:
  response:
    200:
      status: string
      mongodb: string
      rabbitmq: string
      llm_api: string
      
GET /metrics:
  response:
    200:
      scoring_requests_total: integer
      scoring_avg_time_ms: float
      message_requests_total: integer
      llm_calls_today: integer
      llm_cost_today_usd: float
```

### 5.2 RabbitMQ Message Schemas

```json
// Queue: vkchart.scoring.requests
{
  "request_id": "uuid",
  "priority": "high|normal|low",
  "timestamp": "2024-11-30T10:00:00Z",
  "payload": {
    "vessel": { ... },
    "cargo": { ... },
    "company": { ... },
    "person": { ... }
  }
}

// Queue: vkchart.scoring.results
{
  "request_id": "uuid",
  "success": true,
  "timestamp": "2024-11-30T10:00:05Z",
  "result": {
    "scoring_result_id": "mongo_object_id",
    "final_score": 78,
    "classification": "good",
    "is_blocked": false
  },
  "error": null
}

// Queue: vkchart.normalize.logs
{
  "timestamp": "2024-11-30T10:00:00Z",
  "action": "auto_add",
  "entity_type": "port",
  "original": "cta",
  "normalized": "ROCTA",
  "confidence": 0.92,
  "requires_review": true
}

// Queue: vkchart.learning.feedback
{
  "timestamp": "2024-11-30T10:00:00Z",
  "scoring_result_id": "mongo_object_id",
  "feedback_type": "score_correction",
  "criterion": "P1",
  "original_score": 15,
  "corrected_score": 8,
  "user_explanation": "Vessel is too far, Libya to Black Sea is not profitable for Handysize",
  "user_id": 1
}
```

---

## 6. DATABASE SCHEMA

### 6.1 MySQL Tables (Laravel)

```sql
-- Existing tables used:
-- vessels, cargos, companies, people, ports, vessel_cargo_statuses

-- NEW TABLES:

-- Scoring results reference (main data in MongoDB)
CREATE TABLE scoring_results (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    mongo_id VARCHAR(24) NOT NULL UNIQUE,
    vessel_id BIGINT UNSIGNED NOT NULL,
    cargo_id BIGINT UNSIGNED NOT NULL,
    final_score TINYINT UNSIGNED NOT NULL,
    classification ENUM('excellent','good','acceptable','marginal','poor','blocked') NOT NULL,
    is_blocked BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_vessel_cargo (vessel_id, cargo_id),
    INDEX idx_cargo_score (cargo_id, final_score DESC),
    INDEX idx_created (created_at),
    
    FOREIGN KEY (vessel_id) REFERENCES vessels(id),
    FOREIGN KEY (cargo_id) REFERENCES cargos(id)
);

-- Learning feedback tracking
CREATE TABLE learning_feedbacks (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    scoring_result_id BIGINT UNSIGNED NOT NULL,
    feedback_type ENUM('score_correction','text_edit','alias_approval') NOT NULL,
    criterion VARCHAR(10),  -- P1, P2, etc.
    original_value TEXT,
    corrected_value TEXT,
    user_explanation TEXT,
    user_id BIGINT UNSIGNED NOT NULL,
    processed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_scoring (scoring_result_id),
    INDEX idx_user (user_id),
    
    FOREIGN KEY (scoring_result_id) REFERENCES scoring_results(id),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Normalization audit log
CREATE TABLE normalization_logs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    entity_type ENUM('port','region','cargo','vessel_size') NOT NULL,
    original_value VARCHAR(255) NOT NULL,
    normalized_value VARCHAR(255) NOT NULL,
    method ENUM('exact','fuzzy','llm') NOT NULL,
    confidence DECIMAL(3,2),
    auto_added BOOLEAN DEFAULT FALSE,
    status ENUM('pending','approved','rejected') DEFAULT 'approved',
    reviewed_by BIGINT UNSIGNED,
    reviewed_at TIMESTAMP,
    rejection_reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_entity (entity_type, original_value),
    INDEX idx_status (status),
    
    FOREIGN KEY (reviewed_by) REFERENCES users(id)
);

-- AI job queue tracking
CREATE TABLE ai_jobs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    job_type ENUM('scoring','message','normalize','learning') NOT NULL,
    request_id VARCHAR(36) NOT NULL UNIQUE,
    vessel_id BIGINT UNSIGNED,
    cargo_id BIGINT UNSIGNED,
    status ENUM('pending','processing','completed','failed') DEFAULT 'pending',
    priority ENUM('high','normal','low') DEFAULT 'normal',
    attempts TINYINT UNSIGNED DEFAULT 0,
    error_message TEXT,
    processing_time_ms INT UNSIGNED,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    
    INDEX idx_status (status, priority),
    INDEX idx_request (request_id),
    INDEX idx_vessel_cargo (vessel_id, cargo_id)
);
```

### 6.2 MongoDB Collections (detailed in Section 3.4)

See Section 3.4 for full MongoDB schema.

---

## 7. AI AGENTS DESIGN

### 7.1 Scoring Agent

```python
# app/agents/scoring_agent.py

from langchain.agents import AgentExecutor, create_structured_chat_agent
from langchain.prompts import ChatPromptTemplate
from langchain_anthropic import ChatAnthropic

class ScoringAgent:
    """
    Agent responsible for vessel-cargo scoring.
    Calculates P1-P7 criteria and generates reasoning.
    """
    
    SYSTEM_PROMPT = """You are an expert maritime freight broker scoring system.
Your task is to evaluate vessel-cargo compatibility across 7 criteria.

SCORING CRITERIA:
- P1 Proximity (0-20): Geographic distance from vessel to loading port
- P1A Regional Patterns (0-15): Market logic of vessel positioning
- P2 Regional Preferences (0-15): Owner's regional preferences from comments
- P3 Cargo Preferences (0-15): Cargo type compatibility
- P4 Last Ports (0-10): Vessel's familiarity with region
- P5 Intake Capacity (0-15): Ability to load required quantity
- P6 OpenArea Comments (-50 to +25): Current voyage preferences (MODIFIER)
- P7 Readiness (0-10): Timing match with laycan

RULES:
1. Use provided knowledge base for scoring rules
2. Call tools when calculation is needed (intake, distance)
3. Check learning patterns for similar cases
4. Generate clear reasoning for each criterion
5. Apply blocking conditions when necessary

BLOCKING CONDITIONS:
- Technical impossibility (intake < 90% of cargo)
- P6 ≤ -40 (owner explicitly avoids)
- Geopolitical restrictions (sanctions)

OUTPUT FORMAT:
Return JSON with scores, reasoning, and recommendation.
"""

    def __init__(self, llm_client, tools, knowledge_retriever, learning_db):
        self.llm = llm_client
        self.tools = tools
        self.knowledge = knowledge_retriever
        self.learning = learning_db
        
        self.agent = self._build_agent()
    
    def _build_agent(self):
        prompt = ChatPromptTemplate.from_messages([
            ("system", self.SYSTEM_PROMPT),
            ("human", "{input}"),
            ("placeholder", "{agent_scratchpad}")
        ])
        
        return create_structured_chat_agent(
            llm=self.llm,
            tools=self.tools,
            prompt=prompt
        )
    
    async def score(self, snapshot: dict) -> dict:
        """
        Main scoring method.
        
        Args:
            snapshot: Dict with vessel, cargo, company, person data
            
        Returns:
            Dict with scores, reasoning, recommendation
        """
        # 1. Retrieve relevant knowledge
        knowledge_context = await self._get_knowledge_context(snapshot)
        
        # 2. Check for similar learned patterns
        similar_patterns = await self._find_similar_patterns(snapshot)
        
        # 3. Build input for agent
        agent_input = self._build_input(
            snapshot, 
            knowledge_context, 
            similar_patterns
        )
        
        # 4. Execute agent
        executor = AgentExecutor(
            agent=self.agent,
            tools=self.tools,
            verbose=True,
            max_iterations=10
        )
        
        result = await executor.ainvoke({"input": agent_input})
        
        # 5. Parse and validate result
        scoring_result = self._parse_result(result)
        
        return scoring_result
    
    async def _get_knowledge_context(self, snapshot: dict) -> dict:
        """Retrieve relevant knowledge chunks via RAG."""
        queries = {
            "P1": f"proximity scoring {snapshot['cargo']['load_region']}",
            "P1A": f"trade patterns {snapshot['vessel']['open_area']} to {snapshot['cargo']['load_region']}",
            "P2": f"regional preferences {snapshot['company']['name']}",
            "P3": f"cargo preferences {snapshot['cargo']['type']}",
            "P5": f"intake calculation {snapshot['cargo']['load_port']}",
            "P6": f"open area comments interpretation",
            "P7": f"readiness timing laycan"
        }
        
        context = {}
        for criterion, query in queries.items():
            docs = await self.knowledge.aretrieve(query, k=3)
            context[criterion] = [doc.page_content for doc in docs]
        
        return context
    
    async def _find_similar_patterns(self, snapshot: dict) -> list:
        """Find similar learned patterns from corrections."""
        return await self.learning.find_similar({
            "vessel_type": self._get_vessel_type(snapshot['vessel']['dwt']),
            "cargo_type": snapshot['cargo']['type'],
            "region_from": snapshot['vessel']['open_area'],
            "region_to": snapshot['cargo']['load_region']
        }, limit=5)
    
    def _build_input(self, snapshot, knowledge, patterns) -> str:
        """Build comprehensive input string for agent."""
        return f"""
SCORING REQUEST:

VESSEL: {snapshot['vessel']['name']} ({snapshot['vessel']['dwt']} DWT)
- Open: {snapshot['vessel']['open_area']} on {snapshot['vessel']['open_date']}
- Destination: {snapshot['vessel']['destination']} ETA {snapshot['vessel']['eta']}
- Comments: {snapshot['vessel']['comments']}

CARGO: {snapshot['cargo']['type']} {snapshot['cargo']['quantity']}t
- Load: {snapshot['cargo']['load_port']} ({snapshot['cargo']['load_region']})
- Disch: {snapshot['cargo']['disch_port']}
- Laycan: {snapshot['cargo']['laycan_start']} - {snapshot['cargo']['laycan_end']}

COMPANY: {snapshot['company']['name']}
- Comments: {snapshot['company']['comments']}
- Preferences: {snapshot['company'].get('preferences', 'None specified')}

KNOWLEDGE BASE CONTEXT:
{self._format_knowledge(knowledge)}

SIMILAR LEARNED PATTERNS:
{self._format_patterns(patterns)}

Calculate scores for all criteria P1-P7 with reasoning.
"""
```

### 7.2 Message Agent

```python
# app/agents/message_agent.py

class MessageAgent:
    """
    Agent responsible for generating personalized messages.
    """
    
    SYSTEM_PROMPT = """You are Vitaly Kravets, a professional maritime freight broker.
Write personalized, engaging messages to shipowners.

PERSONALITY:
- Professional but warm
- Direct and efficient
- Knowledgeable about market
- Respectful of recipient's time

RULES:
1. Match recipient's communication style (formal/informal)
2. Use scoring insights to highlight relevant points
3. If today_contact=True, skip greeting (continuing thread)
4. Keep WhatsApp messages shorter than emails
5. Include vessel name and key cargo details
6. For reminders, reference previous message naturally

TEMPLATES AVAILABLE:
- Strong match (score ≥85): Enthusiastic, highlight perfect fit
- Good match (70-84): Professional, mention strengths
- Acceptable (60-69): Balanced, acknowledge caveats
- Reminder: Gentle follow-up, add new info if available

PERSONALIZATION FACTORS:
- Nationality → Cultural nuances
- Previous messages → Continuity
- Relationship history → Familiarity level
"""

    def __init__(self, llm_client, template_loader):
        self.llm = llm_client
        self.templates = template_loader
    
    async def generate(self, request: dict) -> dict:
        """
        Generate personalized message.
        
        Args:
            request: Dict with scoring_result, person, message_type, etc.
        """
        # 1. Load appropriate template
        template = self._select_template(
            request['message_type'],
            request['scoring_result']['final_score']
        )
        
        # 2. Build personalization context
        context = self._build_context(request)
        
        # 3. Generate message
        prompt = f"""
{self.SYSTEM_PROMPT}

TEMPLATE STRUCTURE:
{template}

CONTEXT:
{context}

Generate the message now. Return JSON with:
- subject (for email)
- body
- personalization_notes (what you customized)
"""
        
        response = await self.llm.ainvoke(prompt)
        
        return self._parse_response(response)
    
    def _select_template(self, message_type: str, score: int) -> str:
        """Select appropriate template based on type and score."""
        if message_type.startswith("reminder"):
            return self.templates.get(f"reminder_{message_type[-1]}")
        
        if score >= 85:
            return self.templates.get("case_a_strong_match")
        elif score >= 70:
            return self.templates.get("case_a_standard")
        elif score >= 60:
            return self.templates.get("case_b_not_ideal")
        else:
            return self.templates.get("case_b_positioning")
    
    def _build_context(self, request: dict) -> str:
        """Build personalization context string."""
        scoring = request['scoring_result']
        person = request['person']
        
        return f"""
RECIPIENT:
- Name: {person['name']}
- Nationality: {person['nationality']}
- Style: {person['comm_style']}
- Language: {person['language']}

SCORING INSIGHTS:
- Final Score: {scoring['final_score']}/100
- Strengths: {', '.join(scoring['strengths'])}
- Weaknesses: {', '.join(scoring['weaknesses'])}
- Recommendation: {scoring['recommendation']}

VESSEL HIGHLIGHTS:
{scoring['reasoning_summary']}

PREVIOUS MESSAGES:
{self._format_previous(request.get('previous_messages', []))}

TODAY CONTACT: {request.get('today_contact', False)}
CHANNEL: {request['channel']}
"""
```

### 7.3 Tools Definition

```python
# app/tools/intake_calculator.py

from langchain.tools import tool
from typing import Dict

@tool
def calculate_intake(
    vessel_dwt: int,
    vessel_summer_draft: float,
    port_draft_limit: float,
    water_density: float = 1.025,
    cargo_stowage_factor: float = None,
    vessel_grain_cubic: float = None
) -> Dict:
    """
    Calculate maximum cargo intake for vessel at specific port.
    
    Args:
        vessel_dwt: Vessel deadweight tonnage
        vessel_summer_draft: Summer draft in meters
        port_draft_limit: Port draft limitation in meters
        water_density: Water density (1.025 for salt, 1.000 for fresh)
        cargo_stowage_factor: Cargo SF in cbft/t (optional)
        vessel_grain_cubic: Vessel grain capacity in m³ (optional)
    
    Returns:
        Dict with intake_weight, intake_volume, limiting_factor
    """
    # Constants
    UKC = 0.5  # Under keel clearance
    SQUAT = 0.3  # Squat allowance
    ROB_FACTOR = 0.04  # 4% for bunkers/stores
    
    # 1. Calculate TPC (tonnes per cm)
    lpp_estimated = vessel_summer_draft * 20  # Rough estimation
    cwp = 0.85  # Block coefficient for bulk carriers
    tpc_salt = (lpp_estimated * (vessel_dwt / 10000) * 10 * cwp * 1.025) / 100
    
    # 2. Calculate allowable draft
    t_allow = port_draft_limit - UKC - SQUAT
    
    # 3. Density correction
    if water_density < 1.025:
        fwa = (vessel_dwt / (4 * tpc_salt * 10)) * 100  # in cm
        density_factor = (1.025 - water_density) / 0.025
        t_allow_sw = t_allow + (fwa * density_factor / 100)
    else:
        t_allow_sw = t_allow
    
    # 4. Draft reduction needed
    dt = max(0, vessel_summer_draft - t_allow_sw)
    loss_tonnes = dt * 100 * tpc_salt
    
    # 5. Weight-limited intake
    intake_weight = vessel_dwt - loss_tonnes - (vessel_dwt * ROB_FACTOR)
    
    # 6. Volume-limited intake (if data available)
    intake_volume = None
    if cargo_stowage_factor and vessel_grain_cubic:
        sf_m3 = cargo_stowage_factor * 0.0283168  # cbft to m³
        intake_volume = vessel_grain_cubic / sf_m3
    
    # 7. Final intake
    if intake_volume:
        final_intake = min(intake_weight, intake_volume)
        limiting = "volume" if intake_volume < intake_weight else "weight"
    else:
        final_intake = intake_weight
        limiting = "weight"
    
    return {
        "intake_weight": round(intake_weight),
        "intake_volume": round(intake_volume) if intake_volume else None,
        "intake_final": round(final_intake),
        "limiting_factor": limiting,
        "draft_reduction_m": round(dt, 2),
        "tonnage_lost": round(loss_tonnes)
    }


@tool
def estimate_distance(
    lat1: float, lon1: float,
    lat2: float, lon2: float,
    route_type: str = "direct"
) -> Dict:
    """
    Estimate sea distance between two points.
    
    Args:
        lat1, lon1: Origin coordinates
        lat2, lon2: Destination coordinates
        route_type: "direct", "via_suez", "via_gibraltar"
    
    Returns:
        Dict with distance_nm, estimated_days, route_notes
    """
    import math
    
    # Haversine formula
    R = 3440.065  # Earth radius in nautical miles
    
    lat1, lon1, lat2, lon2 = map(math.radians, [lat1, lon1, lat2, lon2])
    dlat = lat2 - lat1
    dlon = lon2 - lon1
    
    a = math.sin(dlat/2)**2 + math.cos(lat1) * math.cos(lat2) * math.sin(dlon/2)**2
    c = 2 * math.asin(math.sqrt(a))
    
    direct_distance = R * c
    
    # Apply route factor (sea routes are longer than great circle)
    route_factors = {
        "direct": 1.15,
        "via_suez": 1.3,
        "via_gibraltar": 1.25
    }
    
    distance = direct_distance * route_factors.get(route_type, 1.15)
    
    # Estimate sailing days (12 knots average)
    days = distance / (12 * 24)
    
    return {
        "distance_nm": round(distance),
        "estimated_days": round(days, 1),
        "route_type": route_type,
        "is_estimate": True,
        "note": "Calculated from coordinates. Actual distance may vary."
    }


@tool
def parse_comments(
    comments: str,
    comment_type: str = "vessel"
) -> Dict:
    """
    Parse structured information from free-text comments.
    
    Args:
        comments: Raw comment text
        comment_type: "vessel", "company", "person", "open_area"
    
    Returns:
        Dict with extracted preferences, restrictions, technical data
    """
    import re
    
    result = {
        "specialization": [],
        "geographic_preferences": [],
        "cargo_preferences": [],
        "technical_data": {},
        "communication_notes": [],
        "raw_text": comments
    }
    
    if not comments:
        return result
    
    comments_lower = comments.lower()
    
    # Geographic preferences
    geo_patterns = [
        (r"(spec|specializes?)[\s:]+(.+?)(?:,|$)", "specializes"),
        (r"(\w+)\s+ok", "can_work"),
        (r"no\s+(\w+)", "avoids"),
        (r"cannot\s+(\w+)", "cannot"),
        (r"prefers?\s+(\w+)", "prefers")
    ]
    
    for pattern, pref_type in geo_patterns:
        matches = re.findall(pattern, comments_lower)
        for match in matches:
            value = match[1] if isinstance(match, tuple) else match
            result["geographic_preferences"].append({
                "region": value.strip(),
                "preference": pref_type
            })
    
    # Intake data
    intake_pattern = r"(?:real\s+)?intake[:\s]+(\w+)[,\s]+(?:draft\s+)?(\d+(?:\.\d+)?)\s*m[,\s]+(\w+)\s*=\s*(\d+)\s*t"
    intake_match = re.search(intake_pattern, comments_lower)
    if intake_match:
        result["technical_data"]["manual_intake"] = {
            "cargo": intake_match.group(1),
            "draft": float(intake_match.group(2)),
            "port": intake_match.group(3),
            "tonnage": int(intake_match.group(4))
        }
    
    # Cargo preferences
    cargo_patterns = [
        (r"no\s+(grains?|steel|fertilizers?)", "avoids"),
        (r"(grains?|steel|fertilizers?)\s+only", "specializes"),
        (r"prefers?\s+(grains?|steel|fertilizers?)", "prefers")
    ]
    
    for pattern, pref_type in cargo_patterns:
        matches = re.findall(pattern, comments_lower)
        for match in matches:
            result["cargo_preferences"].append({
                "cargo_type": match,
                "preference": pref_type
            })
    
    return result
```

---

## 8. KNOWLEDGE BASE STRUCTURE

### 8.1 File Organization

```
knowledge_base/
├── scoring/
│   ├── proximity_scoring_matrix.yaml      # P1 rules
│   ├── regional_trade_patterns.yaml       # P1A rules
│   ├── comments_processing_guide.yaml     # P2, P3 rules
│   ├── intake_calculator_formula.yaml     # P5 rules
│   ├── open_area_comments_scoring.yaml    # P6 rules
│   └── readiness_eta_scoring.yaml         # P7 rules
│
├── reference/
│   ├── vessel_size_classification.yaml
│   ├── vessel_location_determination.yaml
│   ├── port_restrictions_map.yaml
│   └── cargo_export_map.yaml
│
├── templates/
│   ├── email/
│   │   ├── case_a_strong_match.txt
│   │   ├── case_a_standard.txt
│   │   ├── case_b_not_ideal.txt
│   │   ├── case_b_positioning.txt
│   │   ├── reminder_1.txt
│   │   ├── reminder_2.txt
│   │   └── reminder_3.txt
│   └── whatsapp/
│       ├── offer_short.txt
│       └── reminder_short.txt
│
├── examples/
│   ├── scoring_examples.yaml
│   ├── message_examples.yaml
│   └── edge_cases.yaml
│
└── learning/
    └── learned_patterns.yaml  # Updated by system
```

### 8.2 RAG Configuration

```python
# app/knowledge/rag_retriever.py

from langchain_mongodb import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter

class KnowledgeRetriever:
    """
    RAG-based knowledge retrieval from MongoDB Atlas.
    """
    
    def __init__(self, mongo_collection, embedding_model="text-embedding-3-small"):
        self.embeddings = OpenAIEmbeddings(model=embedding_model)
        self.vector_store = MongoDBAtlasVectorSearch(
            collection=mongo_collection,
            embedding=self.embeddings,
            index_name="vector_index",
            text_key="chunk_text",
            embedding_key="embedding"
        )
        self.splitter = RecursiveCharacterTextSplitter(
            chunk_size=500,
            chunk_overlap=50,
            separators=["\n\n", "\n", ". ", " "]
        )
    
    async def ingest_file(self, file_path: str, metadata: dict):
        """Ingest knowledge file into vector store."""
        with open(file_path, 'r') as f:
            content = f.read()
        
        chunks = self.splitter.split_text(content)
        
        documents = []
        for i, chunk in enumerate(chunks):
            documents.append({
                "chunk_text": chunk,
                "chunk_id": f"{file_path}_{i}",
                "source_file": file_path,
                "metadata": metadata
            })
        
        await self.vector_store.aadd_documents(documents)
    
    async def aretrieve(self, query: str, k: int = 5, filter: dict = None) -> list:
        """Retrieve relevant knowledge chunks."""
        return await self.vector_store.asimilarity_search(
            query,
            k=k,
            pre_filter=filter
        )
    
    async def retrieve_by_criterion(self, criterion: str, context: str, k: int = 3) -> list:
        """Retrieve knowledge specific to a scoring criterion."""
        return await self.aretrieve(
            query=context,
            k=k,
            filter={"metadata.criterion": criterion}
        )
```

---

## 9. NORMALIZATION SYSTEM

### 9.1 Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        NORMALIZATION PIPELINE                                │
└─────────────────────────────────────────────────────────────────────────────┘

Input: "cta" (from vessel.open_area)
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: EXACT MATCH                                          │
│                                                              │
│ Check: aliases/ports_aliases.yaml                            │
│ {                                                            │
│   "cta": "ROCTA",                                           │
│   "konstanza": "ROCTA",                                     │
│   "constancia": "ROCTA",                                    │
│   ...                                                        │
│ }                                                            │
│                                                              │
│ Found? → Return "ROCTA" (confidence: 1.0)                   │
│ Not found? → Continue                                        │
└─────────────────────────────────────────────────────────────┘
         │ Not found
         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: FUZZY MATCH                                          │
│                                                              │
│ Algorithm: Levenshtein distance + phonetic matching          │
│ Threshold: similarity > 0.85                                 │
│                                                              │
│ Check against all known values:                              │
│ - "cta" vs "constanta" → similarity: 0.62 ❌                │
│ - "cta" vs "catania" → similarity: 0.71 ❌                  │
│                                                              │
│ Best match > 0.85? → Return with confidence                  │
│ No match? → Continue                                         │
└─────────────────────────────────────────────────────────────┘
         │ No match
         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: LLM FALLBACK                                         │
│                                                              │
│ Prompt to Claude:                                            │
│ "In maritime shipping context, what port does 'cta' refer   │
│  to? Options: [list of known ports in region]               │
│  Return the UNLOCODE and confidence level."                 │
│                                                              │
│ Response: {"port": "Constanta", "unlocode": "ROCTA",        │
│            "confidence": 0.92}                               │
│                                                              │
│ Confidence > 0.7? → Accept and continue                      │
│ Confidence ≤ 0.7? → Flag for manual review                  │
└─────────────────────────────────────────────────────────────┘
         │ Accepted
         ▼
┌─────────────────────────────────────────────────────────────┐
│ STEP 4: AUTO-ADD ALIAS                                       │
│                                                              │
│ 1. Add to MongoDB: aliases_auto collection                   │
│    {                                                         │
│      entity_type: "port",                                   │
│      original_value: "cta",                                 │
│      normalized_value: "ROCTA",                             │
│      source: "llm_fallback",                                │
│      confidence: 0.92,                                      │
│      status: "pending"                                      │
│    }                                                         │
│                                                              │
│ 2. Publish to queue: vkchart.normalize.logs                 │
│    → Laravel receives notification                           │
│    → Shows in "Pending Aliases" UI                          │
│                                                              │
│ 3. User reviews:                                             │
│    - Approve → Move to ports_aliases.yaml                   │
│    - Reject → Log reason, don't use again                   │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
Output: "ROCTA" (Constanta)
```

### 9.2 Implementation

```python
# app/normalizers/port_normalizer.py

from rapidfuzz import fuzz, process
from typing import Optional, Tuple

class PortNormalizer:
    """
    Multi-stage port name normalizer.
    """
    
    def __init__(self, aliases_path: str, llm_client, mongo_client):
        self.aliases = self._load_aliases(aliases_path)
        self.llm = llm_client
        self.mongo = mongo_client
        
        # Known ports for fuzzy matching
        self.known_ports = list(self.aliases.values())
    
    async def normalize(
        self, 
        value: str, 
        context: dict = None
    ) -> Tuple[str, float, str]:
        """
        Normalize port name.
        
        Returns:
            Tuple of (normalized_value, confidence, method)
        """
        value_lower = value.lower().strip()
        
        # Step 1: Exact match
        if value_lower in self.aliases:
            return self.aliases[value_lower], 1.0, "exact"
        
        # Step 2: Fuzzy match
        fuzzy_result = self._fuzzy_match(value_lower)
        if fuzzy_result:
            return fuzzy_result
        
        # Step 3: LLM fallback
        llm_result = await self._llm_normalize(value, context)
        if llm_result[1] > 0.7:  # confidence threshold
            await self._auto_add_alias(value_lower, llm_result[0], llm_result[1], context)
        
        return llm_result
    
    def _fuzzy_match(self, value: str) -> Optional[Tuple[str, float, str]]:
        """Try fuzzy matching against known ports."""
        # Check aliases keys first
        alias_match = process.extractOne(
            value,
            self.aliases.keys(),
            scorer=fuzz.ratio,
            score_cutoff=85
        )
        
        if alias_match:
            return self.aliases[alias_match[0]], alias_match[1] / 100, "fuzzy"
        
        return None
    
    async def _llm_normalize(
        self, 
        value: str, 
        context: dict
    ) -> Tuple[str, float, str]:
        """Use LLM to interpret unknown port name."""
        # Get regional context if available
        region_hint = ""
        if context and context.get("vessel_open_area"):
            region_hint = f"The vessel is in {context['vessel_open_area']} region."
        
        prompt = f"""
In maritime shipping, what port does "{value}" refer to?
{region_hint}

Known ports in this context:
{self._get_regional_ports(context)}

Return JSON:
{{
  "port_name": "Full port name",
  "unlocode": "XXYYY format",
  "confidence": 0.0-1.0,
  "reasoning": "Why this match"
}}

If uncertain, set confidence below 0.7.
"""
        
        response = await self.llm.ainvoke(prompt)
        result = self._parse_llm_response(response)
        
        return result["unlocode"], result["confidence"], "llm"
    
    async def _auto_add_alias(
        self, 
        original: str, 
        normalized: str, 
        confidence: float,
        context: dict
    ):
        """Add new alias for review."""
        await self.mongo.aliases_auto.insert_one({
            "entity_type": "port",
            "original_value": original,
            "normalized_value": normalized,
            "source": "llm_fallback",
            "confidence": confidence,
            "context": context,
            "status": "pending",
            "created_at": datetime.utcnow()
        })
        
        # Publish notification
        await self._notify_new_alias(original, normalized, confidence)
```

---

## 10. LEARNING MECHANISM

### 10.1 Feedback Loop

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           LEARNING FEEDBACK LOOP                             │
└─────────────────────────────────────────────────────────────────────────────┘

[1] AI generates scoring result
         │
         ▼
┌─────────────┐
│   UI Shows  │  Score: 72/100
│   Results   │  P1: 15, P2: 10, P3: 12...
└─────────────┘
         │
         │ [2] User reviews
         ▼
┌─────────────────────────────────────────────────────────────┐
│ USER ACTION                                                  │
│                                                              │
│ Option A: Approve ✓                                         │
│   → Score confirmed                                          │
│   → Increase confidence in similar patterns                  │
│                                                              │
│ Option B: Correct ✎                                         │
│   → User changes P1 from 15 to 8                            │
│   → System asks: "Why should P1 be lower?"                  │
│   → User: "Libya to Black Sea is not profitable for Handy"  │
│                                                              │
│ Option C: Skip ✗                                            │
│   → Don't send offer                                         │
│   → Optional: explain why                                    │
└─────────────────────────────────────────────────────────────┘
         │
         │ [3] If corrected
         ▼
┌─────────────────────────────────────────────────────────────┐
│ PATTERN EXTRACTION                                           │
│                                                              │
│ System analyzes:                                             │
│ - What was different?                                        │
│ - What context led to error?                                 │
│ - Is there a pattern?                                        │
│                                                              │
│ Extracted pattern:                                           │
│ {                                                            │
│   "context": {                                              │
│     "vessel_type": "Handysize",                            │
│     "region_from": "Libya",                                │
│     "region_to": "Black Sea"                               │
│   },                                                         │
│   "criterion": "P1",                                        │
│   "adjustment": -7,                                         │
│   "reason": "Not profitable for Handy"                     │
│ }                                                            │
└─────────────────────────────────────────────────────────────┘
         │
         │ [4] Store pattern
         ▼
┌─────────────────────────────────────────────────────────────┐
│ MONGODB: learning_patterns                                   │
│                                                              │
│ Check for similar existing pattern:                          │
│ - Found similar? → Increase confidence                       │
│ - New pattern? → Store with confidence=1                     │
│                                                              │
│ Vector embedding for similarity search                       │
└─────────────────────────────────────────────────────────────┘
         │
         │ [5] Apply in future
         ▼
┌─────────────────────────────────────────────────────────────┐
│ NEXT SCORING REQUEST                                         │
│                                                              │
│ Similar context detected:                                    │
│ - Vessel: Handymax (similar to Handysize)                   │
│ - From: Egypt (similar to Libya)                            │
│ - To: Black Sea ✓                                           │
│                                                              │
│ System applies learned adjustment:                           │
│ "Based on previous corrections, reducing P1 score.          │
│  Similar pattern: Libya→Black Sea not profitable for Handy" │
│                                                              │
│ Confidence shown to user for transparency                    │
└─────────────────────────────────────────────────────────────┘
```

### 10.2 Implementation

```python
# app/learning/feedback_collector.py

class FeedbackCollector:
    """
    Collects and processes user feedback for learning.
    """
    
    async def process_correction(
        self,
        scoring_result_id: str,
        criterion: str,
        original_score: int,
        corrected_score: int,
        user_explanation: str,
        user_id: int
    ):
        """Process user's score correction."""
        
        # 1. Fetch original scoring result
        original = await self.mongo.scoring_results.find_one({
            "_id": ObjectId(scoring_result_id)
        })
        
        # 2. Extract context
        context = {
            "vessel_type": self._categorize_vessel(original["snapshot"]["vessel"]["dwt"]),
            "cargo_type": original["snapshot"]["cargo"]["type"],
            "region_from": original["snapshot"]["vessel"]["open_area"],
            "region_to": original["snapshot"]["cargo"]["load_region"],
            "season": self._get_season(original["created_at"])
        }
        
        # 3. Create pattern
        pattern = {
            "pattern_type": "score_correction",
            "context": context,
            "criterion": criterion,
            "original": {
                "score": original_score,
                "reasoning": original["scores"][criterion]["reasoning"]
            },
            "corrected": {
                "score": corrected_score,
                "user_explanation": user_explanation
            },
            "adjustment": corrected_score - original_score,
            "created_by": user_id,
            "created_at": datetime.utcnow()
        }
        
        # 4. Check for similar patterns
        similar = await self._find_similar_patterns(context, criterion)
        
        if similar:
            # Update existing pattern confidence
            await self._reinforce_pattern(similar["_id"], pattern)
        else:
            # Create new pattern
            pattern["confidence"] = 1
            pattern["applied_count"] = 0
            await self.mongo.learning_patterns.insert_one(pattern)
        
        # 5. Generate embedding for future similarity search
        await self._create_pattern_embedding(pattern)
    
    async def _find_similar_patterns(self, context: dict, criterion: str) -> Optional[dict]:
        """Find similar learned patterns using vector search."""
        query = f"{context['vessel_type']} {context['region_from']} to {context['region_to']} {context['cargo_type']}"
        
        # Vector similarity search
        results = await self.vector_store.asimilarity_search(
            query,
            k=3,
            pre_filter={
                "pattern_type": "score_correction",
                "criterion": criterion
            }
        )
        
        # Check if any result is similar enough
        for result in results:
            if self._context_matches(result["context"], context):
                return result
        
        return None
    
    async def _reinforce_pattern(self, pattern_id: ObjectId, new_correction: dict):
        """Reinforce existing pattern with new confirmation."""
        await self.mongo.learning_patterns.update_one(
            {"_id": pattern_id},
            {
                "$inc": {"confidence": 1},
                "$push": {"confirmations": new_correction},
                "$set": {"last_confirmed": datetime.utcnow()}
            }
        )
```

---

## 11. TECHNOLOGY STACK

### 11.1 Backend Stack

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Main App | Laravel | 11.x | Existing CRM, orchestration |
| AI Service | Python | 3.11+ | AI agents, scoring |
| API Framework | FastAPI | 0.104+ | Python API endpoints |
| AI Framework | LangChain | 0.1+ | Agent orchestration |
| Queue | RabbitMQ | 3.12+ | Async messaging |
| Cache | Redis | 7.x | Caching, sessions |
| Primary DB | MySQL | 8.x | Transactional data |
| AI DB | MongoDB Atlas | 7.x | Vectors, learning |

### 11.2 AI/ML Stack

| Component | Technology | Purpose |
|-----------|------------|---------|  
| Primary LLM | Claude 3.5 Sonnet | Scoring, reasoning |
| Backup LLM | GPT-4 Turbo | Fallback |
| Embeddings | OpenAI Ada-002 | Vector embeddings |
| Vector Search | MongoDB Atlas Search | RAG retrieval |
| Fuzzy Match | RapidFuzz | Text matching |
| **★ Workflow Engine** | **LangGraph** | **State management, conditional routing, checkpoints** |
| **★ Observability** | **LangSmith** | **Tracing, datasets, evaluation, monitoring** |
| **★ API Framework** | **LangServe** | **FastAPI integration, streaming, playground** |

### 11.3 Infrastructure

```yaml
# docker-compose.yaml (development)

version: '3.8'

services:
  
  # Existing Laravel app (DDEV)
  # ... existing config ...
  
  # New Python AI Service
  ai-service:
    build:
      context: ./vkchart-ai-service
      dockerfile: Dockerfile
    ports:
      - "8001:8000"
    environment:
      - MONGODB_URI=${MONGODB_URI}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - RABBITMQ_URL=${RABBITMQ_URL}
      - LOG_LEVEL=INFO
    volumes:
      - ./vkchart-ai-service:/app
      - ./knowledge_base:/app/knowledge_base
    depends_on:
      - rabbitmq
    restart: unless-stopped
  
  # RabbitMQ (if not already in DDEV)
  rabbitmq:
    image: rabbitmq:3.12-management
    ports:
      - "5672:5672"
      - "15672:15672"  # Management UI
    environment:
      - RABBITMQ_DEFAULT_USER=vkchart
      - RABBITMQ_DEFAULT_PASS=${RABBITMQ_PASSWORD}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    restart: unless-stopped

volumes:
  rabbitmq_data:
```

### 11.4 Python Dependencies

```txt
# requirements.txt

# Core
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.2
python-dotenv==1.0.0

# AI/LLM
langchain==0.1.0
langchain-anthropic==0.1.1
langchain-openai==0.0.5
langchain-mongodb==0.1.1
langgraph==0.2.0  # State management, workflows
langsmith==0.1.0  # Observability, tracing
langserve[all]==0.1.0  # API deployment

# Database
motor==3.3.2  # Async MongoDB
pymongo==4.6.1
redis==5.0.1

# Queue
aio-pika==9.3.1  # Async RabbitMQ

# Text Processing
rapidfuzz==3.5.2
pyyaml==6.0.1
tiktoken==0.5.2

# Utilities
httpx==0.25.2
structlog==23.2.0
tenacity==8.2.3  # Retries

# Testing
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0
```

---

## 12. IMPLEMENTATION PHASES

### Phase 1: Foundation (Week 1-2)
**Goal**: Working scoring for 4 criteria + basic message generation

```
Week 1:
├── Day 1-2: Setup
│   ├── Python project structure
│   ├── FastAPI endpoints
│   ├── MongoDB connection
│   └── RabbitMQ setup
│
├── Day 3-4: Normalization
│   ├── Port normalizer (rules + fuzzy)
│   ├── Region normalizer
│   └── Basic aliases files
│
└── Day 5: Laravel Integration
    ├── Event dispatchers
    ├── Queue jobs
    └── Basic API client

Week 2:
├── Day 1-2: Scoring Agent (P1, P5, P6, P7)
│   ├── Tools implementation
│   ├── Knowledge loading
│   └── Basic RAG
│
├── Day 3-4: Message Agent (basic)
│   ├── Template loading
│   ├── Basic personalization
│   └── Email generation
│
└── Day 5: Integration Testing
    ├── End-to-end flow
    ├── Fix issues
    └── Demo
```

**Deliverables**:
- [ ] Scoring for P1 (Proximity), P5 (Intake), P6 (OpenArea), P7 (Readiness)
- [ ] Basic message generation
- [ ] Laravel ↔ Python integration
- [ ] Admin can test scoring via UI

### Phase 2: Full Scoring (Week 3)
**Goal**: All 7 criteria working

```
Week 3:
├── Day 1-2: Remaining Criteria
│   ├── P1A (Regional Patterns)
│   ├── P2 (Regional Preferences)
│   ├── P3 (Cargo Preferences)
│   └── P4 (Last Ports)
│
├── Day 3: Knowledge Base Enhancement
│   ├── All YAML files converted
│   ├── RAG indexing
│   └── Testing retrieval
│
├── Day 4: LLM Fallback Normalizer
│   ├── Implementation
│   ├── Auto-add aliases
│   └── Notification system
│
└── Day 5: Testing & Tuning
    ├── 50 test cases
    ├── Accuracy measurement
    └── Prompt tuning
```

**Deliverables**:
- [ ] All P1-P7 criteria working
- [ ] LLM fallback for normalization
- [ ] Auto-alias pending review UI
- [ ] >80% accuracy on test cases

### Phase 3: Learning & Polish (Week 4)
**Goal**: Learning loop + production-ready

```
Week 4:
├── Day 1-2: Learning System
│   ├── Feedback collector
│   ├── Pattern storage
│   ├── Pattern application
│   └── UI for corrections
│
├── Day 3: Message Enhancement
│   ├── All templates
│   ├── Full personalization
│   ├── Reminder logic
│   └── WhatsApp format
│
├── Day 4: Monitoring & Logging
│   ├── Metrics collection
│   ├── Error tracking
│   ├── Performance monitoring
│   └── Cost tracking
│
└── Day 5: Production Prep
    ├── Security review
    ├── Load testing
    ├── Documentation
    └── Deployment
```

**Deliverables**:
- [ ] Learning from corrections
- [ ] Full message personalization
- [ ] Monitoring dashboard
- [ ] Production deployment

---

## 13. RISKS & MITIGATIONS

### Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| LLM latency too high | Medium | High | Implement caching, batch processing |
| LLM costs exceed budget | Medium | Medium | Monitor costs, use smaller models where possible |
| Scoring accuracy < 80% | Medium | High | More training examples, prompt engineering |
| RabbitMQ bottleneck | Low | Medium | Horizontal scaling, priority queues |
| MongoDB performance | Low | Medium | Proper indexing, connection pooling |

### Schedule Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Integration delays | High | Medium | Start integration early, mock APIs |
| Scope creep | High | High | Strict MVP scope, backlog management |
| Knowledge base incomplete | Medium | Medium | Iterative improvement, fallback rules |
| Testing time insufficient | Medium | High | Automated tests, parallel testing |

### Operational Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| User adoption low | Medium | Medium | Training, gradual rollout |
| False positives in scoring | Medium | High | Human review for high-impact decisions |
| Data quality issues | Medium | Medium | Validation, normalization |

---

## 14. LANGGRAPH & LANGSMITH INTEGRATION

### 14.1 LangGraph Workflow Architecture

**Ключевые концепции:**
- **State Graph**: Граф с конечным состоянием, управляющий потоком скоринга
- **Nodes**: Критерии P1-P7, каждый - нода в графе
- **Edges**: Условные переходы между критериями
- **Checkpoints**: Сохранение состояния для Human-in-the-Loop

```python
# app/workflows/scoring_workflow.py

from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, Annotated
import operator

class ScoringState(TypedDict):
    """State for scoring workflow."""
    # Input data
    vessel: dict
    cargo: dict
    company: dict
    
    # Scores (accumulated)
    scores: Annotated[dict, operator.add]
    
    # Control flow
    current_criterion: str
    is_blocked: bool
    block_reason: str
    
    # Results
    final_score: int
    reasoning_summary: str
    
def create_scoring_workflow():
    """Create LangGraph workflow for scoring."""
    
    # Initialize workflow
    workflow = StateGraph(ScoringState)
    
    # Add nodes for each criterion
    workflow.add_node("normalize", normalize_data)
    workflow.add_node("check_data", check_data_quality)
    workflow.add_node("p1_proximity", score_p1)
    workflow.add_node("p1a_patterns", score_p1a)
    workflow.add_node("p2_regional", score_p2)
    workflow.add_node("p3_cargo", score_p3)
    workflow.add_node("p4_last_ports", score_p4)
    workflow.add_node("p5_intake", score_p5)
    workflow.add_node("p6_open_area", score_p6)
    workflow.add_node("p7_readiness", score_p7)
    workflow.add_node("finalize", finalize_scoring)
    
    # Define edges
    workflow.set_entry_point("normalize")
    workflow.add_edge("normalize", "check_data")
    workflow.add_edge("check_data", "p1_proximity")
    workflow.add_edge("p1_proximity", "p1a_patterns")
    workflow.add_edge("p1a_patterns", "p2_regional")
    workflow.add_edge("p2_regional", "p3_cargo")
    workflow.add_edge("p3_cargo", "p4_last_ports")
    workflow.add_edge("p4_last_ports", "p5_intake")
    
    # Conditional routing after P5 (intake check)
    workflow.add_conditional_edges(
        "p5_intake",
        check_intake_block,
        {
            "continue": "p6_open_area",
            "block": "finalize"
        }
    )
    
    workflow.add_edge("p6_open_area", "p7_readiness")
    
    # Conditional routing after P6 (preferences check)
    workflow.add_conditional_edges(
        "p7_readiness",
        check_p6_block,
        {
            "continue": "finalize",
            "block": "finalize"
        }
    )
    
    workflow.add_edge("finalize", END)
    
    # Add checkpointer for human-in-the-loop
    memory = MemorySaver()
    app = workflow.compile(checkpointer=memory)
    
    return app

def check_intake_block(state: ScoringState) -> str:
    """Check if vessel should be blocked based on intake."""
    if state["scores"]["p5_intake"]["score"] <= 0:
        state["is_blocked"] = True
        state["block_reason"] = "Intake capacity insufficient"
        return "block"
    return "continue"

def check_p6_block(state: ScoringState) -> str:
    """Check if vessel should be blocked based on P6 score."""
    if state["scores"]["p6_open_area"]["score"] <= -40:
        state["is_blocked"] = True
        state["block_reason"] = "Owner explicitly avoids this trade"
        return "block"
    return "continue"
```

### 14.2 LangSmith Integration

**Функционал:**
1. **Auto-tracing**: Автоматическая запись всех LLM вызовов
2. **Datasets**: Создание test sets для evaluation
3. **Evaluation**: Измерение accuracy scoring
4. **Monitoring**: Real-time dashboards в production

```python
# app/config.py

import os
from langsmith import Client

# LangSmith configuration
LANGSMITH_API_KEY = os.getenv("LANGSMITH_API_KEY")
LANGSMITH_PROJECT = "vkchart-scoring"

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = LANGSMITH_PROJECT
os.environ["LANGCHAIN_API_KEY"] = LANGSMITH_API_KEY

langsmith_client = Client()

# app/evaluation/evaluator.py

from langsmith import evaluate
from langsmith.schemas import Run, Example

class ScoringEvaluator:
    """Evaluate scoring accuracy using LangSmith."""
    
    async def create_dataset(self, name: str, examples: list):
        """Create evaluation dataset."""
        dataset = self.client.create_dataset(name)
        
        for example in examples:
            self.client.create_example(
                inputs={
                    "vessel": example["vessel"],
                    "cargo": example["cargo"]
                },
                outputs={
                    "expected_score": example["expected_score"],
                    "expected_classification": example["expected_class"]
                },
                dataset_id=dataset.id
            )
    
    async def run_evaluation(self, dataset_name: str):
        """Run evaluation on dataset."""
        
        def score_accuracy(run: Run, example: Example) -> dict:
            """Calculate scoring accuracy."""
            predicted = run.outputs["final_score"]
            expected = example.outputs["expected_score"]
            
            diff = abs(predicted - expected)
            
            return {
                "score": 1.0 - (diff / 100),  # Normalize to 0-1
                "diff": diff
            }
        
        results = evaluate(
            lambda inputs: self.scoring_agent.score(inputs),
            data=dataset_name,
            evaluators=[score_accuracy],
            experiment_prefix="scoring-eval"
        )
        
        return results
```

### 14.3 LangServe Deployment

```python
# app/main.py

from fastapi import FastAPI
from langserve import add_routes
from app.workflows.scoring_workflow import create_scoring_workflow
from app.agents.message_agent import create_message_chain

app = FastAPI(
    title="VK Chart AI Service",
    version="2.0",
    description="Vessel-Cargo Scoring & Message Generation"
)

# Create workflows
scoring_workflow = create_scoring_workflow()
message_chain = create_message_chain()

# Add LangServe routes with streaming
add_routes(
    app,
    scoring_workflow.with_types(
        input_type=ScoringInput,
        output_type=ScoringOutput
    ),
    path="/scoring",
    enable_feedback_endpoint=True,
    enable_public_trace_link_endpoint=True
)

add_routes(
    app,
    message_chain,
    path="/messages",
    enable_feedback_endpoint=True
)

# Playground UI available at /scoring/playground
```

### 14.4 Human-in-the-Loop Pattern

```python
# app/workflows/hitl.py

async def score_with_correction_loop(
    vessel: dict,
    cargo: dict,
    user_id: int
):
    """Score with ability to correct and resume."""
    
    # Create unique thread ID
    thread_id = f"scoring_{vessel['id']}_{cargo['id']}_{user_id}"
    
    config = {
        "configurable": {
            "thread_id": thread_id
        }
    }
    
    # Run workflow
    workflow = create_scoring_workflow()
    result = await workflow.ainvoke(
        {
            "vessel": vessel,
            "cargo": cargo
        },
        config=config
    )
    
    # Show result to user
    # User can correct specific criterion
    
    return result

async def correct_and_resume(
    thread_id: str,
    criterion: str,
    corrected_score: int,
    reason: str
):
    """Correct score and resume workflow."""
    
    workflow = create_scoring_workflow()
    
    # Get current state
    state = workflow.get_state(
        config={"configurable": {"thread_id": thread_id}}
    )
    
    # Update specific score
    state.values["scores"][criterion]["score"] = corrected_score
    state.values["scores"][criterion]["reasoning"] += f" [User correction: {reason}]"
    
    # Resume from checkpoint
    result = await workflow.ainvoke(
        None,  # Resume from current state
        config={
            "configurable": {"thread_id": thread_id}
        }
    )
    
    # Log correction for learning
    await log_correction(thread_id, criterion, corrected_score, reason)
    
    return result
```

### 14.5 Benefits Summary

| Feature | Before | After (with Lang* Stack) |
|---------|--------|-------------------------|
| **State Management** | Manual dict passing | LangGraph state graph |
| **Conditional Logic** | If-else in code | Conditional edges in graph |
| **Debugging** | Print statements | LangSmith trace explorer |
| **Testing** | Manual test cases | LangSmith datasets |
| **Corrections** | Rerun from scratch | Resume from checkpoint |
| **Monitoring** | Custom logging | LangSmith dashboard |
| **API Docs** | Manual Swagger | LangServe auto-docs + playground |
| **Streaming** | Custom SSE | Built-in streaming |

---

## APPENDIX A: File Conversion Guide

### Converting .txt to .yaml for RAG

Current files need conversion:
- `proximity_scoring_matrix.txt` → `proximity_scoring_matrix.yaml`
- `regional_trade_patterns.txt` → `regional_trade_patterns.yaml`
- etc.

**Structure**:
```yaml
# proximity_scoring_matrix.yaml

metadata:
  criterion: P1
  version: "1.0"
  description: "Proximity scoring rules"

levels:
  - level: 1
    name: "Same Port"
    score_range: [23, 25]
    description: "Vessel at loading port"
    examples:
      - vessel_location: "Odessa"
        load_port: "Odessa"
        score: 25
        
  - level: 2
    name: "Same Sub-Region"
    score_range: [20, 23]
    # ...

size_adjustments:
  - vessel_size: "Coaster"
    dwt_range: [0, 17000]
    adjustments:
      long_distance_penalty: -5
      short_distance_bonus: +2

exceptions:
  - condition: "Libya to Black Sea"
    vessel_sizes: ["Handysize", "Handymax"]
    adjustment: -7
    reason: "Not profitable for this size"
```

---

## APPENDIX B: Testing Checklist

### Scoring Tests
- [ ] P1: Same port = 25 points
- [ ] P1: Different sea = 5-10 points
- [ ] P5: Intake < 90% = BLOCK
- [ ] P6: "no grains" + grain cargo = -40
- [ ] P7: ETA before laycan = high score
- [ ] P7: ETA after laycan = low score
- [ ] Final score calculation correct
- [ ] Blocking conditions work

### Normalization Tests
- [ ] Exact match: "odessa" → "UAODS"
- [ ] Fuzzy match: "odesa" → "UAODS"
- [ ] LLM fallback: "cta" → "ROCTA"
- [ ] Auto-add to pending

### Message Tests
- [ ] Strong match template selected for score > 85
- [ ] Personalization by nationality
- [ ] No greeting if today_contact
- [ ] Reminder references previous message

### Integration Tests
- [ ] Laravel → RabbitMQ → Python → MongoDB → Laravel
- [ ] Error handling on LLM failure
- [ ] Timeout handling
- [ ] Retry logic

---

*End of Architecture Document*
