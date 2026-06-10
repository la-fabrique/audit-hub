---
domain: infra-devops
category: ci-cd
applicable-to: [all]
tags: [ci-cd, pipeline, secrets, docker, supply-chain, github-actions, gitlab-ci]
related-knowledge: ../cicd-security.md
ai-selection-criteria: |
  Inclure si l'organisation a des pipelines CI/CD (quasi-universel). Bonus de
  priorité +3 si les pipelines déploient directement en production. +2 si des
  secrets sont injectés dans les pipelines.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — CI/CD et sécurité des pipelines

## Gestion des secrets

- [ ] `[B]` Aucun secret, token ou mot de passe n'est présent dans les fichiers de pipeline ou le code
- [ ] Les secrets sont stockés dans un gestionnaire dédié (GitHub Secrets, GitLab CI Variables, Vault, AWS Secrets Manager)
- [ ] Les secrets sont scopés par environnement (secret prod inaccessible en staging)
- [ ] Les secrets ne sont pas affichés dans les logs CI (masqués)
- [ ] Les secrets de long terme ont une politique de rotation

## Permissions et accès

- [ ] Les pipelines appliquent le principe du moindre privilège
- [ ] Les workflows GitHub Actions déclarent des permissions minimales (`permissions: contents: read`)
- [ ] Les actions tierces sont pinnées sur un SHA ou une version fixe (pas `@main`)
- [ ] Les déploiements en production nécessitent une approbation manuelle
- [ ] Les personnes pouvant déclencher un déploiement prod sont identifiées et limitées

## Build et artefacts

- [ ] Les images Docker utilisent une version fixée (pas `:latest`)
- [ ] Les images sont scannées pour les CVE avant déploiement (Trivy, Snyk)
- [ ] Les dépendances sont auditées en CI (npm audit, pip-audit, trivy fs)
- [ ] Un SBOM est généré à chaque build (optionnel mais recommandé)
- [ ] Les artefacts de build sont signés ou vérifiés (checksums)

## Qualité du pipeline

- [ ] Les tests passent avant tout déploiement
- [ ] Le linter et l'analyse statique sont bloquants
- [ ] Les déploiements sont automatiquement annulés en cas d'échec des health checks
- [ ] Un mécanisme de rollback est documenté et testé
- [ ] Les DORA metrics sont suivis (deploy frequency, lead time, MTTR, change failure rate)

## Infrastructure as Code

- [ ] La configuration IaC est versionnée dans le repo
- [ ] Un `terraform plan` (ou équivalent) est exécuté et révisé avant chaque `apply`
- [ ] Le state Terraform est distant et verrouillé (pas local)
- [ ] Les configs IaC sont scannées en CI (Checkov, tfsec, tflint)
- [ ] Les changements manuels hors IaC sont détectés (drift detection)

## Score d'évaluation

Les items `[B]` sont éliminatoires : si un seul est en échec, le score est
plafonné à 2.

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
