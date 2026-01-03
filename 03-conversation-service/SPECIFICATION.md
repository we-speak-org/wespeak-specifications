# Conversation Service - Spécification Fonctionnelle v1.0

## 1. Vue d'Ensemble

### 1.1 Description
Le **conversation-service** gère les sessions de conversation en temps réel entre apprenants. Il permet à des utilisateurs parlant différentes langues de pratiquer ensemble via des appels audio/vidéo WebRTC.

### 1.2 Objectifs Principaux
- Permettre aux utilisateurs de rejoindre des sessions de conversation de groupe (2 à 6 participants)
- Matcher les apprenants selon leur langue cible et leur niveau
- Gérer les topics de conversation pour guider les échanges
- Coordonner les connexions WebRTC entre participants
- Enregistrer l'historique des sessions pour le feedback

### 1.3 Périmètre Fonctionnel

**Inclus dans le MVP :**
- Création et gestion des sessions de conversation
- Matchmaking basé sur langue et niveau
- Gestion des participants (rejoindre, quitter)
- Signaling WebRTC (offres, réponses, ICE candidates)
- Topics de conversation simples
- Historique des sessions

**Prévu pour plus tard :**
- Activités interactives pendant les sessions
- Système de modération avancé
- Enregistrement audio automatique

---

## 2. Concepts Métier

### 2.1 Session de Conversation
Une **session** est une réunion virtuelle où plusieurs apprenants pratiquent une langue ensemble.

| Propriété | Description |
|-----------|-------------|
| Nombre de participants | 2 à 6 personnes |
| Durée typique | 5 à 30 minutes |
| Langue cible | La langue que tous les participants pratiquent |
| Topic | Sujet de conversation optionnel pour guider l'échange |

### 2.2 Participant
Un **participant** est un utilisateur qui a rejoint une session.

| Propriété | Description |
|-----------|-------------|
| Rôle | `host` (créateur) ou `participant` |
| Statut | `waiting`, `connected`, `disconnected` |
| Temps de parole | Durée pendant laquelle l'utilisateur a parlé |

### 2.3 Topic
Un **topic** est un sujet de conversation proposé pour guider l'échange.

| Propriété | Description |
|-----------|-------------|
| Titre | Nom court du sujet (ex: "Weekend plans") |
| Description | Questions ou points de discussion suggérés |
| Niveau | Niveau CECR recommandé (A1-C2) |
| Catégorie | Thème général (travel, work, hobbies, etc.) |

### 2.4 File d'Attente (Matchmaking)
Le système de **matchmaking** met en relation les utilisateurs cherchant une conversation.

| Critère | Description |
|---------|-------------|
| Langue cible | Doit être identique pour tous |
| Niveau | Compatible (±1 niveau CECR) |
| Disponibilité | Utilisateurs en attente au même moment |

---

## 3. Parcours Utilisateur

### 3.1 Rejoindre une Conversation (Quick Match)

```
1. L'utilisateur clique sur "Trouver une conversation"
2. Il est placé dans la file d'attente
3. Le système cherche d'autres utilisateurs compatibles
4. Quand 2+ utilisateurs sont trouvés → session créée
5. Tous les participants sont notifiés et connectés
6. La conversation démarre avec un topic suggéré
```

### 3.2 Créer une Session Privée

```
1. L'utilisateur crée une session avec un code d'invitation
2. Il partage le code avec ses amis
3. Les invités rejoignent via le code
4. Quand le host démarre → la session commence
```

### 3.3 Déroulement d'une Session

```
1. Les participants se connectent via WebRTC
2. Un topic est affiché (optionnel)
3. Les participants conversent librement
4. Chacun peut quitter à tout moment
5. La session se termine quand tous sont partis
   OU après le temps maximum (30 min)
```

---

## 4. Modèle de Données

### 4.1 Session

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| targetLanguageCode | String | Oui | Langue pratiquée (ex: "en") |
| status | Enum | Oui | `waiting`, `active`, `ended` |
| type | Enum | Oui | `public` ou `private` |
| inviteCode | String | Non | Code pour sessions privées |
| topicId | String | Non | Topic associé |
| hostUserId | String | Oui | Créateur de la session |
| minParticipants | Number | Oui | Minimum requis (défaut: 2) |
| maxParticipants | Number | Oui | Maximum autorisé (défaut: 6) |
| scheduledStartAt | DateTime | Non | Heure de début prévue |
| startedAt | DateTime | Non | Heure de début effective |
| endedAt | DateTime | Non | Heure de fin |
| createdAt | DateTime | Oui | Date de création |

