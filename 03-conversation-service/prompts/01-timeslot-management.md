# User Story: Gestion des Créneaux Horaires

## Contexte
Le conversation-service doit permettre la création et la gestion de créneaux horaires prédéfinis pour les sessions de pratique orale.

## Objectif
Implémenter le CRUD des créneaux (TimeSlot) et le système de génération automatique.

## Entités concernées
- **TimeSlot** : Créneau avec langue, niveau, horaire, durée, capacité

## Fonctionnalités à implémenter

### 1. API REST TimeSlots
- `GET /api/v1/conversations/timeslots` : Liste des créneaux avec filtres (langue, niveau, dates)
- `GET /api/v1/conversations/timeslots/{id}` : Détail d'un créneau
- `POST /api/v1/conversations/timeslots` : Création (admin)
- `PATCH /api/v1/conversations/timeslots/{id}` : Modification (admin)
- `DELETE /api/v1/conversations/timeslots/{id}` : Suppression (admin)

### 2. Génération automatique
- Job planifié qui génère les créneaux 7 jours à l'avance
- Créneaux récurrents (daily, weekly)
- Configuration des horaires par langue/niveau

### 3. Règles métier
- Un créneau a une durée de 15, 30 ou 45 minutes
- Capacité min 2, max 8 participants
- Les créneaux passés ne sont pas modifiables

## Critères d'acceptation
- [ ] Les créneaux sont listables avec filtres
- [ ] La pagination fonctionne
- [ ] Le job de génération crée les créneaux récurrents
- [ ] Les créneaux indiquent le nombre de places disponibles

## Stack technique
- Spring Boot 4, MongoDB
- @Scheduled pour le job de génération
