# Gamification Service - Diagrammes

## Diagramme de Classes (ERD)

```mermaid
erDiagram
    UserStats ||--o{ XpTransaction : "has many"
    UserStats ||--o{ UserBadge : "has many"
    UserStats ||--o{ UserChallenge : "participates"
    Badge ||--o{ UserBadge : "awarded to"
    Challenge ||--o{ UserChallenge : "joined by"

    UserStats {
        string id PK
        string userId UK
        string targetLanguageCode UK
        int totalXp
        int weeklyXp
        int monthlyXp
        int currentStreak
        int longestStreak
        date lastActivityDate
        int streakFreezeAvailable
        int level
        int lessonsCompleted
        int exercisesCompleted
        int conversationMinutes
        int perfectLessons
    }

    Badge {
        string id PK
        string code UK
        string name
        string description
        string iconUrl
        string category
        int xpReward
        string rarity
        json condition
        boolean isActive
    }

    UserBadge {
        string id PK
        string userId FK
        string badgeId FK
        date unlockedAt
        string targetLanguageCode
    }

    Challenge {
        string id PK
        string title
        string description
        string type
        string targetType
        int targetValue
        int xpReward
        date startDate
        date endDate
        boolean isActive
    }

    UserChallenge {
        string id PK
        string userId FK
        string challengeId FK
        int currentProgress
        string status
        date joinedAt
        date completedAt
    }

    XpTransaction {
        string id PK
        string userId FK
        string targetLanguageCode
        int amount
        string source
        string sourceId
        string description
        date createdAt
    }
```

## Flux d'Attribution d'XP

```mermaid
sequenceDiagram
    participant LS as Lesson Service
    participant K as Kafka
    participant GS as Gamification Service
    participant DB as MongoDB
    participant FE as Frontend

    LS->>K: lesson.completed
    K->>GS: Consume event
    
    GS->>DB: Get UserStats
    DB-->>GS: UserStats
    
    GS->>GS: Calculate XP (base + bonus)
    GS->>GS: Check badge conditions
    
    GS->>DB: Update UserStats (XP, streak, counters)
    GS->>DB: Create XpTransaction
    
    alt Badge Unlocked
        GS->>DB: Create UserBadge
        GS->>K: badge.unlocked
    end
    
    alt Level Up
        GS->>K: level.up
    end
    
    GS->>K: xp.earned
    K-->>FE: Real-time notification
    FE->>FE: Show celebration animation
```

## Gestion du Streak (Job Nocturne)

```mermaid
sequenceDiagram
    participant CRON as Scheduled Job
    participant GS as Gamification Service
    participant DB as MongoDB
    participant K as Kafka

    Note over CRON: Runs daily at 00:05 UTC
    
    CRON->>GS: Trigger streak check
    GS->>DB: Find all UserStats where lastActivityDate < today
    DB-->>GS: List of users
    
    loop For each user
        GS->>GS: Check if activity yesterday
        
        alt Had activity yesterday
            GS->>DB: currentStreak += 1
            GS->>GS: Check streak milestones (7, 30, 100...)
            alt Milestone reached
                GS->>DB: Award streak badge
                GS->>K: badge.unlocked
            end
        else No activity yesterday
            alt Has freeze available
                GS->>DB: streakFreezeAvailable -= 1
                GS->>K: streak.updated (freezeUsed: true)
            else No freeze
                GS->>DB: currentStreak = 0
                GS->>K: streak.updated (streakLost: true)
            end
        end
    end
```

## Leaderboard Architecture

```mermaid
flowchart TB
    subgraph "Data Sources"
        US[(UserStats<br/>MongoDB)]
    end

    subgraph "Leaderboard Service"
        LB[Leaderboard<br/>Calculator]
        CACHE[Leaderboard<br/>Cache]
    end

    subgraph "Types"
        W[Weekly<br/>Reset Monday]
        M[Monthly<br/>Reset 1st]
        G[Global<br/>All time]
    end

    US --> LB
    LB --> W
    LB --> M
    LB --> G
    
    W --> CACHE
    M --> CACHE
    G --> CACHE

    subgraph "API"
        API[GET /leaderboard]
    end

    CACHE --> API

    Note1[Cache TTL: 5 min]
    CACHE --- Note1
```

## Flux de Participation à un Défi

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant GS as Gamification Service
    participant DB as MongoDB

    U->>FE: View available challenges
    FE->>GS: GET /challenges
    GS->>DB: Find active challenges
    DB-->>GS: Challenges list
    GS-->>FE: Challenges with user progress
    
    U->>FE: Join challenge
    FE->>GS: POST /challenges/{id}/join
    GS->>DB: Check not already joined
    GS->>DB: Create UserChallenge
    GS-->>FE: Confirmation
    
    Note over GS: Later, on XP earned...
    
    GS->>DB: Find user's active challenges
    GS->>GS: Update progress for matching challenges
    GS->>DB: Update UserChallenge.currentProgress
    
    alt Target reached
        GS->>DB: UserChallenge.status = COMPLETED
        GS->>DB: Add XP reward
        GS->>FE: challenge.completed notification
    end
```
