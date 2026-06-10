---
domain: infra-devops
category: durcissement
severity: high
applicable-to: [cloud, on-premise, kubernetes, docker, linux]
tags: [hardening, cis-benchmarks, iac, terraform, kubernetes, réseau, accès, cloud-security]
related:
  - ./cicd-security.md
  - ./checklists/infrastructure-checklist.md
ai-prompt-hints: |
  Demander accès à : configuration cloud (policies IAM, security groups, VPC),
  Dockerfiles, manifests Kubernetes, scripts Terraform/Ansible, policy de backup.
  Chercher : règles de firewall ouvertes (0.0.0.0/0), ports exposés inutilement,
  containers en root, secrets dans les manifests K8s, S3 buckets publics, MFA
  désactivé sur les comptes cloud.
  Red flags : accès root direct sur les serveurs de prod, pas de MFA sur les
  comptes cloud admin, backups non testés, réseau à plat (pas de segmentation).
---

# Durcissement de l'infrastructure

## Checklist de risques critiques

| Risque | Impact | Détection |
|--------|--------|-----------|
| MFA désactivé sur les comptes cloud admin | Critique | Vérifier IAM policies |
| Accès root SSH direct aux serveurs de prod | Critique | `grep PermitRootLogin /etc/ssh/sshd_config` |
| Security groups ouverts sur 0.0.0.0/0 | Critique | Audit AWS/Azure/GCP |
| Containers qui tournent en root | Haut | `docker inspect` ou manifests K8s |
| Secrets en clair dans les manifests K8s | Critique | `kubectl get secrets -o yaml` |
| S3 / Blob Storage public par défaut | Critique | Audit de configuration cloud |
| Backups non testés (ou inexistants) | Critique | Demander le dernier test de restore |

## Durcissement par couche

### Cloud (AWS / Azure / GCP)

Suivre les **CIS Benchmarks** pour la plateforme cible :

```bash
# Audit automatisé avec Prowler (AWS)
prowler aws --compliance cis_aws_2.0

# Scout Suite (multi-cloud)
python scout.py aws --report-dir ./scout-report

# Checkov sur les configs Terraform
checkov -d ./terraform --framework terraform --output sarif
```

Contrôles prioritaires :
- **IAM** : principe du moindre privilège, pas de wildcards (`*:*`), rotation des clés
- **Réseau** : VPC segmenté, Security Groups restrictifs, pas d'accès public aux DB
- **Chiffrement** : EBS/S3/RDS chiffrés au repos, KMS géré par client
- **Audit** : CloudTrail / Azure Activity Log activé dans toutes les régions
- **MFA** : obligatoire pour tous les comptes avec accès console

### Kubernetes

```bash
# Audit CIS Kubernetes
kube-bench run --targets node,master

# Scan des manifests
trivy config ./k8s/
checkov -d ./k8s --framework kubernetes

# Vérifier les containers qui tournent en root
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .spec.containers[*]}{.securityContext.runAsUser}{"\n"}{end}{end}'

# Secrets non chiffrés
kubectl get secrets --all-namespaces -o json | jq '.items[] | select(.type != "kubernetes.io/service-account-token") | .metadata.name'
```

Contrôles prioritaires Kubernetes :
- **RBAC** : pas de `cluster-admin` pour les workloads applicatifs
- **Security Context** : `runAsNonRoot: true`, `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`
- **Network Policies** : deny-all par défaut, allowlist explicite
- **Secrets** : External Secrets Operator ou Vault, pas de secrets base64 dans les manifests
- **Admission Controllers** : OPA/Gatekeeper ou Kyverno pour appliquer les politiques

### Docker / Images

```bash
# Scanner une image
trivy image --severity HIGH,CRITICAL myapp:v1.2.3

# Dockerfile — bonnes pratiques
# ❌ Mauvais
FROM ubuntu:latest
RUN apt-get install -y everything
USER root

# ✓ Bon
FROM node:20-alpine AS builder
# ... build multi-stage ...
FROM gcr.io/distroless/nodejs20-debian12
COPY --from=builder /app/dist /app
USER nonroot
```

### Serveurs Linux (on-premise ou VMs cloud)

Contrôles CIS Level 1 prioritaires :

