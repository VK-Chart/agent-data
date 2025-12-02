# VKChart-AI Architecture Diagrams

**Версия**: 2.0  
**Дата**: 2024-12-02  
**Статус**: Updated with Lang* Stack

> 💡 **Note**: Полная обновленная диаграмма с детальной интеграцией LangGraph, LangSmith, LangServe находится в файле:  
> **`Mermaid_FINAL_Complete_System_With_Triggers_v2.md`**

---

## 1. High-Level System Architecture

```mermaid
flowchart TB
    subgraph Laravel["🔵 LARAVEL APPLICATION"]
        UI["👤 UI/Dashboard"]
        API["🔌 API Gateway"]
        Events["📡 Event Dispatcher"]
        Jobs["⚙️ Queue Jobs"]
    end

    subgraph RabbitMQ["🐰 RABBITMQ"]
        Q1["scoring.requests"]
        Q2["scoring.results"]
        Q3["messages.requests"]
        Q4["messages.results"]
        Q5["learning.feedback"]
        Q6["normalize.logs"]
    end

    subgraph Python["🐍 PYTHON AI SERVICE"]
        subgraph Workers["Workers"]
            SW["Scoring Worker"]
            MW["Message Worker"]
            LW["Learning Worker"]
        end
        
        subgraph Agents["AI Agents"]
            SA["🎯 Scoring Agent<br/>P1-P7 Calculation"]
            MA["✉️ Message Agent<br/>Personalization"]
        end
        
        subgraph Components["Components"]
            NRM["📝 Normalizer<br/>Rules + LLM"]
            RAG["🔍 RAG Retriever"]
            Tools["🔧 Tools<br/>Intake, Distance"]
            Learn["🧠 Learning Module"]
        end
    end

    subgraph MongoDB["🍃 MONGODB ATLAS"]
        SR[("scoring_results")]
        KE[("knowledge_embeddings")]
        LP[("learning_patterns")]
        AA[("aliases_auto")]
        MH[("message_history")]
    end

    subgraph External["☁️ EXTERNAL APIs"]
        Claude["Claude API"]
        OpenAI["OpenAI API"]
        Gmail["Gmail API"]
    end

    UI --> API
    API --> Events
    Events --> Jobs
    Jobs --> Q1
    Jobs --> Q3
    Jobs --> Q5

    Q1 --> SW
    Q3 --> MW
    Q5 --> LW

    SW --> SA
    MW --> MA
    LW --> Learn

    SA --> NRM
    SA --> RAG
    SA --> Tools
    SA --> Claude

    MA --> RAG
    MA --> Claude

    NRM --> Claude
    NRM --> AA

    SA --> SR
    MA --> MH
    Learn --> LP
    RAG --> KE

    Q2 --> Jobs
    Q4 --> Jobs
    Q6 --> Jobs

    SA --> Q2
    MA --> Q4
    NRM --> Q6

    Gmail -.-> Jobs

    style Laravel fill:#3498db,color:#fff
    style Python fill:#27ae60,color:#fff
    style MongoDB fill:#16a085,color:#fff
    style RabbitMQ fill:#e67e22,color:#fff
```

## 2. Scoring Data Flow

```mermaid
sequenceDiagram
    participant UI as Laravel UI
    participant E as Event Dispatcher
    participant RMQ as RabbitMQ
    participant SW as Scoring Worker
    participant NRM as Normalizer
    participant SA as Scoring Agent
    participant RAG as RAG Retriever
    participant Tools as Tools
    participant LLM as Claude API
    participant DB as MongoDB

    UI->>E: Trigger: CargoUpdated
    E->>RMQ: Publish to scoring.requests
    Note over E,RMQ: {vessel, cargo, company, person}
    
    RMQ->>SW: Consume message
    
    SW->>NRM: Normalize open_area
    alt Exact Match
        NRM-->>SW: Return cached value
    else Fuzzy Match
        NRM-->>SW: Return fuzzy match
    else LLM Fallback
        NRM->>LLM: "What port is 'cta'?"
        LLM-->>NRM: "ROCTA (Constanta)"
        NRM->>DB: Auto-add alias (pending)
        NRM-->>SW: Return normalized
    end
    
    SW->>SA: Calculate scores
    
    loop For each criterion P1-P7
        SA->>RAG: Get relevant knowledge
        RAG->>DB: Vector search
        DB-->>RAG: Relevant chunks
        RAG-->>SA: Knowledge context
        
        SA->>Tools: Call if needed
        Note over SA,Tools: calculate_intake(), estimate_distance()
        Tools-->>SA: Tool results
        
        SA->>DB: Check learning patterns
        DB-->>SA: Similar corrections
        
        SA->>LLM: Generate score + reasoning
        LLM-->>SA: {score, reasoning}
    end
    
    SA->>SA: Calculate final score
    SA->>SA: Check blocking conditions
    SA->>DB: Save scoring_result
    
    SA->>RMQ: Publish to scoring.results
    RMQ->>UI: Consume & display
    
    UI->>UI: Show score breakdown
```

