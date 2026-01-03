# Recommendation Service - Diagrammes

## Diagramme de Classes (ERD)

```mermaid
erDiagram
    UserPreferences ||--o{ Recommendation : "generates"
    LearningHistory ||--o{ Recommendation : "informs"

    UserPreferences {
        string id PK
        string userId UK
        string targetLanguageCode UK
        array preferredLearningTime
        int dailyGoalMinutes
        array focusAreas
        array excludedTopics
    }

    LearningHistory {
        string id PK
        string userId UK
        string targetLanguageCode UK
        array completedLessonIds
        string lastLessonId
        array weakAreas
        array strongAreas
        float averageScore
        int totalLessons
    }

    Recommendation {
        string id PK
        string userId FK
        string targetLanguageCode
        string type
        string targetId
        string targetType
        string title
        string reason
        int priority
        date expiresAt
        boolean dismissed
        date clickedAt
    }
```

## Architecture du Moteur de Recommandation

```mermaid
flowchart TB
    subgraph "Data Sources"
        LS[Lesson Service]
        FS[Feedback Service]
        CS[Conversation Service]
    end

    subgraph "Events"
        K[Kafka]
    end

    subgraph "Recommendation Service"
        LH[(Learning<br/>History)]
        UP[(User<br/>Preferences)]
        
        RE[Recommendation<br/>Engine]
        
        NL[Next Lesson<br/>Strategy]
        RV[Revision<br/>Strategy]
        CV[Conversation<br/>Strategy]
        PR[Practice<br/>Strategy]
    end

    subgraph "Output"
        API[REST API]
        REC[(Recommendations)]
    end

    LS --> K
    FS --> K
    CS --> K
    
    K --> LH
    K --> UP
    
    LH --> RE
    UP --> RE
    
    RE --> NL
    RE --> RV
    RE --> CV
    RE --> PR
    
    NL --> REC
    RV --> REC
    CV --> REC
    PR --> REC
    
    REC --> API
```

## Flux de Génération de Recommandations

```mermaid
sequenceDiagram
    participant LS as Lesson Service
    participant K as Kafka
    participant RS as Recommendation Service
    participant DB as MongoDB
    participant FE as Frontend

    LS->>K: lesson.completed
    K->>RS: Consume event
    
    RS->>DB: Update LearningHistory
    RS->>DB: Get UserPreferences
    DB-->>RS: Preferences
    
    RS->>RS: Run RecommendationEngine
    
    par Strategy: Next Lesson
        RS->>RS: Find next uncompleted lesson
    and Strategy: Revision
        RS->>RS: Check weakAreas for revision needs
    and Strategy: Conversation
        RS->>RS: Check available slots
    end
    
    RS->>RS: Prioritize & Deduplicate
    RS->>DB: Save new Recommendations
    RS->>DB: Expire old Recommendations
    
    RS->>K: recommendation.generated
    
    Note over FE: User opens app
    FE->>RS: GET /recommendations
    RS->>DB: Get active recommendations
    DB-->>RS: Recommendations
    RS-->>FE: Personalized recommendations
```

## Algorithme de Priorisation

```mermaid
flowchart TD
    START[Start] --> CHECK_NEXT[Has uncompleted lesson?]
    
    CHECK_NEXT -->|Yes| ADD_NEXT[Add NEXT_LESSON<br/>Priority: 1]
    CHECK_NEXT -->|No| CHECK_UNIT[Suggest next unit?]
    CHECK_UNIT -->|Yes| ADD_UNIT[Add NEXT_UNIT<br/>Priority: 1]
    
    ADD_NEXT --> CHECK_WEAK
    ADD_UNIT --> CHECK_WEAK
    CHECK_NEXT -->|No units left| CHECK_WEAK
    
    CHECK_WEAK[Has weak areas?]
    CHECK_WEAK -->|Yes, errors > 5| ADD_REV[Add REVISION<br/>Priority: 2]
    CHECK_WEAK -->|No| CHECK_CONV
    ADD_REV --> CHECK_CONV
    
    CHECK_CONV[Slots available today?]
    CHECK_CONV -->|Yes| ADD_CONV[Add CONVERSATION<br/>Priority: 3]
    CHECK_CONV -->|No| CHECK_GOAL
    ADD_CONV --> CHECK_GOAL
    
    CHECK_GOAL[Daily goal reached?]
    CHECK_GOAL -->|No| ADD_PRACTICE[Add PRACTICE<br/>Priority: 4]
    CHECK_GOAL -->|Yes| DONE
    ADD_PRACTICE --> DONE
    
    DONE[Return top 5<br/>recommendations]
```

## Cycle de Vie d'une Recommandation

```mermaid
stateDiagram-v2
    [*] --> Active: Created
    
    Active --> Clicked: User clicks
    Active --> Dismissed: User dismisses
    Active --> Expired: TTL reached (24h)
    Active --> Superseded: New recommendation<br/>of same type
    
    Clicked --> [*]: Archived for analytics
    Dismissed --> Cooldown: 7 days cooldown
    Expired --> [*]: Deleted
    Superseded --> [*]: Deleted
    
    Cooldown --> [*]: Can regenerate<br/>after 7 days
```
