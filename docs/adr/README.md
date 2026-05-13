# §11 — Décisions d'architecture (ADR)

Les ADR (Architecture Decision Records) suivent le format Nygard. Un ADR ne se supprime jamais ; son statut évolue.

| N° | Titre | Statut | Date | Sections DCT liées |
|---|---|---|---|---|
| [ADR-001](ADR-001.md) | Adopter RabbitMQ comme broker de messages | ✅ Accepté | 13/05/2026 | §6.1, §6.4, §7.1, §8 |
| [ADR-002](ADR-002.md) | Utiliser un verrou pessimiste sur la jauge plutôt qu'un verrou optimiste | ✅ Accepté | 13/05/2026 | §6.2, §6.4, §7.1 |
| [ADR-003](ADR-003.md) | Idempotence des écritures par clé client transmise dans l'en-tête `Idempotency-Key` | ✅ Accepté | 13/05/2026 | §7.1, §7.2, §8 |

## Convention

- Format de nom : `ADR-NNN.md` (numérotation séquentielle, jamais réutilisée).
- Template : [`template.md`](template.md).
- Mise à jour du statut via PR. Le contenu d'un ADR `Accepté` ne se modifie pas — il est `Déprécié` ou `Remplacé par ADR-XXX` si nécessaire.
