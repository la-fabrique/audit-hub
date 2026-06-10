---
domain: infra-devops
category: backup-recovery
severity: high
applicable-to: [web, api, saas, cloud, on-premise, infra]
tags: [backup, recovery, rpo, rto, pca, pra, disaster-recovery, résilience, hds]
related:
  - ./infrastructure-hardening.md
  - ./observability.md
  - ../security/encryption-data-protection.md
regulatory-relevance: [HDS, ISO27001, PCI-DSS]
ai-prompt-hints: |
  Demander : politique de sauvegarde (fréquence, rétention, localisation), RPO et
  RTO cibles et mesurés, dernière restauration testée. Chercher : configuration
  des backups (manifests Velero, scripts de backup, politiques S3 lifecycle),
  documentation du PCA/PRA. Red flags : aucun test de restauration depuis > 6 mois,
  backups sur le même support que les données sources, RPO/RTO non définis ou jamais
  mesurés, backups non chiffrés.
---

# Sauvegarde et reprise d'activité

## Concepts clés

| Concept | Définition | Question à poser |
|---------|-----------|-----------------|
| **RPO** (Recovery Point Objective) | Perte de données maximale acceptable (durée) | Quelle est la dernière sauvegarde avant l'incident ? |
| **RTO** (Recovery Time Objective) | Durée maximale acceptable d'interruption | En combien de temps le service peut-il reprendre ? |
| **PRA** (Plan de Reprise d'Activité) | Procédures pour reprendre l'activité après sinistre | Y a-t-il un document formalisé et testé ? |
| **PCA** (Plan de Continuité d'Activité) | Procédures pour maintenir l'activité pendant un sinistre | Y a-t-il une stratégie de basculement actif ? |

## Politique de sauvegarde

### Règle 3-2-1

- **3** copies des données
- sur **2** supports différents
- dont **1** hors site (offsite ou cloud différent)

### Fréquence et rétention recommandées

| Type de données | Fréquence | Rétention minimale |
|-----------------|-----------|-------------------|
| Bases de données production | Quotidienne (incrémentale toutes les heures) | 30 jours + archivage 1 an |
| Données de santé (HDS) | Quotidienne | 20 ans (réglementation HDS) |
| Données de paiement (PCI-DSS) | Selon politique de rétention définie | Selon exigences légales |
| Configuration et IaC | À chaque changement (git) | Durée de vie du service |
| Secrets (vault) | Quotidienne | 30 jours |

## Meilleures pratiques

### Sauvegarde

- Les backups sont **chiffrés** au repos (voir `encryption-data-protection.md`)
- Les backups sont stockés dans un compte/région distinct des données sources
- La rétention est automatisée (lifecycle policy, pas de nettoyage manuel)
- Les backups incluent les configurations et l'IaC, pas uniquement les données

### Restauration et tests

- **La restauration est testée** régulièrement — un backup non testé n'est pas un backup
- Cadence recommandée : restauration complète trimestrielle, restauration partielle mensuelle
- Les RTO/RPO mesurés lors des tests sont documentés et comparés aux objectifs
- Le processus de restauration est documenté pas-à-pas (runbook)

### PRA / PCA

- Le PRA est documenté, accessible hors du système principal (pas uniquement dans le wiki interne)
- Les responsables de chaque étape de reprise sont identifiés
- Le PCA est testé annuellement (exercice de basculement, même partiel)
- Le plan intègre les dépendances externes (sous-traitants, services tiers)

## Ce que l'agent IA peut vérifier

```bash
# Configuration Velero (backups Kubernetes)
find . -name "velero*" -o -name "backup-schedule*" | head -10
grep -rn "schedule\|retention\|ttl" --include="*.yaml" | grep -i "backup"

# Politiques de lifecycle S3 / GCS / Azure Blob
find . -name "lifecycle*.json" -o -name "lifecycle*.tf"
grep -rn "lifecycle_rule\|lifecycle_policy" --include="*.tf"

# Scripts de backup
find . -name "*backup*" -name "*.sh" | head -10

# Rétention des snapshots RDS / Cloud SQL
grep -rn "backup_retention\|backup_window\|automated_backup" --include="*.tf"

# Présence d'un PRA/PCA documenté
find . -iname "*pra*" -o -iname "*pca*" -o -iname "*disaster-recovery*" \
  -o -iname "*business-continuity*" | grep -i "\.md$\|\.pdf$\|\.docx$"
```

### Questions d'audit (entretien)

1. Quelle est la fréquence des sauvegardes de la base de données de production ?
2. Où sont stockés les backups ? Sont-ils dans la même région / le même compte que les données ?
3. Les backups sont-ils chiffrés ?
4. Quelle est la dernière fois qu'une restauration complète a été testée ? Avec quel résultat ?
5. Quels sont les RPO et RTO cibles ? Ont-ils déjà été mesurés lors d'un incident ou d'un test ?
6. Existe-t-il un PRA documenté ? Qui en est propriétaire ?
7. En cas de perte totale du datacenter principal, en combien de temps le service serait-il rétabli ?

### Contexte HDS

- La durée de rétention des données de santé est de 20 ans minimum (dossier patient)
- Les sauvegardes des données de santé doivent être chiffrées avec une clé gérée séparément
- Le PRA doit inclure la reprise des accès aux données de santé et être testé

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Test de restauration automatisé | **Velero** (K8s), **AWS Backup Audit Manager** | Analyser les rapports de compliance et les résultats de job |
| Vérification des politiques de backup cloud | **Prowler**, **ScoutSuite** | Identifier les ressources sans backup activé |
| Mesure du RPO effectif | Outils de monitoring (Grafana, Datadog) | Analyser les métriques de lag de réplication |

## Références

- [ISO/IEC 22301 — Continuité d'activité](https://www.iso.org/standard/75106.html)
- [ANSSI — Guide de la continuité d'activité](https://www.ssi.gouv.fr/guide/la-continuity-dactivite/)
- [AWS — Disaster Recovery Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)
- [HDS — Référentiel de certification hébergeurs](https://esante.gouv.fr/labels-certifications/hds/certification-des-hebergeurs-de-donnees-de-sante)
