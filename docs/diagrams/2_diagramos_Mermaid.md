# Treniruočių planavimo sistema – Mermaid diagramos

Mermaid versijos tų pačių 9 diagramų. Patikrinta su Mermaid 12.0. Mermaid neturi atskirų panaudos atvejų, komponentų, veiklos su takeliais ir diegimo diagramų tipų, todėl jos atvaizduotos per `flowchart`.

## 1. Panaudos atvejų diagrama

```mermaid
flowchart LR
    Athlete(["👤 Sportuojantysis"])
    Admin(["👤 Administratorius"])
    LLM(["⚙️ LLM paslauga «system»"])

    subgraph SYS["Treniruočių planavimo sistema"]
        UC_Register(["Registruotis"])
        UC_Login(["Prisijungti"])
        UC_Profile(["Tvarkyti profilį ir tikslus"])
        UC_Generate(["Generuoti treniruočių planą"])
        UC_Review(["Peržiūrėti ir patvirtinti planą"])
        UC_Log(["Registruoti treniruotės rezultatus"])
        UC_Adjust(["Koreguoti planą"])
        UC_Validate(["Validuoti sugeneruotą planą"])
        UC_Progress(["Peržiūrėti pažangą"])
        UC_History(["Peržiūrėti plano versijų istoriją"])
        UC_Chat(["Užduoti klausimą apie planą"])
        UC_Catalog(["Tvarkyti pratimų katalogą"])
        UC_Equipment(["Tvarkyti įrangos sąrašą"])
        UC_LlmLog(["Peržiūrėti LLM užklausų žurnalą"])
    end

    Athlete --- UC_Register
    Athlete --- UC_Login
    Athlete --- UC_Profile
    Athlete --- UC_Generate
    Athlete --- UC_Review
    Athlete --- UC_Log
    Athlete --- UC_Adjust
    Athlete --- UC_Progress
    Athlete --- UC_History
    Athlete --- UC_Chat

    Admin --- UC_Login
    Admin --- UC_Catalog
    Admin --- UC_Equipment
    Admin --- UC_LlmLog

    UC_Generate -. "«include»" .-> UC_Validate
    UC_Adjust -. "«include»" .-> UC_Validate
    UC_Adjust -. "«extend»" .-> UC_Log

    UC_Generate --- LLM
    UC_Adjust --- LLM
    UC_Chat --- LLM
```

**Kas prarandama lyginant su PlantUML:** nėra standartinio veikėjo („žmogeliuko“) simbolio ir sistemos ribos stačiakampio semantikos; veikėjai ir panaudos atvejai yra paprasti mazgai, «include»/«extend» – punktyrinės rodyklės su etiketėmis.

## 2. Domeno modelis (klasių diagrama)

