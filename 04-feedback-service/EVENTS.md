# Feedback Service - Événements Kafka

## Topics

| Topic | Producteur | Description |
|-------|------------|-------------|
| `recording.events` | conversation-service | Événements des enregistrements audio |
| `transcript.events` | feedback-service | Événements des transcriptions |
| `feedback.events` | feedback-service | Événements des feedbacks générés |
| `gamification.events` | feedback-service | Événements XP vers gamification |

---

## Événements Consommés

### recording.uploaded

Déclenche le pipeline de transcription quand un enregistrement est uploadé.

**Topic** : `recording.events`
**Clé de partitionnement** : `sessionId`

```json
{
  "eventType": "recording.uploaded",
  "version": "1.0",
  "timestamp": "2025-01-15T10:30:00Z",
  "payload": {
    "recordingId": "rec-uuid-123",
    "sessionId": "session-uuid-456",
    "participantId": "user-uuid-789",
    "targetLanguageCode": "en",
    "recordings": [
      {
        "userId": "user-uuid-789",
        "url": "r2://wespeak-recordings/session-uuid/user-uuid.webm",
        "startTime": "2025-01-15T10:30:00Z"
      }
    ],
    "duration": 125,
    "format": "webm",
    "size": 2500000
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "conversation-service"
  }
}
```

**Action** : Créer un Transcript en status PENDING et démarrer la transcription Whisper.

---

## Événements Publiés

### transcript.completed

Publié quand une transcription est terminée avec succès.

**Topic** : `transcript.events`
**Clé de partitionnement** : `sessionId`

```json
{
  "eventType": "transcript.completed",
  "version": "1.0",
  "timestamp": "2025-01-15T10:31:00Z",
  "payload": {
    "transcriptId": "trans-uuid-123",
    "sessionId": "session-uuid-456",
    "participantId": "user-uuid-789",
    "targetLanguageCode": "en",
    "wordCount": 145,
    "duration": 125,
    "confidence": 0.93
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "feedback-service"
  }
}
```

### transcript.failed

Publié quand une transcription échoue.

**Topic** : `transcript.events`
**Clé de partitionnement** : `sessionId`

```json
{
  "eventType": "transcript.failed",
  "version": "1.0",
  "timestamp": "2025-01-15T10:31:00Z",
  "payload": {
    "transcriptId": "trans-uuid-123",
    "sessionId": "session-uuid-456",
    "participantId": "user-uuid-789",
    "recordingId": "rec-uuid-123",
    "errorCode": "WHISPER_API_ERROR",
    "errorMessage": "Service temporarily unavailable"
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "feedback-service"
  }
}
```

### feedback.generated

Publié quand un feedback est généré avec succès.

**Topic** : `feedback.events`
**Clé de partitionnement** : `userId`

```json
{
  "eventType": "feedback.generated",
  "version": "1.0",
  "timestamp": "2025-01-15T10:32:00Z",
  "payload": {
    "feedbackId": "fb-uuid-123",
    "transcriptId": "trans-uuid-123",
    "sessionId": "session-uuid-456",
    "userId": "user-uuid-789",
    "targetLanguageCode": "en",
    "overallScore": 72,
    "grammarScore": 68,
    "vocabularyScore": 75,
    "fluencyScore": 78,
    "xpAwarded": 25,
    "errorsCount": 5,
    "progressTrend": "IMPROVING"
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "feedback-service"
  }
}
```

### xp.awarded

Publié pour notifier le gamification-service des XP à attribuer.

**Topic** : `gamification.events`
**Clé de partitionnement** : `userId`

```json
{
  "eventType": "xp.awarded",
  "version": "1.0",
  "timestamp": "2025-01-15T10:32:00Z",
  "payload": {
    "userId": "user-uuid-789",
    "amount": 25,
    "source": "CONVERSATION_FEEDBACK",
    "sourceId": "fb-uuid-123",
    "targetLanguageCode": "en",
    "breakdown": {
      "participation": 10,
      "scoreBonus": 5,
      "durationBonus": 10
    }
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "feedback-service"
  }
}
```

---

## Configuration Spring Cloud Stream

```properties
# Kafka bindings
spring.cloud.function.definition=recordingUploadedListener

# Consumer - Recording events
spring.cloud.stream.bindings.recordingUploadedListener-in-0.destination=recording.events
spring.cloud.stream.bindings.recordingUploadedListener-in-0.group=feedback-service

# Producer - Transcript events
spring.cloud.stream.bindings.transcriptOutput-out-0.destination=transcript.events

# Producer - Feedback events
spring.cloud.stream.bindings.feedbackOutput-out-0.destination=feedback.events

# Producer - Gamification events (XP)
spring.cloud.stream.bindings.gamificationOutput-out-0.destination=gamification.events

# Consumer configuration
spring.cloud.stream.kafka.bindings.recordingUploadedListener-in-0.consumer.enableDlq=true
spring.cloud.stream.kafka.bindings.recordingUploadedListener-in-0.consumer.dlqName=recording.events.dlq
spring.cloud.stream.kafka.bindings.recordingUploadedListener-in-0.consumer.autoCommitOnError=false
```

---

## Listener Implementation

```java
@Configuration
public class KafkaListenerConfig {

    @Bean
    public Consumer<CloudEvent<RecordingUploadedPayload>> recordingUploadedListener(
            TranscriptionService transcriptionService) {
        return event -> transcriptionService.processRecording(event.getData());
    }
}
```

---

## Diagramme de Flux

```
conversation-service                    feedback-service                    gamification-service
       │                                       │                                    │
       │  recording.uploaded                   │                                    │
       ├──────────────────────────────────────▶│                                    │
       │                                       │                                    │
       │                                       │ ┌─────────────────────┐            │
       │                                       │ │ 1. Fetch audios R2    │            │
       │                                       │ │ 2. Call AssemblyAI  │            │
       │                                       │ │ 3. Merge transcripts  │            │
       │                                       │ └─────────────────────┘            │
       │                                       │                                    │
       │                                       │  transcript.completed              │
       │                                       ├────────────────────────────────────│
       │                                       │                                    │
       │                                       │ ┌─────────────────────┐            │
       │                                       │ │ 4. Call LLM API     │            │
       │                                       │ │ 5. Generate Feedback│            │
       │                                       │ │ 6. Calculate XP     │            │
       │                                       │ └─────────────────────┘            │
       │                                       │                                    │
       │                                       │  feedback.generated                │
       │                                       ├────────────────────────────────────│
       │                                       │                                    │
       │                                       │  xp.awarded                        │
       │                                       ├───────────────────────────────────▶│
       │                                       │                                    │
```
