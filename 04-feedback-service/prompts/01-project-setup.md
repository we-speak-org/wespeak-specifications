# User Story 01 - Configuration du Projet

## Contexte
Tu travailles sur le **feedback-service** du projet WeSpeak, une plateforme d'apprentissage des langues. Ce service est responsable de la transcription et de l'analyse IA des conversations.

## User Story
**En tant que** développeur
**Je veux** configurer le projet feedback-service avec Spring Boot 4
**Afin de** disposer d'une base solide pour développer les fonctionnalités

## Tâches

1. **Initialiser le projet Spring Boot 4** avec les dépendances :
   - Spring Web
   - Spring Data MongoDB
   - Spring Cloud Stream avec Kafka Binder
   - Spring Validation
   - Lombok

2. **Configurer MongoDB** :
   - Connexion à la base `wespeak-feedback`
   - Activer les auditing annotations (@CreatedDate, @LastModifiedDate)

3. **Configurer Kafka** :
   - Consumer group : `feedback-service`
   - Topic à consommer : `recording.events`
   - Topics à produire : `transcript.events`, `feedback.events`, `gamification.events`

4. **Créer la structure de packages** :
   ```
   com.wespeak.feedback
   ├── config/
   ├── controller/
   ├── service/
   ├── repository/
   ├── model/
   ├── dto/
   ├── event/
   └── client/
   ```

5. **Configurer les profils** :
   - `application.properties` (commun)
   - `application-dev.properties`
   - `application-test.properties` (désactive Kafka)

## Critères d'Acceptation

- [ ] L'application démarre sans erreur
- [ ] La connexion MongoDB est établie
- [ ] Le consumer Kafka est prêt à recevoir des messages
- [ ] Les tests unitaires passent avec le profil `test`
- [ ] La structure de packages est créée

## Notes Techniques

- Utiliser Java 21
- Pas d'interfaces sauf si plus de 2 implémentations nécessaires
- Structure de code simple et aplatie
