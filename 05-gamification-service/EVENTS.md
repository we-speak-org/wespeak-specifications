# Gamification Service - Événements Kafka

## Configuration Spring Cloud Stream

```properties
# Bindings
spring.cloud.stream.bindings.lessonCompletedListener-in-0.destination=lesson.events
spring.cloud.stream.bindings.lessonCompletedListener-in-0.group=gamification-service
spring.cloud.stream.bindings.exerciseCompletedListener-in-0.destination=lesson.events
spring.cloud.stream.bindings.exerciseCompletedListener-in-0.group=gamification-service
spring.cloud.stream.bindings.conversationEndedListener-in-0.destination=conversation.events
spring.cloud.stream.bindings.conversationEndedListener-in-0.group=gamification-service

spring.cloud.stream.bindings.xpEarnedProducer-out-0.destination=gamification.events
spring.cloud.stream.bindings.badgeUnlockedProducer-out-0.destination=gamification.events

spring.cloud.function.definition=lessonCompletedListener;exerciseCompletedListener;conversationEndedListener
```

---

## Événements Consommés

### lesson.completed

**Topic:** `lesson.events`  
**Consumer Group:** `gamification-service`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "lesson.completed",
  "version": "1.0",
  "timestamp": "2026-01-03T10:30:00Z",
  "payload": {
    "lessonId": "lesson-uuid",
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "score": 95,
    "isPerfect": false,
    "xpEarned": 15,
    "completedAt": "2026-01-03T10:30:00Z"
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "lesson-service"
  }
}
```

**Action:**
1. Ajouter XP au UserStats
2. Incrémenter lessonsCompleted
3. Si isPerfect → incrémenter perfectLessons
4. Mettre à jour lastActivityDate
5. Vérifier déblocage de badges
6. Mettre à jour progression des défis

---

### exercise.completed

**Topic:** `lesson.events`  
**Consumer Group:** `gamification-service`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "exercise.completed",
  "version": "1.0",
  "timestamp": "2026-01-03T10:25:00Z",
  "payload": {
    "exerciseId": "exercise-uuid",
    "lessonId": "lesson-uuid",
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "score": 100,
    "xpEarned": 15
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "lesson-service"
  }
}
```

**Action:**
1. Ajouter XP au UserStats
2. Incrémenter exercisesCompleted
3. Mettre à jour lastActivityDate

---

### conversation.ended

**Topic:** `conversation.events`  
**Consumer Group:** `gamification-service`  
**Clé de partitionnement:** `sessionId`

```json
{
  "eventType": "conversation.ended",
  "version": "1.0",
  "timestamp": "2026-01-03T11:00:00Z",
  "payload": {
    "sessionId": "session-uuid",
    "participants": [
      {
        "userId": "user-uuid-1",
        "durationMinutes": 15,
        "targetLanguageCode": "en"
      },
      {
        "userId": "user-uuid-2",
        "durationMinutes": 15,
        "targetLanguageCode": "fr"
      }
    ],
    "totalDurationMinutes": 15,
    "endedAt": "2026-01-03T11:00:00Z"
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "conversation-service"
  }
}
```

**Action (pour chaque participant):**
1. Calculer XP = (durationMinutes / 5) * 20 (arrondi inférieur)
2. Ajouter XP au UserStats
3. Ajouter minutes à conversationMinutes
4. Mettre à jour lastActivityDate
5. Vérifier badges conversation

---

## Événements Publiés

### xp.earned

**Topic:** `gamification.events`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "xp.earned",
  "version": "1.0",
  "timestamp": "2026-01-03T10:30:00Z",
  "payload": {
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "amount": 15,
    "source": "lesson",
    "sourceId": "lesson-uuid",
    "totalXp": 1265,
    "weeklyXp": 335,
    "newLevel": null
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "gamification-service"
  }
}
```

---

### badge.unlocked

**Topic:** `gamification.events`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "badge.unlocked",
  "version": "1.0",
  "timestamp": "2026-01-03T10:30:00Z",
  "payload": {
    "userId": "user-uuid",
    "badgeId": "badge-uuid",
    "badgeCode": "lesson_master_50",
    "badgeName": "Étudiant",
    "badgeRarity": "uncommon",
    "xpReward": 50,
    "targetLanguageCode": "en"
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "gamification-service"
  }
}
```

---

### streak.updated

**Topic:** `gamification.events`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "streak.updated",
  "version": "1.0",
  "timestamp": "2026-01-03T00:05:00Z",
  "payload": {
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "currentStreak": 13,
    "longestStreak": 15,
    "streakLost": false,
    "freezeUsed": false
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "gamification-service"
  }
}
```

---

### level.up

**Topic:** `gamification.events`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "level.up",
  "version": "1.0",
  "timestamp": "2026-01-03T10:30:00Z",
  "payload": {
    "userId": "user-uuid",
    "targetLanguageCode": "en",
    "previousLevel": 7,
    "newLevel": 8,
    "totalXp": 1265
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "gamification-service"
  }
}
```

---

### challenge.completed

**Topic:** `gamification.events`  
**Clé de partitionnement:** `userId`

```json
{
  "eventType": "challenge.completed",
  "version": "1.0",
  "timestamp": "2026-01-03T10:30:00Z",
  "payload": {
    "userId": "user-uuid",
    "challengeId": "challenge-uuid",
    "challengeTitle": "Défi de la semaine",
    "xpReward": 100,
    "completedAt": "2026-01-03T10:30:00Z"
  },
  "metadata": {
    "correlationId": "corr-uuid",
    "source": "gamification-service"
  }
}
```

---

## Listeners Spring Cloud Stream

```java
@Configuration
public class GamificationListeners {

    @Bean
    public Consumer<CloudEvent<LessonCompletedPayload>> lessonCompletedListener(
            GamificationService gamificationService) {
        return event -> gamificationService.handleLessonCompleted(event.getData());
    }

    @Bean
    public Consumer<CloudEvent<ExerciseCompletedPayload>> exerciseCompletedListener(
            GamificationService gamificationService) {
        return event -> gamificationService.handleExerciseCompleted(event.getData());
    }

    @Bean
    public Consumer<CloudEvent<ConversationEndedPayload>> conversationEndedListener(
            GamificationService gamificationService) {
        return event -> gamificationService.handleConversationEnded(event.getData());
    }
}
```
