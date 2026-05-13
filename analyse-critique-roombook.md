# TP 1.1 — Analyse critique de DCT RoomBook

**Auteur** : Nicolas
**Date** : 13/05/2026
**Cible** : Évaluer la qualité documentaire de trois extraits de DCT portant sur le même système (RoomBook — réservation de salles de réunion).

---

## 1. Synthèse comparative

| Critère | Extrait A | Extrait B | Extrait C |
|---|---|---|---|
| Statut documentaire | Approuvé v1.4.0 | Non précisé | Aucun en-tête |
| Diagramme contexte (C4 L1) | Présent, légendé | Absent | Absent |
| Diagramme séquence | Présent, complet | Absent | Absent |
| Glossaire | 8 termes contextualisés | Acronymes utilisés sans définition | Aucun |
| Endpoints documentés | Cités via flux | Tableau partiel | Code source en clair |
| ADR référencés | ADR-003, ADR-007, ADR-011 | Aucun | Aucun |
| Justification des choix | Explicite (verrou pessimiste, délégation auth) | Recopie de techno | Aucune |
| Erreurs documentées | Codes 401/403/409 traçables au code | Codes incomplets (PUT vide) | Inexistant |
| Niveau MVD (Bloc 2) | Conforme | Sous-documenté | Anti-pattern Frankenstein |

---

## 2. Extrait A — Conformité aux standards DCT

### 2.1 Forces

- **Page de garde complète** : titre, version sémantique, date, auteur, statut. Conforme §1 du cours.
- **Diagramme C4 Context** : périmètre clair, acteurs typés (`<<person>>`, `<<system>>`, `<<external_system>>`), protocoles annotés sur chaque flèche (`HTTPS`, `LDAPS`, `CalDAV/TLS`, `HTTPS/REST`). Conforme Bloc 3.
- **Diagramme de séquence** : 12 échanges, pas de raccourci, distinction sync (`->>`) / async (`-->>`, `--)`). Couvre le chemin nominal et renvoie explicitement aux cas d'erreur (§5.2).
- **Maillage croisé avec les ADR** : ADR-003 (délégation auth), ADR-007 (verrou pessimiste), ADR-011 (notification async). Le *pourquoi* est externalisé là où il doit l'être.
- **Glossaire local à la section** : suffisant pour comprendre sans aller chercher ailleurs.
- **Choix techniques justifiés** : la mention `SELECT FOR UPDATE` est nommée, justifiée (concurrence sur créneau) et renvoyée à l'ADR-007.

### 2.2 Faiblesses

- Le glossaire répété en §3.3 risque de diverger avec un éventuel §3 transverse — la responsabilité de la définition doit appartenir à une seule section.
- Le diagramme de contexte ne mentionne pas le canal CDN cité dans le texte introductif (incohérence mineure texte ↔ diagramme).
- Aucune information de versioning d'API ni de fenêtre de rétention sur la séquence : si un dev externe doit l'implémenter, il manque le contrat précis.

### 2.3 Verdict

**Conforme à un niveau professionnel.** Extrait représentatif d'une DCT vivante : diagrammes contextualisés, paragraphes de lecture, ADR croisés, conventions de notation respectées.

---

## 3. Extrait B — Anti-patterns identifiés

### 3.1 Anti-patterns du cours détectés

| Anti-pattern (Bloc 1 / Bloc 3) | Manifestation dans l'extrait |
|---|---|
| **Diagramme PowerPoint** | Boîtes alignées sans direction de flèches, sans protocole, sans interface. Le diagramme ne dit pas qui appelle qui. |
| **Documentation rituelle** | « Le système est conçu pour être scalable horizontalement » sans cible chiffrée, sans composant impacté, sans méthode de vérification. |
| **Diagramme orphelin** | Schéma posé sans paragraphe introductif ni paragraphe de lecture. |
| **Exigence cosmétique** | « Améliorer les performances » sans SLI/SLO. |

### 3.2 Défauts précis

- **Glossaire absent** : `RBAC`, `AMQP`, `SSO`, `JWT` apparaissent sans définition. Un nouvel arrivant est bloqué dès la lecture de la section.
- **Tableau des endpoints incomplet** :
  - `PUT /reservations/{id}` : aucune description, aucun code retour, aucun corps documenté.
  - Colonne `Auth` manquante (impossible de savoir si le endpoint exige `JWT`, `JWT + role:admin`, etc.).
  - Pas de préfixe de version (`/v1/`).
  - Pas de colonne `Dépendances aval` → analyse d'impact impossible.
- **Mécanismes de sécurité narrés en prose** : « Les droits sont vérifiés à chaque appel » — aucune référence à un middleware, à un module, à un test.
- **Section « Performances et scalabilité » non chiffrée** : aucun SLO, aucun budget d'erreur (Bloc 6), aucune méthode de vérification.
- **Confusion entre architecture et inventaire** : `SSO` est représenté comme un composant interne alors que c'est un système tiers (qui devrait apparaître dans le diagramme de contexte C4 L1).

