# Lesson Service - Modèle de Données

## Collections MongoDB

Le service utilise **MongoDB** comme base de données. Voici les collections et leurs structures.

---

## 1. Course (Cours)

Un cours regroupe plusieurs unités pour un niveau CECRL donné.

### Champs

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `_id` | ObjectId | Oui | Identifiant unique |
| `targetLanguageCode` | String | Oui | Code langue cible (ex: "en", "fr", "es") |
| `level` | String | Oui | Niveau CECRL : "A1", "A2", "B1", "B2", "C1", "C2" |
| `title` | String | Oui | Titre du cours (ex: "Anglais pour Débutants") |
| `description` | String | Non | Description du cours |
| `imageUrl` | String | Non | URL de l'image du cours |
| `order` | Number | Oui | Position dans le curriculum (1, 2, 3...) |
| `requiredXP` | Number | Oui | XP minimum pour débloquer ce cours (0 pour le premier) |
| `estimatedHours` | Number | Non | Durée estimée en heures |
| `isPublished` | Boolean | Oui | Cours visible aux utilisateurs |
| `createdAt` | Date | Oui | Date de création |
| `updatedAt` | Date | Oui | Date de dernière modification |

### Exemple

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "targetLanguageCode": "en",
  "level": "A1",
  "title": "Anglais pour Débutants",
  "description": "Apprenez les bases de l'anglais pour communiquer au quotidien",
  "imageUrl": "https://cdn.wespeak.com/courses/en-a1.png",
  "order": 1,
  "requiredXP": 0,
  "estimatedHours": 20,
  "isPublished": true,
  "createdAt": "2025-01-01T00:00:00Z",
  "updatedAt": "2025-01-01T00:00:00Z"
}
```

---

## 2. Unit (Unité)

Une unité thématique contenant plusieurs leçons.

### Champs

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `_id` | ObjectId | Oui | Identifiant unique |
| `courseId` | ObjectId | Oui | Référence vers le cours parent |
| `title` | String | Oui | Titre de l'unité (ex: "Les Salutations") |
| `description` | String | Non | Description de l'unité |
| `imageUrl` | String | Non | URL de l'image |
| `order` | Number | Oui | Position dans le cours (1, 2, 3...) |
| `createdAt` | Date | Oui | Date de création |
| `updatedAt` | Date | Oui | Date de dernière modification |

### Exemple

```json
{
  "_id": "507f1f77bcf86cd799439012",
  "courseId": "507f1f77bcf86cd799439011",
  "title": "Les Salutations",
  "description": "Apprenez à saluer et à vous présenter en anglais",
  "imageUrl": "https://cdn.wespeak.com/units/greetings.png",
  "order": 1,
  "createdAt": "2025-01-01T00:00:00Z",
  "updatedAt": "2025-01-01T00:00:00Z"
}
```

---

## 3. Lesson (Leçon)

Une leçon individuelle avec ses exercices.

### Champs

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `_id` | ObjectId | Oui | Identifiant unique |
| `unitId` | ObjectId | Oui | Référence vers l'unité parente |
| `title` | String | Oui | Titre de la leçon |
| `description` | String | Non | Description courte |
| `type` | String | Oui | Type : "vocabulary", "grammar", "listening", "speaking", "conversation_prep" |
| `order` | Number | Oui | Position dans l'unité (1, 2, 3...) |
| `estimatedMinutes` | Number | Oui | Durée estimée en minutes |
| `xpReward` | Number | Oui | XP gagné à la complétion (base) |
| `exercises` | Array | Oui | Liste des exercices (voir ci-dessous) |
| `createdAt` | Date | Oui | Date de création |
| `updatedAt` | Date | Oui | Date de dernière modification |

### Structure d'un Exercise (embedded)

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `_id` | ObjectId | Oui | Identifiant unique de l'exercice |
| `type` | String | Oui | Type : "mcq", "fill_gap", "translation", "listen_repeat", "match_pairs", "ordering" |
| `order` | Number | Oui | Position dans la leçon |
| `question` | String | Oui | Consigne ou question |
| `hint` | String | Non | Indice pour aider l'utilisateur |
| `audioUrl` | String | Non | URL audio (pour écoute) |
| `imageUrl` | String | Non | URL image |
| `content` | Object | Oui | Contenu spécifique selon le type (voir exemples) |
| `correctAnswer` | Object | Oui | Réponse(s) correcte(s) |
| `points` | Number | Oui | Points pour cet exercice |

### Exemple de Leçon complète

```json
{
  "_id": "507f1f77bcf86cd799439013",
  "unitId": "507f1f77bcf86cd799439012",
  "title": "Dire bonjour",
  "description": "Apprenez les salutations de base en anglais",
  "type": "vocabulary",
  "order": 1,
  "estimatedMinutes": 10,
  "xpReward": 15,
  "exercises": [
    {
      "_id": "507f1f77bcf86cd799439101",
      "type": "mcq",
      "order": 1,
      "question": "Comment dit-on 'Bonjour' en anglais ?",
      "content": {
        "options": [
          {"id": "a", "text": "Hello"},
          {"id": "b", "text": "Goodbye"},
          {"id": "c", "text": "Thank you"},
          {"id": "d", "text": "Sorry"}
        ]
      },
      "correctAnswer": {"optionId": "a"},
      "points": 10
    },
    {
      "_id": "507f1f77bcf86cd799439102",
      "type": "fill_gap",
      "order": 2,
      "question": "Complétez : Nice to ___ you!",
      "hint": "Un verbe courant pour les rencontres",
      "content": {
        "sentence": "Nice to ___ you!",
        "blanks": ["___"]
      },
      "correctAnswer": {"text": "meet", "alternatives": ["Meet", "MEET"]},
      "points": 10
    },
    {
      "_id": "507f1f77bcf86cd799439103",
      "type": "listen_repeat",
      "order": 3,
      "question": "Écoutez et répétez",
      "audioUrl": "https://cdn.wespeak.com/audio/hello-how-are-you.mp3",
      "content": {
        "targetSentence": "Hello, how are you?"
      },
      "correctAnswer": {"expectedPhonemes": ["həˈloʊ", "haʊ", "ɑːr", "juː"]},
      "points": 15
    }
  ],
  "createdAt": "2025-01-01T00:00:00Z",
  "updatedAt": "2025-01-01T00:00:00Z"
}
```

---

## 4. UserProgress (Progression Utilisateur)

Suit l'avancement d'un utilisateur pour une langue donnée.

### Champs

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `_id` | ObjectId | Oui | Identifiant unique |
| `userId` | String | Oui | ID utilisateur (de auth-service) |
| `targetLanguageCode` | String | Oui | Langue d'apprentissage |
| `currentCourseId` | ObjectId | Non | Cours en cours |
| `currentUnitId` | ObjectId | Non | Unité en cours |
| `currentLessonId` | ObjectId | Non | Leçon en cours |
| `lessonsCompleted` | Number | Oui | Nombre total de leçons terminées |
| `averageScore` | Number | Oui | Score moyen (0-100) |
| `totalTimeMinutes` | Number | Oui | Temps total passé en minutes |
| `lastActivityAt` | Date | Oui | Dernière activité |
| `createdAt` | Date | Oui | Date de création |
| `updatedAt` | Date | Oui | Date de dernière modification |

### Exemple

```json
{
  "_id": "507f1f77bcf86cd799439020",
  "userId": "user-123-uuid",
  "targetLanguageCode": "en",
  "currentCourseId": "507f1f77bcf86cd799439011",
  "currentUnitId": "507f1f77bcf86cd799439012",
  "currentLessonId": "507f1f77bcf86cd799439013",
  "lessonsCompleted": 5,
  "averageScore": 82,
  "totalTimeMinutes": 45,
  "lastActivityAt": "2025-01-15T14:30:00Z",
  "createdAt": "2025-01-10T10:00:00Z",
  "updatedAt": "2025-01-15T14:30:00Z"
}
```

---

## 5. LessonCompletion (Complétion de Leçon)

Enregistre chaque fois qu'un utilisateur termine une leçon.

### Champs

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `_id` | ObjectId | Oui | Identifiant unique |
| `userId` | String | Oui | ID utilisateur |
| `lessonId` | ObjectId | Oui | ID de la leçon terminée |
| `score` | Number | Oui | Score obtenu (0-100) |
| `xpEarned` | Number | Oui | XP gagnés |
| `correctAnswers` | Number | Oui | Nombre de bonnes réponses |
| `totalExercises` | Number | Oui | Nombre total d'exercices |
| `timeSpentSeconds` | Number | Oui | Temps passé en secondes |
| `attemptNumber` | Number | Oui | Numéro de la tentative (1, 2, 3...) |
| `completedAt` | Date | Oui | Date de complétion |

### Exemple

```json
{
  "_id": "507f1f77bcf86cd799439030",
  "userId": "user-123-uuid",
  "lessonId": "507f1f77bcf86cd799439013",
  "score": 85,
  "xpEarned": 15,
  "correctAnswers": 17,
  "totalExercises": 20,
  "timeSpentSeconds": 420,
  "attemptNumber": 1,
  "completedAt": "2025-01-15T14:30:00Z"
}
```

---

## Index MongoDB Recommandés

```javascript
// Courses - recherche par langue et niveau
db.courses.createIndex({ "targetLanguageCode": 1, "level": 1, "isPublished": 1 })
db.courses.createIndex({ "targetLanguageCode": 1, "order": 1 })

