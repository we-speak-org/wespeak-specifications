# Auth Service - Checklist Développement (Emergent.sh / Cursor AI)

## 🎯 Objectif

Générer le code Spring Boot 4 complet pour le **Auth Service** avec Spring WebFlux (reactive), MongoDB, Kafka, et intégration Keycloak.

---

## Phase 1 : Setup Initial du Projet ✅

### 1.1 Générer le projet Spring Boot

```
Génère un projet Spring Boot 4.0 avec les caractéristiques suivantes :

- **Java** : 21
- **Build Tool** : Maven
- **Packaging** : Jar
- **Group ID** : com.wespeak
- **Artifact ID** : auth-service
- **Version** : 1.0.0-SNAPSHOT
- **Port** : 8081

**Dépendances** :
- spring-boot-starter-webflux (reactive web)
- spring-boot-starter-data-mongodb-reactive
- spring-boot-starter-data-redis-reactive
- spring-kafka
- spring-boot-starter-security
- spring-boot-starter-oauth2-resource-server
- spring-boot-starter-validation
- spring-boot-starter-actuator
- lombok
- jackson-databind
- jackson-datatype-jsr310

Structure de packages :
- com.wespeak.auth.controller
- com.wespeak.auth.service
- com.wespeak.auth.repository
- com.wespeak.auth.model
- com.wespeak.auth.dto
- com.wespeak.auth.event
- com.wespeak.auth.config
- com.wespeak.auth.exception

Fichier application.yml avec :
- server.port: 8081
- spring.application.name: auth-service
- MongoDB URI (variable d'environnement)
- Redis host/port
- Kafka bootstrap servers
- Logging niveau INFO
```

**Checklist** :
- [ ] Projet Maven généré
- [ ] Toutes les dépendances ajoutées dans pom.xml
- [ ] Structure de packages créée
- [ ] application.yml configuré
- [ ] AuthServiceApplication.java avec @SpringBootApplication

### 1.2 Configurer application.yml

```yaml
server:
  port: 8081

spring:
  application:
    name: auth-service
  
  data:
    mongodb:
      uri: ${MONGODB_URI:mongodb://localhost:27017/wespeak_auth}
      database: wespeak_auth
  
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
  
  kafka:
    bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS:localhost:9092}
    consumer:
      group-id: auth-service
      auto-offset-reset: earliest
      enable-auto-commit: false
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer

  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${KEYCLOAK_ISSUER_URI:http://localhost:8080/realms/wespeak}
          jwk-set-uri: ${KEYCLOAK_JWK_SET_URI:http://localhost:8080/realms/wespeak/protocol/openid-connect/certs}

logging:
  level:
    com.wespeak: INFO
    org.springframework.data.mongodb: DEBUG
    org.springframework.kafka: INFO

management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      show-details: always
```

---

## Phase 2 : Modèles de Données (Entités MongoDB) ✅

### 2.1 Créer entité UserProfile

