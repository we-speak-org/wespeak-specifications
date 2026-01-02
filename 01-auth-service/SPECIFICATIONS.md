# Auth Service - Spécifications Techniques Détaillées v1.0

## 📋 Table des Matières

1. [Vue d'Ensemble](#1-vue-densemble)
2. [Modèle de Données](#2-modèle-de-données)
3. [API REST](#3-api-rest)
4. [Événements Asynchrones](#4-événements-asynchrones)
5. [Règles Métier](#5-règles-métier)
6. [Performance et Scalabilité](#6-performance-et-scalabilité)
7. [Sécurité](#7-sécurité)
8. [Tests](#8-tests)
9. [Monitoring et Logs](#9-monitoring-et-logs)
10. [Configuration](#10-configuration)
11. [Migration et Déploiement](#11-migration-et-déploiement)

---

## 1. Vue d'Ensemble

### 1.1 Responsabilité

Le **Auth Service** est le microservice central de WeSpeak responsable de :
- **Authentification** : Inscription, connexion, gestion des sessions utilisateurs
- **Gestion des profils utilisateurs** : Informations personnelles, préférences UI
- **Gestion des profils d'apprentissage** : Support multi-langues avec profils par langue cible
- **Gestion des abonnements** : Free, Premium, Enterprise
- **Sécurité** : JWT, tokens de refresh, vérification email, réinitialisation mot de passe

### 1.2 Dépendances

**Services consommés** :
- Aucun (service autonome de base)

**Services consommateurs** :
- `lesson-service` : Vérification identité et niveau utilisateur
- `conversation-service` : Match

making basé sur profils
- `feedback-service` : Attribution feedback par utilisateur
- `gamification-service` : Points XP et achievements
- `recommendation-service` : Personnalisation recommandations
- `api-gateway` : Validation tokens JWT

**Infrastructure externe** :
- PostgreSQL : Stockage données utilisateurs et profils
- Redis : Cache sessions et tokens
- Kafka : Publication événements utilisateurs
- SMTP : Envoi emails de vérification

### 1.3 Technologies

- **Framework** : Spring Boot 3.x
- **Langage** : Java 21
- **Build** : Gradle 9.2.1
- **Base de données** : PostgreSQL 16+
- **Cache** : Redis 7+
- **Messaging** : Spring Cloud Stream + Kafka
- **Sécurité** : Spring Security 6, JWT (jjwt library)
- **Documentation API** : SpringDoc OpenAPI 3
- **Validation** : Jakarta Bean Validation
- **Tests** : JUnit 5, Testcontainers, MockMVC

---

## 2. Modèle de Données

### 2.1 Entités

#### User Entity
```java
package org.wespeak.auth;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_user_email", columnList = "email", unique = true),
    @Index(name = "idx_user_subscription", columnList = "subscriptionTier, subscriptionExpiresAt")
})
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    
    @Email
    @NotNull
    @Column(unique = true, nullable = false, length = 255)
    private String email;
    
    @NotNull
    @Column(nullable = false, length = 255)
    private String passwordHash;
    
    @NotNull
    @Size(min = 2, max = 100)
    @Column(nullable = false, length = 100)
    private String displayName;
    
    @Column(length = 500)
    private String avatarUrl;
    
    private LocalDate dateOfBirth;
    
    @Column(length = 2)
    private String country; // ISO 3166-1 alpha-2
    
    @NotNull
    @Column(nullable = false, length = 10)
    private String uiLanguageCode; // ex: 'en', 'fr', 'es'
    
    @NotNull
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private SubscriptionTier subscriptionTier;
    
    private LocalDateTime subscriptionExpiresAt;
    
    @NotNull
    @Column(nullable = false)
    private Boolean emailVerified = false;
    
    @Column(length = 500)
    private String emailVerificationToken;
    
    private LocalDateTime emailVerificationTokenExpiresAt;
    
    @Column(length = 500)
    private String passwordResetToken;
    
    private LocalDateTime passwordResetTokenExpiresAt;
    
    @NotNull
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @NotNull
    @Column(nullable = false)
    private LocalDateTime updatedAt;
    
    private LocalDateTime lastLoginAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}

enum SubscriptionTier {
    FREE,
    PREMIUM,
    ENTERPRISE
}
```

#### LearningProfile Entity
```java
package org.wespeak.auth;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "learning_profiles", indexes = {
    @Index(name = "idx_lp_user", columnList = "userId"),
    @Index(name = "idx_lp_active", columnList = "userId, targetLanguageCode, isActive"),
    @Index(name = "idx_lp_level", columnList = "targetLanguageCode, currentLevel")
})
@Data
@NoArgsConstructor
@AllArgsConstructor
public class LearningProfile {
    
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    
    @NotNull
    @Column(nullable = false)
    private UUID userId;
    
    @NotNull
    @Column(nullable = false, length = 10)
    private String nativeLanguageCode; // ex: 'fr', 'en'
    
    @NotNull
    @Column(nullable = false, length = 10)
    private String targetLanguageCode; // ex: 'en', 'es'
    
    @NotNull
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 5)
    private CEFRLevel currentLevel;
    
    @Enumerated(EnumType.STRING)
    @Column(length = 5)
    private CEFRLevel assessedLevel; // Niveau évalué par test
    
    @NotNull
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private LearningGoal goal;
    
    @Column(length = 10)
    private String preferredAccent; // ex: 'en-US', 'en-GB', 'es-ES'
    
    @NotNull
    @Min(0)
    @Max(1440) // 24 heures max
    @Column(nullable = false)
    private Integer weeklyGoalMinutes;
    
    @NotNull
    @Column(nullable = false)
    private Boolean isActive = true;
    
    @NotNull
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @NotNull
    @Column(nullable = false)
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}

enum CEFRLevel {
    A1, A2, B1, B2, C1, C2
}

enum LearningGoal {
    WORK,
    TRAVEL,
    STUDIES,
    PERSONAL,
    OTHER
}
```

### 2.2 Relations

```mermaid
erDiagram
    USER ||--o{ LEARNING_PROFILE : "has"
    USER {
        uuid id PK
        string email UK
        string passwordHash
        string displayName
        string avatarUrl
        date dateOfBirth
        string country
        string uiLanguageCode
        enum subscriptionTier
        datetime subscriptionExpiresAt
        boolean emailVerified
        datetime createdAt
        datetime updatedAt
        datetime lastLoginAt
    }
    LEARNING_PROFILE {
        uuid id PK
        uuid userId FK
        string nativeLanguageCode
        string targetLanguageCode
        enum currentLevel
        enum assessedLevel
        enum goal
        string preferredAccent
        int weeklyGoalMinutes
        boolean isActive
        datetime createdAt
        datetime updatedAt
    }
```

**Contraintes** :
- Un `User` peut avoir plusieurs `LearningProfile`
- Un seul profil actif par `targetLanguageCode` par utilisateur
- Suppression en cascade des profils si l'utilisateur est supprimé

### 2.3 Indexes

**Table `users`** :
- `PRIMARY KEY (id)`
- `UNIQUE INDEX idx_user_email (email)`
- `INDEX idx_user_subscription (subscriptionTier, subscriptionExpiresAt)` - pour recherches par tier

**Table `learning_profiles`** :
- `PRIMARY KEY (id)`
- `INDEX idx_lp_user (userId)` - pour récupération profils par utilisateur
- `INDEX idx_lp_active (userId, targetLanguageCode, isActive)` - pour profil actif
- `INDEX idx_lp_level (targetLanguageCode, currentLevel)` - pour stats et matchmaking

**Optimisations** :
- Index partiel sur `emailVerified = false` si volume élevé de comptes non vérifiés
- Index B-tree par défaut pour recherches par plages de dates

---

## 3. API REST

### 3.1 Endpoints

| Méthode | Route | Description | Auth requise |
|---------|-------|-------------|--------------|
| POST | `/api/auth/register` | Inscription nouvel utilisateur | Non |
| POST | `/api/auth/login` | Connexion utilisateur | Non |
| POST | `/api/auth/logout` | Déconnexion utilisateur | Oui |
| POST | `/api/auth/refresh-token` | Renouvellement token JWT | Non (refresh token) |
| POST | `/api/auth/forgot-password` | Demande réinitialisation mot de passe | Non |
| POST | `/api/auth/reset-password` | Réinitialisation mot de passe | Non |
| GET | `/api/auth/verify-email` | Vérification email via token | Non |
| GET | `/api/users/me` | Récupération profil utilisateur connecté | Oui |
| PUT | `/api/users/me` | Mise à jour profil utilisateur | Oui |
| DELETE | `/api/users/me` | Suppression compte utilisateur | Oui |
| POST | `/api/users/me/change-password` | Changement mot de passe | Oui |
| GET | `/api/learning-profiles` | Liste profils d'apprentissage | Oui |
| POST | `/api/learning-profiles` | Création profil d'apprentissage | Oui |
| GET | `/api/learning-profiles/:id` | Récupération profil spécifique | Oui |
| PUT | `/api/learning-profiles/:id` | Mise à jour profil | Oui |
| DELETE | `/api/learning-profiles/:id` | Suppression profil | Oui |
| PUT | `/api/learning-profiles/:id/activate` | Activation profil pour une langue | Oui |

### 3.2 Schémas Détaillés

#### POST /api/auth/register

**Request Body** :
```json
{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd",
  "displayName": "John Doe",
  "uiLanguageCode": "en",
  "nativeLanguageCode": "fr",
  "targetLanguageCode": "en",
  "currentLevel": "A2",
  "goal": "WORK",
  "weeklyGoalMinutes": 120
}
```

**Response 201 Created** :
```json
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "displayName": "John Doe",
  "emailVerified": false,
  "subscriptionTier": "FREE",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "dGVzdC1yZWZyZXNoLXRva2Vu...",
  "expiresIn": 3600,
  "learningProfile": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "targetLanguageCode": "en",
    "nativeLanguageCode": "fr",
    "currentLevel": "A2"
  }
}
```

**Response 400 Bad Request** :
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Email already exists",
  "timestamp": "2025-01-02T21:10:00Z",
  "path": "/api/auth/register"
}
```

#### POST /api/auth/login

**Request Body** :
```json
{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd"
}
```

**Response 200 OK** :
```json
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "displayName": "John Doe",
  "emailVerified": true,
  "subscriptionTier": "PREMIUM",
  "subscriptionExpiresAt": "2025-12-31T23:59:59Z",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "dGVzdC1yZWZyZXNoLXRva2Vu...",
  "expiresIn": 3600
}
```

**Response 401 Unauthorized** :
```json
{
  "error": "INVALID_CREDENTIALS",
  "message": "Email or password is incorrect",
  "timestamp": "2025-01-02T21:10:00Z",
  "path": "/api/auth/login"
}
```

#### POST /api/auth/refresh-token

**Request Body** :
```json
{
  "refreshToken": "dGVzdC1yZWZyZXNoLXRva2Vu..."
}
```

**Response 200 OK** :
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "bmV3LXJlZnJlc2gtdG9rZW4...",
  "expiresIn": 3600
}
```

#### GET /api/users/me

**Headers** :
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response 200 OK** :
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "displayName": "John Doe",
  "avatarUrl": "https://cdn.wespeak.com/avatars/user123.jpg",
  "dateOfBirth": "1990-05-15",
  "country": "FR",
  "uiLanguageCode": "fr",
  "subscriptionTier": "PREMIUM",
  "subscriptionExpiresAt": "2025-12-31T23:59:59Z",
  "emailVerified": true,
  "createdAt": "2024-01-15T10:30:00Z",
  "lastLoginAt": "2025-01-02T20:45:00Z"
}
```

#### GET /api/learning-profiles

**Headers** :
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response 200 OK** :
```json
{
  "profiles": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "nativeLanguageCode": "fr",
      "targetLanguageCode": "en",
      "currentLevel": "B1",
      "assessedLevel": "A2",
      "goal": "WORK",
      "preferredAccent": "en-US",
      "weeklyGoalMinutes": 180,
      "isActive": true,
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2025-01-01T14:20:00Z"
    },
    {
      "id": "770e8400-e29b-41d4-a716-446655440002",
      "nativeLanguageCode": "fr",
      "targetLanguageCode": "es",
      "currentLevel": "A1",
      "goal": "TRAVEL",
      "preferredAccent": "es-ES",
      "weeklyGoalMinutes": 60,
      "isActive": false,
      "createdAt": "2024-06-10T15:45:00Z",
      "updatedAt": "2024-12-01T09:10:00Z"
    }
  ]
}
```

### 3.3 Codes d'Erreur

| Code HTTP | Error Code | Message | Description |
|-----------|------------|---------|-------------|
| 400 | `VALIDATION_ERROR` | Validation failed | Données invalides (format email, mot de passe faible) |
| 400 | `EMAIL_ALREADY_EXISTS` | Email already registered | Email déjà utilisé |
| 400 | `INVALID_TOKEN` | Token is invalid or expired | Token JWT invalide ou expiré |
| 401 | `INVALID_CREDENTIALS` | Email or password is incorrect | Identifiants incorrects |
| 401 | `EMAIL_NOT_VERIFIED` | Email must be verified | Email non vérifié, action bloquée |
| 401 | `UNAUTHORIZED` | Authentication required | Token manquant ou invalide |
| 403 | `SUBSCRIPTION_REQUIRED` | Premium subscription required | Fonctionnalité premium |
| 404 | `USER_NOT_FOUND` | User not found | Utilisateur inexistant |
| 404 | `PROFILE_NOT_FOUND` | Learning profile not found | Profil d'apprentissage inexistant |
| 409 | `PROFILE_ALREADY_ACTIVE` | Active profile already exists for this language | Profil actif existe déjà |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many requests | Trop de requêtes |
| 500 | `INTERNAL_ERROR` | Internal server error | Erreur serveur |

---

## 4. Événements Asynchrones

### 4.1 Messages Publiés

Le auth-service publie des événements sur le topic Kafka **`user.events`** avec partitionnement par `userId`.

#### Event: user.registered

**Topic** : `user.events`  
**Key** : `userId` (pour partitionnement)  
**Format** : CloudEvent JSON

```json
{
  "specversion": "1.0",
  "type": "org.wespeak.user.registered",
  "source": "auth-service",
  "id": "a1b2c3d4-e5f6-7890-abcd-1234567890ab",
  "time": "2025-01-02T21:10:00Z",
  "datacontenttype": "application/json",
  "data": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "displayName": "John Doe",
    "uiLanguageCode": "fr",
    "subscriptionTier": "FREE",
    "learningProfiles": [
      {
        "id": "660e8400-e29b-41d4-a716-446655440001",
        "targetLanguageCode": "en",
        "nativeLanguageCode": "fr",
        "currentLevel": "A2",
        "goal": "WORK"
      }
    ],
    "createdAt": "2025-01-02T21:10:00Z"
  }
}
```

**Consommé par** :
- `gamification-service` : Création compte XP, attribution badge "Welcome"
- `recommendation-service` : Initialisation profil recommandations
- `notification-service` : Envoi email de bienvenue

#### Event: user.profile.updated

```json
{
  "specversion": "1.0",
  "type": "org.wespeak.user.profile.updated",
  "source": "auth-service",
  "id": "b2c3d4e5-f6a7-8901-bcde-234567890abc",
  "time": "2025-01-02T21:15:00Z",
  "datacontenttype": "application/json",
  "data": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "changes": {
      "displayName": "John Smith",
      "avatarUrl": "https://cdn.wespeak.com/avatars/new-avatar.jpg"
    },
    "updatedAt": "2025-01-02T21:15:00Z"
  }
}
```

**Consommé par** :
- `conversation-service` : Mise à jour profil visible en conversation
- `gamification-service` : Mise à jour leaderboards

#### Event: user.subscription.changed

```json
{
  "specversion": "1.0",
  "type": "org.wespeak.user.subscription.changed",
  "source": "auth-service",
  "id": "c3d4e5f6-a7b8-9012-cdef-34567890abcd",
  "time": "2025-01-02T21:20:00Z",
  "datacontenttype": "application/json",
  "data": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "previousTier": "FREE",
    "newTier": "PREMIUM",
    "expiresAt": "2026-01-02T21:20:00Z",
    "changedAt": "2025-01-02T21:20:00Z"
  }
}
```

**Consommé par** :
- `lesson-service` : Déblocage leçons premium
- `conversation-service` : Activation conversations illimitées
- `feedback-service` : Activation feedback détaillé
- `gamification-service` : Attribution badge "Premium Member"

#### Event: learning.profile.created

```json
{
  "specversion": "1.0",
  "type": "org.wespeak.learning.profile.created",
  "source": "auth-service",
  "id": "d4e5f6a7-b8c9-0123-def0-4567890abcde",
  "time": "2025-01-02T21:25:00Z",
  "datacontenttype": "application/json",
  "data": {
    "profileId": "770e8400-e29b-41d4-a716-446655440002",
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "targetLanguageCode": "es",
    "nativeLanguageCode": "fr",
    "currentLevel": "A1",
    "goal": "TRAVEL",
    "weeklyGoalMinutes": 60,
    "createdAt": "2025-01-02T21:25:00Z"
  }
}
```

**Consommé par** :
- `lesson-service` : Initialisation progression pour la langue
- `recommendation-service` : Suggestions premiers cours
- `gamification-service` : Création tracker XP pour la langue

#### Event: learning.profile.level.changed

```json
{
  "specversion": "1.0",
  "type": "org.wespeak.learning.profile.level.changed",
  "source": "auth-service",
  "id": "e5f6a7b8-c9d0-1234-ef01-567890abcdef",
  "time": "2025-01-02T21:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    "profileId": "660e8400-e29b-41d4-a716-446655440001",
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "targetLanguageCode": "en",
    "previousLevel": "A2",
    "newLevel": "B1",
    "changedAt": "2025-01-02T21:30:00Z"
  }
}
```

**Consommé par** :
- `lesson-service` : Déblocage nouveau contenu
- `conversation-service` : Mise à jour pool matchmaking
- `gamification-service` : Attribution achievement "Level Up"

### 4.2 Messages Consommés

Le auth-service **ne consomme aucun événement** des autres services. Il est un service source d'événements uniquement.

---

## 5. Règles Métier

### 5.1 Inscription et Authentification

1. **Validation email** :
   - Format RFC 5322 valide
   - Domaine avec MX record valide (optionnel en production)
   - Email unique dans la base

2. **Validation mot de passe** :
   - Minimum 8 caractères
   - Au moins 1 majuscule, 1 minuscule, 1 chiffre, 1 caractère spécial
   - Pas de mots de passe communs (liste OWASP top 10k)
   - Hash avec bcrypt (cost factor 12)

3. **Vérification email** :
   - Token généré à l'inscription (UUID sécurisé)
   - Validité : 24 heures
   - Email envoyé de manière asynchrone (Kafka → notification-service)
   - Accès limité aux fonctionnalités si non vérifié

4. **Réinitialisation mot de passe** :
   - Token à usage unique (UUID)
   - Validité : 1 heure
   - Invalide après utilisation
   - Rate limiting : 3 demandes max par heure par email

### 5.2 Profils d'Apprentissage

1. **Création profil** :
   - Un utilisateur peut avoir plusieurs profils (une langue cible par profil)
   - Un seul profil actif par `targetLanguageCode`
   - Niveau initial : saisie manuelle ou test d'évaluation (onboarding)
   - `weeklyGoalMinutes` : min 30, max 1440 (24h)

2. **Activation/Désactivation** :
   - Activer un profil désactive automatiquement l'ancien profil actif pour cette langue
   - Un profil désactivé conserve la progression (pas de suppression auto)

3. **Changement de niveau** :
   - Niveau changé manuellement par l'utilisateur ou automatiquement par le système
   - Événement publié pour mettre à jour le contenu accessible

### 5.3 Abonnements

1. **Tier FREE** :
   - 3 conversations par semaine
   - Accès leçons niveau A1-A2 uniquement
   - Feedback IA basique
   - Pas d'accès aux statistiques avancées

2. **Tier PREMIUM** :
   - Conversations illimitées
   - Accès tout contenu (A1-C2)
   - Feedback IA détaillé
   - Statistiques avancées
   - Coaching personnalisé

3. **Tier ENTERPRISE** :
   - Fonctionnalités Premium +
   - Dashboard admin
   - Reporting avancé
   - Support prioritaire

4. **Expiration abonnement** :
   - Vérification quotidienne (job planifié)
   - Downgrade automatique vers FREE si expiré
   - Notification 7 jours avant expiration

### 5.4 Suppression de Compte

1. **Soft delete** (recommandé) :
   - Marquer comme `deleted = true`
   - Anonymiser données personnelles
   - Conserver données agrégées pour analytics

2. **Hard delete** (si demandé RGPD) :
   - Suppression cascade des profils d'apprentissage
   - Anonymisation des conversations passées (garder transcripts sans identité)
   - Suppression complète après 30 jours (période de grâce)

---

## 6. Performance et Scalabilité

### 6.1 Stratégies de Cache

**Redis Cache** :

1. **User Session** :
   - Key : `user:session:{userId}`
   - TTL : 1 heure (durée access token)
   - Données : userId, email, displayName, subscriptionTier, roles
   - Invalidation : Logout, changement mot de passe, mise à jour profil critique

2. **Learning Profiles** :
   - Key : `user:profiles:{userId}`
   - TTL : 5 minutes
   - Données : Liste complète des profils
   - Invalidation : Création, mise à jour, suppression profil

3. **Subscription Status** :
   - Key : `user:subscription:{userId}`
   - TTL : 15 minutes
   - Données : tier, expiresAt, isActive
   - Invalidation : Changement abonnement

**Cache-Aside Pattern** :
```java
public User getUserById(UUID userId) {
    // 1. Try cache
    User cached = redisTemplate.opsForValue().get("user:session:" + userId);
    if (cached != null) return cached;
    
    // 2. Query database
    User user = userRepository.findById(userId)
        .orElseThrow(() -> new UserNotFoundException(userId));
    
    // 3. Store in cache
    redisTemplate.opsForValue().set(
        "user:session:" + userId, 
        user, 
        Duration.ofHours(1)
    );
    
    return user;
}
```

### 6.2 Optimisations Requêtes

1. **Pagination** :
   - Limit par défaut : 20
   - Max : 100
   - Utiliser curseurs pour grandes listes (profils, historique)

2. **Requêtes N+1** :
   - Utiliser `@EntityGraph` ou fetch joins pour charger les relations
   - Exemple : Charger user + learning profiles en une requête

3. **Indexes** :
   - Tous les indexes décrits en section 2.3
   - Analyser requêtes lentes avec `EXPLAIN ANALYZE` (PostgreSQL)

4. **Projections** :
   - DTO projections pour éviter de charger entités complètes
   - Exemple : `UserSummaryDTO` pour listes (id, email, displayName uniquement)

### 6.3 Limites et Quotas

| Action | Limite | Fenêtre | Tier |
|--------|--------|---------|------|
| Login attempts | 5 | 15 minutes | Tous |
| Register | 3 | 1 heure | Tous |
| Password reset | 3 | 1 heure | Tous |
| Profile creation | 5 | 1 jour | FREE |
| Profile creation | Illimité | - | PREMIUM |
| API calls | 100 | 1 minute | FREE |
| API calls | 1000 | 1 minute | PREMIUM |

**Implémentation** : Spring Security + Redis (rate limiter)

---

## 7. Sécurité

### 7.1 Authentification JWT

**Access Token** :
- Algorithme : HMAC-SHA256
- Durée : 1 heure
- Claims :
  ```json
  {
    "sub": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "displayName": "John Doe",
    "tier": "PREMIUM",
    "roles": ["USER"],
    "iat": 1704225000,
    "exp": 1704228600
  }
  ```

**Refresh Token** :
- Algorithme : UUID v4 sécurisé
- Durée : 7 jours
- Stocké en base (table `refresh_tokens`) avec hash
- Révocable (logout, changement mot de passe)

**Configuration** :
```java
@Configuration
public class JwtConfig {
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.access-token-validity:3600}") // 1 heure
    private long accessTokenValidity;
    
    @Value("${jwt.refresh-token-validity:604800}") // 7 jours
    private long refreshTokenValidity;
    
    // Bean configuration...
}
```

### 7.2 Autorisation RBAC

**Roles** :
- `USER` : Utilisateur standard
- `ADMIN` : Administrateur plateforme
- `MODERATOR` : Modération contenu conversations

**Endpoints** :
- Tous les endpoints `/api/users/*` et `/api/learning-profiles/*` : `ROLE_USER`
- Endpoints admin (futurs) : `ROLE_ADMIN`

**Spring Security Configuration** :
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/register", "/api/auth/login", 
                    "/api/auth/forgot-password", "/api/auth/reset-password",
                    "/api/auth/verify-email").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/api/users/**", "/api/learning-profiles/**")
                    .hasRole("USER")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterBefore(jwtAuthenticationFilter(), 
                UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

### 7.3 Validation des Entrées

1. **Jakarta Bean Validation** :
   - `@Email`, `@NotNull`, `@Size`, `@Min`, `@Max` sur entités
   - Validation automatique avec `@Valid` dans controllers

2. **Sanitization** :
   - Protection XSS : échapper HTML dans `displayName`, `avatarUrl`
   - SQL Injection : utiliser uniquement requêtes paramétrées (JPA)

3. **Rate Limiting** :
   - Bucket4j + Redis pour limiter requêtes par IP/utilisateur
   - Configuration par endpoint (voir section 6.3)

### 7.4 Protection Données Sensibles

1. **Mot de passe** :
   - Hash bcrypt (cost 12)
   - Jamais retourné dans responses API
   - Exclusion avec `@JsonIgnore` sur champ `passwordHash`

2. **Tokens** :
   - Tokens de vérification et reset stockés hashés (SHA-256)
   - Refresh tokens stockés hashés
   - Transmission uniquement via HTTPS

3. **HTTPS Only** :
   - Cookies `Secure`, `HttpOnly`, `SameSite=Strict` (si utilisés)
   - Header `Strict-Transport-Security` (HSTS)

4. **Logs** :
   - Jamais logger les mots de passe, tokens
   - Masquer email dans logs (ex: `u***@example.com`)

---

## 8. Tests

### 8.1 Tests Unitaires

**Frameworks** : JUnit 5, Mockito, AssertJ

**Couverture cible** : 80% minimum

**Exemples** :

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldRegisterNewUser() {
        // Given
        RegisterRequest request = new RegisterRequest(
            "test@example.com", 
            "Password123!", 
            "Test User"
        );
        
        when(userRepository.existsByEmail(request.getEmail())).thenReturn(false);
        when(passwordEncoder.encode(request.getPassword())).thenReturn("hashedPassword");
        when(userRepository.save(any(User.class))).thenAnswer(i -> i.getArgument(0));
        
        // When
        UserResponse response = userService.register(request);
        
        // Then
        assertThat(response.getEmail()).isEqualTo("test@example.com");
        assertThat(response.getDisplayName()).isEqualTo("Test User");
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void shouldThrowExceptionWhenEmailAlreadyExists() {
        // Given
        RegisterRequest request = new RegisterRequest(
            "existing@example.com", 
            "Password123!", 
            "Test User"
        );
        
        when(userRepository.existsByEmail(request.getEmail())).thenReturn(true);
        
        // When / Then
        assertThatThrownBy(() -> userService.register(request))
            .isInstanceOf(EmailAlreadyExistsException.class)
            .hasMessage("Email already registered");
    }
}
```

### 8.2 Tests d'Intégration

**Frameworks** : Spring Boot Test, Testcontainers, MockMVC

**Configuration** :
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@ActiveProfiles("test")
class AuthControllerIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
        .withDatabaseName("wespeak_test")
        .withUsername("test")
        .withPassword("test");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldRegisterAndLoginUser() {
        // Register
        RegisterRequest registerRequest = new RegisterRequest(
            "integration@test.com",
            "SecureP@ss123",
            "Integration Test"
        );
        
        ResponseEntity<AuthResponse> registerResponse = restTemplate.postForEntity(
            "/api/auth/register",
            registerRequest,
            AuthResponse.class
        );
        
        assertThat(registerResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(registerResponse.getBody().getAccessToken()).isNotBlank();
        
        // Login
        LoginRequest loginRequest = new LoginRequest(
            "integration@test.com",
            "SecureP@ss123"
        );
        
        ResponseEntity<AuthResponse> loginResponse = restTemplate.postForEntity(
            "/api/auth/login",
            loginRequest,
            AuthResponse.class
        );
        
        assertThat(loginResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(loginResponse.getBody().getUserId()).isEqualTo(
            registerResponse.getBody().getUserId()
        );
    }
}
```

### 8.3 Tests de Charge

**Outils** : Gatling, JMeter

**Scénarios** :

1. **Login** :
   - 100 users concurrents
   - Ramp-up : 10 secondes
   - Durée : 60 secondes
   - Cible : < 200ms p95, < 1% erreurs

2. **Register** :
   - 50 users concurrents
   - Ramp-up : 10 secondes
   - Durée : 30 secondes
   - Cible : < 500ms p95, < 1% erreurs

3. **Get Profile** :
   - 200 users concurrents
   - Ramp-up : 20 secondes
   - Durée : 120 secondes
   - Cible : < 100ms p95 (avec cache), < 1% erreurs

**Exemple Gatling** :
```scala
class AuthLoadTest extends Simulation {
  
  val httpProtocol = http
    .baseUrl("http://localhost:8080")
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")
  
  val scn = scenario("Auth Load Test")
    .exec(http("Register")
      .post("/api/auth/register")
      .body(StringBody("""{"email":"user${userId}@test.com","password":"Test123!","displayName":"User ${userId}"}"""))
      .check(status.is(201))
      .check(jsonPath("$.accessToken").saveAs("token"))
    )
    .pause(2)
    .exec(http("Get Profile")
      .get("/api/users/me")
      .header("Authorization", "Bearer ${token}")
      .check(status.is(200))
    )
  
  setUp(
    scn.inject(rampUsers(100).during(10.seconds))
  ).protocols(httpProtocol)
}
```

---

## 9. Monitoring et Logs

### 9.1 Métriques à Tracker

**Spring Boot Actuator + Micrometer + Prometheus**

**Métriques métier** :
- `wespeak.auth.registrations.total` (Counter) : Nombre total d'inscriptions
- `wespeak.auth.logins.total` (Counter) : Nombre total de connexions
- `wespeak.auth.logins.failed.total` (Counter) : Nombre d'échecs de connexion
- `wespeak.auth.password_resets.total` (Counter) : Demandes de reset
- `wespeak.auth.email_verifications.total` (Counter) : Vérifications email
- `wespeak.auth.profiles.created.total` (Counter) : Profils d'apprentissage créés
- `wespeak.auth.subscriptions.premium.total` (Gauge) : Nombre abonnés premium

**Métriques techniques** :
- `http.server.requests` (Timer) : Latence endpoints
- `jvm.memory.used` (Gauge) : Mémoire JVM
- `hikaricp.connections.active` (Gauge) : Connexions DB actives
- `cache.gets` (Counter) : Hits/Miss cache Redis

**Configuration** :
```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
management.metrics.export.prometheus.enabled=true
management.metrics.tags.application=auth-service
management.metrics.tags.environment=${ENVIRONMENT:dev}
```

### 9.2 Logs Applicatifs

**Format** : JSON structuré (Logback + Logstash encoder)

**Niveaux** :
- `ERROR` : Erreurs critiques (exceptions non gérées, échecs Kafka)
- `WARN` : Situations anormales (tentatives login échouées répétées, tokens expirés)
- `INFO` : Événements importants (inscription, login, changement subscription)
- `DEBUG` : Détails techniques (requêtes SQL, appels Kafka) - dev/test uniquement

**Exemples** :
```json
{
  "timestamp": "2025-01-02T21:45:00.123Z",
  "level": "INFO",
  "logger": "org.wespeak.auth.service.AuthService",
  "message": "User registered successfully",
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "email": "u***@example.com",
  "subscriptionTier": "FREE",
  "traceId": "a1b2c3d4e5f67890",
  "spanId": "1234567890abcdef"
}
```

**Configuration** :
```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeContext>true</includeContext>
            <includeMdc>true</includeMdc>
            <includeStructuredArguments>true</includeStructuredArguments>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
    
    <logger name="org.wespeak" level="DEBUG" additivity="false">
        <appender-ref ref="CONSOLE" />
    </logger>
</configuration>
```

### 9.3 Alertes

**Prometheus Alertmanager**

**Règles d'alerte** :

```yaml
groups:
  - name: auth-service
    rules:
      - alert: HighLoginFailureRate
        expr: rate(wespeak_auth_logins_failed_total[5m]) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High login failure rate detected"
          description: "More than 10 failed logins per second in the last 5 minutes"
      
      - alert: DatabaseConnectionPoolExhausted
        expr: hikaricp_connections_active >= hikaricp_connections_max * 0.9
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database connection pool near exhaustion"
          description: "90% of DB connections are in use"
      
      - alert: ServiceDown
        expr: up{job="auth-service"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Auth service is down"
          description: "The auth-service instance is unreachable"
      
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_server_requests_seconds_bucket{uri="/api/auth/login"}[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time on login endpoint"
          description: "P95 latency on /api/auth/login is above 1 second"
```

---

## 10. Configuration

### 10.1 Variables d'Environnement

**application.properties** :

```properties
# Application
spring.application.name=auth-service
server.port=8080

# Database
spring.datasource.url=jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:wespeak_auth}
spring.datasource.username=${DB_USERNAME:postgres}
spring.datasource.password=${DB_PASSWORD:postgres}
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=${DB_DDL_AUTO:validate}
spring.jpa.show-sql=${DB_SHOW_SQL:false}
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true

# Redis
spring.data.redis.host=${REDIS_HOST:localhost}
spring.data.redis.port=${REDIS_PORT:6379}
spring.data.redis.password=${REDIS_PASSWORD:}
spring.data.redis.timeout=${REDIS_TIMEOUT:2000}

# Kafka
spring.kafka.bootstrap-servers=${KAFKA_BROKERS:localhost:9092}
spring.cloud.stream.kafka.binder.brokers=${KAFKA_BROKERS:localhost:9092}

# Spring Cloud Stream
spring.cloud.function.definition=
spring.cloud.stream.bindings.userEvents-out-0.destination=user.events
spring.cloud.stream.bindings.userEvents-out-0.content-type=application/json
spring.cloud.stream.kafka.bindings.userEvents-out-0.producer.configuration.key.serializer=org.apache.kafka.common.serialization.StringSerializer

# JWT
jwt.secret=${JWT_SECRET:changeme-this-is-not-secure}
jwt.access-token-validity=${JWT_ACCESS_TOKEN_VALIDITY:3600}
jwt.refresh-token-validity=${JWT_REFRESH_TOKEN_VALIDITY:604800}

# Email
spring.mail.host=${SMTP_HOST:smtp.gmail.com}
spring.mail.port=${SMTP_PORT:587}
spring.mail.username=${SMTP_USERNAME:}
spring.mail.password=${SMTP_PASSWORD:}
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

# Application URLs
app.frontend.url=${FRONTEND_URL:http://localhost:4200}
app.backend.url=${BACKEND_URL:http://localhost:8080}

# Security
app.security.password.min-length=${PASSWORD_MIN_LENGTH:8}
app.security.email-verification.token-validity=${EMAIL_VERIFICATION_TOKEN_VALIDITY:86400}
app.security.password-reset.token-validity=${PASSWORD_RESET_TOKEN_VALIDITY:3600}

# Rate Limiting
app.rate-limit.login.attempts=${RATE_LIMIT_LOGIN_ATTEMPTS:5}
app.rate-limit.login.window=${RATE_LIMIT_LOGIN_WINDOW:900}
app.rate-limit.register.attempts=${RATE_LIMIT_REGISTER_ATTEMPTS:3}
app.rate-limit.register.window=${RATE_LIMIT_REGISTER_WINDOW:3600}

# Actuator
management.endpoints.web.exposure.include=health,metrics,prometheus
management.endpoint.health.show-details=when-authorized
management.metrics.export.prometheus.enabled=true
```

### 10.2 Configuration par Environnement

**application-dev.properties** :
```properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
logging.level.org.wespeak=DEBUG
```

**application-test.properties** :
```properties
spring.jpa.hibernate.ddl-auto=create-drop
spring.cloud.stream.enabled=false
spring.cloud.function.definition=
```

**application-prod.properties** :
```properties
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
logging.level.org.wespeak=INFO
management.endpoint.health.show-details=never
```

---

## 11. Migration et Déploiement

### 11.1 Stratégie de Migration Base de Données

**Outil** : Flyway ou Liquibase

**Flyway Configuration** :
```properties
spring.flyway.enabled=true
spring.flyway.baseline-on-migrate=true
spring.flyway.locations=classpath:db/migration
```

**Exemple Migration** :
```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    display_name VARCHAR(100) NOT NULL,
    avatar_url VARCHAR(500),
    date_of_birth DATE,
    country CHAR(2),
    ui_language_code VARCHAR(10) NOT NULL DEFAULT 'en',
    subscription_tier VARCHAR(20) NOT NULL DEFAULT 'FREE',
    subscription_expires_at TIMESTAMP,
    email_verified BOOLEAN NOT NULL DEFAULT FALSE,
    email_verification_token VARCHAR(500),
    email_verification_token_expires_at TIMESTAMP,
    password_reset_token VARCHAR(500),
    password_reset_token_expires_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    last_login_at TIMESTAMP
);

CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_user_subscription ON users(subscription_tier, subscription_expires_at);

-- V2__create_learning_profiles_table.sql
CREATE TABLE learning_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    native_language_code VARCHAR(10) NOT NULL,
    target_language_code VARCHAR(10) NOT NULL,
    current_level VARCHAR(5) NOT NULL,
    assessed_level VARCHAR(5),
    goal VARCHAR(20) NOT NULL,
    preferred_accent VARCHAR(10),
    weekly_goal_minutes INTEGER NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_lp_user ON learning_profiles(user_id);
CREATE INDEX idx_lp_active ON learning_profiles(user_id, target_language_code, is_active);
CREATE INDEX idx_lp_level ON learning_profiles(target_language_code, current_level);
```

### 11.2 Stratégie de Déploiement

**Blue-Green Deployment** :

1. Déployer nouvelle version (green)
2. Tests de santé automatiques
3. Switch traffic progressif (10% → 50% → 100%)
4. Rollback automatique si erreurs détectées
5. Conserver ancienne version (blue) 24h

**Kubernetes Configuration** :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
  labels:
    app: auth-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-service
  template:
    metadata:
      labels:
        app: auth-service
    spec:
      containers:
      - name: auth-service
        image: wespeak/auth-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: auth-config
              key: db.host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: auth-secrets
              key: db.password
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: auth-secrets
              key: jwt.secret
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  selector:
    app: auth-service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

### 11.3 Checklist de Déploiement

**Pré-déploiement** :
- [ ] Tests unitaires et d'intégration passent
- [ ] Spotless appliqué (code formaté)
- [ ] Migrations de base de données testées
- [ ] Variables d'environnement configurées
- [ ] Secrets Kubernetes créés/mis à jour
- [ ] Revue de code approuvée

**Déploiement** :
- [ ] Migrations de base de données exécutées
- [ ] Image Docker buildée et pushée
- [ ] Déploiement Kubernetes appliqué
- [ ] Health checks validés
- [ ] Métriques Prometheus visibles
- [ ] Logs accessibles dans ELK

**Post-déploiement** :
- [ ] Tests de smoke (login, register, get profile)
- [ ] Vérification événements Kafka publiés
- [ ] Monitoring actif (alertes configurées)
- [ ] Performance acceptable (latence, taux d'erreur)
- [ ] Rollback plan prêt

---

## Annexes

### A. Exemples de Requêtes cURL

```bash
# Register
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "password": "SecureP@ss123",
    "displayName": "New User",
    "uiLanguageCode": "en",
    "nativeLanguageCode": "fr",
    "targetLanguageCode": "en",
    "currentLevel": "A2",
    "goal": "WORK",
    "weeklyGoalMinutes": 120
  }'

# Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "password": "SecureP@ss123"
  }'

# Get User Profile
curl -X GET http://localhost:8080/api/users/me \
  -H "Authorization: Bearer <access_token>"

# Create Learning Profile
curl -X POST http://localhost:8080/api/learning-profiles \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "nativeLanguageCode": "fr",
    "targetLanguageCode": "es",
    "currentLevel": "A1",
    "goal": "TRAVEL",
    "weeklyGoalMinutes": 60
  }'
```

### B. Schéma CloudEvent Complet

```json
{
  "specversion": "1.0",
  "type": "org.wespeak.user.registered",
  "source": "auth-service",
  "id": "<UUID>",
  "time": "<ISO8601 timestamp>",
  "datacontenttype": "application/json",
  "dataschema": "https://wespeak.com/schemas/user-registered-v1.json",
  "data": {
    "userId": "<UUID>",
    "email": "<string>",
    "displayName": "<string>",
    "uiLanguageCode": "<string>",
    "subscriptionTier": "FREE|PREMIUM|ENTERPRISE",
    "learningProfiles": [
      {
        "id": "<UUID>",
        "targetLanguageCode": "<string>",
        "nativeLanguageCode": "<string>",
        "currentLevel": "A1|A2|B1|B2|C1|C2",
        "goal": "WORK|TRAVEL|STUDIES|PERSONAL|OTHER"
      }
    ],
    "createdAt": "<ISO8601 timestamp>"
  }
}
```

---

**Document créé le** : 2025-01-02  
**Version** : 1.0  
**Auteur** : WeSpeak Product Owner AI  
**Dernière mise à jour** : 2025-01-02
