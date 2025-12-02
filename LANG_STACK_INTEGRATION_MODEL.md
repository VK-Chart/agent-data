# 🔗 VK Chart AI - МОДЕЛЬ ИНТЕГРАЦИИ LANG* STACK

**Версия**: 1.0  
**Дата**: 2024-12-02  
**Статус**: BLOCK B - Step B2 COMPLETED  
**Автор**: Claude (System Architect)

---

## 📋 EXECUTIVE SUMMARY

Этот документ определяет **КАК именно** LangChain экосистема будет интегрирована в VK Chart AI:
- Архитектурные паттерны
- Integration diagrams
- Конкретные use cases для каждого продукта
- Implementation guide с примерами кода

---

## 1. ПОЛНАЯ АРХИТЕКТУРА С LANG* STACK

### 1.1 High-Level Integration Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         VK CHART AI SYSTEM                                   │
│                      WITH FULL LANG* STACK                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ LAYER 1: LARAVEL APPLICATION (Orchestrator)                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  UI/Dashboard → Events → Jobs → RabbitMQ Publisher                          │
│                                                                              │
└──────────────────────────────┬──────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ LAYER 2: RABBITMQ (Message Broker)                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  [scoring.requests] [scoring.results] [messages.requests] [learning]        │
│                                                                              │
└──────────────────────────────┬──────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ LAYER 3: PYTHON AI SERVICE (LangChain + LangGraph + LangServe)             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ LANGSERVE (API Layer)                                                  │  │
│  │                                                                        │  │
│  │  FastAPI App with LangServe routes:                                   │  │
│  │  • POST /scoring/invoke                                               │  │
│  │  • POST /scoring/stream         ← Streaming progress                  │  │
│  │  • POST /scoring/batch                                                │  │
│  │  • GET  /scoring/playground     ← Interactive testing                 │  │
│  │                                                                        │  │
│  └────────────────────────┬──────────────────────────────────────────────┘  │
│                           │                                                  │
│                           ▼                                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ LANGGRAPH (Workflow Orchestration)                                    │  │
│  │                                                                        │  │
│  │  ScoringWorkflow = StateGraph(ScoringState)                           │  │
│  │                                                                        │  │
│  │   START → [Normalize] → [P1] → [P1A] → [P2] → [P3]                  │  │
│  │              ↓           ↓                                            │  │
│  │         [Check Data]  [RAG]                                           │  │
│  │                           ↓                                            │  │
│  │   [P4] → [P5] → [Check Intake] → [P6] → [Check Block]               │  │
│  │           ↓          ↓               ↓         ↓                      │  │
│  │        [Tool]   [BLOCKED?]      [Parse]   [P6≤-40?]                 │  │
│  │                     ↓               ↓         ↓                      │  │
│  │                   [END]         [P7] → [Final Score] → [END]         │  │
│  │                                   ↓                                    │  │
│  │                             [Checkpointing]                           │  │
│  │                                   ↓                                    │  │
│  │                          [Human-in-the-loop]                          │  │
│  │                                                                        │  │
│  └────────────────────────┬──────────────────────────────────────────────┘  │
│                           │                                                  │
│                           ▼                                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ LANGCHAIN (Core Components)                                           │  │
│  │                                                                        │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │  │
│  │  │  Agents     │  │    RAG      │  │   Tools     │                  │  │
│  │  │             │  │             │  │             │                  │  │
│  │  │ • Scoring   │  │ • MongoDB   │  │ • intake    │                  │  │
│  │  │ • Message   │  │   Vector    │  │ • distance  │                  │  │
│  │  │ • Normalize │  │   Search    │  │ • comments  │                  │  │
│  │  │             │  │ • Retriever │  │ • eta       │                  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │  │
│  │                                                                        │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │  │
│  │  │  LLMs       │  │ Embeddings  │  │  Splitters  │                  │  │
│  │  │             │  │             │  │             │                  │  │
│  │  │ • Claude    │  │ • OpenAI    │  │ • Recursive │                  │  │
│  │  │ • OpenAI    │  │   Ada-002   │  │ • Character │                  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │  │
│  │                                                                        │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└──────────────────────────────┬───────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ LAYER 4: LANGSMITH (Observability & Monitoring)                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐   │
│  │  Automatic Tracing │  │     Datasets       │  │    Evaluation      │   │
│  │                    │  │                    │  │                    │   │
│  │  • LLM calls       │  │  • Test cases      │  │  • Accuracy        │   │
│  │  • Tool calls      │  │  • Ground truth    │  │  • Latency         │   │
│  │  • RAG queries     │  │  • Edge cases      │  │  • Cost            │   │
│  │  • Errors          │  │                    │  │                    │   │
│  └────────────────────┘  └────────────────────┘  └────────────────────┘   │
│                                                                              │
│  ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐   │
│  │  Production        │  │    Debugging       │  │   Analytics        │   │
│  │  Monitoring        │  │                    │  │                    │   │
│  │                    │  │  • Trace viewer    │  │  • Cost tracking   │   │
│  │  • Requests/min    │  │  • Replay          │  │  • Token usage     │   │
│  │  • Avg latency     │  │  • Breakpoints     │  │  • Model usage     │   │
│  │  • Error rate      │  │  • Comparisons     │  │  • Trends          │   │
│  └────────────────────┘  └────────────────────┘  └────────────────────┘   │
│                                                                              │
└──────────────────────────────┬───────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ LAYER 5: DATA STORES                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │  MongoDB Atlas   │  │  PostgreSQL      │  │    MySQL         │          │
│  │                  │  │  (Checkpoints)   │  │    (Laravel)     │          │
│  │  • Vectors       │  │                  │  │                  │          │
│  │  • Scoring       │  │  • LangGraph     │  │  • Vessels       │          │
│  │  • Learning      │  │    states        │  │  • Cargos        │          │
│  │  • Messages      │  │  • Resume data   │  │  • Companies     │          │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. ДЕТАЛИЗАЦИЯ ПО КАЖДОМУ ПРОДУКТУ