**Prompt** :
```
Crée l'entité MongoDB UserProfile avec Spring Data Reactive :

@Document(collection = "user_profiles")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserProfile {
    @Id
    private String id; // MongoDB ObjectId
    
    @Indexed(unique = true)
    private String keycloakUserId; // UUID Keycloak
    
    @Indexed(unique = true)
    @Email
    private String email;
    
    @NotNull
    @Size(min = 2, max = 100)
    private String displayName;
    
    private String avatarUrl;
    private LocalDate dateOfBirth;
    
    @Pattern(regexp = "^[A-Z]{2}$")
    private String country;
    
    private String timezone; // IANA timezone (ex: Europe/Paris)
    
    @NotNull
    @Pattern(regexp = "^[a-z]{2}$")
    private String uiLanguageCode; // ISO 639-1
    
    @NotNull
    private SubscriptionTier subscriptionTier; // enum FREE, PREMIUM, ENTERPRISE
    
    private LocalDateTime subscriptionExpiresAt;
    private Boolean subscriptionAutoRenew;
    
    private Boolean emailVerified;
    private Boolean onboardingCompleted;
    private LocalDateTime onboardingCompletedAt;
    
    @NotNull
    private Credits credits; // embedded object
    
    @NotNull
    private Preferences preferences; // embedded object
    
    private LocalDateTime lastLoginAt;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
}

// Enum SubscriptionTier
public enum SubscriptionTier {
    FREE, PREMIUM, ENTERPRISE
}

// Embedded object Credits
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Credits {
    private Integer conversationsRemaining;
    private LocalDateTime conversationsResetAt;
    private Integer aiMinutesRemaining;
    private List<String> premiumFeaturesAccess;
}

// Embedded object Preferences
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Preferences {
    private Boolean notificationsEnabled;
    private EmailDigest emailDigest; // enum DAILY, WEEKLY, NEVER
    private Theme theme; // enum LIGHT, DARK, AUTO
    private Boolean autoPlayAudio;
}

Ajoute les annotations Spring Data MongoDB :
- @EnableMongoAuditing dans la classe de configuration
- @CreatedDate et @LastModifiedDate pour createdAt/updatedAt
```

**Checklist** :
- [ ] UserProfile.java créé
- [ ] SubscriptionTier enum créé
- [ ] Credits embedded object créé
- [ ] Preferences embedded object créé
- [ ] EmailDigest enum créé
- [ ] Theme enum créé
- [ ] Annotations Lombok ajoutées
- [ ] Annotations Bean Validation ajoutées
- [ ] Indexes MongoDB définis

### 2.2 Créer entité LearningProfile

**Prompt** :
```
Crée l'entité MongoDB LearningProfile :

@Document(collection = "learning_profiles")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class LearningProfile {
    @Id
    private String id;
    
    @Indexed
    @NotNull
    private String userId; // Référence UserProfile._id
    
    @Indexed
    @NotNull
    private String keycloakUserId;
    
    @NotNull
    @Pattern(regexp = "^[a-z]{2}(-[A-Z]{2})?$")
    private String nativeLanguageCode; // ex: fr, en, pt-BR
    
    @NotNull
    @Pattern(regexp = "^[a-z]{2}(-[A-Z]{2})?$")
    private String targetLanguageCode;
    
    @NotNull
    private CefrLevel currentLevel; // enum A1, A2, B1, B2, C1, C2
    
    private CefrLevel assessedLevel;
    
    @NotNull
    private LearningGoal goal; // enum WORK, TRAVEL, STUDIES, PERSONAL, OTHER
    
    @Size(max = 500)
    private String goalDescription;
    
    @Pattern(regexp = "^[a-z]{2}-[A-Z]{2}$")
    private String preferredAccent; // ex: en-US, en-GB
    
    @NotNull
    @Min(0)
    @Max(1680)
    private Integer weeklyGoalMinutes;
    
    @NotNull
    private Boolean isActive;
    
    @NotNull
    private Boolean isPrimary;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
}

// Enum CefrLevel
public enum CefrLevel {
    A1, A2, B1, B2, C1, C2
}

// Enum LearningGoal
public enum LearningGoal {
    WORK, TRAVEL, STUDIES, PERSONAL, OTHER
}

Ajoute un index composite unique :
- userId + targetLanguageCode (un seul profil par langue par user)
```

**Checklist** :
- [ ] LearningProfile.java créé
- [ ] CefrLevel enum créé
- [ ] LearningGoal enum créé
- [ ] Index composite userId + targetLanguageCode défini
- [ ] Annotations validation ajoutées

---

## Phase 3 : Repositories (Spring Data MongoDB Reactive) ✅

### 3.1 Créer UserProfileRepository

