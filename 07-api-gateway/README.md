# API Gateway - Spécifications Fonctionnelles v1.0

## 1. Vue d'Ensemble

### 1.1 Responsabilité
L'**api-gateway** est le point d'entrée unique pour tous les clients (web, mobile). Il assure :
- Routage des requêtes vers les microservices appropriés
- Authentification et validation des tokens JWT
- Rate limiting et protection contre les abus
- Agrégation de réponses si nécessaire
- CORS et gestion des headers

### 1.2 Objectif Fonctionnel
- Simplifier l'accès aux microservices pour les clients
- Centraliser la sécurité et l'authentification
- Fournir une API unifiée et documentée
- Protéger les services backend

### 1.3 Services Routés
| Route Prefix | Service Cible | Port |
|--------------|---------------|------|
| /api/v1/auth | auth-service | 8081 |
| /api/v1/lessons | lesson-service | 8082 |
| /api/v1/conversations | conversation-service | 8083 |
| /api/v1/feedback | feedback-service | 8084 |
| /api/v1/gamification | gamification-service | 8085 |
| /api/v1/recommendations | recommendation-service | 8086 |

---

## 2. Fonctionnalités

### 2.1 Routage

Le gateway route les requêtes vers le service approprié en fonction du préfixe d'URL.

**Exemple :**
```
Client → GET /api/v1/lessons/courses
Gateway → GET http://lesson-service:8082/api/v1/lessons/courses
```

### 2.2 Authentification

**Endpoints publics (sans token) :**
- POST /api/v1/auth/register
- POST /api/v1/auth/login
- POST /api/v1/auth/refresh
- GET /api/v1/auth/health
- GET /health

**Endpoints protégés (token JWT requis) :**
- Tous les autres endpoints

**Validation du token :**
1. Extraire le token du header `Authorization: Bearer {token}`
2. Valider la signature JWT avec la clé secrète
3. Vérifier l'expiration
4. Extraire userId et roles du payload
5. Ajouter headers `X-User-Id` et `X-User-Roles` pour les services downstream

### 2.3 Rate Limiting

**Limites par défaut :**
| Type | Limite | Fenêtre |
|------|--------|---------|
| Par IP (non authentifié) | 100 req | 1 minute |
| Par utilisateur (authentifié) | 300 req | 1 minute |
| Par endpoint sensible | 10 req | 1 minute |

**Endpoints sensibles :**
- POST /api/v1/auth/login (protection brute force)
- POST /api/v1/auth/register
- POST /api/v1/feedback/analyze (coûteux en IA)

### 2.4 CORS

**Configuration :**
```
Allowed Origins: https://wespeak.app, http://localhost:4200
Allowed Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Allowed Headers: Authorization, Content-Type, X-Request-ID
Exposed Headers: X-RateLimit-Remaining, X-RateLimit-Reset
Max Age: 3600
```

### 2.5 Headers de Sécurité

Headers ajoutés à toutes les réponses :
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

---

## 3. Routes Détaillées

### 3.1 Auth Service Routes

| Méthode | Route | Auth | Description |
|---------|-------|------|-------------|
| POST | /api/v1/auth/register | Non | Inscription |
| POST | /api/v1/auth/login | Non | Connexion |
| POST | /api/v1/auth/refresh | Non | Rafraîchir token |
| POST | /api/v1/auth/logout | Oui | Déconnexion |
| GET | /api/v1/auth/me | Oui | Profil utilisateur |
| PUT | /api/v1/auth/me | Oui | Modifier profil |
| GET | /api/v1/auth/learning-profiles | Oui | Profils d'apprentissage |
| POST | /api/v1/auth/learning-profiles | Oui | Créer profil |

### 3.2 Lesson Service Routes

| Méthode | Route | Auth | Description |
|---------|-------|------|-------------|
| GET | /api/v1/lessons/courses | Oui | Liste des cours |
| GET | /api/v1/lessons/courses/{id} | Oui | Détail cours |
| GET | /api/v1/lessons/units/{id} | Oui | Détail unité |
| GET | /api/v1/lessons/lessons/{id} | Oui | Détail leçon |
| POST | /api/v1/lessons/exercises/{id}/submit | Oui | Soumettre réponse |
| GET | /api/v1/lessons/progress | Oui | Progression |

### 3.3 Conversation Service Routes

