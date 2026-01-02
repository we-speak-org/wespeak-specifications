# API Gateway - Spécifications Techniques Détaillées

## 📋 Table des Matières

1. [Vue d'Ensemble](#1-vue-densemble)
2. [Architecture Fonctionnelle](#2-architecture-fonctionnelle)
3. [Routing et Orchestration](#3-routing-et-orchestration)
4. [Sécurité et Authentification](#4-sécurité-et-authentification)
5. [Rate Limiting et Quotas](#5-rate-limiting-et-quotas)
6. [Monitoring et Observabilité](#6-monitoring-et-observabilité)
7. [Configuration](#7-configuration)

---

## 1. Vue d'Ensemble

### 1.1 Responsabilité

L'**API Gateway** est le **point d'entrée unique** de la plateforme WeSpeak, agissant comme :
- **Reverse proxy** : Routing des requêtes vers les microservices appropriés
- **Security gateway** : Authentification, autorisation, validation tokens JWT
- **Traffic manager** : Rate limiting, throttling, load balancing
- **Protocol translator** : REST → gRPC, REST → WebSocket
- **Response aggregator** : Composition de réponses multi-services
- **Observability hub** : Logs centralisés, tracing distribué, métriques

### 1.2 Principes architecturaux

**Design Patterns** :
- **API Gateway Pattern** : Point d'entrée centralisé
- **Backend for Frontend (BFF)** : Routes optimisées par client (web, mobile)
- **Circuit Breaker** : Protection contre défaillances services
- **Bulkhead** : Isolation des pools de ressources
- **Service Discovery** : Résolution dynamique des instances

**Non-responsabilités** :
- ❌ Logique métier (délégation aux microservices)
- ❌ Transformation complexe de données
- ❌ Stockage de données (cache léger autorisé)
- ❌ Orchestration business complexe (→ orchestrator service si besoin)

### 1.3 Technologies

**Stack** :
- **Framework** : Spring Cloud Gateway (Spring Boot 3.2 + WebFlux)
- **Alternative** : Kong, Apache APISIX, Traefik
- **Service Discovery** : Consul / Eureka
- **Circuit Breaker** : Resilience4j
- **Rate Limiting** : Redis + Token Bucket algorithm
- **Observability** : Micrometer + Prometheus + Grafana + Jaeger

---

## 2. Architecture Fonctionnelle

### 2.1 Vue d'ensemble

```mermaid
graph TB
    subgraph "Clients"
        A1[Web App<br/>Angular]
        A2[Mobile App<br/>React Native]
        A3[3rd Party<br/>API Consumers]
    end
    
    subgraph "API Gateway Layer"
        B[API Gateway<br/>Spring Cloud Gateway]
        
        subgraph "Gateway Components"
            C[Authentication Filter]
            D[Rate Limiter]
            E[Request Validator]
            F[Circuit Breaker]
            G[Response Aggregator]
        end
    end
    
    subgraph "Service Discovery"
        H[Consul/Eureka]
    end
    
    subgraph "Microservices"
        I[Auth Service]
        J[Lesson Service]
        K[Conversation Service]
        L[Feedback Service]
        M[Gamification Service]
        N[Recommendation Service]
    end
    
    subgraph "Supporting Services"
        O[Keycloak<br/>IAM]
        P[Redis<br/>Rate Limiting]
        Q[Kafka<br/>Event Bus]
    end
    
    A1 --> B
    A2 --> B
    A3 --> B
    
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    
    B <--> H
    H <--> I
    H <--> J
    H <--> K
    H <--> L
    H <--> M
    H <--> N
    
    C <--> O
    D <--> P
    
    style B fill:#FFE5B4
    style O fill:#E1F5FF
    style P fill:#FFE5E5
```

### 2.2 Flux de requête

```mermaid
sequenceDiagram
    participant C as Client
    participant G as API Gateway
    participant KC as Keycloak
    participant RL as Redis (Rate Limiter)
    participant SD as Service Discovery
    participant MS as Microservice
    
    C->>G: GET /api/lessons?lang=en
    
    Note over G: 1. Authentication
    G->>KC: Validate JWT Token
    KC-->>G: Token Valid + User Claims
    
    Note over G: 2. Rate Limiting
    G->>RL: Check rate limit for user
    RL-->>G: Allowed (15/20 req/min)
    
    Note over G: 3. Request Validation
    G->>G: Validate query params
    
    Note over G: 4. Service Discovery
    G->>SD: Resolve lesson-service
    SD-->>G: http://lesson-service:8082
    
    Note over G: 5. Circuit Breaker
    G->>G: Check circuit state (CLOSED)
    
    Note over G: 6. Forward Request
    G->>MS: GET /lessons?lang=en<br/>+ Headers (User-Id, Correlation-Id)
    MS-->>G: 200 OK + Lessons Data
    
    Note over G: 7. Response Enhancement
    G->>G: Add correlation-id, cache headers
    
    G-->>C: 200 OK + Enhanced Response
    
    Note over G: 8. Metrics & Logging
    G->>G: Record latency, status, user
```

### 2.3 Architecture en couches

```mermaid
graph LR
    subgraph "Request Pipeline"
        A[Pre-Filters] --> B[Route Resolution]
        B --> C[Post-Filters]
        C --> D[Response]
    end
    
    subgraph "Pre-Filters"
        A1[CORS Handler]
        A2[Authentication]
        A3[Rate Limiter]
        A4[Request Logger]
    end
    
    subgraph "Post-Filters"
        C1[Response Enhancer]
        C2[Error Handler]
        C3[Response Logger]
    end
    
    A1 --> A2
    A2 --> A3
    A3 --> A4
    
    C1 --> C2
    C2 --> C3
    
    style A fill:#FFE5B4
    style C fill:#E5FFE5
```

---

## 3. Routing et Orchestration

### 3.1 Configuration des routes

**Spring Cloud Gateway DSL** :
```yaml
spring:
  cloud:
    gateway:
      routes:
        # Auth Service
        - id: auth-service
          uri: lb://auth-service
          predicates:
            - Path=/api/auth/**, /api/users/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
        
        # Lesson Service
        - id: lesson-service
          uri: lb://lesson-service
          predicates:
            - Path=/api/lessons/**, /api/courses/**, /api/skills/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: lessonServiceCircuitBreaker
                fallbackUri: forward:/fallback/lessons
            - AddRequestHeader=X-Request-Source, gateway
        
        # Conversation Service (REST)
        - id: conversation-service-rest
          uri: lb://conversation-service
          predicates:
            - Path=/api/conversations/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 20
        
        # Conversation Service (WebSocket)
        - id: conversation-service-ws
          uri: lb:ws://conversation-service
          predicates:
            - Path=/ws/conversations/**
          filters:
            - StripPrefix=0
        
        # Feedback Service
        - id: feedback-service
          uri: lb://feedback-service
          predicates:
            - Path=/api/feedback/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: feedbackServiceCircuitBreaker
        
        # Gamification Service
        - id: gamification-service
          uri: lb://gamification-service
          predicates:
            - Path=/api/gamification/**
          filters:
            - StripPrefix=1
        
        # Recommendation Service
        - id: recommendation-service
          uri: lb://recommendation-service
          predicates:
            - Path=/api/recommendations/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 30
      
      # Global CORS
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins:
              - "https://app.wespeak.com"
              - "http://localhost:4200"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
              - OPTIONS
            allowedHeaders: "*"
            allowCredentials: true
            maxAge: 3600
```

### 3.2 Agrégation de réponses (BFF Pattern)

**Exemple : Dashboard Utilisateur**

Single request `/api/bff/dashboard` → Agrégation de :
- User profile (auth-service)
- Learning stats (gamification-service)
- Next lesson recommendation (recommendation-service)
- Active challenges (gamification-service)

```java
@RestController
@RequestMapping("/api/bff")
public class BFFController {
    
    @Autowired
    private WebClient.Builder webClientBuilder;
    
    @GetMapping("/dashboard")
    public Mono<DashboardResponse> getDashboard(@AuthenticationPrincipal Jwt jwt) {
        String userId = jwt.getSubject();
        
        // Parallel calls to microservices
        Mono<UserProfile> profileMono = webClientBuilder.build()
            .get()
            .uri("lb://auth-service/users/" + userId)
            .retrieve()
            .bodyToMono(UserProfile.class);
        
        Mono<GamificationProfile> gamificationMono = webClientBuilder.build()
            .get()
            .uri("lb://gamification-service/profile")
            .header("Authorization", "Bearer " + jwt.getTokenValue())
            .retrieve()
            .bodyToMono(GamificationProfile.class);
        
        Mono<NextLessonRecommendation> recommendationMono = webClientBuilder.build()
            .get()
            .uri("lb://recommendation-service/next-lesson")
            .header("Authorization", "Bearer " + jwt.getTokenValue())
            .retrieve()
            .bodyToMono(NextLessonRecommendation.class);
        
        // Combine responses
        return Mono.zip(profileMono, gamificationMono, recommendationMono)
            .map(tuple -> new DashboardResponse(
                tuple.getT1(), // profile
                tuple.getT2(), // gamification
                tuple.getT3()  // recommendation
            ))
            .timeout(Duration.ofSeconds(5))
            .onErrorResume(e -> Mono.just(DashboardResponse.empty()));
    }
}
```

---

## 4. Sécurité et Authentification

### 4.1 Intégration Keycloak

**Architecture OAuth2 / OIDC** :

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant GW as API Gateway
    participant KC as Keycloak
    participant MS as Microservice
    
    Note over U,KC: 1. Authentication Flow
    U->>FE: Login (email, password)
    FE->>KC: POST /auth/realms/wespeak/protocol/openid-connect/token
    KC-->>FE: Access Token (JWT) + Refresh Token
    FE->>FE: Store tokens (secure storage)
    
    Note over U,MS: 2. Authenticated API Call
    U->>FE: Request protected resource
    FE->>GW: GET /api/lessons<br/>Authorization: Bearer <token>
    
    GW->>GW: Extract JWT
    GW->>KC: Validate JWT signature + expiry
    KC-->>GW: Token valid + User claims
    
    GW->>MS: Forward request<br/>X-User-Id: uuid<br/>X-User-Roles: [student, premium]
    MS-->>GW: Response
    GW-->>FE: Response
    FE-->>U: Display data
    
    Note over U,KC: 3. Token Refresh (before expiry)
    FE->>KC: POST /token<br/>grant_type=refresh_token
    KC-->>FE: New Access Token
```

**Configuration Spring Security** :
```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        http
            .cors().and()
            .csrf().disable()
            .authorizeExchange(exchanges -> exchanges
                // Public endpoints
                .pathMatchers("/api/auth/register", "/api/auth/login").permitAll()
                .pathMatchers("/actuator/health", "/actuator/prometheus").permitAll()
                
                // Protected endpoints
                .pathMatchers("/api/**").authenticated()
                .pathMatchers("/ws/**").authenticated()
                
                .anyExchange().denyAll()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            );
        
        return http.build();
    }
    
    @Bean
    public ReactiveJwtDecoder jwtDecoder() {
        return ReactiveJwtDecoders.fromIssuerLocation(
            "https://keycloak.wespeak.com/auth/realms/wespeak"
        );
    }
    
    @Bean
    public Converter<Jwt, Mono<AbstractAuthenticationToken>> jwtAuthenticationConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(new KeycloakRoleConverter());
        return new ReactiveJwtAuthenticationConverterAdapter(converter);
    }
}
```

**Extraction des rôles Keycloak** :
```java
public class KeycloakRoleConverter implements Converter<Jwt, Collection<GrantedAuthority>> {
    
    @Override
    public Collection<GrantedAuthority> convert(Jwt jwt) {
        Map<String, Object> realmAccess = jwt.getClaim("realm_access");
        
        if (realmAccess == null) {
            return Collections.emptyList();
        }
        
        List<String> roles = (List<String>) realmAccess.get("roles");
        
        return roles.stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role.toUpperCase()))
            .collect(Collectors.toList());
    }
}
```

### 4.2 Enrichissement des requêtes

**Global Filter** : Injection de headers pour microservices
```java
@Component
public class UserContextEnrichmentFilter implements GlobalFilter, Ordered {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        return exchange.getPrincipal()
            .filter(principal -> principal instanceof JwtAuthenticationToken)
            .map(principal -> (JwtAuthenticationToken) principal)
            .map(authentication -> {
                Jwt jwt = (Jwt) authentication.getCredentials();
                
                // Enrich request with user context
                ServerHttpRequest enrichedRequest = exchange.getRequest().mutate()
                    .header("X-User-Id", jwt.getSubject())
                    .header("X-User-Email", jwt.getClaim("email"))
                    .header("X-User-Roles", String.join(",", extractRoles(jwt)))
                    .header("X-Correlation-Id", UUID.randomUUID().toString())
                    .build();
                
                return exchange.mutate().request(enrichedRequest).build();
            })
            .defaultIfEmpty(exchange)
            .flatMap(chain::filter);
    }
    
    @Override
    public int getOrder() {
        return -100; // High priority
    }
    
    private List<String> extractRoles(Jwt jwt) {
        Map<String, Object> realmAccess = jwt.getClaim("realm_access");
        return realmAccess != null 
            ? (List<String>) realmAccess.get("roles") 
            : Collections.emptyList();
    }
}
```

---

## 5. Rate Limiting et Quotas

### 5.1 Stratégie de Rate Limiting

**Token Bucket Algorithm** avec Redis :

```mermaid
graph LR
    A[Request] --> B{Check Bucket}
    B -->|Tokens Available| C[Consume 1 Token]
    B -->|No Tokens| D[429 Too Many Requests]
    C --> E[Forward Request]
    
    F[Token Refill<br/>Every Second] -.-> B
    
    style D fill:#FFE5E5
```

**Quotas par tier** :
```yaml
rate-limiting:
  tiers:
    free:
      requests-per-minute: 20
      burst-capacity: 30
      conversation-per-week: 3
    
    premium:
      requests-per-minute: 100
      burst-capacity: 150
      conversation-per-week: -1  # Unlimited
    
    enterprise:
      requests-per-minute: 500
      burst-capacity: 1000
      conversation-per-week: -1
```

**Implémentation** :
```java
@Component
public class TieredRateLimiterGatewayFilterFactory 
    extends AbstractGatewayFilterFactory<TieredRateLimiterGatewayFilterFactory.Config> {
    
    @Autowired
    private ReactiveRedisTemplate<String, String> redisTemplate;
    
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            return exchange.getPrincipal()
                .flatMap(principal -> {
                    String userId = extractUserId(principal);
                    String tier = extractUserTier(principal);
                    
                    RateLimitConfig rateLimitConfig = getRateLimitForTier(tier);
                    
                    return isAllowed(userId, rateLimitConfig)
                        .flatMap(allowed -> {
                            if (allowed) {
                                return chain.filter(exchange);
                            } else {
                                exchange.getResponse().setStatusCode(HttpStatus.TOO_MANY_REQUESTS);
                                return exchange.getResponse().setComplete();
                            }
                        });
                });
        };
    }
    
    private Mono<Boolean> isAllowed(String userId, RateLimitConfig config) {
        String key = "rate_limit:" + userId;
        
        return redisTemplate.opsForValue()
            .increment(key)
            .flatMap(count -> {
                if (count == 1) {
                    // First request, set expiry
                    return redisTemplate.expire(key, Duration.ofMinutes(1))
                        .thenReturn(true);
                }
                
                return Mono.just(count <= config.getRequestsPerMinute());
            });
    }
}
```

### 5.2 Circuit Breaker

**Resilience4j Configuration** :
```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
    
    instances:
      lessonServiceCircuitBreaker:
        baseConfig: default
      
      feedbackServiceCircuitBreaker:
        baseConfig: default
        failureRateThreshold: 70  # More lenient (AI service)
        waitDurationInOpenState: 30s
