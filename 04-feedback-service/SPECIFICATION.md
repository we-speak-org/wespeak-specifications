# Feedback Service - Spécifications Fonctionnelles v1.0

## 📋 Table des Matières

1. [Vue d'Ensemble](#1-vue-densemble)
2. [Modèle de Données](#2-modèle-de-données)
3. [Fonctionnalités](#3-fonctionnalités)
4. [Règles Métier](#4-règles-métier)
5. [Intégrations](#5-intégrations)

---

## 1. Vue d'Ensemble

### 1.1 Responsabilité

Le **feedback-service** est responsable de l'analyse et du feedback IA sur les performances linguistiques des utilisateurs. Il traite les enregistrements audio des sessions de conversation pour fournir :

- **Transcription automatique** (Speech-to-Text via Whisper)
- **Analyse linguistique** des erreurs de grammaire, vocabulaire et prononciation
- **Génération de rapports** personnalisés avec conseils d'amélioration
- **Suivi de progression** basé sur les feedbacks accumulés

### 1.2 Dépendances

| Service | Interaction | Description |
|---------|-------------|-------------|
| **conversation-service** | Consomme événements | Reçoit les enregistrements audio à analyser |
| **auth-service** | Appel API | Récupère les profils d'apprentissage des utilisateurs |
| **gamification-service** | Publie événements | Notifie les XP gagnés suite aux feedbacks |
| **Cloudflare R2** | Stockage | Récupère les fichiers audio à transcrire |
| **Whisper API** | Externe | Transcription Speech-to-Text |
| **LLM (Claude/GPT)** | Externe | Analyse linguistique et génération de conseils |

### 1.3 Stack Technique

- **Spring Boot 4** avec Java 21
- **MongoDB** pour le stockage des transcripts et feedbacks
- **Kafka** pour la communication événementielle
- **Cloudflare R2** pour récupérer les fichiers audio
- **Whisper API** pour la transcription
- **Claude/GPT API** pour l'analyse IA

---

## 2. Modèle de Données

### 2.1 Transcript

Le **Transcript** représente la transcription d'un enregistrement audio.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique (UUID) |
| sessionId | String | Oui | ID de la session de conversation |
| participantId | String | Oui | ID de l'utilisateur transcrit |
| recordingId | String | Oui | ID de l'enregistrement audio source |
| targetLanguageCode | String | Oui | Langue de la transcription (ex: "en") |
| content | String | Oui | Texte transcrit complet |
| segments | List | Oui | Segments temporels avec texte |
| duration | Integer | Oui | Durée en secondes |
| wordCount | Integer | Oui | Nombre de mots |
| confidence | Double | Oui | Score de confiance (0.0 à 1.0) |
| status | String | Oui | PENDING, PROCESSING, COMPLETED, FAILED |
| createdAt | DateTime | Oui | Date de création |
| completedAt | DateTime | Non | Date de fin de transcription |

**Segment** (objet imbriqué) :

| Champ | Type | Description |
|-------|------|-------------|
| startTime | Double | Temps de début en secondes |
| endTime | Double | Temps de fin en secondes |
| text | String | Texte du segment |
| confidence | Double | Confiance du segment |

### 2.2 Feedback

Le **Feedback** représente l'analyse IA d'une transcription.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique (UUID) |
| transcriptId | String | Oui | ID du transcript analysé |
| userId | String | Oui | ID de l'utilisateur |
| sessionId | String | Oui | ID de la session source |
| targetLanguageCode | String | Oui | Langue analysée |
| overallScore | Integer | Oui | Score global (0-100) |
| grammarScore | Integer | Oui | Score grammaire (0-100) |
| vocabularyScore | Integer | Oui | Score vocabulaire (0-100) |
| fluencyScore | Integer | Oui | Score fluidité (0-100) |
| pronunciationScore | Integer | Non | Score prononciation (0-100) |
| errors | List | Oui | Liste des erreurs détectées |
| strengths | List | Oui | Points forts identifiés |
| improvements | List | Oui | Conseils d'amélioration |
| summary | String | Oui | Résumé textuel du feedback |
| xpAwarded | Integer | Oui | XP attribués pour cette session |
| status | String | Oui | PENDING, PROCESSING, COMPLETED, FAILED |
| createdAt | DateTime | Oui | Date de création |
| completedAt | DateTime | Non | Date de fin d'analyse |

**Error** (objet imbriqué) :

| Champ | Type | Description |
|-------|------|-------------|
| type | String | GRAMMAR, VOCABULARY, PRONUNCIATION, SYNTAX |
| original | String | Ce qui a été dit |
| correction | String | Correction suggérée |
| explanation | String | Explication de l'erreur |
| severity | String | LOW, MEDIUM, HIGH |
| segmentIndex | Integer | Index du segment concerné |

### 2.3 UserFeedbackStats

Statistiques agrégées des feedbacks d'un utilisateur par langue.

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| id | String | Oui | Identifiant unique |
| userId | String | Oui | ID de l'utilisateur |
| targetLanguageCode | String | Oui | Langue cible |
| totalSessions | Integer | Oui | Nombre total de sessions analysées |
| totalMinutes | Integer | Oui | Minutes totales de conversation |
| averageOverallScore | Double | Oui | Score moyen global |
| averageGrammarScore | Double | Oui | Score moyen grammaire |
| averageVocabularyScore | Double | Oui | Score moyen vocabulaire |
| averageFluencyScore | Double | Oui | Score moyen fluidité |
| commonErrors | List | Oui | Erreurs les plus fréquentes |
| progressTrend | String | Oui | IMPROVING, STABLE, DECLINING |
| lastFeedbackAt | DateTime | Non | Date du dernier feedback |
| updatedAt | DateTime | Oui | Dernière mise à jour |

---

## 3. Fonctionnalités

### 3.1 Pipeline de Traitement

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Enregistrement │────▶│  Transcription  │────▶│    Analyse IA   │
│  Audio (R2)     │     │   (Whisper)     │     │  (Claude/GPT)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                        │
                                                        ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Notification   │◀────│  Calcul XP      │◀────│   Génération    │
│  Utilisateur    │     │  Gamification   │     │    Feedback     │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### 3.2 Transcription Automatique

**Déclencheur** : Événement `recording.uploaded` reçu de conversation-service

**Processus** :
1. Récupérer les fichiers audio (un par participant) depuis Cloudflare R2
2. Envoyer chaque fichier à l'API Whisper pour transcription
3. Fusionner les segments temporels de tous les participants
4. Sauvegarder le Transcript unifié en base
5. Déclencher l'analyse IA

**Langues supportées** : Toutes les langues supportées par Whisper

### 3.3 Analyse Linguistique IA

**Entrée** : Transcript complété

**Analyse réalisée** :
- **Grammaire** : Temps verbaux, accords, structure de phrase
- **Vocabulaire** : Richesse lexicale, expressions idiomatiques, registre
- **Fluidité** : Hésitations, répétitions, débit
- **Prononciation** : Basée sur les mots mal reconnus par Whisper

**Sortie** : Feedback détaillé avec scores et recommandations

### 3.4 Génération de Rapports

**Types de rapports** :
- **Rapport de session** : Feedback détaillé après chaque conversation
- **Rapport hebdomadaire** : Synthèse des progrès de la semaine
- **Rapport de progression** : Évolution sur le temps

### 3.5 Consultation des Feedbacks

L'utilisateur peut consulter :
- L'historique de tous ses feedbacks
- Le détail d'un feedback spécifique
- Ses statistiques globales par langue
- Sa courbe de progression

---

## 4. Règles Métier

### 4.1 Calcul des Scores

**Score global** : Moyenne pondérée
```
overallScore = (grammarScore × 0.35) + (vocabularyScore × 0.25) 
             + (fluencyScore × 0.25) + (pronunciationScore × 0.15)
```

**Échelle des scores** :
- 0-39 : Débutant (beaucoup d'erreurs)
- 40-59 : Intermédiaire (erreurs fréquentes)
- 60-79 : Avancé (quelques erreurs)
- 80-100 : Expert (très peu d'erreurs)

### 4.2 Attribution des XP

| Critère | XP |
|---------|-----|
| Participation (session complétée) | 10 XP |
| Score global ≥ 60 | +5 XP |
| Score global ≥ 80 | +10 XP |
| Amélioration vs session précédente | +5 XP |
| Session ≥ 10 minutes | +5 XP |
| Session ≥ 20 minutes | +10 XP |

**Maximum par session** : 40 XP

### 4.3 Détection des Erreurs

**Sévérité des erreurs** :
- **LOW** : Erreur mineure n'affectant pas la compréhension
- **MEDIUM** : Erreur notable mais compréhensible
- **HIGH** : Erreur grave affectant la compréhension

**Limite d'erreurs affichées** : Maximum 10 erreurs par feedback (les plus importantes)

### 4.4 Calcul de la Tendance de Progression

Basé sur les 5 dernières sessions :
- **IMPROVING** : Score moyen en hausse de +5 points ou plus
- **STABLE** : Variation de moins de 5 points
- **DECLINING** : Score moyen en baisse de -5 points ou plus

### 4.5 Confidentialité

- Les transcripts sont accessibles uniquement par l'utilisateur concerné
- Les enregistrements audio peuvent être supprimés à la demande
- Les feedbacks sont anonymisés pour les statistiques globales

---

## 5. Intégrations

### 5.1 Whisper API (OpenAI)

**Utilisation** : Transcription Speech-to-Text

**Configuration** :
- Modèle : `whisper-1`
- Format de sortie : Segments avec timestamps
- Langue : Auto-détection ou spécifiée

**Gestion des erreurs** :
- Retry automatique (3 tentatives)
- Fallback : Marquer le transcript comme FAILED

### 5.2 Claude/GPT API

**Utilisation** : Analyse linguistique et génération de conseils

**Prompt type** :
```
Analyse cette transcription d'un apprenant de niveau {level} en {language}.
Identifie les erreurs de grammaire, vocabulaire et fluidité.
Fournis des conseils personnalisés pour s'améliorer.
Format de sortie : JSON structuré avec scores et erreurs.
```

**Température** : 0.3 (réponses plus déterministes)

### 5.3 Événements Kafka

**Événements consommés** :
- `recording.uploaded` → Déclenche la transcription

**Événements publiés** :
- `transcript.completed` → Notifie la fin de transcription
- `feedback.generated` → Notifie la génération du feedback
- `xp.awarded` → Notifie les XP à attribuer (vers gamification-service)

---

## 6. Limites et Contraintes

### 6.1 Limites Techniques

| Ressource | Limite |
|-----------|--------|
| Durée audio maximale | 60 minutes |
| Taille fichier audio | 100 MB |
| Feedbacks par jour (utilisateur free) | 3 |
| Feedbacks par jour (utilisateur premium) | Illimité |
| Rétention des transcripts | 1 an |
| Rétention des enregistrements audio | 30 jours |

### 6.2 Temps de Traitement

| Opération | Temps estimé |
|-----------|--------------|
| Transcription (5 min audio) | 30-60 secondes |
| Analyse IA | 10-20 secondes |
| Total pipeline | 1-2 minutes |

---

## 7. Évolutions Futures (Hors MVP)

- Feedback en temps réel pendant la conversation
- Analyse de la prononciation avec spectrogramme
- Comparaison avec des locuteurs natifs
- Exercices de correction personnalisés
- Export PDF des rapports