**Prompt** :
```
Crée l'interface UserProfileRepository avec Spring Data MongoDB Reactive :

public interface UserProfileRepository extends ReactiveMongoRepository<UserProfile, String> {
    
    Mono<UserProfile> findByKeycloakUserId(String keycloakUserId);
    
    Mono<UserProfile> findByEmail(String email);
    
    Mono<Boolean> existsByKeycloakUserId(String keycloakUserId);
    
    Flux<UserProfile> findBySubscriptionTier(SubscriptionTier tier);
}
```

**Checklist** :
- [ ] UserProfileRepository.java créé
- [ ] Méthodes de requête définies
- [ ] Types de retour Mono/Flux corrects

### 3.2 Créer LearningProfileRepository

**Prompt** :
```
Crée l'interface LearningProfileRepository :

public interface LearningProfileRepository extends ReactiveMongoRepository<LearningProfile, String> {
    
    Flux<LearningProfile> findByUserId(String userId);
    
    Flux<LearningProfile> findByKeycloakUserId(String keycloakUserId);
    
    Mono<LearningProfile> findByUserIdAndTargetLanguageCode(String userId, String targetLanguageCode);
    
    Mono<Long> countByUserId(String userId);
    
    Mono<LearningProfile> findByUserIdAndIsPrimary(String userId, Boolean isPrimary);
}
```

**Checklist** :
- [ ] LearningProfileRepository.java créé
- [ ] Méthodes de requête définies

---

## Phase 4 : DTOs (Data Transfer Objects) ✅

### 4.1 Créer DTOs Request

**Prompt** :
```
Crée les DTOs de requête suivants avec Bean Validation :

1. UpdateUserProfileRequest
   - displayName (optional)
   - avatarUrl (optional)
   - timezone (optional)
   - country (optional)
   - uiLanguageCode (optional)
   - preferences (optional PreferencesDto)

2. CreateLearningProfileRequest
   - nativeLanguageCode (required, @Pattern)
   - targetLanguageCode (required, @Pattern)
   - currentLevel (required, @NotNull)
   - goal (required, @NotNull)
   - goalDescription (optional, @Size max 500)
   - preferredAccent (optional, @Pattern)
   - weeklyGoalMinutes (required, @Min 0, @Max 1680)

3. UpdateLearningProfileRequest
   - currentLevel (optional)
   - weeklyGoalMinutes (optional)
   - isPrimary (optional)
   - goalDescription (optional)

Utilise @Data et @Builder de Lombok.
```

**Checklist** :
- [ ] UpdateUserProfileRequest.java
- [ ] CreateLearningProfileRequest.java
- [ ] UpdateLearningProfileRequest.java
- [ ] Annotations de validation ajoutées

### 4.2 Créer DTOs Response

**Prompt** :
```
Crée les DTOs de réponse :

1. UserProfileDto (sans données sensibles)
   - id, keycloakUserId, email, displayName, avatarUrl, country, timezone
   - uiLanguageCode, subscriptionTier, subscriptionExpiresAt
   - emailVerified, onboardingCompleted, preferences
   - createdAt, lastLoginAt

2. LearningProfileDto
   - id, nativeLanguageCode, targetLanguageCode, currentLevel
   - goal, goalDescription, preferredAccent, weeklyGoalMinutes
   - isActive, isPrimary, createdAt

3. CreditsDto
   - subscriptionTier, credits (conversationsRemaining, etc.)
   - quotas (lessonsPerDay, conversationsPerWeek, etc.)

Ajoute des mappers statiques (ex: UserProfileDto.from(UserProfile entity)).
```

**Checklist** :
- [ ] UserProfileDto.java
- [ ] LearningProfileDto.java
- [ ] CreditsDto.java
- [ ] Méthodes de mapping créées

---

## Phase 5 : Services (Business Logic) ✅

### 5.1 Créer UserProfileService

