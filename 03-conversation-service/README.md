# Conversation Service - Spécifications Fonctionnelles v1.0

## 📋 Table des Matières

1. [Vue d'Ensemble](#1-vue-densemble)
2. [Modèle de Données](#2-modèle-de-données)
3. [Fonctionnalités Principales](#3-fonctionnalités-principales)
4. [Règles Métier](#4-règles-métier)
5. [Interactions avec les Autres Services](#5-interactions-avec-les-autres-services)

---

## 1. Vue d'Ensemble

### 1.1 Responsabilité

Le **conversation-service** gère les conversations orales en temps réel entre apprenants. C'est le cœur de la pratique orale de WeSpeak, permettant aux utilisateurs de s'exercer avec de vrais partenaires humains.

**Fonctions principales :**
- Gestion des sujets de conversation (topics)
- Matchmaking entre apprenants de niveaux compatibles
- Gestion des sessions de conversation WebRTC
- Signalisation WebRTC (offres/réponses SDP, candidats ICE)
- Historique des conversations

### 1.2 Dépendances

| Service | Interaction |
|---------|-------------|
| auth-service | Validation JWT, récupération profil utilisateur et niveau |
| feedback-service | Envoi de l'audio pour transcription et analyse après session |
| gamification-service | Attribution XP après conversation terminée |

### 1.3 Stack Technique

- **Spring Boot 4** avec WebFlux pour le temps réel
- **MongoDB** pour le stockage des sessions et topics
- **Redis** pour la file d'attente de matchmaking et état temps réel
- **Kafka** pour les événements asynchrones
- **WebSocket** pour la signalisation WebRTC

---

## 2. Modèle de Données

### 2.1 Topic (Sujet de conversation)

Un topic est un sujet proposé pour guider la conversation entre apprenants.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| targetLanguageCode | String | Oui | Langue cible (ex: "en", "fr") |
| level | String | Oui | Niveau CECRL: A1, A2, B1, B2, C1, C2 |
| title | String | Oui | Titre du sujet |
| description | String | Non | Description détaillée |
| promptQuestions | String[] | Non | Questions suggérées pour lancer la discussion |
| category | String | Oui | Catégorie: daily_life, work, travel, culture, hobbies, news |
| estimatedDurationMinutes | Integer | Oui | Durée suggérée (5, 10, 15, 20 min) |
| isActive | Boolean | Oui | Topic disponible ou non |
| createdAt | DateTime | Oui | Date de création |

### 2.2 ConversationSession

Une session représente une conversation entre deux apprenants.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| topicId | String | Non | Topic choisi (peut être conversation libre) |
| targetLanguageCode | String | Oui | Langue pratiquée |
| participant1Id | String | Oui | ID du premier participant |
| participant2Id | String | Oui | ID du second participant |
| status | String | Oui | État: waiting, active, completed, cancelled |
| scheduledAt | DateTime | Non | Si planifiée à l'avance |
| startedAt | DateTime | Non | Début effectif |
| endedAt | DateTime | Non | Fin de la session |
| actualDurationSeconds | Integer | Non | Durée réelle en secondes |
| endReason | String | Non | Raison fin: completed, dropped, timeout, reported |
| audioRecordingUrl | String | Non | URL R2 de l'enregistrement |
| createdAt | DateTime | Oui | Date de création |

### 2.3 MatchmakingRequest

Demande d'un utilisateur pour trouver un partenaire de conversation.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Utilisateur demandeur |
| learningProfileId | String | Oui | Profil d'apprentissage actif |
| targetLanguageCode | String | Oui | Langue à pratiquer |
| userLevel | String | Oui | Niveau de l'utilisateur |
| preferredTopicId | String | Non | Topic préféré |
| preferredDuration | Integer | Oui | Durée souhaitée en minutes |
| status | String | Oui | État: pending, matched, expired, cancelled |
| matchedWithUserId | String | Non | Partenaire trouvé |
| sessionId | String | Non | Session créée après match |
| createdAt | DateTime | Oui | Date de création |
| expiresAt | DateTime | Oui | Expiration (2 minutes max) |

---

## 3. Fonctionnalités Principales

### 3.1 Gestion des Topics

Le service maintient une bibliothèque de sujets de conversation adaptés aux différents niveaux. Les topics sont créés par les administrateurs et filtrés par langue et niveau pour chaque utilisateur.

**Parcours utilisateur :**
1. L'utilisateur accède à la liste des topics pour sa langue cible
2. Les topics sont filtrés par son niveau actuel (et niveaux adjacents)
3. L'utilisateur peut choisir un topic ou opter pour une conversation libre

### 3.2 Matchmaking

Le matchmaking trouve un partenaire compatible en temps réel.

**Algorithme de compatibilité :**
1. **Même langue cible** - Critère obligatoire
2. **Niveau compatible** - Même niveau ou ±1 niveau (A2 peut parler avec A1, A2, B1)
3. **Durée similaire** - Préférence de durée compatible
4. **Topic commun** - Si les deux ont choisi le même topic, priorité

**File d'attente Redis :**
- Clé: `matchmaking:{targetLanguageCode}:{level}`
- Les utilisateurs sont ajoutés à la file correspondant à leur niveau
- Scan des files de niveaux adjacents si pas de match direct

**Timeout :**
- Expiration après 2 minutes sans match
- Notification à l'utilisateur

### 3.3 Session WebRTC

Une fois le match établi, une session de conversation démarre.

**Flux de signalisation :**
1. Le service notifie les deux participants du match
2. Participant1 crée une offre SDP et l'envoie via WebSocket
3. Le service relaye l'offre à Participant2
4. Participant2 répond avec une réponse SDP
5. Échange des candidats ICE
6. Connexion P2P établie

**Gestion de la session :**
- Heartbeat toutes les 10 secondes pour vérifier la connexion
- Détection de déconnexion après 30 secondes sans heartbeat
- Notification au partenaire si l'autre quitte
- Timer visible avec alerte à 1 minute de la fin

### 3.4 Fin de Session

**Fin normale :**
1. Un participant clique "Terminer"
2. L'autre est notifié et la session se termine
3. L'audio est uploadé vers R2
4. Événement envoyé au feedback-service pour analyse
5. XP attribué via gamification-service

**Fin anormale :**
- Déconnexion détectée → session marquée "dropped"
- Timeout sans activité → session annulée
- Signalement → session marquée "reported"

---

## 4. Règles Métier

### 4.1 Règles de Matchmaking

| Règle | Description |
|-------|-------------|
| Niveau ±1 | Un A2 peut être matché avec A1, A2, B1 |
| Pas soi-même | Un utilisateur ne peut pas se matcher avec lui-même |
| Pas de doublon récent | Éviter le même partenaire dans les 24h (soft rule) |
| Durée compatible | Écart max de 5 minutes entre durées souhaitées |

### 4.2 Règles de Session

| Règle | Description |
|-------|-------------|
| Durée minimum | 2 minutes minimum pour valider une session |
| Durée maximum | 30 minutes maximum par session |
| XP conditionnel | XP attribué uniquement si durée ≥ 2 minutes |
| Enregistrement consent | Les deux participants doivent accepter l'enregistrement |

### 4.3 Limites

| Limite | Valeur | Description |
|--------|--------|-------------|
| Sessions/jour (free) | 3 | Limite pour compte gratuit |
| Sessions/jour (premium) | Illimité | Pas de limite premium |
| Timeout matchmaking | 120s | Temps max d'attente |
| Timeout inactivité | 60s | Déconnexion si pas de heartbeat |

---

## 5. Interactions avec les Autres Services

### 5.1 Événements Publiés (Kafka)

**Topic: `conversation.events`**

#### session.completed
Publié quand une session se termine normalement.
```
Déclenche: 
- feedback-service: Analyse de l'audio
- gamification-service: Attribution XP
```

#### session.cancelled
Publié quand une session est annulée ou abandonnée.

### 5.2 Événements Consommés

**Topic: `user.events`**
- `user.deleted`: Supprimer les données de l'utilisateur

**Topic: `feedback.events`**
- `feedback.completed`: Mettre à jour la session avec le rapport

---

## 6. Parcours Utilisateur Type

```
1. Marie ouvre l'app et veut pratiquer l'anglais
2. Elle consulte les topics disponibles pour son niveau B1
3. Elle choisit "Travel experiences" et demande 10 minutes
4. Elle clique "Trouver un partenaire"
5. Le système la met en file d'attente (matchmaking)
6. Après 15 secondes, Jean (B1 aussi) est trouvé
7. Les deux sont notifiés et la connexion WebRTC s'établit
8. Ils conversent pendant 10 minutes sur leurs voyages
9. Marie clique "Terminer" à la fin du timer
10. Session enregistrée, audio envoyé pour analyse
11. Marie reçoit 50 XP pour sa participation
12. Elle pourra consulter son feedback dans quelques minutes
```
