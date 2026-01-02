# WeSpeak - Spécifications Techniques

Ce répertoire contient toutes les spécifications techniques détaillées du projet WeSpeak.

---

## 📁 Structure

```
specs/
├── README.md (ce fichier)
├── SPECIFICATIONS_SUMMARY.md ⭐ (Document principal - 1109 lignes)
└── services/
    ├── 01-auth-service.md ✅ (Complet - 1275 lignes)
    ├── 02-lesson-service.md (Voir SUMMARY)
    ├── 03-conversation-service.md (Voir SUMMARY)
    ├── 04-feedback-service.md (Voir SUMMARY)
    ├── 05-gamification-service.md (Voir SUMMARY)
    ├── 06-recommendation-service.md (Voir SUMMARY)
    └── 07-api-gateway.md (Voir SUMMARY)
```

---

## 🎯 Documents Principaux

### ⭐ SPECIFICATIONS_SUMMARY.md
**Le document le plus important** - Résumé exécutif complet de tous les microservices avec :
- Architecture globale
- Modèles de données complets
- Endpoints API documentés
- Événements Kafka avec exemples JSON
- Règles métier détaillées
- Algorithmes (matchmaking, recommandations, XP, etc.)
- Standards de sécurité
- Stratégies de cache
- Quotas par tier

**👉 [Lire SPECIFICATIONS_SUMMARY.md](./SPECIFICATIONS_SUMMARY.md)**

---

## ✅ Services Spécifiés