**Prompt** :
```
Crée le service UserProfileService avec les méthodes suivantes :

@Service
@RequiredArgsConstructor
public class UserProfileService {
    private final UserProfileRepository userProfileRepository;
    private final RedisTemplate<String, UserProfile> redisTemplate;
    
    // Récupérer profil par Keycloak User ID (avec cache Redis)
    public Mono<UserProfile> getByKeycloakUserId(String keycloakUserId);
    
    // Créer profil utilisateur (appelé par Kafka consumer)
    public Mono<UserProfile> createUserProfile(String keycloakUserId, String email, String displayName);
    
    // Mettre à jour profil
    public Mono<UserProfile> updateUserProfile(String keycloakUserId, UpdateUserProfileRequest request);
    
    // Supprimer profil (soft delete)
    public Mono<Void> deleteUserProfile(String keycloakUserId);
    
    // Mettre à jour lastLoginAt
    public Mono<Void> updateLastLogin(String keycloakUserId);
}

Implémente le caching Redis :
- Cache profil après récupération (TTL 1 heure)
- Invalider cache lors de la mise à jour
- Clé Redis : "user:profile:{keycloakUserId}"
```

**Checklist** :
- [ ] UserProfileService.java créé
- [ ] Méthodes CRUD implémentées
- [ ] Caching Redis implémenté
- [ ] Gestion des erreurs (Mono.error)

### 5.2 Créer LearningProfileService

**Prompt** :
```
Crée le service LearningProfileService :

@Service
@RequiredArgsConstructor
public class LearningProfileService {
    private final LearningProfileRepository learningProfileRepository;
    private final UserProfileService userProfileService;
    private final KafkaProducer kafkaProducer;
    
    // Lister tous les profils d'un utilisateur
    public Flux<LearningProfile> getProfilesByUser(String keycloakUserId);
    
    // Créer un profil d'apprentissage
    public Mono<LearningProfile> createProfile(String keycloakUserId, CreateLearningProfileRequest request);
    
    // Mettre à jour un profil
    public Mono<LearningProfile> updateProfile(String profileId, String keycloakUserId, UpdateLearningProfileRequest request);
    
    // Supprimer un profil
    public Mono<Void> deleteProfile(String profileId, String keycloakUserId);
    
    // Valider contraintes métier
    private Mono<Void> validateProfileCreation(String userId);
}

Règles métier à implémenter :
- Maximum 5 profils par utilisateur
- Un seul profil "primary" par utilisateur
- Cannot delete primary profile (must set another as primary first)
- nativeLanguageCode != targetLanguageCode

Publier événement Kafka "learning_profile.created" après création.
```

**Checklist** :
- [ ] LearningProfileService.java créé
- [ ] Validation max 5 profils
- [ ] Validation profil primary unique
- [ ] Publication événement Kafka
- [ ] Gestion des erreurs métier

### 5.3 Créer CreditsService

**Prompt** :
```
Crée le service CreditsService pour gérer les crédits/quotas :

@Service
@RequiredArgsConstructor
public class CreditsService {
    private final UserProfileRepository userProfileRepository;
    private final KafkaProducer kafkaProducer;
    
    // Récupérer crédits d'un utilisateur
    public Mono<Credits> getCredits(String keycloakUserId);
    
    // Consommer un crédit (appelé par conversation-service)
    public Mono<ConsumeCreditsResponse> consumeCredit(String userId, CreditType creditType, Integer amount);
    
    // Reset crédits hebdomadaires (scheduled job)
    @Scheduled(cron = "0 0 0 * * MON")
    public void resetWeeklyCredits();
}

Règles :
- Free tier : 3 conversations/semaine
- Premium tier : -1 (illimité)
- Si quota épuisé → Mono.error(QuotaExceededException)
- Publier événement "credits.consumed" après consommation
```

**Checklist** :
- [ ] CreditsService.java créé
- [ ] Logique consommation de crédits
- [ ] Scheduled job reset hebdomadaire
- [ ] Gestion quota exceeded
- [ ] Publication événement Kafka

---

## Phase 6 : Controllers (REST API) ✅