| Méthode | Route | Auth | Description |
|---------|-------|------|-------------|
| GET | /api/v1/conversations/slots | Oui | Créneaux disponibles |
| POST | /api/v1/conversations/slots/{id}/register | Oui | S'inscrire |
| DELETE | /api/v1/conversations/slots/{id}/register | Oui | Se désinscrire |
| GET | /api/v1/conversations/sessions/{id} | Oui | Détail session |
| POST | /api/v1/conversations/sessions/{id}/join | Oui | Rejoindre |
| POST | /api/v1/conversations/sessions/{id}/leave | Oui | Quitter |

### 3.4 Feedback Service Routes

| Méthode | Route | Auth | Description |
|---------|-------|------|-------------|
| GET | /api/v1/feedback/reports | Oui | Mes rapports |
| GET | /api/v1/feedback/reports/{id} | Oui | Détail rapport |
| GET | /api/v1/feedback/transcripts/{sessionId} | Oui | Transcript session |

### 3.5 Gamification Service Routes

| Méthode | Route | Auth | Description |
|---------|-------|------|-------------|
| GET | /api/v1/gamification/stats | Oui | Mes stats |
| GET | /api/v1/gamification/badges | Oui | Tous les badges |
| GET | /api/v1/gamification/badges/my | Oui | Mes badges |
| GET | /api/v1/gamification/leaderboard | Oui | Classement |
| GET | /api/v1/gamification/challenges | Oui | Défis actifs |
| POST | /api/v1/gamification/challenges/{id}/join | Oui | Rejoindre défi |

### 3.6 Recommendation Service Routes

| Méthode | Route | Auth | Description |
|---------|-------|------|-------------|
| GET | /api/v1/recommendations | Oui | Mes recommandations |
| POST | /api/v1/recommendations/{id}/click | Oui | Clic recommandation |
| POST | /api/v1/recommendations/{id}/dismiss | Oui | Ignorer |
| GET | /api/v1/recommendations/preferences | Oui | Préférences |
| PUT | /api/v1/recommendations/preferences | Oui | Modifier préférences |

---

## 4. Gestion des Erreurs

### 4.1 Format Standard

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Token invalide ou expiré",
    "details": null,
    "timestamp": "2026-01-03T10:00:00Z",
    "path": "/api/v1/lessons/courses",
    "requestId": "req-uuid"
  }
}
```

### 4.2 Codes d'Erreur Gateway

| Code HTTP | Code | Description |
|-----------|------|-------------|
| 401 | UNAUTHORIZED | Token manquant ou invalide |
| 401 | TOKEN_EXPIRED | Token expiré |
| 403 | FORBIDDEN | Accès non autorisé |
| 429 | RATE_LIMITED | Trop de requêtes |
| 502 | SERVICE_UNAVAILABLE | Service backend indisponible |
| 504 | GATEWAY_TIMEOUT | Timeout du service backend |

---

## 5. Observabilité

### 5.1 Logging

Chaque requête génère un log avec :
- Request ID (généré ou passé via X-Request-ID)
- Méthode et URL
- User ID (si authentifié)
- Status code
- Durée
- Service cible

### 5.2 Métriques

- Nombre de requêtes par service
- Latence par service (p50, p95, p99)
- Taux d'erreur par service
- Rate limit hits

### 5.3 Health Check

**GET /health**
```json
{
  "status": "UP",
  "services": {
    "auth-service": "UP",
    "lesson-service": "UP",
    "conversation-service": "UP",
    "feedback-service": "UP",
    "gamification-service": "UP",
    "recommendation-service": "UP"
  }
}
```

---

## 6. Configuration

### 6.1 Variables d'Environnement

| Variable | Description | Défaut |
|----------|-------------|--------|
| PORT | Port du gateway | 8080 |
| JWT_SECRET | Clé secrète JWT | - |
| AUTH_SERVICE_URL | URL auth-service | http://localhost:8081 |
| LESSON_SERVICE_URL | URL lesson-service | http://localhost:8082 |
| CONVERSATION_SERVICE_URL | URL conversation-service | http://localhost:8083 |
| FEEDBACK_SERVICE_URL | URL feedback-service | http://localhost:8084 |
| GAMIFICATION_SERVICE_URL | URL gamification-service | http://localhost:8085 |
| RECOMMENDATION_SERVICE_URL | URL recommendation-service | http://localhost:8086 |
| RATE_LIMIT_ENABLED | Activer rate limiting | true |
| CORS_ORIGINS | Origines autorisées | http://localhost:4200 |

---

## 7. Stack Technique

Pour ce service spécifique, utiliser **Spring Cloud Gateway** :
- Routage réactif haute performance
- Filtres pré/post requête
- Intégration native avec Spring Security
- Circuit breaker avec Resilience4j
