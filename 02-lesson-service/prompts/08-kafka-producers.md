# User Story : Kafka Producers

## Contexte

Implémenter la publication d'événements Kafka pour notifier les autres services.

## Topic : `lesson.events`

**Clé de partitionnement :** `userId` (garantit l'ordre par utilisateur)

## Événements à Publier

### 1. lesson.started

Publié quand un utilisateur démarre une leçon.

**Déclencheur :** `POST /lessons/{id}/start`

**Payload :**
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

**Consommé par :** gamification-service (pour streaks)

---

### 2. lesson.completed

Publié quand un utilisateur termine une leçon.

**Déclencheur :** `POST /lessons/{id}/complete`

**Payload :**
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
- gamification-service (XP, badges)
- recommendation-service (recommandations)

---

### 3. unit.completed

Publié quand toutes les leçons d'une unité sont terminées.

**Déclencheur :** `POST /lessons/{id}/complete` (si dernière leçon de l'unité)

**Payload :**
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

---

### 4. course.completed

Publié quand toutes les unités d'un cours sont terminées.

**Déclencheur :** `POST /lessons/{id}/complete` (si dernière leçon du cours)

**Payload :**
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

## Implémentation

### Configuration (application.properties)

```properties
spring.cloud.stream.bindings.lessonEvents-out-0.destination=lesson.events
spring.cloud.stream.kafka.bindings.lessonEvents-out-0.producer.message-key-expression=headers['partitionKey']
```

### EventPublisher Service

```java
@Service
@RequiredArgsConstructor
public class LessonEventPublisher {

    private final StreamBridge streamBridge;
    
    private static final String BINDING = "lessonEvents-out-0";

    public void publishLessonStarted(LessonStartedPayload payload) {
        publish("lesson.started", payload, payload.getUserId());
    }
    
    public void publishLessonCompleted(LessonCompletedPayload payload) {
        publish("lesson.completed", payload, payload.getUserId());
    }
    
    public void publishUnitCompleted(UnitCompletedPayload payload) {
        publish("unit.completed", payload, payload.getUserId());
    }
    
    public void publishCourseCompleted(CourseCompletedPayload payload) {
        publish("course.completed", payload, payload.getUserId());
    }
    
    private <T> void publish(String eventType, T payload, String partitionKey) {
        CloudEvent<T> event = CloudEvent.<T>builder()
            .eventType(eventType)
            .version("1.0")
            .timestamp(Instant.now())
            .payload(payload)
            .metadata(Metadata.builder()
                .correlationId(UUID.randomUUID().toString())
                .source("lesson-service")
                .build())
            .build();
            
        Message<CloudEvent<T>> message = MessageBuilder
            .withPayload(event)
            .setHeader("partitionKey", partitionKey)
            .build();
            
        streamBridge.send(BINDING, message);
    }
}
```

### Utilisation dans LessonService

```java
@Service
@RequiredArgsConstructor
public class LessonServiceImpl implements LessonService {

    private final LessonEventPublisher eventPublisher;
    
    @Override
    public LessonSessionDto startLesson(String lessonId, String userId) {
        // ... logique métier ...
        
        eventPublisher.publishLessonStarted(
            LessonStartedPayload.builder()
                .userId(userId)
                .lessonId(lessonId)
                .lessonTitle(lesson.getTitle())
                .lessonType(lesson.getType())
                .targetLanguageCode(course.getTargetLanguageCode())
                .unitId(lesson.getUnitId())
                .courseId(unit.getCourseId())
                .build()
        );
        
        return session;
    }
    
    @Override
    public LessonCompletionResultDto completeLesson(...) {
        // ... logique métier ...
        
        eventPublisher.publishLessonCompleted(...);
        
        if (isLastLessonOfUnit) {
            eventPublisher.publishUnitCompleted(...);
        }
        
        if (isLastLessonOfCourse) {
            eventPublisher.publishCourseCompleted(...);
        }
        
        return result;
    }
}
```

## Critères d'Acceptation

- [ ] `lesson.started` est publié au démarrage
- [ ] `lesson.completed` est publié à la fin
- [ ] `unit.completed` est publié si dernière leçon de l'unité
- [ ] `course.completed` est publié si dernière leçon du cours
- [ ] Le partitionnement par userId fonctionne
- [ ] Tests d'intégration vérifient la publication