### 6.1 Créer UserProfileController

**Prompt** :
```
Crée le contrôleur REST UserProfileController avec Spring WebFlux :

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserProfileController {
    private final UserProfileService userProfileService;
    
    @GetMapping("/me")
    public Mono<ResponseEntity<UserProfileDto>> getCurrentUser(@AuthenticationPrincipal Jwt jwt);
    
    @PutMapping("/me")
    public Mono<ResponseEntity<UserProfileDto>> updateCurrentUser(
        @AuthenticationPrincipal Jwt jwt,
        @Valid @RequestBody UpdateUserProfileRequest request);
    
    @DeleteMapping("/me")
    public Mono<ResponseEntity<Void>> deleteCurrentUser(@AuthenticationPrincipal Jwt jwt);
}

Extraire keycloakUserId depuis JWT :
- jwt.getSubject() contient le Keycloak User ID
```

**Checklist** :
- [ ] UserProfileController.java créé
- [ ] Endpoints GET, PUT, DELETE implémentés
- [ ] Extraction JWT keycloakUserId
- [ ] Conversion Entity → DTO
- [ ] Gestion des erreurs (404, 400)

### 6.2 Créer LearningProfileController

**Prompt** :
```
Crée le contrôleur LearningProfileController :

@RestController
@RequestMapping("/api/learning-profiles")
@RequiredArgsConstructor
public class LearningProfileController {
    private final LearningProfileService learningProfileService;
    
    @GetMapping
    public Mono<ResponseEntity<ProfilesListResponse>> getProfiles(@AuthenticationPrincipal Jwt jwt);
    
    @PostMapping
    public Mono<ResponseEntity<LearningProfileDto>> createProfile(
        @AuthenticationPrincipal Jwt jwt,
        @Valid @RequestBody CreateLearningProfileRequest request);
    
    @PutMapping("/{id}")
    public Mono<ResponseEntity<LearningProfileDto>> updateProfile(
        @PathVariable String id,
        @AuthenticationPrincipal Jwt jwt,
        @Valid @RequestBody UpdateLearningProfileRequest request);
    
    @DeleteMapping("/{id}")
    public Mono<ResponseEntity<Void>> deleteProfile(
        @PathVariable String id,
        @AuthenticationPrincipal Jwt jwt);
}
```

**Checklist** :
- [ ] LearningProfileController.java créé
- [ ] Endpoints CRUD implémentés
- [ ] Validation des requêtes
- [ ] Codes HTTP corrects (201 Created, 204 No Content)

### 6.3 Créer CreditsController

**Prompt** :
```
Crée le contrôleur CreditsController :

@RestController
@RequestMapping("/api/credits")
@RequiredArgsConstructor
public class CreditsController {
    private final CreditsService creditsService;
    
    @GetMapping
    public Mono<ResponseEntity<CreditsDto>> getCredits(@AuthenticationPrincipal Jwt jwt);
    
    // Endpoint interne (service-to-service)
    @PostMapping("/consume")
    public Mono<ResponseEntity<ConsumeCreditsResponse>> consumeCredits(
        @RequestHeader("X-API-Key") String apiKey,
        @Valid @RequestBody ConsumeCreditsRequest request);
}

Valider X-API-Key pour l'endpoint /consume (service-to-service).
```

**Checklist** :
- [ ] CreditsController.java créé
- [ ] GET /credits implémenté
- [ ] POST /consume avec validation API Key
- [ ] Gestion QuotaExceededException → 403 Forbidden

---

## Phase 7 : Kafka Integration ✅

### 7.1 Créer KafkaProducer

