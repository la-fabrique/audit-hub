---
domain: code-quality
category: review-process
severity: high
applicable-to: [all]
tags: [code-review, pr, tests, coverage, linting, dette-technique, conventions]
related:
  - ./testing-strategy.md
  - ./checklists/code-review-checklist.md
ai-prompt-hints: |
  Demander accès à : dépôt git, historique des PRs, configuration linter/formatter,
  métriques de coverage, rapport de dette technique (SonarQube, CodeClimate).
  Observer : taille moyenne des PRs, ratio commentaires/approbations, temps moyen
  de review, présence de tests dans les PRs, convention de nommage dans le code.
  Questions clés : "Combien de PRs sont mergées sans review ?" et
  "Quel % du code a des tests automatisés ?"
---

# Qualité de code et processus de revue

## Indicateurs de santé

| Indicateur | Cible | Alerte |
|---|---|---|
| Coverage de tests | > 70% (critique > 90%) | < 40% |
| Taille moyenne des PRs | < 400 lignes | > 800 lignes |
| Délai moyen de review | < 24h | > 72h |
| PRs mergées sans review | 0% | > 10% |
| Dette technique (SonarQube) | Grade A ou B | Grade D ou E |
| Linter/formatter configuré | Oui, en CI | Non |

## Meilleures pratiques

### Process de revue de code
- **Chaque PR nécessite au moins 1 reviewer** — pas d'auto-merge sans approbation
- **PRs petites et atomiques** : une PR = un changement logique, < 400 lignes idéalement
- **Description de PR** : contexte, décision technique, impact, comment tester
- **Review dans les 24h** pour ne pas bloquer le flow
- Reviewer en priorité : logique métier, sécurité, performance, puis style

### Tests
- **AAA pattern** : Arrange / Act / Assert — une seule assertion logique par test
- **Tests unitaires** pour la logique pure, **tests d'intégration** pour les interactions
- **Pas de logique dans les tests** : les tests doivent être lisibles comme de la documentation
- Tests de régression systématiques sur les bugs corrigés
- Mutation testing pour valider l'efficacité des tests (Pitest, Mutmut)

### Conventions et lisibilité
- Linter + formatter configurés et **bloquants en CI** (ne pas juste avertir)
- Nommage explicite : préférer `isUserAuthenticated()` à `checkUser()`
- Fonctions courtes (< 20 lignes idéalement) avec responsabilité unique
- Pas de nombres magiques ni de chaînes hardcodées : utiliser des constantes nommées
- Commentaires uniquement pour le "pourquoi", jamais pour le "quoi"

### Dette technique
- **Identifier et tagger** la dette (TODO/FIXME avec date et ticket)
- Budget de réduction : au moins 20% du temps de dev sur la dette
- Refactoring en continu > big bang refactoring
- Métriques trackées : complexité cyclomatique, duplication, coupling

## Ce que l'agent IA peut vérifier

### Red flags dans l'historique git

```bash
# PRs trop grosses
git log --oneline --stat | grep "files changed" | awk '{print $1}' | sort -n

# Commits directs sur main (bypasse les reviews)
git log main --no-merges --author=".*" --format="%an: %s" | head -20

# Fichiers jamais touchés depuis longtemps (dette potentielle)
git log --diff-filter=M --name-only --format="" | sort | uniq -c | sort -n | head -20
```

### Analyse de la configuration

```bash
# Linter / formatter configurés
ls .eslintrc* eslint.config.* .prettierrc* ruff.toml .rubocop.yml .golangci.yml 2>/dev/null

# Bloquants en CI ? (présents dans les workflows, pas seulement en local)
grep -rn "lint\|eslint\|ruff\|flake8\|rubocop" .github/workflows/ .gitlab-ci.yml 2>/dev/null

# Seuil de coverage imposé
grep -rn "coverageThreshold\|fail_under\|cov-fail-under" --include="*.json" --include="*.toml" --include="*.cfg" --include="*.yml"

# Protection de branche (si gh CLI disponible)
gh api repos/{owner}/{repo}/branches/main/protection
```

### Questions d'audit (entretien)

1. Quelle est la politique de revue de code ? Est-elle documentée et respectée ?
2. Y a-t-il des tests automatisés ? Quel est le taux de coverage ?
3. Les tests passent-ils en CI avant chaque merge ?
4. Y a-t-il un linter/formatter configuré ? Est-il bloquant ou advisory ?
5. Comment la dette technique est-elle trackée et priorisée ?
6. Quelle est la taille typique d'une PR ? Combien de reviewers ?

## Références

- [Google Engineering Practices — Code Review](https://google.github.io/eng-practices/review/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Test Pyramid — Martin Fowler](https://martinfowler.com/articles/practical-test-pyramid.html)
