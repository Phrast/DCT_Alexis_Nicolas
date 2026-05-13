# §10 — Matrice de traçabilité

Cette section croise les 21 exigences du cahier des charges SupEvents (13 EF + 8 ENF) avec les éléments de DCT qui les couvrent : modules détaillés en §7, sections de référence (§6.4, §8, §9), ADR pertinents. Elle constitue le point d'entrée d'un audit ou d'une revue de couverture : un auditeur doit pouvoir vérifier en 30 minutes que toutes les exigences ont au moins un porteur explicite. Les exigences encore non couvertes à ce stade (TP 1.6) sont signalées et renvoyées au TP 1.8.

## §10.1 — Matrice principale

### Exigences fonctionnelles (EF)

| Réf. | Intitulé (court) | Module(s) | Section(s) DCT | ADR |
|---|---|---|---|---|
| EF-01 | CRUD événements | `EventModule` (TP 1.8) | §6.4 (`Event`, `Category`), §8 (`/events*`) | — |
| EF-02 | Catalogue public | `EventModule` (TP 1.8) | §6.4 (`Event`), §8 (`GET /events`), §9.2.1 | — |
| EF-03 | Jauge limitée | `EventModule`, `TicketModule` | §6.4 (`Event.gauge_*`), §7.1, §6.2.1, §9.2.3 | ADR-002 |
| EF-04 | Types de tickets (gratuit / standard / early-bird) | `TicketModule`, `EventModule` (TP 1.8) | §6.4 (`Ticket.ticket_type`), §7.1, §8 | — |
| EF-05 | Inscription + paiement Stripe | `TicketModule`, `PaymentModule` | §6.4 (`Ticket`, `Payment`), §6.2.1, §7.1, §7.2, §8, §9.1.1 | ADR-002, ADR-003 |
| EF-06 | Billet électronique + QR | `TicketModule` | §6.4 (`Ticket.qr_code_token`), §7.1 | — |
| EF-07 | Email confirmation via SendGrid | `NotificationModule` (TP 1.8) | §6.4 (`Notification`), §8 (événement `ticket.confirmed`), §6.1.2 | ADR-001 |
| EF-08 | Rappels paramétrables | `NotificationModule` (TP 1.8), `EventModule` (TP 1.8) | ⚠️ Non couvert — à traiter en TP 1.8 (worker scheduler + paramétrage) | — |
| EF-09 | Notifications annulation / modification | `NotificationModule` (TP 1.8), `EventModule` (TP 1.8) | §8 (événement `event.cancelled` — à documenter TP 1.8) | ADR-001 |
| EF-10 | Tableau de bord organisateur temps-réel | `EventModule` (TP 1.8), `TicketModule` | §8 (`/organizers/me/events/{id}/dashboard`) — ⚠️ partiellement couvert | — |
| EF-11 | Export CSV participants | `TicketModule`, `EventModule` (TP 1.8) | §6.4 (stockage S3), §8 (`GET /organizers/.../export.csv`) | — |
| EF-12 | Authentification SSO OIDC + rôles | `AuthModule`, `UserModule` (TP 1.8) | §6.4 (`User.role`, `RefreshToken`), §6.1.3, §7.3, §8 (`/auth/*`), §9.1.2 | — |
| EF-13 | Validation / révocation organisateur par admin | `UserModule` (TP 1.8) | §6.4 (`Organizer`), §8 (`/admin/organizers/{userId}/approve\|revoke`) | — |

### Exigences non-fonctionnelles (ENF)

