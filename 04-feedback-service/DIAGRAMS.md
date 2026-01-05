# Feedback Service - Diagrammes

## Diagramme ERD

```mermaid
erDiagram
    Transcript ||--|| Feedback : generates
    Transcript ||--|{ TranscriptSegment : contains
    Feedback ||--|{ FeedbackError : contains
    User ||--|{ Feedback : receives
    User ||--|| UserFeedbackStats : has
    Session ||--|{ Transcript : produces
    Recording ||--|| Transcript : transcribed_to

    Transcript {
        string id PK
        string sessionId FK
        string participantId FK
        string recordingId FK
        string targetLanguageCode
        string content
        int duration
        int wordCount
        float confidence
        string status
        datetime createdAt
        datetime completedAt
    }

    TranscriptSegment {
        float startTime
        float endTime
        string text
        float confidence
    }

    Feedback {
        string id PK
        string transcriptId FK
        string userId FK
        string sessionId FK
        string targetLanguageCode
        int overallScore
        int grammarScore
        int vocabularyScore
        int fluencyScore
        int pronunciationScore
        string summary
        int xpAwarded
        string status
        datetime createdAt
        datetime completedAt
    }

    FeedbackError {
        string type
        string original
        string correction
        string explanation
        string severity
        int segmentIndex
    }

    UserFeedbackStats {
        string id PK
        string userId FK
        string targetLanguageCode
        int totalSessions
        int totalMinutes
        float averageOverallScore
        float averageGrammarScore
        float averageVocabularyScore
        float averageFluencyScore
        string progressTrend
        datetime lastFeedbackAt
        datetime updatedAt
    }
```

## Diagramme de Séquence - Pipeline de Feedback

```mermaid
sequenceDiagram
    participant CS as Conversation Service
    participant Kafka as Kafka
    participant FS as Feedback Service
    participant R2 as Cloudflare R2
    participant Whisper as Whisper API
    participant LLM as Claude/GPT API
    participant GS as Gamification Service

    CS->>Kafka: recording.uploaded
    Kafka->>FS: Consume event
    
    FS->>FS: Create Transcript (PENDING)
    FS->>R2: Fetch audio file
    R2-->>FS: Audio data
    
    FS->>Whisper: Transcribe audio
    Whisper-->>FS: Transcription + segments
    
    FS->>FS: Save Transcript (COMPLETED)
    FS->>Kafka: transcript.completed
    
    FS->>FS: Create Feedback (PENDING)
    FS->>LLM: Analyze transcript
    Note over FS,LLM: Prompt with context:<br/>- User level<br/>- Target language<br/>- Transcript content
    
    LLM-->>FS: Analysis JSON
    Note over FS: Parse scores, errors,<br/>strengths, improvements
    
    FS->>FS: Calculate XP
    FS->>FS: Update UserFeedbackStats
    FS->>FS: Save Feedback (COMPLETED)
    
    FS->>Kafka: feedback.generated
    FS->>Kafka: xp.awarded
    
    Kafka->>GS: Consume xp.awarded
    GS->>GS: Add XP to user
```

## Diagramme de Séquence - Consultation Feedback

```mermaid
sequenceDiagram
    participant User as User (App)
    participant API as API Gateway
    participant FS as Feedback Service
    participant DB as MongoDB

    User->>API: GET /feedback/feedbacks/me
    API->>FS: Forward request
    
    FS->>DB: Find feedbacks by userId
    DB-->>FS: Feedback list
    
    FS-->>API: FeedbackListResponse
    API-->>User: 200 OK + feedbacks

    User->>API: GET /feedback/feedbacks/{id}
    API->>FS: Forward request
    
    FS->>DB: Find feedback by id
    DB-->>FS: Feedback details
    
    alt User owns feedback
        FS-->>API: FeedbackResponse
        API-->>User: 200 OK + feedback details
    else User doesn't own feedback
        FS-->>API: 403 Forbidden
        API-->>User: 403 Forbidden
    end
```

## Diagramme d'Architecture

```mermaid
flowchart TB
    subgraph External["Services Externes"]
        R2[(Cloudflare R2)]
        Whisper[Whisper API]
        LLM[Claude/GPT API]
    end

    subgraph Kafka["Message Broker"]
        RecordingTopic[recording.events]
        TranscriptTopic[transcript.events]
        FeedbackTopic[feedback.events]
        GamificationTopic[gamification.events]
    end

    subgraph FeedbackService["Feedback Service"]
        Controller[REST Controller]
        Listener[Kafka Listener]
        TranscriptionSvc[Transcription Service]
        AnalysisSvc[Analysis Service]
        StatsSvc[Stats Service]
        Repository[(MongoDB)]
    end

    subgraph OtherServices["Autres Services"]
        ConvService[Conversation Service]
        GamService[Gamification Service]
    end

    ConvService -->|upload| R2
    ConvService -->|publish| RecordingTopic
    
    RecordingTopic -->|consume| Listener
    Listener --> TranscriptionSvc
    
    TranscriptionSvc -->|fetch audio| R2
    TranscriptionSvc -->|transcribe| Whisper
    TranscriptionSvc -->|save| Repository
    TranscriptionSvc -->|publish| TranscriptTopic
    
    TranscriptionSvc --> AnalysisSvc
    AnalysisSvc -->|analyze| LLM
    AnalysisSvc -->|save| Repository
    AnalysisSvc -->|publish| FeedbackTopic
    AnalysisSvc -->|publish| GamificationTopic
    
    GamificationTopic -->|consume| GamService
    
    Controller -->|read| Repository
    Controller --> StatsSvc
    StatsSvc -->|read/write| Repository
```

## Diagramme d'État - Transcript

```mermaid
stateDiagram-v2
    [*] --> PENDING: recording.uploaded received
    
    PENDING --> PROCESSING: Start transcription
    PROCESSING --> COMPLETED: Whisper success
    PROCESSING --> FAILED: Whisper error
    
    FAILED --> PENDING: Retry (max 3)
    FAILED --> [*]: Max retries reached
    
    COMPLETED --> [*]: Trigger analysis
```

## Diagramme d'État - Feedback

```mermaid
stateDiagram-v2
    [*] --> PENDING: transcript.completed
    
    PENDING --> PROCESSING: Start analysis
    PROCESSING --> COMPLETED: LLM success
    PROCESSING --> FAILED: LLM error
    
    FAILED --> PENDING: Retry (max 3)
    FAILED --> [*]: Max retries reached
    
    COMPLETED --> [*]: Publish events
```
