# Conversation Service - API REST

Base URL: `/api/v1/conversations`

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
      "suggestedQuestions": [
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
curl -X GET "https://api.wespeak.com/api/v1/conversations/topics?language=en&level=B1" \
  -H "Authorization: Bearer {token}"
```

---

## Matchmaking (Quick Match)

### Rejoindre la file d'attente

```
POST /matchmaking/join
```

**Body:**
```json
{
  "targetLanguageCode": "en",
  "level": "B1",
  "preferredTopicCategory": "travel"
}
```

**Réponse 200:**
```json
{
  "queueEntryId": "queue-456",
  "status": "waiting",
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
curl -X POST "https://api.wespeak.com/api/v1/conversations/matchmaking/join" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{"targetLanguageCode":"en","level":"B1"}'
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
  "status": "waiting",
  "waitingSeconds": 45
}
```

**Réponse 200 (matché):**
```json
{
  "status": "matched",
  "sessionId": "session789"
}
```

---

## Sessions

### Créer une session privée

```
POST /sessions
```

**Body:**
```json
{
  "targetLanguageCode": "en",
  "type": "private",
  "topicId": "topic123",
  "maxParticipants": 4
}
```

**Réponse 201:**
```json
{
  "id": "session789",
  "inviteCode": "ABC123",
  "status": "waiting",
  "type": "private",
  "targetLanguageCode": "en",
  "maxParticipants": 4
}
```

---

### Rejoindre une session par code

```
POST /sessions/join
```

**Body:**
```json
{
  "inviteCode": "ABC123"
}
```

**Réponse 200:**
```json
{
  "sessionId": "session789",
  "status": "waiting",
  "participantCount": 2,
  "maxParticipants": 4
}
```

---

### Obtenir la session active

```
GET /sessions/current
```

**Réponse 200:**
```json
{
  "id": "session789",
  "status": "active",
  "type": "public",
  "targetLanguageCode": "en",
  "topic": {
    "id": "topic123",
    "title": "Travel experiences"
  },
  "participants": [
    {
      "userId": "user123",
      "displayName": "Marie",
      "role": "host",
      "status": "connected"
    },
    {
      "userId": "user456",
      "displayName": "Jean",
      "role": "participant",
      "status": "connected"
    }
  ],
  "startedAt": "2026-01-03T10:00:00Z",
  "maxDurationSeconds": 1800
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

### Démarrer une session (host only)

```
POST /sessions/{sessionId}/start
```

**Réponse 200:**
```json
{
  "id": "session789",
  "status": "active",
  "startedAt": "2026-01-03T10:00:00Z"
}
```

---

### Quitter une session

```
POST /sessions/{sessionId}/leave
```

**Réponse 200:**
```json
{
  "message": "Left session successfully"
}
```

---

### Terminer une session (host only)

```
POST /sessions/{sessionId}/end
```

**Réponse 200:**
```json
{
  "id": "session789",
  "status": "ended",
  "endedAt": "2026-01-03T10:25:00Z",
  "totalDurationSeconds": 1500
}
```

---

### Liste des participants

```
GET /sessions/{sessionId}/participants
```

**Réponse 200:**
```json
{
  "participants": [
    {
      "id": "part-1",
      "userId": "user123",
      "displayName": "Marie",
      "role": "host",
      "status": "connected",
      "joinedAt": "2026-01-03T10:00:00Z"
    },
    {
      "id": "part-2",
      "userId": "user456",
      "displayName": "Jean",
      "role": "participant",
      "status": "connected",
      "joinedAt": "2026-01-03T10:00:30Z"
    }
  ]
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
      "targetLanguageCode": "en",
      "participantCount": 3,
      "startedAt": "2026-01-03T10:00:00Z",
      "durationSeconds": 1500,
      "status": "ended"
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
  "type": "public",
  "targetLanguageCode": "en",
  "status": "ended",
  "topic": {
    "id": "topic123",
    "title": "Travel experiences"
  },
  "participants": [
    {
      "userId": "user123",
      "displayName": "Marie",
      "speakingTimeSeconds": 450
    },
    {
      "userId": "user456",
      "displayName": "Jean",
      "speakingTimeSeconds": 380
    }
  ],
  "startedAt": "2026-01-03T10:00:00Z",
  "endedAt": "2026-01-03T10:25:00Z",
  "durationSeconds": 1500
}
```

---

## Signaling WebRTC

Connexion WebSocket pour la signalisation temps réel entre participants.

### Connexion WebSocket

```
WS /ws/signaling?token={jwt}&sessionId={sessionId}
```

### Messages WebSocket

**Offre SDP (client → serveur):**
```json
{
  "type": "offer",
  "toUserId": "user456",
  "sdp": "v=0\r\no=- ..."
}
```

**Réponse SDP (client → serveur):**
```json
{
  "type": "answer",
  "toUserId": "user123",
  "sdp": "v=0\r\no=- ..."
}
```

**Candidat ICE (client → serveur):**
```json
{
  "type": "ice-candidate",
  "toUserId": "user456",
  "candidate": {
    "candidate": "candidate:...",
    "sdpMid": "0",
    "sdpMLineIndex": 0
  }
}
```

**Participant joined (serveur → client):**
```json
{
  "type": "participant-joined",
  "participant": {
    "userId": "user789",
    "displayName": "Pierre"
  }
}
```

**Participant left (serveur → client):**
```json
{
  "type": "participant-left",
  "userId": "user789"
}
```

**Session started (serveur → client):**
```json
{
  "type": "session-started",
  "startedAt": "2026-01-03T10:00:00Z"
}
```

**Session ended (serveur → client):**
```json
{
  "type": "session-ended",
  "reason": "host_ended"
}
```

---

## Codes d'Erreur

| Code | Erreur | Description |
|------|--------|-------------|
| 400 | INVALID_LANGUAGE | Langue non supportée |
| 400 | INVALID_LEVEL | Niveau CECR invalide |
| 403 | SESSION_FULL | Session complète (max participants) |
| 403 | DAILY_LIMIT_REACHED | Limite quotidienne atteinte (free tier) |
| 403 | NOT_HOST | Action réservée au host |
| 404 | TOPIC_NOT_FOUND | Topic inexistant |
| 404 | SESSION_NOT_FOUND | Session inexistante |
| 404 | INVALID_INVITE_CODE | Code d'invitation invalide |
| 409 | ALREADY_IN_QUEUE | Déjà en file d'attente |
| 409 | ALREADY_IN_SESSION | Déjà en session active |
| 410 | SESSION_ENDED | Session terminée |
