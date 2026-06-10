---
domain: code-quality
category: analyse-statique
applicable-to: [all]
tags: [sast, linting, sonarqube, sca, dependency-check, ci, security]
related-knowledge: ../static-analysis.md
ai-selection-criteria: |
  Inclure cette checklist dans tous les audits couvrant la qualité ou la sécurité du code.
  Bonus de priorité +3 si aucun outil d'analyse statique n'est en place.
  +2 si les dépendances ne sont jamais scannées pour les vulnérabilités.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Analyse statique

## Linting et type checking

- [ ] Un linter est configuré et versionné dans le repo
- [ ] Le linter est bloquant en CI (échec = build rouge, pas juste des warnings)
- [ ] Un formatter automatique est configuré (Prettier, Black, gofmt…)
- [ ] Le type checking strict est activé (TypeScript `strict: true`, mypy strict)
- [ ] Les exceptions au linter sont documentées avec justification (`// eslint-disable-next-line — raison`)

## Analyse de sécurité (SAST)

- [ ] Un outil SAST est configuré (SonarQube, CodeQL, Semgrep, Bandit…)
- [ ] L'analyse SAST s'exécute en CI à chaque PR
- [ ] Les findings de sécurité HIGH/CRITICAL bloquent le merge
- [ ] Un Quality Gate est défini (SonarQube) avec seuils documentés
- [ ] Les Security Hotspots sont revus et marqués (pas ignorés par défaut)
- [ ] Les faux positifs sont gérés explicitement (marqués `Won't Fix` avec justification)

## Analyse des dépendances (SCA)

- [ ] Un lockfile est présent et versionné (`package-lock.json`, `poetry.lock`, `go.sum`…)
- [ ] Un outil SCA est configuré (npm audit, pip-audit, Trivy, Snyk, OWASP Dep-Check)
- [ ] Le scan SCA s'exécute en CI
- [ ] Les vulnérabilités HIGH/CRITICAL dans les dépendances bloquent le build
- [ ] Une politique de mise à jour des dépendances est définie (Dependabot, Renovate…)
- [ ] Les licences des dépendances sont compatibles avec l'usage (commercial, open source)

## Métriques de qualité

- [ ] La complexité cyclomatique est mesurée (SonarQube, Lizard…)
- [ ] Le taux de duplication est mesuré et < 5%
- [ ] La dette technique est mesurée et visible (SonarQube grade ou équivalent)
- [ ] Les métriques évoluent dans le bon sens (trend suivi sur les derniers mois)

## Score d'évaluation

Compter le nombre de cases cochées :

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
