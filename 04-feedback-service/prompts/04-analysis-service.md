# User Story 04 - Service d'Analyse IA

## Contexte
Tu travailles sur le **feedback-service** du projet WeSpeak. Tu dois implémenter le service qui analyse les transcriptions via un LLM (Claude/GPT) et génère des feedbacks personnalisés.

## User Story
**En tant que** apprenant
**Je veux** recevoir un feedback détaillé après chaque conversation
**Afin de** comprendre mes erreurs et m'améliorer

## Tâches

1. **Créer AnalysisService** avec les méthodes :
   - `analyzeTranscript(Transcript transcript)` : Feedback
   - `generateFeedback(String transcriptContent, String language, String userLevel)` : LLMAnalysisResult

2. **Créer le prompt pour le LLM** :
   ```
   Tu es un professeur de langues expert. Analyse cette transcription d'un 
   apprenant de niveau {level} en {language}.
   
   Transcription :
   {content}
   
   Fournis une analyse détaillée au format JSON avec :
   - overallScore (0-100)
   - grammarScore (0-100)
   - vocabularyScore (0-100)
   - fluencyScore (0-100)
   - errors : liste d'erreurs avec type, original, correction, explanation, severity
   - strengths : liste de 3 points forts
   - improvements : liste de 3 conseils d'amélioration
   - summary : résumé de 2-3 phrases
   
   Limite les erreurs aux 10 plus importantes.
   ```

3. **Créer LLMClient** pour appeler Claude/GPT :
   - Configurable via properties (API key, model)
   - Température : 0.3
   - Timeout : 60 secondes

4. **Créer LLMAnalysisResult** (DTO pour parser la réponse) :
   - Tous les champs du feedback

5. **Implémenter le calcul des XP** :
   ```java
   int xp = 10; // participation
   if (overallScore >= 60) xp += 5;
   if (overallScore >= 80) xp += 10;
   if (isImproving) xp += 5;
   if (duration >= 600) xp += 5;  // 10 min
   if (duration >= 1200) xp += 10; // 20 min
   return Math.min(xp, 40); // max 40 XP
   ```

6. **Mettre à jour UserFeedbackStats** après chaque feedback :
   - Incrémenter totalSessions
   - Ajouter les minutes
   - Recalculer les moyennes
   - Mettre à jour commonErrors
   - Calculer progressTrend (basé sur 5 dernières sessions)

7. **Publier les événements** :
   - `feedback.generated`
   - `xp.awarded` (vers gamification-service)

## Critères d'Acceptation

- [ ] Le LLM est appelé avec le bon prompt
- [ ] La réponse JSON est parsée correctement
- [ ] Le Feedback est créé avec tous les champs
- [ ] Les XP sont calculés selon les règles
- [ ] Les UserFeedbackStats sont mises à jour
- [ ] Les événements Kafka sont publiés
- [ ] Les erreurs LLM sont gérées (retry, fallback)

## Notes Techniques

- Parser le JSON du LLM avec ObjectMapper
- Si le parsing échoue, retenter avec un prompt plus strict
- Logger les réponses brutes du LLM pour debug
- Température basse (0.3) pour des réponses plus cohérentes
