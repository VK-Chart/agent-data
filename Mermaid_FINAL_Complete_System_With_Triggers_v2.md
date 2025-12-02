# 🎯 VK Charts - ПОЛНАЯ СИСТЕМНАЯ СХЕМА С LANG* STACK И ТРИГГЕРАМИ

**Версия**: 2.0 (Updated with LangGraph, LangSmith, LangServe)  
**Дата**: 2024-12-02  
**Изменения**: Детализирована архитектура AI Service с полным Lang* Stack

```mermaid
flowchart TB
    %% ═══════════════════════════════════════════════════════════════════
    %% ТРИГГЕРЫ ПЕРЕСЧЕТА СКОРИНГА
    %% ═══════════════════════════════════════════════════════════════════
    
    subgraph Triggers["🔔 ТРИГГЕРЫ ПЕРЕСЧЕТА (События → Действия)"]
        direction LR
        
        subgraph TriggerEvents["📡 События-инициаторы"]
            T1["📦 Груз изменен:<br>• Порты (load/disch)<br>• Laycan даты<br>• Количество<br>• Тип груза<br>• Ставки фрахта"]
            T2["🚢 Судно обновлено:<br>• Open area/port<br>• Open date<br>• ETA/Destination<br>• Current position<br>• Status (laden/ballast)"]
            T3["💬 Комментарии:<br>• OpenArea comments<br>• Company prefs<br>• Vessel notes<br>• Person instructions"]
            T4["⏰ Время:<br>• Прошло N часов<br>• Приближается laycan<br>• Смена месяца/сезона<br>• Начало grain season"]
            T5["🌍 Внешние:<br>• Геополитика<br>• Порт закрыт<br>• Погода/лед<br>• Рынок изменился"]
            T6["👤 Ответ клиента:<br>• Обновил позицию<br>• Изменил preferences<br>• Судно зафиксировано<br>• Контр-оффер"]
        end
        
        subgraph TriggerActions["⚡ Действия по триггерам"]
            TA1["🔄 Полный пересчет<br>всех критериев"]
            TA2["📊 Частичный пересчет<br>affected criteria only"]
            TA3["🚫 Блокировка оффера<br>если P6 ≤ -40"]
            TA4["💾 Обновление кэша"]
            TA5["📧 Генерация нового текста"]
            TA6["⏰ Сброс таймера reminder"]
        end
        
        T1 --> TA1
        T2 --> TA2
        T3 --> TA2
        T4 --> TA2
        T5 --> TA1
        T6 --> TA1
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% МОДУЛЬ СКОРИНГА (P1-P7) - UPDATED WITH LANGGRAPH
    %% ═══════════════════════════════════════════════════════════════════
    
    subgraph ScoringEngine["🧮 SCORING ENGINE [LangGraph + LangChain + MongoDB]"]
        direction TB
        
        subgraph DataCollection["📊 Сбор данных [Laravel]"]
            LoadVessel["🚢 vessel_data<br>vessel_attributes<br>vessel_comments"]
            LoadCargo["📦 cargos, ports<br>port_restrictions<br>cargo_types"]
            LoadCompany["🏢 companies<br>company_vessel<br>preferences"]
            LoadHistory["📜 vessel_cargo_statuses<br>last_ports_calls<br>message_history"]
        end
        
        subgraph LangGraphWorkflow["🔄 LANGGRAPH WORKFLOW (State Management)"]
            StateInit["🎬 Initialize State:<br>ScoringState TypedDict<br>vessel, cargo, company<br>scores: {}, is_blocked: False"]
            
            Normalize["⚙️ NODE: Normalize Data<br>Port names, dates<br>vessel position"]
            
            CheckData["✅ Validate Input:<br>Required fields present?<br>Data quality OK?"]
        end
        
        subgraph CriteriaCalc["🎯 Расчет критериев P1-P7 [LangGraph Nodes]"]
            P1["🗺️ NODE P1: PROXIMITY (0-20)<br>RAG: proximity_scoring_matrix.txt<br>LLM: Calculate distance score<br>Tool: estimate_distance()"]
            P1A["🌐 NODE P1A: PATTERNS (0-15)<br>RAG: regional_trade_patterns.txt<br>LLM: Analyze trade viability<br>Season-aware scoring"]
            P2["🌍 NODE P2: REGIONAL (0-15)<br>RAG: COMMENTS_PROCESSING_GUIDE<br>LLM: Parse regional preferences<br>Match cargo region"]
            P3["📦 NODE P3: CARGO (0-15)<br>RAG: COMMENTS_PROCESSING_GUIDE<br>LLM: Parse cargo preferences<br>Match cargo type"]
            P4["📊 NODE P4: LAST PORTS (0-10)<br>Query: vessel_cargo_statuses<br>Calculate: region % history<br>LLM: Analyze pattern"]
            P5["⚖️ NODE P5: INTAKE (0-15)<br>Tool: calculate_intake()<br>Check: ≥90% cargo qty?<br>BLOCK if insufficient"]
            P6["💬 NODE P6: OPEN AREA (-50,+25)<br>RAG: OPEN_AREA_COMMENTS<br>Tool: parse_comments()<br>LLM: Evaluate alignment<br>BLOCK if ≤ -40"]
            P7["⏰ NODE P7: READINESS (0-10)<br>RAG: READINESSETA_SCORING<br>Tool: calculate_eta()<br>LLM: Timing analysis"]
            
            CheckP5Block{"CONDITIONAL:<br>P5 blocked?<br>(intake < 90%)"}
            CheckP6Block{"CONDITIONAL:<br>P6 ≤ -40?<br>(Owner avoids)"}
        end
        
        subgraph FinalProcessing["📊 Финализация [LangGraph Nodes]"]
            FinalScore["🎯 NODE: FINAL SCORE<br>Base = sum(P1-P5,P7)<br>Modified = Base + P6<br>Final = CLAMP(0,100)<br>Classification: excellent/good/..."]
            
            Reasoning["📝 NODE: Generate Reasoning<br>LLM: Analyze all scores<br>• Strengths (high scores)<br>• Weaknesses (low scores)<br>• Overall recommendation"]
            
            SaveState["💾 Checkpoint State<br>PostgreSQL: full state<br>For human-in-loop<br>corrections"]
        end
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% LANGSMITH OBSERVABILITY - NEW
    %% ═══════════════════════════════════════════════════════════════════
    
    subgraph LangSmith["🔍 LANGSMITH OBSERVABILITY"]
        direction LR
        
        AutoTrace["📊 Automatic Tracing:<br>• All LLM calls<br>• Tool executions<br>• RAG queries<br>• Full execution graph"]
        
        Evaluation["✅ Evaluation:<br>• Dataset: 20 test cases<br>• Metrics: accuracy ≥85%<br>• Regression testing<br>• A/B prompt testing"]
        
        Production["🚨 Production Monitor:<br>• Requests/min<br>• Avg latency<br>• Error rate<br>• Cost tracking<br>• Alerts"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% LANGSERVE API - NEW
    %% ═══════════════════════════════════════════════════════════════════
    
    subgraph LangServe["🚀 LANGSERVE API LAYER"]
        direction LR
        
        APIEndpoints["🔌 Endpoints:<br>POST /scoring/invoke<br>POST /scoring/stream<br>POST /scoring/batch<br>GET /scoring/playground"]
        
        StreamProgress["📡 Streaming:<br>Real-time progress:<br>✓ P1 calculating...<br>✓ P2 calculating...<br>✓ Final score..."]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% МОДУЛЬ МАТЧИНГА И ОЦЕНКИ [из старой схемы]
    %% ═══════════════════════════════════════════════════════════════════

    subgraph MatchingEngine["🎯 МОДУЛЬ МАТЧИНГА И ОЦЕНКИ [Laravel]"]
        AutoMatch["🤖 Автоматический матчинг<br>груза с судами"]
        LoadSettings["⚙️ Загрузка настроек:<br>- Батч N с компаниями<br>- Пороги оценок<br>- Интервалы ремайндеров<br>- API ключи ИИ"]
        CalcScore["📊 Вызов LangServe API<br>POST /scoring/invoke<br>для каждой пары<br>судно-груз"]
        SortCompanies["🔽 Сортировка компаний<br>по общей оценке<br>(от высокой к низкой)"]
        SelectTop["✂️ Выбор первого батча<br>из настроек (default: 20)"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% МОДУЛЬ ГЕНЕРАЦИИ ИИ - UPDATED WITH LANGGRAPH
    %% ═══════════════════════════════════════════════════════════════════

    subgraph AIGeneration["🤖 МОДУЛЬ ГЕНЕРАЦИИ ИИ [LangChain + LangGraph]"]
        CheckThreshold{"Оценка ><br>порога?"}
        PrepareFullData["📋 Подготовка полных данных"]
        PrepareEmailData["📧 Подготовка данных email"]
        
        subgraph MessageWorkflow["🔄 MESSAGE GENERATION WORKFLOW"]
            CollectData["🗂️ Сбор данных для ИИ:<br>- Person: nationality, timezone, prompt<br>- Company: location, prompt<br>- Vessel: position, compatibility<br>- Cargo: description, ports<br>- Context: time, history"]
            CheckCharterer{"Есть связка<br>с charterer?"}
            AddWarning["⚠️ Добавить предупреждение<br>о связке с грузовладельцем"]
            AICall["🧠 LLM Call (Claude/GPT-4):<br>Temperature: 0.7<br>Max tokens: 1000<br>LangSmith: auto-traced"]
            GenerateTexts["✍️ Генерация текстов:<br>- Email для компании<br>- WhatsApp/Teams сообщения<br>emailsexamples.txt"]
        end
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% МОДУЛЬ ОДОБРЕНИЯ - UPDATED WITH HUMAN-IN-LOOP
    %% ═══════════════════════════════════════════════════════════════════

    subgraph UserApproval["👤 МОДУЛЬ ОДОБРЕНИЯ [Laravel UI + LangGraph Checkpoints]"]
        ShowPreview["👁️ Показ пользователю:<br>- Score breakdown P1-P7<br>- Reasoning<br>- Сгенерированный текст<br>- LangSmith trace link"]
        UserCheck{"Действие<br>пользователя?"}
        EditText["✏️ Редактирование"]
        
        subgraph HumanInLoop["🔄 HUMAN-IN-THE-LOOP [LangGraph]"]
            CorrectScore["✏️ Исправление score:<br>User: 'P1 should be 8, not 15'<br>Reason: 'Libya→BlackSea<br>not profitable for Handy'"]
            LoadCheckpoint["💾 Load checkpoint state:<br>Get full ScoringState<br>from PostgreSQL"]
            UpdateState["🔄 Update state:<br>scores.p1 = 8<br>scores.p1.reasoning = user_reason"]
            ResumeWorkflow["▶️ Resume from P1A node:<br>Recalculate P1A-P7<br>using new P1"]
        end
        
        RateFeedback["👍👎 Оценка ИИ"]
        AskLater{"Отправить<br>позже?"}
        SaveFeedback["💾 Сохранение для<br>fine-tuning"]
        QueueLater["📌 В конец очереди"]
        MarkSkipped["❌ Метка: не отправлять"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% МОДУЛЬ ОТПРАВКИ [из старой схемы]
    %% ═══════════════════════════════════════════════════════════════════

    subgraph SendingModule["📤 МОДУЛЬ ОТПРАВКИ [Laravel]"]
        CheckDuplicate{"Проверка<br>дубликата"}
        SkipDuplicate["⚠️ Пропуск дубликата<br>Настройка: дней до повтора"]
        AddCCEmails["📧 Добавление в копию:<br>- Emails компании<br>- Emails людей по судну"]
        SendChannels["📨 Отправка:<br>- Email SMTP<br>- WhatsApp API<br>- Teams Webhook"]
        RecordSent["💾 Запись в БД:<br>- Кому, когда, что<br>- Статус: отправлено"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% СИСТЕМА РЕМАЙНДЕРОВ [детально из старой схемы]
    %% ═══════════════════════════════════════════════════════════════════

    subgraph ReminderSystem["⏰ СИСТЕМА РЕМАЙНДЕРОВ - ДЕТАЛЬНАЯ ЛОГИКА"]
        StartTimer["⏱️ Запуск таймера:<br>Интервал из настроек"]
        WaitEvents["⏳ Ожидание событий"]
        CheckEvent{"Событие?"}
        CheckReminderSettings{"Проверки<br>ремайндера"}
        ProcessResponse["💬 Обработка ответа"]
        ComboOffer["📦 Комбо-оффер"]
        NotifyIncrease["📈 Уведомление"]
        ForceReminder["⚡ Принудительный ремайндер"]
        StopReminder["🛑 Не отправлять"]
        SkipLowScore["⬇️ Низкая оценка"]
        MaxReached["📊 Лимит достигнут"]
        PrepareReminder["📝 Подготовка ремайндера"]
        BuildReminderPrompt["🔨 ФОРМИРОВАНИЕ ПРОМПТА:<br>1. Базовый промпт<br>2. Номер ремайндера<br>3. ВСЕ предыдущие<br>4. История ответов<br>5. Время с первого<br>6. Оригинальный оффер<br>7. Статус груза<br>8. Изменения позиции<br>9. Рыночная ситуация"]
        CheckTodayContact{"Писали сегодня<br>этому клиенту?"}
        NoGreeting["❌ Промпт: БЕЗ приветствия"]
        WithGreeting["✅ Промпт: С приветствием"]
        GenerateReminder["🤖 ИИ генерирует ремайндер<br>LangSmith: traced"]
        BuildComboPrompt["🔄 ФОРМИРОВАНИЕ КОМБО:<br>1. НОВЫЙ ГРУЗ - фокус<br>2. СТАРЫЙ - кратко<br>3. Причина замены<br>4. Связка грузов<br>5. История офферов"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% ОБРАБОТКА ОТВЕТОВ КЛИЕНТОВ [из старой схемы]
    %% ═══════════════════════════════════════════════════════════════════

    subgraph ResponseHandling["💬 ОБРАБОТКА ОТВЕТОВ КЛИЕНТОВ"]
        CategorySelect["📂 Выбор категории"]
        MainCategory{"Основная<br>категория?"}
        NotInterested["❌ Груз не интересен"]
        AskedFreight["💰 Спросил фрахт"]
        AskedDetails["📋 Спросил детали"]
        OtherChannel["🔄 Другой канал"]
        UpdatedVessel["🚢 Обновил судно"]
        SubCategoryA{"Причина?"}
        RecordReason["💾 Запись причины:<br>A1-A10"]
        UpdateStatus["🔄 Обновление статусов"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% ЦИКЛ ОБРАБОТКИ [из старой схемы]
    %% ═══════════════════════════════════════════════════════════════════

    subgraph ProcessingLoop["🔄 ЦИКЛ ОБРАБОТКИ"]
        NextCompany{"Еще есть<br>компании?"}
        ProcessQueue["📋 Обработка<br>отложенной очереди"]
        EndCycle["✅ Цикл завершен"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% LEARNING & MONITORING - UPDATED WITH LANGSMITH
    %% ═══════════════════════════════════════════════════════════════════

    subgraph Learning["🧠 ОБУЧЕНИЕ [MongoDB Atlas + LangSmith]"]
        StorePattern["💾 Сохранение паттерна:<br>- Score correction<br>- User reasoning<br>- Context<br>→ LangSmith Dataset"]
        VectorSearch["🔍 Vector search<br>похожих случаев<br>MongoDB Atlas"]
        UpdateWeights["⚖️ Обновление весов<br>на основе corrections"]
        EvaluateChanges["📊 LangSmith Evaluation:<br>Compare old vs new<br>accuracy"]
    end

    subgraph Monitoring["📊 МОНИТОРИНГ [LangSmith + Laravel]"]
        LogAction["📝 Логирование<br>LangSmith: auto-trace"]
        Metrics["📈 Метрики:<br>- Отправлено офферов<br>- Конверсия<br>- Время ответа<br>- Причины отказов<br>- LLM cost/request<br>- Avg latency"]
        Dashboard["📊 Dashboard:<br>- Статистика<br>- Графики<br>- Отчеты<br>- LangSmith traces"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% СИСТЕМНЫЕ НАСТРОЙКИ [детально из старой схемы]
    %% ═══════════════════════════════════════════════════════════════════

    subgraph SystemSettings["⚙️ ВСЕ НАСТРОЙКИ СИСТЕМЫ"]
        Settings["🎛️ ПАРАМЕТРЫ:<br>━━━━━━━━━━━━━<br>📤 ОТПРАВКА:<br>• Компаний в батче: 20<br>• Подготовка след.батча: после 15<br>• Пауза между отправками: 0 сек<br>━━━━━━━━━━━━━<br>📊 ПОРОГИ:<br>• Мин. для личных: 75<br>• Мин. для email reminder: 50<br>• Мин. для msg reminder: 80<br>━━━━━━━━━━━━━<br>⏰ РЕМАЙНДЕРЫ:<br>• До 1го: 2ч<br>• Между 1-2: 1ч<br>• Между 2-3: 2ч<br>• Последующие: 3ч<br>• Активность: 8-20ч Киев<br>━━━━━━━━━━━━━<br>🤖 ИИ:<br>• LLM: Claude Sonnet 4<br>• Embeddings: OpenAI Ada-002<br>• Temperature: 0.7<br>• Max tokens: 1000<br>• LangSmith: enabled<br>━━━━━━━━━━━━━<br>🔄 ДУБЛИКАТЫ:<br>• Дней до повтора: 7-30"]
    end

    %% ═══════════════════════════════════════════════════════════════════
    %% ОСНОВНЫЕ СВЯЗИ И ПОТОКИ - UPDATED
    %% ═══════════════════════════════════════════════════════════════════

    Start(["🚀 СТАРТ: Новый груз добавлен"]) --> AutoMatch
    
    %% Триггеры инициируют пересчет через LangGraph
    TriggerActions -.->|"TRIGGER<br>RECALC"| StateInit
    
    %% Основной поток матчинга
    AutoMatch --> LoadSettings
    LoadSettings --> LoadVessel & LoadCargo & LoadCompany & LoadHistory
    
    %% LangGraph Workflow Start
    LoadVessel & LoadCargo & LoadCompany & LoadHistory --> StateInit
    StateInit --> Normalize
    Normalize --> CheckData
    CheckData --> P1
    
    %% Sequential P1-P7 with RAG and Tools
    P1 --> P1A --> P2 --> P3 --> P4 --> P5
    P5 --> CheckP5Block
    CheckP5Block -->|"Blocked"| SaveState
    CheckP5Block -->|"Continue"| P6
    P6 --> CheckP6Block
    CheckP6Block -->|"Blocked (≤-40)"| SaveState
    CheckP6Block -->|"Continue"| P7
    P7 --> FinalScore
    FinalScore --> Reasoning
    Reasoning --> SaveState
    SaveState --> CalcScore
    
    %% LangSmith observability (passive monitoring)
    P1 & P1A & P2 & P3 & P4 & P5 & P6 & P7 & FinalScore -.->|"Auto-trace"| AutoTrace
    AutoTrace --> Evaluation
    Evaluation --> Production
    
    %% LangServe API layer
    CalcScore --> APIEndpoints
    APIEndpoints --> StreamProgress
    StreamProgress --> SortCompanies
    
    %% От скоринга к сортировке
    SortCompanies --> SelectTop
    SelectTop --> CheckThreshold
    
    %% Генерация текстов
    CheckThreshold -->|"> Порог личных"| PrepareFullData
    CheckThreshold -->|"≤ Порог"| PrepareEmailData
    PrepareFullData & PrepareEmailData --> CollectData
    CollectData --> CheckCharterer
    CheckCharterer -->|"Да"| AddWarning
    CheckCharterer -->|"Нет"| AICall
    AddWarning --> AICall
    AICall --> GenerateTexts
    
    %% Одобрение пользователя с Human-in-the-Loop
    GenerateTexts --> ShowPreview
    ShowPreview --> UserCheck
    UserCheck -->|"Исправить score"| CorrectScore
    CorrectScore --> LoadCheckpoint
    LoadCheckpoint --> UpdateState
    UpdateState --> ResumeWorkflow
    ResumeWorkflow --> ShowPreview
    
    UserCheck -->|"Отправить"| RateFeedback
    UserCheck -->|"Редактировать"| EditText --> AICall
    UserCheck -->|"Не отправлять"| AskLater
    RateFeedback --> SaveFeedback
    AskLater -->|"Да"| QueueLater
    AskLater -->|"Нет"| MarkSkipped
    SaveFeedback --> StorePattern
    
    %% Отправка
    SaveFeedback --> CheckDuplicate
    CheckDuplicate -->|"Дубликат"| SkipDuplicate
    CheckDuplicate -->|"OK"| AddCCEmails
    AddCCEmails --> SendChannels
    SendChannels --> RecordSent
    RecordSent --> StartTimer
    
    %% Ремайндеры
    StartTimer --> WaitEvents
    WaitEvents --> CheckEvent
    CheckEvent -->|"Таймер истек"| CheckReminderSettings
    CheckEvent -->|"Ответ клиента"| ProcessResponse
    CheckEvent -->|"Новый груз"| ComboOffer
    CheckEvent -->|"Оценка выросла"| NotifyIncrease
    CheckEvent -->|"Форсировать"| ForceReminder
    
    CheckReminderSettings -->|"Остановлен"| StopReminder
    CheckReminderSettings -->|"< Мин.оценка"| SkipLowScore
    CheckReminderSettings -->|">= Макс.кол-во"| MaxReached
    CheckReminderSettings -->|"Все OK"| PrepareReminder
    
    PrepareReminder --> BuildReminderPrompt
    BuildReminderPrompt --> CheckTodayContact
    CheckTodayContact -->|"Да"| NoGreeting
    CheckTodayContact -->|"Нет"| WithGreeting
    NoGreeting & WithGreeting --> GenerateReminder
    GenerateReminder --> ShowPreview
    
    ComboOffer --> BuildComboPrompt
    BuildComboPrompt --> GenerateTexts
    
    %% Обработка ответов
    ProcessResponse --> CategorySelect
    CategorySelect --> MainCategory
    MainCategory -->|"A"| NotInterested
    MainCategory -->|"B"| AskedFreight
    MainCategory -->|"C"| AskedDetails
    MainCategory -->|"D"| OtherChannel
    MainCategory -->|"E"| UpdatedVessel
    
    NotInterested --> SubCategoryA
    SubCategoryA --> RecordReason
    RecordReason --> UpdateStatus
    UpdatedVessel --> T6
    
    %% Обновление статусов и триггеры
    UpdateStatus --> LogAction
    LogAction --> Metrics
    Metrics --> Dashboard
    
    %% Цикл обработки
    SkipDuplicate & QueueLater & MarkSkipped --> NextCompany
    NextCompany -->|"Да"| CheckThreshold
    NextCompany -->|"Отложенные"| ProcessQueue
    NextCompany -->|"Все готово"| EndCycle
    ProcessQueue --> CheckThreshold
    
    %% Обучение с LangSmith
    StorePattern --> VectorSearch
    VectorSearch --> UpdateWeights
    UpdateWeights --> EvaluateChanges
    EvaluateChanges -.->|"Улучшение<br>модели"| P1
    
    %% Настройки влияют на все
    Settings -.-> LoadSettings & CheckThreshold & CheckReminderSettings & AICall
    
    EndCycle --> End(["🏁 ЗАВЕРШЕНИЕ"])
    Dashboard --> End

    %% ═══════════════════════════════════════════════════════════════════
    %% СТИЛИ И ЦВЕТОВАЯ СХЕМА - UPDATED
    %% ═══════════════════════════════════════════════════════════════════

    classDef triggerStyle fill:#ffccbc,stroke:#d84315,stroke-width:3px,stroke-dasharray: 5 5
    classDef scoringStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:3px
    classDef langgraphStyle fill:#e1bee7,stroke:#6a1b9a,stroke-width:3px
    classDef langsmithStyle fill:#fff3e0,stroke:#e65100,stroke-width:3px
    classDef langserveStyle fill:#e0f2f1,stroke:#00695c,stroke-width:3px
    classDef langchainStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:3px
    classDef laravelStyle fill:#bbdefb,stroke:#1565c0,stroke-width:3px
    classDef settingsStyle fill:#fff9c4,stroke:#f57c00,stroke-width:2px,stroke-dasharray: 5 5
    
    class Triggers,T1,T2,T3,T4,T5,T6,TriggerActions triggerStyle
    class ScoringEngine,P1,P1A,P2,P3,P4,P5,P6,P7,FinalScore,Reasoning scoringStyle
    class LangGraphWorkflow,StateInit,Normalize,CheckData,CheckP5Block,CheckP6Block,SaveState langgraphStyle
    class HumanInLoop,CorrectScore,LoadCheckpoint,UpdateState,ResumeWorkflow langgraphStyle
    class LangSmith,AutoTrace,Evaluation,Production langsmithStyle
    class LangServe,APIEndpoints,StreamProgress langserveStyle
    class AIGeneration,AICall,GenerateTexts,GenerateReminder,MessageWorkflow langchainStyle
    class MatchingEngine,UserApproval,SendingModule,ResponseHandling,ProcessingLoop laravelStyle
    class SystemSettings,Settings settingsStyle
    class Learning,StorePattern,VectorSearch,UpdateWeights,EvaluateChanges langsmithStyle
    
    style BuildReminderPrompt fill:#ffecb3,stroke:#ff6f00,stroke-width:3px
    style BuildComboPrompt fill:#ffecb3,stroke:#ff6f00,stroke-width:3px
    style Start fill:#e1f5e1,stroke:#4caf50,stroke-width:4px
    style End fill:#ffebee,stroke:#f44336,stroke-width:4px
    style ReminderSystem fill:#fff9c4,stroke:#ffeb3b,stroke-width:2px
    style Monitoring fill:#e1f5fe,stroke:#03a9f4,stroke-width:2px
```