## 3. Normalization Pipeline

```mermaid
flowchart TD
    Input["Input: 'cta'"]
    
    subgraph Step1["Step 1: Exact Match"]
        S1Check{"In aliases.yaml?"}
        S1Yes["Return: ROCTA<br/>Confidence: 1.0"]
    end
    
    subgraph Step2["Step 2: Fuzzy Match"]
        S2Check{"Similarity > 85%?"}
        S2Yes["Return match<br/>Confidence: 0.85-0.99"]
    end
    
    subgraph Step3["Step 3: LLM Fallback"]
        S3Call["Call Claude:<br/>'What port is cta?'"]
        S3Check{"Confidence > 70%?"}
        S3Yes["Accept result"]
        S3No["Flag for review"]
    end
    
    subgraph Step4["Step 4: Auto-Add"]
        S4Add["Add to aliases_auto<br/>status: pending"]
        S4Notify["Notify admin"]
    end
    
    Output["Output: 'ROCTA'"]
    
    Input --> S1Check
    S1Check -->|Yes| S1Yes
    S1Check -->|No| S2Check
    S1Yes --> Output
    
    S2Check -->|Yes| S2Yes
    S2Check -->|No| S3Call
    S2Yes --> Output
    
    S3Call --> S3Check
    S3Check -->|Yes| S3Yes
    S3Check -->|No| S3No
    
    S3Yes --> S4Add
    S3No --> S4Add
    
    S4Add --> S4Notify
    S4Notify --> Output
    
    style Step1 fill:#27ae60,stroke:#1e8449
    style Step2 fill:#3498db,stroke:#2980b9
    style Step3 fill:#e74c3c,stroke:#c0392b
    style Step4 fill:#9b59b6,stroke:#8e44ad
```

## 4. Learning Feedback Loop

```mermaid
flowchart TD
    subgraph Generate["1. AI Generates Score"]
        AI["Scoring Agent"]
        Score["Score: 72/100<br/>P1: 15, P2: 10..."]
    end
    
    subgraph Review["2. User Reviews"]
        UI["Show in UI"]
        Decision{"User Action?"}
        Approve["✓ Approve"]
        Correct["✎ Correct"]
        Skip["✗ Skip"]
    end
    
    subgraph Feedback["3. Collect Feedback"]
        Change["User changes P1: 15 → 8"]
        Ask["System asks: Why?"]
        Explain["User: 'Libya→BS not profitable<br/>for Handysize'"]
    end
    
    subgraph Learn["4. Learn Pattern"]
        Extract["Extract Context:<br/>- Vessel: Handysize<br/>- From: Libya<br/>- To: Black Sea"]
        Check{"Similar pattern<br/>exists?"}
        Reinforce["Increase confidence"]
        Create["Create new pattern"]
        Embed["Generate embedding"]
    end
    
    subgraph Apply["5. Apply in Future"]
        NewReq["New similar request"]
        Find["Find matching pattern"]
        Adjust["Apply adjustment:<br/>P1: -7 points"]
        Show["Show: 'Based on<br/>previous correction...'"]
    end
    
    AI --> Score
    Score --> UI
    UI --> Decision
    
    Decision -->|Approve| Approve
    Decision -->|Correct| Correct
    Decision -->|Skip| Skip
    
    Approve --> Reinforce
    Correct --> Change
    Skip --> Create
    
    Change --> Ask
    Ask --> Explain
    Explain --> Extract
    
    Extract --> Check
    Check -->|Yes| Reinforce
    Check -->|No| Create
    
    Reinforce --> Embed
    Create --> Embed
    
    Embed -.-> Find
    NewReq --> Find
    Find --> Adjust
    Adjust --> Show
    
    style Generate fill:#3498db
    style Review fill:#27ae60
    style Feedback fill:#e67e22
    style Learn fill:#9b59b6
    style Apply fill:#1abc9c
```

