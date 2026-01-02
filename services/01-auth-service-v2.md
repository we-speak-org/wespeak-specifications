# Auth Service - Spécifications Techniques v2.0 (Keycloak Integration)

*Voir le début du document dans 01-auth-service.md pour l'architecture avec Keycloak*

---

## 3. Modèle de Données (MongoDB)

### 3.1 Collection : user_profiles

```javascript
{
  "_id": ObjectId,
  "keycloakUserId": "uuid", // Référence vers Keycloak
  "email": "user@example.com",
  "displayName": "John Doe",
  "avatarUrl": "https://cdn.wespeak.com/avatars/user123.jpg",
  "dateOfBirth": ISODate("1990-05-15"),
  "country": "FR",
  "timezone": "Europe/Paris",
  "uiLanguageCode": "fr", // Langue de l'interface
  
  "subscriptionTier": "free", // free, premium, enterprise
  "subscriptionExpiresAt": ISODate("2025-12-31"),
  "subscriptionAutoRenew": true,
  
  "emailVerified": true,
  "onboardingCompleted": true,
  "onboardingCompletedAt": ISODate("2025-01-15T10:30:00Z"),
  
  "credits": {
    "conversationsRemaining": 3, // Free tier: 3/week
    "conversationsResetAt": ISODate("2025-01-22T00:00:00Z"),
    "aiMinutesRemaining": 30, // Premium: 30 min/month
    "premiumFeaturesAccess": ["advanced_feedback", "custom_challenges"]
  },
  
  "preferences": {
    "notificationsEnabled": true,
    "emailDigest": "weekly", // daily, weekly, never
    "theme": "light", // light, dark, auto
    "autoPlayAudio": true
  },
  
  "lastLoginAt": ISODate("2025-01-15T10:00:00Z"),
  "createdAt": ISODate("2025-01-01T08:00:00Z"),
  "updatedAt": ISODate("2025-01-15T10:30:00Z")
}
```

**Indexes** :
```javascript
db.user_profiles.createIndex({ "keycloakUserId": 1 }, { unique: true })
db.user_profiles.createIndex({ "email": 1 }, { unique: true })
db.user_profiles.createIndex({ "subscriptionTier": 1 })
db.user_profiles.createIndex({ "createdAt": -1 })
```

### 3.2 Collection : learning_profiles

```javascript
{
  "_id": ObjectId,
  "userId": ObjectId, // Référence user_profiles._id
  "keycloakUserId": "uuid",
  
  "nativeLanguageCode": "fr", // ISO 639-1
  "targetLanguageCode": "en",
  "currentLevel": "B1", // A1, A2, B1, B2, C1, C2
  "assessedLevel": "A2", // Niveau évalué par test initial
  
  "goal": "work", // work, travel, studies, personal, other
  "goalDescription": "Improve my business English for international meetings",
  "preferredAccent": "en-US", // en-US, en-GB, es-ES, etc.
  "weeklyGoalMinutes": 150,
  
  "isActive": true,
  "isPrimary": true, // Un seul profil primary par user
  
  "createdAt": ISODate("2025-01-01T08:00:00Z"),
  "updatedAt": ISODate("2025-01-15T10:30:00Z")
}
```

**Indexes** :
```javascript
db.learning_profiles.createIndex({ "userId": 1, "targetLanguageCode": 1 }, { unique: true })
db.learning_profiles.createIndex({ "keycloakUserId": 1 })
db.learning_profiles.createIndex({ "isActive": 1, "isPrimary": 1 })
```

---

## 4. API REST

### 4.1 Endpoints

| Méthode | Route | Description | Auth |
|---------|-------|-------------|------|
| `GET` | `/api/users/me` | Profil utilisateur connecté | JWT |
| `PUT` | `/api/users/me` | Mise à jour profil | JWT |
| `DELETE` | `/api/users/me` | Suppression compte | JWT |
| `GET` | `/api/users/{id}` | Profil par ID (internal) | Service-to-Service |
| `GET` | `/api/learning-profiles` | Liste profils apprentissage | JWT |
| `POST` | `/api/learning-profiles` | Créer profil apprentissage | JWT |
| `PUT` | `/api/learning-profiles/{id}` | Mise à jour profil | JWT |
| `DELETE` | `/api/learning-profiles/{id}` | Supprimer profil | JWT |
| `GET` | `/api/credits` | Crédits utilisateur | JWT |
| `POST` | `/api/credits/consume` | Consommer crédit (internal) | Service-to-Service |
| `POST` | `/api/credits/reset` | Reset crédits hebdo (internal) | Service-to-Service |

### 4.2 Schemas

#### GET /api/users/me

**Response 200** :
```json
{
  "id": "mongo-objectid",
  "keycloakUserId": "keycloak-uuid",
  "email": "user@example.com",
  "displayName": "John Doe",
  "avatarUrl": "https://cdn.wespeak.com/avatars/user123.jpg",
  "country": "FR",
  "timezone": "Europe/Paris",
  "uiLanguageCode": "fr",
  "subscriptionTier": "premium",
  "subscriptionExpiresAt": "2025-12-31T23:59:59Z",
  "emailVerified": true,
  "onboardingCompleted": true,
  "preferences": {
    "notificationsEnabled": true,
    "theme": "dark"
  },
  "createdAt": "2025-01-01T08:00:00Z",
  "lastLoginAt": "2025-01-15T10:00:00Z"
}
```

#### PUT /api/users/me

**Request Body** :
```json
{
  "displayName": "Jean Dupont",
  "avatarUrl": "https://new-avatar.jpg",
  "timezone": "America/New_York",
  "preferences": {
    "theme": "dark",
    "emailDigest": "never"
  }
}
```

#### GET /api/learning-profiles

