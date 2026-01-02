# Auth Service - API REST Endpoints

## Base URL

```
http://localhost:8081/api
```

---

## 📍 Endpoints

### User Profile Management

#### GET /users/me

Récupère le profil de l'utilisateur connecté.

**Authentication** : JWT Bearer Token (Keycloak)

**Response 200 OK** :
```json
{
  "id": "507f1f77bcf86cd799439011",
  "keycloakUserId": "a3b5c7d9-1234-5678-90ab-cdef12345678",
  "email": "john.doe@example.com",
  "displayName": "John Doe",
  "avatarUrl": "https://cdn.wespeak.com/avatars/john123.jpg",
  "country": "FR",
  "timezone": "Europe/Paris",
  "uiLanguageCode": "fr",
  "subscriptionTier": "premium",
  "subscriptionExpiresAt": "2025-12-31T23:59:59Z",
  "emailVerified": true,
  "onboardingCompleted": true,
  "preferences": {
    "notificationsEnabled": true,
    "theme": "dark",
    "emailDigest": "weekly",
    "autoPlayAudio": true
  },
  "createdAt": "2025-01-01T08:00:00Z",
  "lastLoginAt": "2025-01-15T10:00:00Z"
}
```

**curl** :
```bash
curl -X GET http://localhost:8081/api/users/me \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

#### PUT /users/me

Met à jour le profil de l'utilisateur.

**Authentication** : JWT Bearer Token

**Request Body** :
```json
{
  "displayName": "Jean Dupont",
  "avatarUrl": "https://cdn.wespeak.com/avatars/new-avatar.jpg",
  "timezone": "America/New_York",
  "country": "US",
  "uiLanguageCode": "en",
  "preferences": {
    "theme": "dark",
    "emailDigest": "never",
    "notificationsEnabled": false
  }
}
```

**Response 200 OK** :
```json
{
  "id": "507f1f77bcf86cd799439011",
  "displayName": "Jean Dupont",
  "timezone": "America/New_York",
  "updatedAt": "2025-01-15T14:30:00Z"
}
```

**curl** :
```bash
curl -X PUT http://localhost:8081/api/users/me \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "Jean Dupont",
    "timezone": "America/New_York",
    "preferences": {
      "theme": "dark"
    }
  }'
```

---

#### DELETE /users/me

Supprime le compte utilisateur (soft delete).

**Authentication** : JWT Bearer Token

**Response 204 No Content**

**curl** :
```bash
curl -X DELETE http://localhost:8081/api/users/me \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

### Learning Profiles Management

#### GET /learning-profiles

Liste tous les profils d'apprentissage de l'utilisateur connecté.

**Authentication** : JWT Bearer Token

**Response 200 OK** :
```json
{
  "profiles": [
    {
      "id": "507f1f77bcf86cd799439022",
      "nativeLanguageCode": "fr",
      "targetLanguageCode": "en",
      "currentLevel": "B1",
      "assessedLevel": "A2",
      "goal": "work",
      "goalDescription": "Improve business English",
      "preferredAccent": "en-US",
      "weeklyGoalMinutes": 180,
      "isActive": true,
      "isPrimary": true,
      "createdAt": "2025-01-01T08:00:00Z"
    },
    {
      "id": "507f1f77bcf86cd799439033",
      "nativeLanguageCode": "fr",
      "targetLanguageCode": "es",
      "currentLevel": "A1",
      "goal": "travel",
      "preferredAccent": "es-ES",
      "weeklyGoalMinutes": 120,
      "isActive": true,
      "isPrimary": false,
      "createdAt": "2025-01-10T12:00:00Z"
    }
  ],
  "total": 2,
  "maxAllowed": 5
}
```

**curl** :
```bash
curl -X GET http://localhost:8081/api/learning-profiles \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

#### POST /learning-profiles

Crée un nouveau profil d'apprentissage.

**Authentication** : JWT Bearer Token

**Request Body** :
```json
{
  "nativeLanguageCode": "fr",
  "targetLanguageCode": "es",
  "currentLevel": "A1",
  "goal": "travel",
  "goalDescription": "Learn Spanish for my upcoming trip to Spain",
  "preferredAccent": "es-ES",
  "weeklyGoalMinutes": 120
}
```

**Response 201 Created** :
```json
{
  "profile": {
    "id": "507f1f77bcf86cd799439033",
    "nativeLanguageCode": "fr",
    "targetLanguageCode": "es",
    "currentLevel": "A1",
    "goal": "travel",
    "isActive": true,
    "isPrimary": false,
    "createdAt": "2025-01-15T14:30:00Z"
  }
}
```

**Response 400 Bad Request** (max profiles reached) :
```json
{
  "error": "MAX_PROFILES_REACHED",
  "message": "You have reached the maximum number of learning profiles (5)",
  "maxAllowed": 5
}
```

**curl** :
```bash
curl -X POST http://localhost:8081/api/learning-profiles \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "nativeLanguageCode": "fr",
    "targetLanguageCode": "es",
    "currentLevel": "A1",
    "goal": "travel",
    "weeklyGoalMinutes": 120
  }'
