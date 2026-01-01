# WeSpeak - Résumé des Spécifications Applicatives Détaillées

## 📊 Vue d'Ensemble

Ce document fournit un résumé exécutif des spécifications techniques détaillées pour les 7 microservices de la plateforme WeSpeak.

---

## ✅ État d'Avancement

| Service | Statut | Fichier | Lignes |
|---------|--------|---------|--------|
| auth-service | ✅ **COMPLÉTÉ** | [01-auth-service.md](./services/01-auth-service.md) | 1275 |
| lesson-service | 📋 Spécifié ci-dessous | - | - |
| conversation-service | 📋 Spécifié ci-dessous | - | - |
| feedback-service | 📋 Spécifié ci-dessous | - | - |
| gamification-service | 📋 Spécifié ci-dessous | - | - |
| recommendation-service | 📋 Spécifié ci-dessous | - | - |
| api-gateway | 📋 Spécifié ci-dessous | - | - |

---

## 1. ✅ Auth Service (COMPLÉTÉ)

### Responsabilités
- Authentification (JWT RS256, OAuth Google/Facebook)
- Gestion utilisateurs et profils d'apprentissage multi-langues
- Abonnements (Free/Premium/Enterprise)
- Vérification email et récupération mot de passe

### Entités Clés
- **User** : Compte utilisateur (email, displayName, subscriptionTier)
- **LearningProfile** : Profil par langue cible (nativeLanguageCode, targetLanguageCode, currentLevel)
- **OAuthProvider** : Liens OAuth (Google, Facebook)

### Points Techniques
- JWT RS256 avec refresh tokens (rotation automatique, TTL 30j)
- Rate limiting : 5 tentatives login/15min, 3 registrations/heure
- Redis cache : profiles (TTL 1h), blacklist tokens
- PostgreSQL + indexes optimisés

### Événements Kafka
- `user.registered` → gamification, recommendation, notification
- `user.subscription.upgraded` → gamification, notification
- `user.learning_profile.created` → lesson, recommendation
- `user.email.verified` → gamification (badge)

### Endpoints Principaux
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh-token
POST /api/auth/forgot-password
GET  /api/users/me
POST /api/learning-profiles
```

**Voir fichier complet** : [services/01-auth-service.md](./services/01-auth-service.md)

---

## 2. Lesson Service

### Responsabilités
- Curriculum : Courses → Units → Lessons → Exercises
- Progression utilisateur (scores, déblocage séquentiel)
- Système de skills linguistiques (grammar, vocabulary, pronunciation)
- Spaced repetition (révisions espacées selon algorithme SM-2)

### Entités Clés
- **Course** : Cours par langue/niveau (ex: "English A1")
- **Unit** : Unités thématiques avec critères de déblocage
- **Lesson** : Leçons 15-30min (type: vocab, grammar, speaking, listening)
- **Exercise** : Types variés (MCQ, fill_gap, listen_and_repeat, speak_sentence, translate, match_pairs)
- **Skill** : Compétences linguistiques hiérarchisées
- **LessonProgress** : Statut (not_started, in_progress, completed, mastered), score, attempts
- **UserSkill** : Niveau maîtrise (learning, practiced, mastered), proficiency 0-100

### Règles Métier

**Déblocage Séquentiel** :
- Leçon N+1 débloquée si score leçon N ≥ 70%
- Déblocage alternatif si skills requis maîtrisés

**Attribution XP** :
```
xp_earned = base_xp × multiplier

multiplier selon score:
- 70-79%: 1.0×
- 80-89%: 1.25×
- 90-100%: 1.5×

