# Conversation Service - Modèle de Données

## Collections MongoDB

### TimeSlot (Créneau)

Représente une plage horaire prédéfinie pour les sessions.

```typescript
{
  id: string;                    // ObjectId MongoDB
  targetLanguageCode: string;    // Langue pratiquée ("en", "fr", "es")
  level: string;                 // "A1" | "A2" | "B1" | "B2" | "C1" | "C2"
  startTime: Date;               // Heure de début
  durationMinutes: number;       // 15, 30 ou 45 minutes
  maxParticipants: number;       // Capacité max (défaut: 8)
  minParticipants: number;       // Minimum pour démarrer (défaut: 2)
  recurrence?: string;           // "daily" | "weekly" | "once"
  isActive: boolean;             // Créneau disponible
  createdAt: Date;
  updatedAt: Date;
}
```

### Registration (Inscription)

Inscription d'un utilisateur à un créneau.

```typescript
{
  id: string;                    // ObjectId MongoDB
  timeSlotId: string;            // Référence au créneau
  userId: string;                // Utilisateur inscrit
  status: string;                // "registered" | "cancelled" | "attended" | "noshow"
  registeredAt: Date;            // Date d'inscription
  cancelledAt?: Date;            // Date d'annulation
}
```

### Session

Session de conversation créée pour un créneau.

```typescript
{
  id: string;                    // ObjectId MongoDB
  timeSlotId: string;            // Référence au créneau
  targetLanguageCode: string;    // Langue pratiquée
  level: string;                 // "A1" | "A2" | "B1" | "B2" | "C1" | "C2"
  status: string;                // "waiting" | "active" | "ended"
  startedAt?: Date;              // Début effectif
  endedAt?: Date;                // Fin de la session
  recordingEnabled: boolean;     // Enregistrement actif
  recordings?: [{                // Liste des enregistrements
    userId: string;
    url: string;                 // URL R2
    startTime: Date;             // Timestamp synchronisation
  }];
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
  status: string;                // "waiting" | "connected" | "disconnected"
  cameraEnabled: boolean;        // État de la caméra
  micEnabled: boolean;           // État du micro
  recordingConsent: boolean;     // Consentement à l'enregistrement
  joinedAt: Date;                // Heure de connexion
  leftAt?: Date;                 // Heure de déconnexion
}
```

---

## Index MongoDB Recommandés

### TimeSlot
```
{ targetLanguageCode: 1, level: 1, startTime: 1 }
{ startTime: 1, isActive: 1 }
```

### Registration
```
{ timeSlotId: 1, status: 1 }
{ userId: 1, status: 1 }
{ timeSlotId: 1, userId: 1 }  // unique
```

### Session
```
{ timeSlotId: 1 }
{ status: 1, targetLanguageCode: 1 }
{ createdAt: 1 }  // TTL index 30 jours
```

### Participant
```
{ sessionId: 1 }
{ userId: 1, status: 1 }
```