### 2.1 LangChain - Core Framework

**Роль:** Базовые компоненты для работы с LLMs

**Используется для:**

#### A. Agents
```python
from langchain.agents import AgentExecutor, create_structured_chat_agent
from langchain.prompts import ChatPromptTemplate

class ScoringAgent:
    """Базовый агент для скоринга (до LangGraph)"""
    
    def __init__(self, llm, tools):
        self.agent = create_structured_chat_agent(
            llm=llm,
            tools=tools,
            prompt=self._build_prompt()
        )
        self.executor = AgentExecutor(
            agent=self.agent,
            tools=tools,
            verbose=True
        )
```

#### B. RAG (Retrieval)
```python
from langchain_mongodb import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings

class KnowledgeRetriever:
    """RAG для базы знаний"""
    
    def __init__(self, mongo_collection):
        self.embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
        self.vector_store = MongoDBAtlasVectorSearch(
            collection=mongo_collection,
            embedding=self.embeddings,
            index_name="vector_index"
        )
    
    async def retrieve_for_criterion(self, criterion: str, context: str):
        """Retrieve knowledge for P1-P7"""
        return await self.vector_store.asimilarity_search(
            query=context,
            k=5,
            pre_filter={"metadata.criterion": criterion}
        )
```

#### C. Tools
```python
from langchain.tools import tool

@tool
def calculate_intake(
    vessel_dwt: int,
    port_draft_limit: float,
    cargo_stowage_factor: float
) -> dict:
    """Calculate max cargo intake"""
    # Implementation
    return {"intake_weight": ..., "limiting_factor": ...}

@tool
def estimate_distance(port_from: str, port_to: str) -> dict:
    """Estimate sea distance"""
    # Implementation
    return {"distance_nm": ..., "estimated_days": ...}

@tool
def parse_comments(comments: str, comment_type: str) -> dict:
    """Parse structured data from comments"""
    # Implementation
    return {"preferences": [...], "technical_data": {...}}
```

#### D. LLM Integration
```python
from langchain_anthropic import ChatAnthropic
from langchain_openai import ChatOpenAI

# Primary LLM
claude = ChatAnthropic(
    model="claude-sonnet-4-20250514",
    temperature=0.7,
    max_tokens=2000
)

# Backup LLM
gpt4 = ChatOpenAI(
    model="gpt-4-turbo-preview",
    temperature=0.7
)
```

