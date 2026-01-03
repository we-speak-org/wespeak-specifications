# US-003: Recommandation de Session Conversation

## Contexte
L'utilisateur doit être encouragé à participer aux sessions de conversation.

## User Story
**En tant qu'** apprenant  
**Je veux** voir les sessions de conversation disponibles qui me correspondent  
**Afin de** pratiquer l'oral avec d'autres apprenants

## Critères d'Acceptation

1. **Récupération des créneaux**
   - Appeler conversation-service pour obtenir les slots disponibles
   - Filtrer par langue et niveau de l'utilisateur

2. **Filtrage par préférences**
   - Respecter UserPreferences.preferredLearningTime
   - Ne suggérer que les créneaux des prochaines 24h

3. **Priorisation**
   - Prioriser les créneaux avec places restantes
   - Bonus si des utilisateurs de niveau similaire sont inscrits

4. **Création de la recommandation**
   - Type: "conversation"
   - TargetType: "slot"
   - Priority: 3
   - Title: "Session conversation - {heure}"
   - Reason: "Pratiquez avec d'autres apprenants"
   - Metadata: startTime, participantsCount, maxParticipants

5. **Limites**
   - Maximum 2 recommandations de conversation actives
   - Supprimer si le créneau est passé ou complet

## Appel API vers Conversation Service

```
GET /api/v1/slots/available?language={lang}&level={level}&from={now}&to={+24h}
```

## Tests Requis
- [ ] Test: recommandation créée pour slot compatible
- [ ] Test: filtrage par préférences horaires
- [ ] Test: pas de recommandation si aucun slot
- [ ] Test: suppression si slot complet
