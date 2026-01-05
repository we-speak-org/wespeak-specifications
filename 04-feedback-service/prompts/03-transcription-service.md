# User Story 03 - Service de Transcription

## Contexte
Tu travailles sur le **feedback-service** du projet WeSpeak. Tu dois implémenter le service qui traite les enregistrements audio et les transcrit via l'API Whisper.

## User Story
**En tant que** système
**Je veux** transcrire automatiquement les enregistrements audio
**Afin de** pouvoir analyser le contenu des conversations

## Tâches

1. **Créer le Kafka Listener** pour l'événement `recording.uploaded` :
   ```java
   @Bean
   public Consumer<CloudEvent<RecordingUploadedPayload>> recordingUploadedListener(
           TranscriptionService transcriptionService) {
       return event -> transcriptionService.processRecording(event.getData());
   }
   ```

2. **Créer RecordingUploadedPayload** :
   - `recordingId` : String
   - `sessionId` : String
   - `participantId` : String
   - `targetLanguageCode` : String
   - `recordings` : List<{userId, url, startTime}>
   - `duration` : Integer
   - `format` : String

3. **Créer TranscriptionService** avec les méthodes :
   - `processRecording(RecordingUploadedPayload payload)` : void
   - `transcribeAudio(String audioUrl, String language)` : WhisperResponse

4. **Implémenter le pipeline de transcription** :
   ```
   1. Créer un Transcript en status PENDING
   2. Pour chaque enregistrement :
      a. Télécharger l'audio depuis R2
      b. Appeler Whisper API
      c. Récupérer les segments
   3. Fusionner et trier tous les segments par timestamp
   4. Mettre à jour le Transcript en COMPLETED
   6. Publier l'événement transcript.completed
   7. Déclencher l'analyse IA
   ```

5. **Créer un client R2 (S3 compatible)** simplifié :
   - `downloadFile(String url)` : byte[]

6. **Créer un client Whisper** :
   - URL : configurable via properties
   - Modèle : `whisper-1`
   - Format de sortie : segments avec timestamps

7. **Gérer les erreurs** :
   - Retry automatique (3 tentatives)
   - En cas d'échec final : status FAILED + publier transcript.failed

8. **Publier les événements** :
   - `transcript.completed` en cas de succès
   - `transcript.failed` en cas d'échec

## Critères d'Acceptation

- [ ] Le listener Kafka reçoit les événements recording.uploaded
- [ ] Le Transcript est créé et mis à jour correctement
- [ ] L'audio est récupéré depuis Cloudflare R2
- [ ] L'API Whisper est appelée correctement
- [ ] Les segments sont parsés et sauvegardés
- [ ] Les événements sont publiés sur Kafka
- [ ] Les erreurs sont gérées avec retry

## Notes Techniques

- Utiliser WebClient pour les appels HTTP
- API Whisper : POST /v1/audio/transcriptions
- Timeout : 120 secondes (transcription peut être longue)
- Log chaque étape du pipeline
