---
domain: code-quality
category: tests
severity: high
applicable-to: [all]
tags: [tests, coverage, tdd, unit, integration, e2e, mutation, ci]
related:
  - ./code-review.md
  - ./checklists/testing-checklist.md
ai-prompt-hints: |
  Demander : taux de coverage actuel, outil de coverage utilisé, dernière fois
  qu'un bug de prod aurait été attrapé par les tests.
  Chercher dans le repo : répertoire de tests, configuration CI (étape de test),
  présence de tests d'intégration vs seulement unitaires, mocks excessifs.
  Red flags : coverage < 40%, tests uniquement sur les happy paths, aucun test
  d'intégration, tests qui ne testent pas vraiment le code (assertions triviales),
  suite de tests qui met > 10 min à tourner.
---

# Stratégie de tests

## Pyramide de tests — niveaux et cibles

| Niveau | Cible coverage | Vitesse | Coût | Ce qu'il détecte |
|--------|---------------|---------|------|-----------------|
| **Unitaires** | 70-90% de la logique métier | < 100ms/test | Faible | Logique pure, edge cases, régressions |
| **Intégration** | Flux critiques + contrats d'API | < 5s/test | Moyen | Interactions entre composants, DB, services |
| **E2E / fonctionnels** | Parcours utilisateur critiques (5-10) | < 2 min total | Élevé | Régressions UX, intégrations tierces |
| **Mutation** | Sur les modules critiques | Lent (CI nightly) | Élevé | Qualité des tests eux-mêmes |

## Meilleures pratiques

### Qualité des tests unitaires

- **Pattern AAA** : Arrange (setup) / Act (appel) / Assert (vérification) — une seule assertion logique par test
- **FIRST** : Fast, Independent, Repeatable, Self-validating, Timely
- Nommage explicite : `test_calculate_total_with_discount_applies_percentage_correctly`
- Pas de logique conditionnelle dans les tests (pas de `if`, pas de boucles)
- Un test qui ne peut pas échouer est inutile — vérifier avec mutation testing

### Mocks et test doubles

- Mocker uniquement les dépendances **externes** (HTTP, DB, filesystem, horloge)
- Ne pas mocker les classes internes du domaine — si difficile à tester sans mock, c'est un signal de couplage
- Préférer les **fakes** (implémentation in-memory) aux mocks pour les repositories
- Risque : over-mocking donne un coverage élevé mais ne teste pas la vraie intégration

```python
# Mauvais : mock de la logique interne
mock_user_service.get_user.return_value = User(id=1)

# Mieux : fake repository en mémoire
class InMemoryUserRepository:
    def __init__(self): self.users = {}
    def find_by_id(self, id): return self.users.get(id)
```

### Tests d'intégration

- Tester contre les vraies dépendances (base de données de test, pas de mock DB)
- Utiliser **testcontainers** pour les bases de données en CI (PostgreSQL, Redis, Kafka)
- Tester les contrats d'API avec des **consumer-driven contract tests** (Pact) pour les microservices
- Réinitialiser l'état entre les tests (transactions rollback, fixtures, truncate)

### Coverage — interprétation

| Seuil | Signification |
|-------|---------------|
| > 80% | Bonne couverture — vérifier la qualité, pas seulement la quantité |
| 60-80% | Acceptable — identifier les chemins non couverts (chemins d'erreur, edge cases) |
| 40-60% | Risqué — les régressions passent souvent inaperçues |
| < 40% | Critique — le CI ne protège pas contre les régressions |

Le coverage de lignes est une métrique faible — préférer le **branch coverage** et le **mutation score**.

### Mutation testing

Le mutation testing modifie le code (inverse les conditions, change les opérateurs) et vérifie que les tests échouent. Un **mutation score** > 70% indique des tests efficaces.

```bash
# Python
mutmut run && mutmut results

# Java
mvn org.pitest:pitest-maven:mutationCoverage

# JavaScript
npx stryker run
```

## Ce que l'agent IA peut vérifier

### Analyse de la structure de tests

```bash
# Ratio code / tests
find src -name "*.ts" -o -name "*.py" | wc -l
find tests -name "*.test.*" -o -name "*.spec.*" -o -name "test_*.py" | wc -l

# Présence de tests d'intégration vs unitaires
find . -path "*/tests/integration*" -o -path "*/tests/e2e*" | head -10

# Tests qui ne contiennent pas d'assertions (tests vides)
grep -rn "def test_\|it(\|test(" --include="*.py" --include="*.ts" -l | \
  xargs grep -L "assert\|expect\|should"

# Mocks excessifs dans les tests unitaires
grep -rn "mock\|Mock\|MagicMock\|jest.fn\|sinon" --include="*.py" --include="*.ts" | wc -l

# Configuration CI pour les tests
cat .github/workflows/*.yml .gitlab-ci.yml 2>/dev/null | grep -A5 "test\|coverage"
```

### Questions d'audit (entretien)

1. Quel est le taux de coverage actuel ? Est-il mesuré en CI ?
2. Y a-t-il des tests d'intégration qui tournent contre une vraie base de données ?
3. Les tests passent-ils avant chaque merge ? Bloquent-ils le merge en cas d'échec ?
4. Quel est le temps d'exécution de la suite complète de tests ?
5. Le dernier bug de production aurait-il été détecté par les tests existants ?
6. Y a-t-il des tests de régression systématiques sur les bugs corrigés ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Rapport de coverage | JaCoCo (Java), Istanbul/c8 (JS), pytest-cov (Python) | Analyser le rapport XML/JSON, identifier les modules non couverts |
| Mutation testing | Pitest, Stryker, Mutmut | Analyser le rapport de mutations survivantes |
| Tests de contrat | Pact | Lire les contrats et identifier les incompatibilités |
| Analyse de la qualité des tests | SonarQube (test smells) | Analyser les métriques exportées |

## Références

- [Test Pyramid — Martin Fowler](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Testcontainers](https://testcontainers.com/)
- [Pact — Consumer-Driven Contract Testing](https://docs.pact.io/)
- [Mutation Testing — Stryker](https://stryker-mutator.io/)
- [Google Testing Blog](https://testing.googleblog.com/)
