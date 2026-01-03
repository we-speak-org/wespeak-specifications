# User Story : Tests

## Contexte

Implémenter une suite de tests complète pour le lesson-service.

## Structure des Tests

```
src/test/java/org/wespeak/lesson/
├── controller/
│   ├── CourseControllerTest.java
│   ├── LessonControllerTest.java
│   └── ProgressControllerTest.java
├── service/
│   ├── CourseServiceTest.java
│   ├── LessonServiceTest.java
│   ├── ProgressServiceTest.java
│   └── ExerciseValidatorTest.java
├── repository/
│   └── (tests d'intégration)
├── listener/
│   └── UserEventListenerTest.java
└── integration/
    ├── LessonFlowIntegrationTest.java
    └── KafkaIntegrationTest.java
```

## Tests Unitaires

### ExerciseValidatorTest

```java
@ExtendWith(MockitoExtension.class)
class McqAnswerValidatorTest {

    private McqAnswerValidator validator;

    @BeforeEach
    void setUp() {
        validator = new McqAnswerValidator();
    }

    @Test
    void shouldReturnTrueForCorrectAnswer() {
        // Given
        McqAnswer userAnswer = new McqAnswer("a");
        McqCorrectAnswer correctAnswer = new McqCorrectAnswer("a");

        // When
        ValidationResult result = validator.validate(userAnswer, correctAnswer);

        // Then
        assertThat(result.isCorrect()).isTrue();
        assertThat(result.getPointsEarned()).isEqualTo(10);
    }

    @Test
    void shouldReturnFalseForIncorrectAnswer() {
        // Given
        McqAnswer userAnswer = new McqAnswer("b");
        McqCorrectAnswer correctAnswer = new McqCorrectAnswer("a");

        // When
        ValidationResult result = validator.validate(userAnswer, correctAnswer);

        // Then
        assertThat(result.isCorrect()).isFalse();
        assertThat(result.getPointsEarned()).isZero();
    }
}
```

### LessonServiceTest

```java
@ExtendWith(MockitoExtension.class)
class LessonServiceTest {

    @Mock
    private LessonRepository lessonRepository;
    @Mock
    private UserProgressRepository progressRepository;
    @Mock
    private LessonEventPublisher eventPublisher;

    @InjectMocks
    private LessonServiceImpl lessonService;

    @Test
    void shouldStartLessonWhenUnlocked() {
        // Given
        String lessonId = "lesson-1";
        String userId = "user-1";
        Lesson lesson = createTestLesson(lessonId);
        
        when(lessonRepository.findById(lessonId)).thenReturn(Optional.of(lesson));
        when(progressRepository.isLessonUnlocked(userId, lessonId)).thenReturn(true);

        // When
        LessonSessionDto result = lessonService.startLesson(lessonId, userId);

        // Then
        assertThat(result.getLessonId()).isEqualTo(lessonId);
        verify(eventPublisher).publishLessonStarted(any());
    }

    @Test
    void shouldThrowExceptionWhenLessonLocked() {
        // Given
        String lessonId = "lesson-1";
        String userId = "user-1";
        
        when(progressRepository.isLessonUnlocked(userId, lessonId)).thenReturn(false);

        // When/Then
        assertThatThrownBy(() -> lessonService.startLesson(lessonId, userId))
            .isInstanceOf(LessonLockedException.class)
            .hasMessageContaining("LESSON_LOCKED");
    }

    @Test
    void shouldCalculateXPWithBonus() {
        // Given
        int baseXP = 15;
        int score = 95; // >= 90% -> bonus 20%

        // When
        int xpEarned = lessonService.calculateXP(baseXP, score, 1);

        // Then
        assertThat(xpEarned).isEqualTo(18); // 15 * 1.2 = 18
    }
}
```

## Tests d'Intégration

### Configuration de Test

```java
@SpringBootTest
@AutoConfigureMockMvc
@Testcontainers
@ActiveProfiles("test")
public abstract class IntegrationTestBase {

    @Container
    static MongoDBContainer mongoContainer = new MongoDBContainer("mongo:7.0");

    @Container
    static GenericContainer<?> redisContainer = new GenericContainer<>("redis:7")
        .withExposedPorts(6379);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.mongodb.uri", mongoContainer::getReplicaSetUrl);
        registry.add("spring.data.redis.host", redisContainer::getHost);
        registry.add("spring.data.redis.port", () -> redisContainer.getMappedPort(6379));
    }
}
```

### LessonFlowIntegrationTest

```java
class LessonFlowIntegrationTest extends IntegrationTestBase {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private CourseRepository courseRepository;

    @Autowired
    private UserProgressRepository progressRepository;

    @BeforeEach
    void setUp() {
        // Insérer données de test
        courseRepository.save(createTestCourse());
    }

    @Test
    void shouldCompleteFullLessonFlow() throws Exception {
        // 1. Lister les cours
        mockMvc.perform(get("/api/v1/courses")
                .param("language", "en"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.courses").isArray())
            .andExpect(jsonPath("$.courses[0].level").value("A1"));

        // 2. Obtenir une leçon
        mockMvc.perform(get("/api/v1/lessons/{id}", "lesson-1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.exercises").isArray());

        // 3. Démarrer la leçon (avec JWT)
        mockMvc.perform(post("/api/v1/lessons/{id}/start", "lesson-1")
                .header("Authorization", "Bearer " + getTestJwt()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.sessionId").exists());

        // 4. Soumettre une réponse
        mockMvc.perform(post("/api/v1/exercises/{id}/submit", "ex-1")
                .header("Authorization", "Bearer " + getTestJwt())
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                      "answer": {"optionId": "a"},
                      "timeSpentSeconds": 10
                    }
                    """))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.isCorrect").value(true));

        // 5. Terminer la leçon
        mockMvc.perform(post("/api/v1/lessons/{id}/complete", "lesson-1")
                .header("Authorization", "Bearer " + getTestJwt())
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                      "score": 85,
                      "correctAnswers": 17,
                      "totalExercises": 20,
                      "timeSpentSeconds": 420
                    }
                    """))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.completion.xpEarned").value(15))
            .andExpect(jsonPath("$.unlocked.nextLesson").exists());
    }
}
```

### KafkaIntegrationTest

```java
@EmbeddedKafka(partitions = 1, topics = {"user.events", "lesson.events"})
class KafkaIntegrationTest extends IntegrationTestBase {

    @Autowired
    private EmbeddedKafkaBroker embeddedKafka;

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    private UserProgressRepository progressRepository;

    @Test
    void shouldCreateProgressOnUserRegistered() throws Exception {
        // Given
        String event = """
            {
              "eventType": "user.registered",
              "payload": {
                "userId": "test-user-123",
                "learningProfiles": [
                  {"targetLanguageCode": "en"}
                ]
              }
            }
            """;

        // When
        kafkaTemplate.send("user.events", "test-user-123", event);

        // Then
        await().atMost(5, TimeUnit.SECONDS).until(() ->
            progressRepository.existsByUserIdAndTargetLanguageCode("test-user-123", "en")
        );
    }
}
```

## Profil de Test

### application-test.properties

```properties
# Désactiver Kafka pour tests unitaires
spring.cloud.stream.enabled=false
spring.cloud.function.definition=

# Logging réduit
logging.level.org.springframework=WARN
```

## Critères d'Acceptation

- [ ] Couverture de code > 80%
- [ ] Tests unitaires pour tous les services
- [ ] Tests d'intégration pour les flows principaux
- [ ] Tests Kafka avec EmbeddedKafka
- [ ] Tests MongoDB avec Testcontainers
- [ ] CI passe tous les tests