```mermaid
classDiagram
    class Role {
        <<enumeration>>
        ATHLETE
        ADMIN
    }
    class Goal {
        <<enumeration>>
        MUSCLE_GAIN
        ENDURANCE
        WEIGHT_LOSS
        GENERAL_FITNESS
    }
    class FitnessLevel {
        <<enumeration>>
        BEGINNER
        INTERMEDIATE
        ADVANCED
    }
    class PlanStatus {
        <<enumeration>>
        DRAFT
        ACTIVE
        ADJUSTING
        COMPLETED
        ARCHIVED
    }
    class LlmPurpose {
        <<enumeration>>
        PLAN_GENERATION
        PLAN_ADJUSTMENT
        CHAT
    }
    class MessageSender {
        <<enumeration>>
        USER
        ASSISTANT
    }

    class User {
        UUID id
        String email
        String passwordHash
        String displayName
        Role role
        DateTime createdAt
    }
    class AthleteProfile {
        UUID id
        Goal goal
        FitnessLevel fitnessLevel
        int sessionsPerWeek
        int minutesPerSession
        String notes
        DateTime updatedAt
    }
    class Equipment {
        UUID id
        String name
    }
    class Exercise {
        UUID id
        String name
        String description
        String muscleGroup
        FitnessLevel difficulty
        boolean isActive
    }
    class TrainingPlan {
        UUID id
        String title
        PlanStatus status
        Date startDate
        int durationWeeks
        DateTime createdAt
    }
    class PlanVersion {
        UUID id
        int versionNumber
        String changeReason
        DateTime createdAt
    }
    class PlannedSession {
        UUID id
        int weekNumber
        int dayOfWeek
        String title
    }
    class PlannedExercise {
        UUID id
        int orderIndex
        int sets
        int reps
        decimal targetWeightKg
        int restSeconds
    }
    class WorkoutLog {
        UUID id
        DateTime performedAt
        int rpe
        String notes
    }
    class ExerciseResult {
        UUID id
        int setsDone
        int repsDone
        decimal weightKg
    }
    class LlmRequest {
        UUID id
        LlmPurpose purpose
        String model
        String prompt
        String rawResponse
        boolean isValid
        String validationErrors
        int attemptNumber
        int tokensUsed
        DateTime createdAt
    }
    class ChatMessage {
        UUID id
        MessageSender sender
        String content
        DateTime createdAt
    }

    User "1" -- "0..1" AthleteProfile
    AthleteProfile "*" -- "*" Equipment : turi prieinamą
    Exercise "*" -- "*" Equipment : reikalauja
    User "1" -- "*" TrainingPlan
    TrainingPlan "1" *-- "1..*" PlanVersion
    PlanVersion "1" *-- "1..*" PlannedSession
    PlannedSession "1" *-- "1..*" PlannedExercise
    PlannedExercise "*" --> "1" Exercise
    User "1" -- "*" WorkoutLog
    WorkoutLog "*" --> "1" PlannedSession
    WorkoutLog "1" *-- "1..*" ExerciseResult
    ExerciseResult "*" --> "1" PlannedExercise
    PlanVersion "*" --> "0..1" WorkoutLog : inicijuota
    PlanVersion "*" --> "0..1" LlmRequest : sugeneruota
    User "1" -- "*" LlmRequest
    User "1" -- "*" ChatMessage
    ChatMessage "*" --> "0..1" TrainingPlan : susijusi su
```

**Kas prarandama:** praktiškai nieko – Mermaid classDiagram palaiko kardinalumus, kompoziciją ir enumeracijas. Ryšių pavadinimų krypties rodyklių (►) nėra.

## 3. Duomenų bazės ER diagrama