---

### 2.2 LangGraph - State Management & Workflows

**Роль:** Управление state и сложными multi-step workflows

**Используется для:**

#### A. Scoring Workflow

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.postgres import PostgresSaver
from typing import TypedDict, Annotated
from operator import add

class ScoringState(TypedDict):
    """State для scoring workflow"""
    
    # Input
    vessel: dict
    cargo: dict
    company: dict
    
    # Intermediate state
    normalized_data: dict
    knowledge_context: dict
    
    # Scores
    scores: Annotated[dict, add]  # P1-P7 scores
    
    # Blocking
    is_blocked: bool
    block_reason: str
    
    # Output
    final_score: int
    reasoning: str
    strengths: list[str]
    weaknesses: list[str]
    
    # Metadata
    processing_steps: list[str]
    errors: list[str]


# Define nodes
def normalize_data(state: ScoringState) -> ScoringState:
    """Step 1: Normalize vessel/cargo data"""
    normalizer = PortNormalizer(...)
    state["normalized_data"] = {
        "open_port": await normalizer.normalize(state["vessel"]["open_area"]),
        "load_port": await normalizer.normalize(state["cargo"]["load_port"]),
        # ...
    }
    state["processing_steps"].append("normalize_data")
    return state


def calculate_p1_proximity(state: ScoringState) -> ScoringState:
    """Step 2: Calculate P1 Proximity"""
    
    # RAG: Get proximity rules
    retriever = KnowledgeRetriever(...)
    knowledge = await retriever.retrieve_for_criterion(
        "P1",
        f"proximity {state['vessel']['open_area']} to {state['cargo']['load_port']}"
    )
    
    # LLM: Calculate score
    prompt = f"""
Based on this knowledge:
{knowledge}

Calculate P1 Proximity score (0-20) for:
Vessel: {state["vessel"]["name"]} at {state["normalized_data"]["open_port"]}
Cargo: Loading at {state["normalized_data"]["load_port"]}

Return JSON: {{"score": ..., "reasoning": "..."}}
"""
    
    result = await claude.ainvoke(prompt)
    parsed = parse_json(result.content)
    
    state["scores"]["p1"] = {
        "score": parsed["score"],
        "max": 20,
        "reasoning": parsed["reasoning"]
    }
    state["processing_steps"].append("calculate_p1")
    return state


def calculate_p5_intake(state: ScoringState) -> ScoringState:
    """Step 3: Calculate P5 Intake"""
    
    # Use tool
    intake_result = calculate_intake_tool(
        vessel_dwt=state["vessel"]["dwt"],
        port_draft_limit=get_port_draft(state["cargo"]["load_port"]),
        cargo_stowage_factor=state["cargo"]["stowage_factor"]
    )
    
    # Check blocking condition
    if intake_result["intake_final"] < 0.9 * state["cargo"]["quantity"]:
        state["is_blocked"] = True
        state["block_reason"] = "Insufficient intake capacity"
        state["processing_steps"].append("calculate_p5_BLOCKED")
        return state
    
    # Calculate score based on intake percentage
    intake_pct = (intake_result["intake_final"] / state["cargo"]["quantity"]) * 100
    if intake_pct >= 100:
        score = 15
    elif intake_pct >= 95:
        score = 12
    else:
        score = 8
    
    state["scores"]["p5"] = {
        "score": score,
        "max": 15,
        "reasoning": f"Intake {intake_result['intake_final']}t vs cargo {state['cargo']['quantity']}t ({intake_pct:.1f}%)"
    }
    state["processing_steps"].append("calculate_p5")
    return state


