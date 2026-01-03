# User Story: Système de Matchmaking

## Contexte
En tant qu'apprenant, je veux pouvoir rejoindre une file d'attente pour trouver automatiquement un partenaire de conversation de niveau compatible.

## Critères d'Acceptation

### AC1: Rejoindre la file
- POST /matchmaking/join ajoute l'utilisateur à la file Redis
- L'utilisateur spécifie: learningProfileId, preferredDuration, recordingConsent
- Optionnellement: preferredTopicId
- Erreur 409 si déjà en file ou en session active

### AC2: Algorithme de match
- Recherche dans l'ordre: même niveau > niveau -1 > niveau +1
- Critères obligatoires: même langue cible
- Préférence: même topic si spécifié
- Préférence: durée compatible (±5 min)

### AC3: Timeout
- Expiration après 2 minutes sans match
- Notification au client via WebSocket
- Nettoyage automatique de la file

### AC4: Annulation
- DELETE /matchmaking/leave retire l'utilisateur de la file
- Réponse 200 même si pas en file

## Tâches Techniques

1. Configurer Redis avec Spring Data Redis
2. Créer MatchmakingRequest entity (MongoDB pour historique)
3. Créer MatchmakingService avec:
   - joinQueue(userId, request): Ajouter à Redis sorted set
   - findMatch(userId): Scanner les files compatibles
   - leaveQueue(userId): Retirer de la file
4. Structure Redis:
   - Sorted Set `matchmaking:{lang}:{level}` avec score = timestamp
   - TTL de 120 secondes sur les entrées
5. Créer MatchmakingController
6. Implémenter le scheduler pour cleanup des requêtes expirées
7. Tests unitaires et d'intégration

## Logique de Compatibilité
```
Niveau utilisateur -> Files à scanner (par priorité)
A1 -> [A1, A2]
A2 -> [A2, A1, B1]
B1 -> [B1, A2, B2]
B2 -> [B2, B1, C1]
C1 -> [C1, B2, C2]
C2 -> [C2, C1]
```

## Definition of Done
- [ ] File Redis fonctionnelle
- [ ] Matching en moins de 100ms
- [ ] Timeout géré correctement
- [ ] Tests avec plusieurs utilisateurs simultanés
