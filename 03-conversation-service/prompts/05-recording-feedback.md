# User Story: Enregistrement Audio et Intégration Feedback

## Contexte
Les participants peuvent consentir à l'enregistrement audio de la session pour recevoir un feedback IA.

## Objectif
Implémenter l'enregistrement audio opt-in et l'envoi au feedback-service.

## Fonctionnalités à implémenter

### 1. Consentement
- Chaque participant indique `recordingConsent` au moment de rejoindre
- L'enregistrement démarre si AU MOINS 1 participant consent
- Seuls les audios des participants consentants sont capturés

### 2. Capture audio
- Enregistrer le flux audio mixé des participants consentants
- Format : WebM ou MP3
- Qualité : suffisante pour transcription (16kHz mono minimum)

### 3. Upload S3
- À la fin de la session, uploader l'enregistrement vers S3
- Chemin : `s3://wespeak-recordings/{year}/{month}/{day}/{sessionId}.webm`
- Générer une URL signée (expiration 24h)

### 4. Événement Kafka
- Publier `session.recorded` avec :
  - sessionId
  - recordingUrl
  - targetLanguageCode
  - level
  - durationSeconds
  - participantsWithConsent (userId, displayName)

### 5. Rétention
- Les enregistrements sont conservés 30 jours
- Job de nettoyage automatique des fichiers expirés

## Critères d'acceptation
- [ ] Le consentement est collecté à la connexion
- [ ] L'enregistrement démarre si au moins 1 consentement
- [ ] L'audio est uploadé vers S3 à la fin de session
- [ ] L'événement session.recorded est publié
- [ ] Les enregistrements sont supprimés après 30 jours

## Stack technique
- Spring Boot 4
- AWS SDK pour S3
- Spring Cloud Stream pour Kafka
