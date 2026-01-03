# User Story: Événements Kafka

## Contexte
Le conversation-service doit publier des événements pour informer les autres services (feedback, gamification) et consommer des événements pour réagir aux changements utilisateurs.

## Critères d'Acceptation

### AC1: Publication session.completed
- Publié sur topic `conversation.events`
- Contient: sessionId, topicId, participants, durée, audioUrl
- Clé de partitionnement: sessionId
- Déclenché uniquement si durée >= 2 minutes

### AC2: Publication session.cancelled
- Publié sur topic `conversation.events`
- Contient: sessionId, participants, reason, durée avant annulation
- Raisons: dropped, timeout, cancelled, reported

### AC3: Consommation user.deleted
- Écoute topic `user.events`
- Quand user.deleted reçu:
  - Supprimer les sessions où l'utilisateur est participant
  - Supprimer les enregistrements audio associés (S3)
  - Annuler les matchmaking requests en cours

### AC4: Consommation feedback.completed
- Écoute topic `feedback.events`
- Associer le feedbackId à la session correspondante

## Tâches Techniques

1. Configurer Spring Cloud Stream avec Kafka binder
2. Créer les DTOs CloudEvent pour chaque type d'événement:
   - SessionCompletedPayload
   - SessionCancelledPayload
   - UserDeletedPayload
   - FeedbackCompletedPayload
3. Créer ConversationEventPublisher:
   ```java
   @Component
   public class ConversationEventPublisher {
       private final StreamBridge streamBridge;
       
       public void publishSessionCompleted(ConversationSession session) {
           // Build CloudEvent and send
       }
   }
   ```
4. Créer les listeners fonctionnels:
   ```java
   @Bean
   public Consumer<CloudEvent<UserDeletedPayload>> userEventListener(...) {
       return event -> { ... };
   }
   ```
5. Configurer application.properties:
   ```properties
   spring.cloud.function.definition=userEventListener;feedbackEventListener
   spring.cloud.stream.bindings.userEventListener-in-0.destination=user.events
   spring.cloud.stream.bindings.userEventListener-in-0.group=conversation-service
   ```
6. Ajouter retry et DLQ pour les consumers
7. Tests d'intégration avec Kafka embarqué

## Format CloudEvent
```json
{
  "specversion": "1.0",
  "type": "session.completed",
  "source": "conversation-service",
  "id": "evt-uuid",
  "time": "2026-01-03T10:00:00Z",
  "datacontenttype": "application/json",
  "data": { ... payload ... }
}
```

## Definition of Done
- [ ] Events publiés correctement
- [ ] Consumers fonctionnels
- [ ] Retry configuré (3 tentatives)
- [ ] DLQ pour messages en échec
- [ ] Tests avec Testcontainers Kafka
