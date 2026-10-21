# Treniruočių planavimo sistema – PlantUML diagramos

Techniniai pavadinimai (klasės, lentelės, komponentai, API) – anglų kalba, aprašomieji tekstai – lietuvių kalba. Visos diagramos patikrintos su PlantUML 1.2025.4.

## 1. Panaudos atvejų diagrama

Dvi pagrindinės rolės (sportuojantysis, administratorius) ir LLM paslauga kaip antrinis veikėjas. Plano generavimas ir koregavimas įtraukia (include) bendrą validavimo atvejį, o plano koregavimas išplečia (extend) rezultatų registravimą, nes vyksta tik tada, kai analizė parodo poreikį.

```plantuml
@startuml UseCase
left to right direction
skinparam packageStyle rectangle
actor "Sportuojantysis" as Athlete
actor "Administratorius" as Admin
actor "LLM paslauga" as LLM <<system>>

rectangle "Treniruočių planavimo sistema" {
  usecase "Registruotis" as UC_Register
  usecase "Prisijungti" as UC_Login
  usecase "Tvarkyti profilį ir tikslus" as UC_Profile
  usecase "Generuoti treniruočių planą" as UC_Generate
  usecase "Peržiūrėti ir patvirtinti planą" as UC_Review
  usecase "Registruoti treniruotės rezultatus" as UC_Log
  usecase "Koreguoti planą" as UC_Adjust
  usecase "Validuoti sugeneruotą planą" as UC_Validate
  usecase "Peržiūrėti pažangą" as UC_Progress
  usecase "Peržiūrėti plano versijų istoriją" as UC_History
  usecase "Užduoti klausimą apie planą" as UC_Chat
  usecase "Tvarkyti pratimų katalogą" as UC_Catalog
  usecase "Tvarkyti įrangos sąrašą" as UC_Equipment
  usecase "Peržiūrėti LLM užklausų žurnalą" as UC_LlmLog
}

Athlete --> UC_Register
Athlete --> UC_Login
Athlete --> UC_Profile
Athlete --> UC_Generate
Athlete --> UC_Review
Athlete --> UC_Log
Athlete --> UC_Adjust
Athlete --> UC_Progress
Athlete --> UC_History
Athlete --> UC_Chat

Admin --> UC_Login
Admin --> UC_Catalog
Admin --> UC_Equipment
Admin --> UC_LlmLog

UC_Generate ..> UC_Validate : <<include>>
UC_Adjust ..> UC_Validate : <<include>>
UC_Adjust ..> UC_Log : <<extend>>

UC_Generate --> LLM
UC_Adjust --> LLM
UC_Chat --> LLM
@enduml
```

## 2. Domeno modelis (klasių diagrama)

Planas turi versijas (PlanVersion), todėl saugoma pilna koregavimų istorija: kiekviena versija žino, kuri treniruotė (WorkoutLog) ją inicijavo ir kuri LLM užklausa ją sugeneravo. Pratimai planuose visada nurodo katalogo įrašą (Exercise), o ne laisvą tekstą.

