# US-001: Recommandation de Prochaine Leçon

## Contexte
L'utilisateur doit toujours savoir quelle est la prochaine étape de son apprentissage.

## User Story
**En tant qu'** apprenant  
**Je veux** voir quelle leçon je dois faire ensuite  
**Afin de** continuer ma progression sans me perdre

## Critères d'Acceptation

1. **Calcul de la prochaine leçon**
   - Récupérer le cours actuel de l'utilisateur (via lesson-service API)
   - Trouver la première leçon non complétée dans l'ordre
   - Vérifier que la leçon précédente est complétée (prérequis)

2. **Création de la recommandation**
   - Type: "next_lesson"
   - Priority: 1 (haute)
   - Title: nom de la leçon
   - Reason: "Continuez votre progression" ou "Nouvelle unité débloquée"

3. **Mise à jour automatique**
   - Sur réception de `lesson.completed`, recalculer
   - Supprimer l'ancienne recommandation "next_lesson"
   - Créer la nouvelle

4. **Cas particuliers**
   - Si toutes les leçons du cours sont complétées → pas de recommandation
   - Si c'est la première leçon d'une nouvelle unité → raison = "Nouvelle unité débloquée"

## Appel API vers Lesson Service

```
GET /api/v1/lessons/progress?userId={userId}&language={lang}
→ Returns: current course, completed lessons, next available lesson
```

## Tests Requis
- [ ] Test: recommandation créée pour prochaine leçon
- [ ] Test: mise à jour après complétion
- [ ] Test: gestion fin de cours
- [ ] Test: prérequis respectés
