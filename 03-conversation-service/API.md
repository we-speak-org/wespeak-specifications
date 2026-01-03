# Conversation Service - API REST

Base URL: `/api/v1/conversation`

---

## Topics

### Lister les topics

```
GET /topics?language={code}&level={level}&category={category}
```

**Query Parameters:**
- `language` (requis): Code langue (en, fr, es...)
- `level` (optionnel): Filtre par niveau CECRL
- `category` (optionnel): Filtre par catégorie

**Réponse 200:**
```json
{
  "topics": [
    {
      "id": "topic123",
      "title": "Travel experiences",
      "description": "Share your best travel memories",
      "category": "travel",
      "level": "B1",
      "estimatedDurationMinutes": 10,
      "promptQuestions": [
        "What's the best trip you've ever taken?",
        "Do you prefer beach or mountain vacations?"
      ]
    }
  ],
  "total": 25
}
```

**Exemple curl:**
```bash
curl -X GET "https://api.wespeak.com/api/v1/conversation/topics?language=en&level=B1" \
  -H "Authorization: Bearer {token}"
```

---

### Obtenir un topic

```
GET /topics/{topicId}
```

**Réponse 200:**
```json
{
  "id": "topic123",
  "title": "Travel experiences",
  "description": "Share your best travel memories and dream destinations",
  "category": "travel",
  "level": "B1",
  "targetLanguageCode": "en",
  "estimatedDurationMinutes": 10,
  "promptQuestions": [
    "What's the best trip you've ever taken?",
    "Do you prefer beach or mountain vacations?",
    "What's on your travel bucket list?"
  ]
}
```

---

## Matchmaking

### Rejoindre la file d'attente

```
POST /matchmaking/join
```

**Body:**
```json
{
  "learningProfileId": "profile123",
  "preferredTopicId": "topic123",
  "preferredDuration": 10,
  "recordingConsent": true
}
```

**Réponse 200:**
```json
{
  "requestId": "match-req-456",
  "status": "pending",
  "estimatedWaitSeconds": 30,
  "expiresAt": "2026-01-03T10:02:00Z"
}
```

**Réponse 409 (déjà en file):**
```json
{
  "error": "ALREADY_IN_QUEUE",
  "message": "You are already waiting for a match"
}
```

**Exemple curl:**
```bash
curl -X POST "https://api.wespeak.com/api/v1/conversation/matchmaking/join" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{"learningProfileId":"profile123","preferredDuration":10,"recordingConsent":true}'
```

---

### Quitter la file d'attente

```
DELETE /matchmaking/leave
```

**Réponse 200:**
```json
{
  "message": "Left matchmaking queue"
}
```

---

### Statut du matchmaking

```
GET /matchmaking/status
```

**Réponse 200 (en attente):**
```json
{
  "status": "pending",
  "waitingSeconds": 45,
  "position": 3
}
```

**Réponse 200 (matché):**
```json
{
  "status": "matched",
  "sessionId": "session789",
  "partnerId": "user456",
  "partnerDisplayName": "Jean",
  "partnerLevel": "B1",
  "topicId": "topic123"
}
```

---

## Sessions

### Obtenir la session active

```
GET /sessions/active
```

**Réponse 200:**
```json
{
  "id": "session789",
  "status": "active",
  "partnerId": "user456",
  "partnerDisplayName": "Jean",
  "topicId": "topic123",
  "topicTitle": "Travel experiences",
  "startedAt": "2026-01-03T10:00:00Z",
  "durationLimitSeconds": 600
}
```

**Réponse 404:**
```json
{
  "error": "NO_ACTIVE_SESSION",
  "message": "No active conversation session"
}
```

---

### Terminer une session

```
POST /sessions/{sessionId}/end
```

**Body:**
```json
{
  "reason": "completed"
}
```

**Réponse 200:**
```json
{
  "id": "session789",
  "status": "completed",
  "actualDurationSeconds": 542,
  "xpEarned": 50
}
```

---

### Historique des sessions

```
GET /sessions/history?page={page}&limit={limit}
```

**Réponse 200:**
```json
{
  "sessions": [
    {
      "id": "session789",
      "topicTitle": "Travel experiences",
      "partnerDisplayName": "Jean",
      "targetLanguageCode": "en",
      "startedAt": "2026-01-03T10:00:00Z",
      "actualDurationSeconds": 542,
      "status": "completed",
      "hasFeedback": true
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 45
  }
}
```

---

### Détail d'une session

```
GET /sessions/{sessionId}
```

**Réponse 200:**
```json
{
  "id": "session789",
  "topicId": "topic123",
  "topicTitle": "Travel experiences",
  "targetLanguageCode": "en",
  "partnerId": "user456",
  "partnerDisplayName": "Jean",
  "startedAt": "2026-01-03T10:00:00Z",
  "endedAt": "2026-01-03T10:09:02Z",
  "actualDurationSeconds": 542,
  "status": "completed",
  "feedbackId": "feedback123"
}
```

---

## Signalisation WebRTC

Ces endpoints sont utilisés via WebSocket pour la signalisation temps réel.

### Connexion WebSocket

```
WS /ws/signaling?token={jwt}
```

### Messages WebSocket

**Offre SDP (client → serveur):**
```json
{
  "type": "offer",
  "sessionId": "session789",
  "sdp": "v=0\r\no=- ..."
}
```

**Réponse SDP (client → serveur):**
```json
{
  "type": "answer",
  "sessionId": "session789",
  "sdp": "v=0\r\no=- ..."
}
```

**Candidat ICE (client → serveur):**
```json
{
  "type": "ice-candidate",
  "sessionId": "session789",
  "candidate": {
    "candidate": "candidate:...",
    "sdpMid": "0",
    "sdpMLineIndex": 0
  }
}
```

**Heartbeat (client → serveur):**
```json
{
  "type": "heartbeat",
  "sessionId": "session789"
}
```

**Notification de match (serveur → client):**
```json
{
  "type": "match-found",
  "sessionId": "session789",
  "partnerId": "user456",
  "partnerDisplayName": "Jean",
  "isInitiator": true
}
```

**Partner disconnected (serveur → client):**
```json
{
  "type": "partner-disconnected",
  "sessionId": "session789"
}
```

---

## Codes d'Erreur

| Code | Erreur | Description |
|------|--------|-------------|
| 400 | INVALID_DURATION | Durée non valide (5, 10, 15, 20 min) |
| 403 | DAILY_LIMIT_REACHED | Limite quotidienne atteinte (free) |
| 404 | TOPIC_NOT_FOUND | Topic inexistant |
| 404 | SESSION_NOT_FOUND | Session inexistante |
| 409 | ALREADY_IN_QUEUE | Déjà en file d'attente |
| 409 | ALREADY_IN_SESSION | Déjà en session active |
| 410 | SESSION_EXPIRED | Session expirée |
