# 📋 VKChart-AI Development Checklist & Roadmap

## 🎯 Project Overview

**Start Date**: December 1, 2024  
**Target MVP**: December 12, 2024  
**Target Full Release**: December 27, 2024  

---

## 📊 Progress Tracker

| Phase | Status | Progress | ETA |
|-------|--------|----------|-----|
| Phase 1: Foundation | 🔲 Not Started | 0% | Dec 12 |
| Phase 2: Full Scoring | 🔲 Not Started | 0% | Dec 19 |
| Phase 3: Learning & Polish | 🔲 Not Started | 0% | Dec 27 |

---

## 🏗️ PHASE 1: Foundation (Week 1-2)

### 1.1 Project Setup (Day 1-2)

#### Python Service Setup
- [ ] Create project directory structure
- [ ] Initialize Python virtual environment
- [ ] Create `requirements.txt` with all dependencies
- [ ] Setup FastAPI application
- [ ] Configure environment variables (.env)
- [ ] Create Dockerfile
- [ ] Test basic endpoint works

#### MongoDB Connection
- [ ] Get MongoDB Atlas connection string from existing setup
- [ ] Create `mongo_client.py` utility
- [ ] Test connection
- [ ] Create collections (see Architecture doc)
- [ ] Create Vector Search index on `knowledge_embeddings`

#### RabbitMQ Setup
- [ ] Configure RabbitMQ connection
- [ ] Create queues:
  - [ ] `vkchart.scoring.requests`
  - [ ] `vkchart.scoring.results`
  - [ ] `vkchart.messages.requests`
  - [ ] `vkchart.messages.results`
  - [ ] `vkchart.learning.feedback`
  - [ ] `vkchart.normalize.logs`
- [ ] Test publish/consume

#### LLM Client Setup
- [ ] Create `llm_client.py` abstraction
- [ ] Configure Claude API key
- [ ] Configure OpenAI API key (backup)
- [ ] Test basic LLM call
- [ ] Implement retry logic

### 1.2 Normalization System (Day 3-4)

#### Aliases Files
- [ ] Create `aliases/ports_aliases.yaml`
- [ ] Create `aliases/regions_aliases.yaml`
- [ ] Create `aliases/cargo_aliases.yaml`
- [ ] Populate with initial data from existing knowledge

#### Rules Engine
- [ ] Implement `normalizers/rules_engine.py`
- [ ] Implement exact match lookup
- [ ] Implement case-insensitive matching
- [ ] Test with 20 known aliases

#### Fuzzy Matching
- [ ] Install RapidFuzz
- [ ] Implement fuzzy matching in `normalizers/port_normalizer.py`
- [ ] Configure similarity threshold (0.85)
- [ ] Test with misspelled ports

#### LLM Fallback (basic)
- [ ] Implement `normalizers/llm_fallback.py`
- [ ] Create prompt template for port identification
- [ ] Test with unknown abbreviations
- [ ] Log results to MongoDB

### 1.3 Laravel Integration (Day 5)

#### Events
- [ ] Create `CargoUpdatedEvent.php`
- [ ] Create `VesselPositionChangedEvent.php`
- [ ] Create `ScoringRequestedEvent.php`
- [ ] Register events in `EventServiceProvider`

#### Jobs
- [ ] Create `RequestScoringJob.php`
- [ ] Implement RabbitMQ publish
- [ ] Create `ProcessScoringResultJob.php`
- [ ] Test job dispatch

#### API Client
- [ ] Create `AIServiceClient.php`
- [ ] Implement HTTP methods (GET, POST)
- [ ] Add timeout handling
- [ ] Add retry logic

### 1.4 Scoring Agent - Basic (Day 6-7)

#### Tools Implementation
- [ ] Create `tools/intake_calculator.py`
- [ ] Create `tools/distance_calculator.py` (coordinate-based)
- [ ] Create `tools/comments_parser.py`
- [ ] Test each tool independently

#### Knowledge Loading
- [ ] Convert `proximity_scoring_matrix.txt` to YAML
- [ ] Convert `intake_calculator_formula.txt` to YAML
- [ ] Convert `OPEN_AREA_COMMENTS_SCORING.txt` to YAML
- [ ] Convert `READINESSETA_TO_LOADING_PORT_SCORING.txt` to YAML
- [ ] Load into MongoDB with embeddings

#### Scoring Agent (P1, P5, P6, P7)
- [ ] Create `agents/scoring_agent.py`
- [ ] Implement P1 (Proximity) calculation
- [ ] Implement P5 (Intake) calculation
- [ ] Implement P6 (OpenArea Comments) calculation
- [ ] Implement P7 (Readiness/ETA) calculation
- [ ] Implement final score calculation
- [ ] Implement blocking conditions