## 📊 КЛЮЧЕВЫЕ ОБНОВЛЕНИЯ В ВЕРСИИ 2.0

### 🆕 **ЧТО ИЗМЕНИЛОСЬ:**

#### 1. **LangGraph Integration** 🔄
- **StateGraph Workflow**: P1-P7 теперь представлены как nodes в LangGraph
- **State Management**: ScoringState TypedDict с vessel, cargo, scores, is_blocked
- **Conditional Routing**: 
  - После P5: проверка intake → BLOCK если < 90%
  - После P6: проверка score → BLOCK если ≤ -40
- **Checkpointing**: PostgreSQL сохраняет full state для human-in-the-loop
- **Visualization**: Automatic Mermaid diagram generation

#### 2. **LangSmith Observability** 🔍
- **Automatic Tracing**: Все LLM calls, tool calls, RAG queries traced
- **Evaluation Framework**: 
  - Dataset: 20 test cases
  - Metrics: accuracy ≥85%
  - Regression testing after changes
- **Production Monitoring**:
  - Requests/min
  - Avg latency
  - Error rate
  - Cost per request
  - Alerts (error >5%, latency >10s)

#### 3. **LangServe API Layer** 🚀
- **Endpoints**:
  - `POST /scoring/invoke` - single request
  - `POST /scoring/stream` - streaming progress
  - `POST /scoring/batch` - batch processing
  - `GET /scoring/playground` - interactive testing
