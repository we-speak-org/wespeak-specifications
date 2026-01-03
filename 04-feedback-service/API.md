# Feedback Service - API Endpoints

## Base URL
```
/api/v1/feedback
```

## Authentification
Tous les endpoints requièrent un JWT valide dans le header `Authorization: Bearer <token>`.

---

## Endpoints

### Transcripts

#### Récupérer un transcript
```
GET /transcripts/{transcriptId}
```

**Réponse 200** :
```json
{
  "id": "uuid",
  "sessionId": "session-uuid",
  "participantId": "user-uuid",
  "targetLanguageCode": "en",
  "content": "Hello, how are you today? I am learning English...",
  "segments": [
    {
      "startTime": 0.0,
      "endTime": 2.5,
      "text": "Hello, how are you today?",
      "confidence": 0.95
    },
    {
      "startTime": 2.8,
      "endTime": 5.2,
      "text": "I am learning English",
      "confidence": 0.92
    }
  ],
  "duration": 120,
  "wordCount": 45,
  "confidence": 0.93,
  "status": "COMPLETED",
  "createdAt": "2025-01-15T10:30:00Z",
  "completedAt": "2025-01-15T10:31:00Z"
}
```

#### Lister les transcripts d'une session
```
GET /transcripts?sessionId={sessionId}
```

**Réponse 200** :
```json
{
  "items": [...],
  "total": 2
}
```

---

### Feedbacks

#### Récupérer un feedback
```
GET /feedbacks/{feedbackId}
```

**Réponse 200** :
```json
{
  "id": "uuid",
  "transcriptId": "transcript-uuid",
  "userId": "user-uuid",
  "sessionId": "session-uuid",
  "targetLanguageCode": "en",
  "overallScore": 72,
  "grammarScore": 68,
  "vocabularyScore": 75,
  "fluencyScore": 78,
  "pronunciationScore": 70,
  "errors": [
    {
      "type": "GRAMMAR",
      "original": "I go yesterday",
      "correction": "I went yesterday",
      "explanation": "Utilisez le passé simple 'went' pour une action passée.",
      "severity": "MEDIUM",
      "segmentIndex": 3
    },
    {
      "type": "VOCABULARY",
      "original": "very very good",
      "correction": "excellent / outstanding",
      "explanation": "Évitez les répétitions. Utilisez des synonymes plus riches.",
      "severity": "LOW",
      "segmentIndex": 7
    }
  ],
  "strengths": [
    "Bonne utilisation des expressions idiomatiques",
    "Vocabulaire varié sur le thème du voyage",
    "Fluidité naturelle sans hésitations majeures"
  ],
  "improvements": [
    "Revoir les temps du passé (simple vs perfect)",
    "Pratiquer la prononciation du 'th'",
    "Utiliser des connecteurs logiques (however, therefore)"
  ],
  "summary": "Belle performance ! Votre anglais est fluide et compréhensible. Quelques erreurs de grammaire sur les temps du passé à corriger. Continuez à enrichir votre vocabulaire.",
  "xpAwarded": 25,
  "status": "COMPLETED",
  "createdAt": "2025-01-15T10:31:00Z",
  "completedAt": "2025-01-15T10:31:30Z"
}
```

#### Lister mes feedbacks
```
GET /feedbacks/me?targetLanguageCode={lang}&page={page}&size={size}
```

**Query params** :
- `targetLanguageCode` (optionnel) : Filtrer par langue
- `page` (défaut: 0) : Numéro de page
- `size` (défaut: 10) : Taille de page

**Réponse 200** :
```json
{
  "items": [
    {
      "id": "uuid",
      "sessionId": "session-uuid",
      "targetLanguageCode": "en",
      "overallScore": 72,
      "xpAwarded": 25,
      "createdAt": "2025-01-15T10:31:00Z"
    }
  ],
  "page": 0,
  "size": 10,
  "total": 15,
  "totalPages": 2
}
```

#### Récupérer le feedback d'une session
```
GET /feedbacks/session/{sessionId}
```

Retourne le feedback de l'utilisateur connecté pour cette session.

---

### Statistiques

#### Mes statistiques par langue
```
GET /stats/me?targetLanguageCode={lang}
```

**Réponse 200** :
```json
{
  "userId": "user-uuid",
  "targetLanguageCode": "en",
  "totalSessions": 12,
  "totalMinutes": 145,
  "averageOverallScore": 68.5,
  "averageGrammarScore": 65.2,
  "averageVocabularyScore": 72.0,
  "averageFluencyScore": 70.8,
  "commonErrors": [
    {
      "type": "GRAMMAR",
      "pattern": "Temps du passé",
      "frequency": 8
    },
    {
      "type": "PRONUNCIATION",
      "pattern": "Son 'th'",
      "frequency": 5
    }
  ],
  "progressTrend": "IMPROVING",
  "lastFeedbackAt": "2025-01-15T10:31:00Z"
}
```

#### Historique de progression
```
GET /stats/me/history?targetLanguageCode={lang}&period={period}
```

**Query params** :
- `targetLanguageCode` : Langue cible
- `period` : WEEK, MONTH, ALL (défaut: MONTH)

**Réponse 200** :
```json
{
  "userId": "user-uuid",
  "targetLanguageCode": "en",
  "period": "MONTH",
  "dataPoints": [
    {
      "date": "2025-01-01",
      "overallScore": 62,
      "sessionsCount": 2
    },
    {
      "date": "2025-01-08",
      "overallScore": 65,
      "sessionsCount": 3
    },
    {
      "date": "2025-01-15",
      "overallScore": 72,
      "sessionsCount": 2
    }
  ]
}
```

---

## Codes d'Erreur

| Code | Message | Description |
|------|---------|-------------|
| 400 | INVALID_REQUEST | Requête malformée |
| 401 | UNAUTHORIZED | Token manquant ou invalide |
| 403 | FORBIDDEN | Accès non autorisé à cette ressource |
| 404 | TRANSCRIPT_NOT_FOUND | Transcript inexistant |
| 404 | FEEDBACK_NOT_FOUND | Feedback inexistant |
| 404 | SESSION_NOT_FOUND | Session inexistante |
| 429 | RATE_LIMIT_EXCEEDED | Limite de feedbacks atteinte (free tier) |
| 500 | INTERNAL_ERROR | Erreur serveur |
| 503 | AI_SERVICE_UNAVAILABLE | Service IA temporairement indisponible |

---

## Exemples cURL

### Récupérer mes feedbacks en anglais
```bash
curl -X GET "https://api.wespeak.com/api/v1/feedback/feedbacks/me?targetLanguageCode=en" \
  -H "Authorization: Bearer eyJhbGc..."
```

### Récupérer mes statistiques
```bash
curl -X GET "https://api.wespeak.com/api/v1/feedback/stats/me?targetLanguageCode=en" \
  -H "Authorization: Bearer eyJhbGc..."
```

### Récupérer le détail d'un feedback
```bash
curl -X GET "https://api.wespeak.com/api/v1/feedback/feedbacks/abc123-uuid" \
  -H "Authorization: Bearer eyJhbGc..."
```
