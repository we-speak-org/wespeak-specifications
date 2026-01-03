# User Story : API Lessons

## Contexte

Implémenter les endpoints pour consulter et interagir avec les leçons.

## Endpoints à Implémenter

### GET /api/v1/lessons/{lessonId}

Récupérer une leçon avec ses exercices.

**Note :** Les `correctAnswer` ne sont PAS envoyés dans cette réponse (pour éviter la triche).

**Réponse 200 :**
```json
{
  "id": "...",
  "title": "Dire bonjour",
  "description": "...",
  "type": "vocabulary",
  "estimatedMinutes": 10,
  "xpReward": 15,
  "unit": {
    "id": "...",
    "title": "Les Salutations"
  },
  "exercises": [
    {
      "id": "ex-001",
      "type": "mcq",
      "order": 1,
      "question": "Comment dit-on 'Bonjour' ?",
      "content": {
        "options": [
          {"id": "a", "text": "Hello"},
          {"id": "b", "text": "Goodbye"}
        ]
      },
      "points": 10
    }
  ],
  "isUnlocked": true,
  "userProgress": {
    "isCompleted": false,
    "bestScore": null,
    "attempts": 0
  }
}
```

---

### POST /api/v1/lessons/{lessonId}/start

Démarre une session de leçon. **Authentification requise.**

**Headers :** `Authorization: Bearer <JWT>`

**Réponse 200 :**
```json
{
  "sessionId": "session-abc123",
  "lessonId": "...",
  "startedAt": "2025-01-15T14:00:00Z",
  "exerciseCount": 5
}
```

**Erreurs :**
- 401 `UNAUTHORIZED` - JWT manquant/invalide
- 403 `LESSON_LOCKED` - Leçon pas encore débloquée
- 404 `LESSON_NOT_FOUND`

**Action :**
- Publier événement Kafka `lesson.started`

---

### POST /api/v1/lessons/{lessonId}/complete

Termine une leçon et enregistre le score. **Authentification requise.**

**Body :**
```json
{
  "score": 85,
  "correctAnswers": 17,
  "totalExercises": 20,
  "timeSpentSeconds": 420
}
```

**Réponse 200 :**
```json
{
  "completion": {
    "id": "...",
    "lessonId": "...",
    "score": 85,
    "xpEarned": 15,
    "attemptNumber": 1,
    "completedAt": "..."
  },
  "unlocked": {
    "nextLesson": {
      "id": "...",
      "title": "Se présenter"
    }
  },
  "progress": {
    "lessonsCompleted": 6,
    "averageScore": 82
  }
}
```

**Erreurs :**
- 400 `INVALID_SCORE` - Score hors limites
- 401 `UNAUTHORIZED`
- 403 `LESSON_LOCKED`

**Actions :**
1. Créer `LessonCompletion`
2. Mettre à jour `UserProgress`
3. Si score ≥ 70% : débloquer la leçon suivante
4. Publier événement Kafka `lesson.completed`
5. Si dernière leçon de l'unité : publier `unit.completed`
6. Si dernière unité du cours : publier `course.completed`

## Calcul des XP

```
xpBase = lesson.xpReward
xpEarned = xpBase * (score / 100)

Bonus :
- Si score >= 90% : xpEarned * 1.2
- Si attemptNumber == 1 et score >= 70% : xpEarned * 1.1
```

## Services à Créer

### LessonService

```java
public interface LessonService {
    LessonDetailDto findLessonById(String lessonId, String userId);
    LessonSessionDto startLesson(String lessonId, String userId);
    LessonCompletionResultDto completeLesson(String lessonId, String userId, CompleteLessonRequest request);
}
```

### UnlockService

```java
public interface UnlockService {
    boolean isLessonUnlocked(String lessonId, String userId);
    void unlockNextLesson(String lessonId, String userId);
}
```

## Règles de Déblocage

1. **Première leçon** de la première unité : toujours débloquée
2. **Leçons suivantes** : débloquées si la leçon précédente a score ≥ 70%
3. **Première leçon d'une nouvelle unité** : débloquée si toutes les leçons de l'unité précédente sont complétées

## Critères d'Acceptation

- [ ] GET /lessons/{id} retourne la leçon avec exercices (sans réponses)
- [ ] POST /start vérifie le déblocage et publie l'événement
- [ ] POST /complete calcule les XP avec bonus
- [ ] POST /complete débloque correctement la leçon suivante
- [ ] Les événements Kafka sont publiés
- [ ] Tests d'intégration complets
