# User Story: Gestion des Sessions Multi-Participants

## Contexte
En tant qu'apprenant, je veux pouvoir participer à des sessions de conversation avec plusieurs personnes (2 à 6), pour pratiquer dans un contexte plus dynamique et réaliste.

## Critères d'Acceptation

### AC1: Création de session publique (via matchmaking)
- Une session est créée quand le matchmaking trouve 2+ utilisateurs compatibles
- Statut initial: "waiting"
- Type: "public"
- Champs: targetLanguageCode, topicId (optionnel), hostUserId (premier matché)

### AC2: Création de session privée
- POST /sessions avec type="private" crée une session avec code d'invitation
- L'utilisateur créateur devient host
- inviteCode généré automatiquement (6 caractères alphanumériques)
- maxParticipants configurable (2-6, défaut: 4)

### AC3: Rejoindre une session
- POST /sessions/join avec inviteCode
- Vérifie que la session n'est pas pleine (participantCount < maxParticipants)
- Vérifie que la session est en statut "waiting" ou "active"
- Crée un Participant avec role="participant", status="waiting"

### AC4: Participants
- Chaque Participant a: sessionId, userId, role, status, joinedAt, leftAt, speakingTimeSeconds
- Roles: "host" (créateur), "participant" (autres)
- Status: "waiting" (pas encore connecté WebRTC), "connected", "disconnected"

### AC5: Démarrage de session
- Session publique: démarre automatiquement quand minParticipants atteint
- Session privée: POST /sessions/{id}/start par le host uniquement
- startedAt est enregistré, status -> "active"

### AC6: Quitter une session
- POST /sessions/{id}/leave
- Met à jour leftAt et speakingTimeSeconds du participant
- Publie événement participant.left
- Si c'était le host et session privée -> le plus ancien participant devient host

### AC7: Terminer une session
- Host peut terminer avec POST /sessions/{id}/end
- Ou automatiquement si tous les participants sont partis
- Ou après 30 minutes (timeout)
- endedAt enregistré, status -> "ended"
- Événement session.ended publié avec tous les participants et leurs stats

### AC8: Historique
- GET /sessions/history retourne les sessions de l'utilisateur
- Inclut: topicTitle, participantCount, duration, status

### AC9: Limites
- Max 3 sessions actives simultanées par utilisateur
- Free tier: max 10 sessions/jour
- Premium: illimité
- Erreur 403 si dépassé

## Tâches Techniques

1. Créer Session document MongoDB
2. Créer Participant document MongoDB
3. Créer SessionRepository avec:
   - findByStatusAndTargetLanguageCode
   - findByInviteCode
   - findByHostUserId
4. Créer ParticipantRepository avec:
   - findBySessionId
   - findByUserIdAndLeftAtIsNull (sessions actives)
   - countByUserIdAndJoinedAtAfter (comptage quotidien)
5. Créer SessionService avec:
   - createPublicSession(matchedUserIds, targetLanguageCode)
   - createPrivateSession(hostUserId, options)
   - joinSession(inviteCode, userId)
   - startSession(sessionId, userId) - vérifie host
   - leaveSession(sessionId, userId)
   - endSession(sessionId, userId) - vérifie host
   - getParticipants(sessionId)
   - getCurrentSession(userId)
   - getHistory(userId, pagination)
6. Créer SessionController avec endpoints REST
7. Publier événements Kafka

## Definition of Done
- [ ] Sessions multi-participants fonctionnelles
- [ ] Système host/participant opérationnel
- [ ] Sessions publiques et privées
- [ ] Événements Kafka publiés
- [ ] Tests d'intégration complets
