# Recommendation Service - API Endpoints

## Base URL
```
/api/v1/recommendations
```

## Endpoints

### Recommandations

#### GET /
Récupère les recommandations personnalisées de l'utilisateur.

```bash
curl -X GET "http://localhost:8086/api/v1/recommendations?language=en&limit=5" \
  -H "Authorization: Bearer {token}"
```

**Query Parameters:**
- `language` (obligatoire): Code langue cible
- `limit` (optionnel): Nombre max de recommandations (défaut: 5, max: 10)
- `type` (optionnel): Filtrer par type (next_lesson, revision, conversation, practice)

**Response 200:**
```json
{
  "recommendations": [
    {
      "id": "rec-uuid-1",
      "type": "next_lesson",
      "targetId": "lesson-uuid",
      "targetType": "lesson",
      "title": "Leçon 5: Les transports",
      "reason": "Continuez votre progression",
      "priority": 1,
      "metadata": {
        "unitName": "Se déplacer",
        "estimatedMinutes": 10
      }
    },
    {
      "id": "rec-uuid-2",
      "type": "revision",
      "targetId": "exercise-uuid",
      "targetType": "exercise",
      "title": "Révision: Conjugaison passé",
      "reason": "Renforcez vos bases en conjugaison",
      "priority": 2,
      "metadata": {
        "errorCount": 5,
        "lastErrorAt": "2026-01-02T14:00:00Z"
      }
    },
    {
      "id": "rec-uuid-3",
      "type": "conversation",
      "targetId": "slot-uuid",
      "targetType": "slot",
      "title": "Session conversation - 18h00",
      "reason": "Pratiquez avec d'autres apprenants",
      "priority": 3,
      "metadata": {
        "startTime": "2026-01-03T18:00:00Z",
        "participantsCount": 3,
        "maxParticipants": 6
      }
    }
  ],
  "generatedAt": "2026-01-03T10:00:00Z"
}
```

#### POST /{recommendationId}/click
Enregistre un clic sur une recommandation.

```bash
curl -X POST "http://localhost:8086/api/v1/recommendations/rec-uuid/click" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "success": true,
  "recommendation": {
    "id": "rec-uuid",
    "clickedAt": "2026-01-03T10:05:00Z"
  }
}
```

#### POST /{recommendationId}/dismiss
Ignore une recommandation.

```bash
curl -X POST "http://localhost:8086/api/v1/recommendations/rec-uuid/dismiss" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "success": true,
  "message": "Recommandation ignorée"
}
```

---

### Préférences

#### GET /preferences
Récupère les préférences de l'utilisateur.

```bash
curl -X GET "http://localhost:8086/api/v1/recommendations/preferences?language=en" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "id": "pref-uuid",
  "userId": "user-uuid",
  "targetLanguageCode": "en",
  "preferredLearningTime": ["morning", "evening"],
  "dailyGoalMinutes": 20,
  "focusAreas": ["grammar", "vocabulary"],
  "excludedTopics": []
}
```

#### PUT /preferences
Met à jour les préférences.

```bash
curl -X PUT "http://localhost:8086/api/v1/recommendations/preferences?language=en" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "preferredLearningTime": ["evening"],
    "dailyGoalMinutes": 30,
    "focusAreas": ["pronunciation", "listening"]
  }'
```

**Response 200:**
```json
{
  "id": "pref-uuid",
  "preferredLearningTime": ["evening"],
  "dailyGoalMinutes": 30,
  "focusAreas": ["pronunciation", "listening"],
  "updatedAt": "2026-01-03T10:00:00Z"
}
```

---

### Historique d'Apprentissage

#### GET /learning-history
Récupère l'historique d'apprentissage analysé.

```bash
curl -X GET "http://localhost:8086/api/v1/recommendations/learning-history?language=en" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "userId": "user-uuid",
  "targetLanguageCode": "en",
  "summary": {
    "totalLessons": 45,
    "averageScore": 82,
    "totalConversationMinutes": 120
  },
  "weakAreas": [
    {
      "category": "grammar",
      "subcategory": "past_tense",
      "errorCount": 12,
      "recommendation": "Pratiquer les verbes irréguliers au passé"
    }
  ],
  "strongAreas": [
    {
      "category": "vocabulary",
      "subcategory": "daily_life",
      "successRate": 95
    }
  ],
  "recentProgress": {
    "lessonsLastWeek": 8,
    "xpLastWeek": 320,
    "streak": 12
  }
}
```

---

## Codes d'Erreur

| Code HTTP | Code Erreur | Description |
|-----------|-------------|-------------|
| 400 | INVALID_LANGUAGE | Code langue invalide |
| 404 | RECOMMENDATION_NOT_FOUND | Recommandation non trouvée |
| 404 | PREFERENCES_NOT_FOUND | Préférences non trouvées |
| 409 | ALREADY_DISMISSED | Recommandation déjà ignorée |
