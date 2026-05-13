# §4 — Contexte et objectifs

## Contexte technique

SupEvents est une plateforme web destinée à remplacer les processus manuels actuels de gestion d'événements étudiants à Sup de Vinci (formulaires papier, virements bancaires, listes Excel). Les utilisateurs sont les étudiants (consultation, inscription, paiement), les organisateurs (associations, administration) et les administrateurs plateforme.

D'un point de vue technique, ce besoin se traduit par cinq enjeux structurants :

1. **Parcours d'inscription frictionless** : authentification déléguée au SSO école (OIDC) pour ne pas dupliquer la base d'identités, et capture de paiement en moins de trois étapes côté utilisateur.
2. **Conformité réglementaire dès la conception** : RGPD (droit à l'oubli, registre des traitements, durée de rétention) et PCI-DSS entièrement délégué à Stripe — aucune donnée bancaire ne transite par nos serveurs.
3. **Cohérence transactionnelle sur la jauge** : la création d'un ticket et la capture du paiement doivent être atomiquement liées à la décrémentation des places disponibles, sans risque de double allocation lors de pics d'inscription.
4. **Notifications fiables et asynchrones** : confirmation, rappels avant événement et alertes d'annulation passent par un broker de messages pour découpler le parcours synchrone de l'envoi email (SendGrid).
5. **Disponibilité et accessibilité** : SLO ≥ 99,5 % mensuel, p95 < 500 ms sur les routes critiques, conformité WCAG 2.1 AA pour les étudiants en situation de handicap.

## Objectifs techniques implicites du CDC

| Objectif technique | Origine dans le CDC | Niveau de priorité |
|---|---|---|
| Scalabilité horizontale des services synchrones | "500 utilisateurs simultanés au pic" | Élevé |
| Conformité PCI-DSS par délégation Stripe | "Paiement en ligne via Stripe" | Élevé |
| Conformité RGPD avec droit à l'oubli effectif | "Données étudiantes manipulées" + cadre légal européen | Élevé |
| Conformité WCAG 2.1 AA | "Accessibilité pour les étudiants en situation de handicap" | Élevé |
| Internationalisation FR/EN | "Catalogue accessible aux étudiants internationaux" | Moyen |
| Observabilité (logs JSON, métriques, alertes) | "Tableau de bord temps-réel" + diagnostics incidents | Moyen |
| Idempotence des opérations critiques | "Paiement fiable, double-clic toléré" (implicite) | Élevé |
| Découplage asynchrone des notifications | "Rappels paramétrables avant événement" | Moyen |
| Versioning d'API stable | "Plateforme amenée à évoluer" (implicite) | Moyen |
| Cycle de vie documenté des événements | "Brouillon → publié → annulé → archivé" (implicite via EF-01/EF-09) | Élevé |

Ces objectifs constituent le socle des exigences transverses détaillées en §9 et nourrissent la matrice de traçabilité §10.
