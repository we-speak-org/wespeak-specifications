# US-001: Initialisation du Profil Gamification

## Contexte
Lorsqu'un utilisateur s'inscrit et crée un profil d'apprentissage pour une langue, le système doit automatiquement créer ses stats de gamification.

## User Story
**En tant que** nouvel utilisateur  
**Je veux que** mon profil de gamification soit créé automatiquement  
**Afin de** commencer à gagner des XP dès ma première activité

## Critères d'Acceptation

1. **Création automatique sur événement user.registered**
   - Écouter l'événement `user.registered` depuis auth-service
   - Pour chaque learningProfile dans le payload, créer un UserStats
   - Initialiser tous les compteurs à 0, level à 1

2. **Données initiales**
   ```
   totalXp: 0
   weeklyXp: 0
   monthlyXp: 0
   currentStreak: 0
   longestStreak: 0
   streakFreezeAvailable: 0 (ou 2 si premium)
   level: 1
   ```

3. **Idempotence**
   - Si un UserStats existe déjà pour ce userId + langue, ne pas recréer
   - Logger un warning et ignorer

## Spécifications Techniques

### Listener Kafka
```java
@Bean
public Consumer<CloudEvent<UserRegisteredPayload>> userRegisteredListener(
        UserStatsService userStatsService) {
    return event -> userStatsService.initializeUserStats(event.getData());
}
```

### Service
- Méthode `initializeUserStats(UserRegisteredPayload payload)`
- Créer un UserStats par learningProfile
- Sauvegarder en base MongoDB

## Tests Requis
- [ ] Test unitaire: création UserStats avec valeurs par défaut
- [ ] Test intégration: réception événement Kafka et création en base
- [ ] Test idempotence: double événement ne crée pas de doublon
