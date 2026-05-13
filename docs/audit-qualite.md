# Audit qualité DCT — SupEvents

**Audité par** : auto-audit en posture d'auditeur externe (cf. méthode A.2 du TP)
**Date** : 13/05/2026
**Version DCT** : post-TP 1.6 (sections §1 à §11 produites hormis §6.3, §12 et trois fiches §7)
**Auditeur** : Nicolas

Posture adoptée : un développeur senior arrive dans l'équipe demain matin. Il ouvre `README.md`, navigue vers les sections par les liens du sommaire, et essaie de reconstituer le système. Que voit-il ? Qu'est-ce qui le bloque ?

> **Statut post-correction (v0.6)** : ce document est le **snapshot d'audit** au moment où il a été produit. Les défauts identifiés ci-dessous ont été corrigés dans la même PR (cf. §2 — historique des révisions v0.6) — `TDE` reformulé en chiffrement disque géré, en-tête `Idempotency-Key` documenté en §8, fiche `event.cancelled` ajoutée en §8, dates de mise à jour en pied de chaque fichier, CODEOWNERS déplacé vers `.github/`, pipeline CI en place. Les mentions de défaut dans les tableaux ci-dessous sont conservées telles quelles **pour la traçabilité de l'audit** ; supprimer le diagnostic effacerait la preuve de la correction.

---

## Catégorie 1 — Structure (sur 12)

| N° | Critère | Score | Commentaire | Action recommandée |
|---|---|---|---|---|
| S1 | Page de garde présente avec titre, version, statut, auteurs | 2/2 | `docs/01-page-de-garde.md` complète, statut `Brouillon` explicite. | — |
| S2 | Sommaire ou table des matières cliquable | 2/2 | `README.md` racine sert de sommaire avec liens et colonne statut par section. | — |
| S3 | Numérotation cohérente des sections | 1/2 | Les sections §1 à §11 existent. Mais le dossier `11-decisions-architecture/` cohabite avec `docs/adr/` — deux emplacements pour la même chose. | Choisir un seul emplacement (`docs/adr/`) et supprimer le dossier `11-decisions-architecture/` ou n'y garder qu'un README de redirection (déjà le cas, mais à clarifier). |
| S4 | Historique des révisions tenu à jour | 1/2 | Fichier présent jusqu'à v0.4. Manque l'entrée v0.5 correspondant au TP 1.6 et v0.6 pour le TP 1.7. | Ajouter les entrées correspondantes. |
| S5 | Convention de nommage des fichiers stable | 2/2 | Tous les fichiers suivent `NN-section-nom.md`. Cohérent. | — |
| S6 | Glossaire en début de document | 2/2 | `docs/03-glossaire.md` avec 20 termes contextualisés. | — |
| **Total** | | **10/12** | | |

## Catégorie 2 — Complétude (sur 14)

| N° | Critère | Score | Commentaire | Action recommandée |
|---|---|---|---|---|
| CP1 | Les 4 vues d'architecture (§6.1 à §6.4) sont présentes | 1/2 | §6.1, §6.2 et §6.4 produites. §6.3 (déploiement) est un placeholder `TODO`. | Mention explicite dans le README que §6.3 est à produire en TP 1.8. Statut déjà signalé mais peut être renforcé. |
| CP2 | Chaque module cité dans la §6.1 a une fiche §7 ou une mention « à détailler » | 1/2 | 3 fiches sur 6 modules : `EventModule`, `NotificationModule`, `UserModule` cités en §6.1 mais sans fiche §7. La matrice §10 le signale mais le §7 lui-même ne le mentionne pas. | Ajouter un en-tête dans `07-modules.md` qui liste explicitement les modules non encore détaillés et renvoie au TP 1.8. |
| CP3 | Chaque API exposée est dans le tableau synoptique §8 | 2/2 | 22 endpoints listés, alignés avec les fiches §7. | — |
| CP4 | Les événements asynchrones publiés sont documentés | 1/2 | `ticket.confirmed` et `payment.failed` documentés. `ticket.cancelled` et `event.cancelled` cités dans `TicketModule` et la matrice mais absents du §8. | Ajouter fiches + schéma JSON pour `event.cancelled` (priorité élevée car référencé partout). `ticket.cancelled` peut attendre TP 1.8. |
| CP5 | Conventions transverses §8 (versioning, erreurs, rate limiting) renseignées | 1/2 | Versioning, RFC 7807, rate limiting OK. **Manque la convention sur l'en-tête `Idempotency-Key`** alors qu'elle est imposée par l'ADR-003 et utilisée dans la fiche `TicketModule`. | Ajouter une sous-section "Idempotence" dans §8 conventions transverses, avec la sémantique de la clé. |
| CP6 | Exigences transverses §9 chiffrées | 2/2 | 3 SLO chiffrés, budget d'erreur calculé explicitement (3 h 36 / mois), 6 lignes STRIDE par flux. | — |
| CP7 | Matrice de traçabilité couvre les 21 exigences | 2/2 | 13 EF + 8 ENF listées, non couvertes signalées honnêtement. | — |
| **Total** | | **10/14** | | |