- **Streaming Progress**: Real-time UI updates
  - ✓ P1 calculating... (15/20)
  - ✓ P2 calculating... (12/15)
  - ✓ Final score... (78/100)

#### 4. **Human-in-the-Loop Workflow** 👤
- **Correction Flow**:
  1. User: "P1 should be 8, not 15"
  2. System: Load checkpoint from PostgreSQL
  3. System: Update P1 score in state
  4. System: Resume workflow from P1A node
  5. System: Recalculate P1A-P7 with new P1
- **Benefits**: Не нужно пересчитывать всё с нуля

#### 5. **Enhanced Learning** 🧠
- **LangSmith Dataset**: User corrections → test cases
- **Vector Search**: MongoDB Atlas для похожих случаев
- **Evaluation**: Compare old vs new accuracy after updates
- **Feedback Loop**: Corrections improve future scoring

---

## 🔗 ТЕХНОЛОГИЧЕСКИЙ СТЕК

### **Layer 1: Laravel (Orchestrator)**
- Data management (vessels, cargos, companies)
- UI for approval and monitoring
- Event triggers
- Queue management (RabbitMQ)

### **Layer 2: RabbitMQ (Message Broker)**
- `scoring.requests` queue
- `scoring.results` queue
- `messages.requests` queue
- `learning.corrections` queue