```

**États du Circuit Breaker** :
```mermaid
stateDiagram-v2
    [*] --> CLOSED
    
    CLOSED --> OPEN : Failure rate > 50%
    OPEN --> HALF_OPEN : Wait 10s
    HALF_OPEN --> CLOSED : 3 successful calls
    HALF_OPEN --> OPEN : Any failure
    
    note right of CLOSED
        Normal operation
        Requests pass through
    end note
    
    note right of OPEN
        Fail fast
        Return fallback
    end note
    
    note right of HALF_OPEN
        Test recovery
        Limited requests
    end note
```

**Fallback Response** :
```java
@RestController
public class FallbackController {
    
    @GetMapping("/fallback/lessons")
    public Mono<ResponseEntity<FallbackResponse>> lessonServiceFallback() {
        return Mono.just(ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
            .body(new FallbackResponse(
                "lesson-service",
                "Le service de leçons est temporairement indisponible. Veuillez réessayer dans quelques instants.",
                "SERVICE_UNAVAILABLE"
            ))
        );
    }
}
```

---

## 6. Monitoring et Observabilité

### 6.1 Métriques Prometheus

**Métriques exposées** :
```
# Gateway metrics
gateway_requests_total{route, method, status}
gateway_request_duration_seconds{route, method}
gateway_circuit_breaker_state{service, state}
gateway_rate_limit_exceeded_total{user_tier}