Bonus:
- First completion: +20%
- Perfect score (100%): +50%
```

**Spaced Repetition (SM-2)** :
- Score <60% → révision dans 1 jour
- Score 60-79% → révision dans 3 jours
- Score 80-89% → révision dans 7 jours
- Score ≥90% → révision dans 14 jours

**Skills Progression** :
- Proficiency 0-30: Learning
- Proficiency 31-70: Practiced
- Proficiency 71-100: Mastered
- +5 à +15 proficiency par exercice correct (selon difficulté)
- -5 proficiency si erreur

### Types d'Exercices

1. **MCQ** : Questions à choix multiples
2. **Fill Gap** : Compléter phrases ("I ___ to school")
3. **Listen & Repeat** : STT validation (minSimilarityScore)
4. **Speak Sentence** : Prononcer phrase avec contexte
5. **Translate** : Traduction source → target
6. **Match Pairs** : Associer mots/images

### Événements Kafka
- `lesson.started` → gamification (track activity)
- `lesson.completed` → gamification (XP), recommendation
- `lesson.mastered` → gamification (badge "Lesson Master")
- `skill.acquired` → gamification, recommendation

### Endpoints Principaux
```
GET  /api/courses?targetLanguageCode={code}&level={level}
GET  /api/lessons/:id
GET  /api/lessons/:id/exercises
POST /api/lessons/:id/start
POST /api/lessons/:id/complete
POST /api/exercises/:id/submit
GET  /api/users/me/skills?targetLanguageCode={code}
GET  /api/progress/overview
GET  /api/progress/reviews (leçons à réviser)
```

### Quotas Tier
| Feature | Free | Premium |
|---------|------|---------|
| Lessons/day | 5 | Illimité |
| Courses access | A1-A2 | Tous niveaux |
| Reviews/day | 3 | Illimité |
| Hints/exercise | 1 | 3 |

---

## 3. Conversation Service

### Responsabilités
- Matchmaking intelligent (langue, niveau, thème, accent)
- Sessions WebRTC 1v1 en temps réel
- Gestion topics de conversation guidés
- Enregistrement audio pour feedback IA
- Mode "tandem" (échange linguistique)

### Entités Clés
- **MatchRequest** : Demande match avec critères (langue, niveau, topic, preferredPartnerLevel)
- **ConversationSession** : Session avec 2 participants, prompts dynamiques, status, recordingUrl
- **ConversationTopic** : Thèmes (daily_life, travel, work) avec prompts (icebreaker, main, follow_up)
- **ConversationPrompt** : Prompts guidés affichés toutes les 3-5 min

### Algorithme de Matchmaking

**Critères obligatoires** :
1. Même `targetLanguageCode`
2. Niveau compatible (± 1 niveau, ex: A2 peut matcher avec A1, A2, B1)
3. Thème identique (restaurant, travel, work, etc.)

**Critères préférentiels** :
- Accent préféré (en-US, en-GB, es-ES, es-MX)
- Éviter re-match récents (derniers 7 jours)
- Score compatibilité basé sur historique

**Timeout et élargissement** :
- Timeout 2 minutes
- Si pas de match : élargir niveau (± 2 niveaux) et accent (any)

### WebRTC Signaling (WebSocket)

**URL** : `ws://conversation-service/ws/conversations/:sessionId`

**Events Client → Server** :
- `join_session` : { userId, sessionId }
- `offer` : { sdp }
- `answer` : { sdp }
- `ice_candidate` : { candidate }
- `leave_session` : { userId }

**Events Server → Client** :
- `partner_joined` : { partnerId, partnerName, partnerAvatar }
- `offer` : { sdp, from }
- `answer` : { sdp, from }
- `ice_candidate` : { candidate, from }
- `partner_left` : { partnerId, reason }
- `session_ended` : { reason }
- `new_prompt` : { promptKey, prompt } (toutes les 3-5 min)

### Règles Métier

**Quotas Tier** :
- **Free** : 3 conversations/semaine, 15 min max par session
- **Premium** : Illimité, 60 min max par session

**Enregistrement** :
- Automatique pour feedback IA
- Consentement requis à l'inscription
- Stockage S3 avec encryption
- Suppression après 90 jours (sauf si favoris)

**Mode Tandem** :
- User A : native français → target anglais
- User B : native anglais → target français
- Chacun parle sa langue cible
- Correction mutuelle encouragée

### Événements Kafka
- `conversation.matched` → gamification, analytics
- `conversation.started` → gamification (track activity)
- `conversation.completed` → feedback (trigger STT), gamification (XP 100-200)
- `conversation.rated` → analytics, recommendation

### Endpoints Principaux
```
POST   /api/conversations/match-request
DELETE /api/conversations/match-request/:id
GET    /api/conversations/topics?targetLanguageCode={code}&level={level}
POST   /api/conversations/sessions/:id/join
POST   /api/conversations/sessions/:id/end
POST   /api/conversations/sessions/:id/rate (1-5 stars)
GET    /api/conversations/history?userId={id}
WS     /ws/conversations/:sessionId (WebSocket)
```

---

## 4. Feedback Service

### Responsabilités
- Transcription audio (STT via Whisper ou Cloud STT)
- Analyse linguistique (NLP : grammar, vocabulary, pronunciation)
- Détection erreurs avec corrections
- Calcul scores par dimension
- Génération rapports personnalisés avec LLM
- Recommandations leçons ciblées

### Entités Clés
- **ConversationTranscript** : Segments transcrits (speaker, startTime, endTime, text, confidence)
- **FeedbackReport** : Scores (overall, grammar, vocabulary, fluency, pronunciation, comprehension), metrics, strengths, errors
- **ErrorInstance** : Erreurs détectées (type, category, severity, originalText, correctedText, explanation, relatedSkill)
- **AnalysisJob** : Queue asynchrone d'analyses (status: pending, processing, completed, failed)