## 5. Message Generation Flow

```mermaid
flowchart LR
    subgraph Input["Input Data"]
        SR["Scoring Result"]
        Person["Person Data"]
        History["Message History"]
        Template["Template"]
    end
    
    subgraph Agent["Message Agent"]
        Context["Build Context"]
        Select["Select Template"]
        Personal["Personalization"]
        LLM["Claude Generation"]
    end
    
    subgraph Output["Output"]
        Email["📧 Email"]
        WA["📱 WhatsApp"]
        Preview["👁️ Preview"]
    end
    
    SR --> Context
    Person --> Context
    History --> Context
    
    Context --> Select
    Select --> Template
    Template --> Personal
    
    Personal --> LLM
    LLM --> Email
    LLM --> WA
    
    Email --> Preview
    WA --> Preview
```

## 6. MongoDB Collections Relationship

```mermaid
erDiagram
    scoring_results ||--o{ learning_patterns : "generates"
    scoring_results ||--o{ message_history : "triggers"
    scoring_results }o--|| knowledge_embeddings : "uses"
    
    scoring_results {
        ObjectId _id PK
        int vessel_id
        int cargo_id
        object scores
        int final_score
        string classification
        boolean is_blocked
        object snapshot
        datetime created_at
    }
    
    knowledge_embeddings {
        ObjectId _id PK
        string source_file
        string chunk_text
        array embedding
        object metadata
        datetime created_at
    }
    
    learning_patterns {
        ObjectId _id PK
        string pattern_type
        object context
        string criterion
        object original
        object corrected
        int confidence
        datetime created_at
    }
    
    aliases_auto {
        ObjectId _id PK
        string entity_type
        string original_value
        string normalized_value
        float confidence
        string status
        datetime created_at
    }
    
    message_history {
        ObjectId _id PK
        int vessel_id
        int cargo_id
        string message_type
        string generated_text
        string edited_text
        boolean was_edited
        datetime sent_at
    }
```

## 7. Implementation Timeline

```mermaid
gantt
    title VKChart-AI Implementation Plan
    dateFormat  YYYY-MM-DD
    
    section Phase 1 (Foundation)
    Project Setup           :p1_setup, 2024-12-01, 2d
    Normalization System    :p1_norm, after p1_setup, 2d
    Laravel Integration     :p1_laravel, after p1_norm, 1d
    Scoring Agent (P1,P5,P6,P7) :p1_score, 2024-12-06, 2d
    Message Agent (Basic)   :p1_msg, after p1_score, 2d
    Integration Testing     :p1_test, after p1_msg, 1d
    
    section Phase 2 (Full Scoring)
    Remaining Criteria (P1A,P2,P3,P4) :p2_criteria, 2024-12-13, 2d
    Knowledge Base Enhancement :p2_kb, after p2_criteria, 1d
    LLM Fallback Normalizer :p2_llm, after p2_kb, 1d
    Testing & Tuning        :p2_test, after p2_llm, 1d
    
    section Phase 3 (Learning & Polish)
    Learning System         :p3_learn, 2024-12-20, 2d
    Message Enhancement     :p3_msg, after p3_learn, 1d
    Monitoring & Logging    :p3_mon, after p3_msg, 1d
    Production Prep         :p3_prod, after p3_mon, 1d
    
    section Milestones
    MVP Demo                :milestone, 2024-12-12, 0d
    Full Scoring Demo       :milestone, 2024-12-19, 0d
    Production Launch       :milestone, 2024-12-27, 0d
```

## 8. Trigger System