```plantuml
@startuml DomainModel
skinparam classAttributeIconSize 0
hide empty methods

enum Role {
  ATHLETE
  ADMIN
}
enum Goal {
  MUSCLE_GAIN
  ENDURANCE
  WEIGHT_LOSS
  GENERAL_FITNESS
}
enum FitnessLevel {
  BEGINNER
  INTERMEDIATE
  ADVANCED
}
enum PlanStatus {
  DRAFT
  ACTIVE
  ADJUSTING
  COMPLETED
  ARCHIVED
}
enum LlmPurpose {
  PLAN_GENERATION
  PLAN_ADJUSTMENT
  CHAT
}
enum MessageSender {
  USER
  ASSISTANT
}

class User {
  id: UUID
  email: String
  passwordHash: String
  displayName: String
  role: Role
  createdAt: DateTime
}
class AthleteProfile {
  id: UUID
  goal: Goal
  fitnessLevel: FitnessLevel
  sessionsPerWeek: int
  minutesPerSession: int
  notes: String
  updatedAt: DateTime
}
class Equipment {
  id: UUID
  name: String
}
class Exercise {
  id: UUID
  name: String
  description: String
  muscleGroup: String
  difficulty: FitnessLevel
  isActive: boolean
}
class TrainingPlan {
  id: UUID
  title: String
  status: PlanStatus
  startDate: Date
  durationWeeks: int
  createdAt: DateTime
}
class PlanVersion {
  id: UUID
  versionNumber: int
  changeReason: String
  createdAt: DateTime
}
class PlannedSession {
  id: UUID
  weekNumber: int
  dayOfWeek: int
  title: String
}
class PlannedExercise {
  id: UUID
  orderIndex: int
  sets: int
  reps: int
  targetWeightKg: decimal
  restSeconds: int
}
class WorkoutLog {
  id: UUID
  performedAt: DateTime
  rpe: int
  notes: String
}
class ExerciseResult {
  id: UUID
  setsDone: int
  repsDone: int
  weightKg: decimal
}
class LlmRequest {
  id: UUID
  purpose: LlmPurpose
  model: String
  prompt: String
  rawResponse: String
  isValid: boolean
  validationErrors: String
  attemptNumber: int
  tokensUsed: int
  createdAt: DateTime
}
class ChatMessage {
  id: UUID
  sender: MessageSender
  content: String
  createdAt: DateTime
}

User "1" -- "0..1" AthleteProfile
AthleteProfile "*" -- "*" Equipment : turi prieinamą >
Exercise "*" -- "*" Equipment : reikalauja >
User "1" -- "*" TrainingPlan
TrainingPlan "1" *-- "1..*" PlanVersion
PlanVersion "1" *-- "1..*" PlannedSession
PlannedSession "1" *-- "1..*" PlannedExercise
PlannedExercise "*" --> "1" Exercise
User "1" -- "*" WorkoutLog
WorkoutLog "*" --> "1" PlannedSession
WorkoutLog "1" *-- "1..*" ExerciseResult
ExerciseResult "*" --> "1" PlannedExercise
PlanVersion "*" --> "0..1" WorkoutLog : inicijuota >
PlanVersion "*" --> "0..1" LlmRequest : sugeneruota >
User "1" -- "*" LlmRequest
User "1" -- "*" ChatMessage
ChatMessage "*" --> "0..1" TrainingPlan : susijusi su >
@enduml
```

## 3. Duomenų bazės ER diagrama

Fizinis modelis PostgreSQL duomenų bazei. Ryšiai daug-su-daug (profilis–įranga, pratimas–įranga) realizuoti tarpinėmis lentelėmis. Lentelė llm_requests saugo kiekvieną LLM kvietimą su atsakymu ir validavimo klaidomis, tai naudinga derinimui ir vertinimui.

