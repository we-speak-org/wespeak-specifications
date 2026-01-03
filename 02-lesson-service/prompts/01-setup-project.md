# User Story : Configuration Initiale du Projet

## Contexte

Tu travailles sur le **lesson-service** de WeSpeak, une plateforme d'apprentissage des langues. Ce service gère le curriculum pédagogique (cours, unités, leçons, exercices) et la progression des utilisateurs.

## Objectif

Configurer le projet Spring Boot avec les dépendances nécessaires et la structure de base.

## Spécifications Techniques

- **Framework** : Spring Boot 4
- **Base de données** : MongoDB
- **Messaging** : Spring Cloud Stream avec Kafka
- **Cache** : Redis
- **Java** : 21

## Tâches

1. Créer la structure de packages :
   - `org.wespeak.lesson.entity` - Entités MongoDB
   - `org.wespeak.lesson.repository` - Repositories
   - `org.wespeak.lesson.service` - Services métier
   - `org.wespeak.lesson.controller` - Controllers REST
   - `org.wespeak.lesson.listener` - Kafka listeners
   - `org.wespeak.lesson.dto` - DTOs
   - `org.wespeak.lesson.exception` - Exceptions custom

2. Configurer `application.properties` :
   - MongoDB connection
   - Redis connection
   - Kafka/Spring Cloud Stream bindings
   - Port : 8082

3. Créer les entités de base (voir 02-modele-donnees.md) :
   - Course
   - Unit
   - Lesson (avec Exercise embedded)
   - UserProgress
   - LessonCompletion

## Critères d'Acceptation

- [ ] Le projet compile sans erreur
- [ ] Les tests contexte Spring passent
- [ ] Les collections MongoDB sont créées au démarrage
- [ ] Le health check `/actuator/health` répond 200
