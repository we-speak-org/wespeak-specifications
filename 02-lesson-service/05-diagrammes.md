# Lesson Service - Diagrammes

## 1. Modèle de Données (ERD)

```mermaid
erDiagram
    Course ||--o{ Unit : contient
    Unit ||--o{ Lesson : contient
    Lesson ||--o{ Exercise : contient
    
    User ||--o{ UserProgress : "a une progression par langue"
    User ||--o{ LessonCompletion : "termine des leçons"
    Lesson ||--o{ LessonCompletion : "est terminée par"
    
    Course {
        ObjectId _id PK
        string targetLanguageCode
        string level
        string title
        string description
        string imageUrl
        int order
        int requiredXP
        int estimatedHours
        boolean isPublished
        date createdAt
        date updatedAt
    }
    
    Unit {
        ObjectId _id PK
        ObjectId courseId FK
        string title
        string description
        string imageUrl
        int order
        date createdAt
        date updatedAt
    }
    
    Lesson {
        ObjectId _id PK
        ObjectId unitId FK
        string title
        string description
        string type
        int order
        int estimatedMinutes
        int xpReward
        date createdAt
        date updatedAt
    }
    
    Exercise {
        ObjectId _id PK
        string type
        int order
        string question
        string hint
        string audioUrl
        object content
        object correctAnswer
        int points
    }
    
    UserProgress {
        ObjectId _id PK
        string userId
        string targetLanguageCode
        ObjectId currentCourseId FK
        ObjectId currentUnitId FK
        ObjectId currentLessonId FK
        int lessonsCompleted
        int averageScore
        int totalTimeMinutes
        date lastActivityAt
        date createdAt
        date updatedAt
    }
    
    LessonCompletion {
        ObjectId _id PK
        string userId
        ObjectId lessonId FK
        int score
        int xpEarned
        int correctAnswers
        int totalExercises
        int timeSpentSeconds
        int attemptNumber
        date completedAt
    }
```

---

## 2. Architecture du Service

```mermaid
flowchart TB
    subgraph Frontend
        A[Angular App]
    end
    
    subgraph API Gateway
        GW[Gateway]
    end
    
    subgraph LessonService["Lesson Service"]
        direction TB
        API[REST API]
        SVC[Business Logic]
        REPO[Repositories]
        KAFKA_P[Kafka Producer]
        KAFKA_C[Kafka Consumer]
    end
    
    subgraph Databases
        MONGO[(MongoDB)]
        REDIS[(Redis Cache)]
    end
    
    subgraph Kafka
        TOPIC_USER[user.events]
        TOPIC_LESSON[lesson.events]
    end
    
    subgraph OtherServices
        AUTH[Auth Service]
        GAMIF[Gamification Service]
        RECO[Recommendation Service]
    end
    
    A --> GW
    GW --> API
    API --> SVC
    SVC --> REPO
    SVC --> KAFKA_P
    REPO --> MONGO
    REPO --> REDIS
    
    KAFKA_P --> TOPIC_LESSON
    TOPIC_USER --> KAFKA_C
    KAFKA_C --> SVC
    
    TOPIC_LESSON --> GAMIF
    TOPIC_LESSON --> RECO
    AUTH --> TOPIC_USER
```

---

## 3. Flux : Démarrer une Leçon

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant FE as Frontend
    participant GW as API Gateway
    participant LS as Lesson Service
    participant DB as MongoDB
    participant K as Kafka
    participant GS as Gamification Service

    U->>FE: Clique sur "Commencer"
    FE->>GW: POST /lessons/{id}/start (JWT)
    GW->>LS: Requête authentifiée
    
    LS->>DB: Vérifier leçon débloquée
    alt Leçon verrouillée
        LS-->>GW: 403 LESSON_LOCKED
        GW-->>FE: Erreur
        FE-->>U: "Terminez d'abord la leçon précédente"
    else Leçon accessible
        LS->>DB: Récupérer leçon + exercices
        LS->>K: Publier lesson.started
        LS-->>GW: 200 OK + Session créée
        GW-->>FE: Données de la leçon
        FE-->>U: Afficher premier exercice
        K-->>GS: Notifier activité
    end
