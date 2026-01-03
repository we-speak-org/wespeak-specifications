# User Story: Signaling WebRTC

## Contexte
Les participants doivent établir des connexions peer-to-peer via WebRTC pour la vidéo/audio.

## Objectif
Implémenter le serveur de signaling WebSocket pour coordonner les connexions WebRTC.

## Fonctionnalités à implémenter

### 1. Connexion WebSocket
- Endpoint : `WS /ws/signaling?token={jwt}&sessionId={sessionId}`
- Authentification via JWT
- Vérifier que l'utilisateur est participant de la session

### 2. Messages client → serveur
- `offer` : Offre SDP envoyée à un autre participant
- `answer` : Réponse SDP
- `ice-candidate` : Candidat ICE pour établir la connexion

### 3. Messages serveur → client
- `participant-joined` : Un participant a rejoint
- `participant-left` : Un participant est parti
- `media-state-changed` : Un participant a changé son état caméra/micro
- `session-ended` : La session est terminée
- `offer`, `answer`, `ice-candidate` : Relais des messages de signaling

### 4. Gestion des connexions
- Maintenir la liste des WebSockets actifs par session
- Nettoyer les connexions fermées
- Heartbeat pour détecter les déconnexions

### 5. Mesh topology
- Chaque participant se connecte à tous les autres
- Le serveur relaie les messages de signaling
- Pas de serveur média (SFU) dans le MVP

## Critères d'acceptation
- [ ] La connexion WebSocket requiert un JWT valide
- [ ] Les messages de signaling sont relayés aux bons destinataires
- [ ] Les événements participant-joined/left sont diffusés
- [ ] Les changements d'état média sont diffusés
- [ ] La déconnexion est détectée et notifiée

## Stack technique
- Spring Boot WebSocket
- JSON pour les messages
