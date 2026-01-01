# WeSpeak - Index des Spécifications Applicatives

## 📚 Documentation Technique Complète

Ce document sert d'index central pour toute la documentation technique du projet WeSpeak.

---

## 🎯 Objectif du Projet

**WeSpeak** est une plateforme innovante d'apprentissage des langues combinant :
1. **Structure pédagogique progressive** (inspiration Duolingo)
2. **Pratique orale authentique** via conversations 1v1 (inspiration SpeakDuo)
3. **Coaching intelligent par IA** avec feedback personnalisé
4. **Gamification immersive** (XP, badges, streaks, défis)

---

## 📁 Organisation de la Documentation

```
we-speak/
├── .github-private/
│   └── wespeak_specs.md (Spécifications initiales)
├── specs/
│   ├── SPECIFICATIONS_SUMMARY.md ⭐ (Document principal - 1109 lignes)
│   └── services/
│       ├── 01-auth-service.md ✅ (Complet - 1275 lignes)
│       ├── 02-lesson-service.md (Voir SUMMARY)
│       ├── 03-conversation-service.md (Voir SUMMARY)
│       ├── 04-feedback-service.md (Voir SUMMARY)
│       ├── 05-gamification-service.md (Voir SUMMARY)
│       ├── 06-recommendation-service.md (Voir SUMMARY)
│       └── 07-api-gateway.md (Voir SUMMARY)
└── SPECIFICATIONS_INDEX.md (Ce fichier)
```

---

## 📖 Documents Principaux

### 1. ⭐ SPECIFICATIONS_SUMMARY.md
**Chemin** : `/specs/SPECIFICATIONS_SUMMARY.md`  
**Taille** : 33 KB, 1109 lignes  
**Statut** : ✅ Complété

**Contenu** :
- Vue d'ensemble architecture globale
- Spécifications détaillées des 7 microservices
- Modèles de données complets
- Endpoints API avec exemples
- Événements Kafka avec schémas JSON
- Règles métier détaillées
- Algorithmes (matchmaking, recommandations, scoring)
- Standards de sécurité (JWT RS256)
- Quotas et rate limiting par tier
- Stratégies de cache Redis
- Métriques Prometheus

### 2. ✅ Auth Service (Complet)
**Chemin** : `/specs/services/01-auth-service.md`  
**Taille** : 29 KB, 1275 lignes  
**Statut** : ✅ Complété

**Sections** :
1. Vue d'ensemble
2. Modèle de données (User, LearningProfile, OAuthProvider)
3. API REST (18 endpoints documentés)
4. Événements asynchrones Kafka
5. Règles métier
6. Performance et scalabilité
7. Sécurité (JWT RS256, bcrypt, rate limiting)
8. Tests (unit, integration, load)
9. Monitoring et logs
10. Configuration
11. Migration et déploiement
12. Checklist de validation

---

## 🏗️ Architecture Microservices

### Services Backend

| Service | Port | Responsabilités | Statut Specs |
|---------|------|----------------|--------------|
| **auth-service** | 3001 | Auth, users, learning profiles, subscriptions | ✅ Complété |
| **lesson-service** | 3002 | Curriculum, progression, skills, spaced repetition | 📋 Dans SUMMARY |
| **conversation-service** | 3003 | Matchmaking, WebRTC, topics, enregistrements | 📋 Dans SUMMARY |
| **feedback-service** | 3004 | STT, NLP, analyse erreurs, rapports IA | 📋 Dans SUMMARY |
| **gamification-service** | 3005 | XP, badges, streaks, leaderboards, défis | 📋 Dans SUMMARY |
| **recommendation-service** | 3006 | Recommandations personnalisées, learning path | 📋 Dans SUMMARY |
| **api-gateway** | 3000 | Point d'entrée, routing, rate limiting, circuit breaker | 📋 Dans SUMMARY |

### Technologies

**Backend** :
- Node.js 20+ LTS
- TypeScript 5+
- NestJS 10+
- PostgreSQL 15+ (relationnel)
- MongoDB 7+ (transcripts, feedback)
- Redis 7+ (cache, sessions, rate limiting)
- Kafka (événements asynchrones)
- S3 (audio, média)