### Pipeline de Traitement

```
1. Audio Upload → S3 (encryption at rest)
2. STT Processing (Whisper/Google STT/Azure)
   → Transcription segmentée par speaker
3. Language Detection per segment
4. NLP Analysis
   → Tokenization (spaCy)
   → POS Tagging
   → Grammar Check (LanguageTool)
   → Vocabulary Analysis (richesse lexicale)
5. Pronunciation Analysis
   → Phoneme comparison avec modèles natifs
   → Accent detection
6. Error Detection & Categorization
7. Score Calculation (0-100 par dimension)
8. LLM Recommendation Generation (GPT-4/Claude)
9. Report Creation (MongoDB)
10. User Notification (Kafka event)
```

**Temps de traitement** : 2-5 minutes pour 15 min de conversation

### Scores Calculés

**Grammar** (0-100) :
```
grammar_score = (phrases_correctes / total_phrases) × 100
```

**Vocabulary** (0-100) :
```
unique_words_ratio = unique_words / total_words
vocabulary_score = unique_words_ratio × 100 × niveau_factor

niveau_factor:
- A1: 0.5
- A2: 0.7
- B1: 0.9
- B2: 1.0
- C1: 1.1
- C2: 1.2
```

**Fluency** (0-100) :
```
fluency_score = WPM_score × 0.5 + pause_score × 0.3 + filler_score × 0.2

WPM_score: normaliser WPM entre 80-160 (natif moyen: 120-150)
pause_score: pénalité si pauses >2s fréquentes
filler_score: pénalité si "um", "uh", "like" >5% des mots
```

**Pronunciation** (0-100) :
- Comparaison phonétique avec modèles natifs
- Détection erreurs phonèmes
- Score accent (proximité avec accent target)

**Comprehension** (0-100) :
- Pertinence réponses aux prompts
- Turn-taking approprié
- Maintien du contexte

**Overall Score** :
```
overall = (grammar × 0.25) + (vocabulary × 0.20) + (fluency × 0.25) + (pronunciation × 0.20) + (comprehension × 0.10)
```

### Catégories d'Erreurs

**Grammar** :
- `verb_tense` : "I go yesterday" → "I went yesterday"
- `article_usage` : "I am student" → "I am a student"
- `subject_verb_agreement` : "He go" → "He goes"
- `preposition` : "I go to school by foot" → "I go to school on foot"
- `word_order` : "I like very much pizza" → "I like pizza very much"

**Vocabulary** :
- `word_choice` : "Big" vs "Large" (contexte)
- `false_friends` : "Actually" ≠ "Actuellement"
- `inappropriate_register` : Formel vs informel
- `collocations` : "Make a photo" → "Take a photo"

**Pronunciation** :
- `phoneme_substitution` : /θ/ → /s/ ("think" → "sink")
- `stress_pattern` : "PREsent" (noun) vs "preSENT" (verb)
- `intonation` : Questions montantes vs affirmatives

**Usage** :
- `idiomatic_expression` : Utilisation idiomes
- `pragmatics` : Contexte social approprié

### Événements Kafka
- **Consomme** : `conversation.completed` (trigger analyse asynchrone)
- **Publie** : `feedback.report.generated` → recommendation, notification, analytics

### Endpoints Principaux
```
POST /api/feedback/analyze (trigger analyse manuelle)
GET  /api/feedback/reports/:sessionId
GET  /api/feedback/transcripts/:id
GET  /api/feedback/user-stats?userId={id}&targetLanguageCode={code}&period={week|month|all}
GET  /api/feedback/error-patterns?userId={id} (erreurs récurrentes)
```

### Règles Métier
- **Analyse asynchrone** : Queue Kafka (topic: `feedback.analysis.requests`)
- **Priorisation** : Premium > Free
- **Timeout** : 5 min max par analyse
- **Retry** : 3 tentatives si échec
- **Anonymisation** : Données sensibles masquées dans transcripts

---

## 5. Gamification Service

### Responsabilités
- Système XP et niveaux (1-100)
- Streaks quotidiennes avec freeze
- Badges et achievements (150+ badges)
- Leaderboards (global, langue, niveau, amis)
- Défis quotidiens et hebdomadaires
- Système de récompenses