```mermaid
erDiagram
    users {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar display_name
        varchar role
        timestamp created_at
    }
    athlete_profiles {
        uuid id PK
        uuid user_id FK,UK
        varchar goal
        varchar fitness_level
        smallint sessions_per_week
        smallint minutes_per_session
        text notes "nullable"
        timestamp updated_at
    }
    equipment {
        uuid id PK
        varchar name UK
    }
    profile_equipment {
        uuid profile_id PK,FK
        uuid equipment_id PK,FK
    }
    exercises {
        uuid id PK
        varchar name UK
        text description "nullable"
        varchar muscle_group
        varchar difficulty
        boolean is_active
    }
    exercise_equipment {
        uuid exercise_id PK,FK
        uuid equipment_id PK,FK
    }
    training_plans {
        uuid id PK
        uuid user_id FK
        varchar title
        varchar status
        date start_date
        smallint duration_weeks
        timestamp created_at
    }
    plan_versions {
        uuid id PK
        uuid plan_id FK
        int version_number
        text change_reason "nullable"
        uuid triggered_by_log_id FK "nullable"
        uuid llm_request_id FK "nullable"
        timestamp created_at
    }
    planned_sessions {
        uuid id PK
        uuid plan_version_id FK
        smallint week_number
        smallint day_of_week
        varchar title
    }
    planned_exercises {
        uuid id PK
        uuid session_id FK
        uuid exercise_id FK
        smallint order_index
        smallint sets
        smallint reps
        numeric target_weight_kg "nullable"
        smallint rest_seconds
    }
    workout_logs {
        uuid id PK
        uuid user_id FK
        uuid planned_session_id FK
        timestamp performed_at
        smallint rpe
        text notes "nullable"
    }
    exercise_results {
        uuid id PK
        uuid workout_log_id FK
        uuid planned_exercise_id FK
        smallint sets_done
        smallint reps_done
        numeric weight_kg "nullable"
    }
    llm_requests {
        uuid id PK
        uuid user_id FK
        varchar purpose
        varchar model
        text prompt
        text raw_response "nullable"
        boolean is_valid
        text validation_errors "nullable"
        smallint attempt_number
        int tokens_used "nullable"
        timestamp created_at
    }
    chat_messages {
        uuid id PK
        uuid user_id FK
        uuid plan_id FK "nullable"
        varchar sender
        text content
        timestamp created_at
    }

    users ||--o| athlete_profiles : turi
    athlete_profiles ||--o{ profile_equipment : turi
    equipment ||--o{ profile_equipment : priskirta
    exercises ||--o{ exercise_equipment : reikalauja
    equipment ||--o{ exercise_equipment : naudojama
    users ||--o{ training_plans : turi
    training_plans ||--|{ plan_versions : turi
    plan_versions ||--|{ planned_sessions : sudaro
    planned_sessions ||--|{ planned_exercises : sudaro
    exercises ||--o{ planned_exercises : naudojamas
    users ||--o{ workout_logs : atlieka
    planned_sessions ||--o{ workout_logs : registruojama
    workout_logs ||--|{ exercise_results : turi
    planned_exercises ||--o{ exercise_results : vertinama
    workout_logs |o--o{ plan_versions : inicijuoja
    llm_requests |o--o{ plan_versions : sugeneruoja
    users ||--o{ llm_requests : inicijuoja
    users ||--o{ chat_messages : rašo
    training_plans |o--o{ chat_messages : aptariamas
```

**Kas prarandama:** Mermaid atributų tipai negali turėti parametrų, todėl `varchar(255)`, `numeric(6,2)` sutrumpinti iki `varchar`, `numeric`. NOT NULL žymėjimas (`*`) pakeistas komentaru „nullable“ prie neprivalomų laukų. Mermaid reikalauja ryšio etiketės, todėl pridėti ryšių pavadinimai.

## 4. Komponentų diagrama

```mermaid
flowchart TB
    U(["👤 Vartotojas"])

    subgraph FE["Frontend (React SPA)"]
        UI["[component] Web UI"]
    end

    subgraph BE["Backend"]
        REST(("REST API"))
        API["[component] API sluoksnis (REST controllers)"]
        Auth["[component] AuthService"]
        ProfileSvc["[component] ProfileService"]
        PlanSvc["[component] PlanService"]
        WorkoutSvc["[component] WorkoutService"]
        Analyzer["[component] ProgressAnalyzer"]
        ChatSvc["[component] ChatService"]
        CatalogSvc["[component] ExerciseCatalogService"]
        subgraph LLMINT["LLM integracija"]
            Prompt["[component] PromptBuilder"]
            Gateway["[component] LlmGateway"]
            Validator["[component] ResponseValidator (JSON Schema + domeno taisyklės)"]
        end
        Repo["[component] Repositories (ORM)"]
    end

    DB[("PostgreSQL")]
    LLMAPI{{"☁️ LLM API (išorinis tiekėjas)"}}

    U --> UI
    UI -->|"HTTPS, JSON, JWT"| REST
    REST --- API
    API --> Auth
    API --> ProfileSvc
    API --> PlanSvc
    API --> WorkoutSvc
    API --> ChatSvc
    API --> CatalogSvc
    PlanSvc --> CatalogSvc
    PlanSvc --> Prompt
    PlanSvc --> Gateway
    PlanSvc --> Validator
    WorkoutSvc --> Analyzer
    WorkoutSvc -->|"inicijuoja koregavimą"| PlanSvc
    ChatSvc --> Prompt
    ChatSvc --> Gateway
    Validator -->|"pratimų ID patikra"| CatalogSvc
    Gateway -->|"HTTPS"| LLMAPI
    Gateway -->|"LLM užklausų žurnalas"| Repo
    Auth --> Repo
    ProfileSvc --> Repo
    PlanSvc --> Repo
    WorkoutSvc --> Repo
    Analyzer --> Repo
    ChatSvc --> Repo
    CatalogSvc --> Repo
    Repo -->|"SQL"| DB
```

