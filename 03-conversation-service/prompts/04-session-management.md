# User Story: Gestion des Sessions

## Contexte
En tant qu'apprenant, je veux que ma session de conversation soit correctement gérée du début à la fin, avec enregistrement audio et historique consultable.

## Critères d'Acceptation

### AC1: Création de session
- Une session est créée automatiquement quand un match est trouvé
- Statut initial: "waiting"
- Champs: topicId, targetLanguageCode, participant1Id, participant2Id

### AC2: Démarrage de session
- Quand les deux participants sont connectés en WebRTC, statut -> "active"
- startedAt est enregistré
- Timer démarre côté client

### AC3: Fin de session normale
- POST /sessions/{sessionId}/end avec reason="completed"
- endedAt et actualDurationSeconds enregistrés
- Si durée >= 2 minutes:
  - Audio uploadé vers S3
  - Événement session.completed publié
  - XP calculé et retourné
- Si durée < 2 minutes:
  - Session marquée completed mais pas d'XP

### AC4: Fin anormale
- Déconnexion détectée -> status="cancelled", endReason="dropped"
- Timeout inactivité -> status="cancelled", endReason="timeout"
- Événement session.cancelled publié

### AC5: Historique
- GET /sessions/history retourne les sessions de l'utilisateur
- Pagination avec page et limit
- Inclut: topicTitle, partnerDisplayName, duration, hasFeedback

### AC6: Limites quotidiennes
- Utilisateur free: max 3 sessions/jour
- Utilisateur premium: illimité
- Erreur 403 DAILY_LIMIT_REACHED si dépassé

## Tâches Techniques

1. Créer ConversationSession entity avec tous les champs
2. Créer SessionRepository avec:
   - findByParticipantAndDateRange (pour compter sessions/jour)
   - findByParticipantOrderByCreatedAtDesc (historique)
3. Créer SessionService avec:
   - createSession(match): Créer la session
   - startSession(sessionId): Marquer comme active
   - endSession(sessionId, reason): Terminer proprement
   - getActiveSession(userId): Session en cours
   - getHistory(userId, page, limit): Historique paginé
4. Créer SessionController avec les endpoints REST
5. Intégrer avec S3 pour upload audio
6. Publier événements Kafka (session.completed, session.cancelled)
7. Vérifier limites via auth-service (subscription tier)

## Calcul XP
```
Base XP = 30
Bonus durée = (actualDurationSeconds / 60) * 5  // 5 XP par minute
Total = Base + Bonus (max 100 XP)
```

## Definition of Done
- [ ] CRUD sessions fonctionnel
- [ ] Upload S3 opérationnel
- [ ] Événements Kafka publiés
- [ ] Limites quotidiennes respectées
- [ ] Tests d'intégration complets
