# Recommendation Service - Événements Kafka

## Configuration Spring Cloud Stream

```properties
# Consumers
spring.cloud.stream.bindings.lessonCompletedListener-in-0.destination=lesson.events
spring.cloud.stream.bindings.lessonCompletedListener-in-0.group=recommendation-service
spring.cloud.stream.bindings.feedbackGeneratedListener-in-0.destination=feedback.events
spring.cloud.stream.bindings.feedbackGeneratedListener-in-0.group=recommendation-service
spring.cloud.stream.bindings.userProfileUpdatedListener-in-0.destination=user.events
spring.cloud.stream.bindings.userProfileUpdatedListener-in-0.group=recommendation-service

# Producers
spring.cloud.stream.bindings.recommendationProducer-out-0.destination=recommendation.events

spring.cloud.function.definition=lessonCompletedListener;feedbackGeneratedListener;userProfileUpdatedListener
```

---

## Événements Consommés

### lesson.completed

**Topic:** `lesson.events`  
**Consumer Group:** `recommendation-service`

```json
{
  "eventType": "lesson.completed",
  "version": "1.0",
  "timestamp": "2026-01-03T10:30:00Z",
  "payload": {
    "lessonId": "lesson-uuid",
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "score": 85,
    "isPerfect": false
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "lesson-service"
  }
}
```

**Action:**
1. Ajouter lessonId à completedLessonIds dans LearningHistory
2. Mettre à jour lastLessonId
3. Recalculer averageScore
4. Invalider les recommandations "next_lesson" actuelles
5. Générer nouvelle recommandation "next_lesson"

---

### feedback.generated

**Topic:** `feedback.events`  
**Consumer Group:** `recommendation-service`

```json
{
  "eventType": "feedback.generated",
  "version": "1.0",
  "timestamp": "2026-01-03T10:35:00Z",
  "payload": {
    "feedbackId": "feedback-uuid",
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "sourceType": "lesson",
    "sourceId": "lesson-uuid",
    "analysis": {
      "errors": [
        {
          "category": "grammar",
          "subcategory": "past_tense",
          "count": 2
        }
      ],
      "strengths": [
        {
          "category": "vocabulary",
          "subcategory": "travel",
          "score": 95
        }
      ]
    }
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "feedback-service"
  }
}
```

**Action:**
1. Mettre à jour weakAreas dans LearningHistory (incrémenter errorCount)
2. Mettre à jour strongAreas
3. Si nouvelle faiblesse significative → créer recommandation "revision"

---

### user.profile.updated

**Topic:** `user.events`  
**Consumer Group:** `recommendation-service`

```json
{
  "eventType": "user.profile.updated",
  "version": "1.0",
  "timestamp": "2026-01-03T09:00:00Z",
  "payload": {
    "userId": "user-uuid",
    "changes": {
      "learningProfiles": [
        {
          "targetLanguageCode": "en",
          "currentLevel": "B1",
          "weeklyGoalMinutes": 60
        }
      ]
    }
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "auth-service"
  }
}
```

**Action:**
1. Mettre à jour UserPreferences.dailyGoalMinutes si changé
2. Recalculer recommandations si niveau changé

---

## Événements Publiés

### recommendation.generated

**Topic:** `recommendation.events`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "recommendation.generated",
  "version": "1.0",
  "timestamp": "2026-01-03T10:30:00Z",
  "payload": {
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "recommendations": [
      {
        "id": "rec-uuid",
        "type": "next_lesson",
        "targetId": "lesson-uuid",
        "title": "Leçon 6: À l'hôtel",
        "priority": 1
      }
    ],
    "count": 1
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "recommendation-service"
  }
}
```

---

### recommendation.clicked

**Topic:** `recommendation.events`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "recommendation.clicked",
  "version": "1.0",
  "timestamp": "2026-01-03T10:35:00Z",
  "payload": {
    "userId": "user-uuid",
    "recommendationId": "rec-uuid",
    "type": "next_lesson",
    "targetId": "lesson-uuid",
    "targetType": "lesson",
    "clickedAt": "2026-01-03T10:35:00Z",
    "ageSeconds": 300
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "recommendation-service"
  }
}
```

---

## Listeners Spring Cloud Stream

```java
@Configuration
public class RecommendationListeners {

    @Bean
    public Consumer<CloudEvent<LessonCompletedPayload>> lessonCompletedListener(
            LearningHistoryService learningHistoryService,
            RecommendationEngine recommendationEngine) {
        return event -> {
            learningHistoryService.updateOnLessonCompleted(event.getData());
            recommendationEngine.refreshRecommendations(
                event.getData().getUserId(), 
                event.getData().getTargetLanguageCode()
            );
        };
    }

    @Bean
    public Consumer<CloudEvent<FeedbackGeneratedPayload>> feedbackGeneratedListener(
            LearningHistoryService learningHistoryService) {
        return event -> learningHistoryService.updateOnFeedback(event.getData());
    }

    @Bean
    public Consumer<CloudEvent<UserProfileUpdatedPayload>> userProfileUpdatedListener(
            UserPreferencesService preferencesService) {
        return event -> preferencesService.syncFromUserProfile(event.getData());
    }
}
```