# Service health
gateway_service_discovery_instances{service}
gateway_service_latency_seconds{service}
```

**Dashboard Grafana** :
- Request throughput par route
- P95/P99 latency par service
- Circuit breaker states
- Rate limit hits par tier
- Error rate (5xx) par service

### 6.2 Distributed Tracing

**Spring Cloud Sleuth + Jaeger** :
```yaml
spring:
  sleuth:
    sampler:
      probability: 1.0  # 100% in dev, 10% in prod
  
  zipkin:
    base-url: http://jaeger:9411
    service:
      name: api-gateway
```

**Trace Context Propagation** :
```
User Request → Gateway → Microservice → Database

Trace ID: 5e8b3c2a-4f1d-4a3b-8c2d-3e4f5a6b7c8d
  └─ Span: gateway (50ms)
      └─ Span: lesson-service (30ms)
          └─ Span: mongodb-query (15ms)
```

### 6.3 Logs structurés

**Logback + JSON** :
```json
{
  "timestamp": "2025-01-15T10:30:00.123Z",
  "level": "INFO",
  "service": "api-gateway",
  "traceId": "5e8b3c2a4f1d4a3b",
  "spanId": "8c2d3e4f5a6b",
  "userId": "hash-user-123",
  "method": "GET",
  "path": "/api/lessons",
  "route": "lesson-service",
  "status": 200,
  "duration": 52,
  "userAgent": "Mozilla/5.0...",
  "clientIp": "203.0.113.42"
}
```

---

## 7. Configuration

### 7.1 Configuration complète

```yaml
# application.yml
spring:
  application:
    name: api-gateway
  
  cloud:
    gateway:
      # Discovery
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      
      # Global filters
      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Origin
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY, SERVICE_UNAVAILABLE
            backoff:
              firstBackoff: 50ms
              maxBackoff: 500ms
              factor: 2
      
      # Routes (voir section 3.1)
  
  # Security OAuth2
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://keycloak.wespeak.com/auth/realms/wespeak
          jwk-set-uri: ${spring.security.oauth2.resourceserver.jwt.issuer-uri}/protocol/openid-connect/certs
  
  # Redis (Rate Limiting)
  redis:
    host: redis
    port: 6379
    password: ${REDIS_PASSWORD}