### 1. Auth Service (COMPLÉTÉ)
**Fichier** : [services/01-auth-service.md](./services/01-auth-service.md)  
**Taille** : 1275 lignes  
**Sections** :
- 1. Vue d'ensemble
- 2. Modèle de données (TypeScript + TypeORM)
- 3. API REST (18 endpoints)
- 4. Événements asynchrones Kafka
- 5. Règles métier
- 6. Performance et scalabilité
- 7. Sécurité (JWT RS256, OAuth, rate limiting)
- 8. Tests (unit, integration, load)
- 9. Monitoring et logs
- 10. Configuration (variables d'env)
- 11. Migration et déploiement
- 12. Checklist de validation

**Responsabilités** :
- Authentification (JWT RS256 + OAuth Google/Facebook)
- Gestion utilisateurs et profils d'apprentissage multi-langues
- Abonnements (Free/Premium/Enterprise)
- Vérification email et récupération mot de passe

---

### 2-7. Autres Services
Les spécifications des autres services (lesson, conversation, feedback, gamification, recommendation, api-gateway) sont détaillées dans **SPECIFICATIONS_SUMMARY.md**.

Chaque service y dispose de :
- Description des responsabilités
- Entités principales avec schémas
- Endpoints API avec exemples request/response
- Événements Kafka publiés/consommés
- Règles métier et algorithmes
- Quotas par tier
- Points techniques importants

---

## 🏗️ Architecture Microservices

| Service | Port | Base de Données | Statut |
|---------|------|----------------|--------|
| auth-service | 3001 | PostgreSQL | ✅ Spécifié |
| lesson-service | 3002 | PostgreSQL | 📋 Dans SUMMARY |
| conversation-service | 3003 | PostgreSQL + Redis | 📋 Dans SUMMARY |
| feedback-service | 3004 | MongoDB + PostgreSQL | 📋 Dans SUMMARY |
| gamification-service | 3005 | PostgreSQL + Redis | 📋 Dans SUMMARY |
| recommendation-service | 3006 | PostgreSQL + Redis | 📋 Dans SUMMARY |
| api-gateway | 3000 | Redis | 📋 Dans SUMMARY |

**Message Queue** : Kafka (tous services)  
**Storage** : S3 (audio, images)

---

## 🔄 Flux d'Événements Kafka

### Topics Principaux

```
user.events (auth → gamification, recommendation, notification)
├── user.registered
├── user.subscription.upgraded
├── user.learning_profile.created
└── user.email.verified

lesson.events (lesson → gamification, recommendation)
├── lesson.started
├── lesson.completed
├── lesson.mastered
└── skill.acquired

conversation.events (conversation → feedback, gamification)
├── conversation.matched
├── conversation.started
├── conversation.completed
└── conversation.rated

feedback.events (feedback → recommendation, notification)
└── feedback.report.generated

gamification.events (gamification → notification)
├── xp.awarded
├── badge.unlocked
├── level.up
└── streak.extended

recommendation.events (recommendation → notification)
├── recommendation.generated
└── recommendation.completed
```

---

## 📊 Comparaison des Services

### Taille et Complexité

| Service | Entités | Endpoints | Events Pub | Events Sub | Complexité |
|---------|---------|-----------|-----------|-----------|------------|
| auth-service | 3 | 18 | 4 | 0 | Moyenne |
| lesson-service | 7 | 15+ | 4 | 2 | Élevée |
| conversation-service | 3 | 10+ | 4 | 0 | Élevée (WebRTC) |
| feedback-service | 4 | 6 | 1 | 1 | Très Élevée (IA) |
| gamification-service | 6 | 12 | 5 | 4 | Moyenne |
| recommendation-service | 4 | 8 | 2 | 4 | Élevée (ML) |
| api-gateway | 0 | 3 | 0 | 0 | Moyenne |

### Technologies Spécialisées

| Service | Tech Spécifique |
|---------|----------------|
| auth-service | Passport.js, bcrypt, OAuth 2.0 |
| lesson-service | Spaced repetition (SM-2) |
| conversation-service | WebRTC, WebSocket, Simple-peer |
| feedback-service | Whisper (STT), spaCy (NLP), GPT-4 (LLM) |
| gamification-service | Leaderboards real-time (Redis) |
| recommendation-service | ML scoring, collaborative filtering |
| api-gateway | Circuit breaker, rate limiting |

---

## 🎓 Concepts Clés

### Déblocage Séquentiel (Lesson Service)
Leçon N+1 débloquée si :
- Score leçon N ≥ 70%
- OU skills requis maîtrisés

### Spaced Repetition (Lesson Service)
Algorithme SM-2 :
- Score <60% → révision 1 jour
- Score 60-79% → révision 3 jours
- Score 80-89% → révision 7 jours
- Score ≥90% → révision 14 jours

### Matchmaking (Conversation Service)
Critères :
1. Même `targetLanguageCode` (obligatoire)
2. Niveau compatible ±1 (A2 ↔ A1, A2, B1)
3. Thème identique
4. Accent préféré (optionnel)
5. Timeout 2 min → élargir critères

### Attribution XP (Gamification Service)
```
xp_earned = base_xp × multiplier

multiplier:
- 70-79%: 1.0×
- 80-89%: 1.25×
- 90-100%: 1.5×

Bonus:
- First completion: +20%
- Perfect score: +50%
- Streak active: +10%
```

### Scoring Feedback (Feedback Service)
5 dimensions (0-100) :
- **Grammar** : % phrases correctes
- **Vocabulary** : richesse lexicale × niveau
- **Fluency** : WPM + pauses + fillers
- **Pronunciation** : comparaison phonétique
- **Comprehension** : pertinence réponses

**Overall** = weighted average (25%, 20%, 25%, 20%, 10%)

---

## 🔐 Standards de Sécurité

### JWT Structure (RS256)
```json
{
  "userId": "uuid",
  "email": "user@example.com",
  "subscriptionTier": "premium",
  "iat": 1736936400,
  "exp": 1736940000
}
```

**Expiration** :
- Access Token : 1 heure
- Refresh Token : 30 jours (rotation)

### Rate Limiting (API Gateway)
| Tier | Req/min | Burst |
|------|---------|-------|
| Anonymous | 20 | 30 |
| Free | 100 | 150 |
| Premium | 500 | 750 |
| Enterprise | 2000 | 3000 |

---

## 📈 Métriques Importantes

### Performance Targets
- Response time p95 : <500ms (lecture)
- Response time p95 : <2s (écriture)
- Availability : >99.9%
- Error rate : <0.1%

### Scalabilité
- 10,000 utilisateurs actifs simultanés
- 1,000 conversations WebRTC simultanées
- 100,000 requêtes/minute (pic)
- 10,000 jobs feedback/jour

---

## 🚀 Utilisation de cette Documentation

### Pour les Développeurs Backend
1. Commencez par [SPECIFICATIONS_SUMMARY.md](./SPECIFICATIONS_SUMMARY.md)
2. Lisez la section de votre service assigné
3. Implémentez les entités (TypeORM)
4. Créez les endpoints API (NestJS controllers)
5. Implémentez les événements Kafka (producers/consumers)
6. Écrivez les tests (unit + integration)

### Pour les Développeurs Frontend
1. Lisez la section "API Gateway" dans SUMMARY
2. Consultez les endpoints de chaque service
3. Utilisez les exemples request/response
4. Intégrez WebSocket pour conversations

### Pour les Architectes
1. Étudiez l'architecture événementielle (topics Kafka)
2. Validez les choix techniques (PostgreSQL, Redis, MongoDB)
3. Revoyez les stratégies de cache
4. Vérifiez les patterns (circuit breaker, rate limiting)

### Pour les Product Owners
1. Comprenez les règles métier de chaque service
2. Validez les quotas Free vs Premium
3. Priorisez les features selon complexité
4. Planifiez les sprints (roadmap dans INDEX)

---

## 📞 Questions / Feedback

Pour toute question sur les spécifications :
1. Consultez d'abord [SPECIFICATIONS_SUMMARY.md](./SPECIFICATIONS_SUMMARY.md)
2. Vérifiez [01-auth-service.md](./services/01-auth-service.md) comme référence
3. Contactez l'équipe product : product@wespeak.com

---

## 🔄 Mises à Jour

**Version actuelle** : 1.0  
**Dernière mise à jour** : 2025-01-01  

### Changelog
- **v1.0** (2025-01-01) : Spécifications initiales complètes
  - Auth Service : fichier complet (1275 lignes)
  - 6 autres services : spécifiés dans SUMMARY (1109 lignes)
  - Architecture événementielle Kafka définie
  - Standards de sécurité établis

---

**Prêt à développer WeSpeak ! 🚀**
