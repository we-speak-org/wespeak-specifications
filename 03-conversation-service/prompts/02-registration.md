# User Story: Inscription aux Créneaux

## Contexte
Les utilisateurs doivent pouvoir s'inscrire aux créneaux disponibles pour participer aux sessions de conversation.

## Objectif
Implémenter le système d'inscription/désinscription aux créneaux.

## Entités concernées
- **Registration** : Inscription d'un utilisateur à un créneau

## Fonctionnalités à implémenter

### 1. API REST Registrations
- `POST /api/v1/conversations/timeslots/{id}/register` : S'inscrire
- `DELETE /api/v1/conversations/timeslots/{id}/register` : Se désinscrire
- `GET /api/v1/conversations/registrations` : Mes inscriptions

### 2. Règles métier
- Un utilisateur peut s'inscrire jusqu'à 5 min avant le début
- Maximum 3 inscriptions actives simultanées par utilisateur
- Annulation possible jusqu'à 15 min avant le début
- Vérifier que le créneau n'est pas complet

### 3. Notifications
- Rappel 5 minutes avant le créneau (notification push/email)
- Notification si le créneau est annulé (moins de 2 inscrits)

### 4. Statuts d'inscription
- `registered` : Inscrit
- `cancelled` : Annulé par l'utilisateur
- `attended` : A participé
- `noshow` : Inscrit mais non présent

## Critères d'acceptation
- [ ] Un utilisateur peut s'inscrire à un créneau disponible
- [ ] L'inscription est refusée si le créneau est complet
- [ ] L'inscription est refusée si l'utilisateur a déjà 3 inscriptions actives
- [ ] L'annulation tardive (< 15 min) est refusée
- [ ] La notification de rappel est envoyée

## Stack technique
- Spring Boot 4, MongoDB
- Kafka pour les notifications (optionnel)
