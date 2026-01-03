# Gamification Service - Spécifications Fonctionnelles v1.0

## 1. Vue d'Ensemble

### 1.1 Responsabilité
Le **gamification-service** gère tout le système de motivation et d'engagement des utilisateurs :
- Points d'expérience (XP) gagnés pour chaque activité
- Badges et récompenses pour les accomplissements
- Streaks (séries de jours consécutifs d'apprentissage)
- Leaderboards (classements hebdomadaires/mensuels)
- Défis personnels et communautaires

### 1.2 Objectif Fonctionnel
Créer un système de gamification engageant inspiré de Duolingo pour :
- Maintenir la motivation quotidienne des apprenants
- Récompenser la régularité et les progrès
- Créer une compétition saine via les leaderboards
- Encourager l'exploration de toutes les fonctionnalités

### 1.3 Dépendances
| Service | Interaction |
|---------|-------------|
| auth-service | Récupération profil utilisateur |
| lesson-service | Événements de complétion de leçons/exercices |
| conversation-service | Événements de participation aux sessions |
| feedback-service | Événements d'amélioration détectée |

---

## 2. Modèle de Données

### 2.1 UserStats (Statistiques Utilisateur)

Stocke les statistiques gamification d'un utilisateur pour une langue cible.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Référence à l'utilisateur |
| targetLanguageCode | String | Oui | Langue concernée (ex: "en") |
| totalXp | Integer | Oui | Total des XP accumulés (défaut: 0) |
| weeklyXp | Integer | Oui | XP de la semaine en cours (défaut: 0) |
| monthlyXp | Integer | Oui | XP du mois en cours (défaut: 0) |
| currentStreak | Integer | Oui | Nombre de jours consécutifs (défaut: 0) |
| longestStreak | Integer | Oui | Plus longue série atteinte (défaut: 0) |
| lastActivityDate | Date | Non | Date de la dernière activité |
| streakFreezeAvailable | Integer | Oui | Nombre de "freeze" disponibles (défaut: 0) |
| level | Integer | Oui | Niveau actuel (défaut: 1) |
| lessonsCompleted | Integer | Oui | Nombre de leçons terminées |
| exercisesCompleted | Integer | Oui | Nombre d'exercices terminés |
| conversationMinutes | Integer | Oui | Minutes de conversation totales |
| perfectLessons | Integer | Oui | Leçons sans erreur |
| createdAt | Date | Oui | Date de création |
| updatedAt | Date | Oui | Date de mise à jour |

### 2.2 Badge

Représente un badge que les utilisateurs peuvent débloquer.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| code | String | Oui | Code unique du badge (ex: "first_lesson") |
| name | String | Oui | Nom du badge |
| description | String | Oui | Description de comment l'obtenir |
| iconUrl | String | Oui | URL de l'icône |
| category | String | Oui | Catégorie: lessons, conversations, streaks, social, special |
| xpReward | Integer | Oui | XP gagnés en débloquant |
| rarity | String | Oui | Rareté: common, uncommon, rare, epic, legendary |
| condition | Object | Oui | Condition de déblocage (type + valeur) |
| isActive | Boolean | Oui | Badge actif ou non |

### 2.3 UserBadge

Association entre un utilisateur et ses badges débloqués.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Référence à l'utilisateur |
| badgeId | String | Oui | Référence au badge |
| unlockedAt | Date | Oui | Date de déblocage |
| targetLanguageCode | String | Non | Langue concernée si applicable |

### 2.4 Challenge (Défi)

Défis temporaires pour gagner des XP bonus.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| title | String | Oui | Titre du défi |
| description | String | Oui | Description |
| type | String | Oui | Type: daily, weekly, special |
| targetType | String | Oui | Ce qu'il faut accomplir: xp, lessons, conversations, streak |
| targetValue | Integer | Oui | Valeur à atteindre |
| xpReward | Integer | Oui | XP de récompense |
| startDate | Date | Oui | Date de début |
| endDate | Date | Oui | Date de fin |
| isActive | Boolean | Oui | Défi actif |

### 2.5 UserChallenge

Progression d'un utilisateur sur un défi.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Référence à l'utilisateur |
| challengeId | String | Oui | Référence au défi |
| currentProgress | Integer | Oui | Progression actuelle (défaut: 0) |
| status | String | Oui | Statut: in_progress, completed, failed |
| completedAt | Date | Non | Date de complétion |
| joinedAt | Date | Oui | Date d'inscription au défi |

### 2.6 XpTransaction

Historique des gains d'XP.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Référence à l'utilisateur |
| targetLanguageCode | String | Oui | Langue concernée |
| amount | Integer | Oui | XP gagnés (positif) |
| source | String | Oui | Source: lesson, exercise, conversation, badge, challenge, streak_bonus |
| sourceId | String | Non | ID de la source (leçon, badge, etc.) |
| description | String | Non | Description optionnelle |
| createdAt | Date | Oui | Date de la transaction |

---

## 3. Règles Métier

### 3.1 Calcul des XP

| Action | XP de base | Bonus possible |
|--------|-----------|----------------|
| Compléter un exercice | 10 XP | +5 si parfait (100%) |
| Compléter une leçon | 15 XP | +10 si parfait |
| Participer à une conversation (par 5 min) | 20 XP | - |
| Débloquer un badge | Variable selon badge | - |
| Compléter un défi | Variable selon défi | - |
| Bonus streak (7 jours) | 50 XP | - |
| Bonus streak (30 jours) | 200 XP | - |

### 3.2 Système de Niveaux

Le niveau est calculé en fonction du total d'XP :
- Niveau 1 : 0 - 99 XP
- Niveau 2 : 100 - 299 XP
- Niveau 3 : 300 - 599 XP
- Niveau N : XP requis = 100 * (N-1) * N / 2

### 3.3 Gestion des Streaks

**Règles :**
1. Un streak augmente si l'utilisateur fait au moins une activité par jour
2. Une "activité" = compléter au moins 1 exercice OU participer à 1 conversation
3. Le streak se réinitialise à 0 si aucune activité pendant 24h (minuit UTC)
4. Un "Streak Freeze" permet de préserver le streak pendant 1 jour d'inactivité
5. Les utilisateurs premium reçoivent 2 freezes par mois

### 3.4 Leaderboards

**Types de classements :**
- **Hebdomadaire** : Reset chaque lundi à 00:00 UTC
- **Mensuel** : Reset le 1er de chaque mois
- **Global** : Cumul total (par langue)

**Ligues (optionnel pour v2) :**
- Bronze, Silver, Gold, Platinum, Diamond
- Promotion/Relégation basée sur le classement hebdomadaire

---

## 4. Parcours Utilisateur

### 4.1 Premier Badge
1. L'utilisateur complète sa première leçon
2. Le lesson-service publie `lesson.completed`
3. Le gamification-service reçoit l'événement
4. Il attribue 15 XP + badge "first_lesson" (10 XP bonus)
5. Il publie `xp.earned` et `badge.unlocked`
6. Le frontend affiche une animation de célébration

### 4.2 Maintien du Streak
1. Chaque jour à 23:00 UTC, un job vérifie les activités du jour
2. Si activité détectée → streak +1 (si nouveau max → badge potentiel)
3. Si aucune activité et freeze disponible → utiliser freeze, streak maintenu
4. Si aucune activité et pas de freeze → streak = 0

### 4.3 Défi Hebdomadaire
1. Chaque lundi, nouveaux défis générés
2. L'utilisateur voit les défis disponibles sur son dashboard
3. Il s'inscrit aux défis qui l'intéressent
4. Sa progression est mise à jour en temps réel
5. À la fin du défi, récompense si objectif atteint

---

## 5. Badges Prédéfinis

### Catégorie : Lessons
| Code | Nom | Condition | XP | Rareté |
|------|-----|-----------|-----|--------|
| first_lesson | Première Leçon | 1 leçon complétée | 10 | common |
| lesson_master_10 | Apprenti | 10 leçons complétées | 25 | common |
| lesson_master_50 | Étudiant | 50 leçons complétées | 50 | uncommon |
| lesson_master_100 | Scholar | 100 leçons complétées | 100 | rare |
| perfect_lesson | Perfectionniste | 1 leçon parfaite | 15 | common |
| perfect_10 | Expert | 10 leçons parfaites | 50 | uncommon |

### Catégorie : Conversations
| Code | Nom | Condition | XP | Rareté |
|------|-----|-----------|-----|--------|
| first_conversation | Ice Breaker | 1ère conversation | 20 | common |
| social_butterfly | Papillon Social | 10 conversations | 50 | uncommon |
| chatterbox | Bavard | 50 conversations | 100 | rare |
| conversation_marathon | Marathonien | 60 min en une session | 75 | rare |

### Catégorie : Streaks
| Code | Nom | Condition | XP | Rareté |
|------|-----|-----------|-----|--------|
| streak_7 | Semaine Parfaite | 7 jours consécutifs | 50 | common |
| streak_30 | Mois d'Or | 30 jours consécutifs | 200 | rare |
| streak_100 | Centurion | 100 jours consécutifs | 500 | epic |
| streak_365 | Légende | 365 jours consécutifs | 1000 | legendary |

---

## 6. Communication Asynchrone

### 6.1 Événements Consommés

**lesson.completed** (depuis lesson-service)
- Action : Ajouter XP, vérifier badges, mettre à jour streak

**exercise.completed** (depuis lesson-service)
- Action : Ajouter XP, mettre à jour compteurs

**conversation.ended** (depuis conversation-service)
- Action : Ajouter XP basé sur la durée, vérifier badges

**feedback.generated** (depuis feedback-service)
- Action : Bonus XP si amélioration détectée

### 6.2 Événements Publiés

**xp.earned**
- Quand : À chaque gain d'XP
- Données : userId, amount, source, totalXp, newLevel

**badge.unlocked**
- Quand : Déblocage d'un nouveau badge
- Données : userId, badgeId, badgeCode, badgeName

**streak.updated**
- Quand : Changement de streak
- Données : userId, currentStreak, longestStreak, streakLost

**level.up**
- Quand : Passage à un niveau supérieur
- Données : userId, newLevel, totalXp

**challenge.completed**
- Quand : Défi réussi
- Données : userId, challengeId, xpReward

---

## 7. Points d'Attention

### Performance
- Les leaderboards doivent être calculés efficacement (index sur weeklyXp, monthlyXp)
- Cache des classements top 100 (rafraîchi toutes les 5 minutes)

### Intégrité
- Toutes les transactions XP doivent être idempotentes (pas de double comptage)
- Vérification anti-triche sur les gains anormaux

### UX
- Animations de célébration pour les accomplissements
- Notifications push pour rappel de streak en danger