**Prompt** :
```
Crée le service KafkaProducer pour publier des événements :

@Service
@RequiredArgsConstructor
public class KafkaProducer {
    private final KafkaTemplate<String, String> kafkaTemplate;
    private final ObjectMapper objectMapper;
    
    public void publishUserProfileCreated(UserProfile userProfile);
    public void publishLearningProfileCreated(LearningProfile learningProfile);
    public void publishCreditsConsumed(String userId, CreditType creditType, Integer amount, Integer remaining);
    
    private void sendEvent(String topic, String key, Object payload);
}

Format des événements :
{
  "eventType": "...",
  "version": "1.0",
  "timestamp": "2025-01-15T10:30:00Z",
  "payload": { ... },
  "metadata": {
    "correlationId": "uuid",
    "source": "auth-service"
  }
}

Topics :
- user.events (pour tous les événements users et learning_profiles)
```

**Checklist** :
- [ ] KafkaProducer.java créé
- [ ] Méthodes publish implémentées
- [ ] Format JSON standard
- [ ] Gestion des erreurs d'envoi

### 7.2 Créer KeycloakEventConsumer

**Prompt** :
```
Crée le consumer Kafka pour événements Keycloak :

@Service
@RequiredArgsConstructor
public class KeycloakEventConsumer {
    private final UserProfileService userProfileService;
    private final ObjectMapper objectMapper;
    
    @KafkaListener(topics = "keycloak.admin.events", groupId = "auth-service")
    public void consumeKeycloakEvent(String message) {
        // Parser l'événement JSON
        // Switch selon event.type : REGISTER, VERIFY_EMAIL, UPDATE_EMAIL, DELETE_ACCOUNT, LOGIN
    }
    
    private void handleRegister(KeycloakAdminEvent event);
    private void handleVerifyEmail(KeycloakAdminEvent event);
    private void handleUpdateEmail(KeycloakAdminEvent event);
    private void handleDeleteAccount(KeycloakAdminEvent event);
    private void handleLogin(KeycloakAdminEvent event);
}

Implémenter l'idempotence :
- Stocker event.id dans Redis avec TTL 7 jours
- Skip si event.id déjà traité
```

**Checklist** :
- [ ] KeycloakEventConsumer.java créé
- [ ] @KafkaListener configuré
- [ ] Handlers pour chaque type d'événement
- [ ] Idempotence via Redis
- [ ] Gestion des erreurs (retry, dead-letter)

---

## Phase 8 : Configuration Spring Security ✅

### 8.1 Configurer OAuth2 Resource Server

**Prompt** :
```
Crée la configuration Spring Security pour valider JWT Keycloak :

@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        return http
            .authorizeExchange(exchanges -> exchanges
                .pathMatchers("/actuator/health").permitAll()
                .pathMatchers("/api/**").authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
            .csrf(ServerHttpSecurity.CsrfSpec::disable)
            .build();
    }
    
    @Bean
    public ReactiveJwtDecoder jwtDecoder() {
        return ReactiveJwtDecoders.fromIssuerLocation(keycloakIssuerUri);
    }
}
```

