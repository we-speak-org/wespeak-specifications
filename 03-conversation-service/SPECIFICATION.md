# Conversation Service - Spécification Fonctionnelle v1.0

## 1. Vue d'Ensemble

### 1.1 Description
Le **conversation-service** gère les sessions de pratique orale entre apprenants. Les utilisateurs réservent des créneaux horaires, rejoignent des sessions vidéo/audio via WebRTC, et peuvent enregistrer leurs conversations pour recevoir un feedback IA.

### 1.2 Objectifs Principaux
- Proposer des créneaux horaires pour pratiquer une langue
- Matcher les apprenants par langue cible et niveau
- Gérer les sessions vidéo multi-participants (2 à 8 personnes)
- Permettre l'enregistrement audio opt-in pour feedback IA
- Coordonner les connexions WebRTC entre participants

### 1.3 Périmètre Fonctionnel

**Inclus dans le MVP :**
- Créneaux horaires programmés par la plateforme
- Inscription des utilisateurs aux créneaux
- Matchmaking simple par langue/niveau
- Sessions multi-participants avec audio/vidéo
- Contrôles micro/caméra par participant
- Enregistrement audio opt-in
- Envoi de l'enregistrement au feedback-service

**Prévu pour plus tard :**
- Activités interactives pendant les sessions
- Système de modération avancé

---

## 2. Concepts Métier

### 2.1 Créneau (TimeSlot)
Un **créneau** est une plage horaire prédéfinie où des sessions peuvent avoir lieu.

| Propriété | Description |
|-----------|-------------|
| Heure de début | Moment où le créneau commence |
| Durée | 15, 30 ou 45 minutes |
| Langue cible | Langue pratiquée pendant ce créneau |
| Capacité | Nombre max de participants |
| Récurrence | Quotidien, hebdomadaire, ou unique |

### 2.2 Session
Une **session** est une réunion vidéo qui se déroule pendant un créneau. Elle est créée automatiquement quand des participants sont matchés.

| Propriété | Description |
|-----------|-------------|
| Nombre de participants | 2 à 8 personnes |
| Durée | Définie par le créneau |
| Langue cible | Langue que tous pratiquent |
| Enregistrement | Activé si au moins 1 participant consent |

### 2.3 Participant
Un **participant** est un utilisateur inscrit à un créneau et connecté à une session.

| Propriété | Description |
|-----------|-------------|
| État caméra | Activée ou désactivée |
| État micro | Activé ou désactivé |
| Consentement enregistrement | Accepte ou refuse |
| Temps de présence | Durée de participation |

### 2.4 Enregistrement
L'**enregistrement** capture l'audio de la session pour analyse ultérieure.

| Propriété | Description |
|-----------|-------------|
| Format | Audio uniquement (pas de vidéo) |
| Stockage | Cloudflare R2 avec URL signée |
| Consentement | Requis de tous les participants enregistrés |
| Durée de rétention | 30 jours puis suppression |

---

## 3. Parcours Utilisateur

### 3.1 S'inscrire à un Créneau

```
1. L'utilisateur consulte les créneaux disponibles
2. Il filtre par langue et horaire
3. Il sélectionne un créneau et s'inscrit
4. Il reçoit une confirmation avec rappel
5. 5 minutes avant → notification de rappel
```

### 3.2 Rejoindre une Session

```
1. L'utilisateur clique sur "Rejoindre" (disponible 5 min avant)
2. Il autorise l'accès caméra/micro
3. Il choisit s'il consent à l'enregistrement
4. Il rejoint la "salle d'attente"
5. Quand le créneau démarre → connexion WebRTC établie
6. Il voit les autres participants en grille vidéo
```

### 3.3 Pendant la Session

```
1. Les participants conversent librement
2. Chacun peut activer/désactiver sa caméra
3. Chacun peut activer/désactiver son micro
4. Un timer affiche le temps restant
5. L'audio est enregistré si consentement donné
6. À la fin du créneau → session terminée automatiquement
```

### 3.4 Après la Session

```
1. L'utilisateur voit un récapitulatif (durée, participants)
2. S'il a consenti à l'enregistrement :
   - L'audio est envoyé au feedback-service
   - Il recevra un feedback IA sous quelques minutes
3. Il peut noter son expérience (optionnel)
```

---

## 4. Modèle de Données

### 4.1 TimeSlot (Créneau)

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| targetLanguageCode | String | Oui | Langue pratiquée (ex: "en") |
| level | Enum | Oui | Niveau CECR : A1, A2, B1, B2, C1, C2 |
| startTime | DateTime | Oui | Heure de début |
| durationMinutes | Number | Oui | Durée : 15, 30 ou 45 |
| maxParticipants | Number | Oui | Capacité max (défaut: 8) |
| minParticipants | Number | Oui | Minimum pour démarrer (défaut: 2) |
| recurrence | Enum | Non | `daily`, `weekly`, `once` |
| isActive | Boolean | Oui | Créneau disponible |
| createdAt | DateTime | Oui | Date de création |