def calculate_p6_openarea(state: ScoringState) -> ScoringState:
    """Step 4: Calculate P6 OpenArea Comments"""
    
    # Parse comments
    parsed = parse_comments_tool(
        state["vessel"]["comments"],
        comment_type="open_area"
    )
    
    # RAG: Get scoring rules
    retriever = KnowledgeRetriever(...)
    rules = await retriever.retrieve_for_criterion("P6", "open area comments scoring")
    
    # LLM: Evaluate alignment
    prompt = f"""
Based on these rules:
{rules}

Vessel comments: {state["vessel"]["comments"]}
Cargo: {state["cargo"]["type"]} from {state["cargo"]["load_region"]}

Score P6 OpenArea (-50 to +25): {{"score": ..., "reasoning": "..."}}
"""
    
    result = await claude.ainvoke(prompt)
    parsed_result = parse_json(result.content)
    
    state["scores"]["p6"] = {
        "score": parsed_result["score"],
        "range": (-50, 25),
        "reasoning": parsed_result["reasoning"]
    }
    
    # Check blocking condition
    if parsed_result["score"] <= -40:
        state["is_blocked"] = True
        state["block_reason"] = "Owner explicitly avoids this cargo/region (P6 ≤ -40)"
    
    state["processing_steps"].append("calculate_p6")
    return state


def calculate_final_score(state: ScoringState) -> ScoringState:
    """Final step: Calculate total score"""
    
    # Base score (P1, P1A, P2, P3, P4, P5, P7)
    base = sum([
        state["scores"]["p1"]["score"],
        state["scores"]["p1a"]["score"],
        state["scores"]["p2"]["score"],
        state["scores"]["p3"]["score"],
        state["scores"]["p4"]["score"],
        state["scores"]["p5"]["score"],
        state["scores"]["p7"]["score"]
    ])
    
    # Modifier (P6)
    modifier = state["scores"]["p6"]["score"]
    
    # Final score (clamped 0-100)
    state["final_score"] = max(0, min(100, base + modifier))
    
    # Classification
    if state["final_score"] >= 85:
        classification = "excellent"
    elif state["final_score"] >= 70:
        classification = "good"
    elif state["final_score"] >= 60:
        classification = "acceptable"
    elif state["final_score"] >= 50:
        classification = "marginal"
    else:
        classification = "poor"
    
    state["classification"] = classification
    state["processing_steps"].append("calculate_final_score")
    return state


# Conditional routing
def should_continue_after_p5(state: ScoringState) -> str:
    """Conditional edge after P5"""
    if state["is_blocked"]:
        return "blocked"
    return "continue"


def should_continue_after_p6(state: ScoringState) -> str:
    """Conditional edge after P6"""
    if state["is_blocked"]:
        return "blocked"
    return "continue"


# Build the graph
workflow = StateGraph(ScoringState)

# Add nodes
workflow.add_node("normalize", normalize_data)
workflow.add_node("p1", calculate_p1_proximity)
workflow.add_node("p1a", calculate_p1a_patterns)
workflow.add_node("p2", calculate_p2_regional)
workflow.add_node("p3", calculate_p3_cargo)
workflow.add_node("p4", calculate_p4_lastports)
workflow.add_node("p5", calculate_p5_intake)
workflow.add_node("p6", calculate_p6_openarea)
workflow.add_node("p7", calculate_p7_readiness)
workflow.add_node("final", calculate_final_score)

# Add edges
workflow.set_entry_point("normalize")
workflow.add_edge("normalize", "p1")
workflow.add_edge("p1", "p1a")
workflow.add_edge("p1a", "p2")
workflow.add_edge("p2", "p3")
workflow.add_edge("p3", "p4")
workflow.add_edge("p4", "p5")

# Conditional edges
workflow.add_conditional_edges(
    "p5",
    should_continue_after_p5,
    {
        "continue": "p6",
        "blocked": END
    }
)

workflow.add_conditional_edges(
    "p6",
    should_continue_after_p6,
    {
        "continue": "p7",
        "blocked": END
    }
)

workflow.add_edge("p7", "final")
workflow.add_edge("final", END)

# Compile with checkpointing
checkpointer = PostgresSaver.from_conn_string("postgresql://...")
app = workflow.compile(checkpointer=checkpointer)
```

#### B. Human-in-the-Loop Corrections

```python
# Execute workflow with checkpointing
config = {"configurable": {"thread_id": f"scoring_{vessel_id}_{cargo_id}"}}

result = await app.ainvoke(initial_state, config=config)

# User makes correction
# "P1 should be 8, not 15 - Libya to Black Sea not profitable for Handy"

# Get current state
current_state = await app.aget_state(config)

# Update P1 score
current_state.values["scores"]["p1"]["score"] = 8
current_state.values["scores"]["p1"]["reasoning"] = "Libya to Black Sea not profitable for Handysize"

