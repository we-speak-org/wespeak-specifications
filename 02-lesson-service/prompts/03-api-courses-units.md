# User Story : API Courses et Units

## Contexte

Implémenter les endpoints REST pour consulter les cours et unités.

## Endpoints à Implémenter

### GET /api/v1/courses

Liste les cours disponibles pour une langue.

**Paramètres query :**
- `language` (obligatoire) : Code langue (ex: "en")
- `level` (optionnel) : Filtrer par niveau CECRL

**Réponse 200 :**
```json
{
  "courses": [
    {
      "id": "...",
      "title": "Anglais pour Débutants",
      "level": "A1",
      "description": "...",
      "imageUrl": "...",
      "order": 1,
      "requiredXP": 0,
      "estimatedHours": 20,
      "totalUnits": 5,
      "totalLessons": 25
    }
  ]
}
```

**Erreurs :**
- 400 si `language` manquant

---

### GET /api/v1/courses/{courseId}

Détail d'un cours avec ses unités.

**Réponse 200 :**
```json
{
  "id": "...",
  "title": "Anglais pour Débutants",
  "level": "A1",
  "description": "...",
  "imageUrl": "...",
  "estimatedHours": 20,
  "units": [
    {
      "id": "...",
      "title": "Les Salutations",
      "order": 1,
      "totalLessons": 5,
      "isUnlocked": true
    }
  ]
}
```

**Erreurs :**
- 404 `COURSE_NOT_FOUND`

---

### GET /api/v1/units/{unitId}

Détail d'une unité avec ses leçons.

**Note :** Si l'utilisateur est authentifié (JWT présent), inclure son état de progression (`isUnlocked`, `isCompleted`, `bestScore`).

**Réponse 200 :**
```json
{
  "id": "...",
  "title": "Les Salutations",
  "description": "...",
  "courseId": "...",
  "lessons": [
    {
      "id": "...",
      "title": "Dire bonjour",
      "type": "vocabulary",
      "order": 1,
      "estimatedMinutes": 10,
      "xpReward": 15,
      "isUnlocked": true,
      "isCompleted": true,
      "bestScore": 85
    }
  ]
}
```

**Erreurs :**
- 404 `UNIT_NOT_FOUND`

## Services à Créer

### CourseService

```java
public interface CourseService {
    List<CourseDto> findCourses(String languageCode, String level);
    CourseDetailDto findCourseById(String courseId);
}
```

### UnitService

```java
public interface UnitService {
    UnitDetailDto findUnitById(String unitId, String userId);
}
```

## Règles Métier

1. **Seuls les cours publiés** (`isPublished = true`) sont retournés
2. **totalUnits** et **totalLessons** sont calculés dynamiquement
3. **isUnlocked** pour une unité :
   - La première unité est toujours débloquée
   - Les suivantes sont débloquées si toutes les leçons de l'unité précédente sont complétées (score ≥ 70%)

## Critères d'Acceptation

- [ ] GET /courses retourne les cours par langue
- [ ] GET /courses/{id} inclut les unités triées par `order`
- [ ] GET /units/{id} inclut les leçons avec état de progression si authentifié
- [ ] Gestion des erreurs 404 avec messages clairs
- [ ] Tests d'intégration avec données de test
