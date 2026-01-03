# User Story 05 - API REST Feedbacks

## Contexte
Tu travailles sur le **feedback-service** du projet WeSpeak. Tu dois créer l'API REST pour permettre aux utilisateurs de consulter leurs feedbacks et statistiques.

## User Story
**En tant que** apprenant
**Je veux** consulter mes feedbacks et ma progression
**Afin de** suivre mon évolution dans l'apprentissage

## Tâches

1. **Créer FeedbackController** avec les endpoints :

   ```
   GET /api/v1/feedback/transcripts/{transcriptId}
   GET /api/v1/feedback/transcripts?sessionId={sessionId}
   
   GET /api/v1/feedback/feedbacks/{feedbackId}
   GET /api/v1/feedback/feedbacks/me?targetLanguageCode={lang}&page={p}&size={s}
   GET /api/v1/feedback/feedbacks/session/{sessionId}
   
   GET /api/v1/feedback/stats/me?targetLanguageCode={lang}
   GET /api/v1/feedback/stats/me/history?targetLanguageCode={lang}&period={period}
   ```

2. **Créer les DTOs de réponse** :
   - TranscriptResponse
   - FeedbackResponse
   - FeedbackListItem (version allégée pour liste)
   - FeedbackListResponse (avec pagination)
   - UserStatsResponse
   - ProgressHistoryResponse

3. **Implémenter la sécurité** :
   - Extraire l'userId du JWT
   - Vérifier que l'utilisateur accède uniquement à ses propres données
   - Retourner 403 si tentative d'accès aux données d'autrui

4. **Implémenter la pagination** pour la liste des feedbacks :
   - Page par défaut : 0
   - Size par défaut : 10
   - Size max : 50

5. **Créer FeedbackService** (lecture) avec :
   - `getTranscript(String transcriptId, String userId)` : TranscriptResponse
   - `getTranscriptsBySession(String sessionId, String userId)` : List<TranscriptResponse>
   - `getFeedback(String feedbackId, String userId)` : FeedbackResponse
   - `getMyFeedbacks(String userId, String lang, Pageable pageable)` : Page<FeedbackListItem>
   - `getFeedbackBySession(String sessionId, String userId)` : FeedbackResponse

6. **Créer StatsService** avec :
   - `getMyStats(String userId, String lang)` : UserStatsResponse
   - `getMyHistory(String userId, String lang, String period)` : ProgressHistoryResponse

7. **Gérer les erreurs** :
   - 404 si ressource non trouvée
   - 403 si accès non autorisé
   - Messages d'erreur clairs

## Critères d'Acceptation

- [ ] Tous les endpoints sont implémentés
- [ ] La pagination fonctionne correctement
- [ ] L'utilisateur ne peut voir que ses propres données
- [ ] Les DTOs sont correctement mappés
- [ ] Les erreurs retournent les bons codes HTTP
- [ ] Les endpoints sont documentés (Swagger/OpenAPI)

## Exemples de Requêtes

```bash
# Mes feedbacks en anglais
GET /api/v1/feedback/feedbacks/me?targetLanguageCode=en&page=0&size=10
Authorization: Bearer eyJhbGc...

# Détail d'un feedback
GET /api/v1/feedback/feedbacks/abc123-uuid
Authorization: Bearer eyJhbGc...

# Mes statistiques
GET /api/v1/feedback/stats/me?targetLanguageCode=en
Authorization: Bearer eyJhbGc...

# Mon historique sur le mois
GET /api/v1/feedback/stats/me/history?targetLanguageCode=en&period=MONTH
Authorization: Bearer eyJhbGc...
```

## Notes Techniques

- Utiliser @AuthenticationPrincipal pour extraire l'utilisateur
- Mapper les entités vers les DTOs (MapStruct ou manuel)
- Optimiser les requêtes MongoDB avec projections si nécessaire