# Resume from P1A node (recalculate downstream)
new_result = await app.ainvoke(
    None,  # Resume from checkpoint
    config={
        **config,
        "configurable": {
            **config["configurable"],
            "checkpoint_id": current_state.config["configurable"]["checkpoint_id"]
        }
    }
)
```

#### C. Visualization

```python
# Generate Mermaid diagram
from langgraph.graph import draw_mermaid

mermaid_code = draw_mermaid(app)
# → Автоматически создает Mermaid диаграмму workflow

# Save to file
with open("scoring_workflow.mmd", "w") as f:
    f.write(mermaid_code)
```

---

### 2.3 LangServe - API Deployment

**Роль:** Быстрое создание production-ready API

**Используется для:**

#### A. Deploy LangGraph Workflow as API

```python
from fastapi import FastAPI
from langserve import add_routes

app = FastAPI(
    title="VK Chart AI Scoring API",
    version="1.0.0",
    description="Vessel-cargo scoring with LangChain"
)

# Add LangGraph workflow as API endpoint
add_routes(
    app,
    app=scoring_workflow,  # The compiled LangGraph app
    path="/scoring",
    enabled_endpoints=["invoke", "stream", "batch"],
    playground_type="default"
)

# Automatically creates:
# POST /scoring/invoke       - Single request
# POST /scoring/stream       - Streaming response
# POST /scoring/batch        - Batch processing
# GET  /scoring/playground   - Interactive playground
```

#### B. Streaming Progress

```python
# Client-side (Laravel/React)
const response = await fetch("/scoring/stream", {
    method: "POST",
    headers: {"Content-Type": "application/json"},
    body: JSON.stringify({
        input: {vessel: ..., cargo: ...},
        config: {configurable: {thread_id: "..."}}
    })
});

const reader = response.body.getReader();
while (true) {
    const {done, value} = await reader.read();
    if (done) break;
    
    // Parse streaming events
    const event = JSON.parse(value);
    
    if (event.event === "on_chain_stream") {
        // Update UI with progress
        updateProgress(event.data.chunk.processing_steps);
    }
}

// UI shows:
// ✓ Normalizing data...
// ✓ Calculating P1 Proximity... (15/20)
// ✓ Calculating P1A Patterns... (12/15)
// ✓ Calculating P2 Regional... (10/15)
// ...
```

#### C. Interactive Playground

```python
# Just navigate to:
# http://localhost:8000/scoring/playground

# Interactive UI automatically generated:
# - Input form for ScoringState
# - Execute button
# - Response viewer
# - Streaming progress
# - Error display
```

---

### 2.4 LangSmith - Observability & Monitoring

**Роль:** Debugging, testing, evaluation, monitoring

**Используется для:**

#### A. Automatic Tracing

```python
import os

# Setup (one-time)
os.environ["LANGSMITH_TRACING"] = "true"
os.environ["LANGSMITH_API_KEY"] = "lsv2_pt_..."
os.environ["LANGSMITH_PROJECT"] = "vkchart-ai-scoring"

# That's it! All LangChain/LangGraph calls are now traced
```

**What gets traced automatically:**
- Every LLM call (prompt, response, tokens, cost)
- Every tool call (input, output, duration)
- Every RAG query (query, retrieved docs, relevance)
- Errors and exceptions
- Full execution graph

#### B. Datasets & Evaluation

```python
from langsmith import Client

client = Client()

# Create dataset
dataset = client.create_dataset(
    dataset_name="vkchart-scoring-test-cases",
    description="20 test cases for scoring accuracy"
)

# Add test cases
test_cases = [
    {
        "input": {
            "vessel": {"name": "TEST VESSEL 1", "dwt": 55000, ...},
            "cargo": {"type": "grain", "quantity": 50000, ...}
        },
        "expected_output": {
            "final_score": 78,
            "scores": {"p1": 15, "p2": 10, ...}
        }
    },
    # ... 19 more cases
]

for case in test_cases:
    client.create_example(
        dataset_id=dataset.id,
        inputs=case["input"],
        outputs=case["expected_output"]
    )
```

```python
from langsmith.evaluation import evaluate

