# §5 — Périmètre et limites

## Périmètre issu du CDC

| Dans le périmètre | Hors périmètre |
|---|---|
| Création et gestion d'événements étudiants (CRUD, publication, annulation, archivage) | Application mobile native iOS / Android |
| Catalogue d'événements consultable sans authentification | Diffusion ou streaming vidéo des événements |
| Inscription d'un étudiant authentifié + jauge limitée | Marketplace de sponsors ou régie publicitaire |
| Plusieurs types de tickets : gratuit, payant prix fixe, early-bird | Intégration avec calendriers externes (Google Calendar, Outlook) |
| Paiement via Stripe (PaymentIntent + webhook) | Stockage de données bancaires (PCI-DSS délégué à Stripe) |
| Émission d'un billet électronique avec QR code | Contrôle d'accès physique aux salles d'événements |
| Notifications email (confirmation, rappels, annulation) via SendGrid | Notifications SMS ou push mobiles natives |
| Tableau de bord organisateur quasi temps-réel | Outils décisionnels avancés / data warehouse |
| Export CSV de la liste des participants | Export comptable ou intégration ERP |
| Authentification SSO OIDC école + gestion des trois rôles | Création de comptes hors SSO école |
| Validation / révocation du statut d'organisateur par un admin | Modération automatisée par IA des descriptions d'événements |

## Limites de conception techniques ajoutées par le groupe

| Limite technique | Justification |
|---|---|
| **Migration depuis un système existant : hors périmètre** | Aucune base existante centralisée à Sup de Vinci. Le projet démarre from scratch. |
| **Support des navigateurs IE11 et antérieurs : hors périmètre** | Cible = étudiants équipés de navigateurs récents (Chrome, Firefox, Safari, Edge derniers et N-1). La conformité WCAG 2.1 AA reste assurée sur ces cibles. |
| **Multi-tenant : hors périmètre** | Une seule école déployée. Les éventuelles écoles partenaires feront l'objet d'une instance déployée séparément, pas d'un tenant logique. |
| **Mode hors-ligne / PWA : hors périmètre** | Le service exige une connexion réseau pour authentifier, payer et émettre le billet. Une version offline n'est pas justifiée par les EF. |
| **Surveillance temps-réel des paiements frauduleux côté SupEvents : hors périmètre** | Délégué au moteur Radar de Stripe. SupEvents consomme uniquement les statuts retournés via webhook. |
| **Synchronisation bidirectionnelle avec un agenda externe : hors périmètre** | Le billet inclut un lien `.ics` téléchargeable mais aucune intégration CalDAV / Google Calendar API n'est implémentée. |

Ces limites engagent l'équipe et seront référencées dans la matrice de traçabilité §10 pour expliciter ce qui **n'est pas** tracé vers une exigence CDC.
