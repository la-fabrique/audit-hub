---
domain: security
category: network-security
applicable-to: [web, api, saas, cloud, infra, on-premise]
tags: [network, firewall, segmentation, tls, vpc, ingress, waf, kubernetes]
related-knowledge: ../network-security.md
ai-selection-criteria: |
  Inclure si l'organisation opère sa propre infrastructure (cloud ou on-premise).
  Bonus de priorité +3 si contexte PCI-DSS (segmentation CDE obligatoire) ou HDS
  (segmentation zone santé). +2 si déploiement sur K8s ou si l'organisation a subi
  un incident réseau. Vérification automatique possible si `access.infrastructure` true.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Sécurité réseau

## Segmentation et exposition

- [ ] `[B]` Aucun port d'administration (SSH 22, RDP 3389) n'est ouvert directement sur Internet
- [ ] `[B]` Les bases de données ne sont pas dans un subnet public
- [ ] `[B]` Les security groups / règles firewall appliquent un deny by default
- [ ] Les environnements prod, staging et dev sont sur des réseaux distincts (pas de VPC partagé)
- [ ] L'egress est filtré : seules les destinations légitimes sont autorisées

## Accès administration

- [ ] L'accès SSH/RDP aux machines de production passe par un bastion, VPN ou SSM Session Manager
- [ ] MFA obligatoire sur les accès d'administration
- [ ] Les sessions d'administration sont loggées
- [ ] Aucune clé SSH partagée entre plusieurs personnes

## TLS et certificats

- [ ] TLS 1.2 minimum sur tous les endpoints (délégué à la checklist encryption si présente)
- [ ] Le renouvellement des certificats est automatisé (pas d'expiration manquée en prod)
- [ ] HSTS activé sur tous les domaines applicatifs

## WAF et protection applicative

- [ ] Un WAF est déployé devant les applications publiques (règles OWASP CRS minimum)
- [ ] Le rate limiting est appliqué au niveau du reverse proxy / CDN
- [ ] Une protection anti-DDoS est en place pour les services exposés

## Kubernetes (si applicable)

- [ ] `[B]` Des NetworkPolicies sont actives sur tous les namespaces (deny by default)
- [ ] L'ingress controller gère la terminaison TLS avec certs managés (cert-manager, ACM…)
- [ ] `hostNetwork: true` n'est utilisé que si explicitement justifié
- [ ] Les namespaces combinent RBAC + NetworkPolicy + limites de ressources

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
