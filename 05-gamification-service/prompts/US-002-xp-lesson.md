# US-002: Attribution d'XP sur Complétion de Leçon

## Contexte
Chaque leçon complétée doit rapporter des XP à l'utilisateur, avec un bonus si la leçon est parfaite.

## User Story
**En tant qu'** apprenant  
**Je veux** gagner des XP quand je complète une leçon  
**Afin de** voir ma progression et monter de niveau

## Critères d'Acceptation

1. **Réception événement lesson.completed**
   - Écouter topic `lesson.events` avec eventType `lesson.completed`
   - Extraire: userId, targetLanguageCode, score, isPerfect

2. **Calcul des XP**
   - XP de base: 15
   - Bonus parfait (isPerfect=true): +10
   - Total max: 25 XP par leçon

3. **Mise à jour UserStats**
   - totalXp += xpEarned
   - weeklyXp += xpEarned
   - monthlyXp += xpEarned
   - lessonsCompleted += 1
   - Si isPerfect: perfectLessons += 1
   - lastActivityDate = now()

4. **Création XpTransaction**
   - Enregistrer la transaction pour historique
   - source: "lesson", sourceId: lessonId

5. **Vérification niveau**
   - Recalculer le niveau basé sur totalXp
   - Si nouveau niveau > ancien niveau → publier `level.up`

6. **Publication événement**
   - Publier `xp.earned` avec amount, totalXp, newLevel (si applicable)

## Règle de Calcul du Niveau
```
Niveau N nécessite: 100 * N * (N-1) / 2 XP
Niveau 1: 0 XP
Niveau 2: 100 XP
Niveau 3: 300 XP
Niveau 4: 600 XP
...
```

## Tests Requis
- [ ] Test unitaire: calcul XP avec/sans bonus
- [ ] Test unitaire: calcul niveau
- [ ] Test intégration: événement → mise à jour stats → événement publié
- [ ] Test: vérifier lastActivityDate mise à jour
