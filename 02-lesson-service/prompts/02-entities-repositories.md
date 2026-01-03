# User Story : Entités et Repositories MongoDB

## Contexte

Créer les entités MongoDB et leurs repositories pour le lesson-service.

## Entités à Créer

### Course (Cours)

```
Collection: courses
Champs:
- _id: ObjectId (auto)
- targetLanguageCode: String (obligatoire) - ex: "en", "fr"
- level: String (obligatoire) - "A1", "A2", "B1", "B2", "C1", "C2"
- title: String (obligatoire)
- description: String (optionnel)
- imageUrl: String (optionnel)
- order: Integer (obligatoire) - position dans le curriculum
- requiredXP: Integer (obligatoire) - XP pour débloquer (0 pour le premier)
- estimatedHours: Integer (optionnel)
- isPublished: Boolean (obligatoire, défaut: false)
- createdAt: Date (auto)
- updatedAt: Date (auto)
```

### Unit (Unité)

```
Collection: units
Champs:
- _id: ObjectId (auto)
- courseId: ObjectId (obligatoire) - référence vers Course
- title: String (obligatoire)
- description: String (optionnel)
- imageUrl: String (optionnel)
- order: Integer (obligatoire)
- createdAt: Date (auto)
- updatedAt: Date (auto)
```

### Lesson (Leçon)

```
Collection: lessons
Champs:
- _id: ObjectId (auto)
- unitId: ObjectId (obligatoire) - référence vers Unit
- title: String (obligatoire)
- description: String (optionnel)
- type: String (obligatoire) - "vocabulary", "grammar", "listening", "speaking", "conversation_prep"
- order: Integer (obligatoire)
- estimatedMinutes: Integer (obligatoire)
- xpReward: Integer (obligatoire)
- exercises: Array<Exercise> (embedded)
- createdAt: Date (auto)
- updatedAt: Date (auto)
```

### Exercise (Exercice - embedded dans Lesson)

```
Champs:
- _id: ObjectId (auto)
- type: String - "mcq", "fill_gap", "translation", "listen_repeat", "match_pairs", "ordering"
- order: Integer
- question: String
- hint: String (optionnel)
- audioUrl: String (optionnel)
- imageUrl: String (optionnel)
- content: Object (structure variable selon type)
- correctAnswer: Object
- points: Integer
```

### UserProgress (Progression)

```
Collection: user_progress
Index unique: { userId, targetLanguageCode }
Champs:
- _id: ObjectId (auto)
- userId: String (obligatoire)
- targetLanguageCode: String (obligatoire)
- currentCourseId: ObjectId (optionnel)
- currentUnitId: ObjectId (optionnel)
- currentLessonId: ObjectId (optionnel)
- lessonsCompleted: Integer (défaut: 0)
- averageScore: Integer (défaut: 0)
- totalTimeMinutes: Integer (défaut: 0)
- lastActivityAt: Date
- createdAt: Date (auto)
- updatedAt: Date (auto)
```

### LessonCompletion (Complétion)

```
Collection: lesson_completions
Index: { userId, lessonId, completedAt }
Champs:
- _id: ObjectId (auto)
- userId: String (obligatoire)
- lessonId: ObjectId (obligatoire)
- score: Integer (0-100)
- xpEarned: Integer
- correctAnswers: Integer
- totalExercises: Integer
- timeSpentSeconds: Integer
- attemptNumber: Integer
- completedAt: Date
```

## Repositories à Créer

1. **CourseRepository**
   - `findByTargetLanguageCodeAndIsPublished(String code, boolean published)`
   - `findByTargetLanguageCodeAndLevelAndIsPublished(String code, String level, boolean published)`

2. **UnitRepository**
   - `findByCourseIdOrderByOrder(ObjectId courseId)`

3. **LessonRepository**
   - `findByUnitIdOrderByOrder(ObjectId unitId)`

4. **UserProgressRepository**
   - `findByUserIdAndTargetLanguageCode(String userId, String code)`
   - `existsByUserIdAndTargetLanguageCode(String userId, String code)`

5. **LessonCompletionRepository**
   - `findByUserIdAndLessonIdOrderByCompletedAtDesc(String userId, ObjectId lessonId)`
   - `findByUserIdOrderByCompletedAtDesc(String userId, Pageable pageable)`
   - `countByUserIdAndLessonId(String userId, ObjectId lessonId)`

## Critères d'Acceptation

- [ ] Toutes les entités utilisent `@Document` avec le bon nom de collection
- [ ] Les champs Date utilisent `@CreatedDate` et `@LastModifiedDate`
- [ ] Les indexes sont définis avec `@CompoundIndex`
- [ ] Les repositories étendent `MongoRepository`
- [ ] Tests unitaires pour les repositories
