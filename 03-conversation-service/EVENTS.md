# Conversation Service - Événements Kafka

## Configuration Spring Cloud Stream

```properties
# Bindings
spring.cloud.stream.bindings.sessionEventPublisher-out-0.destination=conversation.events
spring.cloud.stream.bindings.userEventListener-in-0.destination=user.events
spring.cloud.stream.bindings.userEventListener-in-0.group=conversation-service
```

---

## Événements Publiés

### Topic: `conversation.events`

#### session.started

Publié quand une session démarre.

```json
{
  "eventType": "session.started",
  "version": "1.0",
  "timestamp": "2026-01-03T14:00:00Z",
  "payload": {
    "sessionId": "session789",
    "timeSlotId": "slot123",
    "targetLanguageCode": "en",
    "level": "B1",
    "participantIds": ["user123", "user456", "user789"],
    "participantCount": 3,
    "recordingEnabled": true
  },
  "metadata": {
    "correlationId": "corr-abc123",
    "source": "conversation-service"
  }
}
```

**Consommé par:**
- `gamification-service`: Initialise le suivi de la session

**Clé de partitionnement:** `sessionId`

---

#### session.ended

Publié quand une session se termine.

```json
{
  "eventType": "session.ended",
  "version": "1.0",
  "timestamp": "2026-01-03T14:30:00Z",
  "payload": {
    "sessionId": "session789",
    "timeSlotId": "slot123",
    "targetLanguageCode": "en",
    "level": "B1",
    "participants": [
      {
        "userId": "user123",
        "joinedAt": "2026-01-03T14:00:00Z",
        "leftAt": "2026-01-03T14:30:00Z",
        "recordingConsent": true
      },
      {
        "userId": "user456",
        "joinedAt": "2026-01-03T14:00:30Z",
        "leftAt": "2026-01-03T14:30:00Z",
        "recordingConsent": true
      }
    ],
    "startedAt": "2026-01-03T14:00:00Z",
    "endedAt": "2026-01-03T14:30:00Z",
    "durationSeconds": 1800
  },
  "metadata": {
    "correlationId": "corr-abc123",
    "source": "conversation-service"
  }
}
```

**Consommé par:**
- `gamification-service`: Attribue XP aux participants

**Clé de partitionnement:** `sessionId`

---

#### session.recorded

Publié quand l'enregistrement audio est disponible sur R2.

```json
{
  "eventType": "session.recorded",
  "version": "1.0",
  "timestamp": "2026-01-03T14:31:00Z",
  "payload": {
    "sessionId": "session789",
    "targetLanguageCode": "en",
    "level": "B1",
    "recordings": [
      {
        "userId": "user123",
        "url": "r2://wespeak-recordings/2026/01/03/session789_user123.webm",
        "startTime": "2026-01-03T14:00:00Z"
      },
      {
        "userId": "user456",
        "url": "r2://wespeak-recordings/2026/01/03/session789_user456.webm",
        "startTime": "2026-01-03T14:00:00Z"
      }
    ],
    "durationSeconds": 1800,
    "participantsWithConsent": [
      {
        "userId": "user123",
        "displayName": "Marie"
      },
      {
        "userId": "user456",
        "displayName": "Jean"
      }
    ]
  },
  "metadata": {
    "correlationId": "corr-abc123",
    "source": "conversation-service"
  }
}
```

**Consommé par:**
- `feedback-service`: Lance la transcription et l'analyse IA

**Clé de partitionnement:** `sessionId`

---

## Événements Consommés

### Topic: `user.events`

#### user.deleted

Quand un utilisateur supprime son compte.

```json
{
  "eventType": "user.deleted",
  "payload": {
    "userId": "user123"
  }
}
```

**Action:** 
- Supprimer les inscriptions futures
- Anonymiser les participations passées

---

## Listeners Spring Cloud Stream

```java
@Configuration
public class ConversationEventListener {

    @Bean
    public Consumer<CloudEvent<UserDeletedPayload>> userEventListener(
            ConversationCleanupService cleanupService) {
        return event -> {
            if ("user.deleted".equals(event.getType())) {
                cleanupService.handleUserDeleted(event.getData().getUserId());
            }
        };
    }
}
```

---

## Stratégie de Partitionnement

| Topic | Clé | Raison |
|-------|-----|--------|
| conversation.events | sessionId | Tous les événements d'une session sur la même partition |
| user.events | userId | Cohérence par utilisateur |