### 1.5 Message Agent - Basic (Day 8-9)

#### Templates
- [ ] Create `templates/email/case_a_strong_match.txt`
- [ ] Create `templates/email/case_a_standard.txt`
- [ ] Create `templates/email/case_b_not_ideal.txt`
- [ ] Create `templates/whatsapp/offer_short.txt`

#### Message Agent
- [ ] Create `agents/message_agent.py`
- [ ] Implement template selection logic
- [ ] Implement basic personalization
- [ ] Test email generation

### 1.6 Integration Testing (Day 10)

#### End-to-End Flow
- [ ] Test: Laravel → RabbitMQ → Python → MongoDB → Laravel
- [ ] Verify scoring result appears in UI
- [ ] Verify message generation works
- [ ] Fix any integration issues

#### Demo Preparation
- [ ] Prepare 5 test cases
- [ ] Document known limitations
- [ ] Create demo script

---

## 🎯 PHASE 2: Full Scoring (Week 3)

### 2.1 Remaining Criteria (Day 11-12)

#### P1A: Regional Patterns
- [ ] Convert `regional_trade_patterns.txt` to YAML
- [ ] Implement P1A calculation in Scoring Agent
- [ ] Test with 10 region pairs

#### P2: Regional Preferences
- [ ] Implement company preferences parsing
- [ ] Implement vessel preferences parsing
- [ ] Implement priority hierarchy (vessel > person > company)
- [ ] Test preference conflicts

#### P3: Cargo Preferences
- [ ] Implement cargo type matching
- [ ] Implement technical requirements check
- [ ] Test with different cargo types

#### P4: Last Ports
- [ ] Implement port history calculation
- [ ] Handle vessels with no history
- [ ] Test percentage calculations

### 2.2 Knowledge Base Enhancement (Day 13)

#### Convert All Files
- [ ] Convert `vessel_size_classification.txt` to YAML
- [ ] Convert `vessel_location_determination.txt` to YAML
- [ ] Convert `port_restrictions_map.txt` to YAML
- [ ] Convert `COMMENTS_PROCESSING_GUIDE.txt` to YAML

#### RAG Indexing
- [ ] Index all knowledge files
- [ ] Create embeddings for each chunk
- [ ] Test retrieval quality
- [ ] Tune chunk size if needed

### 2.3 LLM Fallback Normalizer - Full (Day 14)

#### Auto-Add Aliases
- [ ] Implement auto-add to `aliases_auto` collection
- [ ] Implement confidence thresholds
- [ ] Create notification system

#### Laravel UI for Pending Aliases
- [ ] Create "Pending Aliases" page
- [ ] Implement approve/reject actions
- [ ] Show context (which vessel/cargo triggered)
- [ ] Log rejection reasons

### 2.4 Testing & Tuning (Day 15)

#### Create Test Dataset
- [ ] Create 50 vessel-cargo test pairs
- [ ] Define expected scores for each
- [ ] Include edge cases

#### Accuracy Testing
- [ ] Run all test cases
- [ ] Measure accuracy per criterion
- [ ] Document deviations

#### Prompt Tuning
- [ ] Analyze worst performers
- [ ] Adjust prompts
- [ ] Re-run tests
- [ ] Target: >80% accuracy

---

## 🧠 PHASE 3: Learning & Polish (Week 4)

### 3.1 Learning System (Day 16-17)

#### Feedback Collector
- [ ] Create `learning/feedback_collector.py`
- [ ] Implement correction storage
- [ ] Implement explanation capture

#### Pattern Extraction
- [ ] Implement context extraction
- [ ] Implement similarity detection
- [ ] Store patterns in MongoDB

#### Pattern Application
- [ ] Implement pattern lookup during scoring
- [ ] Show "Based on previous correction" in reasoning
- [ ] Track pattern usage

#### Laravel UI for Corrections
- [ ] Add "Edit Score" button
- [ ] Create correction form
- [ ] Show explanation prompt
- [ ] Submit to learning queue

### 3.2 Message Enhancement (Day 18)

#### All Templates
- [ ] Create remaining email templates
- [ ] Create `reminder_1.txt`, `reminder_2.txt`, `reminder_3.txt`
- [ ] Create WhatsApp templates

#### Full Personalization
- [ ] Implement nationality-based tone adjustment
- [ ] Implement language detection/adjustment
- [ ] Implement communication style matching
- [ ] Implement relationship history awareness