### Entités Clés
- **UserGamificationProfile** : totalXP, currentLevel, xpToNextLevel, streaks (current, longest), stats
- **Badge** : code, nameKey, category (milestone, skill, social, streak, special), rarity, unlockCriteria, xpReward
- **Achievement** : Réalisations complexes multi-critères
- **Challenge** : type (daily, weekly, special), goal, rewards, startDate, endDate
- **Leaderboard** : type (global, language, level, friends), period (daily, weekly, monthly, all_time), entries (rank, userId, xp, streak, change)
- **XPTransaction** : Historique gains XP (source, amount, metadata, timestamp)

### Système XP

**Sources XP** :
- Leçon complétée : 50-150 XP (selon score 70-100%)
- Exercice réussi : 10-30 XP
- Conversation complétée : 100-200 XP (selon durée 5-60 min)
- Streak quotidienne maintenue : +10% bonus sur tous les XP du jour
- Badge débloqué : 50-500 XP (selon rareté)
- Défi quotidien : 100-300 XP
- Défi hebdomadaire : 500-1500 XP

**Formule Niveaux** :
```
XP requis pour niveau N = 100 × N²

Exemples:
- Niveau 1 → 2: 100 XP
- Niveau 2 → 3: 400 XP
- Niveau 5 → 6: 2,500 XP
- Niveau 10 → 11: 10,000 XP
- Niveau 50 → 51: 250,000 XP
- Niveau 99 → 100: 990,000 XP
```

**Total XP pour niveau 100** : ~33,500,000 XP (gamification long-terme)

### Types de Badges

**Milestone** (jalon) :
- "First Steps" : Première leçon complétée (50 XP)
- "Dedicated Learner" : 10 leçons complétées (100 XP)
- "Lesson Master" : 100 leçons complétées (500 XP)
- "Conversation Starter" : Première conversation (100 XP)
- "Social Butterfly" : 50 conversations complétées (750 XP)
- "Century Club" : 100 heures d'apprentissage (1000 XP)

**Skill** (compétence) :
- "Grammar Guru" : 20 skills grammar mastered (300 XP)
- "Vocabulary Virtuoso" : 50 skills vocabulary mastered (500 XP)
- "Pronunciation Pro" : 10 skills pronunciation mastered (400 XP)

**Social** :
- "Tandem Partner" : 10 conversations tandem (200 XP)
- "Five Star Conversationalist" : 20 conversations 5★ rating (300 XP)
- "Language Exchange Master" : 3 langues pratiquées en tandem (500 XP)

**Streak** :
- "On Fire" : 7 jours streak (150 XP)
- "Unstoppable" : 30 jours streak (600 XP)
- "Legendary" : 100 jours streak (2000 XP)
- "Epic Journey" : 365 jours streak (5000 XP)

**Special** (événements) :
- "Early Adopter" : Inscription première semaine (250 XP)
- "Beta Tester" : Participation beta (500 XP)
- "Feedback Champion" : 100 feedbacks soumis (300 XP)

### Streaks

**Règles** :
- Compte si activité ≥ 10 min/jour (leçon OU conversation)
- Réinitialisation à minuit (timezone utilisateur)
- **Freeze Streak** (Premium) : 2×/mois pour sauter 1 jour sans perdre streak
- Notification quotidienne si streak à risque (22h sans activité)

**Récompenses Streak** :
- 7 jours : +10% XP bonus pendant 24h
- 30 jours : +15% XP bonus pendant 48h
- 100 jours : +25% XP bonus pendant 7 jours

### Défis

**Défis Quotidiens** (3 par jour, rotation minuit) :
- "Complete 3 lessons today" → 150 XP
- "Have 1 conversation" → 200 XP
- "Practice 30 minutes" → 100 XP
- "Master 2 exercises perfectly" → 120 XP
- "Earn 500 XP today" → 100 XP bonus

**Défis Hebdomadaires** (1 par semaine, rotation lundi) :
- "Complete 15 lessons this week" → 1000 XP + Badge "Weekly Warrior"
- "Have 5 conversations" → 1500 XP
- "Master 3 new skills" → 800 XP
- "Earn 5000 XP this week" → 1200 XP + Badge "XP Champion"

**Défis Spéciaux** (événements) :
- "Valentine's Day Love for Languages" : Février
- "Summer Learning Sprint" : Juillet-Août
- "New Year Resolution" : Janvier

### Leaderboards

**Types** :
- **Global** : Top 100 tous utilisateurs, tous critères
- **Language** : Top 100 par langue cible (en, fr, es, etc.)
- **Level** : Top 50 par niveau CEFR (A1, A2, B1, etc.)
- **Friends** : Classement amis (feature sociale)

**Périodes** :
- Daily (reset minuit UTC)
- Weekly (reset lundi 00:00 UTC)
- Monthly (reset 1er du mois)
- All-time (depuis inscription)

