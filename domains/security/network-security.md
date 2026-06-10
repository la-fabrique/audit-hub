---
domain: security
category: network-security
severity: critical
applicable-to: [web, api, saas, cloud, infra, on-premise]
tags: [network, firewall, segmentation, tls, vpc, ingress, waf]
related:
  - ./access-control.md
  - ./encryption-data-protection.md
  - ../infra-devops/infrastructure-hardening.md
regulatory-relevance: [ISO27001, HDS, PCI-DSS, NIS2, SOC2]
ai-prompt-hints: |
  Demander à voir : topologie réseau (VPC, subnets, security groups), règles
  d'ingress/egress, configuration WAF/CDN, terminaison TLS, schéma de
  segmentation entre environnements et entre zones de confiance.
  Chercher dans le code/IaC : security groups trop ouverts (0.0.0.0/0),
  ports admin exposés, absence de WAF en amont, certificats expirés ou TLS
  < 1.2 toléré.
  Red flags immédiats : SSH/RDP ouvert sur Internet, base de données dans
  un subnet public, aucune séparation prod/dev/test, certificats wildcard
  partagés entre tenants.
---

# Sécurité réseau

## Risques principaux

| Risque | Impact | Fréquence |
|---|---|---|
| Ports d'administration exposés à Internet (SSH 22, RDP 3389, MongoDB 27017…) | Critique | Élevée |
| Base de données accessible depuis subnet public | Critique | Moyenne |
| Pas de segmentation entre environnements (prod/staging/dev partagent réseau) | Haute | Élevée |
| TLS faible (< 1.2) ou cipher suites obsolètes | Haute | Moyenne |
| Certificats expirés / non renouvelés automatiquement | Haute | Moyenne |
| Pas de WAF devant les applications publiques | Haute | Moyenne |
| Egress non restreint (data exfiltration silencieuse possible) | Haute | Très élevée |
| Security groups en `allow all` (0.0.0.0/0 sur tous ports) | Critique | Faible (mais catastrophique) |

## Meilleures pratiques

### Segmentation et zones de confiance

- Découpage en zones : DMZ (publique), application (privée), données (privée stricte), management (privée + bastion).
- Une **VPC/VNet par environnement** (prod, staging, dev) — pas de peering croisé sans justification.
- Microsegmentation interne pour les déploiements multi-tenants (network policies K8s, security groups granulaires).
- Pas de base de données dans un subnet public. Accès uniquement via subnet privé + endpoint dédié.

### Filtrage et contrôle d'accès réseau

- Security groups / firewall en **deny by default**, ouverture explicite par règle.
- Egress filtré : autoriser uniquement les destinations légitimes (APIs externes connues, registries, DNS).
- Accès admin (SSH, RDP, kubectl) **jamais directement sur Internet** — passer par bastion, VPN, SSM Session Manager, Tailscale ou équivalent.
- Si un bastion est nécessaire, MFA obligatoire et logs de session conservés.

### Chiffrement en transit (TLS)

- TLS **1.2 minimum**, 1.3 recommandé. Désactiver 1.0/1.1 et SSLv3.
- Cipher suites : seules les suites AEAD (`AES-GCM`, `ChaCha20-Poly1305`). Pas de `RC4`, `3DES`, `CBC` avec hash faibles.
- HSTS activé sur tous les domaines applicatifs (`max-age ≥ 31536000`, `includeSubDomains`).
- Renouvellement de certificats automatisé (Let's Encrypt + cert-manager, ACM, etc.). Aucun renouvellement manuel pour la prod.
- Vérifier régulièrement : test SSL Labs grade ≥ A.

### WAF et protection applicative

- WAF en amont des applications publiques (AWS WAF, Cloudflare, Akamai…) avec a minima les règles OWASP CRS.
- Rate limiting au niveau du reverse proxy / CDN (pas seulement applicatif).
- Protection anti-DDoS (Cloudflare, AWS Shield, GCP Cloud Armor) pour les services exposés.

### Cas Kubernetes

- NetworkPolicies actives sur tous les namespaces (deny by default, allow ciblé).
- Ingress controller avec TLS terminé et certs gérés par cert-manager.
- Pas de `hostNetwork: true` sauf justification explicite.
- Séparation par namespace + RBAC + NetworkPolicy (les trois ensemble).

## Ce que l'agent IA peut vérifier

### Analyse statique IaC

```bash
# Security groups ouverts à 0.0.0.0/0 (Terraform)
grep -rn "0\.0\.0\.0/0" --include="*.tf"

# Ports admin exposés
grep -rnE "from_port[[:space:]]*=[[:space:]]*(22|3389|3306|5432|6379|27017)" --include="*.tf"

# TLS toléré < 1.2
grep -rnE "tls.?1[._]?0|tls.?1[._]?1|ssl.?v3" --include="*.tf" --include="*.yaml" --include="*.conf"

# Ingress K8s sans TLS
grep -rn "kind:\s*Ingress" -A20 . | grep -B1 -A1 "host:" | grep -v "tls"

# NetworkPolicy absente — namespace sans policy
ls k8s/**/networkpolicy*.yaml 2>/dev/null
```

### Vérification de configuration

- Lire les SGs / firewall rules et lister les règles `0.0.0.0/0` non justifiées.
- Vérifier la topologie : DB dans subnet privé sans route vers Internet Gateway.
- Examiner la configuration TLS du load balancer / ingress (versions, ciphers, HSTS).
- Vérifier les NetworkPolicies K8s par namespace.

### Questions d'audit (entretien)

1. Comment l'accès SSH/admin aux machines de production est-il fait ?
2. La base de données est-elle accessible depuis Internet ? Depuis le réseau de bureau ?
3. Y a-t-il un WAF devant les applications publiques ? Quelles règles ?
4. Comment les certificats TLS sont-ils renouvelés ? Y a-t-il déjà eu une expiration en prod ?
5. Le trafic egress est-il filtré ? Que peuvent appeler les services en sortie ?
6. Comment l'équipe détecte-t-elle une connexion depuis une IP inhabituelle ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Type | Ce que l'IA peut faire avec l'output |
|---|---|---|---|
| Scan de ports externes | **Nmap**, **masscan** | Network scan | Lire le rapport, identifier les ports ouverts non justifiés |
| Audit configuration TLS | **SSL Labs**, **testssl.sh** | TLS scan | Analyser le rapport, prioriser par sévérité |
| Audit IaC réseau cloud | **Checkov**, **tfsec**, **Prowler**, **ScoutSuite** | IaC/cloud scan | Trier les findings par criticité métier |
| Détection d'exposition involontaire | **Shodan**, **Censys** | Reconnaissance | Identifier les actifs non répertoriés |
| Audit Kubernetes réseau | **kube-bench**, **kube-hunter** | K8s security | Contextualiser les findings par namespace |

## Références

- [OWASP — Transport Layer Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)
- [Mozilla — Server Side TLS Configuration Guide](https://wiki.mozilla.org/Security/Server_Side_TLS)
- [CIS Benchmarks — AWS / GCP / Azure Networking](https://www.cisecurity.org/cis-benchmarks)
- [NIST SP 800-41 Rev. 1 — Guidelines on Firewalls and Firewall Policy](https://csrc.nist.gov/publications/detail/sp/800-41/rev-1/final)