```mermaid
flowchart TD
    subgraph Triggers["🔔 TRIGGER EVENTS"]
        T1["📦 Cargo Changed<br/>port, quantity, laycan"]
        T2["🚢 Vessel Updated<br/>position, ETA, status"]
        T3["💬 Comments Changed<br/>OpenArea, preferences"]
        T4["⏰ Time Passed<br/>schedule, season"]
        T5["🌍 External Event<br/>geopolitics, weather"]
        T6["👤 Client Response<br/>update, counter-offer"]
    end
    
    subgraph Priority["Priority Classification"]
        Critical["🔴 CRITICAL<br/>Immediate"]
        High["🟠 HIGH<br/>&lt;5 min"]
        Medium["🟡 MEDIUM<br/>&lt;10 min"]
        Low["🟢 LOW<br/>Scheduled"]
    end
    
    subgraph Actions["⚡ ACTIONS"]
        A1["Full Recalc<br/>All P1-P7"]
        A2["Partial Recalc<br/>Affected only"]
        A3["Block Check<br/>P6 validation"]
        A4["Cache Update"]
        A5["New Message"]
    end
    
    T1 --> Critical
    T5 --> Critical
    T6 --> Critical
    T2 --> High
    T3 --> High
    T4 --> Low
    
    Critical --> A1
    High --> A2
    Medium --> A2
    Low --> A4
    
    A1 --> A3
    A2 --> A3
    A3 --> A5
```

## 9. LangGraph & LangSmith Integration

### 9.1 LangGraph Scoring Workflow

```mermaid
flowchart TD
    Start(["Start"]) --> Init["StateInit<br/>📊 Initialize State"]
    
    Init --> Normalize["Normalize<br/>🔄 Port/Region/Cargo"]
    Normalize --> CheckData["CheckData<br/>✔️ Validate Input"]
    
    subgraph ScoringFlow["Scoring Workflow (LangGraph)"]
        CheckData --> P1["P1: Proximity<br/>📍 RAG + Tool"]
        P1 --> P1A["P1A: Patterns<br/>🌍 RAG + LLM"]
        P1A --> P2["P2: Regional<br/>🏳️ Comments + RAG"]
        P2 --> P3["P3: Cargo<br/>📦 Preferences"]
        P3 --> P4["P4: Last Ports<br/>⚓ History"]
        P4 --> P5["P5: Intake<br/>⚖️ Tool Calc"]
        
        P5 --> CheckP5{"Intake >= 90%?"}
        CheckP5 -->|No| BlockP5["Block: Intake"]
        CheckP5 -->|Yes| P6["P6: OpenArea<br/>💬 Current Voyage"]
        
        P6 --> P7["P7: Readiness<br/>⏰ ETA Match"]
        P7 --> CheckP6{"P6 Score > -40?"}
        CheckP6 -->|No| BlockP6["Block: Preferences"]
        CheckP6 -->|Yes| Finalize["Finalize<br/>🎯 Calculate Total"]
    end
    
    BlockP5 --> SaveState1["SaveState<br/>💾 Checkpoint"]
    BlockP6 --> SaveState2["SaveState<br/>💾 Checkpoint"]
    Finalize --> SaveState3["SaveState<br/>💾 Checkpoint"]
    
    SaveState1 --> End(["End"])
    SaveState2 --> End
    SaveState3 --> End
    
    subgraph LangSmith["LangSmith Auto-Tracing"]
        Trace["🔍 Trace All LLM Calls"]
        Log["📊 Log Inputs/Outputs"]
        Eval["🎯 Evaluate Accuracy"]
    end
    
    P1 -.-> Trace
    P1A -.-> Trace
    P2 -.-> Trace
    P6 -.-> Trace
    P7 -.-> Trace
    
    style ScoringFlow fill:#e8f5e9
    style LangSmith fill:#fff3e0
    style CheckP5 fill:#ffebee
    style CheckP6 fill:#ffebee
    style BlockP5 fill:#ef5350,color:#fff
    style BlockP6 fill:#ef5350,color:#fff
```

### 9.2 Human-in-the-Loop with Checkpoints

