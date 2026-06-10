---
domain: infra-devops
category: durcissement
applicable-to: [cloud, on-premise, kubernetes, docker]
tags: [hardening, cis, cloud, kubernetes, docker, iam, réseau, backup]
related-knowledge: ../infrastructure-hardening.md
ai-selection-criteria: |
  Inclure si l'audit couvre l'infrastructure. Bonus de priorité +3 si l'organisation
  est dans le cloud sans audit de sécurité récent. +2 si Kubernetes est utilisé en
  production. +3 si des données personnelles ou sensibles sont traitées.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Durcissement de l'infrastructure

## Gestion des identités et accès (IAM)

- [ ] `[B]` MFA obligatoire pour tous les comptes avec accès console cloud
- [ ] Principe du moindre privilège appliqué (pas de wildcards `*:*` dans les policies)
- [ ] Aucun accès root/admin partagé entre plusieurs personnes
- [ ] Les comptes de service ont des permissions minimales et des clés rotatives
- [ ] Les accès sont revus et nettoyés au moins trimestriellement
- [ ] Les comptes inactifs depuis > 90 jours sont désactivés

## Réseau

- [ ] Le réseau est segmenté (VPC, sous-réseaux privés/publics)
- [ ] `[B]` Les bases de données ne sont pas accessibles depuis l'internet public
- [ ] Les Security Groups / firewalls n'autorisent pas `0.0.0.0/0` sur les ports sensibles
- [ ] Le trafic entre services internes est chiffré (TLS mutuel ou VPN)
- [ ] Les ports exposés sont inventoriés et justifiés

## Serveurs et OS

- [ ] L'accès SSH root direct est désactivé (`PermitRootLogin no`)
- [ ] L'authentification par mot de passe SSH est désactivée (clés uniquement)
- [ ] Les mises à jour de sécurité sont appliquées automatiquement ou sous SLA de 72h
- [ ] Un bastion / jump host centralise les accès SSH avec audit trail
- [ ] Les services inutiles sont désactivés

## Containers et Kubernetes

- [ ] Les containers ne tournent pas en root (`runAsNonRoot: true`)
- [ ] Le filesystem est en lecture seule quand possible (`readOnlyRootFilesystem: true`)
- [ ] L'escalade de privilèges est interdite (`allowPrivilegeEscalation: false`)
- [ ] Les Network Policies K8s sont définies (deny-all + allowlist)
- [ ] RBAC K8s : pas de `cluster-admin` pour les workloads applicatifs
- [ ] Les secrets K8s sont gérés via External Secrets ou Vault (pas encodés en base64 dans les manifests)
- [ ] Les images de base sont scannées régulièrement (Trivy, Grype)

## Chiffrement

- [ ] Les données au repos sont chiffrées (EBS, RDS, S3, disques VM)
- [ ] Les données en transit sont chiffrées (TLS 1.2+ minimum, TLS 1.3 recommandé)
- [ ] Les clés de chiffrement sont gérées par un KMS (pas hard-codées)
- [ ] Les certificats SSL/TLS sont valides et renouvelés avant expiration

## Audit et journalisation

- [ ] Les logs d'audit cloud sont activés (CloudTrail, Azure Activity Log, GCP Audit Logs)
- [ ] Les logs couvrent toutes les régions / zones actives
- [ ] Les logs sont centralisés et protégés contre la suppression
- [ ] Les alertes existent sur les actions administratives critiques (création de compte, modification de policy)

## Sauvegardes

- [ ] Des sauvegardes automatiques existent pour toutes les données critiques
- [ ] Les sauvegardes sont stockées dans une région / zone différente
- [ ] Les sauvegardes sont chiffrées
- [ ] Un test de restore a été effectué dans les 3 derniers mois et documenté
- [ ] Le RTO (Recovery Time Objective) et RPO (Recovery Point Objective) sont définis

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
