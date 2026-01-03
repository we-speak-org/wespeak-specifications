# User Story : API Progress

## Contexte

Implémenter les endpoints de consultation de la progression utilisateur.

## Endpoints à Implémenter

### GET /api/v1/progress

Récupérer la progression de l'utilisateur pour une langue. **Authentification requise.**

**Paramètres query :**
- `language` (obligatoire) : Code langue

**Réponse 200 :**
```json
{
  "userId": "user-123",
  "targetLanguageCode": "en",
  "currentCourse": {
    "id": "...",
    "title": "Anglais pour Débutants",
    "level": "A1"
  },
  "currentUnit": {
    "id": "...",
    "title": "Les Salutations"
  },
  "currentLesson": {
    "id": "...",
    "title": "Se présenter"
  },
  "stats": {
    "lessonsCompleted": 5,
    "averageScore": 82,
    "totalTimeMinutes": 45
  },
  "lastActivityAt": "2025-01-15T14:30:00Z"
}
```

**Erreurs :**
- 400 si `language` manquant
- 401 `UNAUTHORIZED`
- 404 si aucune progression pour cette langue

---

### GET /api/v1/progress/history

Historique des leçons terminées. **Authentification requise.**

**Paramètres query :**
- `language` (obligatoire) : Code langue
- `limit` (optionnel, défaut: 10, max: 50) : Nombre de résultats
- `offset` (optionnel, défaut: 0) : Pagination

**Réponse 200 :**
```json
{
  "completions": [
    {
      "lessonId": "...",
      "lessonTitle": "Dire bonjour",
      "score": 85,
      "xpEarned": 15,
      "completedAt": "2025-01-15T14:30:00Z"
    },
    {
      "lessonId": "...",
      "lessonTitle": "Introduction",
      "score": 90,
      "xpEarned": 12,
      "completedAt": "2025-01-14T10:15:00Z"
    }
  ],
  "total": 5,
  "hasMore": false
}
```

---

### GET /api/v1/progress/unlocked-lessons

Liste des leçons débloquées pour une langue. **Authentification requise.**

**Paramètres query :**
- `language` (obligatoire)

**Réponse 200 :**
```json
{
  "unlockedLessons": [
    {
      "lessonId": "...",
      "lessonTitle": "Dire bonjour",
      "isCompleted": true,
      "bestScore": 85
    },
    {
      "lessonId": "...",
      "lessonTitle": "Se présenter",
      "isCompleted": false,
      "bestScore": null
    }
  ],
  "nextToUnlock": {
    "lessonId": "...",
    "lessonTitle": "Demander l'heure",
    "requirement": "Terminez 'Se présenter' avec 70%+"
  }
}
```

## Service à Créer

### ProgressService

```java
public interface ProgressService {
    UserProgressDto getProgress(String userId, String languageCode);
    Page<LessonCompletionDto> getHistory(String userId, String languageCode, Pageable pageable);
    UnlockedLessonsDto getUnlockedLessons(String userId, String languageCode);
    
    // Méthodes internes
    void initializeProgress(String userId, String languageCode);
    void updateProgress(String userId, String languageCode, LessonCompletion completion);
}
```

## Calcul de la Progression Courante

La "leçon courante" est déterminée ainsi :

1. Trouver la **dernière leçon complétée**
2. La leçon courante = **prochaine leçon débloquée non complétée**
3. Si toutes les leçons de l'unité sont complétées → passer à la première leçon de l'unité suivante
4. Si toutes les unités du cours sont complétées → passer au cours suivant

## Critères d'Acceptation

- [ ] GET /progress retourne la progression avec stats
- [ ] GET /progress/history est paginé correctement
- [ ] GET /progress/unlocked-lessons identifie correctement les leçons débloquées
- [ ] Les statistiques (lessonsCompleted, averageScore) sont exactes
- [ ] 404 si l'utilisateur n'a pas de progression pour la langue demandée
- [ ] Tests d'intégration