**IA/ML** :
- Whisper (STT)
- GPT-4 / Claude (LLM feedback)
- spaCy / Transformers (NLP)
- ElevenLabs (TTS)

**Frontend** :
- Angular 17+ avec SSR
- WebRTC (Simple-peer / PeerJS)
- NgRx ou Signals (state management)
- Angular Material / Tailwind CSS

---

## 📊 Résumé des Spécifications par Service

### 1. Auth Service ✅
- **Entités** : User, LearningProfile, OAuthProvider
- **Authentification** : JWT RS256 + Refresh tokens (rotation)
- **OAuth** : Google, Facebook
- **Rate Limiting** : 5 login/15min, 3 register/heure
- **Events Kafka** : user.registered, user.subscription.upgraded

### 2. Lesson Service
- **Entités** : Course, Unit, Lesson, Exercise, Skill, LessonProgress, UserSkill
- **Types Exercices** : MCQ, fill_gap, listen_and_repeat, speak_sentence, translate, match_pairs
- **Déblocage** : Séquentiel (score ≥70%)
- **XP** : 1.0× (70-79%), 1.25× (80-89%), 1.5× (90-100%)
- **Spaced Repetition** : Algorithme SM-2
- **Quotas Free** : 5 leçons/jour, A1-A2 seulement

### 3. Conversation Service
- **Matchmaking** : Langue + Niveau (±1) + Thème
- **WebRTC** : Signaling WebSocket
- **Prompts** : Dynamiques toutes les 3-5 min
- **Quotas Free** : 3 conversations/semaine, 15 min max
- **Mode Tandem** : Échange linguistique bidirectionnel

### 4. Feedback Service
- **Pipeline** : Audio → STT → NLP → Pronunciation → Scoring → LLM → Report
- **Scores** : Grammar, Vocabulary, Fluency, Pronunciation, Comprehension
- **Erreurs** : 4 catégories (grammar, vocabulary, pronunciation, usage)
- **Traitement** : Asynchrone (Kafka queue), 2-5 min pour 15 min audio

### 5. Gamification Service
- **XP** : 50-150 (leçon), 100-200 (conversation)
- **Niveaux** : 1-100 (formule: XP = 100 × N²)
- **Badges** : 150+ (milestone, skill, social, streak, special)
- **Streaks** : ≥10 min/jour, freeze 2×/mois (Premium)
- **Défis** : Quotidiens (3) + Hebdomadaires (1)
- **Leaderboards** : Global, langue, niveau, amis

### 6. Recommendation Service
- **Algorithmes** : Score multi-critères (relevance, difficulty, likelihood, engagement)
- **Types** : Lesson, conversation, skill_practice, review
- **Raisons** : skill_gap, error_pattern, level_progression, review_due
- **Cache** : Pré-calcul (TTL 1h)

### 7. API Gateway
- **Routing** : Vers 6 microservices backend
- **Auth** : Validation JWT centralisée
- **Rate Limiting** : Par tier (20-2000 req/min)
- **Circuit Breaker** : Protection services down
- **Logging** : Structured JSON + correlation IDs

---

## 🔄 Architecture Événementielle (Kafka)

### Topics Principaux

| Topic | Producteur | Consommateurs | Key |
|-------|-----------|---------------|-----|
| `user.events` | auth | gamification, recommendation, notification | userId |
| `lesson.events` | lesson | gamification, recommendation, analytics | userId |
| `conversation.events` | conversation | feedback, gamification, analytics | userId |
| `feedback.events` | feedback | recommendation, notification, analytics | userId |
| `gamification.events` | gamification | notification, analytics | userId |
| `recommendation.events` | recommendation | notification, analytics | userId |

### Format Standard Événement
```json
{
  "eventType": "service.action",
  "version": "1.0",
  "timestamp": "ISO 8601",
  "payload": { /* data */ },
  "metadata": {
    "correlationId": "uuid",
    "source": "service-name",
    "userId": "uuid"
  }
}
```

---

## 🔐 Standards de Sécurité

