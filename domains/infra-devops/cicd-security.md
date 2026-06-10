---
domain: infra-devops
category: ci-cd
severity: high
applicable-to: [all]
tags: [ci-cd, pipeline, secrets, iac, github-actions, gitlab-ci, docker, supply-chain]
related:
  - ./observability.md
  - ./checklists/cicd-checklist.md
ai-prompt-hints: |
  Demander accès à : fichiers de pipeline CI/CD (.github/workflows, .gitlab-ci.yml),
  configuration des environnements (staging, prod), gestion des secrets (vault, env vars),
  Dockerfiles, scripts IaC (Terraform, Ansible).
  Chercher : secrets exposés dans les logs, permissions excessives dans les pipelines,
  images Docker sans version fixée, IaC sans state distant, absence de scan de vulnérabilités.
  Questions clés : "Comment les secrets de prod sont-ils injectés en CI ?" et
  "Qui peut déclencher un déploiement en production ?"
---

# CI/CD et sécurité des pipelines

## Risques critiques

| Risque | Impact | Fréquence |
|---|---|---|
| Secrets dans les logs CI/CD | Critique | Élevée |
| Permissions excessives pipeline (admin sur tout) | Critique | Élevée |
| Images Docker sans version fixée (`:latest`) | Haut | Très élevée |
| Pas de scan de vulnérabilités des dépendances | Haut | Élevée |
| IaC sans validation automatique avant apply | Haut | Moyenne |
| Déploiements prod déclenchables sans approbation | Critique | Moyenne |

## Meilleures pratiques

### Gestion des secrets
- Jamais de secrets dans le code ou les logs — utiliser **Vault**, AWS Secrets Manager, GitHub Secrets
- **Rotation automatique** des secrets de long terme
- Secrets scopés par environnement : un secret prod n'est pas accessible en staging
- Audit régulier des accès : qui a accès à quoi

### Pipeline sécurisé
- **Principe du moindre privilège** : le pipeline n'a que les droits dont il a besoin
- Validation des PRs avant tout merge (branch protection rules)
- **Approbation manuelle** obligatoire pour les déploiements en production
- Séparation des pipelines CI (test/build) et CD (déploiement)
- Pinning des actions tierces (GitHub Actions) : `uses: actions/checkout@v4.1.0` pas `@main`

### Docker et supply chain
- Images de base : distroless ou alpine, jamais `latest`
- Scanner les images pour les CVE : **Trivy**, Snyk, Grype en CI
- Multi-stage builds pour minimiser la surface d'attaque
- Scanner les dépendances : Dependabot, `npm audit`, `pip-audit`, `cargo audit`
- SBOM (Software Bill of Materials) généré à chaque build

### Infrastructure as Code
- State distant sécurisé (S3 + DynamoDB lock, Terraform Cloud)
- `plan` automatique sur PR, `apply` uniquement après approbation
- Linting IaC en CI : **tflint**, **checkov**, **trivy** pour les configs IaC
- Drift detection : détecter les changements manuels hors IaC

### Observabilité du pipeline
- Métriques de pipeline : durée, taux d'échec, DORA metrics
- Alertes sur les échecs de déploiement en prod
- Traçabilité : chaque déploiement prod doit être lié à un commit et un auteur

## Ce que l'agent IA peut vérifier

### Analyse des pipelines et du dépôt

```bash
# Chercher des secrets potentiels dans l'historique git
git log --all --oneline | xargs -I {} git show {} | grep -iE "password|secret|token|api_key" | head -20

# Images sans version fixée dans les Dockerfiles
grep -r "FROM.*:latest\|FROM.*[^:]$" --include="Dockerfile*"

# Actions GitHub sans version pinned
grep -rE "uses: .+@(main|master|HEAD)" .github/

# Vérifier les permissions des workflows GitHub Actions
grep -A5 "permissions:" .github/workflows/*.yml
```

### Revue de configuration

- Lire les workflows CI/CD : séparation CI (test/build) et CD (déploiement), environnements protégés, approbation manuelle pour la prod
- Vérifier la présence de scans dans les pipelines (Trivy, Dependabot, `npm audit`, checkov…)
- Examiner la configuration IaC : state distant, `plan` automatique sur PR, `apply` après approbation
- Vérifier les branch protection rules (si `gh` CLI disponible : `gh api repos/{owner}/{repo}/branches/main/protection`)

### Questions d'audit (entretien)

1. Comment les secrets sont-ils injectés dans les pipelines ? Sont-ils dans les logs ?
2. Qui peut déclencher un déploiement en production ? Y a-t-il une approbation ?
3. Les images Docker utilisent-elles des versions fixées ou `:latest` ?
4. Y a-t-il un scan de CVE dans le pipeline ? Sur les dépendances ? Sur les images ?
5. L'IaC est-elle validée automatiquement avant application ?
6. Quels sont les DORA metrics actuels (deploy frequency, lead time, MTTR, change failure rate) ?

## DORA Metrics — niveaux de référence

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| Deploy frequency | Plusieurs/jour | 1/semaine-1/jour | 1/mois-1/semaine | < 1/mois |
| Lead time | < 1h | 1j-1 sem | 1 sem-1 mois | > 1 mois |
| MTTR | < 1h | < 24h | 1j-1 sem | > 1 sem |
| Change failure rate | 0-5% | 5-10% | 10-15% | > 15% |

## Références

- [OWASP CI/CD Security Top 10](https://owasp.org/www-project-top-10-ci-cd-security-risks/)
- [DORA Research](https://dora.dev/research/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
