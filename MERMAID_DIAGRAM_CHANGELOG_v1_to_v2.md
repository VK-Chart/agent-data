# 🔄 CHANGELOG: Mermaid System Diagram v1 → v2

**Date**: 2024-12-02  
**Author**: Claude (System Architect)  
**Task**: Block B - Step B3 (Updating architectural diagrams with Lang* blocks)

---

## 📊 SUMMARY OF CHANGES

### Version 1.0 → Version 2.0
- **Old**: LangChain mentioned generically
- **New**: Full Lang* Stack with LangGraph, LangSmith, LangServe explicitly shown
- **Impact**: 5x more detailed AI architecture, clear workflow visualization

---

## 🆕 NEW COMPONENTS ADDED

### 1. **LangGraph Workflow Section** 🔄

**What was added:**
```mermaid
subgraph LangGraphWorkflow["🔄 LANGGRAPH WORKFLOW (State Management)"]
    StateInit["🎬 Initialize State: ScoringState TypedDict"]
    Normalize["⚙️ NODE: Normalize Data"]
    CheckData["✅ Validate Input"]
end
```

**Why it's important:**
- Shows P1-P7 as **nodes** in a graph (not just sequential code)
- Visualizes **conditional routing** (P5 blocking, P6 blocking)
- Demonstrates **state management** (ScoringState TypedDict)
- Illustrates **checkpointing** for human-in-the-loop

---

### 2. **LangSmith Observability Section** 🔍

**What was added:**
```mermaid
subgraph LangSmith["🔍 LANGSMITH OBSERVABILITY"]
    AutoTrace["📊 Automatic Tracing"]
    Evaluation["✅ Evaluation: accuracy ≥85%"]
    Production["🚨 Production Monitor"]
end
```

**Why it's important:**
- **Debugging**: See full trace when score is wrong
- **Testing**: Measure accuracy on 20 test cases
- **Monitoring**: Track latency, errors, cost in production
- **Alerts**: Notify when error >5% or latency >10s

---

### 3. **LangServe API Layer Section** 🚀

**What was added:**
```mermaid
subgraph LangServe["🚀 LANGSERVE API LAYER"]
    APIEndpoints["POST /scoring/invoke, /stream, /batch"]
    StreamProgress["📡 Streaming: Real-time progress"]
end
```

**Why it's important:**
- **Streaming**: UI shows "Calculating P1... P2..." in real-time
- **Playground**: Interactive testing without code
- **Batch**: Process multiple cargo-vessel pairs efficiently

---

### 4. **Human-in-the-Loop Workflow** 👤

**What was added:**
```mermaid
subgraph HumanInLoop["🔄 HUMAN-IN-THE-LOOP [LangGraph]"]
    CorrectScore["User corrects: P1 = 8, not 15"]
    LoadCheckpoint["Load state from PostgreSQL"]
    UpdateState["Update P1 in state"]
    ResumeWorkflow["Resume from P1A, recalc P1A-P7"]
end
```

**Why it's important:**
- **Behavioral Cloning**: System learns from corrections
- **Efficiency**: No need to recalculate P1-P5 if only P1 changed
- **Learning**: Corrections → test cases → improved accuracy

---

### 5. **Enhanced Learning Section** 🧠

**What was changed:**
```diff
- subgraph Learning["🧠 ОБУЧЕНИЕ [MongoDB Atlas]"]
+ subgraph Learning["🧠 ОБУЧЕНИЕ [MongoDB Atlas + LangSmith]"]
    StorePattern["Save correction"]
    VectorSearch["Find similar cases"]
    UpdateWeights["Update weights"]
+   EvaluateChanges["LangSmith: Compare old vs new accuracy"]
```

**Why it's important:**
- **Measurable Improvement**: Track if corrections actually improve accuracy
- **A/B Testing**: Test new prompts against old ones
- **Regression Prevention**: Ensure changes don't break existing good cases

---

## 🔄 UPDATED FLOWS

### Flow 1: Scoring with LangGraph

**Before (v1):**
```
LoadData → P1 → P1A → P2 → ... → P7 → FinalScore
```
(Sequential code, no visualization)

**After (v2):**
```
LoadData → StateInit → Normalize → CheckData →
→ P1 (node) → P1A (node) → ... → P5 (node) →
→ [CONDITIONAL: P5 blocked?] →
  - YES → SaveState → END
  - NO → P6 (node) →
→ [CONDITIONAL: P6 ≤ -40?] →
  - YES → SaveState → END
  - NO → P7 (node) →
→ FinalScore (node) → Reasoning (node) → SaveState → END
```
(Graph with conditional routing, visual diagram)