### 4.2 Registration (Inscription)

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| timeSlotId | String | Oui | Référence au créneau |
| userId | String | Oui | Utilisateur inscrit |
| status | Enum | Oui | `registered`, `cancelled`, `attended`, `noshow` |
| registeredAt | DateTime | Oui | Date d'inscription |
| cancelledAt | DateTime | Non | Date d'annulation |

### 4.3 Session

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| timeSlotId | String | Oui | Référence au créneau |
| targetLanguageCode | String | Oui | Langue pratiquée |
| level | Enum | Oui | Niveau CECR |
| status | Enum | Oui | `waiting`, `active`, `ended` |
| startedAt | DateTime | Non | Heure de début effective |
| endedAt | DateTime | Non | Heure de fin |
| recordingEnabled | Boolean | Oui | Enregistrement actif |
| recordings | Array | Non | Liste des enregistrements par participant |
| createdAt | DateTime | Oui | Date de création |

### 4.4 Participant

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| sessionId | String | Oui | Référence à la session |
| userId | String | Oui | Référence à l'utilisateur |
| status | Enum | Oui | `waiting`, `connected`, `disconnected` |
| cameraEnabled | Boolean | Oui | État de la caméra |
| micEnabled | Boolean | Oui | État du micro |
| recordingConsent | Boolean | Oui | Consentement à l'enregistrement |
| joinedAt | DateTime | Oui | Heure de connexion |
| leftAt | DateTime | Non | Heure de déconnexion |

---

## 5. Règles Métier

### 5.1 Gestion des Créneaux

**Création des créneaux :**
- Créés automatiquement par un job planifié (ou manuellement par admin)
- Créneaux générés 7 jours à l'avance
- Horaires adaptés aux fuseaux horaires principaux

**Inscription :**
- Un utilisateur peut s'inscrire jusqu'à 5 minutes avant le début
- Maximum 3 inscriptions simultanées par utilisateur
- Annulation possible jusqu'à 15 minutes avant

### 5.2 Matchmaking

**Algorithme simple :**
```
1. Pour chaque créneau avec inscriptions :
   a. Grouper les inscrits par langue + niveau
   b. Créer des sessions de 2-8 participants
   c. Si moins de 2 participants → session annulée
   d. Notifier les participants de leur session
```

**Critères :**
- Même langue cible (obligatoire)
- Même niveau CECR (pas de tolérance ±1 pour simplifier)

### 5.3 Gestion des Sessions

**Démarrage :**
- La session démarre à l'heure du créneau
- Les participants ont 5 min de grâce pour rejoindre
- Si < 2 participants après 5 min → session annulée

**Pendant la session :**
- Chaque participant contrôle son micro/caméra
- Le signaling WebRTC gère les connexions P2P
- Timer visible pour tous

**Fin de session :**
- Automatique à la fin du créneau
- Ou si tous les participants quittent

### 5.4 Enregistrement Audio

**Consentement :**
- Chaque participant choisit avant de rejoindre
- L'enregistrement démarre si AU MOINS 1 participant consent
- Seuls les audios des participants consentants sont capturés

**Processus :**
```
1. Session démarre → recording initialisé si consentement
2. Audio mixé en temps réel (côté serveur ou client)
3. Session termine → fichiers uploadés sur R2 (1 par participant)
4. Événement envoyé au feedback-service avec URL
5. Feedback-service transcrit et analyse
6. Résultat stocké et notifié à l'utilisateur
```

**Rétention :**
- Enregistrements conservés 30 jours
- Suppression automatique ensuite

### 5.5 Signaling WebRTC

```
1. Participant rejoint → reçoit liste des autres
2. Établissement des connexions peer-to-peer
3. Échange SDP offers/answers via WebSocket
4. Échange ICE candidates
5. Connexions directes établies
```

---

## 6. Interactions avec Autres Services

### 6.1 Services Appelés

| Service | Raison |
|---------|--------|
| auth-service | Valider JWT, récupérer profil et niveau |

### 6.2 Événements Publiés

| Événement | Quand | Consommateurs |
|-----------|-------|---------------|
| session.started | Session démarre | gamification-service |
| session.ended | Session termine | gamification-service |
| session.recorded | Enregistrement disponible | feedback-service |

### 6.3 Événements Consommés

| Événement | Source | Action |
|-----------|--------|--------|
| user.deleted | auth-service | Supprimer inscriptions et historique |

---

## 7. Points d'Attention

### 7.1 Performance
- MongoDB suffit pour ce service (pas de Redis)
- Indexes sur timeSlotId, userId, status
- Le signaling WebRTC doit être < 100ms

### 7.2 Résilience
- Si un participant perd sa connexion → 30s pour se reconnecter
- Sessions orphelines nettoyées après 5 min d'inactivité
- Enregistrements uploadés de manière asynchrone

### 7.3 Limites
- Max 8 participants par session
- Max 3 inscriptions actives par utilisateur
- Max 10 sessions par jour (free tier)

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
