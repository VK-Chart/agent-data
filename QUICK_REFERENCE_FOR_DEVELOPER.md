# VK CHART AI - QUICK REFERENCE FOR DEVELOPER

**Для:** Oleksiy  
**Версия:** 1.0 - Quick Start

---

## 🎯 ЧТО СТРОИМ?

**Задача:** AI система для автоматического скоринга судов и генерации офферов.

**Архитектура:**
```
React Frontend → Laravel Backend → RabbitMQ → Python AI Service → MongoDB Atlas
```

---

## 📚 MUST-READ ДОКУМЕНТЫ

**Начни с этих файлов:**

1. **DEVELOPER_IMPLEMENTATION_GUIDE.md** ← ГЛАВНЫЙ документ (полная инструкция)
2. **VKCHART_AI_ARCHITECTURE.md** - архитектура системы
3. **MASTER_SCORING_SYSTEM.txt** - single source of truth для scoring
4. **LANG_STACK_INTEGRATION_MODEL.md** - как работает AI часть

---

## 🔧 TECH STACK

### Backend:
- **Laravel 11** - основной backend
- **MySQL** - relational data
- **RabbitMQ** - async queues
- **Redis** - caching

### AI Microservice (Python):
- **FastAPI** - REST API
- **LangChain** - AI chains
- **LangGraph** - state management
- **LangSmith** - observability
- **MongoDB Atlas** - vector search + knowledge bases

---

## 📋 SCORING CRITERIA (P1-P7)

| Criterion | Max Points | Description |
|-----------|------------|-------------|
| **P1** | 20 | Proximity (distance to loading port) |
| **P1A** | 15 | Regional Patterns (market logic) |
| **P2** | 15 | Regional Preferences (manual settings) |
| **P3** | 15 | Cargo Preferences |
| **P4** | 10 | Last Ports (familiarity) |
| **P5** | 15 | Intake Capacity |
| **P6** | -50 to +25 | OpenArea Comments (modifier) |
| **P7** | 10 | Readiness/ETA |

**Final Score = (P1 + P1A + P2 + P3 + P4 + P5 + P7) + P6**  
Clamped to [0, 100]

---

## 🚀 IMPLEMENTATION PHASES

### Phase 1: Foundation (Week 1-2)
- ✅ Set up Laravel scoring service
- ✅ Set up Python FastAPI
- ✅ Connect Laravel → Python → MongoDB
- ✅ Test basic communication

### Phase 2: P1 Scoring (Week 3)
- ✅ Implement proximity calculation
- ✅ Test with real data

### Phase 3: RAG + P1A (Week 4)
- ✅ Set up MongoDB vector search
- ✅ Implement P1A with RAG

### Phase 4: Comments Parsing (Week 5-6)
- ✅ P2, P3, P6 with AI parsing

### Phase 5: Remaining Scoring (Week 7)
- ✅ P4, P5, P7

### Phase 6: Message Generation (Week 8)
- ✅ AI offer text generation

### Phase 7: Learning (Week 9-10)
- ✅ User corrections + behavioral cloning

### Phase 8: Testing (Week 11-12)
- ✅ Production ready

---

## 💻 QUICK CODE EXAMPLES

### Laravel: Queue Scoring Request

```php
use App\Services\ScoringService;

$service = app(ScoringService::class);
$service->queueScoring($vessel, $cargo);
```

### Laravel: Get Score

```php
$score = VesselCargoScore::where('vessel_id', $vesselId)
    ->where('cargo_id', $cargoId)
    ->latest('scored_at')
    ->first();
```

### Python: Scoring Endpoint

```python
@app.post("/api/v1/score")
async def score_vessel_cargo(request: ScoringRequest):
    result = await scoring_agent.score(request)
    return result
```

### Python: RAG Query

```python
patterns = await vector_store.similarity_search(
    collection="regional_trade_patterns",
    query="Coaster Black Sea to Marmara grain",
    k=5
)
```

