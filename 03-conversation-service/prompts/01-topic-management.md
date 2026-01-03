# User Story: Gestion des Topics

## Contexte
En tant qu'apprenant, je veux pouvoir consulter une liste de sujets de conversation adaptés à mon niveau et à ma langue cible pour choisir un thème avant de démarrer une session.

## Critères d'Acceptation

### AC1: Liste des topics
- L'API GET /topics retourne les topics filtrés par langue et niveau
- Seuls les topics actifs (isActive=true) sont retournés
- Les topics sont triés par catégorie puis par titre

### AC2: Filtrage par niveau
- Un utilisateur B1 voit les topics A2, B1 et B2
- Le niveau est récupéré depuis le profil d'apprentissage actif

### AC3: Structure Topic
- Chaque topic contient: id, title, description, category, level, estimatedDurationMinutes, promptQuestions

## Tâches Techniques

1. Créer l'entité Topic avec les champs définis dans ENTITIES.md
2. Créer TopicRepository avec les méthodes:
   - findByTargetLanguageCodeAndLevelInAndIsActiveTrue(language, levels)
   - findByIdAndIsActiveTrue(id)
3. Créer TopicService avec la logique de filtrage par niveau adjacent
4. Créer TopicController avec les endpoints GET /topics et GET /topics/{id}
5. Ajouter les index MongoDB définis
6. Écrire les tests unitaires du service
7. Écrire les tests d'intégration de l'API

## Données de Test
Créer 5 topics pour chaque combinaison langue/niveau:
- Langues: en, fr, es
- Niveaux: A1, A2, B1, B2, C1, C2
- Catégories variées: daily_life, work, travel, culture, hobbies

## Definition of Done
- [ ] Endpoints fonctionnels
- [ ] Tests passants (>80% coverage)
- [ ] Documentation OpenAPI générée
