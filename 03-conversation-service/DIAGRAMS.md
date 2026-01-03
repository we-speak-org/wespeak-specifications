# Conversation Service - Diagrammes

## Diagramme ERD

```mermaid
erDiagram
    Topic {
        string id PK
        string targetLanguageCode
        string title
        string description
        string[] suggestedQuestions
        string level
        string category
        boolean isActive
        datetime createdAt
    }

    Session {
        string id PK
        string targetLanguageCode
        string status
        string type
        string inviteCode
        string topicId FK
        string hostUserId
        int minParticipants
        int maxParticipants
        datetime scheduledStartAt
        datetime startedAt
        datetime endedAt
        string endReason
        datetime createdAt
    }

    Participant {
        string id PK
        string sessionId FK
        string userId
        string role
        string status
        datetime joinedAt
        datetime leftAt
        int speakingTimeSeconds
    }

    QueueEntry {
        string id PK
        string userId
        string targetLanguageCode
        string level
        string preferredTopicCategory
        datetime joinedQueueAt
        datetime expiresAt
    }

    Topic ||--o{ Session : "guides"
    Session ||--|{ Participant : "contains"
```

---

## Flux de Matchmaking (Quick Match)

```mermaid
sequenceDiagram
    participant U1 as Marie (B1)
    participant U2 as Jean (B1)
    participant U3 as Pierre (B2)
    participant API as Conversation API
    participant Redis as Redis Queue
    participant WS as WebSocket

    U1->>API: POST /matchmaking/join
    API->>Redis: Add to queue en:B1
    API-->>U1: { status: waiting }
    
    U2->>API: POST /matchmaking/join
    API->>Redis: Check queue en:B1
    Redis-->>API: Found Marie
    
    Note over API: 2 users found - create session
    API->>API: Create Session (waiting)
    API->>Redis: Remove both from queue
    
    API->>WS: Notify Marie
    API->>WS: Notify Jean
    WS-->>U1: { type: matched, sessionId }
    WS-->>U2: { type: matched, sessionId }
    
    U3->>API: POST /matchmaking/join
    API->>Redis: Add to queue en:B2
    Note over U3: Continue waiting...
```

---

## Flux Session Multi-Participants

```mermaid
sequenceDiagram
    participant Host as Marie (Host)
    participant P1 as Jean
    participant P2 as Pierre
    participant API as API
    participant WS as WebSocket

    Host->>API: POST /sessions (type: private)
    API-->>Host: { sessionId, inviteCode: "ABC123" }
    
    Host->>WS: Connect to session
    
    P1->>API: POST /sessions/join { inviteCode }
    API-->>P1: { sessionId }
    P1->>WS: Connect to session
    WS-->>Host: { type: participant-joined, user: Jean }
    
    P2->>API: POST /sessions/join { inviteCode }
    API-->>P2: { sessionId }
    P2->>WS: Connect to session
    WS-->>Host: { type: participant-joined, user: Pierre }
    WS-->>P1: { type: participant-joined, user: Pierre }
    
    Host->>API: POST /sessions/{id}/start
    WS-->>Host: { type: session-started }
    WS-->>P1: { type: session-started }
    WS-->>P2: { type: session-started }
    
    Note over Host,P2: WebRTC connections established
    Note over Host,P2: Conversation in progress...
    
    P2->>API: POST /sessions/{id}/leave
    WS-->>Host: { type: participant-left, user: Pierre }
    WS-->>P1: { type: participant-left, user: Pierre }
    
    Host->>API: POST /sessions/{id}/end
    WS-->>Host: { type: session-ended }
    WS-->>P1: { type: session-ended }
```

---

## Flux WebRTC Signaling (Multi-Peers)

```mermaid
sequenceDiagram
    participant U1 as Marie
    participant WS as WebSocket Server
    participant U2 as Jean
    participant U3 as Pierre

    Note over U1,U3: Marie creates session, Jean and Pierre join

    U2->>WS: Connect
    WS->>U1: { type: participant-joined, userId: Jean }
    
    Note over U1,U2: Marie initiates connection with Jean
    U1->>WS: { type: offer, toUserId: Jean, sdp }
    WS->>U2: { type: offer, fromUserId: Marie, sdp }
    U2->>WS: { type: answer, toUserId: Marie, sdp }
    WS->>U1: { type: answer, fromUserId: Jean, sdp }
    
    U3->>WS: Connect
    WS->>U1: { type: participant-joined, userId: Pierre }
    WS->>U2: { type: participant-joined, userId: Pierre }
    
    Note over U1,U3: Marie initiates connection with Pierre
    U1->>WS: { type: offer, toUserId: Pierre, sdp }
    WS->>U3: { type: offer, fromUserId: Marie, sdp }
    
    Note over U2,U3: Jean initiates connection with Pierre
    U2->>WS: { type: offer, toUserId: Pierre, sdp }
    WS->>U3: { type: offer, fromUserId: Jean, sdp }
    
    Note over U1,U3: ICE candidates exchanged...
    Note over U1,U3: All P2P connections established
```

---

## Cycle de Vie d'une Session

```mermaid
stateDiagram-v2
    [*] --> Waiting: Session created
    
    Waiting --> Active: Host starts OR minParticipants reached
    Waiting --> Ended: Timeout (5 min) / Host cancels
    
    Active --> Ended: Host ends / All leave / Timeout (30 min)
    
    Ended --> [*]
    
    note right of Waiting
        Waiting for participants
        Private: host decides when to start
        Public: auto-start at minParticipants
    end note
    
    note right of Active
        Participants can join/leave
        Max 6 participants
        Max 30 minutes duration
    end note
    
    note right of Ended
        session.ended event published
        XP awarded to participants
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
        MATCH[Matchmaking Engine]
        SIG[Signaling Handler]
    end

    subgraph Storage
        MONGO[(MongoDB)]
        REDIS[(Redis)]
    end

    subgraph Kafka
        TOPIC_CONV[conversation.events]
        TOPIC_USER[user.events]
    end

    subgraph OtherServices
        AUTH[Auth Service]
        GAMI[Gamification Service]
    end

    APP -->|REST| API
    APP <-->|WebSocket| WS
    
    API --> MONGO
    API --> REDIS
    
    WS --> SIG
    SIG --> REDIS
    
    MATCH --> REDIS
    
    API -->|Publish| TOPIC_CONV
    TOPIC_USER -->|Consume| API
    
    TOPIC_CONV --> GAMI
    
    API -->|Validate JWT| AUTH
```

---

## Algorithme de Matchmaking

```mermaid
flowchart TD
    START[User requests match] --> CHECK{Already in queue<br/>or session?}
    CHECK -->|Yes| ERROR[Return 409 error]
    CHECK -->|No| ADD[Add to Redis queue]
    
    ADD --> SCAN[Scan compatible users]
    SCAN --> FOUND{2+ users found?}
    
    FOUND -->|Yes| CREATE[Create Session]
    FOUND -->|No| WAIT[Wait in queue]
    
    WAIT --> TIMEOUT{Expired? 2 min}
    TIMEOUT -->|Yes| EXPIRE[Remove from queue]
    TIMEOUT -->|No| SCAN
    
    CREATE --> REMOVE[Remove matched from queue]
    REMOVE --> NOTIFY[Notify via WebSocket]
    NOTIFY --> DONE[Session waiting for connections]

    subgraph Compatibility Rules
        R1[Same targetLanguageCode]
        R2[Level difference <= 1]
    end
    
    SCAN -.-> R1
    SCAN -.-> R2
```
