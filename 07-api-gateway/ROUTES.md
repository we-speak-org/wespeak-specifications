# API Gateway - Configuration et Routes

## Configuration Spring Cloud Gateway

### application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway
    
  cloud:
    gateway:
      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Origin Access-Control-Allow-Credentials, RETAIN_UNIQUE
        - AddResponseHeader=X-Content-Type-Options, nosniff
        - AddResponseHeader=X-Frame-Options, DENY
        - AddResponseHeader=X-XSS-Protection, 1; mode=block
        
      routes:
        # Auth Service
        - id: auth-service
          uri: ${AUTH_SERVICE_URL:http://localhost:8081}
          predicates:
            - Path=/api/v1/auth/**
          filters:
            - name: CircuitBreaker
              args:
                name: authCircuitBreaker
                fallbackUri: forward:/fallback/auth

        # Lesson Service
        - id: lesson-service
          uri: ${LESSON_SERVICE_URL:http://localhost:8082}
          predicates:
            - Path=/api/v1/lessons/**
          filters:
            - AuthenticationFilter
            - name: CircuitBreaker
              args:
                name: lessonCircuitBreaker

        # Conversation Service
        - id: conversation-service
          uri: ${CONVERSATION_SERVICE_URL:http://localhost:8083}
          predicates:
            - Path=/api/v1/conversations/**
          filters:
            - AuthenticationFilter
            - name: CircuitBreaker
              args:
                name: conversationCircuitBreaker

        # Feedback Service
        - id: feedback-service
          uri: ${FEEDBACK_SERVICE_URL:http://localhost:8084}
          predicates:
            - Path=/api/v1/feedback/**
          filters:
            - AuthenticationFilter
            - name: CircuitBreaker
              args:
                name: feedbackCircuitBreaker

        # Gamification Service
        - id: gamification-service
          uri: ${GAMIFICATION_SERVICE_URL:http://localhost:8085}
          predicates:
            - Path=/api/v1/gamification/**
          filters:
            - AuthenticationFilter
            - name: CircuitBreaker
              args:
                name: gamificationCircuitBreaker

        # Recommendation Service
        - id: recommendation-service
          uri: ${RECOMMENDATION_SERVICE_URL:http://localhost:8086}
          predicates:
            - Path=/api/v1/recommendations/**
          filters:
            - AuthenticationFilter
            - name: CircuitBreaker
              args:
                name: recommendationCircuitBreaker

# Rate Limiting
resilience4j:
  circuitbreaker:
    instances:
      default:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10000
        permittedNumberOfCallsInHalfOpenState: 3

# JWT Configuration
jwt:
  secret: ${JWT_SECRET}
  expiration: 3600000  # 1 hour

# CORS
cors:
  allowed-origins: ${CORS_ORIGINS:http://localhost:4200}
  allowed-methods: GET,POST,PUT,PATCH,DELETE,OPTIONS
  allowed-headers: Authorization,Content-Type,X-Request-ID
  max-age: 3600
```

## Routes Publiques (Sans Authentification)

```yaml
gateway:
  public-paths:
    - /api/v1/auth/register
    - /api/v1/auth/login
    - /api/v1/auth/refresh
    - /api/v1/auth/verify-email
    - /api/v1/auth/forgot-password
    - /api/v1/auth/reset-password
    - /health
    - /actuator/health
```

## Rate Limiting Configuration

```yaml
rate-limiting:
  enabled: true
  
  default:
    requests-per-minute: 300
    
  by-ip:
    requests-per-minute: 100
    
  sensitive-endpoints:
    - path: /api/v1/auth/login
      requests-per-minute: 10
    - path: /api/v1/auth/register
      requests-per-minute: 5
    - path: /api/v1/feedback/analyze
      requests-per-minute: 10
```
