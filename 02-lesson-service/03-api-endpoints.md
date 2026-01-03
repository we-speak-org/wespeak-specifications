# Lesson Service - API Endpoints

## Base URL

```
https://api.wespeak.com/api/v1
```

---

## Authentification

- **Endpoints publics (GET)** : Pas d'authentification requise
- **Endpoints protégés (POST)** : Header `Authorization: Bearer <JWT>`

---

## 1. Courses (Cours)

### Lister les cours par langue

```http
GET /courses?language={code}&level={level}
```

**Paramètres :**

| Param | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `language` | String | Oui | Code langue (ex: "en", "fr") |
| `level` | String | Non | Filtrer par niveau CECRL |

**Exemple :**

```bash
curl -X GET "https://api.wespeak.com/api/v1/courses?language=en"
```

**Réponse 200 :**

```json
{
  "courses": [
    {
      "id": "507f1f77bcf86cd799439011",
      "title": "Anglais pour Débutants",
      "level": "A1",
      "description": "Apprenez les bases de l'anglais",
      "imageUrl": "https://cdn.wespeak.com/courses/en-a1.png",
      "order": 1,
      "requiredXP": 0,
      "estimatedHours": 20,
      "totalUnits": 5,
      "totalLessons": 25
    },
    {
      "id": "507f1f77bcf86cd799439012",
      "title": "Anglais Élémentaire",
      "level": "A2",
      "order": 2,
      "requiredXP": 500,
      "totalUnits": 6,
      "totalLessons": 30
    }
  ]
}
```

---

### Obtenir un cours avec ses unités

```http
GET /courses/{courseId}
```

**Exemple :**

```bash
curl -X GET "https://api.wespeak.com/api/v1/courses/507f1f77bcf86cd799439011"
```

**Réponse 200 :**

```json
{
  "id": "507f1f77bcf86cd799439011",
  "title": "Anglais pour Débutants",
  "level": "A1",
  "description": "Apprenez les bases de l'anglais pour communiquer au quotidien",
  "imageUrl": "https://cdn.wespeak.com/courses/en-a1.png",
  "estimatedHours": 20,
  "units": [
    {
      "id": "507f1f77bcf86cd799439012",
      "title": "Les Salutations",
      "order": 1,
      "totalLessons": 5,
      "isUnlocked": true
    },
    {
      "id": "507f1f77bcf86cd799439013",
      "title": "Au Restaurant",
      "order": 2,
      "totalLessons": 5,
      "isUnlocked": false
    }
  ]
}
```

---

## 2. Units (Unités)

### Obtenir une unité avec ses leçons

```http
GET /units/{unitId}
```

**Exemple :**

```bash
curl -X GET "https://api.wespeak.com/api/v1/units/507f1f77bcf86cd799439012"
```

**Réponse 200 :**

```json
{
  "id": "507f1f77bcf86cd799439012",
  "title": "Les Salutations",
  "description": "Apprenez à saluer et vous présenter",
  "courseId": "507f1f77bcf86cd799439011",
  "lessons": [
    {
      "id": "507f1f77bcf86cd799439101",
      "title": "Dire bonjour",
      "type": "vocabulary",
      "order": 1,
      "estimatedMinutes": 10,
      "xpReward": 15,
      "isUnlocked": true,
      "isCompleted": true,
      "bestScore": 85
    },
    {
      "id": "507f1f77bcf86cd799439102",
      "title": "Se présenter",
      "type": "vocabulary",
      "order": 2,
      "estimatedMinutes": 12,
      "xpReward": 15,
      "isUnlocked": true,
      "isCompleted": false,
      "bestScore": null
    },
    {
      "id": "507f1f77bcf86cd799439103",
      "title": "Demander comment ça va",
      "type": "grammar",
      "order": 3,
      "estimatedMinutes": 15,
      "xpReward": 20,
      "isUnlocked": false,
      "isCompleted": false,
      "bestScore": null
    }
  ]
}
```

---