### 4.2 Participant

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| sessionId | String | Oui | Référence à la session |
| userId | String | Oui | Référence à l'utilisateur |
| role | Enum | Oui | `host` ou `participant` |
| status | Enum | Oui | `waiting`, `connected`, `disconnected` |
| joinedAt | DateTime | Oui | Heure d'arrivée |
| leftAt | DateTime | Non | Heure de départ |
| speakingTimeSeconds | Number | Non | Temps de parole cumulé |

### 4.3 Topic

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| targetLanguageCode | String | Oui | Langue du topic |
| title | String | Oui | Titre court |
| description | String | Non | Description détaillée |
| suggestedQuestions | String[] | Non | Questions pour guider |
| level | Enum | Oui | A1, A2, B1, B2, C1, C2 |
| category | String | Oui | Catégorie thématique |
| isActive | Boolean | Oui | Topic disponible ou non |

### 4.4 QueueEntry (File d'attente)

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | Utilisateur en attente |
| targetLanguageCode | String | Oui | Langue recherchée |
| level | Enum | Oui | Niveau CECR |
| preferredTopicCategory | String | Non | Catégorie préférée |
| joinedQueueAt | DateTime | Oui | Entrée dans la file |
| expiresAt | DateTime | Oui | Expiration automatique |

---

## 5. Règles Métier

### 5.1 Matchmaking

**Critères de compatibilité :**
- Même langue cible (obligatoire)
- Niveau compatible : différence max de 1 niveau
  - A1 peut matcher avec A1, A2
  - B1 peut matcher avec A2, B1, B2
- Temps d'attente max : 2 minutes

**Algorithme simplifié :**
```
1. Trier la file par temps d'attente (plus ancien d'abord)
2. Pour chaque utilisateur en attente :
   a. Chercher d'autres utilisateurs compatibles
   b. Si 2+ trouvés → créer une session
   c. Notifier tous les matchés
3. Supprimer les entrées expirées (> 2 min)
```

### 5.2 Gestion des Sessions

**Démarrage :**
- Session publique : démarre automatiquement quand minParticipants atteint
- Session privée : démarre quand le host le décide

**Fin de session :**
- Tous les participants sont partis
- Durée max atteinte (30 minutes)
- Le host termine manuellement (session privée)

**Limites :**
- Max 6 participants par session
- Max 3 sessions actives par utilisateur
- Max 10 sessions par jour par utilisateur (free tier)

### 5.3 Signaling WebRTC

Le service gère le signaling pour établir les connexions peer-to-peer :

```
1. Participant A rejoint → reçoit liste des autres participants
2. A envoie une "offer" SDP à chaque participant existant
3. Chaque participant répond avec une "answer" SDP
4. Échange des ICE candidates pour établir la connexion
5. Connexion P2P établie → communication directe
```

---

## 6. Interactions avec Autres Services

### 6.1 Services Appelés

| Service | Raison |
|---------|--------|
| auth-service | Valider le token JWT, récupérer profil utilisateur |

### 6.2 Événements Publiés

| Événement | Quand | Consommateurs |
|-----------|-------|---------------|
| session.started | Session démarre | gamification-service |
| session.ended | Session termine | gamification-service, feedback-service |
| participant.joined | Utilisateur rejoint | - |
| participant.left | Utilisateur quitte | - |

### 6.3 Événements Consommés

| Événement | Source | Action |
|-----------|--------|--------|
| user.deleted | auth-service | Supprimer historique utilisateur |

---

## 7. Points d'Attention

### 7.1 Performance
- La file d'attente utilise Redis pour des recherches rapides
- Les sessions actives sont cachées en mémoire
- Le signaling WebRTC doit être < 100ms de latence

### 7.2 Scalabilité
- Le matchmaking peut tourner sur plusieurs instances
- Utiliser un lock distribué (Redis) pour éviter les doubles matchs
- Les WebSockets sont sticky (affinité de session)

### 7.3 Résilience
- Si un participant perd sa connexion → 30s pour se reconnecter
- Si le host quitte → le plus ancien participant devient host
- Sessions orphelines nettoyées après 5 minutes d'inactivité

---

## 8. Évolutions Futures

### Phase 2 : Activités
- Jeux de vocabulaire pendant la session
- Quiz collaboratifs
- Lecture de textes à tour de rôle

### Phase 3 : Modération
- Signalement de comportements inappropriés
- Système de réputation
- Modération par IA

### Phase 4 : Enregistrement
- Enregistrement audio opt-in
- Transcription automatique
- Feedback IA post-session