```mermaid
flowchart LR
    subgraph Original["Original Flow"]
        Start1["Start"] --> Score1["Calculate P1-P7"]
        Score1 --> Save1["SaveState<br/>💾 thread_id: 12345"]
        Save1 --> Show1["Show to User"]
    end
    
    subgraph UserReview["User Review"]
        Show1 --> Decision{"User Action?"}
        Decision -->|Approve| Done["Done ✅"]
        Decision -->|Correct| Correct["User changes<br/>P1: 15 → 8"]
    end
    
    subgraph Resume["Resume from Checkpoint"]
        Correct --> Load["LoadCheckpoint<br/>💾 thread_id: 12345"]
        Load --> Update["UpdateState<br/>🔄 P1 = 8"]
        Update --> Continue["Continue Workflow<br/>▶️ from P1A"]
        Continue --> Recalc["Recalculate Final<br/>📊 with new P1"]
        Recalc --> Show2["Show Updated Result"]
    end
    
    subgraph Learning["Learning Module"]
        Show2 --> LogCorrection["Log Correction<br/>🧠 Pattern"]
        LogCorrection --> Store["Store in<br/>learning_patterns"]
    end
    
    style Original fill:#e3f2fd
    style UserReview fill:#fff3e0
    style Resume fill:#e8f5e9
    style Learning fill:#f3e5f5
```

### 9.3 LangSmith Observability Stack

```mermaid
flowchart TB
    subgraph App["VKChart AI Application"]
        Agent["Scoring Agent"]
        LLM["LLM Calls"]
    end
    
    subgraph LangSmith["LangSmith Platform"]
        subgraph Tracing["Tracing"]
            AutoTrace["✨ Auto-Trace<br/>All LLM Calls"]
            Input["Capture Inputs"]
            Output["Capture Outputs"]
            Latency["⏱️ Measure Latency"]
            Cost["💰 Track Costs"]
        end
        
        subgraph Datasets["Datasets"]
            Create["📝 Create Test Sets"]
            Maintain["🔄 Version Control"]
            Gold["🎯 Gold Standard"]
        end
        
        subgraph Evaluation["Evaluation"]
            Run["🏃 Run Evals"]
            Metrics["📊 Calculate Metrics"]
            Compare["🔀 Compare Versions"]
        end
        
        subgraph Monitoring["Monitoring"]
            Dashboard["📊 Dashboard"]
            Alerts["🔔 Alerts"]
            Analytics["📈 Analytics"]
        end
    end
    
    Agent --> LLM
    LLM --> AutoTrace
    
    AutoTrace --> Input
    AutoTrace --> Output
    AutoTrace --> Latency
    AutoTrace --> Cost
    
    Input --> Create
    Output --> Gold
    
    Create --> Run
    Gold --> Run
    Run --> Metrics
    Metrics --> Compare
    
    Metrics --> Dashboard
    Dashboard --> Alerts
    Dashboard --> Analytics
    
    style LangSmith fill:#fff3e0
    style Tracing fill:#e8f5e9
    style Datasets fill:#e3f2fd
    style Evaluation fill:#f3e5f5
    style Monitoring fill:#ffebee
```

### 9.4 LangServe API Deployment

```mermaid
flowchart LR
    subgraph Client["Clients"]
        Web["🌐 Web App"]
        Mobile["📱 Mobile"]
        CLI["⌨️ CLI"]
    end
    
    subgraph LangServe["LangServe Endpoints"]
        API1["/scoring<br/>🎯 Scoring Workflow"]
        API2["/messages<br/>✉️ Message Gen"]
        Stream["🌊 Streaming<br/>SSE Support"]
        Playground["🎮 Playground UI"]
    end
    
    subgraph Backend["Backend"]
        Workflow["LangGraph<br/>Workflow"]
        Chain["LangChain<br/>Chain"]
    end
    
    Web --> API1
    Mobile --> API1
    CLI --> API2
    
    API1 --> Stream
    API2 --> Stream
    
    API1 --> Workflow
    API2 --> Chain
    
    Playground -.-> API1
    Playground -.-> API2
    
    style LangServe fill:#e3f2fd
    style Backend fill:#e8f5e9
```