// Units - recherche par cours
db.units.createIndex({ "courseId": 1, "order": 1 })

// Lessons - recherche par unité
db.lessons.createIndex({ "unitId": 1, "order": 1 })

// UserProgress - recherche par utilisateur et langue (unique)
db.userProgress.createIndex({ "userId": 1, "targetLanguageCode": 1 }, { unique: true })

// LessonCompletion - historique par utilisateur
db.lessonCompletions.createIndex({ "userId": 1, "lessonId": 1, "completedAt": -1 })
db.lessonCompletions.createIndex({ "userId": 1, "completedAt": -1 })
```

---

## Schéma TypeScript (pour référence)

```typescript
// Types de base
type CECRLevel = 'A1' | 'A2' | 'B1' | 'B2' | 'C1' | 'C2';
type LessonType = 'vocabulary' | 'grammar' | 'listening' | 'speaking' | 'conversation_prep';
type ExerciseType = 'mcq' | 'fill_gap' | 'translation' | 'listen_repeat' | 'match_pairs' | 'ordering';

// Course
interface Course {
  _id: string;
  targetLanguageCode: string;
  level: CECRLevel;
  title: string;
  description?: string;
  imageUrl?: string;
  order: number;
  requiredXP: number;
  estimatedHours?: number;
  isPublished: boolean;
  createdAt: Date;
  updatedAt: Date;
}

