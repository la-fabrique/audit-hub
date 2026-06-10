---
domain: code-quality
category: review-process
applicable-to: [all]
tags: [code-review, pr, conventions, tests, linting]
related-knowledge: ../code-review.md
ai-selection-criteria: |
  Inclure cette checklist dans tous les audits couvrant la qualité du code.
  Bonus de priorité +2 si l'équipe a des régressions fréquentes en production.
  +2 si les PRs sont régulièrement > 800 lignes ou mergées sans review.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Processus de revue de code

## Process et gouvernance des PRs

- [ ] Chaque PR nécessite au moins 1 approbation avant merge
- [ ] Les merges directs sur la branche principale sont bloqués (branch protection)
- [ ] Les PRs ont une description minimale (contexte, impact, comment tester)
- [ ] La taille typique des PRs est < 400 lignes (hors fichiers générés)
- [ ] Les reviews se font en moins de 24h (défini dans les accords d'équipe)
- [ ] Les PRs de hotfix ont un processus d'approbation accéléré documenté

## Linting et formatting

- [ ] Un linter est configuré (ESLint, Pylint, Checkstyle…)
- [ ] Un formatter est configuré (Prettier, Black, gofmt…)
- [ ] Le linter est **bloquant en CI** (pas seulement advisory)
- [ ] Le formatter est appliqué automatiquement (pre-commit hook ou CI)
- [ ] La configuration du linter est versionnée dans le repo
- [ ] Les exceptions (`eslint-disable`, `noqa`) sont documentées avec justification

## Conventions de code

- [ ] Une convention de nommage est définie et appliquée
- [ ] Les fonctions ont une responsabilité unique (< 30 lignes idéalement)
- [ ] Les nombres magiques et chaînes hardcodées sont remplacés par des constantes
- [ ] Les commentaires expliquent le "pourquoi", pas le "quoi"
- [ ] Pas de code mort commenté dans les PRs mergées

## Tests dans les PRs

- [ ] Toute nouvelle fonctionnalité inclut des tests
- [ ] Tout bug corrigé inclut un test de régression
- [ ] Les tests passent en CI avant le merge
- [ ] Le coverage ne régresse pas entre la branche de base et la PR
- [ ] Les tests couvrent les cas nominaux ET les cas d'erreur

## Sécurité

- [ ] Les entrées utilisateur sont validées dans les nouvelles fonctions
- [ ] `[B]` Aucun secret, token ou credential n'est introduit dans le code
- [ ] Les nouvelles dépendances ont été vérifiées (origine, licence, vulnérabilités)
- [ ] Les modifications de logique d'autorisation sont revues avec attention particulière

## Score d'évaluation

Compter le nombre de cases cochées :

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
