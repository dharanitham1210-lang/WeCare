# WeCare Project Report

## 1. Executive Summary

WeCare is a caregiver-supervised digital companion for people living with memory impairment, especially dementia or Alzheimer’s-related cognitive decline. The system is designed to reduce anxiety, increase orientation, and provide a safe memory-support experience for the patient while allowing family members and caregivers to monitor risk, approve memories, and intervene when needed.

The product combines:

- a patient-facing Android voice companion,
- a guardian dashboard for oversight and memory approval,
- a Supabase-powered backend for storage, auth, realtime updates, and role-based access,
- Firebase push notifications for alerts,
- a policy-first conversation engine to keep responses fact-safe and grounded in verified personal history.

The design philosophy is intentionally restrictive: the AI is not allowed to invent facts about the patient’s life. It can only speak using approved biographical memory, and it must follow caregiver-defined boundaries such as “do not discuss” and sensitive-topic handling.

---

## 2. Product Vision

The core mission of WeCare is to help a patient feel emotionally anchored, oriented, and safe in everyday conversations without overwhelming them with complex interfaces or memory tests.

### Main goals

- protect the patient’s dignity and reduce confusion,
- provide warm, familiar conversational support,
- keep responses limited to verified biographical facts,
- monitor for repeated questioning or distress signals,
- escalate to caregivers appropriately,
- increase caregiver confidence and continuity of care.

### Primary users

1. Patient
   - voice-first communication,
   - calm and simple interface,
   - reassurance and gentle orientation.

2. Guardian / family caregiver
   - review patient activity,
   - approve memory entries,
   - monitor risk events,
   - send prompts or care reminders.

3. Clinical / professional caregiver
   - view patient trends,
   - review alerts and logs,
   - monitor care quality and escalation events.

---

## 3. Business and Clinical Problem

People with dementia often struggle with:

- short-term memory loss,
- repeated questioning,
- disorientation,
- anxiety during evening hours or stressful situations,
- confusion around family identities and routines,
- emotional distress when asked about lost loved ones or sensitive topics.

A generic AI assistant is dangerous in this context because it may:

- invent personal facts,
- contradict the patient,
- increase confusion,
- mention sensitive or painful topics without approval.

WeCare solves this by forcing the system to operate inside a strict policy and validated-memory boundary.

---

## 4. Product Scope

The project includes two major surfaces:

### 4.1 Android application

The Android app in the workspace is the main operational product. It includes:

- user auth and onboarding,
- patient creation and consent workflows,
- guardian dashboard,
- patient voice screen,
- de-escalation sanctuary screen,
- real-time monitoring and alerting.

### 4.2 Web demonstrator/dashboard prototype

The extracted web project under WeCare-extracted/WeCare demonstrates the intended clinical dashboard and patient voice experience. It presents the intended UX as a product demo and design specification, including clinical summaries, trends, and caregiver prompts.

---

## 5. Product Architecture

The system follows a layered architecture:

### 5.1 Presentation layer

Responsible for user interactions and screens.

Key files:

- app/src/main/kotlin/com/wecare/navigation/AppNavGraph.kt
- app/src/main/kotlin/com/wecare/ui/patient/PatientVoiceScreen.kt
- app/src/main/kotlin/com/wecare/ui/guardian/GuardianDashboardScreen.kt
- app/src/main/kotlin/com/wecare/ui/onboarding/CreatePatientScreen.kt
- app/src/main/kotlin/com/wecare/ui/deescalation/DeEscalationSanctuaryScreen.kt

Responsibilities:

- login / signup / invite flow,
- patient onboarding,
- guardian dashboard views,
- patient voice interaction,
- de-escalation support screens.

### 5.2 ViewModel / application logic layer

This layer coordinates screen state and application actions.

Key files:

- app/src/main/kotlin/com/wecare/ui/patient/PatientVoiceViewModel.kt
- app/src/main/kotlin/com/wecare/ui/guardian/GuardianDashboardViewModel.kt
- app/src/main/kotlin/com/wecare/ui/auth/AuthViewModel.kt
- app/src/main/kotlin/com/wecare/ui/onboarding/CreatePatientViewModel.kt

Responsibilities:

- load patient data,
- manage session lifecycle,
- initiate speech processing,
- trigger backend queries,
- process realtime UI updates,
- handle caregiver approvals and memory additions.

### 5.3 Repository / service layer

Responsible for backend access and data coordination.

Key files:

- app/src/main/kotlin/com/wecare/data/repository/PatientRepository.kt
- app/src/main/kotlin/com/wecare/data/repository/ConversationRepository.kt

Responsibilities:

- fetch patient profile and memory data,
- retrieve risk events and prompts,
- create conversation sessions,
- call the policy-turn backend function,
- upload approved memory data,
- manage notification creation.

### 5.4 Data model layer

Responsible for typed database and application representations.