# Consul Service Discovery
spring:
  cloud:
    consul:
      host: consul
      port: 8500
      discovery:
        instanceId: ${spring.application.name}:${random.value}
        healthCheckPath: /actuator/health
        healthCheckInterval: 15s

# Resilience4j (voir section 5.2)

# Actuator
management:
  endpoints:
    web:
      exposure:
        include: health, prometheus, metrics, gateway
  metrics:
    export:
      prometheus:
        enabled: true
  endpoint:
    health:
      show-details: always

# Logging
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    reactor.netty: INFO
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"

# Custom
gateway:
  rate-limiting:
    enabled: true
  
  circuit-breaker:
    enabled: true
  
  cors:
    allowed-origins:
      - https://app.wespeak.com
      - http://localhost:4200
```

### 7.2 Environnements

**Development** :
```yaml
spring:
  profiles: dev

gateway:
  rate-limiting:
    enabled: false  # Disable for easier testing
  
  cors:
    allowed-origins: "*"

logging:
  level:
    root: DEBUG
```

**Production** :
```yaml
spring:
  profiles: prod

gateway:
  rate-limiting:
    enabled: true
  
  circuit-breaker:
    enabled: true

resilience4j:
  circuitbreaker:
    configs:
      default:
        slidingWindowSize: 20
        waitDurationInOpenState: 30s

