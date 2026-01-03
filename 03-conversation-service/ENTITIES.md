# Conversation Service - Modèle de Données

## Collections MongoDB

### Topic

```typescript
{
  id: string;                    // ObjectId MongoDB
  targetLanguageCode: string;    // "en", "fr", "es", etc.
  level: string;                 // "A1" | "A2" | "B1" | "B2" | "C1" | "C2"
  title: string;                 // Titre du sujet
  description?: string;          // Description détaillée
  promptQuestions?: string[];    // Questions pour lancer la discussion
  category: string;              // "daily_life" | "work" | "travel" | "culture" | "hobbies" | "news"
  estimatedDurationMinutes: number; // 5, 10, 15, 20
  isActive: boolean;             // Topic disponible
  createdAt: Date;
  updatedAt: Date;
}
```

### ConversationSession

```typescript
{
  id: string;                    // ObjectId MongoDB
  topicId?: string;              // Référence au topic (optionnel si libre)
  targetLanguageCode: string;    // Langue pratiquée
  participant1Id: string;        // UserId du participant 1
  participant2Id: string;        // UserId du participant 2
  status: string;                // "waiting" | "active" | "completed" | "cancelled"
  scheduledAt?: Date;            // Si session planifiée
  startedAt?: Date;              // Début effectif de la conversation
  endedAt?: Date;                // Fin de la session
  actualDurationSeconds?: number; // Durée réelle
  endReason?: string;            // "completed" | "dropped" | "timeout" | "reported"
  audioRecordingUrl?: string;    // URL S3 de l'enregistrement
  recordingConsent: {
    participant1: boolean;
    participant2: boolean;
  };
  createdAt: Date;
  updatedAt: Date;
}
```

### MatchmakingRequest

```typescript
{
  id: string;                    // ObjectId MongoDB
  userId: string;                // Utilisateur demandeur
  learningProfileId: string;     // Profil d'apprentissage utilisé
  targetLanguageCode: string;    // Langue à pratiquer
  userLevel: string;             // Niveau CECRL de l'utilisateur
  preferredTopicId?: string;     // Topic préféré (optionnel)
  preferredDuration: number;     // Durée souhaitée en minutes
  status: string;                // "pending" | "matched" | "expired" | "cancelled"
  matchedWithUserId?: string;    // Partenaire trouvé
  sessionId?: string;            // Session créée après match
  createdAt: Date;
  expiresAt: Date;               // Expiration (createdAt + 2 minutes)
}
```

---

## Index MongoDB Recommandés

### Topics
```
{ targetLanguageCode: 1, level: 1, isActive: 1 }
{ category: 1, targetLanguageCode: 1 }
```

### ConversationSession
```
{ participant1Id: 1, createdAt: -1 }
{ participant2Id: 1, createdAt: -1 }
{ status: 1, targetLanguageCode: 1 }
```

### MatchmakingRequest
```
{ userId: 1, status: 1 }
{ targetLanguageCode: 1, userLevel: 1, status: 1, createdAt: 1 }
{ expiresAt: 1 }  // TTL index pour nettoyage automatique
```

---

## Structures Redis

### File de Matchmaking
```
Key: matchmaking:{targetLanguageCode}:{level}
Type: Sorted Set
Score: timestamp de création
Value: JSON { userId, learningProfileId, preferredTopicId, preferredDuration }
TTL: 120 seconds
```

### Session Active
```
Key: session:active:{sessionId}
Type: Hash
Fields: 
  - participant1Id
  - participant2Id
  - status
  - lastHeartbeat1
  - lastHeartbeat2
TTL: 1800 seconds (30 min max session)
```

### User Session Mapping
```
Key: user:session:{userId}
Type: String
Value: sessionId
TTL: 1800 seconds
```