---

## 🗂️ FILE LOCATIONS

### Rules (алгоритмы scoring):
```
COMMENTS_PROCESSING_RULES.txt
COMMENTS_PROCESSING_PROMPTS.txt
P1A_REGIONAL_SCORING_RULES.txt
READINESSETA_TO_LOADING_PORT_SCORING.txt (P7)
OPEN_AREA_COMMENTS_SCORING.txt (P6)
vessel_location_determination.txt
intake_calculator_formula.txt
```

### Knowledge Bases (данные для RAG):
```
REGIONAL_TRADE_PATTERNS_KB.txt
port_restrictions_map.txt
vessel_size_classification.txt
proximity_scoring_matrix.txt
ABBREVIATIONS_KNOWLEDGE_BASE.txt
```

### Examples (тестовые данные):
```
COMMENTS_PROCESSING_EXAMPLES.txt
emailsexamples.txt
```

---

## 🔑 ENVIRONMENT VARIABLES

### Laravel (.env):
```bash
LANGCHAIN_API_URL=http://localhost:8000
RABBITMQ_HOST=localhost
RABBITMQ_QUEUE=scoring_queue
MONGODB_URI=mongodb+srv://...
REDIS_HOST=localhost
```

### Python (.env):
```bash
OPENAI_API_KEY=sk-...
LANGSMITH_API_KEY=ls-...
LANGSMITH_PROJECT=vkchart-ai-prod
MONGODB_URI=mongodb+srv://...
MONGODB_DB_NAME=vkchart_ai
```

---

## 📊 DATA FLOW

```
1. User requests scoring (Frontend)
   ↓
2. Laravel prepares data (vessel + cargo info)
   ↓
3. Job queued in RabbitMQ
   ↓
4. Worker picks up job
   ↓
5. HTTP request to Python microservice
   ↓
6. Python runs P1→P2→P3→P4→P5→P6→P7
   ↓
7. MongoDB Atlas for RAG queries
   ↓
8. Python returns scores + reasoning
   ↓
9. Laravel stores in MySQL
   ↓
10. User reviews scores (Frontend)
```

---

## 🐛 COMMON ISSUES

**MongoDB connection fails:**
```bash
# Check Atlas IP whitelist
# Add your IP in Network Access
```

**RabbitMQ not processing:**
```bash
php artisan queue:restart
tail -f storage/logs/laravel.log
```

**Python service not responding:**
```bash
# Check if running
curl http://localhost:8000/health

# Restart
uvicorn app.main:app --reload
```

---

## 📞 HELP & SUPPORT

**Questions about:**
- Architecture → Read VKCHART_AI_ARCHITECTURE.md
- Scoring logic → Read MASTER_SCORING_SYSTEM.txt
- Specific criterion (P1-P7) → Read corresponding Rules file
- Business logic → Ask Vitaly

**Full documentation:**
→ DEVELOPER_IMPLEMENTATION_GUIDE.md (этот файл - quick reference)

---

## ✅ WEEK 1 CHECKLIST

- [ ] Прочитал DEVELOPER_IMPLEMENTATION_GUIDE.md
- [ ] Прочитал VKCHART_AI_ARCHITECTURE.md
- [ ] Прочитал MASTER_SCORING_SYSTEM.txt
- [ ] Установил Python dependencies (requirements.txt)
- [ ] Создал MongoDB Atlas cluster
- [ ] Настроил RabbitMQ
- [ ] Создал LangSmith account
- [ ] Laravel: создал basic API endpoint `/api/scoring/score`
- [ ] Python: создал FastAPI app с `/health` endpoint
- [ ] Протестировал Laravel → Python communication
- [ ] Спросил Виталия о непонятных моментах

---

**Quick Reference v1.0**  
**Для полной информации смотри DEVELOPER_IMPLEMENTATION_GUIDE.md**

🚀 **Good luck!**