## Catégorie 3 — Cohérence (sur 12)

| N° | Critère | Score | Commentaire | Action recommandée |
|---|---|---|---|---|
| CO1 | Nommage des entités cohérent entre §6.4, §7, §8 | 2/2 | `Ticket`, `Event`, `Payment` partout. Pas de `Reservation` ou `Booking` parasite. | — |
| CO2 | Cardinalités du diagramme ER cohérentes avec le dictionnaire | 2/2 | Vérification ligne à ligne : OK. | — |
| CO3 | Codes HTTP cohérents entre §7 (tableaux d'erreurs) et §8 (tableau synoptique) | 2/2 | 409 sur `GAUGE_FULL`, 422 sur idempotence, 503 sur Stripe down — cohérents. | — |
| CO4 | Modules cités en §10 existent en §7 ou sont marqués TP 1.8 | 2/2 | `EventModule`, `NotificationModule`, `UserModule` marqués `(TP 1.8)` dans la matrice. Pas d'invention. | — |
| CO5 | ADR cités dans les fiches existent réellement | 2/2 | Les fiches mentionnent ADR-001, ADR-002, ADR-003 — tous les trois existent dans `/docs/adr/`. | — |
| CO6 | Pas d'erreur technique factuelle | 1/2 | Le §6.4 mentionne **"PostgreSQL TDE"** pour le chiffrement au repos. TDE est un terme propriétaire SQL Server / Oracle, pas une feature PostgreSQL native. Risque de confusion technique pour un dev qui chercherait à l'implémenter. | Reformuler en « chiffrement disque de l'instance gérée (RDS encryption / Cloud SQL encryption) » comme c'est correctement fait en §9.3.b. |
| **Total** | | **11/12** | | |

## Catégorie 4 — Lisibilité (sur 12)

| N° | Critère | Score | Commentaire | Action recommandée |
|---|---|---|---|---|
| L1 | Chaque diagramme a un paragraphe introductif + un paragraphe de lecture | 2/2 | Tous les diagrammes Mermaid (C4 Context, Containers, Composants, Séquences, ER) ont la structure « intro / diagramme / lecture ». | — |
| L2 | Phrases ≤ 25 mots dans les passages critiques | 1/2 | Quelques phrases longues en §9.1.1 (paragraphe descriptif inscription payante) et §9.3 (procédure droit à l'oubli, ligne "Données conservées"). | Découper les 2-3 phrases > 30 mots identifiées en §9. |
| L3 | Tableaux préférés à la prose pour les contenus structurés | 2/2 | Tableaux systématiques pour erreurs, contrats d'interface, registre RGPD, STRIDE, traçabilité. | — |
| L4 | Mise en gras utilisée pour le scan (mots-clés) | 2/2 | Termes-clés en gras dans §9 (STRIDE, budgets, périmètre). | — |
| L5 | Cohérence du français professionnel (pas de jargon marketing) | 2/2 | Style direct, terminologie technique précise. | — |
| L6 | Acronymes définis à première occurrence ou dans le glossaire | 2/2 | Glossaire couvre les 20 acronymes principaux. | — |
| **Total** | | **11/12** | | |

## Catégorie 5 — Maintenabilité (sur 10)

| N° | Critère | Score | Commentaire | Action recommandée |
|---|---|---|---|---|
| M1 | Liens internes valides (pas de lien mort) | 1/2 | Le `README.md` pointe vers `docs/06-architecture-generale/06.3-vue-deploiement.md` (placeholder) et `docs/12-annexes.md` (placeholder). Liens techniquement valides mais vers contenu vide — confusion possible. | Marquer ces liens avec le statut `🟡 TODO` visible côté lecteur (déjà le cas dans le sommaire, mais à renforcer). |
| M2 | Date de dernière mise à jour visible par section | 0/2 | Aucune métadonnée date dans les fichiers `.md` hormis §1 et §2. Un lecteur ne sait pas si une fiche est récente. | Ajouter une ligne `*Dernière mise à jour : YYYY-MM-DD*` en bas (ou tête) de chaque fichier de section. |
| M3 | CODEOWNERS présent et adapté au groupe | 2/2 | Fichier `CODEOWNERS` à la racine, attribution par section, commentaires explicites. Note : à déplacer dans `.github/CODEOWNERS` pour conformité TP 1.7. |
| M4 | ADR numérotés séquentiellement et indexés | 2/2 | ADR-001, 002, 003 + index `README.md` avec table. Format Nygard respecté. | — |
| M5 | Pipeline CI documentaire en place | 0/2 | Aucun workflow CI au moment de l'audit. Aucun linter Markdown. Aucune vérification automatique des liens ou des blocs Mermaid. | Mise en place dans la phase B/C du TP 1.7 — workflow `.github/workflows/docs-quality.yml`. |
| **Total** | | **5/10** | | |

---

## Synthèse

**Total général** : **47 / 60** (78 %)
Lecture : DCT correcte, plusieurs points à reprendre avant rendu. Cible TP 1.8 : ≥ 50 / 60.

### Top 5 des défauts les plus impactants

1. **M5 — Pipeline CI documentaire absent (0/2)**. Sans CI, rien ne détecte une régression Mermaid, un lien mort ou une faute Markdown à la prochaine PR. Impact direct sur la maintenabilité long terme. Impact élevé / coût faible (le squelette du workflow est fourni dans le TP).
2. **CO6 — Erreur technique "PostgreSQL TDE" (1/2)**. Un développeur qui lit le §6.4 et part chercher comment activer TDE sur PostgreSQL perd 30 min avant de réaliser que ça n'existe pas. Erreur factuelle ponctuelle mais visible. Impact moyen / coût faible.
3. **CP5 — Convention `Idempotency-Key` manquante en §8 (1/2)**. L'ADR-003 impose cette convention, la fiche §7.1 l'utilise, mais le contrat d'API ne la documente nulle part pour un consommateur d'API externe. Trou contractuel net. Impact élevé / coût faible.
4. **CP4 — Événement `event.cancelled` non documenté en §8 (1/2)**. Cité dans `TicketModule` (consommé) et dans la matrice (EF-09), mais sans fiche descriptive ni schéma JSON. Casse la cohérence §7 ↔ §8. Impact moyen / coût faible.
5. **M2 — Aucune date de dernière mise à jour (0/2)**. Lors d'une revue à 6 mois, impossible de savoir quelle fiche a été touchée récemment. Maintenabilité dégradée. Impact moyen / coût très faible.

### Plan de correction (priorité × impact)

| Priorité | Action | Catégorie d'audit | Coût estimé |
|---|---|---|---|
| 1 | Mettre en place le pipeline CI (`.github/workflows/docs-quality.yml`, `.markdownlint.json`) | M5 | 20 min |
| 2 | Documenter la convention `Idempotency-Key` en §8 (conventions transverses) | CP5 | 5 min |
| 3 | Corriger "PostgreSQL TDE" → formulation correcte en §6.4 | CO6 | 2 min |
| 4 | Ajouter la fiche événement `event.cancelled` en §8 (fiche descriptive + schéma JSON) | CP4 | 10 min |
| 5 | Ajouter `*Dernière mise à jour : 13/05/2026*` en pied de chaque fichier de section | M2 | 5 min |
| 6 (si temps) | Mettre à jour l'historique des révisions avec entrées v0.5 (TP 1.6) et v0.6 (TP 1.7) | S4 | 2 min |
| 7 (si temps) | Déplacer `CODEOWNERS` à `.github/CODEOWNERS` | M3 | 1 min |
| 8 (différé TP 1.8) | Découper les phrases > 30 mots en §9 | L2 | 10 min |
| Différé TP 1.8 | Produire les 3 fiches §7 manquantes (`EventModule`, `NotificationModule`, `UserModule`) | CP2 | 1 h |
| Différé TP 1.8 | Produire §6.3 vue de déploiement | CP1 | 45 min |

Les corrections 1 à 7 sont traitées dans ce TP. Les actions « Différé TP 1.8 » sont documentées comme dette acceptée jusqu'à la séance suivante.
