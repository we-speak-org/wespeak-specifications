# Conversation Service - Événements Kafka

## Configuration Spring Cloud Stream

```properties
# Bindings
spring.cloud.stream.bindings.sessionCompletedPublisher-out-0.destination=conversation.events
spring.cloud.stream.bindings.userEventListener-in-0.destination=user.events
spring.cloud.stream.bindings.userEventListener-in-0.group=conversation-service
spring.cloud.stream.bindings.feedbackEventListener-in-0.destination=feedback.events
spring.cloud.stream.bindings.feedbackEventListener-in-0.group=conversation-service
```

---

## Événements Publiés

### Topic: `conversation.events`

#### session.completed

Publié quand une session de conversation se termine avec succès (durée ≥ 2 minutes).

```json
{
  "eventType": "session.completed",
  "version": "1.0",
  "timestamp": "2026-01-03T10:09:02Z",
  "payload": {
    "sessionId": "session789",
    "topicId": "topic123",
    "targetLanguageCode": "en",
    "participant1Id": "user123",
    "participant2Id": "user456",
    "startedAt": "2026-01-03T10:00:00Z",
    "endedAt": "2026-01-03T10:09:02Z",
    "actualDurationSeconds": 542,
    "audioRecordingUrl": "s3://wespeak-audio/sessions/session789.webm"
  },
  "metadata": {
    "correlationId": "corr-abc123",
    "source": "conversation-service"
  }
}
```

**Consommé par:**
- `feedback-service`: Lance l'analyse STT/NLP de l'audio
- `gamification-service`: Attribue XP aux deux participants

**Clé de partitionnement:** `sessionId`

---

#### session.cancelled

Publié quand une session est annulée ou abandonnée.

```json
{
  "eventType": "session.cancelled",
  "version": "1.0",
  "timestamp": "2026-01-03T10:03:00Z",
  "payload": {
    "sessionId": "session790",
    "participant1Id": "user123",
    "participant2Id": "user456",
    "cancelledByUserId": "user123",
    "reason": "dropped",
    "durationBeforeCancelSeconds": 45
  },
  "metadata": {
    "correlationId": "corr-def456",
    "source": "conversation-service"
  }
}
```

**Raisons possibles:**
- `dropped`: Déconnexion d'un participant
- `timeout`: Inactivité prolongée
- `cancelled`: Annulation volontaire avant le début
- `reported`: Session signalée pour abus

---

#### match.found

Publié quand deux utilisateurs sont matchés avec succès.

```json
{
  "eventType": "match.found",
  "version": "1.0",
  "timestamp": "2026-01-03T10:00:00Z",
  "payload": {
    "sessionId": "session789",
    "participant1Id": "user123",
    "participant2Id": "user456",
    "targetLanguageCode": "en",
    "topicId": "topic123",
    "matchDurationMs": 15230
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

Quand un utilisateur supprime son compte, nettoyer ses données.

```json
{
  "eventType": "user.deleted",
  "payload": {
    "userId": "user123"
  }
}
```

**Action:** Supprimer toutes les sessions et enregistrements de l'utilisateur.

---

#### user.profile.updated

Quand le profil d'apprentissage change (niveau, langue).

```json
{
  "eventType": "user.profile.updated",
  "payload": {
    "userId": "user123",
    "learningProfileId": "profile123",
    "targetLanguageCode": "en",
    "newLevel": "B2"
  }
}
```

**Action:** Mettre à jour le cache des niveaux utilisateur.

---

### Topic: `feedback.events`

#### feedback.completed

Quand l'analyse d'une session est terminée.

```json
{
  "eventType": "feedback.completed",
  "payload": {
    "sessionId": "session789",
    "feedbackId": "feedback123",
    "overallScore": 75
  }
}
```

**Action:** Associer le feedbackId à la session.

---

## Listeners Spring Cloud Stream

```java
@Configuration
public class ConversationEventListener {

    @Bean
    public Consumer<CloudEvent<UserDeletedPayload>> userEventListener(
            ConversationService conversationService) {
        return event -> {
            if ("user.deleted".equals(event.getType())) {
                conversationService.handleUserDeleted(event.getData());
            }
        };
    }

    @Bean
    public Consumer<CloudEvent<FeedbackCompletedPayload>> feedbackEventListener(
            SessionService sessionService) {
        return event -> {
            sessionService.linkFeedback(
                event.getData().getSessionId(),
                event.getData().getFeedbackId()
            );
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
| feedback.events | sessionId | Alignement avec les sessions |
