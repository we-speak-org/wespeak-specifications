# Recommendation Service - Schémas des Entités

## UserPreferences

```typescript
class UserPreferences {
  id: string;                       // UUID auto-généré
  userId: string;                   // Référence utilisateur (obligatoire)
  targetLanguageCode: string;       // Code langue (obligatoire)
  
  preferredLearningTime: string[];  // ["morning", "afternoon", "evening", "night"]
  dailyGoalMinutes: number;         // Défaut: 15
  focusAreas: string[];             // ["grammar", "vocabulary", "pronunciation", "listening", "reading"]
  excludedTopics: string[];         // Topics à éviter
  
  createdAt: Date;
  updatedAt: Date;
}
```

## LearningHistory

```typescript
class LearningHistory {
  id: string;                       // UUID auto-généré
  userId: string;                   // Référence utilisateur (obligatoire)
  targetLanguageCode: string;       // Code langue (obligatoire)
  
  completedLessonIds: string[];     // Liste des leçons complétées
  lastLessonId?: string;            // Dernière leçon suivie
  
  weakAreas: WeakArea[];            // Points faibles
  strongAreas: StrongArea[];        // Points forts
  
  averageScore: number;             // Score moyen (0-100)
  totalLessons: number;             // Nombre de leçons complétées
  totalConversationMinutes: number; // Minutes de conversation
  
  lastUpdated: Date;
}

class WeakArea {
  category: string;                 // Ex: "grammar", "vocabulary"
  subcategory: string;              // Ex: "past_tense", "food_vocabulary"
  errorCount: number;               // Nombre d'erreurs
  lastErrorAt: Date;                // Dernière erreur
}

class StrongArea {
  category: string;
  subcategory: string;
  successRate: number;              // Taux de réussite (0-100)
}
```

## Recommendation

```typescript
class Recommendation {
  id: string;                       // UUID auto-généré
  userId: string;                   // Référence utilisateur (obligatoire)
  targetLanguageCode: string;       // Code langue (obligatoire)
  
  type: RecommendationType;         // Type de recommandation
  targetId: string;                 // ID de la ressource (leçon, slot, etc.)
  targetType: TargetType;           // Type de ressource
  
  title: string;                    // Titre affiché
  reason: string;                   // Raison de la recommandation
  priority: number;                 // 1 = haute priorité
  
  expiresAt?: Date;                 // Date d'expiration
  dismissed: boolean;               // Défaut: false
  clickedAt?: Date;                 // Date de clic si cliqué
  
  createdAt: Date;
}

enum RecommendationType {
  NEXT_LESSON = "next_lesson",
  REVISION = "revision",
  CONVERSATION = "conversation",
  PRACTICE = "practice"
}

enum TargetType {
  LESSON = "lesson",
  EXERCISE = "exercise",
  SLOT = "slot",
  UNIT = "unit"
}
```

## Index Recommandés

```
UserPreferences:
  - userId + targetLanguageCode (unique)

LearningHistory:
  - userId + targetLanguageCode (unique)
  - userId (pour récupérer toutes les langues)

Recommendation:
  - userId + targetLanguageCode + dismissed + expiresAt
  - userId + createdAt (desc)
  - expiresAt (pour cleanup)
```