```plantuml
@startuml ERDiagram
hide circle
skinparam linetype ortho

entity users {
  * id : uuid <<PK>>
  --
  * email : varchar(255) <<UK>>
  * password_hash : varchar(255)
  * display_name : varchar(100)
  * role : varchar(20)
  * created_at : timestamp
}
entity athlete_profiles {
  * id : uuid <<PK>>
  --
  * user_id : uuid <<FK, UK>>
  * goal : varchar(30)
  * fitness_level : varchar(20)
  * sessions_per_week : smallint
  * minutes_per_session : smallint
  notes : text
  * updated_at : timestamp
}
entity equipment {
  * id : uuid <<PK>>
  --
  * name : varchar(100) <<UK>>
}
entity profile_equipment {
  * profile_id : uuid <<PK, FK>>
  * equipment_id : uuid <<PK, FK>>
}
entity exercises {
  * id : uuid <<PK>>
  --
  * name : varchar(150) <<UK>>
  description : text
  * muscle_group : varchar(50)
  * difficulty : varchar(20)
  * is_active : boolean
}
entity exercise_equipment {
  * exercise_id : uuid <<PK, FK>>
  * equipment_id : uuid <<PK, FK>>
}
entity training_plans {
  * id : uuid <<PK>>
  --
  * user_id : uuid <<FK>>
  * title : varchar(150)
  * status : varchar(20)
  * start_date : date
  * duration_weeks : smallint
  * created_at : timestamp
}
entity plan_versions {
  * id : uuid <<PK>>
  --
  * plan_id : uuid <<FK>>
  * version_number : int
  change_reason : text
  triggered_by_log_id : uuid <<FK>>
  llm_request_id : uuid <<FK>>
  * created_at : timestamp
}
entity planned_sessions {
  * id : uuid <<PK>>
  --
  * plan_version_id : uuid <<FK>>
  * week_number : smallint
  * day_of_week : smallint
  * title : varchar(100)
}
entity planned_exercises {
  * id : uuid <<PK>>
  --
  * session_id : uuid <<FK>>
  * exercise_id : uuid <<FK>>
  * order_index : smallint
  * sets : smallint
  * reps : smallint
  target_weight_kg : numeric(6,2)
  * rest_seconds : smallint
}
entity workout_logs {
  * id : uuid <<PK>>
  --
  * user_id : uuid <<FK>>
  * planned_session_id : uuid <<FK>>
  * performed_at : timestamp
  * rpe : smallint
  notes : text
}
entity exercise_results {
  * id : uuid <<PK>>
  --
  * workout_log_id : uuid <<FK>>
  * planned_exercise_id : uuid <<FK>>
  * sets_done : smallint
  * reps_done : smallint
  weight_kg : numeric(6,2)
}
entity llm_requests {
  * id : uuid <<PK>>
  --
  * user_id : uuid <<FK>>
  * purpose : varchar(30)
  * model : varchar(100)
  * prompt : text
  raw_response : text
  * is_valid : boolean
  validation_errors : text
  * attempt_number : smallint
  tokens_used : int
  * created_at : timestamp
}
entity chat_messages {
  * id : uuid <<PK>>
  --
  * user_id : uuid <<FK>>
  plan_id : uuid <<FK>>
  * sender : varchar(20)
  * content : text
  * created_at : timestamp
}

users ||--o| athlete_profiles
athlete_profiles ||--o{ profile_equipment
equipment ||--o{ profile_equipment
exercises ||--o{ exercise_equipment
equipment ||--o{ exercise_equipment
users ||--o{ training_plans
training_plans ||--|{ plan_versions
plan_versions ||--|{ planned_sessions
planned_sessions ||--|{ planned_exercises
exercises ||--o{ planned_exercises
users ||--o{ workout_logs
planned_sessions ||--o{ workout_logs
workout_logs ||--|{ exercise_results
planned_exercises ||--o{ exercise_results
workout_logs |o--o{ plan_versions
llm_requests |o--o{ plan_versions
users ||--o{ llm_requests
users ||--o{ chat_messages
training_plans |o--o{ chat_messages
@enduml
```

## 4. Komponentų diagrama

LLM integracija atskirta į tris komponentus: PromptBuilder sudaro užklausas, LlmGateway bendrauja su išoriniu API ir registruoja kvietimus, ResponseValidator tikrina JSON pagal schemą ir domeno taisykles (pratimų ID, krūvio ribas). Taip LLM tiekėją galima pakeisti nekeičiant verslo logikos.