**Response 200** :
```json
{
  "profiles": [
    {
      "id": "mongo-objectid",
      "nativeLanguageCode": "fr",
      "targetLanguageCode": "en",
      "currentLevel": "B1",
      "goal": "work",
      "preferredAccent": "en-US",
      "weeklyGoalMinutes": 150,
      "isActive": true,
      "isPrimary": true,
      "createdAt": "2025-01-01T08:00:00Z"
    }
  ]
}
```

#### POST /api/learning-profiles

**Request Body** :
```json
{
  "nativeLanguageCode": "fr",
  "targetLanguageCode": "es",
  "currentLevel": "A1",
  "goal": "travel",
  "goalDescription": "Learn Spanish for my upcoming trip to Spain",
  "preferredAccent": "es-ES",
  "weeklyGoalMinutes": 100
}
```

**Response 201** :
```json
{
  "profile": {
    "id": "mongo-objectid",
    "nativeLanguageCode": "fr",
    "targetLanguageCode": "es",
    "currentLevel": "A1",
    "isActive": true,
    "createdAt": "2025-01-15T10:30:00Z"
  }
}
```

#### GET /api/credits

**Response 200** :
```json
{
  "subscriptionTier": "free",
  "credits": {
    "conversationsRemaining": 2,
    "conversationsTotal": 3,
    "conversationsResetAt": "2025-01-22T00:00:00Z",
    "aiMinutesRemaining": null,
    "premiumFeaturesAccess": []
  },
  "quotas": {
    "lessonsPerDay": -1,
    "conversationsPerWeek": 3,
    "feedbackReportsPerMonth": 10
  }
}
```

#### POST /api/credits/consume

**Authentication** : Service-to-Service (API Key)

**Request Body** :
```json
{
  "userId": "mongo-objectid",
  "creditType": "conversation", // conversation, ai_analysis
  "amount": 1
}
```

**Response 200** :
```json
{
  "success": true,
  "remainingCredits": 2,
  "resetAt": "2025-01-22T00:00:00Z"
}
```

**Response 403** (quota épuisé) :
```json
{
  "error": "QUOTA_EXCEEDED",
  "message": "Weekly conversation limit reached (3/3)",
  "resetAt": "2025-01-22T00:00:00Z",
  "upgradeUrl": "https://wespeak.com/upgrade-premium"
}
```

---

## 5. Événements Asynchrones

### 5.1 Messages publiés (Kafka)

**Topic** : `user.events`

**Event : user.registered**
```json
{
  "eventType": "user.registered",
  "version": "1.0",
  "timestamp": "2025-01-15T10:30:00Z",
  "payload": {
    "userId": "mongo-objectid",
    "keycloakUserId": "keycloak-uuid",
    "email": "user@example.com",
    "displayName": "John Doe",
    "subscriptionTier": "free",
    "learningProfiles": []
  },
  "metadata": {
    "correlationId": "uuid",
    "source": "auth-service"
  }
}
```

**Event : learning_profile.created**
```json
{
  "eventType": "learning_profile.created",
  "version": "1.0",
  "timestamp": "2025-01-15T10:30:00Z",
  "payload": {
    "userId": "mongo-objectid",
    "profileId": "mongo-objectid",
    "targetLanguageCode": "en",
    "nativeLanguageCode": "fr",
    "currentLevel": "A1",
    "goal": "work"
  },
  "metadata": {
    "correlationId": "uuid",
    "source": "auth-service"
  }
}
```

**Consommé par** :
- gamification-service : Initialiser profil gamification
- recommendation-service : Créer profil apprenant
- lesson-service : Débloquer leçons niveau A1

### 5.2 Messages consommés

**Topic** : `keycloak.admin.events` (voir section 2.3)

---

## 6. Règles Métier

### 6.1 Gestion des crédits

**Free Tier** :
- 3 conversations/semaine
- Reset tous les lundis à 00:00 (timezone user)
- 10 rapports feedback IA/mois
- Pas d'accès features premium

**Premium Tier** :
- Conversations illimitées
- 30 minutes feedback IA/mois (Whisper + GPT-4)
- Accès toutes features premium
- Priorité matchmaking

**Règles** :
- Si quota épuisé → Proposer upgrade Premium
- Si Premium expire → Passage automatique Free après grace period (7 jours)
- Crédits non utilisés ne se cumulent pas

### 6.2 Learning Profiles

**Contraintes** :
- Maximum 5 profils d'apprentissage par utilisateur
- Un seul profil "primary" par utilisateur
- Cannot delete primary profile (must set another as primary first)
- Niveau initial : A1 par défaut, modifiable après test de niveau

---

## 7. Configuration

```yaml
spring:
  application:
    name: auth-service
  
  data:
    mongodb:
      uri: mongodb://mongo:27017/wespeak_auth
      database: wespeak_auth
  
  redis:
    host: redis
    port: 6379
  
  kafka:
    bootstrap-servers: kafka:9092
    consumer:
      group-id: auth-service
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer

keycloak:
  realm: wespeak
  auth-server-url: https://keycloak.wespeak.com/auth
  admin:
    username: ${KEYCLOAK_ADMIN_USERNAME}
    password: ${KEYCLOAK_ADMIN_PASSWORD}
  event-listener:
    topics: keycloak.admin.events
    enabled: true

credits:
  free-tier:
    conversations-per-week: 3
    ai-minutes-per-month: 0
  premium-tier:
    conversations-per-week: -1  # Unlimited
    ai-minutes-per-month: 30
```

---

**Version** : 2.0.0  
**Dernière mise à jour** : 2025-01-15  
**Auteur** : WeSpeak Product Owner AI  
**Focus** : Keycloak Integration, MongoDB, Credits Management