**Métriques classement** :
- XP total (période)
- Streak actuel
- Leçons complétées
- Conversations complétées

**Mise à jour** : Toutes les heures (cache Redis)

### Événements Kafka
- **Consomme** :
  - `lesson.started` → Track activity pour streak
  - `lesson.completed` → Award XP, check badges
  - `conversation.completed` → Award XP, check badges
  - `skill.acquired` → Check skill badges
  - `user.registered` → Créer profil gamification initial
  
- **Publie** :
  - `xp.awarded` → analytics, notification
  - `badge.unlocked` → notification, feed
  - `level.up` → notification, analytics
  - `streak.extended` → notification
  - `streak.lost` → notification
  - `challenge.completed` → notification

### Endpoints Principaux
```
GET  /api/gamification/profile
GET  /api/gamification/profile/stats?period={week|month|all}
GET  /api/gamification/badges (catalogue)
GET  /api/gamification/profile/badges (badges utilisateur)
GET  /api/gamification/challenges?type={daily|weekly}
POST /api/gamification/challenges/:id/claim-reward
GET  /api/gamification/leaderboard?type={global|language|friends}&period={daily|weekly}
GET  /api/gamification/xp-history?limit=50
```

---

## 6. Recommendation Service

### Responsabilités
- Recommandations leçons personnalisées
- Suggestions topics de conversation
- Identification lacunes (skills manquants)
- Parcours d'apprentissage adaptatif
- Prédiction prochaine meilleure action ("Next Best Action")

### Entités Clés
- **Recommendation** : type (lesson, conversation, skill_practice, review), itemId, reason, priority (0-100), context, status
- **UserLearningState** : État agrégé (niveau, skills maîtrisés, erreurs récurrentes, préférences)
- **SkillGap** : Lacunes identifiées (skillId, importance, detectedFrom)
- **RecommendationFeedback** : Feedback utilisateur (accepted, dismissed, completed)

### Algorithmes

**Score Recommandation Leçon** :
```typescript
function calculateLessonScore(user, lesson): number {
  const relevanceScore = calculateRelevance(user.skills, lesson.requiredSkills); // 0-100
  const difficultyMatch = calculateDifficultyMatch(user.level, lesson.targetLevel); // 0-100
  const completionLikelihood = predictCompletionLikelihood(user.history, lesson); // 0-100
  const engagementPrediction = predictEngagement(user.preferences, lesson.topic); // 0-100
  
  return (
    relevanceScore * 0.4 +
    difficultyMatch * 0.3 +
    completionLikelihood * 0.2 +
    engagementPrediction * 0.1
  );
}

function calculateRelevance(userSkills, requiredSkills): number {
  // Score basé sur skills requis vs acquis
  const missingSkills = requiredSkills.filter(s => !userSkills.includes(s));
  const relevanceRatio = 1 - (missingSkills.length / requiredSkills.length);
  return relevanceRatio * 100;
}

function calculateDifficultyMatch(userLevel, lessonLevel): number {
  // Distance entre niveau user et leçon
  const levelMap = { A1: 1, A2: 2, B1: 3, B2: 4, C1: 5, C2: 6 };
  const distance = Math.abs(levelMap[userLevel] - levelMap[lessonLevel]);
  
  if (distance === 0) return 100; // Niveau exact
  if (distance === 1) return 80;  // ±1 niveau
  if (distance === 2) return 50;  // ±2 niveaux
  return 20; // Trop loin
}
```

**Score Recommandation Conversation** :
```typescript
function calculateConversationScore(user, topic): number {
  const levelMatch = calculateLevelMatch(user.level, topic.level);
  const interestScore = calculateInterest(user.topicHistory, topic);
  const freshness = calculateFreshness(user.lastConversationTime);
  const skillRelevance = calculateSkillRelevance(user.weakSkills, topic.skills);
  
  return (
    levelMatch * 0.3 +
    interestScore * 0.3 +
    freshness * 0.2 +
    skillRelevance * 0.2
  );
}
```

### Moteur de Recommandation

**Pipeline** :
1. **Analyse Profil Utilisateur**
   - Niveau actuel (CEFR)
   - Skills maîtrisés vs manquants
   - Historique complétions (taux succès par type)
   - Préférences topics (basé sur ratings)
   
2. **Identification Erreurs Récurrentes** (via feedback-service)
   - Top 3 catégories erreurs
   - Skills associés aux erreurs
   
3. **Détection Skills Manquants**
   - Skills requis pour progression niveau suivant
   - Skills pré-requis pour leçons avancées
   
