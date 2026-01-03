# US-002: Recommandation de Révision

## Contexte
Quand un utilisateur fait des erreurs récurrentes, le système doit suggérer des exercices de révision.

## User Story
**En tant qu'** apprenant  
**Je veux** recevoir des suggestions de révision sur mes points faibles  
**Afin de** corriger mes erreurs et progresser

## Critères d'Acceptation

1. **Détection des points faibles**
   - Écouter `feedback.generated` depuis feedback-service
   - Extraire les erreurs par catégorie/sous-catégorie
   - Mettre à jour LearningHistory.weakAreas

2. **Seuil de déclenchement**
   - Si errorCount >= 5 pour une sous-catégorie → créer recommandation
   - Prioriser par nombre d'erreurs (plus d'erreurs = plus prioritaire)

3. **Trouver exercice de révision**
   - Appeler lesson-service pour trouver des exercices ciblant cette compétence
   - Choisir un exercice non encore fait récemment

4. **Création de la recommandation**
   - Type: "revision"
   - Priority: 2
   - Title: "Révision: {subcategory}"
   - Reason: "Vous avez fait {n} erreurs sur ce point"

5. **Éviter les répétitions**
   - Ne pas re-suggérer le même exercice pendant 7 jours
   - Maximum 2 recommandations de révision actives

## Structure WeakArea

```typescript
{
  category: "grammar",
  subcategory: "past_tense",
  errorCount: 8,
  lastErrorAt: "2026-01-03T10:00:00Z"
}
```

## Tests Requis
- [ ] Test: détection point faible après feedback
- [ ] Test: recommandation créée si seuil atteint
- [ ] Test: pas de doublon de recommandation
- [ ] Test: expiration après 7 jours sans clic
