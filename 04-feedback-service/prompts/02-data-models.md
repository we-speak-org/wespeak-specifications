# User Story 02 - Modèles de Données

## Contexte
Tu travailles sur le **feedback-service** du projet WeSpeak. Tu dois créer les modèles MongoDB pour stocker les transcriptions et feedbacks.

## User Story
**En tant que** développeur
**Je veux** créer les modèles de données MongoDB
**Afin de** pouvoir persister les transcripts et feedbacks

## Tâches

1. **Créer l'entité Transcript** avec les champs :
   - `id` : String (UUID)
   - `sessionId` : String (obligatoire)
   - `participantId` : String (obligatoire)
   - `recordingId` : String (obligatoire)
   - `targetLanguageCode` : String (obligatoire)
   - `content` : String (texte complet)
   - `segments` : List<TranscriptSegment> (embedded)
   - `duration` : Integer (secondes)
   - `wordCount` : Integer
   - `confidence` : Double (0.0 - 1.0)
   - `status` : TranscriptStatus enum (PENDING, PROCESSING, COMPLETED, FAILED)
   - `createdAt` : Instant
   - `completedAt` : Instant (nullable)

2. **Créer la classe TranscriptSegment** (embedded) :
   - `startTime` : Double
   - `endTime` : Double
   - `text` : String
   - `confidence` : Double

3. **Créer l'entité Feedback** avec les champs :
   - `id` : String (UUID)
   - `transcriptId` : String (obligatoire)
   - `userId` : String (obligatoire)
   - `sessionId` : String (obligatoire)
   - `targetLanguageCode` : String (obligatoire)
   - `overallScore` : Integer (0-100)
   - `grammarScore` : Integer (0-100)
   - `vocabularyScore` : Integer (0-100)
   - `fluencyScore` : Integer (0-100)
   - `pronunciationScore` : Integer (nullable)
   - `errors` : List<FeedbackError> (embedded)
   - `strengths` : List<String>
   - `improvements` : List<String>
   - `summary` : String
   - `xpAwarded` : Integer
   - `status` : FeedbackStatus enum
   - `createdAt` : Instant
   - `completedAt` : Instant (nullable)

4. **Créer la classe FeedbackError** (embedded) :
   - `type` : ErrorType enum (GRAMMAR, VOCABULARY, PRONUNCIATION, SYNTAX)
   - `original` : String
   - `correction` : String
   - `explanation` : String
   - `severity` : ErrorSeverity enum (LOW, MEDIUM, HIGH)
   - `segmentIndex` : Integer

5. **Créer l'entité UserFeedbackStats** :
   - `id` : String
   - `userId` : String
   - `targetLanguageCode` : String
   - `totalSessions` : Integer
   - `totalMinutes` : Integer
   - `averageOverallScore` : Double
   - `averageGrammarScore` : Double
   - `averageVocabularyScore` : Double
   - `averageFluencyScore` : Double
   - `commonErrors` : List<CommonError>
   - `progressTrend` : ProgressTrend enum (IMPROVING, STABLE, DECLINING)
   - `lastFeedbackAt` : Instant
   - `updatedAt` : Instant

6. **Créer les repositories** :
   - TranscriptRepository
   - FeedbackRepository
   - UserFeedbackStatsRepository

## Critères d'Acceptation

- [ ] Toutes les entités sont annotées avec `@Document`
- [ ] Les indexes sont définis sur sessionId, participantId, userId, targetLanguageCode
- [ ] Les enums sont créés
- [ ] Les repositories étendent MongoRepository
- [ ] Index composé sur (userId, targetLanguageCode) pour UserFeedbackStats

## Notes Techniques

- Utiliser `@Id` de Spring Data MongoDB
- Utiliser `@CreatedDate` et `@LastModifiedDate` où approprié
- Utiliser Lombok pour réduire le boilerplate
