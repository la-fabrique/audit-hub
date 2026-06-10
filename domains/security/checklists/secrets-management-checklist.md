---
domain: security
category: secrets-management
applicable-to: [web, api, mobile, saas, infra]
tags: [secrets, credentials, env-vars, vault, rotation, cicd, gitignore]
related-knowledge: ../secrets-management.md
ai-selection-criteria: |
  Inclure dans tout audit couvrant le périmètre sécurité. Bonus de priorité +3
  si présence d'un pipeline CI/CD avec accès à la production. +2 si l'organisation
  a déjà eu un incident de fuite de credentials. Toujours vérifier en priorité si
  `access.code_repo` est true (vérification automatique possible).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Gestion des secrets

## Stockage des secrets

- [ ] `[B]` Aucun secret (mot de passe, clé API, token) n'est présent dans le code source
- [ ] `[B]` Aucun fichier `.env` avec des secrets réels n'est commité dans git (y compris l'historique)
- [ ] Un vault dédié est utilisé en production (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager…)
- [ ] Les secrets de production et de développement sont distincts

## Contrôle git

- [ ] `.gitignore` contient `.env`, `.env.*`, `*.pem`, `*.key`
- [ ] Un hook pre-commit anti-secrets est en place (truffleHog, git-secrets, detect-secrets)
- [ ] L'historique git a été audité pour détecter des secrets anciennement commités

## Secrets CI/CD

- [ ] Les secrets CI/CD sont stockés dans le secret store du pipeline (GitHub Secrets, GitLab CI Variables…)
- [ ] Les secrets de production ne sont accessibles qu'aux pipelines de production
- [ ] Les secrets n'apparaissent pas dans les logs de build (masqués)
- [ ] La liste des secrets CI/CD est revue régulièrement (secrets inactifs supprimés)

## Rotation

- [ ] Une politique de rotation est définie et appliquée (≤ 90 jours pour les API keys sans rotation auto)
- [ ] La rotation est automatisée pour les credentials de bases de données
- [ ] Les secrets sont immédiatement révoqués après un départ d'employé ou un incident
- [ ] Les clés de chiffrement suivent une politique de rotation distincte

## Principe du moindre privilège

- [ ] Chaque service possède ses propres credentials (pas de credentials partagés)
- [ ] Les API keys sont scopées au minimum requis (read-only si possible)
- [ ] Les comptes de service cloud ont des policies IAM minimales

## Score d'évaluation

Compter le nombre de cases cochées. Les items `[B]` sont éliminatoires :
si un seul est en échec, le score est plafonné à 2.

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