4. **Calcul Scores Recommandation**
   - Pour chaque leçon/conversation disponible
   - Application filtres (tier, déblocage, quotas)
   
5. **Ranking Final**
   - Tri par priority score DESC
   - Diversification (équilibre types)
   - Sélection top N (default: 5)

### Types de Recommandations

**Lesson** :
- `reason: skill_gap` : Skill manquant identifié
- `reason: error_pattern` : Erreur récurrente (ex: verb_tense)
- `reason: level_progression` : Progression naturelle curriculum
- `reason: review_due` : Révision espacée (spaced repetition)

**Conversation** :
- `reason: practice_speaking` : Peu de pratique orale récente
- `reason: skill_reinforcement` : Renforcer skill acquis en leçon
- `reason: comfort_topic` : Topic familier pour confiance

**Skill Practice** :
- `reason: weak_skill` : Proficiency <50%
- `reason: prerequisite` : Pré-requis pour contenu avancé

**Review** :
- `reason: spaced_repetition` : nextReviewAt atteint
- `reason: low_retention` : Score baisse sur révisions précédentes

### Événements Kafka
- **Consomme** :
  - `lesson.completed` → Mettre à jour état apprentissage
  - `conversation.completed` → Mettre à jour préférences topics
  - `feedback.report.generated` → Analyser erreurs récurrentes
  - `skill.acquired` → Mettre à jour skills maîtrisés
  - `user.learning_profile.created` → Initialiser recommandations
  
- **Publie** :
  - `recommendation.generated` → notification, analytics
  - `recommendation.completed` → analytics (A/B testing)

### Endpoints Principaux
```
GET  /api/recommendations/next-best-action (top recommandation)
GET  /api/recommendations/lessons?targetLanguageCode={code}&limit=5
GET  /api/recommendations/conversations?targetLanguageCode={code}
GET  /api/recommendations/reviews (leçons à réviser)
GET  /api/recommendations/learning-path (parcours suggéré 4 semaines)
POST /api/recommendations/:id/feedback (accept/dismiss)
GET  /api/recommendations/skill-gaps (skills manquants)
```

### Stratégie de Cache (Redis)
```typescript
// Recommandations pré-calculées
key: `recommendations:user:{userId}:lang:{targetLanguageCode}`
TTL: 1 heure
Invalidation: Complétion leçon/conversation

// Learning state agrégé
key: `learning_state:user:{userId}:lang:{targetLanguageCode}`
TTL: 30 minutes
Invalidation: Événements progression

// Skill gaps
key: `skill_gaps:user:{userId}:lang:{targetLanguageCode}`
TTL: 2 heures
Invalidation: Skill acquired
```

---

## 7. API Gateway

### Responsabilités
- Point d'entrée unique pour tous les clients (web, mobile, desktop)
- Routing intelligent vers microservices
- Authentification centralisée (validation JWT)
- Rate limiting global et par endpoint
- Request/Response transformation
- Logging centralisé avec correlation IDs
- Circuit breaker pattern (protection services down)
- CORS et sécurité headers

### Technologies
- **Framework** : NestJS avec `@nestjs/microservices`
- **Proxy** : HTTP proxy vers services backend
- **Cache** : Redis pour rate limiting et circuit breaker states
- **Auth** : JWT validation (clé publique RS256 depuis auth-service)
- **Monitoring** : Prometheus metrics agrégées

### Routing Table

```typescript
const ROUTES = {
  // Auth Service (port 3001)
  '/api/auth/*': 'http://auth-service:3001',
  '/api/users/*': 'http://auth-service:3001',
  '/api/learning-profiles/*': 'http://auth-service:3001',
  
  // Lesson Service (port 3002)
  '/api/courses/*': 'http://lesson-service:3002',
  '/api/units/*': 'http://lesson-service:3002',
  '/api/lessons/*': 'http://lesson-service:3002',
  '/api/exercises/*': 'http://lesson-service:3002',
  '/api/skills/*': 'http://lesson-service:3002',
  '/api/progress/*': 'http://lesson-service:3002',
  
  // Conversation Service (port 3003)
  '/api/conversations/*': 'http://conversation-service:3003',
  '/ws/conversations/*': 'ws://conversation-service:3003', // WebSocket
  
  // Feedback Service (port 3004)
  '/api/feedback/*': 'http://feedback-service:3004',
  
  // Gamification Service (port 3005)
  '/api/gamification/*': 'http://gamification-service:3005',
  
  // Recommendation Service (port 3006)
  '/api/recommendations/*': 'http://recommendation-service:3006',
};
```

### Middleware Chain