**Checklist** :
- [ ] SecurityConfig.java créé
- [ ] JWT validation configurée
- [ ] Endpoints /api/** protégés
- [ ] /actuator/health public

---

## Phase 9 : Exception Handling ✅

### 9.1 Créer exceptions personnalisées

**Prompt** :
```
Crée les exceptions métier :

1. UserNotFoundException extends RuntimeException
2. LearningProfileNotFoundException extends RuntimeException
3. MaxProfilesReachedException extends RuntimeException
4. QuotaExceededException extends RuntimeException
5. CannotDeletePrimaryProfileException extends RuntimeException

Ajoute des constructeurs avec message personnalisé.
```

### 9.2 Créer GlobalExceptionHandler

**Prompt** :
```
Crée un gestionnaire d'exceptions global avec @RestControllerAdvice :

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    public Mono<ResponseEntity<ErrorResponse>> handleUserNotFound(UserNotFoundException ex);
    
    @ExceptionHandler(QuotaExceededException.class)
    public Mono<ResponseEntity<ErrorResponse>> handleQuotaExceeded(QuotaExceededException ex);
    
    // Autres handlers...
}

Format de réponse d'erreur :
{
  "error": "ERROR_CODE",
  "message": "Human readable message",
  "timestamp": "2025-01-15T10:30:00Z"
}
```

**Checklist** :
- [ ] Exceptions personnalisées créées
- [ ] GlobalExceptionHandler créé
- [ ] Mapping exception → HTTP status
- [ ] Format ErrorResponse standardisé

---

## Phase 10 : Tests ✅

### 10.1 Tests Unitaires

**Prompt** :
```
Crée des tests unitaires avec JUnit 5 et Mockito :

1. UserProfileServiceTest
   - testGetByKeycloakUserId_Success
   - testGetByKeycloakUserId_NotFound
   - testCreateUserProfile_Success
   - testUpdateUserProfile_Success

2. LearningProfileServiceTest
   - testCreateProfile_Success
   - testCreateProfile_MaxProfilesReached
   - testCreateProfile_DuplicateTargetLanguage

3. CreditsServiceTest
   - testConsumeCredit_Success
   - testConsumeCredit_QuotaExceeded

Utilise :
- @ExtendWith(MockitoExtension.class)
- @Mock pour les dépendances
- @InjectMocks pour le service testé
- StepVerifier pour tester Mono/Flux
```

**Checklist** :
- [ ] Tests UserProfileService
- [ ] Tests LearningProfileService
- [ ] Tests CreditsService
- [ ] Coverage > 80%

### 10.2 Tests d'Intégration (Testcontainers)

**Prompt** :
```
Crée des tests d'intégration avec Testcontainers MongoDB :

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@Testcontainers
public class UserProfileIntegrationTest {
    
    @Container
    static MongoDBContainer mongoDBContainer = new MongoDBContainer("mongo:7.0");
    
    @Autowired
    private WebTestClient webTestClient;
    
    @Test
    void testCreateAndRetrieveUserProfile() {
        // Test complet : POST puis GET
    }
}
```

**Checklist** :
- [ ] Tests d'intégration MongoDB
- [ ] Tests d'intégration API REST
- [ ] Testcontainers configurés

---

## Phase 11 : Dockerfile et Déploiement ✅

### 11.1 Créer Dockerfile multi-stage

**Prompt** :
```
Crée un Dockerfile multi-stage pour Java 21 :

# Stage 1: Build
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/auth-service-1.0.0-SNAPSHOT.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]

Optimisations :
- Layer caching Maven
- Image runtime légère (alpine)
- Non-root user
```

**Checklist** :
- [ ] Dockerfile créé
- [ ] Build multi-stage
- [ ] Image < 200MB

### 11.2 Créer docker-compose.yml (pour tests locaux)

**Prompt** :
```
Crée un docker-compose.yml avec :
- MongoDB
- Redis
- Kafka (Redpanda)
- Keycloak
- Auth Service

Avec health checks et dépendances.
```

**Checklist** :
- [ ] docker-compose.yml créé
- [ ] Services démarrent correctement
- [ ] Health checks configurés

---

## Phase 12 : GitHub Actions CI/CD ✅

### 12.1 Créer workflow GitHub Actions

Voir fichier : `../github-workflows/auth-service-ci.yml`

**Checklist** :
- [ ] Workflow CI créé
- [ ] Tests automatisés
- [ ] Build Docker image
- [ ] Push vers GHCR

---

## 🎯 Validation Finale

### Checklist de validation

- [ ] Application démarre sans erreur
- [ ] Tous les tests passent (mvn test)
- [ ] Endpoints API répondent correctement
- [ ] JWT Keycloak validé
- [ ] Événements Kafka publiés
- [ ] Consumer Keycloak fonctionne
- [ ] Docker image build OK
- [ ] Health checks OK (GET /actuator/health)
- [ ] Documentation OpenAPI générée (Springdoc)

---

**Prêt pour génération avec Emergent.sh ou Cursor AI !**