// Unit
interface Unit {
  _id: string;
  courseId: string;
  title: string;
  description?: string;
  imageUrl?: string;
  order: number;
  createdAt: Date;
  updatedAt: Date;
}

// Exercise (embedded in Lesson)
interface Exercise {
  _id: string;
  type: ExerciseType;
  order: number;
  question: string;
  hint?: string;
  audioUrl?: string;
  imageUrl?: string;
  content: Record<string, any>;
  correctAnswer: Record<string, any>;
  points: number;
}

// Lesson
interface Lesson {
  _id: string;
  unitId: string;
  title: string;
  description?: string;
  type: LessonType;
  order: number;
  estimatedMinutes: number;
  xpReward: number;
  exercises: Exercise[];
  createdAt: Date;
  updatedAt: Date;
}

// UserProgress
interface UserProgress {
  _id: string;
  userId: string;
  targetLanguageCode: string;
  currentCourseId?: string;
  currentUnitId?: string;
  currentLessonId?: string;
  lessonsCompleted: number;
  averageScore: number;
  totalTimeMinutes: number;
  lastActivityAt: Date;
  createdAt: Date;
  updatedAt: Date;
}

// LessonCompletion
interface LessonCompletion {
  _id: string;
  userId: string;
  lessonId: string;
  score: number;
  xpEarned: number;
  correctAnswers: number;
  totalExercises: number;
  timeSpentSeconds: number;
  attemptNumber: number;
  completedAt: Date;
}
```