**Ordre d'exécution** :
```
1. CORS Handler
   ↓
2. Rate Limiter (Redis)
   ↓
3. Request Logger (structured JSON)
   ↓
4. JWT Validator (si endpoint protégé)
   ↓
5. Tier Validator (quotas)
   ↓
6. Service Router (proxy)
   ↓
7. Circuit Breaker (protection)
   ↓
8. Response Transformer
   ↓
9. Response Logger
   ↓
10. Error Handler (global)
```

### Rate Limiting

**Limites par Tier** :
| Tier | Requests/min | Burst | Endpoints sensibles |
|------|--------------|-------|---------------------|
| Anonymous | 20 | 30 | Auth endpoints seulement |
| Free | 100 | 150 | Tous endpoints |
| Premium | 500 | 750 | Tous endpoints |
| Enterprise | 2000 | 3000 | Tous endpoints |

**Implémentation** :
```typescript
// Redis sliding window
key: `ratelimit:{tier}:{userId or IP}:{endpoint}`
TTL: 60 secondes
Counter: INCR par requête
```

**Endpoints sensibles** (rate limit réduit) :
- `POST /api/auth/login` : 5/min
- `POST /api/auth/register` : 3/min
- `POST /api/exercises/*/submit` : 60/min (anti-cheat)

### Circuit Breaker

**Configuration** :
```typescript
const circuitBreakerConfig = {
  threshold: 50, // % erreurs (ex: 50%)
  timeout: 5000, // ms (5 secondes)
  resetTimeout: 30000, // ms (30 secondes)
  volumeThreshold: 10, // min requêtes avant activation
};

// États:
// CLOSED: Tout fonctionne
// OPEN: Service down, reject immédiat avec 503
// HALF_OPEN: Test 1 requête pour voir si service récupéré
```

**Implémentation per service** :
```typescript
// Redis state
key: `circuit_breaker:{serviceName}`
values: { state: 'CLOSED|OPEN|HALF_OPEN', failures: number, lastFailure: timestamp }
```

### Authentification

**JWT Validation** :
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token && isPublicEndpoint(request.url)) {
      return true; // Endpoint public
    }
    
    if (!token) {
      throw new UnauthorizedException('Token required');
    }
    
    try {
      // Validate JWT avec clé publique RS256
      const payload = await this.jwtService.verify(token, {
        publicKey: this.publicKey,
        algorithms: ['RS256']
      });
      
      // Attach user to request
      request.user = {
        userId: payload.userId,
        email: payload.email,
        subscriptionTier: payload.subscriptionTier
      };
      
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
}
```

**Endpoints Publics** (pas d'auth) :
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh-token
POST /api/auth/forgot-password
GET  /api/auth/verify-email/:token
GET  /health
GET  /metrics
```

### Logging

**Structured JSON Logs** :
```json
// Request log
{
  "timestamp": "2025-01-15T10:30:00Z",
  "level": "info",
  "service": "api-gateway",
  "event": "request.received",
  "correlationId": "uuid",
  "userId": "uuid",
  "method": "GET",
  "path": "/api/lessons/uuid",
  "ip": "192.168.1.1",
  "userAgent": "Mozilla/5.0 ..."
}

// Response log
{
  "timestamp": "2025-01-15T10:30:01Z",
  "level": "info",
  "service": "api-gateway",
  "event": "request.completed",
  "correlationId": "uuid",
  "userId": "uuid",
  "method": "GET",
  "path": "/api/lessons/uuid",
  "statusCode": 200,
  "duration": 1234, // ms
  "targetService": "lesson-service"
}
```

### Endpoints Propres Gateway

```
GET  /health (health check agrégé)
  Response: {
    status: 'ok',
    timestamp: '2025-01-15T10:30:00Z',
    services: {
      'auth-service': 'ok',
      'lesson-service': 'ok',
      'conversation-service': 'degraded',
      'feedback-service': 'ok',
      'gamification-service': 'ok',
      'recommendation-service': 'ok'
    }
  }

GET  /metrics (Prometheus metrics)
  - gateway_requests_total (counter)
  - gateway_request_duration_seconds (histogram)
  - gateway_rate_limit_exceeded_total (counter)
  - gateway_circuit_breaker_state (gauge)

GET  /docs (Swagger UI agrégé)
  - Agrégation OpenAPI specs de tous les services
```

### Sécurité Headers

```typescript
// Helmet.js configuration
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https://cdn.wespeak.com"],
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));

// Custom headers
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Configuration

```bash
# Application
NODE_ENV=production
PORT=3000
LOG_LEVEL=info

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=secret

# JWT
JWT_PUBLIC_KEY_PATH=/secrets/jwt-public.pem