```plantuml
@startuml Components
skinparam componentStyle uml2

actor "Vartotojas" as U

package "Frontend (React SPA)" {
  component "Web UI" as UI
}

package "Backend" {
  interface "REST API" as REST
  component "API sluoksnis\n(REST controllers)" as API
  component "AuthService" as Auth
  component "ProfileService" as ProfileSvc
  component "PlanService" as PlanSvc
  component "WorkoutService" as WorkoutSvc
  component "ProgressAnalyzer" as Analyzer
  component "ChatService" as ChatSvc
  component "ExerciseCatalogService" as CatalogSvc
  package "LLM integracija" {
    component "PromptBuilder" as Prompt
    component "LlmGateway" as Gateway
    component "ResponseValidator\n(JSON Schema + domeno taisyklės)" as Validator
  }
  component "Repositories (ORM)" as Repo
}

database "PostgreSQL" as DB
cloud "LLM API\n(išorinis tiekėjas)" as LLMAPI

U --> UI
UI --> REST : HTTPS, JSON, JWT
REST - API
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
WorkoutSvc --> PlanSvc : inicijuoja koregavimą
ChatSvc --> Prompt
ChatSvc --> Gateway
Validator --> CatalogSvc : pratimų ID patikra
Gateway --> LLMAPI : HTTPS
Gateway --> Repo : LLM užklausų žurnalas
Auth --> Repo
ProfileSvc --> Repo
PlanSvc --> Repo
WorkoutSvc --> Repo
Analyzer --> Repo
ChatSvc --> Repo
CatalogSvc --> Repo
Repo --> DB : SQL
@enduml
```

## 5. Sekų diagrama: plano generavimas

LLM gauna tik tuos pratimus, kurie tinka vartotojo lygiui ir įrangai. Jei atsakymas nevalidus, validavimo klaidos pridedamos prie užklausos ir bandoma dar kartą (iki 3 kartų). Planas išsaugomas vienoje transakcijoje.

```plantuml
@startuml SeqGeneratePlan
autonumber
actor "Sportuojantysis" as User
participant "Web UI" as UI
participant "PlanController" as Ctrl
participant "PlanService" as PlanSvc
participant "ExerciseCatalogService" as Catalog
participant "PromptBuilder" as Prompt
participant "LlmGateway" as Gateway
participant "ResponseValidator" as Validator
database "PostgreSQL" as DB
participant "LLM API" as LLM

User -> UI : Spaudžia „Generuoti planą“
UI -> Ctrl : POST /api/plans
activate Ctrl
Ctrl -> PlanSvc : generatePlan(userId)
activate PlanSvc
PlanSvc -> DB : SELECT profilis ir įranga
DB --> PlanSvc : AthleteProfile
PlanSvc -> Catalog : findSuitable(fitnessLevel, equipment)
Catalog -> DB : SELECT exercises
DB --> Catalog : pratimai
Catalog --> PlanSvc : tinkamų pratimų sąrašas
PlanSvc -> Prompt : buildGenerationPrompt(profile, exercises, jsonSchema)
Prompt --> PlanSvc : prompt

loop iki 3 bandymų, kol gaunamas validus atsakymas
  PlanSvc -> Gateway : complete(prompt)
  activate Gateway
  Gateway -> LLM : užklausa (struktūrizuotos išvesties režimas)
  LLM --> Gateway : JSON atsakymas
  Gateway -> DB : INSERT llm_requests
  Gateway --> PlanSvc : atsakymas
  deactivate Gateway
  PlanSvc -> Validator : validate(json)
  activate Validator
  Validator -> Validator : JSON Schema patikra
  Validator -> Catalog : ar visi exerciseId egzistuoja ir tinka?
  Catalog --> Validator : patikros rezultatas
  Validator -> Validator : krūvio ribų patikra
  Validator --> PlanSvc : ValidationResult
  deactivate Validator
  opt validacija nepavyko
    PlanSvc -> Prompt : addValidationErrors(prompt, errors)
    Prompt --> PlanSvc : pataisytas prompt
  end
end

alt planas validus
  PlanSvc -> DB : INSERT plan (DRAFT), version v1, sessions, exercises (viena transakcija)
  DB --> PlanSvc : OK
  PlanSvc --> Ctrl : TrainingPlan
  Ctrl --> UI : 201 Created
  UI --> User : rodo plano juodraštį
else visi 3 bandymai nepavyko
  PlanSvc --> Ctrl : PlanGenerationException
  Ctrl --> UI : 502 Bad Gateway
  UI --> User : klaidos pranešimas
end
deactivate PlanSvc
deactivate Ctrl
@enduml
```

