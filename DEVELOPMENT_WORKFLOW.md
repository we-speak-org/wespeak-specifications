# 🚀 Workflow de Développement WeSpeak

Ce document décrit le processus complet pour développer un nouveau microservice WeSpeak, de la spécification à la production.

---

## 📋 Table des Matières

1. [Prérequis](#prérequis)
2. [Workflow Complet](#workflow-complet)
3. [Étape 1 : Consultation des Spécifications](#étape-1--consultation-des-spécifications)
4. [Étape 2 : Création du Projet depuis le Template](#étape-2--création-du-projet-depuis-le-template)
5. [Étape 3 : Développement avec Emergent.sh](#étape-3--développement-avec-emergentsh)
6. [Étape 4 : Build et Publication](#étape-4--build-et-publication)
7. [Étape 5 : Déploiement](#étape-5--déploiement)
8. [Bonnes Pratiques](#bonnes-pratiques)

---

## Prérequis

### Outils Requis
- **Git** : Gestion de version
- **GitHub CLI (`gh`)** : Interaction avec GitHub
- **Docker** : Pour les builds locaux (optionnel)
- **Emergent.sh** : Plateforme de génération de code IA

### Accès Nécessaires
- Accès à l'organisation GitHub `we-speak-org`
- Compte Emergent.sh avec accès au projet WeSpeak
- Variables d'environnement configurées (voir `.env.example` dans chaque service)

---

## Workflow Complet

```mermaid
graph TD
    A[📖 Consulter Spécifications] --> B[🆕 Créer Projet depuis Template]
    B --> C[⚙️ Configuration Initiale]
    C --> D[🤖 Développement avec Emergent]
    D --> E[✅ Tests & Validation]
    E --> F{Tests OK?}
    F -->|Non| D
    F -->|Oui| G[📤 Push vers GitHub]
    G --> H[🏗️ GitHub Actions Build]
    H --> I[📦 Publication Image Docker]
    I --> J[🚀 Déploiement Docker Compose]
    
    style A fill:#e1f5ff
    style D fill:#fff4e1
    style J fill:#e8f5e9
```

---

## Étape 1 : Consultation des Spécifications

### 1.1 Localiser les Spécifications du Service

Toutes les spécifications sont dans ce repository, organisées par service :

```
wespeak-specifications/
├── services/
│   ├── 01-auth-service/
│   │   ├── README.md                    # Spécifications fonctionnelles
│   │   ├── api-endpoints.md             # Documentation API
│   │   ├── data-models.md               # Schémas de données
│   │   ├── kafka-events.md              # Événements asynchrones
│   │   ├── diagrams/                    # Diagrammes Mermaid
│   │   ├── user-stories.md              # User stories
│   │   └── emergent-checklist.md        # Checklist pour Emergent
│   ├── 02-lesson-service/
│   ├── 03-conversation-service/
│   └── ...
```

### 1.2 Documents à Lire

Pour chaque service, consultez dans l'ordre :

1. **README.md** : Vue d'ensemble et responsabilités
2. **data-models.md** : Structure des données MongoDB
3. **api-endpoints.md** : Tous les endpoints REST
4. **kafka-events.md** : Messages Kafka publiés/consommés
5. **diagrams/** : Diagrammes d'architecture et séquence
6. **emergent-checklist.md** : ⚠️ **IMPORTANT** pour Emergent

### 1.3 Exemple : Démarrer avec Auth Service

```bash
# Cloner le repository de spécifications
git clone https://github.com/we-speak-org/wespeak-specifications.git
cd wespeak-specifications/services/01-auth-service

# Lire les specs principales
cat README.md
cat emergent-checklist.md
```

---

## Étape 2 : Création du Projet depuis le Template

### 2.1 Utiliser le Template GitHub

Le template Spring Boot contient toute la structure de base :

**Template Repository** : [`springboot-service-template`](https://github.com/we-speak-org/springboot-service-template)

**Méthode 1 : Via GitHub UI**

1. Aller sur https://github.com/we-speak-org/springboot-service-template
2. Cliquer sur **"Use this template"** → **"Create a new repository"**
3. Nommer le repository : `{service-name}` (ex: `auth-service`)
4. Sélectionner `we-speak-org` comme propriétaire
5. Choisir **Public** ou **Private**
6. Cliquer sur **"Create repository"**

**Méthode 2 : Via GitHub CLI**

```bash
# Créer un nouveau service depuis le template
gh repo create we-speak-org/auth-service \
  --template we-speak-org/springboot-service-template \
  --public \
  --clone

# Se déplacer dans le nouveau projet
cd auth-service
```

### 2.2 Configuration Initiale

**Fichiers à Personnaliser :**

1. **`pom.xml`** :
   ```xml
   <artifactId>auth-service</artifactId>
   <name>WeSpeak Auth Service</name>
   <description>Authentication and user profile management service</description>
   ```

2. **`application.yml`** :
   ```yaml
   spring:
     application:
       name: auth-service
   
   server:
     port: 8081  # Port unique par service
   ```

3. **`.env.example`** :
   ```env
   SERVICE_NAME=auth-service
   MONGODB_DATABASE=wespeak_auth
   KAFKA_TOPICS=user.events,auth.events
   ```

4. **`README.md`** : Adapter la documentation

5. **Package Java** : Renommer `org.wespeak.template` → `org.wespeak.authservice`

### 2.3 Structure du Template

```
auth-service/
├── .github/
│   └── workflows/
│       └── build-and-publish.yml    # ✅ Déjà configuré pour GitHub Actions
├── docker/
│   ├── Dockerfile                   # ✅ Multi-stage build optimisé
│   └── docker-compose.yml           # ✅ Setup local avec MongoDB/Kafka
├── src/
│   ├── main/
│   │   ├── java/org/wespeak/authservice/
│   │   │   ├── config/              # Configurations (Security, Kafka, OpenAPI)
│   │   │   ├── controller/          # REST Controllers
│   │   │   ├── service/             # Business Logic
│   │   │   ├── repository/          # MongoDB Repositories
│   │   │   ├── model/               # Entities & DTOs
│   │   │   ├── kafka/               # Producers & Consumers
│   │   │   └── exception/           # Exception Handling
│   │   └── resources/
│   │       ├── application.yml      # Configuration principale
│   │       ├── application-dev.yml
│   │       └── application-prod.yml
│   └── test/                        # Tests unitaires & intégration
├── docs/
│   └── EMERGENT_GUIDE.md           # ⭐ Guide pour Emergent
├── pom.xml                          # Maven dependencies
└── README.md
```

---

## Étape 3 : Développement avec Emergent.sh

### 3.1 Préparer le Contexte pour Emergent

**Documents à Fournir à Emergent :**

1. **Spécifications du service** (`services/{service-name}/README.md`)
2. **Checklist Emergent** (`services/{service-name}/emergent-checklist.md`)
3. **Data models** (`services/{service-name}/data-models.md`)
4. **API endpoints** (`services/{service-name}/api-endpoints.md`)
5. **Kafka events** (`services/{service-name}/kafka-events.md`)

### 3.2 Workflow Emergent Recommandé

```mermaid
sequenceDiagram
    participant Dev as Vous
    participant Emergent as Emergent.sh
    participant Specs as Spécifications
    participant Code as Code Repository
    
    Dev->>Specs: 1. Lire emergent-checklist.md
    Dev->>Emergent: 2. Créer nouveau projet depuis template
    Dev->>Emergent: 3. Upload docs (README, data-models, api-endpoints)
    Emergent->>Emergent: 4. Analyser les spécifications
    Dev->>Emergent: 5. "Implémente les entités MongoDB"
    Emergent->>Code: 6. Génère les entities
    Dev->>Emergent: 7. "Implémente les repositories"
    Emergent->>Code: 8. Génère les repositories
    Dev->>Emergent: 9. "Implémente les services métier"
    Emergent->>Code: 10. Génère les services
    Dev->>Emergent: 11. "Implémente les controllers REST"
    Emergent->>Code: 12. Génère les controllers
    Dev->>Emergent: 13. "Implémente Kafka producers/consumers"
    Emergent->>Code: 14. Génère Kafka integration
    Dev->>Code: 15. Tests & Validation
```

### 3.3 Prompts Emergent Recommandés

**Phase 1 : Modèle de Données**
```
Contexte : Je développe le service [nom] pour WeSpeak.
Consulte le fichier data-models.md et implémente toutes les entités MongoDB
avec annotations Spring Data MongoDB, validation Bean Validation,
et indexes appropriés.
```

**Phase 2 : Repositories**
```
Crée les repositories Spring Data MongoDB pour toutes les entités.
Ajoute les méthodes de requête custom nécessaires selon api-endpoints.md.
Utilise les indexes définis dans data-models.md pour optimiser les requêtes.
```

**Phase 3 : Services Métier**
```
Implémente les services métier avec toute la logique définie dans README.md
section "Règles Métier". Assure-toi de gérer les transactions,
la validation, et les exceptions business.
```

**Phase 4 : Controllers REST**
```
Implémente tous les endpoints REST décrits dans api-endpoints.md.
Utilise les annotations OpenAPI/Swagger.
Respecte les codes HTTP et formats de réponse spécifiés.
```

**Phase 5 : Kafka Integration**
```
Implémente les Kafka producers et consumers selon kafka-events.md.
Assure l'idempotence des consumers et la sérialisation JSON des events.
Utilise les topics et partitionnement spécifiés.
```

### 3.4 Checklist de Validation

Avant de passer à l'étape suivante, vérifier :

- [ ] Toutes les entités MongoDB sont créées avec validation
- [ ] Tous les repositories ont les méthodes requises
- [ ] La logique métier respecte les règles définies
- [ ] Tous les endpoints API sont implémentés
- [ ] Les Kafka producers/consumers fonctionnent
- [ ] Les tests unitaires passent
- [ ] La documentation OpenAPI est générée
- [ ] Le code compile sans erreurs
- [ ] Les variables d'environnement sont dans `.env.example`

---

## Étape 4 : Build et Publication

### 4.1 Build Local (Optionnel)

```bash
# Build Maven
./mvnw clean package

# Build Docker image localement
docker build -f docker/Dockerfile -t auth-service:local .

# Tester avec docker-compose
cd docker
cp ../.env.example .env
# Éditer .env avec vos valeurs
docker-compose up -d
```

### 4.2 Push vers GitHub

```bash
# Commit et push
git add .
git commit -m "feat: implement auth-service core features"
git push origin main
```

### 4.3 GitHub Actions (Automatique)

Le workflow `.github/workflows/build-and-publish.yml` se déclenche automatiquement :

1. **Compile** le projet Maven
2. **Teste** le code (tests unitaires + intégration)
3. **Build** l'image Docker (multi-arch: linux/amd64, linux/arm64)
4. **Publie** sur GitHub Container Registry (`ghcr.io`)

**Images Publiées :**
```
ghcr.io/we-speak-org/auth-service:latest
ghcr.io/we-speak-org/auth-service:<commit-sha>
ghcr.io/we-speak-org/auth-service:<tag>  # si tag Git
```

### 4.4 Vérifier la Publication

```bash
# Voir les images disponibles
gh api /orgs/we-speak-org/packages/container/auth-service/versions

# Télécharger l'image
docker pull ghcr.io/we-speak-org/auth-service:latest
```

---

## Étape 5 : Déploiement

### 5.1 Setup Docker Compose pour MVP

**Fichier `docker-compose.yml` global (tous les services) :**

```yaml
version: '3.8'

services:
  # Infrastructure
  mongodb:
    image: mongo:7
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  # Services WeSpeak
  auth-service:
    image: ghcr.io/we-speak-org/auth-service:latest
    ports:
      - "8081:8081"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      MONGODB_URI: mongodb://admin:${MONGO_PASSWORD}@mongodb:27017
      KAFKA_BOOTSTRAP_SERVERS: kafka:29092
      REDIS_HOST: redis
    depends_on:
      - mongodb
      - redis
      - kafka

  lesson-service:
    image: ghcr.io/we-speak-org/lesson-service:latest
    ports:
      - "8082:8082"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      MONGODB_URI: mongodb://admin:${MONGO_PASSWORD}@mongodb:27017
      KAFKA_BOOTSTRAP_SERVERS: kafka:29092
      REDIS_HOST: redis
    depends_on:
      - mongodb
      - redis
      - kafka

  # ... autres services

volumes:
  mongo_data:
  redis_data:
```

### 5.2 Déploiement

```bash
# Créer le fichier .env
cat > .env << EOF
MONGO_PASSWORD=your_secure_password
JWT_SECRET=your_jwt_secret
# ... autres variables
EOF

# Démarrer tous les services
docker-compose up -d

# Vérifier les logs
docker-compose logs -f auth-service

# Voir les services actifs
docker-compose ps
```

### 5.3 Health Checks

Chaque service expose un endpoint `/actuator/health` :

```bash
# Vérifier la santé des services
curl http://localhost:8081/actuator/health  # auth-service
curl http://localhost:8082/actuator/health  # lesson-service
```

---

## Bonnes Pratiques

### 🎯 Développement

1. **Lire les specs complètes** avant de commencer
2. **Suivre l'ordre** : Entities → Repositories → Services → Controllers → Kafka
3. **Tester au fur et à mesure** : Ne pas tout coder d'un coup
4. **Utiliser la checklist Emergent** comme guide
5. **Commit régulièrement** avec des messages clairs

### 🔒 Sécurité

1. **Ne jamais commit** de secrets dans le code
2. **Utiliser `.env.example`** pour documenter les variables requises
3. **Variables sensibles** uniquement dans `.env` (git-ignoré)
4. **Valider les entrées** utilisateur avec Bean Validation
5. **Sanitizer les logs** (pas de données sensibles)

### 📦 CI/CD

1. **Branch `main`** protégée (require PR + reviews)
2. **Tests obligatoires** avant merge
3. **Semantic versioning** pour les tags (v1.0.0, v1.1.0, etc.)
4. **Images Docker** taguées par commit SHA + version
5. **Rollback facile** avec images versionnées

### 🚀 Performance

1. **Indexes MongoDB** selon spécifications
2. **Cache Redis** pour données fréquentes
3. **Pagination** pour les listes
4. **Rate limiting** sur API Gateway
5. **Monitoring** avec Actuator metrics

### 📚 Documentation

1. **README.md** à jour dans chaque service
2. **OpenAPI/Swagger** auto-généré
3. **Commentaires** uniquement pour logique complexe
4. **Architecture Decision Records (ADR)** pour choix importants
5. **Changelog** pour tracking des modifications

---

## 🆘 Troubleshooting

### Problème : Build Maven échoue

```bash
# Vérifier les dépendances
./mvnw dependency:tree

# Nettoyer et rebuild
./mvnw clean install -U
```

### Problème : Docker image non publiée

```bash
# Vérifier le workflow GitHub Actions
gh run list --limit 5

# Voir les logs du dernier run
gh run view --log

# Re-déclencher le workflow
gh workflow run build-and-publish.yml
```

### Problème : Service ne démarre pas

```bash
# Voir les logs détaillés
docker-compose logs -f auth-service

# Vérifier les variables d'environnement
docker-compose exec auth-service env | grep SPRING

# Redémarrer le service
docker-compose restart auth-service
```

---

## 📞 Support

- **Spécifications** : https://github.com/we-speak-org/wespeak-specifications
- **Template** : https://github.com/we-speak-org/springboot-service-template
- **Issues** : Créer une issue dans le repository concerné
- **Documentation Emergent** : Voir `docs/EMERGENT_GUIDE.md` dans chaque service

---

**Version** : 1.0.0  
**Dernière mise à jour** : 2026-01-02  
**Auteur** : WeSpeak Team