### **Layer 3: Python AI Service**
#### LangChain (Core)
- Agents: ScoringAgent, MessageAgent
- RAG: MongoDBAtlasVectorSearch
- Tools: @tool decorators
- LLMs: Claude Sonnet 4, GPT-4

#### LangGraph (Workflows)
- ScoringWorkflow: StateGraph(ScoringState)
- Nodes: P1-P7, Normalize, FinalScore, Reasoning
- Conditional Edges: P5 blocking, P6 blocking
- Checkpointing: PostgresSaver

#### LangServe (API)
- FastAPI integration
- Streaming endpoints
- Automatic playground
- OpenAPI docs

### **Layer 4: LangSmith (Observability)**
- Automatic tracing
- Evaluation framework
- Production monitoring
- Cost tracking
- Debugging tools

### **Layer 5: Data Stores**
- **MongoDB Atlas**: Vectors, scoring, learning
- **PostgreSQL**: LangGraph checkpoints
- **MySQL**: Laravel data (vessels, cargos)

---

## 📈 КРИТЕРИИ СКОРИНГА (P1-P7) - UPDATED WITH LANGGRAPH

| Критерий | Диапазон | LangGraph Node | Компоненты | Триггеры пересчета |
|----------|----------|----------------|------------|-------------------|
| **P1: Proximity** | 0-20 | `calculate_p1` | RAG + LLM + Tool(estimate_distance) | T2, T5 |
| **P1A: Regional** | 0-15 | `calculate_p1a` | RAG + LLM (season-aware) | T4, T5 |
| **P2: Regional Prefs** | 0-15 | `calculate_p2` | RAG + LLM + Tool(parse_comments) | T3 |
| **P3: Cargo Prefs** | 0-15 | `calculate_p3` | RAG + LLM + Tool(parse_comments) | T1, T3 |
| **P4: Last Ports** | 0-10 | `calculate_p4` | DB Query + LLM | Редко |
| **P5: Intake** | 0-15 | `calculate_p5` | Tool(calculate_intake) → **BLOCK** | T1, T3 |
| **P6: OpenArea** | -50,+25 | `calculate_p6` | RAG + Tool + LLM → **BLOCK** | T3 |
| **P7: Readiness** | 0-10 | `calculate_p7` | RAG + Tool(calculate_eta) + LLM | T1, T2, T4 |
| **Final Score** | 0-100 | `calculate_final` | sum + clamp + classify | - |
| **Reasoning** | text | `generate_reasoning` | LLM analysis | - |

