# WeSpeak - Structure de la Documentation

## 📁 Arborescence Complète

```
we-speak/
│
├── 📄 README_SPECIFICATIONS.md          ⭐ COMMENCER ICI (Guide principal)
├── 📄 SPECIFICATIONS_COMPLETE.md        (Résumé final et validation)
├── 📄 SPECIFICATIONS_INDEX.md           (Index central et roadmap)
├── 📄 STRUCTURE.md                      (Ce fichier)
│
├── .github-private/
│   ├── wespeak_specs.md                 (Spécifications initiales)
│   └── agents/
│       └── wespeak-specification-agent.md
│
└── specs/                               📁 RÉPERTOIRE PRINCIPAL
    ├── 📄 README.md                     (Guide d'utilisation)
    ├── ⭐ SPECIFICATIONS_SUMMARY.md     (Document principal - 1109 lignes)
    │                                    (Contient les 6 services: lesson,
    │                                     conversation, feedback, gamification,
    │                                     recommendation, api-gateway)
    │
    └── services/
        ├── ✅ 01-auth-service.md        (Complet - 1275 lignes)
        │
        ├── 02-lesson-service.md         → Voir SPECIFICATIONS_SUMMARY.md
        ├── 03-conversation-service.md   → Voir SPECIFICATIONS_SUMMARY.md
        ├── 04-feedback-service.md       → Voir SPECIFICATIONS_SUMMARY.md
        ├── 05-gamification-service.md   → Voir SPECIFICATIONS_SUMMARY.md
        ├── 06-recommendation-service.md → Voir SPECIFICATIONS_SUMMARY.md
        └── 07-api-gateway.md            → Voir SPECIFICATIONS_SUMMARY.md
```

---

## 📊 Statistiques par Fichier

| Fichier | Lignes | Taille | Contenu |
|---------|--------|--------|---------|
| **README_SPECIFICATIONS.md** | 428 | 12 KB | Guide principal de navigation |
| **SPECIFICATIONS_COMPLETE.md** | 374 | 11 KB | Résumé final et checklist validation |
| **SPECIFICATIONS_INDEX.md** | 329 | 11 KB | Index central et roadmap développement |
| **specs/README.md** | 313 | 8.8 KB | Guide d'utilisation des spécifications |
| **specs/SPECIFICATIONS_SUMMARY.md** ⭐ | 1,109 | 33 KB | **Document principal** (6 services) |
| **specs/services/01-auth-service.md** ✅ | 1,275 | 29 KB | Service complet (référence) |
| **TOTAL** | **3,828** | **~105 KB** | Documentation complète |

---

## 🎯 Points d'Entrée selon Votre Rôle

### 👨‍💻 Développeur Backend
```
1. README_SPECIFICATIONS.md (aperçu)
2. specs/SPECIFICATIONS_SUMMARY.md (votre service)
3. specs/services/01-auth-service.md (référence complète)
```

### 🎨 Développeur Frontend
```
1. SPECIFICATIONS_INDEX.md (architecture)
2. specs/SPECIFICATIONS_SUMMARY.md → Section "API Gateway"
3. specs/SPECIFICATIONS_SUMMARY.md → Endpoints de chaque service
```

### 🏗️ Architecte
```
1. SPECIFICATIONS_INDEX.md (vue d'ensemble)
2. specs/SPECIFICATIONS_SUMMARY.md → Architecture événementielle (Kafka)
3. specs/services/01-auth-service.md → Exemple patterns
```

### 📋 Product Owner
```
1. SPECIFICATIONS_COMPLETE.md (résumé exécutif)
2. specs/SPECIFICATIONS_SUMMARY.md → Règles métier par service
3. SPECIFICATIONS_INDEX.md → Roadmap (16 semaines)
```

### 🧪 QA Engineer
```
1. specs/services/01-auth-service.md → Section "Tests"
2. specs/SPECIFICATIONS_SUMMARY.md → Quotas et règles métier
3. Définir cas de test (unit, integration, charge)
```

### ⚙️ DevOps Engineer
```
1. specs/services/01-auth-service.md → Section "Configuration"
2. specs/SPECIFICATIONS_SUMMARY.md → Infrastructure et monitoring
3. Préparer: Docker, Kubernetes, CI/CD
```

---

## 📚 Contenu par Service (dans SPECIFICATIONS_SUMMARY.md)

