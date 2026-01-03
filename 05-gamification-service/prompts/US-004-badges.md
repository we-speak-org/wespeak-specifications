# US-004: Système de Badges

## Contexte
Les badges récompensent les accomplissements des utilisateurs et donnent des XP bonus.

## User Story
**En tant qu'** apprenant  
**Je veux** débloquer des badges pour mes accomplissements  
**Afin de** me sentir récompensé et collectionner des achievements

## Critères d'Acceptation

1. **Vérification automatique**
   - À chaque mise à jour de UserStats, vérifier les conditions de badges
   - Ne vérifier que les badges non encore débloqués par l'utilisateur

2. **Types de conditions**
   - lessons_completed: nombre de leçons
   - exercises_completed: nombre d'exercices
   - perfect_lessons: nombre de leçons parfaites
   - streak_days: nombre de jours de streak
   - conversation_minutes: minutes de conversation
   - first_X: première action de type X

3. **Déblocage de badge**
   - Créer UserBadge avec unlockedAt = now()
   - Ajouter xpReward au UserStats
   - Créer XpTransaction avec source="badge"
   - Publier événement `badge.unlocked`

4. **API Liste des badges**
   - GET /badges: tous les badges disponibles
   - GET /badges/my: badges de l'utilisateur avec date de déblocage
   - Inclure pourcentage de progression vers prochain badge

## Badges à Implémenter (MVP)

```javascript
const BADGES = [
  { code: "first_lesson", condition: { type: "lessons_completed", value: 1 }, xp: 10, rarity: "common" },
  { code: "lesson_master_10", condition: { type: "lessons_completed", value: 10 }, xp: 25, rarity: "common" },
  { code: "lesson_master_50", condition: { type: "lessons_completed", value: 50 }, xp: 50, rarity: "uncommon" },
  { code: "first_conversation", condition: { type: "conversations_completed", value: 1 }, xp: 20, rarity: "common" },
  { code: "streak_7", condition: { type: "streak_days", value: 7 }, xp: 50, rarity: "common" },
  { code: "streak_30", condition: { type: "streak_days", value: 30 }, xp: 200, rarity: "rare" },
]
```

## Tests Requis
- [ ] Test: badge débloqué quand condition atteinte
- [ ] Test: XP bonus ajouté
- [ ] Test: pas de double déblocage
- [ ] Test: événement publié correctement