### 🔄 **LangGraph Execution Flow:**
```
START → StateInit → Normalize → CheckData →
→ P1 → P1A → P2 → P3 → P4 → P5 →
→ [CHECK: P5 blocked?] →
  - YES → SaveState → END
  - NO → P6 →
→ [CHECK: P6 ≤ -40?] →
  - YES → SaveState → END
  - NO → P7 →
→ FinalScore → Reasoning → SaveState → END
```

### 🔍 **LangSmith Tracing (Automatic):**
Каждый node автоматически traced:
- Input state
- RAG query + retrieved docs
- Tool calls + results
- LLM prompt + response
- Output state
- Execution time
- Cost

---

## 🔄 ПОТОКИ ПЕРЕСЧЕТА - UPDATED WITH TRIGGERS

### **Сценарий 1: Изменился порт погрузки**
```
T1 (Cargo port changed) → TA1 (Full recalc) →
→ StateInit → Normalize → P1 (new proximity) →
→ P1A (new pattern) → P5 (new port restrictions) →
→ P7 (new ETA) → New Score →
→ [LangSmith: Compare with previous] →
→ Check threshold → New offer text
```

### **Сценарий 2: Owner добавил "no grains"**
```
T3 (OpenArea comment) → TA2 (Check P6) →
→ LoadCheckpoint (previous state) →
→ P6 node (RAG + parse_comments) →
→ P6 score = -40 (conflict detected) →
→ CheckP6Block: BLOCKED →
→ SaveState (is_blocked: true) → END →
→ [LangSmith: Log blocking reason] →
→ Mark vessel unsuitable → Next vessel
```

