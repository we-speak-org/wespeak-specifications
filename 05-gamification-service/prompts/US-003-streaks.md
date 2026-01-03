# US-003: Gestion des Streaks

## Contexte
Le streak compte le nombre de jours consécutifs d'apprentissage. Il se réinitialise si l'utilisateur manque un jour.

## User Story
**En tant qu'** apprenant régulier  
**Je veux** voir mon streak augmenter chaque jour d'activité  
**Afin de** rester motivé à pratiquer quotidiennement

## Critères d'Acceptation

1. **Mise à jour lastActivityDate**
   - À chaque XP gagné, mettre à jour lastActivityDate = now()
   - Cette date sert de référence pour le job nocturne

2. **Job Nocturne (00:05 UTC)**
   - Récupérer tous les UserStats
   - Pour chaque utilisateur, vérifier si lastActivityDate = hier

3. **Si activité hier**
   - currentStreak += 1
   - Si currentStreak > longestStreak → longestStreak = currentStreak
   - Publier `streak.updated`

4. **Si pas d'activité hier**
   - Si streakFreezeAvailable > 0:
     - streakFreezeAvailable -= 1
     - Publier `streak.updated` avec freezeUsed=true
   - Sinon:
     - currentStreak = 0
     - Publier `streak.updated` avec streakLost=true

5. **Badges de streak**
   - À chaque mise à jour, vérifier milestones: 7, 30, 100, 365 jours
   - Débloquer badge correspondant si atteint

## API Streak Freeze

### POST /streak/freeze
- Permet d'utiliser un freeze manuellement (protection préventive)
- Retourne erreur si streakFreezeAvailable = 0

## Scheduled Job
```java
@Scheduled(cron = "0 5 0 * * *") // 00:05 UTC
public void processStreaks() {
    // Implementation
}
```

## Tests Requis
- [ ] Test: streak incrémenté si activité hier
- [ ] Test: streak réinitialisé si pas d'activité et pas de freeze
- [ ] Test: freeze utilisé automatiquement si disponible
- [ ] Test: badge débloqué à 7 jours