### 1. Auth Service ✅
**Fichier dédié** : `specs/services/01-auth-service.md`  
**Sections** : 12 sections complètes (1275 lignes)
- Vue d'ensemble
- Modèle de données (User, LearningProfile, OAuthProvider)
- API REST (18 endpoints)
- Événements Kafka (4 events)
- Règles métier
- Performance et scalabilité
- Sécurité (JWT RS256, OAuth, rate limiting)
- Tests (unit, integration, load)
- Monitoring et logs
- Configuration
- Migration et déploiement
- Checklist validation

### 2. Lesson Service
**Dans** : `specs/SPECIFICATIONS_SUMMARY.md` (Section 2)  
**Couverture** :
- Responsabilités
- 7 entités (Course, Unit, Lesson, Exercise, Skill, LessonProgress, UserSkill)
- Types d'exercices (MCQ, fill_gap, speaking, listening, translate, match_pairs)
- Règles métier (déblocage séquentiel, XP, spaced repetition)
- Endpoints API (15+)
- Événements Kafka (4 events)
- Quotas Free vs Premium

### 3. Conversation Service
**Dans** : `specs/SPECIFICATIONS_SUMMARY.md` (Section 3)  
**Couverture** :
- Matchmaking intelligent (algorithme détaillé)
- WebRTC signaling (WebSocket events)
- Topics de conversation
- Mode tandem
- Enregistrement audio
- Événements Kafka (4 events)

### 4. Feedback Service
**Dans** : `specs/SPECIFICATIONS_SUMMARY.md` (Section 4)  
**Couverture** :
- Pipeline STT → NLP → Scoring → LLM
- 5 scores (Grammar, Vocabulary, Fluency, Pronunciation, Comprehension)
- Catégories d'erreurs (4 types)
- Entités (ConversationTranscript, FeedbackReport, ErrorInstance)
- Analyse asynchrone (Kafka)

### 5. Gamification Service
**Dans** : `specs/SPECIFICATIONS_SUMMARY.md` (Section 5)  
**Couverture** :
- Système XP et niveaux (formule: 100 × N²)
- Badges (150+ types)
- Streaks (règles et freeze)
- Défis quotidiens/hebdomadaires
- Leaderboards (global, langue, niveau, amis)
- Événements Kafka (5 events)

### 6. Recommendation Service
**Dans** : `specs/SPECIFICATIONS_SUMMARY.md` (Section 6)  
**Couverture** :
- Algorithmes ML (scoring multi-critères)
- Types de recommandations (lesson, conversation, skill_practice, review)
- Moteur de recommandation (6 étapes)
- Cache pré-calculé (Redis)
- Événements Kafka (2 events)

### 7. API Gateway
**Dans** : `specs/SPECIFICATIONS_SUMMARY.md` (Section 7)  
**Couverture** :
- Routing vers 6 microservices
- Middleware chain (10 étapes)
- Rate limiting par tier
- Circuit breaker pattern
- JWT validation centralisée
- Logging structuré

---

## 🔄 Architecture Événementielle (Kafka)

Tous les détails dans `specs/SPECIFICATIONS_SUMMARY.md` - Section "Architecture Événementielle"

### Topics Kafka (6)
```
1. user.events (auth → gamification, recommendation)
2. lesson.events (lesson → gamification, recommendation)
3. conversation.events (conversation → feedback, gamification)
4. feedback.events (feedback → recommendation, notification)
5. gamification.events (gamification → notification)
6. recommendation.events (recommendation → notification)
```

**Partitioning Key** : `userId` (tous les topics)  
**Retention** : 30 jours (90 jours pour feedback)  
**Format** : JSON standard avec version, timestamp, payload, metadata

---

## 🔐 Standards de Sécurité

Détails dans `specs/services/01-auth-service.md` - Section 7 + `SPECIFICATIONS_SUMMARY.md`

- **JWT** : RS256 (asymmetric)
- **Access Token** : 1 heure
- **Refresh Token** : 30 jours (rotation)
- **OAuth** : Google + Facebook
- **Rate Limiting** : Par tier (20-2000 req/min)
- **Validation** : class-validator
- **Encryption** : bcrypt (passwords), AES (data at rest)

---

## 📈 Métriques et Monitoring