### 3.3 Verdict

**Sous-documenté.** L'extrait donne l'impression qu'une DCT existe, mais aucun élément n'est actionnable. Test du développeur externe (Bloc 2) : un dev compétent ne peut pas implémenter sur cette base sans poser plus de 10 questions.

---

## 4. Extrait C — Anti-patterns majeurs

### 4.1 Anti-patterns du cours détectés

| Anti-pattern | Manifestation |
|---|---|
| **Copier-Coller de Code** (Bloc 1) | Le contrôleur `ReservationController` en Java est intégralement collé. La DCT devient un brouillon de code, pas un document de conception. |
| **DCT Frankenstein** (Bloc 1) | Mélange architecture + tutoriel utilisateur (« allez dans Paramètres > Notifications ») + procédure de déploiement (commandes `mvn`). 3 documents fusionnés. |
| **DCT Fantôme** (Bloc 1) | « Le code est notre doc » assumé par la présence du code dans la DCT. |
| **Incohérence techno** | MySQL est annoncé, puis MongoDB cité plus loin, HikariCP (pool relationnel) sur du NoSQL : contradictions internes non résolues. |
| **Affirmations non étayées** | « Le système est sécurisé car il utilise des tokens », « L'application est conforme RGPD » — aucune base STRIDE, aucun registre de traitements. |

### 4.2 Problèmes critiques

- **Aucune structure formelle** : pas de page de garde, pas de glossaire, pas de §6 ventilée en 4 vues (Bloc 2).
- **Procédures opérationnelles dans la DCT** : `mvn clean package` relève du DEX (Dossier d'Exploitation), pas de la DCT.
- **Manuel utilisateur intercalé** : « Allez dans Paramètres > Notifications » est du contenu SFD (Spécifications Fonctionnelles Détaillées), pas DCT.
- **Code source révélant des défauts** :
  - `userRepository.findByToken(token.substring(7))` : faille d'authentification (token recherché en base, pas validé cryptographiquement). Un audit relèverait l'absence de validation de signature JWT.
  - Annulation sans transaction, sans audit log, sans publication d'événement.
- **Fausses garanties** : « conforme RGPD » sans registre, sans procédure d'oubli, sans marquage RGPD des champs. C'est de la déclaration creuse.

### 4.3 Verdict

**Non conforme.** L'extrait C concentre quatre anti-patterns sur les cinq répertoriés au Bloc 1. Un audit interne ou un onboarding sur cette base est impossible. Ce contenu doit être réparti entre la DCT (architecture), le DEX (déploiement), le manuel utilisateur (interface) et purgé du code source.

---

## 5. Recommandations de réécriture

### 5.1 Pour l'extrait B

| Action | Section concernée |
|---|---|
| Ajouter un diagramme C4 Context avec acteurs et tiers (SSO externe) | §6.1 ou équivalent |
| Reprendre le diagramme de composants avec flèches dirigées + protocoles | §6.1 |
| Compléter le tableau des endpoints (Auth, Codes retour, Dépendances aval, version `/v1/`) | §8 |
| Chiffrer les exigences de performance (p95, débit, méthode de vérification) | §9 |
| Ajouter un glossaire couvrant `RBAC`, `JWT`, `SSO`, `AMQP` | §3 |

### 5.2 Pour l'extrait C

| Action | Destination |
|---|---|
| Retirer le code Java | À supprimer (vit dans le repo source) |
| Extraire la procédure `mvn` | Vers un DEX (`docs/exploitation/`) |
| Extraire la configuration email utilisateur | Vers un guide utilisateur |
| Reconstruire une §6 en 4 vues (logique, processus, déploiement, données) | DCT à reprendre intégralement |
| Trancher la contradiction MySQL/MongoDB par un ADR | `docs/adr/ADR-001-choix-stockage.md` |
| Décrire la sécurité avec STRIDE par flux critique | §9 |

---

## 6. Bilan pédagogique personnel

| Notion du cours | Confirmation par l'exercice |
|---|---|
| 4 anti-patterns du Bloc 1 | Reconnaissables en lecture rapide sur l'extrait C |
| Test du développeur externe (Bloc 2) | Sépare clairement A (réussit) de B et C (échouent) |
| Diagramme orphelin / PowerPoint (Bloc 3) | L'extrait B illustre les deux à la fois |
| ADR comme marqueur de DCT vivante (Bloc 5) | Présents en A, absents en B et C |
| Exigences mesurables vs rituelles (Bloc 6) | L'extrait B confond les deux |

**Conclusion** : une DCT n'est pas longue, elle est **structurée**. L'extrait A tient en 4 pages et reste actionnable. L'extrait C dépasse en volume sans documenter quoi que ce soit.
