# WeSpeak Specifications - Ready for Emergent.sh

**Spécifications techniques complètes** pour générer le code avec l'IA (Emergent.sh, Cursor, etc.)

---

## 🎯 WeSpeak : Plateforme d'apprentissage des langues

Combine :
1. 📚 Structure pédagogique progressive
2. 🗣️ Conversations réelles 1v1 (WebRTC)
3. 🤖 Feedback IA (STT + NLP)
4. 🎮 Gamification (XP, badges, streaks)

---

## 📂 Structure du Repository

Chaque microservice a son dossier avec :
- ✅ **README.md** : Vue d'ensemble
- ✅ **data-models/** : Schémas MongoDB (JSON Schema)
- ✅ **api/** : Endpoints REST + curl examples
- ✅ **events/** : Kafka events (published/consumed)
- ✅ **diagrams/** : Mermaid diagrams
- ✅ **emergent/** : 🔥 **CHECKLIST + PROMPTS pour Emergent.sh**

---

## 🚀 Microservices

### 1. Auth Service ([Voir specs](./01-auth-service/README.md))
- Profils utilisateurs + profils d'apprentissage multi-langues
- Crédits/quotas (free: 3 conversations/semaine)
- Sync Keycloak (Kafka)

### 2-7. Autres services
- Lesson, Conversation, Feedback, Gamification, Recommendation, API Gateway

---

## 🛠️ Utiliser avec Emergent.sh

### Fichiers clés par service :

```
01-auth-service/
├── emergent/
│   ├── CHECKLIST.md          ✅ Liste de tâches complète
│   ├── TECHNICAL_CONTEXT.md  🛠️ Contexte pour l'IA
│   └── PROMPTS.md            💬 Prompts prêts à copier
```

### Workflow :

1. **Lire les specs** : `cd 01-auth-service && cat README.md`
2. **Copier la checklist** : `cat emergent/CHECKLIST.md`
3. **Utiliser les prompts** : `cat emergent/PROMPTS.md`
4. **Générer le code** avec Emergent.sh/Cursor
5. **Tester** : `mvn spring-boot:run`
6. **Pusher** : GitHub Actions build automatique !

---

## 🏗️ Stack Technique

**Backend (tous services)** :
- ☕ Java 21 + Spring Boot 4.0
- 🔄 Spring WebFlux (Reactive)
- 🍃 MongoDB 7.0
- 🔴 Redis 7.0
- 📨 Kafka 3.6
- 🐳 Docker Compose (pas de Kubernetes pour MVP)

**Frontend** :
- 🅰️ Angular 17+ avec SSR
- 📞 WebRTC

**Infrastructure** :
- 🔐 Keycloak 23+ (auth)
- 🐙 GitHub Actions (CI/CD)
- 📦 GHCR (images Docker)

---

## 📋 Plan d'Implémentation

### Phase 1 (MVP Core - 4 semaines)
1. Auth Service (semaines 1-2)
2. Lesson Service (semaines 2-3)
3. Conversation Service (semaines 3-4)
4. API Gateway (semaine 4)

### Phase 2 (Feedback IA - 2 semaines)
### Phase 3 (Gamification - 2 semaines)

Voir [IMPLEMENTATION_ROADMAP.md](./IMPLEMENTATION_ROADMAP.md)

---

## 🐳 Docker Compose

```bash
cd docker
docker-compose up -d
```

Services : Keycloak, MongoDB, Redis, Kafka, Auth Service, etc.

---

## 🧪 Tests

```bash
mvn test          # Tests unitaires
mvn verify        # Tests d'intégration (Testcontainers)
mvn jacoco:report # Coverage
```

---

## 🚀 CI/CD GitHub Actions

Voir `github-workflows/auth-service-ci.yml`

Auto-build sur push → Tests → Docker image → Push GHCR

---

## 📞 Contact

**Organisation** : [github.com/we-speak-org](https://github.com/we-speak-org)

---

**🔥 Prêt à générer avec Emergent.sh !**