**Kas prarandama:** nėra UML komponento simbolio ir „lollipop“ sąsajos; komponentai pažymėti tekstu `[component]`, REST API sąsaja – apskritimu, LLM API – šešiakampiu (cloud forma yra tik naujausiose Mermaid versijose).

## 5. Sekų diagrama: plano generavimas

```mermaid
sequenceDiagram
    autonumber
    actor User as Sportuojantysis
    participant UI as Web UI
    participant Ctrl as PlanController
    participant PlanSvc as PlanService
    participant Catalog as ExerciseCatalogService
    participant Prompt as PromptBuilder
    participant Gateway as LlmGateway
    participant Validator as ResponseValidator
    participant DB as PostgreSQL
    participant LLM as LLM API

    User->>UI: Spaudžia „Generuoti planą“
    UI->>Ctrl: POST /api/plans
    activate Ctrl
    Ctrl->>PlanSvc: generatePlan(userId)
    activate PlanSvc
    PlanSvc->>DB: SELECT profilis ir įranga
    DB-->>PlanSvc: AthleteProfile
    PlanSvc->>Catalog: findSuitable(fitnessLevel, equipment)
    Catalog->>DB: SELECT exercises
    DB-->>Catalog: pratimai
    Catalog-->>PlanSvc: tinkamų pratimų sąrašas
    PlanSvc->>Prompt: buildGenerationPrompt(profile, exercises, jsonSchema)
    Prompt-->>PlanSvc: prompt

    loop iki 3 bandymų, kol gaunamas validus atsakymas
        PlanSvc->>Gateway: complete(prompt)
        activate Gateway
        Gateway->>LLM: užklausa (struktūrizuotos išvesties režimas)
        LLM-->>Gateway: JSON atsakymas
        Gateway->>DB: INSERT llm_requests
        Gateway-->>PlanSvc: atsakymas
        deactivate Gateway
        PlanSvc->>Validator: validate(json)
        activate Validator
        Validator->>Validator: JSON Schema patikra
        Validator->>Catalog: ar visi exerciseId egzistuoja ir tinka?
        Catalog-->>Validator: patikros rezultatas
        Validator->>Validator: krūvio ribų patikra
        Validator-->>PlanSvc: ValidationResult
        deactivate Validator
        opt validacija nepavyko
            PlanSvc->>Prompt: addValidationErrors(prompt, errors)
            Prompt-->>PlanSvc: pataisytas prompt
        end
    end

    alt planas validus
        PlanSvc->>DB: INSERT plan (DRAFT), version v1, sessions, exercises (viena transakcija)
        DB-->>PlanSvc: OK
        PlanSvc-->>Ctrl: TrainingPlan
        Ctrl-->>UI: 201 Created
        UI-->>User: rodo plano juodraštį
    else visi 3 bandymai nepavyko
        PlanSvc-->>Ctrl: PlanGenerationException
        Ctrl-->>UI: 502 Bad Gateway
        UI-->>User: klaidos pranešimas
    end
    deactivate PlanSvc
    deactivate Ctrl
```

**Kas prarandama:** nieko esminio – sequenceDiagram palaiko activate, loop, opt, alt. Duomenų bazė rodoma kaip įprastas dalyvis, o ne cilindras.

## 6. Sekų diagrama: plano koregavimas

