# US-004: CORS et Headers de Sécurité

## Contexte
Le gateway doit gérer CORS pour les clients web et ajouter des headers de sécurité.

## User Story
**En tant que** application web frontend  
**Je veux** pouvoir appeler l'API depuis mon domaine  
**Afin de** fonctionner correctement dans le navigateur

## Critères d'Acceptation

1. **Configuration CORS**
   - Origins autorisées: https://wespeak.app, http://localhost:4200
   - Méthodes: GET, POST, PUT, PATCH, DELETE, OPTIONS
   - Headers autorisés: Authorization, Content-Type, X-Request-ID
   - Headers exposés: X-RateLimit-Remaining, X-RateLimit-Reset
   - Max-Age: 3600 (1 heure de cache preflight)

2. **Gestion OPTIONS (preflight)**
   - Répondre immédiatement avec 200
   - Ne pas router vers les services

3. **Headers de sécurité sur toutes les réponses**
   ```
   X-Content-Type-Options: nosniff
   X-Frame-Options: DENY
   X-XSS-Protection: 1; mode=block
   Strict-Transport-Security: max-age=31536000; includeSubDomains
   Content-Security-Policy: default-src 'self'
   ```

4. **Configuration dynamique**
   - Origines configurables via variable d'environnement
   - CORS_ORIGINS=https://wespeak.app,https://staging.wespeak.app

## Configuration Spring

```java
@Configuration
public class CorsConfig {
    
    @Value("${cors.allowed-origins}")
    private String allowedOrigins;
    
    @Bean
    public CorsWebFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(Arrays.asList(allowedOrigins.split(",")));
        config.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(Arrays.asList("Authorization", "Content-Type", "X-Request-ID"));
        config.setExposedHeaders(Arrays.asList("X-RateLimit-Remaining", "X-RateLimit-Reset"));
        config.setMaxAge(3600L);
        config.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        
        return new CorsWebFilter(source);
    }
}
```

## Tests Requis
- [ ] Test: preflight OPTIONS retourne 200 avec headers CORS
- [ ] Test: origine autorisée → headers CORS présents
- [ ] Test: origine non autorisée → pas de headers CORS
- [ ] Test: headers sécurité sur toutes les réponses
