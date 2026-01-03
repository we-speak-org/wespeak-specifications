# US-003: Rate Limiting

## Contexte
Protéger les services contre les abus et les attaques par déni de service.

## User Story
**En tant que** système de protection  
**Je veux** limiter le nombre de requêtes par client  
**Afin de** prévenir les abus et garantir la disponibilité

## Critères d'Acceptation

1. **Limites par défaut**
   - Utilisateur authentifié: 300 req/min
   - IP non authentifiée: 100 req/min

2. **Limites spécifiques**
   - POST /api/v1/auth/login: 10 req/min (anti brute force)
   - POST /api/v1/auth/register: 5 req/min
   - POST /api/v1/feedback/analyze: 10 req/min (coûteux)

3. **Identification du client**
   - Si authentifié: par userId
   - Sinon: par IP (X-Forwarded-For ou remote address)

4. **Headers de réponse**
   ```
   X-RateLimit-Limit: 300
   X-RateLimit-Remaining: 245
   X-RateLimit-Reset: 1704290460
   ```

5. **Réponse si limite dépassée**
   - Status: 429 Too Many Requests
   - Body: { "error": "RATE_LIMITED", "retryAfter": 45 }
   - Header: Retry-After: 45

## Implémentation avec Bucket4j

```java
@Component
public class RateLimitFilter implements GatewayFilter {
    
    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String key = resolveKey(exchange);
        Bucket bucket = buckets.computeIfAbsent(key, this::createBucket);
        
        ConsumptionProbe probe = bucket.tryConsumeAndReturnRemaining(1);
        
        if (probe.isConsumed()) {
            exchange.getResponse().getHeaders().add("X-RateLimit-Remaining", 
                String.valueOf(probe.getRemainingTokens()));
            return chain.filter(exchange);
        } else {
            return onRateLimited(exchange, probe.getNanosToWaitForRefill());
        }
    }
    
    private Bucket createBucket(String key) {
        return Bucket.builder()
            .addLimit(Bandwidth.classic(300, Refill.intervally(300, Duration.ofMinutes(1))))
            .build();
    }
}
```

## Tests Requis
- [ ] Test: requêtes sous la limite passent
- [ ] Test: requête dépassant limite → 429
- [ ] Test: headers rate limit présents
- [ ] Test: limite spécifique pour login
- [ ] Test: compteur reset après 1 minute