# Define evaluator
def scoring_accuracy(run, example):
    """Check if score is within ±5 of expected"""
    actual = run.outputs["final_score"]
    expected = example.outputs["expected_output"]["final_score"]
    
    return {
        "key": "accuracy",
        "score": abs(actual - expected) <= 5
    }

def p1_accuracy(run, example):
    """Check P1 specifically"""
    actual = run.outputs["scores"]["p1"]["score"]
    expected = example.outputs["expected_output"]["scores"]["p1"]
    
    return {
        "key": "p1_accuracy",
        "score": abs(actual - expected) <= 2
    }

# Run evaluation
results = evaluate(
    lambda inputs: scoring_workflow.invoke(inputs),
    data="vkchart-scoring-test-cases",
    evaluators=[scoring_accuracy, p1_accuracy],
    experiment_prefix="scoring-v1"
)

# Results:
# Accuracy: 87% (17/20 within ±5)
# P1 Accuracy: 90% (18/20 within ±2)
```

#### C. Production Monitoring

```python
# Dashboard automatically shows:
# - Requests per minute
# - Average latency
# - Error rate
# - Cost per request
# - Token usage trends

# Alerts setup
client.create_alert(
    name="High Error Rate",
    condition="error_rate > 0.05",  # >5% errors
    notification_channels=["email", "slack"]
)

client.create_alert(
    name="High Latency",
    condition="p95_latency > 10000",  # >10 seconds
    notification_channels=["email"]
)
```

#### D. Debugging Workflow

```python
# Scenario: P1 score is wrong (15 instead of expected 8)

# 1. Find the trace in LangSmith UI
# 2. View full execution graph:
#    normalize → p1 → p1a → p2 → ...
# 3. Click on "p1" node
# 4. See:
#    - Input state
#    - RAG query: "proximity Libya to Black Sea"
#    - Retrieved documents (with relevance scores)
#    - LLM prompt sent to Claude
#    - LLM response: {"score": 15, "reasoning": "..."}
#    - Output state
# 5. Identify issue:
#    - RAG didn't retrieve document about "Handy not profitable Libya→BlackSea"
#    - Need to add this to knowledge base or adjust embedding

# 6. Fix and re-run evaluation
```

---

## 3. АРХИТЕКТУРНЫЕ ПАТТЕРНЫ

### 3.1 Pattern: Sequential Workflow with Conditional Routing

**Use Case:** Scoring P1-P7 с блокировками

```python
# LangGraph pattern
workflow = StateGraph(State)
workflow.add_node("step1", func1)
workflow.add_node("step2", func2)
workflow.add_conditional_edges(
    "step2",
    condition_func,
    {"continue": "step3", "stop": END}
)
```

**Преимущества:**
- Визуальная диаграмма flow
- Легко добавить новые шаги
- Conditional routing declarative

---

### 3.2 Pattern: Human-in-the-Loop Corrections

**Use Case:** Пользователь исправляет скоринг

```python
# Execute with checkpointing
result = await app.ainvoke(input, config={"configurable": {"thread_id": id}})

# User corrects
state = await app.aget_state(config)
state.values["scores"]["p1"] = corrected_value

# Resume workflow
new_result = await app.ainvoke(None, config=resume_config)
```

**Преимущества:**
- Не нужно пересчитывать всё с нуля
- State сохраняется автоматически
- Можно возобновить с любого checkpoint

---

### 3.3 Pattern: RAG-Enhanced Agent

**Use Case:** Каждый P1-P7 критерий использует RAG

```python
def calculate_criterion(state: State) -> State:
    # 1. RAG retrieval
    knowledge = await retriever.retrieve(criterion, context)
    
    # 2. LLM с knowledge context
    prompt = f"Based on: {knowledge}\nCalculate: {criterion}"
    result = await llm.ainvoke(prompt)
    
    # 3. Update state
    state["scores"][criterion] = parse(result)
    return state
```

**Преимущества:**
- Каждый критерий использует relevant knowledge
- LLM не hallucinate правила
- Можно обновлять knowledge base без изменения кода

---

### 3.4 Pattern: Streaming Progress

**Use Case:** UI показывает прогресс скоринга в реальном времени

```python
# Server: LangServe automatic streaming
add_routes(app, workflow, path="/scoring", enabled_endpoints=["stream"])

