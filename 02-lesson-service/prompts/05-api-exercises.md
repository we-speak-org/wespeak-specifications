# User Story : API Exercises

## Contexte

Implémenter l'endpoint de soumission des réponses aux exercices.

## Endpoint à Implémenter

### POST /api/v1/exercises/{exerciseId}/submit

Soumettre une réponse à un exercice. **Authentification requise.**

**Headers :** `Authorization: Bearer <JWT>`

**Body (selon type d'exercice) :**

**QCM (mcq) :**
```json
{
  "answer": {
    "optionId": "a"
  },
  "timeSpentSeconds": 12
}
```

**Texte à trous (fill_gap) :**
```json
{
  "answer": {
    "text": "meet"
  },
  "timeSpentSeconds": 15
}
```

**Traduction (translation) :**
```json
{
  "answer": {
    "text": "Hello, how are you?"
  },
  "timeSpentSeconds": 30
}
```

**Association de paires (match_pairs) :**
```json
{
  "answer": {
    "pairs": [
      {"left": "1", "right": "a"},
      {"left": "2", "right": "b"}
    ]
  },
  "timeSpentSeconds": 25
}
```

**Remise en ordre (ordering) :**
```json
{
  "answer": {
    "order": ["word3", "word1", "word2", "word4"]
  },
  "timeSpentSeconds": 20
}
```

---

**Réponse 200 (Correct) :**
```json
{
  "isCorrect": true,
  "pointsEarned": 10,
  "correctAnswer": {
    "optionId": "a",
    "text": "Hello"
  },
  "feedback": "Excellent ! 'Hello' est la salutation la plus courante.",
  "attemptNumber": 1
}
```

**Réponse 200 (Incorrect) :**
```json
{
  "isCorrect": false,
  "pointsEarned": 0,
  "correctAnswer": {
    "optionId": "a",
    "text": "Hello"
  },
  "feedback": "Pas tout à fait. 'Hello' signifie 'Bonjour'.",
  "attemptNumber": 1
}
```

**Erreurs :**
- 400 `INVALID_ANSWER` - Format de réponse invalide
- 401 `UNAUTHORIZED`
- 403 `MAX_ATTEMPTS_REACHED` - Trop de tentatives
- 404 `EXERCISE_NOT_FOUND`

## Service à Créer

### ExerciseService

```java
public interface ExerciseService {
    ExerciseSubmissionResult submitAnswer(
        String exerciseId, 
        String lessonId,
        String userId, 
        SubmitAnswerRequest request
    );
}
```

### Validation des Réponses

Créer un validateur par type d'exercice :

```java
public interface AnswerValidator {
    boolean supports(String exerciseType);
    ValidationResult validate(Object userAnswer, Object correctAnswer);
}

// Implémentations :
- McqAnswerValidator
- FillGapAnswerValidator  
- TranslationAnswerValidator
- MatchPairsAnswerValidator
- OrderingAnswerValidator
```

## Règles de Validation

### QCM
- Comparer `optionId` exact

### Texte à trous
- Comparaison case-insensitive
- Ignorer les espaces en début/fin
- Accepter les alternatives définies dans `correctAnswer.alternatives`

### Traduction
- Comparaison case-insensitive
- Normaliser la ponctuation
- Accepter les alternatives

### Association de paires
- Toutes les paires doivent être correctes
- L'ordre des paires n'importe pas

### Remise en ordre
- L'ordre exact doit correspondre

## Limites de Tentatives

| Tier | Max tentatives par exercice |
|------|----------------------------|
| Free | 3 |
| Premium | 5 |

Après le max :
- Réponse 403 `MAX_ATTEMPTS_REACHED`
- L'utilisateur doit recommencer la leçon

## Critères d'Acceptation

- [ ] Tous les types d'exercices sont supportés
- [ ] La validation est correcte pour chaque type
- [ ] Le feedback est approprié
- [ ] Les tentatives sont comptées par exercice
- [ ] Tests unitaires pour chaque validateur
- [ ] Tests d'intégration
