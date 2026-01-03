# User Story : Cache Redis

## Contexte

Implémenter le cache Redis pour optimiser les performances sur le contenu statique.

## Stratégie de Cache

### Données à mettre en cache

| Donnée | TTL | Clé | Raison |
|--------|-----|-----|--------|
| Liste des cours | 1 heure | `courses:{lang}` | Contenu stable |
| Détail d'un cours | 1 heure | `course:{id}` | Contenu stable |
| Unités d'un cours | 1 heure | `units:{courseId}` | Contenu stable |
| Détail d'une unité | 1 heure | `unit:{id}` | Contenu stable |
| Leçon avec exercices | 30 min | `lesson:{id}` | Contenu stable |

### Données NON cachées

| Donnée | Raison |
|--------|--------|
| UserProgress | Données dynamiques |
| LessonCompletion | Données dynamiques |
| État de déblocage | Dépend de la progression |

## Implémentation

### Configuration

```properties
# Redis
spring.data.redis.host=${REDIS_HOST:localhost}
spring.data.redis.port=${REDIS_PORT:6379}

# Cache
spring.cache.type=redis
spring.cache.redis.time-to-live=3600000
spring.cache.redis.cache-null-values=false
```

### Activer le cache

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));

        Map<String, RedisCacheConfiguration> cacheConfigs = Map.of(
            "courses", defaultConfig.entryTtl(Duration.ofHours(1)),
            "course", defaultConfig.entryTtl(Duration.ofHours(1)),
            "units", defaultConfig.entryTtl(Duration.ofHours(1)),
            "unit", defaultConfig.entryTtl(Duration.ofHours(1)),
            "lesson", defaultConfig.entryTtl(Duration.ofMinutes(30))
        );

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(defaultConfig)
            .withInitialCacheConfigurations(cacheConfigs)
            .build();
    }
}
```

### Utilisation dans les Services

```java
@Service
public class CourseServiceImpl implements CourseService {

    @Cacheable(value = "courses", key = "#languageCode + ':' + #level")
    public List<CourseDto> findCourses(String languageCode, String level) {
        // Requête MongoDB
    }

    @Cacheable(value = "course", key = "#courseId")
    public CourseDetailDto findCourseById(String courseId) {
        // Requête MongoDB
    }

    // Invalidation si le contenu est modifié (admin)
    @CacheEvict(value = "courses", allEntries = true)
    public void updateCourse(String courseId, CourseUpdateRequest request) {
        // ...
    }
}
```

```java
@Service
public class UnitServiceImpl implements UnitService {

    @Cacheable(value = "units", key = "#courseId")
    public List<UnitDto> findUnitsByCourse(String courseId) {
        // Requête MongoDB
    }

    @Cacheable(value = "unit", key = "#unitId", unless = "#result == null")
    public UnitDetailDto findUnitById(String unitId) {
        // Requête MongoDB
    }
}
```

```java
@Service
public class LessonServiceImpl implements LessonService {

    @Cacheable(value = "lesson", key = "#lessonId")
    public LessonEntity findLessonEntity(String lessonId) {
        // Retourne l'entité brute (sans état utilisateur)
        return lessonRepository.findById(lessonId).orElseThrow();
    }
    
    // La méthode publique combine cache + données dynamiques
    public LessonDetailDto findLessonById(String lessonId, String userId) {
        Lesson lesson = findLessonEntity(lessonId); // Caché
        UserLessonState state = progressService.getLessonState(userId, lessonId); // Non caché
        return LessonDetailDto.from(lesson, state);
    }
}
```

## Pattern Cache-Aside

Pour les données combinées (statiques + dynamiques) :

```java
public LessonDetailDto findLessonById(String lessonId, String userId) {
    // 1. Récupérer données statiques (cachées)
    Lesson lesson = getCachedLesson(lessonId);
    
    // 2. Récupérer état utilisateur (jamais caché)
    LessonState state = getLessonState(userId, lessonId);
    
    // 3. Combiner
    return LessonDetailDto.builder()
        .id(lesson.getId())
        .title(lesson.getTitle())
        // ... autres champs statiques
        .isUnlocked(state.isUnlocked())
        .isCompleted(state.isCompleted())
        .bestScore(state.getBestScore())
        .build();
}
```

## Invalidation du Cache

### Quand invalider ?

| Événement | Caches à invalider |
|-----------|-------------------|
| Cours modifié (admin) | `courses:*`, `course:{id}` |
| Unité modifiée (admin) | `units:{courseId}`, `unit:{id}` |
| Leçon modifiée (admin) | `lesson:{id}` |

### Endpoint Admin (optionnel)

```java
@RestController
@RequestMapping("/admin/cache")
@PreAuthorize("hasRole('ADMIN')")
public class CacheController {

    private final CacheManager cacheManager;

    @DeleteMapping("/{cacheName}")
    public void clearCache(@PathVariable String cacheName) {
        Cache cache = cacheManager.getCache(cacheName);
        if (cache != null) {
            cache.clear();
        }
    }
}
```

## Critères d'Acceptation

- [ ] Redis est configuré et connecté
- [ ] Les cours sont cachés pendant 1 heure
- [ ] Les leçons sont cachées pendant 30 minutes
- [ ] La progression utilisateur n'est JAMAIS cachée
- [ ] Le cache est invalidé si le contenu change
- [ ] Tests vérifient que le cache fonctionne (hit/miss)