```mermaid
sequenceDiagram
    autonumber
    actor User as Sportuojantysis
    participant UI as Web UI
    participant Ctrl as WorkoutController
    participant WorkoutSvc as WorkoutService
    participant Analyzer as ProgressAnalyzer
    participant PlanSvc as PlanService
    participant Prompt as PromptBuilder
    participant Gateway as LlmGateway
    participant Validator as ResponseValidator
    participant DB as PostgreSQL
    participant LLM as LLM API

    User->>UI: Įveda atliktus pratimus ir RPE
    UI->>Ctrl: POST /api/workouts
    activate Ctrl
    Ctrl->>WorkoutSvc: logWorkout(dto)
    activate WorkoutSvc
    WorkoutSvc->>DB: INSERT workout_logs, exercise_results
    DB-->>WorkoutSvc: workoutLogId
    WorkoutSvc->>Analyzer: analyze(userId, planId)
    activate Analyzer
    Analyzer->>DB: SELECT paskutinės treniruotės ir aktyvi plano versija
    DB-->>Analyzer: duomenys
    Analyzer->>Analyzer: planuota vs faktiška, RPE tendencija
    Analyzer-->>WorkoutSvc: AnalysisResult(needsAdjustment, reasons)
    deactivate Analyzer

    alt reikia koreguoti planą
        WorkoutSvc->>PlanSvc: adjustPlan(planId, analysis, workoutLogId)
        activate PlanSvc
        PlanSvc->>DB: UPDATE status = ADJUSTING
        PlanSvc->>Prompt: buildAdjustmentPrompt(currentVersion, recentLogs, analysis)
        Prompt-->>PlanSvc: prompt
        loop iki 3 bandymų, kol gaunamas validus atsakymas
            PlanSvc->>Gateway: complete(prompt)
            activate Gateway
            Gateway->>LLM: užklausa
            LLM-->>Gateway: JSON atsakymas
            Gateway->>DB: INSERT llm_requests
            Gateway-->>PlanSvc: atsakymas
            deactivate Gateway
            PlanSvc->>Validator: validate(json)
            Validator-->>PlanSvc: ValidationResult
        end
        alt nauja versija validi
            PlanSvc->>DB: INSERT plan_versions (n+1, changeReason, triggeredByLogId)
            PlanSvc->>DB: UPDATE status = ACTIVE
            PlanSvc-->>WorkoutSvc: pakeitimų santrauka
        else koregavimas nepavyko
            PlanSvc->>DB: UPDATE status = ACTIVE (lieka ankstesnė versija)
            PlanSvc-->>WorkoutSvc: koregavimas atidėtas
        end
        deactivate PlanSvc
    else korekcija nereikalinga
        Note over WorkoutSvc: planas lieka nepakeistas
    end

    WorkoutSvc-->>Ctrl: WorkoutSummary
    deactivate WorkoutSvc
    Ctrl-->>UI: 201 Created
    deactivate Ctrl
    UI-->>User: rezultatai išsaugoti ir plano pakeitimai, jei buvo
```

**Kas prarandama:** nieko esminio.

## 7. Veiklos diagrama: vartotojo ciklas