```

---

#### PUT /learning-profiles/{id}

Met à jour un profil d'apprentissage.

**Authentication** : JWT Bearer Token

**Request Body** :
```json
{
  "currentLevel": "A2",
  "weeklyGoalMinutes": 200,
  "isPrimary": true
}
```

**Response 200 OK** :
```json
{
  "profile": {
    "id": "507f1f77bcf86cd799439033",
    "currentLevel": "A2",
    "weeklyGoalMinutes": 200,
    "isPrimary": true,
    "updatedAt": "2025-01-15T15:00:00Z"
  }
}
```

**curl** :
```bash
curl -X PUT http://localhost:8081/api/learning-profiles/507f1f77bcf86cd799439033 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "currentLevel": "A2",
    "isPrimary": true
  }'
```

---

#### DELETE /learning-profiles/{id}

Supprime un profil d'apprentissage.

**Authentication** : JWT Bearer Token

**Response 204 No Content**

**Response 400 Bad Request** (cannot delete primary) :
```json
{
  "error": "CANNOT_DELETE_PRIMARY",
  "message": "Cannot delete primary learning profile. Set another profile as primary first."
}
```

**curl** :
```bash
curl -X DELETE http://localhost:8081/api/learning-profiles/507f1f77bcf86cd799439033 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

### Credits & Quotas Management

#### GET /credits

Récupère les crédits et quotas de l'utilisateur.

**Authentication** : JWT Bearer Token

**Response 200 OK** :
```json
{
  "subscriptionTier": "free",
  "credits": {
    "conversationsRemaining": 2,
    "conversationsTotal": 3,
    "conversationsResetAt": "2025-01-22T00:00:00Z",
    "aiMinutesRemaining": 0,
    "aiMinutesTotal": 0,
    "premiumFeaturesAccess": []
  },
  "quotas": {
    "lessonsPerDay": -1,
    "conversationsPerWeek": 3,
    "feedbackReportsPerMonth": 10
  },
  "upgradeUrl": "https://wespeak.com/upgrade-premium"
}
```

**curl** :
```bash
curl -X GET http://localhost:8081/api/credits \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

#### POST /credits/consume (Internal Service-to-Service)

Consomme un crédit (appelé par conversation-service ou feedback-service).

**Authentication** : Service-to-Service (API Key)

**Request Body** :
```json
{
  "userId": "507f1f77bcf86cd799439011",
  "creditType": "conversation",
  "amount": 1
}
```

**Response 200 OK** :
```json
{
  "success": true,
  "remainingCredits": 2,
  "resetAt": "2025-01-22T00:00:00Z"
}
```

**Response 403 Forbidden** (quota exceeded) :
```json
{
  "error": "QUOTA_EXCEEDED",
  "message": "Weekly conversation limit reached (3/3)",
  "resetAt": "2025-01-22T00:00:00Z",
  "upgradeUrl": "https://wespeak.com/upgrade-premium"
}
```

**curl** :
```bash
curl -X POST http://localhost:8081/api/credits/consume \
  -H "X-API-Key: YOUR_SERVICE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "507f1f77bcf86cd799439011",
    "creditType": "conversation",
    "amount": 1
  }'
```

---

## 🔒 Authentication

### JWT Bearer Token (User Endpoints)

Tous les endpoints utilisateur nécessitent un JWT émis par Keycloak :

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key (Service-to-Service)

Les endpoints internes nécessitent une clé API :

```http
X-API-Key: wespeak-service-api-key-12345
```

---

## 📊 Error Codes

| Code | Message | Description |
|------|---------|-------------|
| 400 | `INVALID_REQUEST` | Données de requête invalides |
| 400 | `MAX_PROFILES_REACHED` | Limite de 5 profils atteinte |
| 400 | `CANNOT_DELETE_PRIMARY` | Impossible de supprimer le profil principal |
| 401 | `UNAUTHORIZED` | Token JWT manquant ou invalide |
| 403 | `QUOTA_EXCEEDED` | Quota de crédits épuisé |
| 404 | `USER_NOT_FOUND` | Utilisateur introuvable |
| 404 | `PROFILE_NOT_FOUND` | Profil d'apprentissage introuvable |
| 500 | `INTERNAL_ERROR` | Erreur serveur interne |

---

## 📝 Notes

- **Rate Limiting** : 100 requêtes/minute par utilisateur
- **Pagination** : Pas implémentée (max 5 profils par user)
- **Versioning** : API v1 (préfixe `/api`)
