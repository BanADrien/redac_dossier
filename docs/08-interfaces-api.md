# §8 — Interfaces & Contrats d'API

## 8.1 Tableau synoptique des endpoints REST

Tous les endpoints sont préfixés par `/api/v1/`. La colonne **Auth** indique le mécanisme d'authentification requis.

| Méthode | Chemin | Description | Auth | Codes retour | Dépendances aval |
|---------|--------|-------------|------|--------------|-----------------|
| `POST` | `/auth/callback` | Callback OIDC — échange code → JWT applicatif | Public | 200, 400, 502 | SSO École |
| `POST` | `/auth/refresh` | Renouvellement du JWT via refresh token | Public (refresh token) | 200, 401, 403 | Redis |
| `POST` | `/auth/logout` | Révocation du refresh token | JWT | 204, 401 | Redis |
| `GET` | `/events` | Liste paginée des événements publiés (recherche, filtres) | Public | 200, 400 | PostgreSQL |
| `GET` | `/events/:id` | Détail d'un événement | Public | 200, 404 | PostgreSQL |
| `POST` | `/events` | Créer un événement | JWT + role:organizer | 201, 400, 403 | PostgreSQL, MinIO |
| `PATCH` | `/events/:id` | Modifier un événement (titre, description, dates) | JWT + role:organizer | 200, 400, 403, 404 | PostgreSQL |
| `DELETE` | `/events/:id` | Annuler un événement | JWT + role:organizer | 204, 403, 404, 409 | PostgreSQL, RabbitMQ |
| `POST` | `/tickets` | Créer une réservation (inscription) | JWT | 201, 400, 409 | PostgreSQL, Stripe |
| `GET` | `/tickets/:id` | Détail d'un ticket | JWT | 200, 403, 404 | PostgreSQL |
| `DELETE` | `/tickets/:id` | Annuler un ticket (remboursement) | JWT | 204, 400, 403, 404 | Stripe, RabbitMQ |
| `GET` | `/tickets/:id/qr` | Récupérer le QR code du ticket | JWT | 200, 403, 404 | MinIO |
| `POST` | `/webhooks/stripe` | Recevoir un webhook Stripe | HMAC (Stripe-Signature) | 200, 400, 401 | PostgreSQL, RabbitMQ |
| `GET` | `/organizer/events` | Liste des événements de l'organisateur connecté | JWT + role:organizer | 200, 403 | PostgreSQL |
| `GET` | `/organizer/events/:id/participants` | Liste des participants à un événement | JWT + role:organizer | 200, 403, 404 | PostgreSQL |
| `GET` | `/organizer/events/:id/export` | Export CSV des participants | JWT + role:organizer | 200, 403, 404 | PostgreSQL |
| `GET` | `/admin/organizers` | Liste des demandes d'organisateur | JWT + role:admin | 200, 403 | PostgreSQL |
| `PATCH` | `/admin/organizers/:id` | Valider ou révoquer un organisateur | JWT + role:admin | 200, 400, 403, 404 | PostgreSQL, RabbitMQ |
| `GET` | `/users/me` | Profil de l'utilisateur connecté | JWT | 200, 401 | PostgreSQL |
| `PATCH` | `/users/me` | Modifier les préférences | JWT | 200, 400, 401 | PostgreSQL |
| `DELETE` | `/users/me` | Droit à l'oubli RGPD (pseudonymisation) | JWT | 204, 401 | PostgreSQL |

---

## 8.2 Documentation des événements asynchrones

### 8.2.1 Événement `ticket.confirmed`

**Description** : Émis par `PaymentModule` lorsque le webhook Stripe `payment_intent.succeeded` est reçu et traité avec succès. Consommé par `NotificationWorker` pour envoyer l'email de confirmation avec le QR code.

**Exchange** : `supevents.events` (type: topic)  
**Routing key** : `ticket.confirmed`  
**Queue** : `notification.ticket.confirmed`

**Schéma JSON** :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["eventType", "ticketId", "userId", "eventId", "email", "amount", "qrCodeUrl", "timestamp"],
  "properties": {
    "eventType": {
      "type": "string",
      "enum": ["ticket.confirmed"],
      "description": "Type discriminant de l'événement"
    },
    "ticketId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du ticket confirmé"
    },
    "userId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'utilisateur détenteur"
    },
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'événement SupEvents"
    },
    "eventTitle": {
      "type": "string",
      "description": "Titre de l'événement (dénormalisé pour la notification)"
    },
    "email": {
      "type": "string",
      "format": "email",
      "description": "Email du destinataire"
    },
    "amount": {
      "type": "number",
      "description": "Montant payé en EUR"
    },
    "qrCodeUrl": {
      "type": "string",
      "format": "uri",
      "description": "URL MinIO du QR code du ticket"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "Date et heure de confirmation (ISO 8601)"
    }
  }
}
```

**Exemple** :
```json
{
  "eventType": "ticket.confirmed",
  "ticketId": "a3f1b2c4-1234-5678-9abc-def012345678",
  "userId": "b9d2e3f5-abcd-ef01-2345-678901234567",
  "eventId": "c5e4f6a7-0000-1111-2222-333344445555",
  "eventTitle": "Gala de la Promo 2026",
  "email": "adrien.detee@etudiant.ecole.fr",
  "amount": 15.00,
  "qrCodeUrl": "https://minio.supevents.local/tickets/qr/a3f1b2c4-1234-5678-9abc-def012345678.png",
  "timestamp": "2026-05-13T20:35:00Z"
}
```

---

### 8.2.2 Événement `payment.failed`

**Description** : Émis par `PaymentModule` lorsque le webhook Stripe `payment_intent.payment_failed` est reçu, ou lors de la réconciliation des paiements expirés. Consommé par `NotificationWorker` pour notifier l'étudiant de l'échec et libérer la réservation.

**Exchange** : `supevents.events` (type: topic)  
**Routing key** : `payment.failed`  
**Queue** : `notification.payment.failed`

**Schéma JSON** :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["eventType", "ticketId", "userId", "email", "reason", "timestamp"],
  "properties": {
    "eventType": {
      "type": "string",
      "enum": ["payment.failed", "payment.timed_out"],
      "description": "Distingue l'échec explicite du timeout"
    },
    "ticketId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du ticket annulé"
    },
    "userId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'utilisateur"
    },
    "email": {
      "type": "string",
      "format": "email",
      "description": "Email du destinataire"
    },
    "reason": {
      "type": "string",
      "description": "Raison lisible de l'échec (ex: 'card_declined', 'insufficient_funds', 'timeout')"
    },
    "stripeDeclineCode": {
      "type": "string",
      "description": "Code de déclin Stripe si applicable (optionnel)"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "Date et heure de l'échec (ISO 8601)"
    }
  }
}
```

**Exemple** :
```json
{
  "eventType": "payment.failed",
  "ticketId": "a3f1b2c4-1234-5678-9abc-def012345678",
  "userId": "b9d2e3f5-abcd-ef01-2345-678901234567",
  "email": "adrien.detee@etudiant.ecole.fr",
  "reason": "card_declined",
  "stripeDeclineCode": "insufficient_funds",
  "timestamp": "2026-05-13T20:36:30Z"
}
```

---

*Dernière mise à jour : 2026-05-13*
