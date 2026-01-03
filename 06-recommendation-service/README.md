# Recommendation Service - Spécifications Fonctionnelles v1.0

## 1. Vue d'Ensemble

### 1.1 Responsabilité
Le **recommendation-service** fournit des recommandations personnalisées aux utilisateurs :
- Prochaines leçons à suivre basées sur la progression
- Créneaux de conversation suggérés selon disponibilité et niveau
- Exercices de révision basés sur les erreurs passées
- Contenu adapté au profil d'apprentissage

### 1.2 Objectif Fonctionnel
Créer un système de recommandation intelligent pour :
- Guider l'utilisateur dans son parcours d'apprentissage
- Maximiser l'engagement en proposant du contenu pertinent
- Suggérer des sessions de conversation avec partenaires compatibles
- Proposer des révisions ciblées sur les points faibles

### 1.3 Dépendances
| Service | Interaction |
|---------|-------------|
| auth-service | Profil utilisateur et préférences |
| lesson-service | Progression, historique des leçons |
| conversation-service | Historique des sessions, disponibilités |
| feedback-service | Erreurs fréquentes, points à améliorer |
| gamification-service | Niveau, objectifs |

---

## 2. Modèle de Données

### 2.1 UserPreferences (Préférences Utilisateur)

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Référence utilisateur |
| targetLanguageCode | String | Oui | Langue concernée |
| preferredLearningTime | String[] | Non | Heures préférées (ex: ["morning", "evening"]) |
| dailyGoalMinutes | Integer | Oui | Objectif quotidien (défaut: 15) |
| focusAreas | String[] | Non | Domaines prioritaires (grammar, vocabulary, pronunciation) |
| excludedTopics | String[] | Non | Sujets à éviter |
| createdAt | Date | Oui | Date de création |
| updatedAt | Date | Oui | Date de mise à jour |

### 2.2 LearningHistory (Historique d'Apprentissage)

Cache local des données agrégées pour optimiser les recommandations.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Référence utilisateur |
| targetLanguageCode | String | Oui | Langue concernée |
| completedLessonIds | String[] | Oui | IDs des leçons complétées |
| weakAreas | Object[] | Oui | Points faibles identifiés |
| strongAreas | Object[] | Oui | Points forts identifiés |
| lastLessonId | String | Non | Dernière leçon suivie |
| averageScore | Float | Oui | Score moyen (défaut: 0) |
| totalLessons | Integer | Oui | Nombre de leçons (défaut: 0) |
| lastUpdated | Date | Oui | Dernière mise à jour |

### 2.3 Recommendation

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Référence utilisateur |
| type | String | Oui | Type: next_lesson, revision, conversation, practice |
| targetId | String | Oui | ID de la ressource recommandée |
| targetType | String | Oui | Type: lesson, exercise, slot |
| reason | String | Oui | Raison de la recommandation |
| priority | Integer | Oui | Priorité (1 = haute) |
| expiresAt | Date | Non | Date d'expiration |
| dismissed | Boolean | Oui | Ignorée par l'utilisateur (défaut: false) |
| clickedAt | Date | Non | Date de clic |
| createdAt | Date | Oui | Date de création |

---

## 3. Types de Recommandations

### 3.1 Prochaine Leçon (next_lesson)

**Algorithme :**
1. Récupérer le cours actuel de l'utilisateur
2. Trouver la prochaine leçon non complétée dans l'ordre
3. Vérifier que les prérequis sont remplis
4. Si fin d'unité → recommander passage à l'unité suivante

**Raisons possibles :**
- "Continuez votre progression"
- "Nouvelle unité débloquée"
- "Vous avez presque fini ce chapitre"

### 3.2 Révision (revision)

**Algorithme :**
1. Analyser les feedbacks récents pour identifier les erreurs fréquentes
2. Trouver des exercices ciblant ces points faibles
3. Prioriser par fréquence d'erreur et ancienneté

**Raisons possibles :**
- "Renforcez vos bases en conjugaison"
- "Exercice de révision recommandé"
- "Vous avez fait des erreurs sur ce point"

### 3.3 Conversation (conversation)

**Algorithme :**
1. Récupérer les créneaux disponibles compatibles avec le niveau
2. Filtrer par préférences horaires de l'utilisateur
3. Prioriser les créneaux avec places restantes
4. Suggérer des créneaux où des partenaires de niveau similaire sont inscrits

**Raisons possibles :**
- "Pratiquez avec d'autres apprenants"
- "Session disponible à votre heure préférée"
- "Mettez en pratique vos acquis"

### 3.4 Pratique Rapide (practice)

**Algorithme :**
1. Si l'utilisateur n'a pas atteint son objectif quotidien
2. Proposer des exercices courts (5-10 min)
3. Cibler les vocabulaires récemment appris

**Raisons possibles :**
- "5 minutes pour atteindre votre objectif"
- "Révisez le vocabulaire d'hier"

---

## 4. Parcours Utilisateur

### 4.1 Dashboard Personnalisé
1. L'utilisateur ouvre l'application
2. Le frontend appelle GET /recommendations
3. Le service calcule les recommandations en temps réel
4. Retourne 3-5 recommandations prioritaires
5. L'utilisateur clique sur une recommandation
6. Le service enregistre le clic (analytics)

### 4.2 Mise à Jour Continue
1. Événement `lesson.completed` reçu
2. Mise à jour de LearningHistory
3. Recalcul des recommandations
4. Les nouvelles recommandations remplacent les anciennes

---

## 5. Règles Métier

### 5.1 Priorités
| Priorité | Type | Condition |
|----------|------|-----------|
| 1 | next_lesson | Toujours si progression possible |
| 2 | revision | Si erreurs récentes détectées |
| 3 | conversation | Si créneaux disponibles aujourd'hui |
| 4 | practice | Si objectif quotidien non atteint |

### 5.2 Limites
- Maximum 5 recommandations actives par utilisateur
- Une recommandation expire après 24h si non cliquée
- Les recommandations ignorées (dismissed) ne réapparaissent pas pendant 7 jours

### 5.3 Personnalisation
- Respecter les focusAreas de l'utilisateur
- Éviter les excludedTopics
- Adapter aux preferredLearningTime

---

## 6. Communication Asynchrone

### 6.1 Événements Consommés

**lesson.completed** (depuis lesson-service)
- Action : Mettre à jour LearningHistory, recalculer recommandations

**feedback.generated** (depuis feedback-service)
- Action : Mettre à jour weakAreas/strongAreas

**user.profile.updated** (depuis auth-service)
- Action : Rafraîchir préférences utilisateur

### 6.2 Événements Publiés

**recommendation.generated**
- Quand : Nouvelles recommandations calculées
- Usage : Analytics, notifications push

**recommendation.clicked**
- Quand : Utilisateur clique sur une recommandation
- Usage : Analytics, amélioration algorithme

---

## 7. Points d'Attention

### Performance
- Les recommandations doivent être calculées en < 200ms
- Cache des données de progression
- Pré-calcul périodique pour les utilisateurs actifs

### Qualité
- Éviter les recommandations répétitives
- Varier les types de recommandations
- Adapter au niveau de l'utilisateur

### Analytics
- Tracker le taux de clic par type de recommandation
- Mesurer la conversion (clic → complétion)
- A/B testing sur les raisons affichées