Détails dans chaque service (Section 9) + `SPECIFICATIONS_SUMMARY.md`

- **Métriques** : Prometheus
- **Logs** : ELK Stack (JSON structuré)
- **Tracing** : Jaeger
- **Dashboards** : Grafana
- **Alerting** : Prometheus Alertmanager

**Alertes Standards** :
- Error rate >5% → Critical
- Response time p95 >2s → Warning
- Database connections <10% → Critical

---

## 🚀 Roadmap Développement

Détails dans `SPECIFICATIONS_INDEX.md` + `SPECIFICATIONS_COMPLETE.md`

### Phase 1 : MVP Backend (8 semaines)
- Semaines 1-2 : auth + gateway
- Semaines 3-4 : lesson
- Semaines 5-6 : conversation
- Semaine 7 : feedback
- Semaine 8 : gamification

### Phase 2 : Features Avancées (4 semaines)
- Semaine 9 : recommendation
- Semaine 10 : Spaced repetition
- Semaine 11 : Défis
- Semaine 12 : Leaderboards + tandem

### Phase 3 : IA Avancée (4 semaines)
- Semaine 13 : NLP complet
- Semaine 14 : Pronunciation analysis
- Semaine 15 : Recommandations ML
- Semaine 16 : A/B testing

**Total** : 16 semaines (4 mois)

---

## ✅ Checklist Utilisation Documentation

### Pour Commencer (5 min)
- [ ] Lire `README_SPECIFICATIONS.md` (guide principal)
- [ ] Parcourir `SPECIFICATIONS_COMPLETE.md` (résumé exécutif)

### Pour Comprendre l'Architecture (15 min)
- [ ] Lire `SPECIFICATIONS_INDEX.md` (vue d'ensemble)
- [ ] Consulter architecture événementielle dans `SPECIFICATIONS_SUMMARY.md`

### Pour Développer un Service (30-60 min)
- [ ] Lire section service dans `SPECIFICATIONS_SUMMARY.md`
- [ ] Étudier `specs/services/01-auth-service.md` comme référence
- [ ] Implémenter : Entités → API → Events → Tests

### Pour Intégrer Frontend (20 min)
- [ ] Section API Gateway dans `SPECIFICATIONS_SUMMARY.md`
- [ ] Endpoints de chaque service avec exemples JSON
- [ ] WebSocket events (conversation)

### Pour Planifier Sprints (10 min)
- [ ] Roadmap dans `SPECIFICATIONS_INDEX.md`
- [ ] Estimation complexité par service
- [ ] Dépendances entre services

---

## 🎯 FAQ

### Q: Où trouver les modèles de données ?
**A:** `specs/SPECIFICATIONS_SUMMARY.md` - Section 2 de chaque service  
OU `specs/services/01-auth-service.md` - Section 2 (référence complète)

### Q: Où sont les exemples d'API requests/responses ?
**A:** `specs/SPECIFICATIONS_SUMMARY.md` - Section 3 de chaque service  
Exemples JSON complets pour chaque endpoint

### Q: Comment sont gérés les événements Kafka ?
**A:** `specs/SPECIFICATIONS_SUMMARY.md` - Section "Architecture Événementielle"  
Format JSON standard + liste complète des events par topic

### Q: Quelles sont les règles de déblocage des leçons ?
**A:** `specs/SPECIFICATIONS_SUMMARY.md` - Section 2 (Lesson Service) → Règles Métier  
Score ≥70% OU skills requis maîtrisés

### Q: Comment fonctionne le système XP ?
**A:** `specs/SPECIFICATIONS_SUMMARY.md` - Section 5 (Gamification Service)  
Formule : `xp_earned = base_xp × multiplier`  
Multiplier : 1.0× (70-79%), 1.25× (80-89%), 1.5× (90-100%)

### Q: Où trouver la configuration (variables d'env) ?
**A:** `specs/services/01-auth-service.md` - Section 10 (référence complète)  
Ou section Configuration de chaque service dans SUMMARY

---

## 📞 Support

Questions sur les spécifications :
1. Consultez `specs/SPECIFICATIONS_SUMMARY.md` ⭐
2. Référez-vous à `specs/services/01-auth-service.md` (exemple complet)
3. Contactez : product@wespeak.com

---

**Version** : 1.0  
**Date** : 2025-01-01  
**Statut** : ✅ COMPLET