logging:
  level:
    root: INFO
    org.springframework.cloud.gateway: WARN
```

---

## Diagramme d'Architecture Complète

```mermaid
graph TB
    subgraph "External"
        A1[Web Browser]
        A2[Mobile App]
        A3[Third Party API]
    end
    
    subgraph "Edge Layer"
        B[Load Balancer<br/>Nginx/HAProxy]
    end
    
    subgraph "API Gateway Cluster"
        C1[Gateway Instance 1]
        C2[Gateway Instance 2]
        C3[Gateway Instance N]
        
        subgraph "Gateway Components"
            D1[Auth Filter]
            D2[Rate Limiter]
            D3[Circuit Breaker]
            D4[Request Logger]
        end
    end
    
    subgraph "IAM"
        E[Keycloak]
    end
    
    subgraph "Service Discovery"
        F[Consul]
    end
    
    subgraph "Microservices Layer"
        G1[Auth Service]
        G2[Lesson Service]
        G3[Conversation Service]
        G4[Feedback Service]
        G5[Gamification Service]
        G6[Recommendation Service]
    end
    
    subgraph "Observability"
        H1[Prometheus]
        H2[Grafana]
        H3[Jaeger]
    end
    
    subgraph "Data Layer"
        I1[Redis<br/>Rate Limiting]
        I2[MongoDB]
        I3[Kafka]
    end
    
    A1 --> B
    A2 --> B
    A3 --> B
    
    B --> C1
    B --> C2
    B --> C3
    
    C1 --> D1
    C1 --> D2
    C1 --> D3
    C1 --> D4
    
    D1 <--> E
    D2 <--> I1
    
    C1 <--> F
    C2 <--> F
    C3 <--> F
    
    F <--> G1
    F <--> G2
    F <--> G3
    F <--> G4
    F <--> G5
    F <--> G6
    
    C1 --> H1
    C2 --> H1
    C3 --> H1
    
    H1 --> H2
    C1 --> H3
    
    G1 --> I2
    G2 --> I2
    G3 --> I2
    
    G1 --> I3
    G2 --> I3
    G3 --> I3
    
    style C1 fill:#FFE5B4
    style E fill:#E1F5FF
    style F fill:#E8F5E9
    style I1 fill:#FFE5E5
```

---

**Version** : 1.0.0  
**Dernière mise à jour** : 2025-01-15  
**Auteur** : WeSpeak Product Owner AI  
**Focus** : Architecture, Sécurité et Routing