# Client: Consume stream
async for chunk in stream_response:
    if chunk["event"] == "on_chain_stream":
        step = chunk["data"]["chunk"]["processing_steps"][-1]
        update_ui(step)  # "Calculating P1...", "Calculating P2...", etc.
```

**Преимущества:**
- Better UX (пользователь видит progress)
- Можно отменить long-running requests
- Легко debug (видно где застряло)

---

### 3.5 Pattern: Evaluation Loop

**Use Case:** Regression testing после изменений

```python
# 1. Create dataset with 20 test cases
# 2. Run evaluation
results = evaluate(workflow, data="test-cases", evaluators=[...])

# 3. Make changes to prompts/knowledge
# 4. Re-run evaluation → compare results
# 5. If accuracy improved → deploy
```

**Преимущества:**
- Prevent regression
- Measure progress (accuracy 80% → 87%)
- A/B testing prompts

---

## 4. IMPLEMENTATION GUIDE

### 4.1 Phase 1: LangSmith Setup (Week 2-3)

**Duration:** 2-4 hours

#### Step 1: Sign up & Configure

```bash
# 1. Sign up at langsmith.com
# 2. Create API key
# 3. Add to .env

LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_pt_xxx
LANGSMITH_PROJECT=vkchart-ai-scoring
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
```

#### Step 2: Verify Tracing

```python
# Test script
import os
from langchain_anthropic import ChatAnthropic

os.environ["LANGSMITH_TRACING"] = "true"
os.environ["LANGSMITH_API_KEY"] = "..."

llm = ChatAnthropic(model="claude-sonnet-4-20250514")
result = llm.invoke("Test message")

# Check LangSmith UI → Should see trace
```

#### Step 3: Create Dataset

```python
from langsmith import Client

client = Client()
dataset = client.create_dataset("vkchart-scoring-test-cases")

# Add 20 test cases
# Can import from CSV or add programmatically
```

#### Step 4: Setup Evaluation

```python
def scoring_accuracy(run, example):
    actual = run.outputs["final_score"]
    expected = example.outputs["final_score"]
    return {"score": abs(actual - expected) <= 5}

results = evaluate(
    scoring_agent,
    data="vkchart-scoring-test-cases",
    evaluators=[scoring_accuracy]
)
```

#### Step 5: Production Monitoring

```python
# Setup alerts in LangSmith UI:
# - Error rate > 5%
# - P95 latency > 10s
# - Cost per day > $50
```

---

### 4.2 Phase 2: LangGraph Refactoring (Week 4)

**Duration:** 1-2 days

#### Step 1: Design State Schema

```python
from typing import TypedDict, Annotated
from operator import add

class ScoringState(TypedDict):
    # Input
    vessel: dict
    cargo: dict
    company: dict
    
    # Scores (accumulated)
    scores: Annotated[dict, add]
    
    # Blocking
    is_blocked: bool
    block_reason: str
    
    # Output
    final_score: int
    reasoning: str
```

#### Step 2: Convert Each Criterion to Node

```python
def calculate_p1(state: ScoringState) -> ScoringState:
    # Existing logic from ScoringAgent
    # Move into node function
    state["scores"]["p1"] = calculate_p1_logic(state)
    return state
```

#### Step 3: Build Graph

```python
from langgraph.graph import StateGraph, END

workflow = StateGraph(ScoringState)
workflow.add_node("p1", calculate_p1)
workflow.add_node("p2", calculate_p2)
# ...
workflow.add_edge("p1", "p2")
# ...
app = workflow.compile()
```

#### Step 4: Add Checkpointing

```python
from langgraph.checkpoint.postgres import PostgresSaver

checkpointer = PostgresSaver.from_conn_string(
    "postgresql://user:pass@localhost/vkchart"
)
app = workflow.compile(checkpointer=checkpointer)
```

#### Step 5: Testing

```python
# Test with 20 cases
for case in test_cases:
    result = await app.ainvoke(case["input"])
    assert result["final_score"] == case["expected_output"]
```

---

### 4.3 Phase 3: LangServe Integration (Optional)

**Duration:** 2-4 hours

#### Step 1: Add LangServe

```bash
pip install langserve[all]
```

#### Step 2: Update FastAPI

```python
from langserve import add_routes