# Backend Services
AUTH_SERVICE_URL=http://auth-service:3001
LESSON_SERVICE_URL=http://lesson-service:3002
CONVERSATION_SERVICE_URL=http://conversation-service:3003
FEEDBACK_SERVICE_URL=http://feedback-service:3004
GAMIFICATION_SERVICE_URL=http://gamification-service:3005
RECOMMENDATION_SERVICE_URL=http://recommendation-service:3006

# Rate Limiting
RATE_LIMIT_ANONYMOUS_MAX=20
RATE_LIMIT_FREE_MAX=100
RATE_LIMIT_PREMIUM_MAX=500
RATE_LIMIT_ENTERPRISE_MAX=2000

# Circuit Breaker
CIRCUIT_BREAKER_THRESHOLD=50
CIRCUIT_BREAKER_TIMEOUT=5000
CIRCUIT_BREAKER_RESET_TIMEOUT=30000

# CORS
CORS_ORIGINS=https://wespeak.com,https://app.wespeak.com
CORS_METHODS=GET,POST,PUT,DELETE,PATCH
CORS_CREDENTIALS=true
```

---

## 📊 Architecture Événementielle - Topics Kafka

### Topics et Flux d'Événements

| Topic | Producteur | Consommateurs | Partitioning Key | Retention |
|-------|-----------|---------------|------------------|-----------|
| `user.events` | auth-service | gamification, recommendation, notification, analytics | userId | 30 jours |
| `lesson.events` | lesson-service | gamification, recommendation, analytics | userId | 30 jours |
| `conversation.events` | conversation-service | feedback, gamification, analytics | userId | 30 jours |
| `feedback.events` | feedback-service | recommendation, notification, analytics | userId | 90 jours |
| `gamification.events` | gamification-service | notification, analytics | userId | 30 jours |
| `recommendation.events` | recommendation-service | notification, analytics | userId | 30 jours |

### Schema Versioning

Tous les événements suivent ce format standard :
```json
{
  "eventType": "service.action",
  "version": "1.0",
  "timestamp": "2025-01-15T10:30:00Z",
  "payload": {
    /* Données spécifiques à l'événement */
  },
  "metadata": {
    "correlationId": "uuid",
    "source": "service-name",
    "userId": "uuid",
    "sessionId": "uuid",
    "traceId": "uuid"
  }
}
```

---

## 🔐 Standards de Sécurité

### JWT Structure (RS256)

**Access Token Payload** :
```json
{
  "userId": "uuid",
  "email": "user@example.com",
  "subscriptionTier": "premium",
  "iat": 1736936400,
  "exp": 1736940000
}
```

**Validation** :
- Algorithme : RS256 (asymmetric)
- Clé publique : Distribuée par auth-service
- Expiration : 1 heure (access), 30 jours (refresh)

---

## 📈 Métriques et Monitoring

### Métriques Clés par Service

**Toutes les métriques Prometheus** :
```
{service}_requests_total (counter)
{service}_request_duration_seconds (histogram)
{service}_errors_total (counter)
{service}_active_connections (gauge)
{service}_cache_hit_rate (gauge)
{service}_kafka_lag (gauge)
{service}_database_connections (gauge)
```

**Alertes Standards** :
- Error rate >5% → Critical
- Response time p95 >2s → Warning
- Database connections <10% → Critical
- Kafka lag >1000 messages → Warning
- Service down → Critical

---

## 🚀 Prochaines Étapes

### Développement
1. ✅ Créer fichier complet **auth-service.md**
2. 📝 Créer fichiers détaillés pour les 6 services restants
3. 📐 Générer schémas Avro pour événements Kafka
4. 📄 Créer OpenAPI/Swagger specs par service
5. 🧪 Définir stratégie de tests (unit, integration, e2e, load)

### Documentation Complémentaire
1. Architecture Decision Records (ADR)
2. Runbooks opérationnels (déploiement, rollback, debug)
3. Guide développeur (conventions, patterns, best practices)
4. Guide API pour clients frontend/mobile

### Infrastructure
1. Terraform/CloudFormation pour provisionning
2. Helm charts Kubernetes
3. CI/CD pipelines (GitHub Actions)
4. Monitoring dashboards (Grafana)

---

## 📚 Références

- [Auth Service - Specs Complètes](./services/01-auth-service.md) ✅
- [Spécifications Initiales](../.github-private/wespeak_specs.md)

---

**Dernière mise à jour** : 2025-01-01  
**Version** : 1.0  
**Créé par** : WeSpeak Product Owner AI Agent  
**Contact** : product@wespeak.com
