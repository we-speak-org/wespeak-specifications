# Gamification Service - Schémas des Entités

## UserStats

```typescript
class UserStats {
  id: string;                      // UUID auto-généré
  userId: string;                  // Référence utilisateur (obligatoire)
  targetLanguageCode: string;      // Code langue ex: "en" (obligatoire)
  
  // XP
  totalXp: number;                 // Défaut: 0
  weeklyXp: number;                // Défaut: 0, reset chaque lundi
  monthlyXp: number;               // Défaut: 0, reset chaque 1er du mois
  
  // Streaks
  currentStreak: number;           // Défaut: 0
  longestStreak: number;           // Défaut: 0
  lastActivityDate?: Date;         // Dernière activité
  streakFreezeAvailable: number;   // Défaut: 0
  
  // Niveau et compteurs
  level: number;                   // Défaut: 1
  lessonsCompleted: number;        // Défaut: 0
  exercisesCompleted: number;      // Défaut: 0
  conversationMinutes: number;     // Défaut: 0
  perfectLessons: number;          // Défaut: 0
  
  createdAt: Date;
  updatedAt: Date;
}
```

## Badge

```typescript
class Badge {
  id: string;                      // UUID auto-généré
  code: string;                    // Code unique ex: "first_lesson" (obligatoire, unique)
  name: string;                    // Nom affiché (obligatoire)
  description: string;             // Comment l'obtenir (obligatoire)
  iconUrl: string;                 // URL icône (obligatoire)
  
  category: BadgeCategory;         // Catégorie (obligatoire)
  xpReward: number;                // XP gagnés (obligatoire)
  rarity: BadgeRarity;             // Rareté (obligatoire)
  
  condition: BadgeCondition;       // Condition de déblocage (obligatoire)
  isActive: boolean;               // Défaut: true
}

enum BadgeCategory {
  LESSONS = "lessons",
  CONVERSATIONS = "conversations",
  STREAKS = "streaks",
  SOCIAL = "social",
  SPECIAL = "special"
}

enum BadgeRarity {
  COMMON = "common",
  UNCOMMON = "uncommon",
  RARE = "rare",
  EPIC = "epic",
  LEGENDARY = "legendary"
}

class BadgeCondition {
  type: string;                    // Ex: "lessons_completed", "streak_days"
  value: number;                   // Valeur à atteindre
  extra?: Record<string, any>;     // Conditions supplémentaires optionnelles
}
```

## UserBadge

```typescript
class UserBadge {
  id: string;                      // UUID auto-généré
  userId: string;                  // Référence utilisateur (obligatoire)
  badgeId: string;                 // Référence badge (obligatoire)
  unlockedAt: Date;                // Date déblocage (obligatoire)
  targetLanguageCode?: string;     // Langue si applicable
}
```

## Challenge

```typescript
class Challenge {
  id: string;                      // UUID auto-généré
  title: string;                   // Titre (obligatoire)
  description: string;             // Description (obligatoire)
  
  type: ChallengeType;             // Type de défi (obligatoire)
  targetType: ChallengeTargetType; // Ce qu'il faut accomplir (obligatoire)
  targetValue: number;             // Valeur à atteindre (obligatoire)
  
  xpReward: number;                // Récompense XP (obligatoire)
  startDate: Date;                 // Début (obligatoire)
  endDate: Date;                   // Fin (obligatoire)
  isActive: boolean;               // Défaut: true
}

enum ChallengeType {
  DAILY = "daily",
  WEEKLY = "weekly",
  SPECIAL = "special"
}

enum ChallengeTargetType {
  XP = "xp",
  LESSONS = "lessons",
  CONVERSATIONS = "conversations",
  STREAK = "streak"
}
```

## UserChallenge

```typescript
class UserChallenge {
  id: string;                      // UUID auto-généré
  userId: string;                  // Référence utilisateur (obligatoire)
  challengeId: string;             // Référence défi (obligatoire)
  
  currentProgress: number;         // Progression (défaut: 0)
  status: ChallengeStatus;         // Statut (défaut: IN_PROGRESS)
  
  joinedAt: Date;                  // Date inscription (obligatoire)
  completedAt?: Date;              // Date complétion si terminé
}

enum ChallengeStatus {
  IN_PROGRESS = "in_progress",
  COMPLETED = "completed",
  FAILED = "failed"
}
```

## XpTransaction

```typescript
class XpTransaction {
  id: string;                      // UUID auto-généré
  userId: string;                  // Référence utilisateur (obligatoire)
  targetLanguageCode: string;      // Langue concernée (obligatoire)
  
  amount: number;                  // XP gagnés (obligatoire, positif)
  source: XpSource;                // Source du gain (obligatoire)
  sourceId?: string;               // ID de la source si applicable
  description?: string;            // Description optionnelle
  
  createdAt: Date;                 // Date transaction (obligatoire)
}

enum XpSource {
  LESSON = "lesson",
  EXERCISE = "exercise",
  CONVERSATION = "conversation",
  BADGE = "badge",
  CHALLENGE = "challenge",
  STREAK_BONUS = "streak_bonus"
}
```

## Index Recommandés

```
UserStats:
  - userId + targetLanguageCode (unique)
  - weeklyXp (desc) pour leaderboard hebdo
  - monthlyXp (desc) pour leaderboard mensuel
  - totalXp (desc) pour leaderboard global

UserBadge:
  - userId + badgeId (unique)
  - userId (pour liste des badges d'un user)

UserChallenge:
  - userId + challengeId (unique)
  - userId + status

XpTransaction:
  - userId + createdAt (desc)
  - userId + targetLanguageCode + createdAt
```
