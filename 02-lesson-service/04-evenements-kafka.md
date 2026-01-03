# Lesson Service - Événements Kafka

## Configuration Spring Cloud Stream

```properties
# Définition des fonctions
spring.cloud.function.definition=userRegisteredListener;learningProfileCreatedListener

# Bindings - Consommation
spring.cloud.stream.bindings.userRegisteredListener-in-0.destination=user.events
spring.cloud.stream.bindings.userRegisteredListener-in-0.group=lesson-service

spring.cloud.stream.bindings.learningProfileCreatedListener-in-0.destination=user.events
spring.cloud.stream.bindings.learningProfileCreatedListener-in-0.group=lesson-service

# Bindings - Production
spring.cloud.stream.bindings.lessonEvents-out-0.destination=lesson.events
```

---

## Événements Publiés

### Topic : `lesson.events`

**Clé de partitionnement :** `userId` (garantit l'ordre des événements par utilisateur)

---

### lesson.started

Publié quand un utilisateur démarre une leçon.

```json
{
  "eventType": "lesson.started",
  "version": "1.0",
  "timestamp": "2025-01-15T14:00:00Z",
  "payload": {
    "userId": "user-123-uuid",
    "lessonId": "507f1f77bcf86cd799439101",
    "lessonTitle": "Dire bonjour",
    "lessonType": "vocabulary",
    "targetLanguageCode": "en",
    "unitId": "507f1f77bcf86cd799439012",
    "courseId": "507f1f77bcf86cd799439011"
  },
  "metadata": {
    "correlationId": "corr-abc-123",
    "source": "lesson-service"
  }
}
```

**Consommé par :**
- `gamification-service` : Pour comptabiliser l'activité quotidienne (streaks)

---

### lesson.completed

Publié quand un utilisateur termine une leçon.

```json
{
  "eventType": "lesson.completed",
  "version": "1.0",
  "timestamp": "2025-01-15T14:30:00Z",
  "payload": {
    "userId": "user-123-uuid",
    "lessonId": "507f1f77bcf86cd799439101",
    "lessonTitle": "Dire bonjour",
    "lessonType": "vocabulary",
    "targetLanguageCode": "en",
    "unitId": "507f1f77bcf86cd799439012",
    "courseId": "507f1f77bcf86cd799439011",
    "score": 85,
    "xpEarned": 15,
    "correctAnswers": 17,
    "totalExercises": 20,
    "timeSpentSeconds": 420,
    "attemptNumber": 1,
    "isFirstCompletion": true
  },
  "metadata": {
    "correlationId": "corr-abc-123",
    "source": "lesson-service"
  }
}
```

**Consommé par :**
- `gamification-service` : Pour attribuer les XP et vérifier les badges
- `recommendation-service` : Pour mettre à jour les recommandations

---

### unit.completed

Publié quand un utilisateur termine toutes les leçons d'une unité.

```json
{
  "eventType": "unit.completed",
  "version": "1.0",
  "timestamp": "2025-01-16T10:00:00Z",
  "payload": {
    "userId": "user-123-uuid",
    "unitId": "507f1f77bcf86cd799439012",
    "unitTitle": "Les Salutations",
    "courseId": "507f1f77bcf86cd799439011",
    "targetLanguageCode": "en",
    "totalLessons": 5,
    "averageScore": 88,
    "totalXPEarned": 75
  },
  "metadata": {
    "correlationId": "corr-abc-123",
    "source": "lesson-service"
  }
}
```

**Consommé par :**
- `gamification-service` : Pour les badges "Unité complétée"

---

### course.completed

Publié quand un utilisateur termine toutes les unités d'un cours.

```json
{
  "eventType": "course.completed",
  "version": "1.0",
  "timestamp": "2025-02-01T15:00:00Z",
  "payload": {
    "userId": "user-123-uuid",
    "courseId": "507f1f77bcf86cd799439011",
    "courseTitle": "Anglais pour Débutants",
    "level": "A1",
    "targetLanguageCode": "en",
    "totalUnits": 5,
    "totalLessons": 25,
    "averageScore": 90,
    "totalXPEarned": 500
  },
  "metadata": {
    "correlationId": "corr-abc-123",
    "source": "lesson-service"
  }
}
```

**Consommé par :**
- `gamification-service` : Pour les badges "Cours complété" et certificats
- `recommendation-service` : Pour recommander le cours suivant

---

## Événements Consommés

### Topic : `user.events`

---

### user.registered

Reçu quand un nouvel utilisateur s'inscrit.

```json
{
  "eventType": "user.registered",
  "version": "1.0",
  "timestamp": "2025-01-10T10:00:00Z",
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
  },
  "metadata": {
    "correlationId": "corr-xyz-789",
    "source": "auth-service"
  }
}
```

**Action :**
- Créer une entrée `UserProgress` pour chaque `learningProfile`
- Initialiser : `lessonsCompleted = 0`, `averageScore = 0`

**Implémentation :**

```java
@Bean
public Consumer<CloudEvent<UserRegisteredPayload>> userRegisteredListener(
        ProgressService progressService) {
    return event -> {
        UserRegisteredPayload payload = event.getData();
        for (LearningProfile profile : payload.getLearningProfiles()) {
            progressService.initializeProgress(
                payload.getUserId(),
                profile.getTargetLanguageCode()
            );
        }
    };
}
```

---

### learning_profile.created

Reçu quand un utilisateur ajoute une nouvelle langue à apprendre.

```json
{
  "eventType": "learning_profile.created",
  "version": "1.0",
  "timestamp": "2025-01-12T09:00:00Z",
  "payload": {
    "userId": "user-123-uuid",
    "profileId": "profile-789-uuid",
    "targetLanguageCode": "es",
    "nativeLanguageCode": "fr",
    "currentLevel": "A1"
  },
  "metadata": {
    "correlationId": "corr-def-456",
    "source": "auth-service"
  }
}
```

**Action :**
- Créer une nouvelle entrée `UserProgress` pour cette langue
- L'utilisateur peut maintenant suivre le curriculum espagnol

**Implémentation :**

```java
@Bean
public Consumer<CloudEvent<LearningProfileCreatedPayload>> learningProfileCreatedListener(
        ProgressService progressService) {
    return event -> {
        LearningProfileCreatedPayload payload = event.getData();
        progressService.initializeProgress(
            payload.getUserId(),
            payload.getTargetLanguageCode()
        );
    };
}
```

---

## Schéma CloudEvent

Tous les événements suivent le format CloudEvent :

```java
public class CloudEvent<T> {
    private String eventType;      // Type d'événement
    private String version;        // Version du schéma
    private Instant timestamp;     // Date/heure
    private T payload;             // Données métier
    private Metadata metadata;     // Métadonnées
}

public class Metadata {
    private String correlationId;  // Pour traçage distribué
    private String source;         // Service émetteur
}
```

---

## Garanties et Bonnes Pratiques

### Idempotence

Tous les consumers doivent être **idempotents** :

```java
// Vérifier si la progression existe déjà avant de créer
public void initializeProgress(String userId, String languageCode) {
    if (progressRepository.existsByUserIdAndLanguage(userId, languageCode)) {
        log.info("Progress already exists for user {} and language {}", userId, languageCode);
        return;
    }
    // Créer la progression...
}
```

### Retry et DLQ

```properties
# Retry 3 fois avec backoff exponentiel
spring.cloud.stream.bindings.userRegisteredListener-in-0.consumer.max-attempts=3
spring.cloud.stream.bindings.userRegisteredListener-in-0.consumer.back-off-initial-interval=1000
spring.cloud.stream.bindings.userRegisteredListener-in-0.consumer.back-off-multiplier=2

# Dead Letter Queue pour messages en échec
spring.cloud.stream.kafka.bindings.userRegisteredListener-in-0.consumer.enable-dlq=true
```

### Ordre des Messages

Le partitionnement par `userId` garantit que tous les événements d'un même utilisateur sont traités **dans l'ordre**.
