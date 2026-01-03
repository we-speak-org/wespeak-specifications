# US-005: Leaderboards

## Contexte
Les leaderboards créent une compétition saine entre apprenants d'une même langue.

## User Story
**En tant qu'** apprenant compétitif  
**Je veux** voir mon classement par rapport aux autres  
**Afin de** me motiver à progresser

## Critères d'Acceptation

1. **Types de leaderboards**
   - Weekly: basé sur weeklyXp, reset chaque lundi 00:00 UTC
   - Monthly: basé sur monthlyXp, reset le 1er de chaque mois
   - Global: basé sur totalXp, jamais reset

2. **Filtrage par langue**
   - Classement séparé par targetLanguageCode
   - Un utilisateur apparaît dans le classement de chaque langue qu'il apprend

3. **API GET /leaderboard**
   - Paramètres: language, type (weekly/monthly/global), limit
   - Retourner top N utilisateurs avec rank, displayName, avatarUrl, xp, level
   - Inclure la position de l'utilisateur connecté

4. **Cache des classements**
   - Calculer et mettre en cache les top 100
   - TTL: 5 minutes
   - Recalculer en arrière-plan

5. **Reset hebdomadaire (Job)**
   - Chaque lundi à 00:00 UTC
   - weeklyXp = 0 pour tous les UserStats

6. **Reset mensuel (Job)**
   - Le 1er de chaque mois à 00:00 UTC
   - monthlyXp = 0 pour tous les UserStats

## Response Format

```json
{
  "type": "weekly",
  "targetLanguageCode": "en",
  "rankings": [
    { "rank": 1, "userId": "...", "displayName": "Marie D.", "xp": 850, "level": 12 },
    { "rank": 2, "userId": "...", "displayName": "Jean P.", "xp": 720, "level": 10 }
  ],
  "currentUserRank": { "rank": 15, "xp": 320 }
}
```

## Scheduled Jobs

```java
@Scheduled(cron = "0 0 0 * * MON") // Monday 00:00
public void resetWeeklyXp() {
    userStatsRepository.resetAllWeeklyXp();
}

@Scheduled(cron = "0 0 0 1 * *") // 1st of month 00:00
public void resetMonthlyXp() {
    userStatsRepository.resetAllMonthlyXp();
}
```

## Tests Requis
- [ ] Test: classement trié correctement par XP décroissant
- [ ] Test: filtrage par langue fonctionne
- [ ] Test: position utilisateur correcte
- [ ] Test: reset hebdomadaire remet à 0