## 3. Lessons (Leçons)

### Obtenir une leçon avec ses exercices

```http
GET /lessons/{lessonId}
```

**Exemple :**

```bash
curl -X GET "https://api.wespeak.com/api/v1/lessons/507f1f77bcf86cd799439101"
```

**Réponse 200 :**

```json
{
  "id": "507f1f77bcf86cd799439101",
  "title": "Dire bonjour",
  "description": "Apprenez les salutations de base en anglais",
  "type": "vocabulary",
  "estimatedMinutes": 10,
  "xpReward": 15,
  "unit": {
    "id": "507f1f77bcf86cd799439012",
    "title": "Les Salutations"
  },
  "exercises": [
    {
      "id": "ex-001",
      "type": "mcq",
      "order": 1,
      "question": "Comment dit-on 'Bonjour' en anglais ?",
      "content": {
        "options": [
          {"id": "a", "text": "Hello"},
          {"id": "b", "text": "Goodbye"},
          {"id": "c", "text": "Thank you"},
          {"id": "d", "text": "Sorry"}
        ]
      },
      "points": 10
    },
    {
      "id": "ex-002",
      "type": "fill_gap",
      "order": 2,
      "question": "Complétez : Nice to ___ you!",
      "hint": "Un verbe courant pour les rencontres",
      "content": {
        "sentence": "Nice to ___ you!"
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

### Démarrer une leçon

```http
POST /lessons/{lessonId}/start
```

**Headers :**
```
Authorization: Bearer <JWT>
```

**Exemple :**

```bash
curl -X POST "https://api.wespeak.com/api/v1/lessons/507f1f77bcf86cd799439101/start" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Réponse 200 :**

```json
{
  "sessionId": "session-abc123",
  "lessonId": "507f1f77bcf86cd799439101",
  "startedAt": "2025-01-15T14:00:00Z",
  "exerciseCount": 5
}
```

**Erreurs :**

| Code | Message | Raison |
|------|---------|--------|
| 403 | `LESSON_LOCKED` | Leçon non débloquée |
| 404 | `LESSON_NOT_FOUND` | Leçon inexistante |

---

### Terminer une leçon

```http
POST /lessons/{lessonId}/complete
```

**Headers :**
```
Authorization: Bearer <JWT>
Content-Type: application/json
```

**Body :**

```json
{
  "score": 85,
  "correctAnswers": 17,
  "totalExercises": 20,
  "timeSpentSeconds": 420
}
```

**Exemple :**

```bash
curl -X POST "https://api.wespeak.com/api/v1/lessons/507f1f77bcf86cd799439101/complete" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{"score": 85, "correctAnswers": 17, "totalExercises": 20, "timeSpentSeconds": 420}'
```

**Réponse 200 :**

```json
{
  "completion": {
    "id": "507f1f77bcf86cd799439030",
    "lessonId": "507f1f77bcf86cd799439101",
    "score": 85,
    "xpEarned": 15,
    "attemptNumber": 1,
    "completedAt": "2025-01-15T14:30:00Z"
  },
  "unlocked": {
    "nextLesson": {
      "id": "507f1f77bcf86cd799439102",
      "title": "Se présenter"
    }
  },
  "progress": {
    "lessonsCompleted": 6,
    "averageScore": 82
  }
}
```

---

## 4. Exercises (Exercices)

### Soumettre une réponse

```http
POST /exercises/{exerciseId}/submit
```

**Headers :**
```
Authorization: Bearer <JWT>
Content-Type: application/json
```

**Body (QCM) :**

```json
{
  "answer": {
    "optionId": "a"
  },
  "timeSpentSeconds": 12
}
```

**Body (Texte à trous) :**

```json
{
  "answer": {
    "text": "meet"
  },
  "timeSpentSeconds": 15
}
```

**Exemple :**

```bash
curl -X POST "https://api.wespeak.com/api/v1/exercises/ex-001/submit" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{"answer": {"optionId": "a"}, "timeSpentSeconds": 12}'
```