```

---

## 4. Flux : Soumettre un Exercice

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant FE as Frontend
    participant LS as Lesson Service
    participant DB as MongoDB

    U->>FE: Soumet sa réponse
    FE->>LS: POST /exercises/{id}/submit
    
    LS->>LS: Valider la réponse
    
    alt Réponse correcte
        LS->>DB: Enregistrer succès
        LS-->>FE: {isCorrect: true, points: 10}
        FE-->>U: ✅ Bonne réponse + feedback
    else Réponse incorrecte
        LS->>DB: Incrémenter tentatives
        LS-->>FE: {isCorrect: false, correctAnswer: "..."}
        FE-->>U: ❌ Correction + explication
    end
    
    FE->>FE: Passer à l'exercice suivant
```

---

## 5. Flux : Terminer une Leçon

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant FE as Frontend
    participant LS as Lesson Service
    participant DB as MongoDB
    participant K as Kafka
    participant GS as Gamification Service
    participant RS as Recommendation Service

    U->>FE: Termine dernier exercice
    FE->>FE: Calculer score local
    FE->>LS: POST /lessons/{id}/complete
    
    LS->>LS: Calculer score final
    LS->>LS: Calculer XP gagné
    LS->>DB: Créer LessonCompletion
    LS->>DB: Mettre à jour UserProgress
    
    alt Score >= 70%
        LS->>DB: Débloquer leçon suivante
        LS-->>FE: {score, xpEarned, nextLesson}
        LS->>K: lesson.completed
        K-->>GS: Attribuer XP
        K-->>RS: Mettre à jour recommandations
        FE-->>U: 🎉 Bravo ! +15 XP
    else Score < 70%
        LS-->>FE: {score, xpEarned: 0, suggestion: "retry"}
        FE-->>U: 😕 Réessayez pour débloquer la suite
    end
```

---

## 6. Flux : Création de Progression (Event)

```mermaid
sequenceDiagram
    participant AUTH as Auth Service
    participant K as Kafka
    participant LS as Lesson Service
    participant DB as MongoDB

    Note over AUTH: Nouvel utilisateur inscrit
    AUTH->>K: user.registered
    
    K->>LS: Consumer reçoit event
    LS->>LS: Extraire learningProfiles
    
    loop Pour chaque profil langue
        LS->>DB: Vérifier si UserProgress existe
        alt N'existe pas
            LS->>DB: Créer UserProgress
            Note over DB: lessonsCompleted=0<br/>averageScore=0
        else Existe déjà
            LS->>LS: Log et ignorer (idempotent)
        end
    end
```

---

## 7. Structure Hiérarchique du Contenu

```mermaid
graph TD
    subgraph "Curriculum Anglais"
        C1[🎓 Cours A1<br/>Anglais Débutant]
        C2[🎓 Cours A2<br/>Anglais Élémentaire]
    end
    
    subgraph "Cours A1"
        U1[📁 Unité 1<br/>Salutations]
        U2[📁 Unité 2<br/>Restaurant]
        U3[📁 Unité 3<br/>Shopping]
    end
    
    subgraph "Unité 1"
        L1[📖 Leçon 1<br/>Dire bonjour]
        L2[📖 Leçon 2<br/>Se présenter]
        L3[📖 Leçon 3<br/>Demander l'heure]
    end
    
    subgraph "Leçon 1"
        E1[❓ Ex 1: QCM]
        E2[❓ Ex 2: Texte à trous]
        E3[❓ Ex 3: Écouter/Répéter]
    end
    
    C1 --> U1
    C1 --> U2
    C1 --> U3
    C2 -.-> |Débloqué après C1| C2
    
    U1 --> L1
    U1 --> L2
    U1 --> L3
    
    L1 --> E1
    L1 --> E2
    L1 --> E3
    
    style C1 fill:#4CAF50,color:#fff
    style U1 fill:#2196F3,color:#fff
    style L1 fill:#FF9800,color:#fff
    style E1 fill:#9C27B0,color:#fff
```

---

## 8. Règles de Déblocage

```mermaid
stateDiagram-v2
    [*] --> Locked: Leçon créée
    Locked --> Unlocked: Leçon précédente score ≥ 70%
    Locked --> Unlocked: Première leçon (auto)
    
    Unlocked --> InProgress: POST /start
    InProgress --> Completed: POST /complete (score ≥ 70%)
    InProgress --> Failed: POST /complete (score < 70%)
    
    Failed --> InProgress: Nouvelle tentative
    Completed --> Mastered: Score ≥ 90% sur 3 tentatives
    
    note right of Locked
        Contenu visible mais
        non accessible
    end note
    
    note right of Completed
        Déclenche déblocage
        de la leçon suivante
    end note
```
