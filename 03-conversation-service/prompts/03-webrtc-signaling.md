# User Story: Signalisation WebRTC

## Contexte
En tant qu'apprenant, une fois matché avec un partenaire, je veux établir une connexion audio/vidéo peer-to-peer pour converser en temps réel.

## Critères d'Acceptation

### AC1: Connexion WebSocket
- Endpoint WS /ws/signaling?token={jwt}
- Authentification via JWT dans le query param
- Déconnexion propre si token invalide

### AC2: Notification de match
- Quand un match est trouvé, les deux utilisateurs reçoivent:
  ```json
  { "type": "match-found", "sessionId": "...", "partnerId": "...", "isInitiator": true/false }
  ```
- L'initiateur (premier inscrit) crée l'offre SDP

### AC3: Échange SDP
- L'initiateur envoie `{ type: "offer", sessionId, sdp }`
- Le serveur relaye au partenaire
- Le partenaire répond avec `{ type: "answer", sessionId, sdp }`
- Le serveur relaye à l'initiateur

### AC4: Échange ICE Candidates
- Chaque client envoie ses candidats ICE au serveur
- Le serveur relaye au partenaire
- Format: `{ type: "ice-candidate", sessionId, candidate }`

### AC5: Heartbeat
- Chaque client envoie un heartbeat toutes les 10 secondes
- Le serveur met à jour le timestamp dans Redis
- Si pas de heartbeat pendant 30s, considérer déconnecté

### AC6: Notifications de déconnexion
- Si un participant se déconnecte, notifier l'autre:
  ```json
  { "type": "partner-disconnected", "sessionId": "..." }
  ```

## Tâches Techniques

1. Configurer Spring WebSocket avec STOMP ou raw WebSocket
2. Créer WebSocketAuthInterceptor pour valider le JWT
3. Créer SignalingHandler pour gérer les messages:
   - handleOffer(sessionId, sdp, senderId)
   - handleAnswer(sessionId, sdp, senderId)
   - handleIceCandidate(sessionId, candidate, senderId)
   - handleHeartbeat(sessionId, userId)
4. Stocker le mapping userId -> WebSocket session
5. Stocker l'état de session dans Redis (participants, heartbeats)
6. Créer HeartbeatMonitor (scheduled task) pour détecter les déconnexions
7. Tests avec client WebSocket simulé

## Structure Redis pour Session Active
```
Key: session:active:{sessionId}
Type: Hash
Fields:
  participant1Id: string
  participant2Id: string  
  participant1Ws: websocket-session-id
  participant2Ws: websocket-session-id
  lastHeartbeat1: timestamp
  lastHeartbeat2: timestamp
  status: waiting|active
```

## Definition of Done
- [ ] Connexion WebSocket stable
- [ ] Échange SDP fonctionnel
- [ ] ICE candidates relayés correctement
- [ ] Détection de déconnexion en <30s
- [ ] Tests avec 2 clients simultanés