### **Сценарий 3: User corrects P1 score**
```
User: "P1 should be 8, not 15" →
→ LoadCheckpoint (full ScoringState) →
→ UpdateState (scores.p1 = 8, add reasoning) →
→ ResumeWorkflow (from P1A node) →
→ P1A → P2 → P3 → P4 → P5 → P6 → P7 →
→ FinalScore (recalculated) →
→ [LangSmith: Store correction in dataset] →
→ [LangSmith: Add to test cases] →
→ Show updated score to user
```

---

## 🧠 ОБУЧЕНИЕ СИСТЕМЫ - ENHANCED WITH LANGSMITH

### **Behavioral Cloning Workflow:**

1. **User Correction**:
   ```
   User: "P1 = 8 (was 15). Reason: Libya→BlackSea not profitable for Handy"
   ```

2. **Store in LangSmith Dataset**:
   ```python
   client.create_example(
       dataset_id="vkchart-corrections",
       inputs={
           "vessel": {...},
           "cargo": {...},
           "context": "Libya open, BlackSea loading, Handysize"
       },
       outputs={
           "expected_p1": 8,
           "reasoning": "Libya→BlackSea not profitable for Handy"
       }
   )
   ```

3. **Vector Search** (MongoDB Atlas):
   ```
   Find similar: "Libya", "BlackSea", "Handysize", "not profitable"
   → Retrieve 5 similar past corrections
   ```

