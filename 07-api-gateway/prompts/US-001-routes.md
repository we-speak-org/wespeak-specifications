# US-001: Configuration des Routes

## Contexte
Le gateway doit router correctement les requêtes vers les microservices.

## User Story
**En tant que** développeur frontend  
**Je veux** une API unifiée avec un seul point d'entrée  
**Afin de** simplifier les appels aux différents services

## Critères d'Acceptation

1. **Routes configurées**
   - /api/v1/auth/** → auth-service:8081
   - /api/v1/lessons/** → lesson-service:8082
   - /api/v1/conversations/** → conversation-service:8083
   - /api/v1/feedback/** → feedback-service:8084
   - /api/v1/gamification/** → gamification-service:8085
   - /api/v1/recommendations/** → recommendation-service:8086

2. **Préservation du path**
   - Le path complet doit être transmis au service
   - Ex: /api/v1/lessons/courses → lesson-service/api/v1/lessons/courses

3. **Transmission des headers**
   - Tous les headers doivent être transmis
   - Ajouter X-Request-ID si absent (UUID)

4. **Gestion des erreurs de routage**
   - 404 si aucune route ne correspond
   - 502 si le service est indisponible

## Configuration Spring Cloud Gateway

```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("auth-service", r -> r
            .path("/api/v1/auth/**")
            .uri("http://auth-service:8081"))
        .route("lesson-service", r -> r
            .path("/api/v1/lessons/**")
            .filters(f -> f.filter(authFilter))
            .uri("http://lesson-service:8082"))
        // ... autres routes
        .build();
}
```

## Tests Requis
- [ ] Test: route vers chaque service fonctionne
- [ ] Test: path préservé correctement
- [ ] Test: headers transmis
- [ ] Test: 404 pour route inconnue