## 6. Sekų diagrama: plano koregavimas

ProgressAnalyzer deterministiškai (be LLM) nustato, ar reikia koreguoti planą, pvz., pagal RPE tendenciją ar neatliktus pakartojimus. LLM kviečiamas tik tada, kai korekcija reikalinga. Jei koregavimas nepavyksta, planas lieka su ankstesne versija.

```plantuml
@startuml SeqAdjustPlan
autonumber
actor "Sportuojantysis" as User
participant "Web UI" as UI
participant "WorkoutController" as Ctrl
participant "WorkoutService" as WorkoutSvc
participant "ProgressAnalyzer" as Analyzer
participant "PlanService" as PlanSvc
participant "PromptBuilder" as Prompt
participant "LlmGateway" as Gateway
participant "ResponseValidator" as Validator
database "PostgreSQL" as DB
participant "LLM API" as LLM

User -> UI : Įveda atliktus pratimus ir RPE
UI -> Ctrl : POST /api/workouts
activate Ctrl
Ctrl -> WorkoutSvc : logWorkout(dto)
activate WorkoutSvc
WorkoutSvc -> DB : INSERT workout_logs, exercise_results
DB --> WorkoutSvc : workoutLogId
WorkoutSvc -> Analyzer : analyze(userId, planId)
activate Analyzer
Analyzer -> DB : SELECT paskutinės treniruotės ir aktyvi plano versija
DB --> Analyzer : duomenys
Analyzer -> Analyzer : planuota vs faktiška, RPE tendencija
Analyzer --> WorkoutSvc : AnalysisResult(needsAdjustment, reasons)
deactivate Analyzer

alt reikia koreguoti planą
  WorkoutSvc -> PlanSvc : adjustPlan(planId, analysis, workoutLogId)
  activate PlanSvc
  PlanSvc -> DB : UPDATE status = ADJUSTING
  PlanSvc -> Prompt : buildAdjustmentPrompt(currentVersion, recentLogs, analysis)
  Prompt --> PlanSvc : prompt
  loop iki 3 bandymų, kol gaunamas validus atsakymas
    PlanSvc -> Gateway : complete(prompt)
    activate Gateway
    Gateway -> LLM : užklausa
    LLM --> Gateway : JSON atsakymas
    Gateway -> DB : INSERT llm_requests
    Gateway --> PlanSvc : atsakymas
    deactivate Gateway
    PlanSvc -> Validator : validate(json)
    Validator --> PlanSvc : ValidationResult
  end
  alt nauja versija validi
    PlanSvc -> DB : INSERT plan_versions (n+1, changeReason, triggeredByLogId)
    PlanSvc -> DB : UPDATE status = ACTIVE
    PlanSvc --> WorkoutSvc : pakeitimų santrauka
  else koregavimas nepavyko
    PlanSvc -> DB : UPDATE status = ACTIVE (lieka ankstesnė versija)
    PlanSvc --> WorkoutSvc : koregavimas atidėtas
  end
  deactivate PlanSvc
else korekcija nereikalinga
  note over WorkoutSvc : planas lieka nepakeistas
end

WorkoutSvc --> Ctrl : WorkoutSummary
deactivate WorkoutSvc
Ctrl --> UI : 201 Created
deactivate Ctrl
UI --> User : rezultatai išsaugoti ir plano pakeitimai, jei buvo
@enduml
```

## 7. Veiklos diagrama: vartotojo ciklas

Takeliai (swimlanes) rodo, kas atlieka kiekvieną veiksmą: vartotojas, sistema ar LLM paslauga. Atmetus juodraštį, jis archyvuojamas ir generuojamas naujas planas.

