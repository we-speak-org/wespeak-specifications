# Feedback Service - Modèles de Données

## Transcript

```typescript
class Transcript {
  id: string;                    // UUID, généré automatiquement
  sessionId: string;             // Référence à la session de conversation
  participantId: string;         // ID de l'utilisateur transcrit
  recordingId: string;           // ID de l'enregistrement audio source
  targetLanguageCode: string;    // Code langue (ex: "en", "fr", "es")
  content: string;               // Texte transcrit complet
  segments: TranscriptSegment[]; // Segments avec timestamps
  duration: number;              // Durée en secondes
  wordCount: number;             // Nombre de mots
  confidence: number;            // Score de confiance global (0.0 - 1.0)
  status: TranscriptStatus;      // État du traitement
  createdAt: Date;               // Date de création
  completedAt?: Date;            // Date de fin de transcription
}

class TranscriptSegment {
  startTime: number;             // Temps de début en secondes
  endTime: number;               // Temps de fin en secondes
  text: string;                  // Texte du segment
  confidence: number;            // Confiance du segment (0.0 - 1.0)
}

enum TranscriptStatus {
  PENDING = "PENDING",           // En attente de traitement
  PROCESSING = "PROCESSING",     // Transcription en cours
  COMPLETED = "COMPLETED",       // Transcription terminée
  FAILED = "FAILED"              // Échec de la transcription
}
```

## Feedback

```typescript
class Feedback {
  id: string;                    // UUID, généré automatiquement
  transcriptId: string;          // Référence au transcript analysé
  userId: string;                // ID de l'utilisateur
  sessionId: string;             // ID de la session source
  targetLanguageCode: string;    // Langue analysée
  overallScore: number;          // Score global (0-100)
  grammarScore: number;          // Score grammaire (0-100)
  vocabularyScore: number;       // Score vocabulaire (0-100)
  fluencyScore: number;          // Score fluidité (0-100)
  pronunciationScore?: number;   // Score prononciation (0-100), optionnel
  errors: FeedbackError[];       // Liste des erreurs détectées
  strengths: string[];           // Points forts identifiés
  improvements: string[];        // Conseils d'amélioration
  summary: string;               // Résumé textuel du feedback
  xpAwarded: number;             // XP attribués pour cette session
  status: FeedbackStatus;        // État du traitement
  createdAt: Date;               // Date de création
  completedAt?: Date;            // Date de fin d'analyse
}

class FeedbackError {
  type: ErrorType;               // Type d'erreur
  original: string;              // Ce qui a été dit
  correction: string;            // Correction suggérée
  explanation: string;           // Explication de l'erreur
  severity: ErrorSeverity;       // Gravité de l'erreur
  segmentIndex: number;          // Index du segment concerné
}

enum ErrorType {
  GRAMMAR = "GRAMMAR",
  VOCABULARY = "VOCABULARY",
  PRONUNCIATION = "PRONUNCIATION",
  SYNTAX = "SYNTAX"
}

enum ErrorSeverity {
  LOW = "LOW",                   // Erreur mineure
  MEDIUM = "MEDIUM",             // Erreur notable
  HIGH = "HIGH"                  // Erreur grave
}

enum FeedbackStatus {
  PENDING = "PENDING",           // En attente d'analyse
  PROCESSING = "PROCESSING",     // Analyse en cours
  COMPLETED = "COMPLETED",       // Feedback généré
  FAILED = "FAILED"              // Échec de l'analyse
}
```

## UserFeedbackStats

```typescript
class UserFeedbackStats {
  id: string;                    // UUID
  userId: string;                // ID de l'utilisateur
  targetLanguageCode: string;    // Langue cible
  totalSessions: number;         // Nombre total de sessions analysées
  totalMinutes: number;          // Minutes totales de conversation
  averageOverallScore: number;   // Score moyen global
  averageGrammarScore: number;   // Score moyen grammaire
  averageVocabularyScore: number; // Score moyen vocabulaire
  averageFluencyScore: number;   // Score moyen fluidité
  commonErrors: CommonError[];   // Erreurs les plus fréquentes
  progressTrend: ProgressTrend;  // Tendance de progression
  lastFeedbackAt?: Date;         // Date du dernier feedback
  updatedAt: Date;               // Dernière mise à jour
}

class CommonError {
  type: ErrorType;               // Type d'erreur
  pattern: string;               // Pattern identifié (ex: "Temps du passé")
  frequency: number;             // Nombre d'occurrences
}

enum ProgressTrend {
  IMPROVING = "IMPROVING",       // En progression
  STABLE = "STABLE",             // Stable
  DECLINING = "DECLINING"        // En régression
}
```

## DTOs

### TranscriptResponse

```typescript
class TranscriptResponse {
  id: string;
  sessionId: string;
  participantId: string;
  targetLanguageCode: string;
  content: string;
  segments: TranscriptSegment[];
  duration: number;
  wordCount: number;
  confidence: number;
  status: TranscriptStatus;
  createdAt: Date;
  completedAt?: Date;
}
```

### FeedbackResponse

```typescript
class FeedbackResponse {
  id: string;
  transcriptId: string;
  userId: string;
  sessionId: string;
  targetLanguageCode: string;
  overallScore: number;
  grammarScore: number;
  vocabularyScore: number;
  fluencyScore: number;
  pronunciationScore?: number;
  errors: FeedbackError[];
  strengths: string[];
  improvements: string[];
  summary: string;
  xpAwarded: number;
  status: FeedbackStatus;
  createdAt: Date;
  completedAt?: Date;
}
```

### FeedbackListItem

```typescript
class FeedbackListItem {
  id: string;
  sessionId: string;
  targetLanguageCode: string;
  overallScore: number;
  xpAwarded: number;
  createdAt: Date;
}
```

### UserStatsResponse

```typescript
class UserStatsResponse {
  userId: string;
  targetLanguageCode: string;
  totalSessions: number;
  totalMinutes: number;
  averageOverallScore: number;
  averageGrammarScore: number;
  averageVocabularyScore: number;
  averageFluencyScore: number;
  commonErrors: CommonError[];
  progressTrend: ProgressTrend;
  lastFeedbackAt?: Date;
}
```

### ProgressHistoryResponse

```typescript
class ProgressHistoryResponse {
  userId: string;
  targetLanguageCode: string;
  period: string;
  dataPoints: ProgressDataPoint[];
}

class ProgressDataPoint {
  date: string;                  // Format: YYYY-MM-DD
  overallScore: number;
  sessionsCount: number;
}
```
