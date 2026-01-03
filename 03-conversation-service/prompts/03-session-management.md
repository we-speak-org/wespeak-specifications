# User Story: Gestion des Sessions

## Contexte
Quand un créneau démarre, une session est créée pour regrouper les participants inscrits.

## Objectif
Implémenter la création et gestion des sessions de conversation.

## Entités concernées
- **Session** : Session de conversation liée à un créneau
- **Participant** : Utilisateur participant à la session

## Fonctionnalités à implémenter

### 1. Création automatique de session
- À l'heure du créneau, créer une session pour les inscrits
- Grouper par langue + niveau (même créneau)
- Statut initial : `waiting`

### 2. API REST Sessions
- `POST /api/v1/conversations/sessions/join` : Rejoindre (avec timeSlotId et recordingConsent)
- `GET /api/v1/conversations/sessions/current` : Ma session active
- `PATCH /api/v1/conversations/sessions/current/media` : Changer état caméra/micro
- `POST /api/v1/conversations/sessions/current/leave` : Quitter
- `GET /api/v1/conversations/sessions/history` : Historique

### 3. Cycle de vie
- `waiting` → `active` : Quand 2+ participants connectés
- `active` → `ended` : Fin du créneau OU tous partis
- Grace period : 5 min pour rejoindre après le début

### 4. Participant
- Gère l'état : `waiting`, `connected`, `disconnected`
- Stocke : cameraEnabled, micEnabled, recordingConsent
- Enregistre joinedAt, leftAt

### 5. Règles métier
- Session annulée si < 2 participants après 5 min
- Max 8 participants par session
- Durée = durée du créneau

## Critères d'acceptation
- [ ] La session est créée automatiquement à l'heure du créneau
- [ ] Seuls les inscrits peuvent rejoindre
- [ ] Le statut passe à `active` avec 2+ participants
- [ ] L'état média (caméra/micro) est mis à jour en temps réel
- [ ] La session se termine automatiquement à la fin du créneau

## Stack technique
- Spring Boot 4, MongoDB
- WebSocket pour les mises à jour temps réel