### JWT (RS256)
- **Access Token** : 1 heure
- **Refresh Token** : 30 jours (rotation automatique)
- **Payload** : userId, email, subscriptionTier, iat, exp
- **Validation** : Clé publique distribuée par auth-service

### Rate Limiting
| Tier | Req/min | Burst |
|------|---------|-------|
| Anonymous | 20 | 30 |
| Free | 100 | 150 |
| Premium | 500 | 750 |
| Enterprise | 2000 | 3000 |

---

## 📈 Métriques et Monitoring

### Stack
- **Métriques** : Prometheus
- **Logs** : ELK Stack (Elasticsearch, Logstash, Kibana)
- **Tracing** : Jaeger
- **Dashboards** : Grafana
- **Alerting** : Prometheus Alertmanager

### Alertes Standards
- Error rate >5% → Critical
- Response time p95 >2s → Warning
- Database connections <10% → Critical
- Service down → Critical

---

## 🧪 Stratégie de Tests

### Par Service
- **Tests Unitaires** : >80% coverage (Jest)
- **Tests d'Intégration** : Endpoints API (Supertest)
- **Tests de Charge** : k6, Artillery
- **Tests E2E** : Playwright, Cypress

### Scénarios de Charge
- Login burst : 5000 req/s
- Lesson completion : 10000 req/s
- Conversation matchmaking : 2000 req/min
- Feedback analysis : 500 jobs/min

---

## 🚀 Roadmap de Développement

### Phase 1 : MVP Backend (8 semaines)
- [x] Week 1-2 : auth-service + api-gateway
- [ ] Week 3-4 : lesson-service (curriculum de base A1)
- [ ] Week 5-6 : conversation-service (matchmaking + WebRTC)
- [ ] Week 7 : feedback-service (STT + rapport basique)
- [ ] Week 8 : gamification-service (XP + badges)

### Phase 2 : Features Avancées (4 semaines)
- [ ] recommendation-service
- [ ] Spaced repetition
- [ ] Défis quotidiens/hebdomadaires
- [ ] Leaderboards

### Phase 3 : IA Avancée (4 semaines)
- [ ] Feedback IA détaillé (NLP complet)
- [ ] Recommandations prédictives
- [ ] Analyse pronunciation avancée

---

## 📚 Références Utiles

### Documentation Interne
- [Spécifications Initiales](../.github-private/wespeak_specs.md)
- [SPECIFICATIONS_SUMMARY.md](./specs/SPECIFICATIONS_SUMMARY.md) ⭐
- [Auth Service Complet](./specs/services/01-auth-service.md) ✅

### Technologies Externes
- [NestJS Documentation](https://docs.nestjs.com/)
- [TypeORM](https://typeorm.io/)
- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [Whisper (OpenAI)](https://openai.com/research/whisper)
- [WebRTC](https://webrtc.org/)

---

## ✅ Checklist Globale

### Spécifications
- [x] auth-service (1275 lignes) ✅
- [x] lesson-service (dans SUMMARY) ✅
- [x] conversation-service (dans SUMMARY) ✅
- [x] feedback-service (dans SUMMARY) ✅
- [x] gamification-service (dans SUMMARY) ✅
- [x] recommendation-service (dans SUMMARY) ✅
- [x] api-gateway (dans SUMMARY) ✅

### Documentation Complémentaire
- [ ] OpenAPI/Swagger specs par service
- [ ] Schémas Avro pour Kafka events
- [ ] Architecture Decision Records (ADR)
- [ ] Runbooks opérationnels
- [ ] Guide développeur

### Infrastructure
- [ ] Terraform/CloudFormation scripts
- [ ] Helm charts Kubernetes
- [ ] CI/CD pipelines (GitHub Actions)
- [ ] Monitoring dashboards (Grafana)

---

## 📞 Contact

**Product Owner** : WeSpeak Product Owner AI Agent  
**Email** : product@wespeak.com  
**Repository** : https://github.com/wespeak/wespeak-platform

---

**Dernière mise à jour** : 2025-01-01  
**Version** : 1.0  
**Statut** : ✅ Spécifications Core Complétées
