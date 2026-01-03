# Gamification Service - API Endpoints

## Base URL
```
/api/v1/gamification
```

## Endpoints

### Stats Utilisateur

#### GET /stats
Récupère les stats de l'utilisateur connecté pour une langue.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/stats?language=en" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "id": "uuid",
  "userId": "user-uuid",
  "targetLanguageCode": "en",
  "totalXp": 1250,
  "weeklyXp": 320,
  "monthlyXp": 890,
  "currentStreak": 12,
  "longestStreak": 15,
  "lastActivityDate": "2026-01-03T10:30:00Z",
  "streakFreezeAvailable": 1,
  "level": 8,
  "lessonsCompleted": 45,
  "exercisesCompleted": 180,
  "conversationMinutes": 120,
  "perfectLessons": 12
}
```

#### GET /stats/{userId}
Récupère les stats publiques d'un autre utilisateur.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/stats/user-uuid?language=en" \
  -H "Authorization: Bearer {token}"
```

---

### Badges

#### GET /badges
Liste tous les badges disponibles.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/badges" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "badges": [
    {
      "id": "badge-uuid",
      "code": "first_lesson",
      "name": "Première Leçon",
      "description": "Complétez votre première leçon",
      "iconUrl": "/badges/first_lesson.png",
      "category": "lessons",
      "xpReward": 10,
      "rarity": "common"
    }
  ]
}
```

#### GET /badges/my
Liste les badges de l'utilisateur connecté.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/badges/my?language=en" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "unlockedBadges": [
    {
      "badge": {
        "id": "badge-uuid",
        "code": "first_lesson",
        "name": "Première Leçon",
        "iconUrl": "/badges/first_lesson.png",
        "rarity": "common"
      },
      "unlockedAt": "2026-01-01T14:30:00Z"
    }
  ],
  "totalUnlocked": 5,
  "totalAvailable": 25
}
```

---

### Leaderboards

#### GET /leaderboard
Récupère le classement.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/leaderboard?language=en&type=weekly&limit=50" \
  -H "Authorization: Bearer {token}"
```

**Query Parameters:**
- `language` (obligatoire): Code langue
- `type` (obligatoire): weekly, monthly, global
- `limit` (optionnel): Nombre de résultats (défaut: 50, max: 100)

**Response 200:**
```json
{
  "type": "weekly",
  "targetLanguageCode": "en",
  "period": {
    "start": "2025-12-30T00:00:00Z",
    "end": "2026-01-05T23:59:59Z"
  },
  "rankings": [
    {
      "rank": 1,
      "userId": "user-uuid-1",
      "displayName": "Marie D.",
      "avatarUrl": "/avatars/user1.jpg",
      "xp": 850,
      "level": 12
    },
    {
      "rank": 2,
      "userId": "user-uuid-2",
      "displayName": "Jean P.",
      "avatarUrl": "/avatars/user2.jpg",
      "xp": 720,
      "level": 10
    }
  ],
  "currentUserRank": {
    "rank": 15,
    "xp": 320
  }
}
```

---

### Défis

#### GET /challenges
Liste les défis actifs.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/challenges" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "challenges": [
    {
      "id": "challenge-uuid",
      "title": "Défi de la semaine",
      "description": "Gagnez 500 XP cette semaine",
      "type": "weekly",
      "targetType": "xp",
      "targetValue": 500,
      "xpReward": 100,
      "startDate": "2025-12-30T00:00:00Z",
      "endDate": "2026-01-05T23:59:59Z",
      "userProgress": {
        "currentProgress": 320,
        "status": "in_progress",
        "percentComplete": 64
      }
    }
  ]
}
```

#### POST /challenges/{challengeId}/join
S'inscrire à un défi.

```bash
curl -X POST "http://localhost:8085/api/v1/gamification/challenges/challenge-uuid/join" \
  -H "Authorization: Bearer {token}"
```

**Response 201:**
```json
{
  "id": "user-challenge-uuid",
  "challengeId": "challenge-uuid",
  "status": "in_progress",
  "currentProgress": 0,
  "joinedAt": "2026-01-03T10:00:00Z"
}
```

#### GET /challenges/my
Liste les défis de l'utilisateur.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/challenges/my?status=in_progress" \
  -H "Authorization: Bearer {token}"
```

---

### Transactions XP

#### GET /xp/history
Historique des gains d'XP.

```bash
curl -X GET "http://localhost:8085/api/v1/gamification/xp/history?language=en&limit=20" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "transactions": [
    {
      "id": "tx-uuid",
      "amount": 15,
      "source": "lesson",
      "description": "Leçon complétée: Les salutations",
      "createdAt": "2026-01-03T10:30:00Z"
    },
    {
      "id": "tx-uuid-2",
      "amount": 10,
      "source": "badge",
      "description": "Badge débloqué: Première Leçon",
      "createdAt": "2026-01-03T10:30:00Z"
    }
  ],
  "totalToday": 45,
  "totalThisWeek": 320
}
```

---

### Streak

#### POST /streak/freeze
Utiliser un streak freeze.

```bash
curl -X POST "http://localhost:8085/api/v1/gamification/streak/freeze?language=en" \
  -H "Authorization: Bearer {token}"
```

**Response 200:**
```json
{
  "freezeUsed": true,
  "freezesRemaining": 0,
  "streakProtectedUntil": "2026-01-04T23:59:59Z"
}
```

**Response 400:**
```json
{
  "error": "NO_FREEZE_AVAILABLE",
  "message": "Vous n'avez plus de streak freeze disponible"
}
```

---

## Codes d'Erreur

| Code HTTP | Code Erreur | Description |
|-----------|-------------|-------------|
| 400 | NO_FREEZE_AVAILABLE | Pas de streak freeze disponible |
| 400 | CHALLENGE_ALREADY_JOINED | Déjà inscrit à ce défi |
| 400 | CHALLENGE_EXPIRED | Le défi est terminé |
| 404 | STATS_NOT_FOUND | Stats utilisateur non trouvées |
| 404 | CHALLENGE_NOT_FOUND | Défi non trouvé |
| 404 | BADGE_NOT_FOUND | Badge non trouvé |