Key file:

- app/src/main/kotlin/com/wecare/data/model/Models.kt

Key domain objects:

- UserProfile
- Patient
- CommunicationProfile
- BiographicalMemory
- RiskEvent
- ConversationTurn
- PolicyTurnResponse
- FamilyPrompt
- AuditLog
- AccessRole

These models align with the project’s memory-safe architecture and Supabase schema.

### 5.5 Infrastructure layer

Responsible for external dependencies and runtime services.

Key files:

- app/src/main/kotlin/com/wecare/di/SupabaseModule.kt
- app/src/main/kotlin/com/wecare/di/EngineModule.kt
- app/src/main/kotlin/com/wecare/WeCareApplication.kt
- app/src/main/kotlin/com/wecare/notifications/WeCareFirebaseMessagingService.kt

Responsibilities:

- configure Supabase client,
- configure Hilt dependency injection,
- configure speech engines,
- persist FCM tokens,
- receive push notifications for alerts.

---

## 6. Technology Stack

### Frontend / mobile

- Kotlin
- Jetpack Compose
- Navigation Compose
- Material 3
- Hilt for dependency injection
- Android speech engine abstraction
- Android TTS engine

### Backend

- Supabase
  - Postgres Database
  - Supabase Auth
  - Realtime subscriptions
  - Storage
  - Edge Functions
- PostgreSQL Row Level Security (RLS)

### Alerts and messaging

- Firebase Cloud Messaging (FCM)

### Architecture patterns

- MVVM pattern
- Repository pattern
- Realtime event-driven updates
- StateFlow/SharedFlow-based reactive UI state

### Build tooling

- Gradle Kotlin DSL
- Android Gradle Plugin
- Kotlin 17
- Compose BOM

---

## 7. Core Product Workflow

### 7.1 App startup

When the app starts:

- Hilt initializes the app through the application class,
- Supabase client is created,
- navigation listens to auth session state,
- authenticated users are routed to the relevant screen based on role.

### 7.2 Guardian onboarding and patient setup

The guardian may:

- create an account,
- accept invites,
- create or link a patient,
- configure consent and access roles,
- add or approve biographical memories.

### 7.3 Patient voice interaction

The patient voice flow works like this:

1. the app starts speech recognition,
2. audio input is converted to text,
3. transcript is sent to the backend policy-turn function,
4. the policy engine decides the appropriate response strategy,
5. the final spoken response is returned in a structured way,
6. TTS reads the response in a calm voice with stage-appropriate pacing.

### 7.4 Guardian monitoring

The guardian dashboard loads:

- patient profile,
- care continuity metrics,
- risk events,
- verified memory list,
- pending memories,
- family prompts,
- audit logs.

Realtime updates from Supabase immediately refresh the UI when new alerts or memory changes are added.

### 7.5 Alerting

When a risk event or significant distress pattern is detected:

- the event is written to the database,
- the guardian dashboard shows it in near real time,
- FCM can send a push notification to the guardian device,
- the patient can be routed to a de-escalation or calming flow.

---

## 8. Safety and Memory Architecture

This is the most important architectural concept in the project.

### Principle

The system does not allow the LLM to “invent” details about the patient’s life. It must only speak using validated memory records.

### Design intent

This is reflected in:

- verified biographical memory records,
- role-gated memory approval,
- sensitive-topic handling,
- patient-safe response strategies,
- explicit response mode control instead of free conversation.

### Supported response strategies

The Memory Architecture document describes a controlled response policy that covers flags like:

- reassure,
- validate_emotion,
- orient_gently,
- reminisce,
- redirect,
- escalate.

The product is designed to ask the AI to phrase approved content rather than generate new facts.

---

## 9. Data and Storage Model

The database and model layer revolve around a memory-first schema.

### Core tables / concepts represented in the project

- patients
- patient_access_grants
- biographical_memories
- pending_memories
- risk_events
- conversation_sessions
- conversation_turns
- family_prompts
- audit_logs
- notifications
- media_assets
- users

### Example data objects

From the models layer:

- Patient contains preferred name, language, communication profile, and status
- BiographicalMemory contains title, fact, category, response strategy, consent scope, and verification status
- RiskEvent contains risk type, risk level, score, indicators, and acknowledgment data

This means the app is built around a patient memory graph rather than a generic social app model.

---

## 10. Security Model

### RLS and access restriction

The Supabase migrations include row-level security policies for:

- patients,
- patient access grants,
- organization membership,
- guardian actions,
- restricted read/write behavior.

Key security file:

- supabase/migrations/002_rls_policies.sql

This is essential because patient memory and care data are highly sensitive.

### Consent-aware access

The specification and code strongly suggest role restrictions and consent scope logic for:

- memory access,
- alert visibility,
- transcript review,
- guardian privileges.

### FCM token handling

