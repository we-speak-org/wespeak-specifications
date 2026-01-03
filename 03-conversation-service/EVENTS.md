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

Publié quand une session démarre (tous les participants connectés).

```json
{
  "eventType": "session.started",
  "version": "1.0",
  "timestamp": "2026-01-03T10:00:00Z",
  "payload": {
    "sessionId": "session789",
    "targetLanguageCode": "en",
    "topicId": "topic123",
    "participantIds": ["user123", "user456", "user789"],
    "participantCount": 3
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
  "timestamp": "2026-01-03T10:25:00Z",
  "payload": {
    "sessionId": "session789",
    "targetLanguageCode": "en",
    "topicId": "topic123",
    "participants": [
      {
        "userId": "user123",
        "speakingTimeSeconds": 450,
        "joinedAt": "2026-01-03T10:00:00Z",
        "leftAt": "2026-01-03T10:25:00Z"
      },
      {
        "userId": "user456",
        "speakingTimeSeconds": 380,
        "joinedAt": "2026-01-03T10:00:30Z",
        "leftAt": "2026-01-03T10:25:00Z"
      }
    ],
    "startedAt": "2026-01-03T10:00:00Z",
    "endedAt": "2026-01-03T10:25:00Z",
    "durationSeconds": 1500,
    "endReason": "completed"
  },
  "metadata": {
    "correlationId": "corr-abc123",
    "source": "conversation-service"
  }
}
```

**Consommé par:**
- `gamification-service`: Attribue XP aux participants
- `feedback-service`: Lance l'analyse si enregistrement disponible

**Clé de partitionnement:** `sessionId`

**Raisons de fin (endReason):**
- `completed`: Fin normale (host a terminé ou tous partis)
- `timeout`: Durée max atteinte (30 min)
- `host_ended`: Le host a mis fin manuellement

---

#### participant.joined

Publié quand un participant rejoint une session.

```json
{
  "eventType": "participant.joined",
  "version": "1.0",
  "timestamp": "2026-01-03T10:00:30Z",
  "payload": {
    "sessionId": "session789",
    "userId": "user456",
    "role": "participant",
    "currentParticipantCount": 2
  },
  "metadata": {
    "correlationId": "corr-def456",
    "source": "conversation-service"
  }
}
```

---

#### participant.left

Publié quand un participant quitte une session.

```json
{
  "eventType": "participant.left",
  "version": "1.0",
  "timestamp": "2026-01-03T10:20:00Z",
  "payload": {
    "sessionId": "session789",
    "userId": "user789",
    "speakingTimeSeconds": 280,
    "remainingParticipantCount": 2
  },
  "metadata": {
    "correlationId": "corr-ghi789",
    "source": "conversation-service"
  }
}
```

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
- Supprimer l'historique des participations
- Anonymiser les références dans les sessions passées

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
