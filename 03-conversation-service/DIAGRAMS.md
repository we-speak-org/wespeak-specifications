# Conversation Service - Diagrammes

## Diagramme ERD

```mermaid
erDiagram
    Topic {
        string id PK
        string targetLanguageCode
        string level
        string title
        string description
        string[] promptQuestions
        string category
        int estimatedDurationMinutes
        boolean isActive
        datetime createdAt
    }

    ConversationSession {
        string id PK
        string topicId FK
        string targetLanguageCode
        string participant1Id FK
        string participant2Id FK
        string status
        datetime scheduledAt
        datetime startedAt
        datetime endedAt
        int actualDurationSeconds
        string endReason
        string audioRecordingUrl
        datetime createdAt
    }

    MatchmakingRequest {
        string id PK
        string userId FK
        string learningProfileId
        string targetLanguageCode
        string userLevel
        string preferredTopicId FK
        int preferredDuration
        string status
        string matchedWithUserId
        string sessionId FK
        datetime createdAt
        datetime expiresAt
    }

    Topic ||--o{ ConversationSession : "guides"
    ConversationSession ||--o| MatchmakingRequest : "creates"
```

---

## Flux de Matchmaking

```mermaid
sequenceDiagram
    participant User1 as Marie (B1)
    participant API as Conversation API
    participant Redis as Redis Queue
    participant User2 as Jean (B1)
    participant WS as WebSocket

    User1->>API: POST /matchmaking/join
    API->>Redis: Add to queue en:B1
    API-->>User1: { status: pending }
    
    User2->>API: POST /matchmaking/join
    API->>Redis: Check queue en:B1
    Redis-->>API: Found Marie
    
    API->>API: Create Session
    API->>Redis: Remove both from queue
    
    API->>WS: Notify Marie (match-found)
    API->>WS: Notify Jean (match-found)
    
    WS-->>User1: { type: match-found, isInitiator: true }
    WS-->>User2: { type: match-found, isInitiator: false }
```

---

## Flux WebRTC Signaling

```mermaid
sequenceDiagram
    participant U1 as Marie (Initiator)
    participant WS as WebSocket Server
    participant U2 as Jean (Responder)

    Note over U1,U2: Match found - Marie is initiator

    U1->>U1: Create RTCPeerConnection
    U1->>U1: Create Offer SDP
    U1->>WS: { type: offer, sdp: ... }
    WS->>U2: { type: offer, sdp: ... }
    
    U2->>U2: Create RTCPeerConnection
    U2->>U2: Set Remote Description
    U2->>U2: Create Answer SDP
    U2->>WS: { type: answer, sdp: ... }
    WS->>U1: { type: answer, sdp: ... }
    
    U1->>U1: Set Remote Description
    
    loop ICE Candidates
        U1->>WS: { type: ice-candidate, candidate: ... }
        WS->>U2: { type: ice-candidate, candidate: ... }
        U2->>WS: { type: ice-candidate, candidate: ... }
        WS->>U1: { type: ice-candidate, candidate: ... }
    end
    
    Note over U1,U2: P2P Connection Established
    
    loop Every 10 seconds
        U1->>WS: { type: heartbeat }
        U2->>WS: { type: heartbeat }
    end
```

---

## Cycle de Vie d'une Session

```mermaid
stateDiagram-v2
    [*] --> Waiting: Match found
    Waiting --> Active: Both connected
    Waiting --> Cancelled: Timeout/Disconnect
    
    Active --> Completed: Normal end
    Active --> Cancelled: Dropped/Reported
    
    Completed --> [*]
    Cancelled --> [*]
    
    note right of Waiting
        Max 30 seconds
        to establish connection
    end note
    
    note right of Active
        Heartbeat every 10s
        Max 30 minutes
    end note
    
    note right of Completed
        Duration >= 2 min
        XP awarded
        Audio sent for analysis
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
        S3[(S3 Audio)]
    end

    subgraph Kafka
        TOPIC_CONV[conversation.events]
        TOPIC_USER[user.events]
        TOPIC_FB[feedback.events]
    end

    subgraph OtherServices
        AUTH[Auth Service]
        FEED[Feedback Service]
        GAMI[Gamification Service]
    end

    APP -->|REST| API
    APP <-->|WebSocket| WS
    
    API --> MONGO
    API --> REDIS
    
    WS --> SIG
    SIG --> REDIS
    
    MATCH --> REDIS
    
    API -->|Upload audio| S3
    API -->|Publish| TOPIC_CONV
    
    TOPIC_USER -->|Consume| API
    TOPIC_FB -->|Consume| API
    
    TOPIC_CONV --> FEED
    TOPIC_CONV --> GAMI
    
    API -->|Validate JWT| AUTH
```

---

## Algorithme de Matchmaking

```mermaid
flowchart TD
    START[User requests match] --> CHECK{Already in queue?}
    CHECK -->|Yes| ERROR[Return 409 ALREADY_IN_QUEUE]
    CHECK -->|No| ADD[Add to Redis queue]
    
    ADD --> SCAN[Scan compatible queues]
    SCAN --> FOUND{Match found?}
    
    FOUND -->|Yes| CREATE[Create Session]
    FOUND -->|No| WAIT[Wait in queue]
    
    WAIT --> TIMEOUT{Expired?}
    TIMEOUT -->|Yes| EXPIRE[Return timeout]
    TIMEOUT -->|No| SCAN
    
    CREATE --> NOTIFY[Notify both users via WebSocket]
    NOTIFY --> DONE[Return session info]

    subgraph Compatible Queues
        Q1[Same language]
        Q2[Same level]
        Q3[Level - 1]
        Q4[Level + 1]
    end
    
    SCAN --> Q1
    SCAN --> Q2
    SCAN --> Q3
    SCAN --> Q4
```
