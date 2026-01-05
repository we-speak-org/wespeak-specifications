# Conversation Service - Diagrammes

## Diagramme ERD

```mermaid
erDiagram
    TimeSlot {
        string id PK
        string targetLanguageCode
        string level
        datetime startTime
        int durationMinutes
        int maxParticipants
        int minParticipants
        string recurrence
        boolean isActive
        datetime createdAt
    }

    Registration {
        string id PK
        string timeSlotId FK
        string userId
        string status
        datetime registeredAt
        datetime cancelledAt
    }

    Session {
        string id PK
        string timeSlotId FK
        string targetLanguageCode
        string level
        string status
        datetime startedAt
        datetime endedAt
        boolean recordingEnabled
        string recordingUrl
        datetime createdAt
    }

    Participant {
        string id PK
        string sessionId FK
        string userId
        string status
        boolean cameraEnabled
        boolean micEnabled
        boolean recordingConsent
        datetime joinedAt
        datetime leftAt
    }

    TimeSlot ||--o{ Registration : "has"
    TimeSlot ||--o| Session : "creates"
    Session ||--|{ Participant : "contains"
```

---

## Flux Inscription et Session

```mermaid
sequenceDiagram
    participant U1 as Marie
    participant U2 as Jean
    participant API as Conversation API
    participant DB as MongoDB
    participant WS as WebSocket

    Note over U1,U2: Phase 1: Inscription aux créneaux

    U1->>API: GET /timeslots?language=en&level=B1
    API-->>U1: Liste des créneaux disponibles
    
    U1->>API: POST /timeslots/slot123/register
    API->>DB: Create Registration
    API-->>U1: { status: registered }
    
    U2->>API: POST /timeslots/slot123/register
    API->>DB: Create Registration
    API-->>U2: { status: registered }

    Note over U1,U2: Phase 2: 5 min avant le créneau
    
    API->>U1: Notification rappel
    API->>U2: Notification rappel

    Note over U1,U2: Phase 3: Le créneau commence

    U1->>API: POST /sessions/join { timeSlotId, recordingConsent: true }
    API->>DB: Create/Get Session, Add Participant
    API-->>U1: { sessionId, participants: [] }
    U1->>WS: Connect
    
    U2->>API: POST /sessions/join { timeSlotId, recordingConsent: true }
    API->>DB: Add Participant
    API-->>U2: { sessionId, participants: [Marie] }
    U2->>WS: Connect
    
    WS-->>U1: { type: participant-joined, user: Jean }
    
    Note over U1,U2: WebRTC connections established
    Note over U1,U2: Conversation + Recording...
    
    Note over API: Fin du créneau (30 min)
    
    WS-->>U1: { type: session-ended }
    WS-->>U2: { type: session-ended }
    API->>DB: Upload recording to R2
    API->>API: Publish session.recorded event
```

---

## Flux WebRTC Signaling

```mermaid
sequenceDiagram
    participant U1 as Marie
    participant WS as WebSocket Server
    participant U2 as Jean
    participant U3 as Pierre

    Note over U1,U3: Tous rejoignent la session

    U1->>WS: Connect
    U2->>WS: Connect
    WS->>U1: { type: participant-joined, user: Jean }
    
    Note over U1,U2: Marie initie connexion avec Jean
    U1->>WS: { type: offer, toUserId: Jean, sdp }
    WS->>U2: { type: offer, fromUserId: Marie, sdp }
    U2->>WS: { type: answer, toUserId: Marie, sdp }
    WS->>U1: { type: answer, fromUserId: Jean, sdp }
    
    U3->>WS: Connect
    WS->>U1: { type: participant-joined, user: Pierre }
    WS->>U2: { type: participant-joined, user: Pierre }
    
    Note over U1,U3: Connexions avec Pierre
    U1->>WS: { type: offer, toUserId: Pierre, sdp }
    U2->>WS: { type: offer, toUserId: Pierre, sdp }
    
    Note over U1,U3: ICE candidates exchanged...
    Note over U1,U3: All P2P connections established
    
    Note over U2: Jean désactive sa caméra
    U2->>WS: PATCH /sessions/current/media { cameraEnabled: false }
    WS->>U1: { type: media-state-changed, userId: Jean, cameraEnabled: false }
    WS->>U3: { type: media-state-changed, userId: Jean, cameraEnabled: false }
```

---

## Cycle de Vie d'une Session

```mermaid
stateDiagram-v2
    [*] --> Waiting: Session created at slot start
    
    Waiting --> Active: minParticipants reached (2+)
    Waiting --> Ended: Timeout (5 min grace period)
    
    Active --> Ended: Time expired / All participants left
    
    Ended --> [*]
    
    note right of Waiting
        Waiting for participants
        Grace period: 5 minutes
        Recording consent collected
    end note
    
    note right of Active
        Participants can join/leave
        Max 8 participants
        Audio recording if consent
        Controls: mic, camera
    end note
    
    note right of Ended
        session.ended event published
        If recording: upload to R2
        session.recorded event published
    end note
```

---

## Architecture du Service

```mermaid
flowchart TB
    subgraph Clients
        APP[Mobile/Web App]
    end

    subgraph ConversationService
        API[REST API]
        WS[WebSocket Server]
        SCHED[Slot Scheduler]
        REC[Recording Handler]
    end

    subgraph Storage
        MONGO[(MongoDB)]
        R2[(Cloudflare R2)]
    end

    subgraph Kafka
        TOPIC_CONV[conversation.events]
        TOPIC_USER[user.events]
    end

    subgraph OtherServices
        AUTH[Auth Service]
        GAMI[Gamification Service]
        FEED[Feedback Service]
    end

    APP -->|REST| API
    APP <-->|WebSocket| WS
    
    API --> MONGO
    REC --> R2
    
    SCHED -->|Create slots| MONGO
    
    API -->|Publish| TOPIC_CONV
    TOPIC_USER -->|Consume| API
    
    TOPIC_CONV --> GAMI
    TOPIC_CONV -->|session.recorded| FEED
    
    API -->|Validate JWT| AUTH
```

---

## Flux d'Enregistrement

```mermaid
flowchart TD
    START[Session démarre] --> CHECK{Au moins 1 participant<br/>avec consentement?}
    CHECK -->|Non| NOREC[Pas d'enregistrement]
    CHECK -->|Oui| REC[Démarrer enregistrement audio]
    
    REC --> CAPTURE[Capturer audio des participants consentants]
    CAPTURE --> MIX[Mixer les flux audio]
    
    MIX --> END{Session terminée?}
    END -->|Non| CAPTURE
    END -->|Oui| FINALIZE[Finaliser le fichier]
    
    FINALIZE --> UPLOAD[Upload vers R2]
    UPLOAD --> EVENT[Publier session.recorded]
    EVENT --> FEEDBACK[Feedback Service transcrit et analyse]
    
    NOREC --> DONE[Session sans enregistrement]
```
