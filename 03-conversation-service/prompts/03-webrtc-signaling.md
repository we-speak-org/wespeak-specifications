# User Story: Signalisation WebRTC Multi-Participants

## Contexte
En tant qu'apprenant dans une session de groupe (2-6 personnes), je veux établir des connexions audio/vidéo peer-to-peer avec tous les autres participants pour converser en temps réel.

## Critères d'Acceptation

### AC1: Connexion WebSocket
- Endpoint WS /ws/signaling?token={jwt}&sessionId={sessionId}
- Authentification via JWT dans le query param
- Vérification que l'utilisateur est participant de la session
- Déconnexion propre si token invalide ou non-participant

### AC2: Arrivée d'un participant
- Quand un utilisateur se connecte au WebSocket, notifier tous les autres:
  ```json
  { "type": "participant-joined", "participant": { "userId": "...", "displayName": "..." } }
  ```
- Envoyer au nouvel arrivant la liste des participants déjà connectés

### AC3: Établissement des connexions P2P
- Chaque nouveau participant initie une connexion avec chaque participant existant
- Le nouveau envoie une "offer" à chaque participant existant
- Chaque participant existant répond avec une "answer"
- Mesh complet: N participants = N*(N-1)/2 connexions

### AC4: Échange SDP
- L'initiateur envoie `{ type: "offer", toUserId: "...", sdp: "..." }`
- Le serveur relaye au destinataire avec fromUserId ajouté
- Le destinataire répond avec `{ type: "answer", toUserId: "...", sdp: "..." }`

### AC5: Échange ICE Candidates
- Chaque client envoie ses candidats ICE ciblés:
  `{ type: "ice-candidate", toUserId: "...", candidate: {...} }`
- Le serveur relaye au destinataire

### AC6: Départ d'un participant
- Quand un participant quitte ou se déconnecte:
  ```json
  { "type": "participant-left", "userId": "..." }
  ```
- Les autres participants ferment leur connexion P2P avec lui

### AC7: Session démarrée/terminée
- Quand le host démarre la session:
  ```json
  { "type": "session-started", "startedAt": "..." }
  ```
- Quand la session se termine:
  ```json
  { "type": "session-ended", "reason": "completed|timeout|host_ended" }
  ```

### AC8: Heartbeat
- Chaque client envoie un heartbeat toutes les 15 secondes
- Si pas de heartbeat pendant 30s, considérer déconnecté
- Notifier les autres participants de la déconnexion

## Tâches Techniques

1. Configurer Spring WebSocket
2. Créer WebSocketAuthInterceptor pour valider JWT et sessionId
3. Créer SignalingHandler pour gérer les messages:
   - handleConnect(sessionId, userId): Notifier et envoyer liste
   - handleOffer(toUserId, sdp, fromUserId)
   - handleAnswer(toUserId, sdp, fromUserId)
   - handleIceCandidate(toUserId, candidate, fromUserId)
   - handleDisconnect(sessionId, userId)
4. Stocker le mapping userId -> WebSocket session dans Redis
5. Créer HeartbeatMonitor pour détecter les déconnexions
6. Gérer la fermeture propre des connexions

## Structure Redis

### Participants connectés à une session
```
Key: session:ws:{sessionId}
Type: Hash
Fields:
  {userId1}: websocket-session-id
  {userId2}: websocket-session-id
  ...
TTL: 1800 seconds
```

### Heartbeats
```
Key: session:heartbeat:{sessionId}:{userId}
Type: String
Value: timestamp du dernier heartbeat
TTL: 45 seconds (auto-expire si pas de heartbeat)
```

## Definition of Done
- [ ] Connexion WebSocket multi-clients
- [ ] Mesh P2P établi entre tous les participants
- [ ] Échange SDP point-à-point fonctionnel
- [ ] ICE candidates relayés correctement
- [ ] Détection de déconnexion en <30s
- [ ] Tests avec 4 clients simultanés
