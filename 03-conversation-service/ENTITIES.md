# Conversation Service - Modèle de Données

## Collections MongoDB

### Session

Représente une session de conversation pouvant accueillir 2 à 6 participants.

```typescript
{
  id: string;                    // ObjectId MongoDB
  targetLanguageCode: string;    // Langue pratiquée ("en", "fr", "es")
  status: string;                // "waiting" | "active" | "ended"
  type: string;                  // "public" | "private"
  inviteCode?: string;           // Code d'invitation (sessions privées)
  topicId?: string;              // Référence au topic (optionnel)
  hostUserId: string;            // Créateur de la session
  minParticipants: number;       // Minimum requis (défaut: 2)
  maxParticipants: number;       // Maximum autorisé (défaut: 6)
  scheduledStartAt?: Date;       // Heure de début prévue
  startedAt?: Date;              // Début effectif
  endedAt?: Date;                // Fin de la session
  endReason?: string;            // "completed" | "timeout" | "host_ended"
  createdAt: Date;
  updatedAt: Date;
}
```

### Participant

Représente un utilisateur dans une session.

```typescript
{
  id: string;                    // ObjectId MongoDB
  sessionId: string;             // Référence à la session
  userId: string;                // Référence à l'utilisateur
  role: string;                  // "host" | "participant"
  status: string;                // "waiting" | "connected" | "disconnected"
  joinedAt: Date;                // Heure d'arrivée
  leftAt?: Date;                 // Heure de départ
  speakingTimeSeconds?: number;  // Temps de parole cumulé
  connectionInfo?: {
    peerId: string;              // ID pour WebRTC
    lastHeartbeat: Date;
  };
}
```

### Topic

Sujet de conversation pour guider les échanges.

```typescript
{
  id: string;                    // ObjectId MongoDB
  targetLanguageCode: string;    // "en", "fr", "es", etc.
  title: string;                 // Titre court
  description?: string;          // Description détaillée
  suggestedQuestions?: string[]; // Questions pour guider
  level: string;                 // "A1" | "A2" | "B1" | "B2" | "C1" | "C2"
  category: string;              // "daily_life" | "work" | "travel" | "culture" | "hobbies"
  isActive: boolean;             // Topic disponible
  createdAt: Date;
  updatedAt: Date;
}
```

### QueueEntry

Utilisateur en attente de matchmaking.

```typescript
{
  id: string;                    // ObjectId MongoDB
  userId: string;                // Utilisateur en attente
  targetLanguageCode: string;    // Langue recherchée
  level: string;                 // Niveau CECR
  preferredTopicCategory?: string; // Catégorie préférée
  joinedQueueAt: Date;           // Entrée dans la file
  expiresAt: Date;               // Expiration (joinedQueueAt + 2 min)
}
```

---

## Index MongoDB Recommandés

### Session
```
{ status: 1, targetLanguageCode: 1 }
{ hostUserId: 1, createdAt: -1 }
{ inviteCode: 1 } // unique, sparse
{ type: 1, status: 1 }
```

### Participant
```
{ sessionId: 1 }
{ userId: 1, joinedAt: -1 }
{ sessionId: 1, status: 1 }
```

### Topic
```
{ targetLanguageCode: 1, level: 1, isActive: 1 }
{ category: 1, targetLanguageCode: 1 }
```

### QueueEntry
```
{ targetLanguageCode: 1, level: 1, joinedQueueAt: 1 }
{ userId: 1 }
{ expiresAt: 1 }  // TTL index pour nettoyage automatique
```

---

## Structures Redis

### File de Matchmaking (pour recherche rapide)
```
Key: matchmaking:queue:{targetLanguageCode}
Type: Sorted Set
Score: timestamp (joinedQueueAt)
Value: JSON { userd, level, preferredTopicCategory }
TTL: 120 seconds
```

### Session Active
```
Key: session:active:{sessionId}
Type: Hash
Fields: 
  - status
  - participantCount
  - lastActivity
TTL: 1800 seconds (30 min max)
```

### Participants d'une Session
```
Key: session:participants:{sessionId}
Type: Set
Value: userIds des participants
TTL: 1800 seconds
```

### Session de l'Utilisateur
```
Key: user:current-session:{userId}
Type: String
Value: sessionId
TTL: 1800 seconds
```

### Signaling WebRTC
```
Key: signaling:{sessionId}:{fromUserId}:{toUserId}
Type: List
Value: Messages de signaling (offer, answer, ice-candidate)
TTL: 60 seconds
```