4. **Update Knowledge Base**:
   ```
   Add to regional_trade_patterns.txt:
   "Libya → Black Sea (Handysize): LOW profitability due to..."
   ```

5. **Evaluation**:
   ```python
   results = evaluate(
       scoring_workflow,
       data="vkchart-corrections",  # Now includes new case
       evaluators=[scoring_accuracy]
   )
   # Measure: Did accuracy improve?
   ```

6. **A/B Testing** (LangSmith):
   ```
   - Variant A: Old prompt
   - Variant B: New prompt with correction
   - Compare: accuracy, latency, cost
   - Winner: Deploy to production
   ```

---

## ⚙️ СИСТЕМНЫЕ НАСТРОЙКИ - UPDATED WITH AI CONFIG

### **AI Configuration:**
```yaml
llm:
  primary: "claude-sonnet-4-20250514"
  backup: "gpt-4-turbo-preview"
  temperature: 0.7
  max_tokens: 1000

embeddings:
  model: "text-embedding-3-small"
  dimensions: 1536

langsmith:
  enabled: true
  project: "vkchart-ai-scoring"
  dataset: "vkchart-test-cases"
  evaluation:
    accuracy_threshold: 0.85
    regression_tests: true

langgraph:
  checkpointer: "postgresql"
  checkpoint_ttl: 604800  # 7 days
  
langserve:
  enabled: true
  streaming: true
  playground: true
```

