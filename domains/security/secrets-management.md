---
domain: security
category: secrets-management
severity: critical
applicable-to: [web, api, mobile, saas, infra]
tags: [secrets, credentials, env-vars, vault, rotation, cicd]
related:
  - ./authentication.md
  - ./encryption-data-protection.md
regulatory-relevance: [ISO27001, HDS, PCI-DSS]
ai-prompt-hints: |
  Demander à voir : .env files, CI/CD configuration, fichiers de déploiement (k8s manifests, docker-compose).
  Chercher dans le code : secrets hardcodés, connexion strings avec credentials, clés API en clair.
  Red flags immédiats : secrets dans git history, credentials dans les logs, .env commité.
  Chercher dans git history : git log --all --oneline -S "password" -S "secret" -S "api_key"
---

# Gestion des secrets

## Risques principaux

| Risque | Impact | Fréquence |
|---|---|---|
| Secrets hardcodés dans le code source | Critique | Très élevée |
| `.env` commité dans git (y compris historique) | Critique | Élevée |
| Credentials partagés entre environnements (prod/dev) | Critique | Élevée |
| Rotation des secrets absente ou manuelle | Haute | Élevée |
| Secrets exposés dans les logs applicatifs | Critique | Moyenne |
| Clés API sans restrictions de périmètre (IP, scope) | Haute | Élevée |
| Secrets CI/CD accessibles à tous les membres | Haute | Moyenne |

## Meilleures pratiques

### Stockage des secrets

- **Vault dédié** : HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault
- **Jamais dans le code** : ni dans les fichiers sources, ni dans les Dockerfiles, ni dans les manifests K8s
- **Jamais dans git** : même chiffrés dans `.env`, même en branch privée
- `.gitignore` avec `.env*` + git-secrets ou truffleHog en pre-commit hook
- **Variables d'environnement injectées** à l'exécution (CI/CD, runtime), pas dans l'image

### Rotation

- Rotation automatique pour les credentials de bases de données (AWS RDS rotation, Vault dynamic secrets)
- Rotation manuelle ≤ 90 jours pour les API keys sans rotation auto
- Immédiatement après un départ d'employé ou un incident de compromission
- Les clés de chiffrement suivent une politique de rotation distincte (voir `encryption-data-protection.md`)

### Secrets CI/CD

- Utiliser les secret stores natifs (GitHub Secrets, GitLab CI Variables, Vault CI integration)
- Masquer les secrets dans les logs de build (vérifier que les runners le font)
- Accès aux secrets de prod restreint aux pipelines de production uniquement
- Audit régulier des secrets CI/CD (qui a accès, lesquels sont actifs)

### Principe du moindre privilège

- Chaque service a ses propres credentials (pas de credentials partagés)
- Scoper les API keys au minimum requis (read-only si possible, IP whitelist)
- Clés de service cloud avec policies IAM minimales (principe du moindre privilège)

## Ce que l'agent IA peut vérifier

### Analyse statique du code

```bash
# Secrets potentiellement hardcodés
grep -rn "password\s*=\s*[\"'][^\"']\|api_key\s*=\s*[\"']\|secret\s*=\s*[\"']" \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.java" --include="*.go"

# Connexion strings avec credentials
grep -rn "postgresql://\|mysql://\|mongodb://" --include="*.py" --include="*.js" --include="*.env*"

# Clés AWS hardcodées (pattern)
grep -rn "AKIA[0-9A-Z]\{16\}" .

# .env dans git index
git ls-files | grep -E "\.env"

# Secrets dans l'historique git
git log --all --oneline -S "password" -S "SECRET" -S "api_key" | head -20
```

### Vérification de configuration

- Vérifier que `.gitignore` contient `.env`, `.env.*`, `*.pem`, `*.key`
- Vérifier la présence d'un pre-commit hook anti-secrets (truffleHog, git-secrets)
- Examiner les manifests K8s : présence de `kind: Secret` avec valeurs en base64 (non chiffrées)
- Lire les fichiers docker-compose : variables en dur vs `${VAR}` depuis l'environnement

### Questions d'audit (entretien)

1. Où sont stockés les secrets de production ? (vault, env vars, fichiers...)
2. Qui a accès aux secrets de production ? (RBAC ?)
3. Y a-t-il une politique de rotation ? À quelle fréquence ?
4. Que se passe-t-il quand un développeur quitte l'équipe ?
5. Les secrets sont-ils différents entre dev/staging/prod ?
6. Comment les secrets sont-ils injectés dans les pipelines CI/CD ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Type | Ce que l'IA peut faire avec l'output |
|---|---|---|---|
| Scan complet historique git pour secrets | **TruffleHog**, **GitLeaks**, **detect-secrets** | SAST/SCA | Analyser le rapport, identifier les secrets actifs vs expirés |
| Scan des images Docker pour secrets | **Trivy** (`trivy image --security-checks secret`) | Container scan | Lire le rapport JSON Trivy fourni |
| Audit des policies IAM cloud | **AWS IAM Access Analyzer**, **Cloudsplaining** | Cloud security | Analyser le rapport de permissions excessives |
| Scan des secrets K8s non chiffrés | **Kubesec**, **kube-bench** | K8s security | Contextualiser les findings par rapport à la criticité des namespaces |

## Références

- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [CIS Benchmark AWS — IAM section](https://www.cisecurity.org/benchmark/amazon_web_services)
- [HashiCorp Vault — Best Practices](https://developer.hashicorp.com/vault/tutorials/recommended-patterns)
