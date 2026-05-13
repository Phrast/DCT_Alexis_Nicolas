# §8 — Interfaces & contrats d'API

## Tableau synoptique des endpoints REST

Tous les endpoints sont préfixés par `/api/v1/`. Format : JSON (`Content-Type: application/json`). Format des erreurs : RFC 7807 Problem Details.

| Méthode | Chemin | Description | Auth | Codes retour | Dépendances aval |
|---|---|---|---|---|---|
| **Authentification** | | | | | |
| GET | `/api/v1/auth/login` | Redirige vers le SSO école (OIDC) | Public | 302 | SSO École |
| GET | `/api/v1/auth/callback` | Callback OIDC, échange code → tokens | Public | 200, 302, 400, 401 | SSO École, AuthService |
| POST | `/api/v1/auth/refresh` | Échange un refresh token contre un nouvel access token | JWT (refresh) | 200, 401, 403 | AuthService, Redis |
| POST | `/api/v1/auth/logout` | Révoque le refresh token courant | JWT | 204 | AuthService, Redis |
| **Catalogue d'événements** | | | | | |
| GET | `/api/v1/events` | Liste paginée et filtrable du catalogue | Public | 200, 400 | EventService |
| GET | `/api/v1/events/{id}` | Détail d'un événement publié | Public | 200, 404 | EventService |
| POST | `/api/v1/events` | Crée un événement (brouillon) | JWT + role:organizer | 201, 400, 403 | EventService |
| PATCH | `/api/v1/events/{id}` | Modifie un événement | JWT + role:organizer | 200, 400, 403, 404, 409 | EventService |
| POST | `/api/v1/events/{id}/publish` | Publie un événement brouillon | JWT + role:organizer | 200, 403, 404, 409 | EventService |
| POST | `/api/v1/events/{id}/cancel` | Annule un événement publié | JWT + role:organizer | 200, 403, 404, 409 | EventService, RabbitMQ |
| **Inscription / Tickets** | | | | | |
| POST | `/api/v1/tickets` | Crée un ticket en statut `pending` + initie le paiement | JWT | 201, 400, 403, 404, 409, 422 | TicketService, PaymentService, EventService |
| GET | `/api/v1/tickets/{id}` | Détail d'un ticket de l'utilisateur courant | JWT | 200, 403, 404 | TicketService |
| GET | `/api/v1/tickets/me` | Liste des tickets de l'utilisateur courant | JWT | 200 | TicketService |
| POST | `/api/v1/tickets/{id}/cancel` | Annule un ticket (déclenche remboursement si payé) | JWT | 200, 403, 404, 409 | TicketService, PaymentService, RabbitMQ |
| **Paiement** | | | | | |
| POST | `/api/v1/payments/webhook` | Endpoint webhook Stripe (signature HMAC) | **HMAC** (Stripe-Signature) | 200, 400, 401 | PaymentService, TicketService, RabbitMQ |
| **Tableau de bord organisateur** | | | | | |
| GET | `/api/v1/organizers/me/events/{id}/dashboard` | KPI temps-réel d'un événement | JWT + role:organizer | 200, 403, 404 | EventService, TicketService |
| GET | `/api/v1/organizers/me/events/{id}/participants` | Liste paginée des inscrits | JWT + role:organizer | 200, 403, 404 | TicketService |
| GET | `/api/v1/organizers/me/events/{id}/export.csv` | Export CSV des inscrits | JWT + role:organizer | 200, 403, 404, 503 | TicketService, S3 |
| **Utilisateur** | | | | | |
| GET | `/api/v1/users/me` | Profil de l'utilisateur courant | JWT | 200, 401 | UserService |
| PATCH | `/api/v1/users/me` | Met à jour préférences (locale, etc.) | JWT | 200, 400 | UserService |
| DELETE | `/api/v1/users/me` | Déclenche le droit à l'oubli | JWT | 202, 401 | UserService |
| **Administration plateforme** | | | | | |
| POST | `/api/v1/admin/organizers/{userId}/approve` | Valide un organisateur | JWT + role:admin | 200, 403, 404, 409 | UserService |
| POST | `/api/v1/admin/organizers/{userId}/revoke` | Révoque un statut organisateur | JWT + role:admin | 200, 403, 404 | UserService |

---

## Conventions transverses

### Format des erreurs (RFC 7807 Problem Details)

Toutes les réponses d'erreur respectent le format suivant :

```json
{
  "type": "https://supevents.example/errors/gauge-full",
  "title": "Gauge full",
  "status": 409,
  "detail": "L'événement a atteint sa jauge maximale.",
  "instance": "/api/v1/tickets",
  "code": "GAUGE_FULL",
  "traceId": "01HX5K3V7T7Q..."
}
```

### Stratégie de versioning

Versioning par préfixe d'URL (`/api/v1/`). Les évolutions non rupturees (ajout d'un champ optionnel, nouveau endpoint) restent dans `v1`. Une rupture (renommage, suppression, sémantique modifiée) provoque la création de `v2` avec dépréciation annoncée 6 mois avant retrait de `v1`.