```mermaid
flowchart TD
    Start((" ")) --> A1["Registruojasi sistemoje"]
    A1 --> A2["Užpildo profilį ir tikslus"]
    A2 --> A3["Inicijuoja plano generavimą"]
    A3 --> S1["Parenka tinkamus pratimus iš katalogo"]
    S1 --> S2["Sudaro LLM užklausą"]
    S2 --> L1["Sugeneruoja planą JSON formatu"]
    L1 --> S3["Validuoja atsakymą"]
    S3 --> D1{"Nevalidus ir bandymų < 3?"}
    D1 -->|taip| L1
    D1 -->|ne| D2{"Atsakymas validus?"}
    D2 -->|ne| S4["Rodo klaidos pranešimą"]
    S4 --> EndErr(((" ")))
    D2 -->|taip| S5["Išsaugo planą (DRAFT)"]
    S5 --> A4["Peržiūri plano juodraštį"]
    A4 --> D3{"Planas netinka?"}
    D3 -->|taip| S6["Archyvuoja atmestą juodraštį"]
    S6 --> A3
    D3 -->|ne| A5["Patvirtina planą"]
    A5 --> S7["Pakeičia būseną į ACTIVE"]
    S7 --> A6["Atlieka treniruotę"]
    A6 --> A7["Įveda rezultatus ir RPE"]
    A7 --> S8["Išsaugo rezultatus"]
    S8 --> S9["Analizuoja pažangą"]
    S9 --> D4{"Reikia koreguoti planą?"}
    D4 -->|taip| L2["Sugeneruoja pataisytą planą"]
    L2 --> S10["Validuoja ir išsaugo naują versiją"]
    S10 --> D5{"Liko neatliktų treniruočių?"}
    D4 -->|ne| D5
    D5 -->|taip| A6
    D5 -->|ne| S11["Pažymi planą COMPLETED"]
    S11 --> EndOk(((" ")))

    subgraph Legenda
        direction LR
        LA["Sportuojantysis"]
        LS["Sistema"]
        LL["LLM paslauga"]
    end

    classDef athlete fill:#dbeafe,stroke:#1d4ed8,color:#000
    classDef system fill:#dcfce7,stroke:#15803d,color:#000
    classDef llm fill:#fef3c7,stroke:#b45309,color:#000
    classDef terminal fill:#000,stroke:#000
    class A1,A2,A3,A4,A5,A6,A7,LA athlete
    class S1,S2,S3,S4,S5,S6,S7,S8,S9,S10,S11,LS system
    class L1,L2,LL llm
    class Start,EndErr,EndOk terminal
```

**Kas prarandama:** Mermaid neturi takelių (swimlanes), todėl atsakomybė parodyta spalvomis (žr. legendą): mėlyna – sportuojantysis, žalia – sistema, geltona – LLM paslauga. Pradžios ir pabaigos mazgai – juodi apskritimai.

## 8. Būsenų diagrama: treniruočių plano gyvavimo ciklas

```mermaid
stateDiagram-v2
    [*] --> DRAFT : planas sugeneruotas ir validuotas
    DRAFT --> ACTIVE : vartotojas patvirtina
    DRAFT --> ARCHIVED : vartotojas atmeta
    ACTIVE --> ADJUSTING : analizė rodo koregavimo poreikį
    ADJUSTING --> ACTIVE : nauja versija validi [versija + 1] / koregavimas nepavyko [lieka ankstesnė versija]
    ACTIVE --> COMPLETED : atliktos visos treniruotės
    ACTIVE --> ARCHIVED : vartotojas nutraukia planą
    COMPLETED --> ARCHIVED : sukuriamas naujas planas
    ARCHIVED --> [*]

    note right of ACTIVE
        Vartotojas vienu metu gali turėti
        tik vieną ACTIVE būsenos planą
    end note
```

**Kas prarandama:** du perėjimai ADJUSTING → ACTIVE sujungti į vieną su dviem sąlygomis, nes Mermaid lygiagrečias rodykles tarp tų pačių būsenų piešia viena ant kitos.

## 9. Diegimo diagrama

```mermaid
flowchart LR
    subgraph ClientDevice["«device» Vartotojo įrenginys"]
        Browser["Naršyklė"]
    end

    subgraph Server["«device» Aplikacijų serveris (Linux)"]
        subgraph Docker["«executionEnvironment» Docker Engine"]
            subgraph Nginx["nginx konteineris"]
                FeArt["«artifact» frontend build (React)"]
            end
            subgraph ApiNode["api konteineris"]
                BeArt["«artifact» backend API"]
            end
            subgraph PgNode["postgres konteineris"]
                TrainingDb[("training_db")]
            end
        end
    end

    subgraph Provider["☁️ LLM tiekėjas"]
        LlmNode["LLM API"]
    end

    Browser -->|"HTTPS :443"| Nginx
    Nginx -->|"HTTP :8080 (/api reverse proxy)"| ApiNode
    ApiNode -->|"TCP :5432"| PgNode
    ApiNode -->|"HTTPS :443"| LlmNode
```

**Kas prarandama:** nėra 3D mazgų (node) simbolių; mazgai ir vykdymo aplinkos parodyti įdėtais subgraph blokais su stereotipais tekste.