---

### Flow 2: User Correction (NEW in v2)

**Before (v1):**
```
User corrects → Recalculate everything from scratch
```

**After (v2):**
```
User: "P1 should be 8, not 15" →
→ LoadCheckpoint (get full state) →
→ UpdateState (change P1 = 8) →
→ ResumeWorkflow (from P1A node) →
→ Recalculate only P1A-P7 (not P1) →
→ New final score →
→ Store correction in LangSmith dataset →
→ Add to test cases for regression testing
```

---

### Flow 3: Observability (NEW in v2)

**Before (v1):**
```
Score calculated → No insight into "why"
```

**After (v2):**
```
Score calculated →
→ [LangSmith auto-traces]:
  - P1 node: RAG query, LLM prompt, response
  - Tool calls: estimate_distance(Libya, BlackSea)
  - Retrieved docs: proximity_scoring_matrix.txt
  - Execution time: 2.3s
  - Cost: $0.05
→ User can see full trace in LangSmith UI
→ Debug: "Why P1=15?" → See exact LLM reasoning
```

---

## 📋 UPDATED COMPONENT DETAILS

### Scoring Engine (P1-P7)

**Before (v1):**
```
P1["🗺️ P1: PROXIMITY (0-20)"]
```

**After (v2):**
```
P1["🗺️ NODE P1: PROXIMITY (0-20)
RAG: proximity_scoring_matrix.txt
LLM: Calculate distance score
Tool: estimate_distance()"]
```

**Change**: Explicitly shows that each criterion uses:
- **RAG**: Retrieve relevant knowledge
- **LLM**: Claude/GPT-4 for reasoning
- **Tools**: Python functions for calculations

---

### AI Generation Module

**Before (v1):**
```
subgraph AIGeneration["🤖 МОДУЛЬ ГЕНЕРАЦИИ ИИ [LangChain]"]
```

**After (v2):**
```
subgraph AIGeneration["🤖 МОДУЛЬ ГЕНЕРАЦИИ ИИ [LangChain + LangGraph]"]
    subgraph MessageWorkflow["🔄 MESSAGE GENERATION WORKFLOW"]
        ...
    end
```

**Change**: Shows that message generation can also be a LangGraph workflow (not just simple LLM call)

---

### Monitoring Module

**Before (v1):**
```
Metrics["📈 Метрики:
- Отправлено офферов
- Конверсия
- Время ответа"]
```

**After (v2):**
```
Metrics["📈 Метрики:
- Отправлено офферов
- Конверсия
- Время ответа
- LLM cost/request    ← NEW
- Avg latency         ← NEW"]
```

**Change**: Added AI-specific metrics

---

## 🎨 UPDATED STYLES

### New Color Classes:

```css
langgraphStyle  → Purple (#e1bee7) - Workflow management
langsmithStyle  → Orange (#fff3e0) - Observability  
langserveStyle  → Teal (#e0f2f1)  - API layer
```

**Applied to:**
- `LangGraphWorkflow`, `StateInit`, `CheckP5Block`, `CheckP6Block`, `SaveState`
- `HumanInLoop`, `CorrectScore`, `LoadCheckpoint`, `UpdateState`, `ResumeWorkflow`
- `LangSmith`, `AutoTrace`, `Evaluation`, `Production`
- `LangServe`, `APIEndpoints`, `StreamProgress`
- `Learning`, `StorePattern`, `EvaluateChanges`

---

## 📊 UPDATED DOCUMENTATION SECTIONS

### New Section: Technology Stack

**Added comprehensive 5-layer breakdown:**
1. **Layer 1**: Laravel (Orchestrator)
2. **Layer 2**: RabbitMQ (Message Broker)
3. **Layer 3**: Python AI Service
   - LangChain (Core)
   - LangGraph (Workflows)
   - LangServe (API)
4. **Layer 4**: LangSmith (Observability)
5. **Layer 5**: Data Stores (MongoDB, PostgreSQL, MySQL)

---

### Updated Table: Criteria Scoring (P1-P7)

**Added columns:**
- **LangGraph Node**: Shows node name (e.g., `calculate_p1`)
- **Components**: RAG + LLM + Tool breakdown
- **Triggers**: What events cause recalculation

**Example:**
| Критерий | LangGraph Node | Компоненты | Триггеры |
|----------|----------------|------------|----------|
| P1 | `calculate_p1` | RAG + LLM + Tool(estimate_distance) | T2, T5 |

---

### New Section: LangGraph Execution Flow

**Shows complete workflow:**
```
START → StateInit → Normalize → CheckData →
→ P1 → P1A → P2 → P3 → P4 → P5 →
→ [CHECK: P5 blocked?] → ...
```