```bash
# SSH hardening
grep -E "PermitRootLogin|PasswordAuthentication|PubkeyAuthentication|Protocol" /etc/ssh/sshd_config

# Ports ouverts
ss -tlnp | grep LISTEN

# Comptes avec UID 0 (root)
awk -F: '($3 == "0") {print}' /etc/passwd

# Mises à jour de sécurité manquantes
apt list --upgradable 2>/dev/null | grep -i security
# ou
yum check-update --security
```

### Infrastructure as Code (IaC)

- **Toujours versionner** la configuration (Terraform, Ansible, Helm)
- **Ne jamais appliquer manuellement** en production — tout passe par CI/CD avec revue
- **State distant** : Terraform Cloud, S3 + DynamoDB lock (jamais local)
- **Drift detection** : détecter les changements manuels hors IaC (`terraform plan` en CI)

```bash
# Lint Terraform
tflint --recursive

# Sécurité IaC
checkov -d . --framework terraform
tfsec .

# Détection de drift
terraform plan -detailed-exitcode  # exit code 2 = drift détecté
```

## Gestion des accès et des comptes

- **Bastion / Jump host** : accès SSH via bastion avec logs d'audit, jamais directement
- **Accès temporaires** : utiliser des credentials éphémères (AWS STS, Vault dynamic secrets)
- **Revue trimestrielle** des accès : désactiver les comptes inactifs
- **Break-glass** : procédure documentée pour accès d'urgence, avec audit trail

## Sauvegardes

| Aspect | Exigence minimale |
|--------|------------------|
| Fréquence | Quotidienne pour les données critiques |
| Rétention | 30 jours minimum, 1 an pour les données réglementaires |
| Chiffrement | Sauvegardes chiffrées avec clé séparée |
| Localisation | Backup dans une région / zone différente |
| **Test de restore** | Au moins une fois par trimestre, documenté |

Un backup non testé n'est pas un backup.

## Ce que l'agent IA peut vérifier

### Analyse des configurations (IaC, manifests, Dockerfiles)

```bash
# Règles réseau ouvertes au monde dans Terraform
grep -rn "0.0.0.0/0\|::/0" --include="*.tf"

# Containers privilégiés ou root dans les manifests K8s
grep -rn "privileged: true\|runAsUser: 0\|allowPrivilegeEscalation: true" k8s/

# Secrets en clair dans les manifests ou docker-compose
grep -rniE "(password|secret|token)\s*[:=]\s*[\"'][^\"']" k8s/ docker-compose*.yml

# Buckets S3 publics dans Terraform
grep -rn "acl.*public\|block_public.*false" --include="*.tf"

# Dockerfiles sans directive USER (root par défaut)
grep -rL "^USER" --include="Dockerfile*" .
```

Les commandes par couche des sections ci-dessus (contrôles SSH, `kubectl`,
`ss -tlnp`…) sont exécutables directement si l'accès correspondant est disponible
(`access.infra_config`, accès shell aux serveurs).

### Questions d'audit (entretien)

1. Quand le dernier test de restauration de backup a-t-il été réalisé ? Est-il documenté ?
2. Le MFA est-il obligatoire sur tous les comptes cloud avec accès console ?
3. Comment accède-t-on aux serveurs de prod ? (bastion, SSH direct, credentials éphémères)
4. Existe-t-il une procédure break-glass documentée et auditée ?
5. À quand remonte la dernière revue des accès ? Les comptes inactifs sont-ils désactivés ?
6. Les changements d'infra passent-ils tous par l'IaC, ou des modifications manuelles sont-elles possibles ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Audit CIS Cloud complet | Prowler, Scout Suite, Steampipe | Analyser le rapport JSON, prioriser les findings |
| Scan IaC | Checkov, tfsec, Trivy config | Analyser le rapport SARIF |
| Scan images Docker | Trivy, Grype, Snyk Container | Analyser les CVE et prioriser par exposition |
| Audit K8s | kube-bench, Polaris | Lire le rapport et identifier les gaps CIS |
| Pen test infra | Nessus, OpenVAS | Contextualiser les CVE identifiées |

## Références

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
- [OWASP Cloud Security](https://owasp.org/www-project-cloud-security/)
- [Kubernetes Security Best Practices — NSA/CISA](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
- [AWS Well-Architected — Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
- [Trivy](https://trivy.dev/)
