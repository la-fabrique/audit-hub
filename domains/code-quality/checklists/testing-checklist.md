---
domain: code-quality
category: tests
applicable-to: [all]
tags: [tests, coverage, unit, integration, e2e, ci, mutation]
related-knowledge: ../testing-strategy.md
ai-selection-criteria: |
  Inclure cette checklist dans tous les audits couvrant la qualité du code.
  Bonus de priorité +3 si des bugs de production ont récemment échappé aux tests.
  +2 si le coverage est inconnu ou < 50%.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Stratégie de tests

## Infrastructure de test

- [ ] Un framework de test est configuré (Jest, pytest, JUnit, RSpec…)
- [ ] Les tests s'exécutent en CI à chaque PR
- [ ] Les tests bloquent le merge en cas d'échec
- [ ] Le rapport de coverage est produit en CI
- [ ] Un seuil de coverage minimum est défini et bloquant (ex : 70%)

## Tests unitaires

- [ ] Les fonctions/méthodes de logique métier ont des tests unitaires
- [ ] Les tests suivent le pattern AAA (Arrange / Act / Assert)
- [ ] Chaque test a une seule assertion logique principale
- [ ] Les tests sont indépendants (pas d'ordre d'exécution requis)
- [ ] Les tests s'exécutent en < 100ms chacun
- [ ] Les cas d'erreur et edge cases sont couverts (pas seulement le happy path)

## Tests d'intégration

- [ ] Les interactions avec la base de données sont testées (contre une vraie DB ou testcontainer)
- [ ] Les endpoints API critiques ont des tests d'intégration
- [ ] L'état est réinitialisé entre les tests (rollback, fixtures)
- [ ] Les contrats d'API avec les services tiers sont testés (consumer-driven ou mocks documentés)

## Tests E2E (si applicable)

- [ ] Les parcours utilisateur critiques ont des tests E2E (connexion, achat, etc.)
- [ ] Les tests E2E s'exécutent en CI (au moins sur la branche principale)
- [ ] Les tests E2E ont un temps d'exécution tolérable (< 15 min)
- [ ] Les tests E2E ne sont pas le seul filet de sécurité (pyramide respectée)

## Qualité des tests

- [ ] Les tests ne contiennent pas de logique conditionnelle (pas de `if` dans les tests)
- [ ] Les mocks ne remplacent pas les dépendances internes du domaine
- [ ] Les tests de régression existent pour les bugs corrigés en production
- [ ] Les tests sont lisibles sans commenter (nommage explicite)

## Mutation testing (optionnel mais recommandé pour le code critique)

- [ ] Un outil de mutation testing est disponible (Pitest, Stryker, Mutmut)
- [ ] Le mutation score des modules critiques est > 70%
- [ ] Les mutations survivantes sont revues et traitées

## Score d'évaluation

Compter le nombre de cases cochées :

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
