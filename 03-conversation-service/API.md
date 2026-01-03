# Conversation Service - API REST

Base URL: `/api/v1/conversations`

---

## TimeSlots (Créneaux)

### Lister les créneaux disponibles

```
GET /timeslots?language={code}&level={level}&from={date}&to={date}
```

**Query Parameters:**
- `language` (requis): Code langue (en, fr, es...)
- `level` (optionnel): Filtre par niveau CECR
- `from` (optionnel): Date début (défaut: maintenant)
- `to` (optionnel): Date fin (défaut: +7 jours)

**Réponse 200:**
```json
{
  "timeSlots": [
    {
      "id": "slot123",
      "targetLanguageCode": "en",
      "level": "B1",
      "startTime": "2026-01-03T14:00:00Z",
      "durationMinutes": 30,
      "maxParticipants": 8,
      "registeredCount": 3,
      "spotsAvailable": 5
    }
  ],
  "total": 15
}
```

**Exemple curl:**
```bash
curl -X GET "https://api.wespeak.com/api/v1/conversations/timeslots?language=en&level=B1" \
  -H "Authorization: Bearer {token}"
```

---

### Détail d'un créneau

```
GET /timeslots/{timeSlotId}
```

**Réponse 200:**
```json
{
  "id": "slot123",
  "targetLanguageCode": "en",
  "level": "B1",
  "startTime": "2026-01-03T14:00:00Z",
  "durationMinutes": 30,
  "maxParticipants": 8,
  "minParticipants": 2,
  "registeredCount": 3,
  "isUserRegistered": false
}
```

---

## Registrations (Inscriptions)

### S'inscrire à un créneau

```
POST /timeslots/{timeSlotId}/register
```

**Réponse 201:**
```json
{
  "registrationId": "reg456",
  "timeSlotId": "slot123",
  "status": "registered",
  "registeredAt": "2026-01-03T10:00:00Z"
}
```

**Réponse 409 (déjà inscrit):**
```json
{
  "error": "ALREADY_REGISTERED",
  "message": "You are already registered for this time slot"
}
```

**Exemple curl:**
```bash
curl -X POST "https://api.wespeak.com/api/v1/conversations/timeslots/slot123/register" \
  -H "Authorization: Bearer {token}"
```

---

### Annuler une inscription

```
DELETE /timeslots/{timeSlotId}/register
```

**Réponse 200:**
```json
{
  "message": "Registration cancelled"
}
```

**Réponse 400:**
```json
{
  "error": "CANCELLATION_TOO_LATE",
  "message": "Cannot cancel less than 15 minutes before start"
}
```

---

### Mes inscriptions

```
GET /registrations?status={status}
```

**Query Parameters:**
- `status` (optionnel): registered, cancelled, attended, noshow

**Réponse 200:**
```json
{
  "registrations": [
    {
      "id": "reg456",
      "timeSlot": {
        "id": "slot123",
        "targetLanguageCode": "en",
        "level": "B1",
        "startTime": "2026-01-03T14:00:00Z",
        "durationMinutes": 30
      },
      "status": "registered",
      "registeredAt": "2026-01-03T10:00:00Z"
    }
  ],
  "total": 3
}
```

---

## Sessions

### Rejoindre la session (quand le créneau commence)

```
POST /sessions/join
```

**Body:**
```json
{
  "timeSlotId": "slot123",
  "recordingConsent": true
}
```

**Réponse 200:**
```json
{
  "sessionId": "session789",
  "status": "waiting",
  "recordingEnabled": true,
  "participants": [
    {
      "userId": "user123",
      "displayName": "Marie",
      "status": "connected"
    }
  ]
}
```

**Réponse 403:**
```json
{
  "error": "NOT_REGISTERED",
  "message": "You are not registered for this time slot"
}
```

---

### Session courante

```
GET /sessions/current
```

**Réponse 200:**
```json
{
  "id": "session789",
  "status": "active",
  "targetLanguageCode": "en",
  "level": "B1",
  "recordingEnabled": true,
  "participants": [
    {
      "userId": "user123",
      "displayName": "Marie",
      "cameraEnabled": true,
      "micEnabled": true,
      "status": "connected"
    },
    {
      "userId": "user456",
      "displayName": "Jean",
      "cameraEnabled": true,
      "micEnabled": false,
      "status": "connected"
    }
  ],
  "startedAt": "2026-01-03T14:00:00Z",
  "endsAt": "2026-01-03T14:30:00Z",
  "remainingSeconds": 1200
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

### Mettre à jour mes contrôles média

```
PATCH /sessions/current/media
```

**Body:**
```json
{
  "cameraEnabled": false,
  "micEnabled": true
}
```

**Réponse 200:**
```json
{
  "cameraEnabled": false,
  "micEnabled": true
}
```

---

### Quitter la session

```
POST /sessions/current/leave
```

**Réponse 200:**
```json
{
  "message": "Left session successfully"
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
      "targetLanguageCode": "en",
      "level": "B1",
      "participantCount": 4,
      "startedAt": "2026-01-03T14:00:00Z",
      "durationSeconds": 1800,
      "recordingAvailable": true
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25
  }
}
```

---

## Signaling WebRTC

Connexion WebSocket pour la signalisation temps réel.

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
    "displayName": "Pierre",
    "cameraEnabled": true,
    "micEnabled": true
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

**Media state changed (serveur → client):**
```json
{
  "type": "media-state-changed",
  "userId": "user456",
  "cameraEnabled": false,
  "micEnabled": true
}
```

**Session ended (serveur → client):**
```json
{
  "type": "session-ended",
  "reason": "time_expired"
}
```

---

## Codes d'Erreur

| Code | Erreur | Description |
|------|--------|-------------|
| 400 | INVALID_LANGUAGE | Langue non supportée |
| 400 | INVALID_LEVEL | Niveau CECR invalide |
| 400 | CANCELLATION_TOO_LATE | Annulation trop tardive |
| 403 | NOT_REGISTERED | Non inscrit au créneau |
| 403 | SESSION_FULL | Créneau complet |
| 403 | DAILY_LIMIT_REACHED | Limite quotidienne atteinte |
| 404 | TIMESLOT_NOT_FOUND | Créneau inexistant |
| 404 | SESSION_NOT_FOUND | Session inexistante |
| 404 | NO_ACTIVE_SESSION | Pas de session active |
| 409 | ALREADY_REGISTERED | Déjà inscrit |
| 409 | ALREADY_IN_SESSION | Déjà en session |
| 410 | SESSION_ENDED | Session terminée |