**Réponse 200 (Correct) :**

```json
{
  "isCorrect": true,
  "pointsEarned": 10,
  "correctAnswer": {
    "optionId": "a",
    "text": "Hello"
  },
  "feedback": "Excellent ! 'Hello' est la salutation la plus courante en anglais.",
  "attemptNumber": 1
}
```

**Réponse 200 (Incorrect) :**

```json
{
  "isCorrect": false,
  "pointsEarned": 0,
  "correctAnswer": {
    "optionId": "a",
    "text": "Hello"
  },
  "feedback": "Pas tout à fait. 'Hello' signifie 'Bonjour'. 'Goodbye' signifie 'Au revoir'.",
  "attemptNumber": 1
}
```

---

## 5. Progress (Progression)

### Obtenir ma progression

```http
GET /progress?language={code}
```

**Headers :**
```
Authorization: Bearer <JWT>
```

**Paramètres :**

| Param | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `language` | String | Oui | Code langue |

**Exemple :**

```bash
curl -X GET "https://api.wespeak.com/api/v1/progress?language=en" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Réponse 200 :**

```json
{
  "userId": "user-123-uuid",
  "targetLanguageCode": "en",
  "currentCourse": {
    "id": "507f1f77bcf86cd799439011",
    "title": "Anglais pour Débutants",
    "level": "A1"
  },
  "currentUnit": {
    "id": "507f1f77bcf86cd799439012",
    "title": "Les Salutations"
  },
  "currentLesson": {
    "id": "507f1f77bcf86cd799439102",
    "title": "Se présenter"
  },
  "stats": {
    "lessonsCompleted": 5,
    "averageScore": 82,
    "totalTimeMinutes": 45
  },
  "lastActivityAt": "2025-01-15T14:30:00Z"
}
```

---

### Obtenir l'historique des leçons

```http
GET /progress/history?language={code}&limit={n}
```

**Headers :**
```
Authorization: Bearer <JWT>
```

**Exemple :**

```bash
curl -X GET "https://api.wespeak.com/api/v1/progress/history?language=en&limit=10" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Réponse 200 :**

```json
{
  "completions": [
    {
      "lessonId": "507f1f77bcf86cd799439101",
      "lessonTitle": "Dire bonjour",
      "score": 85,
      "xpEarned": 15,
      "completedAt": "2025-01-15T14:30:00Z"
    },
    {
      "lessonId": "507f1f77bcf86cd799439100",
      "lessonTitle": "Introduction au cours",
      "score": 90,
      "xpEarned": 12,
      "completedAt": "2025-01-14T10:15:00Z"
    }
  ],
  "total": 5,
  "hasMore": false
}
```

---

## Codes d'Erreur

| Code HTTP | Code Erreur | Description |
|-----------|-------------|-------------|
| 400 | `INVALID_ANSWER` | Format de réponse invalide |
| 400 | `INVALID_SCORE` | Score hors limites (0-100) |
| 401 | `UNAUTHORIZED` | JWT manquant ou invalide |
| 403 | `LESSON_LOCKED` | Leçon non débloquée |
| 403 | `MAX_ATTEMPTS_REACHED` | Nombre max de tentatives atteint |
| 404 | `COURSE_NOT_FOUND` | Cours inexistant |
| 404 | `UNIT_NOT_FOUND` | Unité inexistante |
| 404 | `LESSON_NOT_FOUND` | Leçon inexistante |
| 404 | `EXERCISE_NOT_FOUND` | Exercice inexistant |
| 429 | `RATE_LIMIT_EXCEEDED` | Trop de requêtes |

**Format d'erreur :**

```json
{
  "error": {
    "code": "LESSON_LOCKED",
    "message": "Cette leçon n'est pas encore débloquée. Terminez d'abord la leçon précédente.",
    "details": {
      "requiredLessonId": "507f1f77bcf86cd799439100",
      "requiredScore": 70
    }
  }
}
```