### Idempotence des écritures critiques

Les endpoints d'écriture critique acceptent un en-tête HTTP `Idempotency-Key: <uuid v4>` généré par le client. Le serveur stocke la paire `(clé, réponse)` dans Redis pendant 24 h ; tout rejeu avec la même clé renvoie la réponse initiale sans rejouer la transaction métier. Détails et alternatives écartées dans l'[ADR-003](adr/ADR-003.md).

| Endpoint | `Idempotency-Key` |
|---|---|
| `POST /api/v1/tickets` | **Obligatoire** |
| `POST /api/v1/tickets/{id}/cancel` | **Obligatoire** |
| `POST /api/v1/events/{id}/cancel` | **Obligatoire** |
| Autres écritures (`PATCH`, `POST /events`, `POST /events/{id}/publish`) | Recommandé |

Cas d'erreur dédiés : `400 IDEMPOTENCY_KEY_MISSING` si l'en-tête est requis mais absent, `422 IDEMPOTENCY_KEY_REPLAY_MISMATCH` si la clé est réutilisée avec un corps différent (durcissement prévu en itération 2, cf. §7.1).

### Rate limiting et quotas

| Périmètre | Limite | Comportement sur dépassement |
|---|---|---|
| Endpoints publics (`/events*`) | 60 req/min par IP | `429 Too Many Requests` + `Retry-After` |
| Endpoints authentifiés | 300 req/min par `userId` | `429 Too Many Requests` + `Retry-After` |
| Endpoint webhook Stripe | Pas de limite côté SupEvents | Stripe gère ses propres retries |

---

## Événements asynchrones

Quatre événements asynchrones de la plateforme sont documentés ci-dessous. Tous transitent par l'exchange topic `supevents.events.v1` ; la liste sera enrichie au fil des itérations (`user.provisioned` est candidat, cf. §7.3).

| Nom | Producteur | Consommateurs principaux |
|---|---|---|
| `ticket.confirmed` | `TicketModule` | `NotificationWorker`, `AnalyticsWorker` (futur) |
| `ticket.cancelled` | `TicketModule` | `NotificationWorker` |
| `payment.failed` | `PaymentModule` | `NotificationWorker`, `OpsAlertWorker` (futur) |
| `event.cancelled` | `EventModule` | `TicketModule`, `NotificationWorker` |

### `ticket.confirmed`

| Champ | Valeur |
|---|---|
| **Nom** | `ticket.confirmed` |
| **Producteur** | `TicketModule` — publié après confirmation du paiement (webhook Stripe `payment_intent.succeeded`) ou après création d'un ticket gratuit |
| **Topic / exchange** | `supevents.events.v1` (exchange topic) — routing key `ticket.confirmed` |
| **Consommateurs connus** | `NotificationWorker` (envoie email de confirmation avec QR via SendGrid) ; `AnalyticsWorker` (incrémente compteurs dashboard organisateur — futur) |
| **Garantie de livraison** | `at-least-once`. Le consommateur doit être idempotent (déduplication par `eventId` du message). |
| **Stratégie de retry** | Politique exponentielle : 3 tentatives en 1 min / 5 min / 15 min. Après 3 échecs → dead letter queue `supevents.events.v1.dlq` + alerte Slack. |

**Schéma JSON du payload** :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TicketConfirmedEvent",
  "type": "object",
  "required": ["eventId", "occurredAt", "ticketId", "userId", "supEventId", "ticketType", "amount", "currency"],
  "properties": {
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du message (clé de déduplication)"
    },
    "occurredAt": {
      "type": "string",
      "format": "date-time",
      "description": "Date d'occurrence du fait métier (ISO 8601 UTC)"
    },
    "ticketId": {
      "type": "string",
      "format": "uuid"
    },
    "userId": {
      "type": "string",
      "format": "uuid"
    },
    "supEventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'événement SupEvents (séparé de eventId qui est l'id du message)"
    },
    "ticketType": {
      "type": "string",
      "enum": ["free", "standard", "early_bird"]
    },
    "amount": {
      "type": "number",
      "minimum": 0
    },
    "currency": {
      "type": "string",
      "pattern": "^[A-Z]{3}$"
    },
    "qrCodeToken": {
      "type": "string",
      "description": "Jeton HMAC à encoder dans le QR code"
    }
  }
}
```

### `payment.failed`

| Champ | Valeur |
|---|---|
| **Nom** | `payment.failed` |
| **Producteur** | `PaymentModule` — publié lorsqu'un webhook Stripe `payment_intent.payment_failed` est reçu, ou qu'un paiement reste `pending` au-delà du timeout (5 min, réconciliation périodique) |
| **Topic / exchange** | `supevents.events.v1` (exchange topic) — routing key `payment.failed` |
| **Consommateurs connus** | `NotificationWorker` (envoie email d'échec avec lien de retry à l'étudiant) ; `OpsAlertWorker` (alerte si le taux d'échec dépasse 5 % sur 5 min — futur). La libération de la place sur l'événement est effectuée **synchroniquement** par `PaymentModule → TicketModule.cancelTicket()` dans la transaction du webhook (cf. §6.2.2), pas via le broker. |
| **Garantie de livraison** | `at-least-once` + idempotence consommateur sur `eventId`. |
| **Stratégie de retry** | Politique exponentielle : 3 tentatives en 1 min / 5 min / 15 min, puis DLQ + alerte. |

**Schéma JSON du payload** :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "PaymentFailedEvent",
  "type": "object",
  "required": ["eventId", "occurredAt", "paymentId", "ticketId", "userId", "supEventId", "failureCode"],
  "properties": {
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du message (clé de déduplication)"
    },
    "occurredAt": {
      "type": "string",
      "format": "date-time"
    },
    "paymentId": {
      "type": "string",
      "format": "uuid"
    },
    "ticketId": {
      "type": "string",
      "format": "uuid"
    },
    "userId": {
      "type": "string",
      "format": "uuid"
    },
    "supEventId": {
      "type": "string",
      "format": "uuid"
    },
    "failureCode": {
      "type": "string",
      "enum": ["card_declined", "insufficient_funds", "expired_card", "authentication_required", "timeout", "other"]
    },
    "failureMessage": {
      "type": "string",
      "description": "Message Stripe traduit côté SupEvents, non sensible"
    },
    "retryUrl": {
      "type": "string",
      "format": "uri",
      "description": "URL d'un nouvel essai de paiement"
    }
  }
}
```