| Réf. | Intitulé (court) | Module(s) | Section(s) DCT | ADR |
|---|---|---|---|---|
| ENF-01 | Performance — p95 < 500 ms + 500 users | `EventModule`, `TicketModule`, `PaymentModule` | §9.2.1, §9.2.3, §6.4 (index), §6.1.2 (autoscaling) | ADR-002 |
| ENF-02 | Disponibilité ≥ 99,5 % mensuel | Tous services | §9.2.2 (budget d'erreur 3 h 36) | — |
| ENF-03 | Sécurité — RGPD, TLS, PCI-DSS délégué | `AuthModule`, `UserModule` (TP 1.8), `PaymentModule` | §9.1.1, §9.1.2, §9.3, §6.4 (marquage RGPD) | — |
| ENF-04 | Accessibilité WCAG 2.1 AA | Frontend SPA | ⚠️ Non couvert côté DCT — relève du périmètre frontend, à documenter en §9.4 au TP 1.8 (audit Axe + Lighthouse en CI) | — |
| ENF-05 | i18n FR + EN | Frontend SPA, `UserModule` (TP 1.8) | §6.4 (`User.locale`, `Category.label_fr/en`) — ⚠️ partiellement couvert | — |
| ENF-06 | Observabilité (logs JSON + métriques + alertes) | Tous services | §9.1 (audit log), §9.2 (Prometheus + Grafana + PagerDuty) — ⚠️ à approfondir en §9.5 au TP 1.8 | — |
| ENF-07 | Maintenabilité (doc + CI/CD) | Dépôt DCT lui-même | §11 (ADR), §10 (cette matrice), CI/CD documentaire (TP 1.7) | — |
| ENF-08 | Scalabilité horizontale | `TicketModule`, `PaymentModule`, `EventModule` (TP 1.8) | §6.1.2 (containers stateless), §9.2.3, ADR-001 (broker découplé) | ADR-001, ADR-002 |

## §10.2 — Synthèse de couverture

### Décompte

| Catégorie | Total | Couvertes pleinement | Couvertes partiellement | Non couvertes |
|---|---|---|---|---|
| EF | 13 | 9 | 3 (EF-08 partiel, EF-09 partiel, EF-10 partiel) | 1 (EF-08 si l'on ne compte pas le partiel comme couvert) |
| ENF | 8 | 4 | 3 (ENF-05, ENF-06 partiel + ENF-04 non documenté) | 1 (ENF-04 non encore documenté côté DCT) |
| **Total** | **21** | **13** | **6** | **2** |

### Exigences en attente

| Réf. | Description du gap | Traitement prévu |
|---|---|---|
| EF-08 | Rappels paramétrables avant événement — le worker scheduler et le paramétrage côté organisateur ne sont pas encore conçus. | TP 1.8 — ajout d'une fiche `NotificationScheduler` en §7 et d'un endpoint `PATCH /events/{id}/reminder-settings` en §8. |
| EF-09 | Événement `event.cancelled` cité dans `TicketModule` mais non documenté en §8 (fiche + schéma JSON). | TP 1.8 — ajout de la fiche événement dans §8. |
| EF-10 | Endpoint `dashboard` listé en §8 mais sans détail sur le calcul KPI quasi temps-réel (CQRS ? projection RabbitMQ → vue matérialisée ?). | TP 1.8 — décision de design + ADR-004 candidat sur la stratégie de calcul des KPI organisateur. |
| ENF-04 | Accessibilité WCAG 2.1 AA — frontend non couvert en DCT (la DCT documente le backend en priorité). | TP 1.8 — ajout d'une §9.4 Accessibilité avec exigences sur les contrôles Axe / Lighthouse exécutés en CI front. |
| ENF-05 | i18n — `User.locale` stocké, mais pas de stratégie complète documentée (formats de dates, monnaies, négociation `Accept-Language`). | TP 1.8 — §9.6 à créer. |
| ENF-06 | Observabilité — bribes éparses dans §9, mais pas de section dédiée centralisant les SLI, les dashboards Grafana, la stratégie de log levels. | TP 1.8 — §9.5 Observabilité à structurer. |

### Vérification par module (lecture inverse)

Décompte des apparitions de chaque module dans la matrice, pour détecter les anomalies de couverture :

| Module | Nombre d'apparitions | Statut | Commentaire |
|---|---|---|---|
| `TicketModule` | 8 | OK | Module central, présence cohérente. Fiche complète en §7.1. |
| `PaymentModule` | 3 | OK | Présence cohérente avec sa responsabilité. Fiche complète en §7.2. |
| `AuthModule` | 2 | OK | Responsabilité ciblée. Fiche complète en §7.3. |
| `EventModule` | 9 | ⚠️ Élevé | Module non encore détaillé en §7 ; risque de devenir fourre-tout. Fiche §7 à produire au TP 1.8 avec attention à la séparation des responsabilités (CRUD + jauge + recherche + KPI organisateur). |
| `NotificationModule` | 3 | ⚠️ | Module non détaillé en §7. Fiche à produire TP 1.8 (envoi async, retry, DLQ, templates). |
| `UserModule` | 4 | ⚠️ | Module non détaillé en §7. Fiche à produire TP 1.8 (profil, droit à l'oubli, statut organisateur). |

### Note d'honnêteté

À l'issue du TP 1.6, **13 exigences sur 21 sont pleinement couvertes**, soit 62 %. C'est un état attendu d'une DCT en construction : la couverture progressive est la norme. La matrice rend visibles les zones non encore traitées plutôt que de les masquer. Le TP 1.8 dispose ainsi d'une liste d'actions précise et ordonnée pour finaliser le document avant soutenance.
