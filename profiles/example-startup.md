---
name: "Acme SaaS"
type: startup
date: "2026-01-15"
auditor: "Dimitri Rayer"

sector: saas
regulatory:
  - RGPD
data_sensitivity: confidential
users:
  count: 10k
  type: B2B

team:
  size: 6-15
  structure: fullstack
  seniority: mixed
  distributed: remote-france

stack:
  frontend: [React, TypeScript]
  backend: [Node.js, TypeScript]
  database: [PostgreSQL, Redis]
  infra: [AWS]
  deployment: [Docker, ECS]
  languages: [TypeScript]

maturity:
  cicd: partial
  testing: some
  monitoring: logs-only
  security_practices: basic
  documentation: partial

audit:
  scope:
    - security
    - architecture
    - infra-devops
    - organizational
  trigger: levée-fonds
  priority_concerns:
    - "Préparer une due diligence technique pour la Série A"
    - "Identifier les risques de sécurité avant l'audit investisseur"
  excluded:
    - code-quality
  depth: standard

access:
  code_repo: true
  ci_cd: true
  infra_config: true
  monitoring: false
  interviews: true
---

# Notes contextuelles

## Contexte
Startup B2B SaaS en phase de croissance, 50 clients entreprises, levée de fonds Série A
dans 3 mois. L'équipe technique est de 8 personnes (4 devs fullstack, 1 DevOps, 1 lead tech,
1 product, 1 data).

## Points d'attention spécifiques
- L'infrastructure a été construite vite pendant la phase early — dette technique probable
- Le lead tech est seul à connaître l'infra complète (bus factor critique)
- Pas de SOC2 ou ISO27001 mais les clients entreprise commencent à demander des preuves de sécurité

## Interlocuteurs
- CTO : point d'entrée principal, connaît l'architecture globale
- Lead DevOps : pour tout ce qui est infra et CI/CD
- Lead Dev : pour la qualité de code et les processus de review
