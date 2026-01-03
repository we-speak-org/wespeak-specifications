# User Story : Kafka Listeners

## Contexte

Implémenter les consumers Kafka pour réagir aux événements des autres services.

## Architecture

Utiliser **Spring Cloud Stream** avec le style fonctionnel :

```java
@Bean
public Consumer<CloudEvent<PayloadType>> listenerName(Service service) {
    return event -> service.handleEvent(event.getData());
}
```

## Événements à Consommer

### Topic : `user.events`

---

### 1. user.registered

Reçu quand un nouvel utilisateur s'inscrit.

**Payload :**
```json
{
  "eventType": "user.registered",
  "payload": {
    "userId": "user-123-uuid",
    "email": "john@example.com",
    "displayName": "John Doe",
    "learningProfiles": [
      {
        "id": "profile-456-uuid",
        "targetLanguageCode": "en",
        "nativeLanguageCode": "fr",
        "currentLevel": "A1"
      }
    ]
  }
}
```

**Action :**
Pour chaque `learningProfile`, créer une entrée `UserProgress` :
- `userId`: de l'événement
- `targetLanguageCode`: du profil
- `lessonsCompleted`: 0
- `averageScore`: 0
- `totalTimeMinutes`: 0

**Idempotence :**
Vérifier si `UserProgress` existe déjà avant de créer.

---

### 2. learning_profile.created

Reçu quand un utilisateur ajoute une nouvelle langue à apprendre.

**Payload :**
```json
{
  "eventType": "learning_profile.created",
  "payload": {
    "userId": "user-123-uuid",
    "profileId": "profile-789-uuid",
    "targetLanguageCode": "es",
    "nativeLanguageCode": "fr",
    "currentLevel": "A1"
  }
}
```

**Action :**
Créer une nouvelle entrée `UserProgress` pour cette langue.

## Implémentation

### Configuration (application.properties)

```properties
spring.cloud.function.definition=userRegisteredListener;learningProfileCreatedListener

spring.cloud.stream.bindings.userRegisteredListener-in-0.destination=user.events
spring.cloud.stream.bindings.userRegisteredListener-in-0.group=lesson-service

spring.cloud.stream.bindings.learningProfileCreatedListener-in-0.destination=user.events
spring.cloud.stream.bindings.learningProfileCreatedListener-in-0.group=lesson-service
```

### Listener userRegisteredListener

```java
@Configuration
public class UserEventListener {

    @Bean
    public Consumer<CloudEvent<UserRegisteredPayload>> userRegisteredListener(
            ProgressService progressService) {
        return event -> {
            if (!"user.registered".equals(event.getEventType())) {
                return; // Ignorer autres types
            }
            UserRegisteredPayload payload = event.getData();
            for (LearningProfile profile : payload.getLearningProfiles()) {
                progressService.initializeProgress(
                    payload.getUserId(),
                    profile.getTargetLanguageCode()
                );
            }
        };
    }
}
```

### Listener learningProfileCreatedListener

```java
@Bean
public Consumer<CloudEvent<LearningProfileCreatedPayload>> learningProfileCreatedListener(
        ProgressService progressService) {
    return event -> {
        if (!"learning_profile.created".equals(event.getEventType())) {
            return;
        }
        LearningProfileCreatedPayload payload = event.getData();
        progressService.initializeProgress(
            payload.getUserId(),
            payload.getTargetLanguageCode()
        );
    };
}
```

## DTOs à Créer

### CloudEvent<T>

```java
public class CloudEvent<T> {
    private String eventType;
    private String version;
    private Instant timestamp;
    private T payload;
    private Metadata metadata;
}

public class Metadata {
    private String correlationId;
    private String source;
}
```

### UserRegisteredPayload

```java
public class UserRegisteredPayload {
    private String userId;
    private String email;
    private String displayName;
    private List<LearningProfile> learningProfiles;
}
```

### LearningProfileCreatedPayload

```java
public class LearningProfileCreatedPayload {
    private String userId;
    private String profileId;
    private String targetLanguageCode;
    private String nativeLanguageCode;
    private String currentLevel;
}
```

## Retry et DLQ

```properties
# Retry 3 fois
spring.cloud.stream.bindings.userRegisteredListener-in-0.consumer.max-attempts=3
spring.cloud.stream.bindings.userRegisteredListener-in-0.consumer.back-off-initial-interval=1000
spring.cloud.stream.bindings.userRegisteredListener-in-0.consumer.back-off-multiplier=2

# Dead Letter Queue
spring.cloud.stream.kafka.bindings.userRegisteredListener-in-0.consumer.enable-dlq=true
```

## Critères d'Acceptation

- [ ] Les listeners utilisent le style fonctionnel Spring Cloud Stream
- [ ] `user.registered` crée une progression par langue
- [ ] `learning_profile.created` crée une nouvelle progression
- [ ] Les handlers sont idempotents
- [ ] Les erreurs sont loggées et envoyées en DLQ
- [ ] Tests d'intégration avec Kafka embarqué
