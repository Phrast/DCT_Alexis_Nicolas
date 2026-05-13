# SupEvents — Document de Conception Technique

Dépôt DCT du projet **SupEvents**, plateforme de gestion d'événements étudiants.

| Champ | Valeur |
|---|---|
| Version courante | 0.6 |
| Statut | 🟡 Brouillon (TP 1.7 intégré, en route vers la soutenance) |
| Mainteneurs | @nicolas, @alexis |
| Score audit qualité | 47 / 60 — cf. [docs/audit-qualite.md](docs/audit-qualite.md) |

> **TP 1.1** — l'analyse critique des DCT RoomBook se trouve à la racine du dépôt : [analyse-critique-roombook.md](analyse-critique-roombook.md). Elle n'appartient pas à la DCT SupEvents elle-même.

## Sommaire

| Section | Fichier | Statut |
|---|---|---|
| §1 — Page de garde | [docs/01-page-de-garde.md](docs/01-page-de-garde.md) | ✅ |
| §2 — Historique des révisions | [docs/02-historique-revisions.md](docs/02-historique-revisions.md) | ✅ |
| §3 — Glossaire | [docs/03-glossaire.md](docs/03-glossaire.md) | ✅ |
| §4 — Contexte et objectifs | [docs/04-contexte-objectifs.md](docs/04-contexte-objectifs.md) | ✅ |
| §5 — Périmètre et limites | [docs/05-perimetre-limites.md](docs/05-perimetre-limites.md) | ✅ |
| §6.1 — Vue logique | [docs/06-architecture-generale/06.1-vue-logique.md](docs/06-architecture-generale/06.1-vue-logique.md) | ✅ |
| §6.2 — Vue des processus | [docs/06-architecture-generale/06.2-vue-processus.md](docs/06-architecture-generale/06.2-vue-processus.md) | ✅ |
| §6.3 — Vue de déploiement | [docs/06-architecture-generale/06.3-vue-deploiement.md](docs/06-architecture-generale/06.3-vue-deploiement.md) | 🟡 TODO (TP 1.8) |
| §6.4 — Vue des données | [docs/06-architecture-generale/06.4-vue-donnees.md](docs/06-architecture-generale/06.4-vue-donnees.md) | ✅ |
| §7 — Conception détaillée par module | [docs/07-conception-detaillee/07-modules.md](docs/07-conception-detaillee/07-modules.md) | 🟡 3 fiches sur 6 (TicketModule, PaymentModule, AuthModule — reste TP 1.8) |
| §8 — Interfaces & contrats d'API | [docs/08-interfaces-api.md](docs/08-interfaces-api.md) | ✅ |
| §9 — Exigences transverses | [docs/09-exigences-transverses.md](docs/09-exigences-transverses.md) | ✅ |
| §10 — Matrice de traçabilité | [docs/10-matrice-tracabilite.md](docs/10-matrice-tracabilite.md) | ✅ |
| §11 — Décisions d'architecture (ADR) | [docs/adr/README.md](docs/adr/README.md) | ✅ (3 ADR) |
| §12 — Annexes | [docs/12-annexes.md](docs/12-annexes.md) | 🟡 TODO (TP 1.8) |

## Documents transverses

| Document | Rôle |
|---|---|
| [docs/audit-qualite.md](docs/audit-qualite.md) | Audit qualité (60 points, 5 catégories) — TP 1.7 |
| [docs/ci-validation.md](docs/ci-validation.md) | Validation du pipeline CI documentaire |
| [.github/CODEOWNERS](.github/CODEOWNERS) | Attribution des sections aux mainteneurs |
| [.github/workflows/docs-quality.yml](.github/workflows/docs-quality.yml) | Pipeline CI (lint + liens + Mermaid) |
| [.markdownlint.json](.markdownlint.json) | Configuration du linter Markdown |

## Conventions

- **Branches** : `docs/section-XX-description`
- **Commits** : `docs(§XX): description courte au présent`
- **Rendu Mermaid** : vérifié sur [mermaid.live](https://mermaid.live) avant merge, et automatiquement par le pipeline CI.
- **Idempotence des API** : tout endpoint d'écriture critique accepte `Idempotency-Key` (cf. §8 conventions transverses + ADR-003).

## CI documentaire

Toute pull request modifiant un fichier `.md` déclenche le workflow `docs-quality.yml` qui exécute trois vérifications :

1. **Lint Markdown** (`markdownlint-cli`) avec règles minimales adaptées à une DCT enrichie.
2. **Vérification des liens** internes et externes (`markdown-link-check`).
3. **Validation du rendu Mermaid** (`@mermaid-js/mermaid-cli`).

Les `CODEOWNERS` de chaque section modifiée sont auto-assignés comme reviewers obligatoires.