### `ticket.cancelled`

| Champ | Valeur |
|---|---|
| **Nom** | `ticket.cancelled` |
| **Producteur** | `TicketModule` — publié lors d'une annulation utilisateur (`POST /tickets/{id}/cancel`), d'un échec de paiement traité (cf. §6.2.2) ou d'une annulation en cascade suite à `event.cancelled` |
| **Topic / exchange** | `supevents.events.v1` (exchange topic) — routing key `ticket.cancelled` |
| **Consommateurs connus** | `NotificationWorker` (envoie l'email d'annulation, mentionnant le remboursement si applicable) |
| **Garantie de livraison** | `at-least-once` + idempotence consommateur sur `eventId`. |
| **Stratégie de retry** | Politique exponentielle : 3 tentatives en 1 min / 5 min / 15 min, puis DLQ `supevents.events.v1.dlq` + alerte. |

**Schéma JSON du payload** :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TicketCancelledEvent",
  "type": "object",
  "required": ["eventId", "occurredAt", "ticketId", "userId", "supEventId", "reason"],
  "properties": {
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du message (clé de déduplication)"
    },
    "occurredAt": {
      "type": "string",
      "format": "date-time"
    },
    "ticketId": {
      "type": "string",
      "format": "uuid"
    },
    "userId": {
      "type": "string",
      "format": "uuid"
    },
    "supEventId": {
      "type": "string",
      "format": "uuid"
    },
    "reason": {
      "type": "string",
      "enum": ["user_request", "payment_failed", "payment_timeout", "event_cancelled"]
    },
    "refundIssued": {
      "type": "boolean",
      "description": "Indique si un remboursement Stripe a été initié"
    }
  }
}
```

### `event.cancelled`

| Champ | Valeur |
|---|---|
| **Nom** | `event.cancelled` |
| **Producteur** | `EventModule` — publié quand un organisateur appelle `POST /events/{id}/cancel` sur un événement déjà publié |
| **Topic / exchange** | `supevents.events.v1` (exchange topic) — routing key `event.cancelled` |
| **Consommateurs connus** | `TicketModule` (annule tous les tickets `pending` ou `confirmed` de l'événement et publie un `ticket.cancelled` par ticket impacté) ; `NotificationWorker` (envoie l'email d'annulation aux inscrits) |
| **Garantie de livraison** | `at-least-once` + idempotence consommateur sur `eventId`. La consommation par `TicketModule` est protégée par un verrou applicatif sur `supEventId` pour éviter les annulations en double. |
| **Stratégie de retry** | Politique exponentielle : 3 tentatives en 1 min / 5 min / 15 min, puis DLQ + alerte. |

**Schéma JSON du payload** :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "EventCancelledEvent",
  "type": "object",
  "required": ["eventId", "occurredAt", "supEventId", "organizerId", "cancelledAt"],
  "properties": {
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du message (clé de déduplication)"
    },
    "occurredAt": {
      "type": "string",
      "format": "date-time"
    },
    "supEventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'événement SupEvents annulé"
    },
    "organizerId": {
      "type": "string",
      "format": "uuid"
    },
    "cancelledAt": {
      "type": "string",
      "format": "date-time"
    },
    "reason": {
      "type": "string",
      "description": "Motif libre saisi par l'organisateur (non sensible)"
    },
    "impactedTicketCount": {
      "type": "integer",
      "minimum": 0,
      "description": "Nombre de tickets actifs au moment de l'annulation (informatif)"
    }
  }
}
```