**With LangSmith Tracing:**
```
Each node auto-traced:
- Input state
- RAG query + docs
- Tool calls
- LLM prompt + response
- Output state
- Time, cost
```

---

### New Section: Behavioral Cloning Workflow

**6-step process:**
1. User Correction
2. Store in LangSmith Dataset
3. Vector Search (MongoDB)
4. Update Knowledge Base
5. Evaluation (measure improvement)
6. A/B Testing (deploy best variant)

---

### Updated Section: System Settings

**Added AI Configuration:**
```yaml
llm:
  primary: "claude-sonnet-4-20250514"
  backup: "gpt-4-turbo-preview"

langsmith:
  enabled: true
  accuracy_threshold: 0.85

langgraph:
  checkpointer: "postgresql"
  checkpoint_ttl: 604800  # 7 days
```

---

### New Section: Monitoring - Enhanced with LangSmith

**LangSmith Dashboard:**
- Traces: Full execution graph
- Latency: P95 < 10s
- Cost: Per request
- Accuracy: 87% (target: 85%)
- Errors: 2% (target: <5%)

**Alerts:**
- Error rate >5%
- Latency >10s
- Cost >$50/day
- Accuracy <85%

---

## 🔢 STATISTICS

### Diagram Complexity:

| Metric | v1 | v2 | Change |
|--------|----|----|--------|
| **Subgraphs** | 10 | 13 | +3 (LangGraph, LangSmith, LangServe) |
| **Nodes** | ~50 | ~65 | +15 (detailed AI components) |
| **Edges** | ~80 | ~100 | +20 (new flows) |
| **Color Classes** | 4 | 7 | +3 (Lang* specific styles) |
| **Lines of Code** | 445 | ~650 | +46% more detailed |

---

### Documentation Sections:

| Section | v1 | v2 | Change |
|---------|----|----|--------|
| **Main Sections** | 5 | 8 | +3 new sections |
| **Tables** | 2 | 4 | +2 (LangGraph, monitoring) |
| **Code Examples** | 3 | 8 | +5 (workflows, configs) |
| **Scenarios** | 3 | 3 | Same (but updated) |

---

## ✅ VALIDATION CHECKLIST

### Architecture Completeness:

- [x] LangChain components detailed
- [x] LangGraph workflow visualized
- [x] LangSmith observability shown
- [x] LangServe API layer added
- [x] Human-in-the-loop flow illustrated
- [x] Behavioral cloning workflow documented
- [x] Conditional routing (P5, P6) shown
- [x] Checkpointing mechanism explained
- [x] Monitoring enhanced with AI metrics
- [x] All color classes updated
- [x] Technology stack documented
- [x] All scenarios updated with Lang* components

---

## 🎯 KEY IMPROVEMENTS

### 1. **Visibility**
- **Before**: "LangChain handles scoring"
- **After**: See exact P1-P7 nodes, RAG queries, tool calls, conditional routing

### 2. **Debuggability**
- **Before**: "Score is wrong, why?"
- **After**: View full LangSmith trace → see LLM reasoning, retrieved docs, tool results

### 3. **Maintainability**
- **Before**: Imperative code hard to modify
- **After**: LangGraph graph easy to extend (add P8, P9)

### 4. **Learnability**
- **Before**: User corrections lost
- **After**: Corrections → test cases → measured accuracy improvement

### 5. **Reliability**
- **Before**: No regression testing
- **After**: Evaluation framework ensures changes don't break existing good cases

---

## 📝 MIGRATION NOTES

### For Developers:

1. **Old diagram (v1)** can still be used for high-level understanding
2. **New diagram (v2)** is the SOURCE OF TRUTH for implementation
3. **All new features** (LangGraph, LangSmith, LangServe) should follow v2 architecture
4. **Existing code** should be refactored to match v2 when possible

### File Locations:

- **v1**: `Mermaid_FINAL_Complete_System_With_Triggers.md` (keep as reference)
- **v2**: `Mermaid_FINAL_Complete_System_With_Triggers_v2.md` (current)

---

## 🔗 RELATED DOCUMENTS

Should be updated next (B3 continuation):
1. **VKCHART_AI_ARCHITECTURE.md** - Add Lang* Stack details
2. **VKCHART_AI_DIAGRAMS.md** - Add LangGraph workflow diagram
3. **Architecture&RAG.txt** - Update with LangSmith integration

---

**VERSION**: v1 → v2  
**DATE**: 2024-12-02  
**STATUS**: ✅ COMPLETED (Mermaid diagram updated)  
**NEXT**: Update remaining architecture files
