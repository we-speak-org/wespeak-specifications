# US-002: Filtre d'Authentification JWT

## Contexte
Les endpoints protégés nécessitent un token JWT valide.

## User Story
**En tant que** système de sécurité  
**Je veux** valider les tokens JWT sur chaque requête protégée  
**Afin de** protéger les ressources des accès non autorisés

## Critères d'Acceptation

1. **Endpoints publics (bypass)**
   ```
   POST /api/v1/auth/register
   POST /api/v1/auth/login
   POST /api/v1/auth/refresh
   POST /api/v1/auth/verify-email
   POST /api/v1/auth/forgot-password
   POST /api/v1/auth/reset-password
   GET /health
   ```

2. **Extraction du token**
   - Header: `Authorization: Bearer {token}`
   - Si absent → 401 UNAUTHORIZED

3. **Validation du token**
   - Vérifier signature avec JWT_SECRET
   - Vérifier expiration (exp claim)
   - Si invalide → 401 UNAUTHORIZED
   - Si expiré → 401 TOKEN_EXPIRED

4. **Extraction des claims**
   - userId (sub)
   - roles (roles array)
   - targetLanguageCode si présent

5. **Ajout headers downstream**
   - X-User-Id: {userId}
   - X-User-Roles: {roles comma-separated}

## Implémentation

```java
@Component
public class AuthenticationFilter implements GatewayFilter {
    
    private final List<String> publicPaths = List.of(
        "/api/v1/auth/register",
        "/api/v1/auth/login",
        "/api/v1/auth/refresh",
        "/health"
    );
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String path = exchange.getRequest().getPath().value();
        
        if (isPublicPath(path)) {
            return chain.filter(exchange);
        }
        
        String authHeader = exchange.getRequest().getHeaders().getFirst("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return onError(exchange, HttpStatus.UNAUTHORIZED, "UNAUTHORIZED");
        }
        
        String token = authHeader.substring(7);
        Claims claims = validateToken(token);
        
        // Add headers for downstream services
        ServerHttpRequest modifiedRequest = exchange.getRequest().mutate()
            .header("X-User-Id", claims.getSubject())
            .header("X-User-Roles", String.join(",", claims.get("roles", List.class)))
            .build();
            
        return chain.filter(exchange.mutate().request(modifiedRequest).build());
    }
}
```

## Tests Requis
- [ ] Test: endpoint public accessible sans token
- [ ] Test: endpoint protégé rejette requête sans token
- [ ] Test: token valide → headers ajoutés
- [ ] Test: token expiré → 401 avec message spécifique
- [ ] Test: token malformé → 401