The app captures and persists the guardian’s FCM token so alerts can be pushed when the app is backgrounded or closed.

---

## 11. Realtime Architecture

Realtime updates are managed by RealtimeManager.

Key responsibilities:

- subscribe to risk_events inserts,
- subscribe to family_prompts inserts,
- subscribe to biographical_memories inserts,
- subscribe to notifications inserts,
- handle patient presence tracking.

This allows the guardian dashboard to react immediately without manual refresh.

The implementation is in:

- app/src/main/kotlin/com/wecare/data/realtime/RealtimeManager.kt

---

## 12. Core Files and Their Purpose

### Android app main files

- app/src/main/kotlin/com/wecare/MainActivity.kt
  - main activity entry point

- app/src/main/kotlin/com/wecare/WeCareApplication.kt
  - Hilt app initialization

- app/src/main/kotlin/com/wecare/navigation/AppNavGraph.kt
  - app routes and auth-driven navigation

- app/src/main/kotlin/com/wecare/ui/patient/PatientVoiceViewModel.kt
  - core patient conversation flow

- app/src/main/kotlin/com/wecare/ui/guardian/GuardianDashboardViewModel.kt
  - guardian state, alerts, memory management

- app/src/main/kotlin/com/wecare/data/repository/ConversationRepository.kt
  - policy-turn communication pipeline

- app/src/main/kotlin/com/wecare/data/repository/PatientRepository.kt
  - patient and memory retrieval logic

- app/src/main/kotlin/com/wecare/notifications/WeCareFirebaseMessagingService.kt
  - push notification handling

### Config and build files

- app/build.gradle.kts
- settings.gradle.kts
- build.gradle.kts
- gradle/libs.versions.toml

### Database and backend

- supabase/migrations/
- supabase/functions/
- deploy-supabase.ps1

### Web prototype

- WeCare-extracted/WeCare/

---

## 13. Product Strengths

- Clear caregiver-first governance model
- Safety-oriented design for dementia patients
- Verified-memory approach reduces hallucination risk
- Real-time guardian visibility and alerts
- Voice-first UX for cognitively burdened users
- Strong separation of patient and guardian responsibilities
- Structured backend and event-driven updates

---

## 14. Product Risks and Gaps

### 14.1 Clinical model maturity

The risk detection and distress logic are conceptually strong but still simplified compared with a clinical-grade monitoring system.

### 14.2 Consent enforcement needs formal operationalization

Consent should be enforced in the backend, audited, and refreshed over time to remain compliant and safe.

### 14.3 Data minimization and retention policy

The product needs explicit retention and deletion policies for sensitive patient data and conversation logs.

### 14.4 Governance and roles need deeper operational enforcement

Role-based access must be robustly validated and tested in production conditions.

---

## 15. Recommended Next Steps

1. Finalize the production policy engine and validate all safety rules with test cases.
2. Add formal consent lifecycle management with reminders and expiry checks.
3. Harden audit logging to cover all memory retrieval, risk detection, and caregiver actions.
4. Implement stronger risk model scoring with explainable signals and threshold tuning.
5. Expand patient and caregiver accessibility QA for real dementia care scenarios.
6. Add automated integration tests for role enforcement, memory verification, and alert generation.

---

## 16. Final Assessment

WeCare is a purpose-built digital care companion designed to support memory-impaired patients with safe, calm, identity-preserving interactions while ensuring family caregivers remain informed and in control. It is not a general-purpose chatbot; it is a bounded, policy-driven care system for emotional reassurance, memory continuity, and caregiver-supported risk management.

The project is technically well structured, clearly aligned with a human-centered dementia-care mission, and strongly grounded in explicit safety rules. With further hardening of consent logic, clinical risk modeling, and backend enforcement, it has strong potential as a real-world care product.

---

## 17. Project File Reference Summary

Main implementation files:

- app/build.gradle.kts
- app/src/main/kotlin/com/wecare/navigation/AppNavGraph.kt
- app/src/main/kotlin/com/wecare/data/model/Models.kt
- app/src/main/kotlin/com/wecare/data/repository/ConversationRepository.kt
- app/src/main/kotlin/com/wecare/data/repository/PatientRepository.kt
- app/src/main/kotlin/com/wecare/data/realtime/RealtimeManager.kt
- app/src/main/kotlin/com/wecare/notifications/WeCareFirebaseMessagingService.kt
- app/src/main/kotlin/com/wecare/ui/patient/PatientVoiceViewModel.kt
- app/src/main/kotlin/com/wecare/ui/guardian/GuardianDashboardViewModel.kt
- supabase/migrations/002_rls_policies.sql
- WeCare-extracted/WeCare/UI_UX_AND_SYSTEM_PROMPT.md

This project is best described as a memory-safe, voice-first dementia care companion with caregiver oversight, realtime alerts, and a policy-managed AI communication layer.
