# User Story: Événements Kafka

## Contexte
Le conversation-service doit communiquer avec les autres services via Kafka.

## Objectif
Implémenter la publication et consommation des événements Kafka.

## Événements à publier

### 1. session.started
- **Quand** : Session passe à `active`
- **Topic** : `conversation.events`
- **Clé** : sessionId
- **Payload** : sessionId, timeSlotId, targetLanguageCode, level, participantIds, recordingEnabled

### 2. session.ended
- **Quand** : Session passe à `ended`
- **Topic** : `conversation.events`
- **Clé** : sessionId
- **Payload** : sessionId, participants (avec joinedAt, leftAt, recordingConsent), durationSeconds

### 3. session.recorded
- **Quand** : Enregistrement uploadé sur R2
- **Topic** : `conversation.events`
- **Clé** : sessionId
- **Payload** : sessionId, recordingUrl, targetLanguageCode, level, participantsWithConsent

## Événements à consommer

### 1. user.deleted (depuis auth-service)
- **Action** : Supprimer inscriptions futures, anonymiser participations passées

## Implémentation

### Publisher
```java
@Component
public class ConversationEventPublisher {
    private final StreamBridge streamBridge;
    
    public void publishSessionStarted(Session session) {
        // Créer CloudEvent et publier
    }
}
```

### Consumer
```java
@Bean
public Consumer<CloudEvent<UserDeletedPayload>> userEventListener(
        ConversationCleanupService service) {
    return event -> service.handleUserDeleted(event.getData().getUserId());
}
```

## Critères d'acceptation
- [ ] session.started est publié quand la session devient active
- [ ] session.ended est publié quand la session se termine
- [ ] session.recorded est publié après upload R2
- [ ] user.deleted déclenche le nettoyage des données

## Stack technique
- Spring Cloud Stream avec Kafka binder
- CloudEvent comme enveloppe
