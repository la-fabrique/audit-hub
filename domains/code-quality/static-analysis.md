---
domain: code-quality
category: analyse-statique
severity: high
applicable-to: [all]
tags: [sast, linting, sonarqube, codeql, dependency-check, sca, formatter, ci]
related:
  - ./technical-debt.md
  - ./code-review.md
  - ./checklists/static-analysis-checklist.md
ai-prompt-hints: |
  Demander : quels outils d'analyse statique sont en place, sont-ils bloquants en CI.
  Chercher dans le repo : configuration SonarQube (sonar-project.properties), ESLint
  (.eslintrc), configuration CI avec étapes d'analyse, rapport de dette actuel.
  Red flags : linter non bloquant (advisory seulement), pas d'analyse statique de
  sécurité, dépendances jamais scannées, SonarQube installé mais non utilisé en CI.
---

# Analyse statique

## Types d'analyse statique

| Type | Objectif | Exemples d'outils |
|------|----------|-------------------|
| **Linting** | Conventions de code, erreurs évidentes | ESLint, Pylint, Checkstyle, RuboCop |
| **Formatting** | Style et indentation | Prettier, Black, gofmt, rustfmt |
| **SAST** | Vulnérabilités de sécurité dans le code | SonarQube, CodeQL, Semgrep, Bandit |
| **SCA** | Vulnérabilités dans les dépendances tierces | OWASP Dependency-Check, Snyk, npm audit |
| **Complexité** | Métriques de maintenabilité | Lizard, Radon, SonarQube |
| **Type checking** | Cohérence de types | mypy, TypeScript strict, Flow |

## Outils — comparatif

| Outil | Type | Langages | Open source | Intégration CI |
|-------|------|----------|-------------|----------------|
| **SonarQube Community** | SAST + qualité | 30+ | Oui | Natif |
| **GitHub CodeQL** | SAST | 10+ | Oui (sur GitHub) | GitHub Actions |
| **Semgrep** | SAST | 30+ | Oui | Natif |
| **Bandit** | SAST | Python | Oui | Simple |
| **ESLint** | Lint + sécurité | JS/TS | Oui | Trivial |
| **Trivy** | SCA + IaC | Multi | Oui | Natif |
| **OWASP Dep-Check** | SCA | Java, JS, .NET | Oui | Plugin Maven/Gradle |
| **Snyk** | SCA + SAST | Multi | Freemium | Natif |

## Meilleures pratiques

### Configuration minimale recommandée

Un pipeline CI minimal doit inclure dans cet ordre :

```yaml
# Exemple GitHub Actions
- name: Lint
  run: npx eslint . --max-warnings 0   # 0 warning toléré

- name: Type check
  run: npx tsc --noEmit

- name: Security scan (SAST)
  run: npx semgrep --config=p/owasp-top-ten --error

- name: Dependency audit
  run: npm audit --audit-level=high    # bloquant sur HIGH et CRITICAL
```

### SonarQube — Quality Gates

Un Quality Gate définit les seuils de blocage. Configuration recommandée :

| Métrique | Seuil bloquant |
|----------|---------------|
| Nouvelles vulnérabilités de sécurité | 0 |
| Nouvelles issues critiques | 0 |
| Nouveau code sans tests | Coverage < 80% |
| Duplication nouveau code | > 3% |
| Security Hotspots reviewed | 100% |

Le Quality Gate doit être **bloquant en CI** — un build qui échoue le gate ne peut pas être mergé.

### Analyse des dépendances (SCA)

```bash
# JavaScript / Node.js
npm audit --json | jq '.vulnerabilities | to_entries[] | select(.value.severity == "high" or .value.severity == "critical")'

# Python
pip-audit --format json --output audit.json

# Java (Maven)
mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7

# Multi-langage (recommandé en CI)
trivy fs --exit-code 1 --severity HIGH,CRITICAL .
```

### Faux positifs — gestion

Les faux positifs sont inévitables ; leur gestion doit être explicite :

- **SonarQube** : marquer `Won't Fix` avec justification (traçable)
- **Semgrep / ESLint** : commentaire `// nosemgrep: rule-id` avec explication
- **npm audit** : `npm audit --ignore-registry-data` ou `.nsprc` pour exceptions documentées
- Revue périodique des exceptions (au moins trimestrielle)

## Ce que l'agent IA peut vérifier

### Présence et configuration des outils

```bash
# Vérifier les configs d'analyse statique
ls .eslintrc* .eslintignore sonar-project.properties .semgreprc 2>/dev/null
find . -name ".pylintrc" -o -name "pyproject.toml" -o -name "setup.cfg" | head -5

# Vérifier l'intégration CI
grep -rn "sonar\|semgrep\|bandit\|eslint\|pylint\|trivy\|snyk" .github/workflows/ .gitlab-ci.yml 2>/dev/null

# Configuration bloquante ou advisory ?
grep -n "continue-on-error\|allow_failure\|--max-warnings" .github/workflows/*.yml .gitlab-ci.yml 2>/dev/null

# Type checking strict
grep -n "strict\|noImplicitAny\|strict-typing" tsconfig.json mypy.ini pyproject.toml 2>/dev/null

# Lockfile présent (SCA possible)
ls package-lock.json yarn.lock poetry.lock Pipfile.lock go.sum Cargo.lock 2>/dev/null
```

### Questions d'audit (entretien)

1. Quels outils d'analyse statique sont en place ? Sont-ils bloquants en CI ?
2. Comment les vulnérabilités dans les dépendances sont-elles détectées et corrigées ?
3. Y a-t-il un Quality Gate configuré (SonarQube ou équivalent) ?
4. Comment les faux positifs sont-ils gérés ? Existe-t-il un processus de revue ?
5. La dernière CVE critique dans les dépendances a-t-elle été corrigée en combien de temps ?
6. Le type checking strict est-il activé pour les langages typés (TypeScript, mypy) ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Rapport SAST complet | SonarQube, CodeQL, Semgrep | Analyser le rapport JSON/SARIF, prioriser les findings par sévérité |
| Rapport SCA | Trivy, Snyk, OWASP Dep-Check | Contextualiser les CVE, identifier les composants exposés en production |
| Métriques de complexité | SonarQube, Lizard | Analyser les modules au-dessus des seuils |
| Rapport SARIF (standard) | Tout outil compatible | Lire le fichier SARIF et synthétiser les findings |

## Références

- [OWASP Source Code Analysis Tools](https://owasp.org/www-community/Source_Code_Analysis_Tools)
- [Semgrep Rules — OWASP Top 10](https://semgrep.dev/p/owasp-top-ten)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [SonarQube Quality Gates](https://docs.sonarsource.com/sonarqube/latest/user-guide/quality-gates/)
- [NIST NVD — CVE Database](https://nvd.nist.gov/)
