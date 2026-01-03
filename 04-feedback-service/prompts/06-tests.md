# User Story 06 - Tests et Validation

## Contexte
Tu travailles sur le **feedback-service** du projet WeSpeak. Tu dois écrire les tests pour valider le bon fonctionnement du service.

## User Story
**En tant que** développeur
**Je veux** avoir une couverture de tests suffisante
**Afin de** garantir la qualité et la maintenabilité du code

## Tâches

1. **Tests unitaires TranscriptionService** :
   - Test création Transcript en PENDING
   - Test mise à jour en COMPLETED après transcription
   - Test mise à jour en FAILED après erreurs
   - Test parsing des segments Whisper
   - Mock du client S3 et Whisper

2. **Tests unitaires AnalysisService** :
   - Test génération du prompt
   - Test parsing de la réponse LLM
   - Test calcul des XP (différents scénarios)
   - Test mise à jour des UserFeedbackStats
   - Mock du client LLM

3. **Tests unitaires des calculs** :
   - Calcul du score global (moyenne pondérée)
   - Calcul des XP selon les règles
   - Calcul de la tendance (IMPROVING, STABLE, DECLINING)

4. **Tests d'intégration FeedbackController** :
   - GET /feedbacks/me avec pagination
   - GET /feedbacks/{id} - cas nominal
   - GET /feedbacks/{id} - 404 not found
   - GET /feedbacks/{id} - 403 forbidden (mauvais user)
   - GET /stats/me

5. **Tests d'intégration Kafka** :
   - Réception de recording.uploaded
   - Publication de transcript.completed
   - Publication de feedback.generated
   - Publication de xp.awarded

6. **Créer des fixtures de test** :
   - Transcript de test
   - Feedback de test
   - UserFeedbackStats de test
   - Réponse Whisper mockée
   - Réponse LLM mockée

7. **Configuration de test** :
   ```properties
   # application-test.properties
   spring.cloud.stream.enabled=false
   spring.data.mongodb.database=wespeak-feedback-test
   ```

## Critères d'Acceptation

- [ ] Couverture de tests > 70%
- [ ] Tous les tests passent en CI
- [ ] Les services externes sont mockés
- [ ] Les cas d'erreur sont testés
- [ ] Les tests sont rapides (< 30s total)

## Exemples de Tests

```java
@Test
void shouldCalculateXpCorrectly() {
    // Given
    int overallScore = 75;
    int duration = 900; // 15 minutes
    boolean isImproving = true;
    
    // When
    int xp = xpCalculator.calculate(overallScore, duration, isImproving);
    
    // Then
    assertThat(xp).isEqualTo(30); // 10 + 5 + 5 + 5 + 5
}

@Test
void shouldReturnForbiddenWhenAccessingOtherUserFeedback() {
    // Given
    String feedbackId = "feedback-123";
    String otherUserId = "other-user";
    
    // When/Then
    mockMvc.perform(get("/api/v1/feedback/feedbacks/" + feedbackId)
            .with(jwt().claim("sub", otherUserId)))
        .andExpect(status().isForbidden());
}
```

## Notes Techniques

- Utiliser @DataMongoTest pour les tests repository
- Utiliser @WebMvcTest pour les tests controller
- Utiliser Testcontainers si besoin d'un vrai MongoDB
- Mocker les appels HTTP avec WireMock ou MockWebServer