```plantuml
@startuml Activity
|Sportuojantysis|
start
:Registruojasi sistemoje;
:Užpildo profilį ir tikslus;
repeat
  :Inicijuoja plano generavimą;
  |Sistema|
  :Parenka tinkamus pratimus iš katalogo;
  :Sudaro LLM užklausą;
  repeat
    |LLM paslauga|
    :Sugeneruoja planą JSON formatu;
    |Sistema|
    :Validuoja atsakymą;
  repeat while (Nevalidus ir bandymų < 3?) is (taip) not (ne)
  if (Atsakymas validus?) then (ne)
    :Rodo klaidos pranešimą;
    stop
  endif
  :Išsaugo planą (DRAFT);
  |Sportuojantysis|
  :Peržiūri plano juodraštį;
  backward :Archyvuoja atmestą juodraštį;
repeat while (Planas netinka?) is (taip) not (ne)
:Patvirtina planą;
|Sistema|
:Pakeičia būseną į ACTIVE;
repeat
  |Sportuojantysis|
  :Atlieka treniruotę;
  :Įveda rezultatus ir RPE;
  |Sistema|
  :Išsaugo rezultatus;
  :Analizuoja pažangą;
  if (Reikia koreguoti planą?) then (taip)
    |LLM paslauga|
    :Sugeneruoja pataisytą planą;
    |Sistema|
    :Validuoja ir išsaugo naują versiją;
  else (ne)
  endif
repeat while (Liko neatliktų treniruočių?) is (taip) not (ne)
:Pažymi planą COMPLETED;
stop
@enduml
```

## 8. Būsenų diagrama: treniruočių plano gyvavimo ciklas

Būsena ADJUSTING apsaugo nuo lygiagrečių koregavimų. Abu išėjimai iš ADJUSTING grąžina planą į ACTIVE, skiriasi tik tai, ar sukuriama nauja versija.

```plantuml
@startuml PlanStates
hide empty description
[*] --> DRAFT : planas sugeneruotas ir validuotas
DRAFT --> ACTIVE : vartotojas patvirtina
DRAFT --> ARCHIVED : vartotojas atmeta
ACTIVE --> ADJUSTING : analizė rodo koregavimo poreikį
ADJUSTING --> ACTIVE : nauja versija validi [versija + 1]
ADJUSTING --> ACTIVE : koregavimas nepavyko [lieka ankstesnė versija]
ACTIVE --> COMPLETED : atliktos visos treniruotės
ACTIVE --> ARCHIVED : vartotojas nutraukia planą
COMPLETED --> ARCHIVED : sukuriamas naujas planas
ARCHIVED --> [*]

note right of ACTIVE
  Vartotojas vienu metu gali turėti
  tik vieną ACTIVE būsenos planą
end note
@enduml
```

## 9. Diegimo diagrama

Frontend, backend ir duomenų bazė veikia atskiruose Docker konteineriuose viename serveryje, nginx atlieka reverse proxy funkciją. Išorinis LLM tiekėjas pasiekiamas per HTTPS.

```plantuml
@startuml Deployment
node "Vartotojo įrenginys" as ClientDevice {
  component "Naršyklė" as Browser
}

node "Aplikacijų serveris (Linux)" <<device>> as Server {
  node "Docker Engine" <<executionEnvironment>> as Docker {
    node "nginx konteineris" as Nginx {
      artifact "frontend build (React)" as FeArt
    }
    node "api konteineris" as ApiNode {
      artifact "backend API" as BeArt
    }
    node "postgres konteineris" as PgNode {
      database "training_db" as TrainingDb
    }
  }
}

cloud "LLM tiekėjas" as Provider {
  node "LLM API" as LlmNode
}

Browser --> Nginx : HTTPS :443
Nginx --> ApiNode : HTTP :8080 (/api reverse proxy)
ApiNode --> PgNode : TCP :5432
ApiNode --> LlmNode : HTTPS :443
@enduml
```