add_routes(
    app,
    app=scoring_workflow,
    path="/scoring"
)
```

#### Step 3: Test Endpoints

```bash
# Invoke
curl -X POST http://localhost:8000/scoring/invoke \
  -H "Content-Type: application/json" \
  -d '{"input": {...}}'

# Stream
curl -X POST http://localhost:8000/scoring/stream \
  -H "Content-Type: application/json" \
  -d '{"input": {...}}'
```

#### Step 4: Update Laravel Client

```php
// Support streaming
$response = Http::asStream()
    ->post('http://ai-service:8000/scoring/stream', $data);

foreach ($response as $chunk) {
    event(new ScoringProgressEvent($chunk));
}
```

---

## 5. BENEFITS SUMMARY

### 5.1 LangSmith Benefits

| Benefit | Impact | Example |
|---------|--------|---------|
| **Debugging** | HIGH | "Why P1=15 not 8?" → See full trace, LLM prompt, RAG results |
| **Evaluation** | HIGH | Test 20 cases → Measure accuracy 87% → Track improvement |
| **Cost Tracking** | MEDIUM | See daily Claude API cost → Optimize expensive calls |
| **Production Monitoring** | HIGH | Alert if error rate >5% → Catch issues before users |

### 5.2 LangGraph Benefits

| Benefit | Impact | Example |
|---------|--------|---------|
| **Visualization** | HIGH | See P1→P2→P3 flow as diagram → Easy to understand |
| **Maintainability** | HIGH | Add P8 criterion → Just add new node, no refactor |
| **Human-in-loop** | HIGH | User corrects P1 → Resume from checkpoint, recalc P2-P7 |
| **Conditional Logic** | MEDIUM | If P6≤-40 → Skip P7 → Declarative, not imperative |

### 5.3 LangServe Benefits

| Benefit | Impact | Example |
|---------|--------|---------|
| **Playground** | MEDIUM | Test API without writing client code → Faster dev |
| **Streaming** | MEDIUM | UI shows "Calculating P1... P2..." → Better UX |
| **Auto Docs** | LOW | OpenAPI spec generated → API docs for free |

---

## 6. NEXT STEPS

1. ✅ **Виталий review этот документ** (LANG_STACK_INTEGRATION_MODEL.md)
2. ✅ **Одобрить план интеграции**
3. 🚀 **Начать Phase 1**: LangSmith setup (2-4 hours)
4. 🔄 **Перейти к B3**: Обновление архитектурных схем

---

## APPENDIX: Сравнение Current vs Proposed

### Current Architecture (Without LangGraph/LangSmith)

```python
# Императивный код
async def score(snapshot):
    scores = {}
    
    # P1
    p1_result = await calculate_p1(snapshot)
    scores["p1"] = p1_result
    
    # P5
    if intake < threshold:
        return {"blocked": True}
    scores["p5"] = p5_result
    
    # P6
    p6 = await calculate_p6(snapshot)
    if p6 <= -40:
        return {"blocked": True}
    
    # ... etc
    return {"scores": scores}
```

**Проблемы:**
- ❌ Нет visualization
- ❌ Нет checkpointing
- ❌ Трудно debug (verbose logs)
- ❌ Нет evaluation framework
- ❌ Нет production monitoring

---

### Proposed Architecture (With LangGraph/LangSmith)

```python
# Декларативный граф + automatic tracing
workflow = StateGraph(ScoringState)
workflow.add_node("p1", calculate_p1)
workflow.add_node("p5", calculate_p5)
workflow.add_conditional_edges("p5", should_continue, {...})
app = workflow.compile(checkpointer=checkpointer)

# Automatic LangSmith tracing
result = await app.ainvoke(input, config={"thread_id": "..."})
```

**Преимущества:**
- ✅ Visual diagram автоматически
- ✅ Checkpointing встроен
- ✅ Full trace в LangSmith UI
- ✅ Evaluation framework ready
- ✅ Production monitoring out-of-box

---

**STATUS:** ✅ BLOCK B - STEP B2 COMPLETED  
**NEXT:** B3 - Обновление архитектурных схем с Lang* блоками
