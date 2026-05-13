# Validation du pipeline CI documentaire

Ce fichier atteste du fonctionnement de la CI mise en place au TP 1.7.

## Objet

Vérifier qu'une pull request modifiant un fichier `.md` du dépôt :

1. Déclenche automatiquement le workflow `.github/workflows/docs-quality.yml`.
2. Échoue en présence de défauts (lien mort, syntaxe Markdown invalide, bloc Mermaid mal formé).
3. Passe au vert une fois les défauts corrigés.
4. Assigne automatiquement comme reviewers les `CODEOWNERS` du fichier modifié.

## Méthode

Pull request de test : `docs/ci/test-broken-link` (branche dédiée, jamais fusionnée sur `main`).

Trois scénarios injectés dans un fichier `docs/_scratch/ci-test.md` :

| Scénario injecté | Job CI attendu | Résultat |
|---|---|---|
| Lien interne mort : `[lien KO](../inexistant.md)` | `Check links` échoue | ✅ Détecté |
| Bloc Mermaid invalide : `graph TD\n  A --> B unclosed (` | `Validate Mermaid blocks` échoue | ✅ Détecté |
| Faute Markdown lint (titre `#` sans saut de ligne) | `Lint Markdown` échoue | ✅ Détecté |

Après correction des trois défauts, l'ensemble des jobs repasse au vert. La PR de test est ensuite fermée sans merge.

## Reviewers auto-assignés

Modification testée sur :

- `docs/07-conception-detaillee/07-modules.md` → reviewer auto : `@nicolas` (cf. `.github/CODEOWNERS`)
- `docs/03-glossaire.md` → reviewer auto : `@alexis`
- `docs/adr/ADR-001.md` → reviewer auto : `@nicolas`

L'auto-assignation fonctionne dès que la branche `main` est configurée avec la protection « Require review from Code Owners ».

## Liens

- Workflow : [`.github/workflows/docs-quality.yml`](../.github/workflows/docs-quality.yml)
- Config lint : [`.markdownlint.json`](../.markdownlint.json)
- CODEOWNERS : [`.github/CODEOWNERS`](../.github/CODEOWNERS)

## Note opérationnelle

Le workflow utilise `actions/checkout@v4` et `actions/setup-node@v4` avec Node 20. Les trois outils (`markdownlint-cli`, `markdown-link-check`, `@mermaid-js/mermaid-cli`) sont installés à chaque exécution — la durée typique du job est de 1 min 30 s, acceptable pour un linter documentaire.

Pour les liens externes, le workflow tolère un timeout de 5 s et accepte les codes `403` (anti-bot fréquent sur certains sites) pour limiter les faux positifs. Les liens internes morts, eux, font échouer le job sans exception.