---

## 🎯 ИНТЕГРАЦИЯ КОМПОНЕНТОВ - UPDATED

| Laravel | Python AI Service | Data Stores |
|---------|-------------------|-------------|
| Event triggers | LangGraph workflows | PostgreSQL checkpoints |
| UI approval | LangChain agents/RAG | MongoDB vectors |
| Message sending | LangServe API | MySQL entities |
| Queue management | LangSmith tracing | Learning patterns |
| Dashboard | Tool execution | Score history |

---

## 📊 МОНИТОРИНГ - ENHANCED WITH LANGSMITH

### **LangSmith Dashboard:**
- **Traces**: Full execution graph for each scoring
- **Latency**: P95 < 10s target
- **Cost**: $X per request (tracked automatically)
- **Accuracy**: 87% on test dataset (target: 85%)
- **Errors**: 2% error rate (target: <5%)

### **Alerts:**
```python
# Alert 1: High Error Rate
if error_rate > 0.05:
    notify("slack", "Error rate >5%: investigate immediately")

# Alert 2: High Latency
if p95_latency > 10000:  # 10 seconds
    notify("email", "Latency spike detected")

# Alert 3: High Cost
if daily_cost > 50:
    notify("email", "Daily LLM cost >$50")

# Alert 4: Low Accuracy
if test_accuracy < 0.85:
    notify("slack", "Accuracy dropped below 85%")
```

---

**VERSION**: 2.0 (With Full Lang* Stack Integration)  
**UPDATED**: 2024-12-02  
**STATUS**: Production-Ready Architecture