#### Reminder Logic
- [ ] Implement "today_contact" check
- [ ] Implement previous message reference
- [ ] Test reminder chain

### 3.3 Monitoring & Logging (Day 19)

#### Metrics Collection
- [ ] Track scoring request count
- [ ] Track average processing time
- [ ] Track LLM costs
- [ ] Track accuracy rate

#### Error Tracking
- [ ] Implement structured logging
- [ ] Setup error alerts
- [ ] Create error dashboard

#### Performance Monitoring
- [ ] Monitor queue depths
- [ ] Monitor response times
- [ ] Setup alerts for slow responses

### 3.4 Production Preparation (Day 20)

#### Security Review
- [ ] Audit API keys storage
- [ ] Review authentication
- [ ] Check data sanitization

#### Load Testing
- [ ] Test with 100 concurrent requests
- [ ] Identify bottlenecks
- [ ] Optimize if needed

#### Documentation
- [ ] API documentation
- [ ] Deployment guide
- [ ] User guide
- [ ] Troubleshooting guide

#### Deployment
- [ ] Configure production environment
- [ ] Deploy to production server
- [ ] Run smoke tests
- [ ] Monitor first 24 hours

---

## 📝 Daily Standup Template

```
Date: ___________

✅ Completed Yesterday:
- 

🎯 Plan for Today:
- 

🚧 Blockers:
- 

📊 Overall Progress: ___%
```

---

## 🔴 Critical Path Items

These items MUST be completed on time:

1. **Day 2**: MongoDB + RabbitMQ connected
2. **Day 4**: Normalization working (at least rules + fuzzy)
3. **Day 7**: P1, P5, P6, P7 scoring working
4. **Day 10**: Full integration (Laravel ↔ Python) working
5. **Day 12**: MVP Demo ready
6. **Day 15**: All 7 criteria working
7. **Day 17**: Learning system storing corrections
8. **Day 20**: Production deployment

---

## 🧪 Test Cases to Prepare

### Scoring Tests

| # | Scenario | Expected Score | Priority |
|---|----------|---------------|----------|
| 1 | Vessel at same port as loading | P1 = 25 | High |
| 2 | Vessel in different sea | P1 = 5-10 | High |
| 3 | Intake < 90% of cargo | BLOCK | Critical |
| 4 | OpenArea says "no grains" + grain cargo | P6 = -40, BLOCK | Critical |
| 5 | ETA 2 days before laycan | P7 = high | Medium |
| 6 | ETA 3 days after laycan end | P7 = low | Medium |
| 7 | Company specializes in loading region | P2 = 15 | Medium |
| 8 | Vessel never been to region | P4 = 0 | Low |

### Normalization Tests

| # | Input | Expected Output | Method |
|---|-------|-----------------|--------|
| 1 | "odessa" | UAODS | Exact |
| 2 | "odesa" | UAODS | Fuzzy |
| 3 | "cta" | ROCTA | LLM |
| 4 | "konshtanza" | ROCTA | Fuzzy |
| 5 | "blacksea" | Black Sea | Fuzzy |

### Message Tests

| # | Score | Message Type | Key Check |
|---|-------|--------------|-----------|
| 1 | 90 | Offer | Strong match tone |
| 2 | 75 | Offer | Standard professional |
| 3 | 62 | Offer | Caveats mentioned |
| 4 | 80 | Reminder 1 | References original |
| 5 | 80 + today_contact | Reminder | No greeting |

---

## 📞 Key Contacts

- **Vitaly (PM/Owner)**: Architecture decisions, domain expertise
- **Oleksiy (Laravel Dev)**: Laravel integration, UI changes
- **MongoDB Atlas**: Check existing setup
- **Anthropic API**: Claude integration

---

## 🗒️ Notes & Decisions Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2024-11-30 | 2 agents (Scoring + Message) | Clear separation of concerns |
| 2024-11-30 | Hybrid RAG + Rules | Balance speed and flexibility |
| 2024-11-30 | RabbitMQ for async | Matches existing infrastructure |
| | | |

---

## 🚨 Risk Register

| Risk | Probability | Impact | Mitigation | Status |
|------|------------|--------|------------|--------|
| LLM latency | Medium | High | Caching, batching | Monitoring |
| Accuracy < 80% | Medium | High | Prompt tuning, examples | Active |
| Integration issues | High | Medium | Early integration | Planning |
| Scope creep | High | High | Strict MVP scope | Active |

---

*Last Updated: 2024-11-30*
