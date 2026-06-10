---
domain: security
category: logging-monitoring
severity: high
applicable-to: [web, api, saas, infra]
tags: [logs, monitoring, siem, alerting, audit-trail, observability]
related:
  - ./authentication.md
  - ./vulnerability-management.md
  - ./checklists/logging-monitoring-checklist.md
regulatory-relevance: [ISO27001, HDS, PCI-DSS, NIS2, SOC2, RGPD]
ai-prompt-hints: |
  Demander : qu'est-ce qui est loggé ? Où ? Combien de temps ?
  Chercher dans le code : configuration des loggers, niveaux de log, présence de données sensibles dans les logs.
  Red flags : mots de passe dans les logs, pas de logs d'accès aux données sensibles,
  logs sans timestamp, rétention < 90 jours en contexte réglementaire.
  Vérifier : les logs des accès admin, les logs d'authentification, les logs d'accès aux données de santé (HDS).
---

# Journalisation et monitoring de sécurité

## Risques principaux

| Risque | Impact | Fréquence |
|---|---|---|
| Données sensibles (mots de passe, tokens) dans les logs | Critique | Élevée |
| Absence de logs d'accès aux données sensibles | Critique (RGPD, HDS) | Moyenne |
| Logs non centralisés (perdus si serveur down) | Haute | Élevée |
| Rétention insuffisante (impossible de mener une investigation) | Haute | Élevée |
| Alertes absentes sur les événements de sécurité | Haute | Très élevée |
| Logs modifiables par les utilisateurs loggés | Critique | Faible |
| Absence de corrélation d'événements (SIEM) | Haute | Élevée |

## Ce qu'il faut logger (minimum)

### Événements d'authentification

- Connexions réussies et échouées (avec IP, user-agent, timestamp, user ID)
- Déconnexions
- Changements de mot de passe / MFA
- Émission et révocation de tokens
- Tentatives d'accès non autorisés (403)

### Accès aux données sensibles (critique pour HDS, RGPD, PCI-DSS)

- Lecture de données personnelles / de santé (qui, quand, quoi)
- Export de données
- Modifications d'enregistrements sensibles
- Suppressions

### Événements d'administration

- Créations/modifications/suppressions de comptes
- Changements de permissions / rôles
- Accès aux fonctions d'administration
- Modifications de configuration de sécurité

### Événements d'infrastructure

- Démarrages / arrêts de services
- Déploiements
- Erreurs système et exceptions non gérées
- Connexions SSH / accès console

## Ce que l'agent IA peut vérifier

### Analyse statique du code

```bash
# Données sensibles dans les logs
grep -rn "log.*password\|log.*token\|log.*secret\|logger.*api_key" \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.java"

# Vérifier les niveaux de log (trop verbeux en prod = risque)
grep -rn "logging.DEBUG\|console.log\|print(" --include="*.py" --include="*.js"

# Configuration des loggers
find . -name "logback.xml" -o -name "log4j*.xml" -o -name "logging.conf" \
  -o -name "logger.config.*" 2>/dev/null | head -10

# Rechercher si les access logs HTTP sont activés
grep -rn "morgan\|access.log\|AccessLog\|request.*log" \
  --include="*.js" --include="*.ts" --include="*.xml" --include="*.conf"
```

### Questions d'audit (entretien)

1. Qu'est-ce qui est loggé aujourd'hui ? (accès, erreurs, actions admin...)
2. Où sont stockés les logs ? (local, centralisé, cloud)
3. Quelle est la durée de rétention ? (HDS : 5 ans, PCI-DSS : 12 mois)
4. Les logs sont-ils protégés contre la modification / suppression ?
5. Existe-t-il des alertes sur les événements de sécurité ? (brute force, accès anormal...)
6. Qui a accès aux logs ? Les logs d'accès aux données de santé sont-ils séparés ?
7. Y a-t-il un SIEM ou une corrélation d'événements ?

### Revue de configuration de monitoring

- Lire les configs Prometheus/Grafana, ELK, Datadog si disponibles
- Vérifier les règles d'alerte configurées (trop peu = manque de couverture)
- Examiner les dashboards existants

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Type | Ce que l'IA peut faire avec l'output |
|---|---|---|---|
| Centralisation et analyse des logs | **ELK Stack**, **Loki/Grafana**, **Datadog**, **Splunk** | SIEM/Observabilité | Analyser les patterns de logs fournis, identifier anomalies |
| Détection d'anomalies comportementales | **Wazuh**, **Falco** (K8s), **Elastic SIEM** | SIEM/IDS | Contextualiser les alertes selon le profil de risque |
| Audit de configuration logging | Revue manuelle + scripts de vérification | Config audit | L'IA peut faire cette revue directement |
| Test de non-répudiation des logs | Solution SIEM avec tamper detection | SIEM | Vérifier l'architecture de protection des logs |

## Exigences de rétention par norme

| Norme | Rétention minimale | Scope |
|---|---|---|
| **HDS** | 5 ans (accès données de santé) | Logs d'accès aux DPS |
| **PCI-DSS** | 12 mois (3 mois online, 9 archivés) | Tous logs CDE |
| **ISO 27001** | Définie dans la politique de l'org | Logs de sécurité |
| **RGPD** | Durée proportionnée au traitement | Logs d'accès aux données perso |
| **NIS2** | Selon politique nationale (ANSSI) | Incidents et événements |
| **SOC 2** | Minimum 6 mois (12 recommandé) | Logs de sécurité |

## Indicateurs clés

- **Couverture de logging** : % des événements de sécurité identifiés qui sont loggés
- **MTTD** (Mean Time To Detect) : délai de détection d'un incident
- **Ratio alertes actionnables / bruit** (trop de faux positifs = alertes ignorées)
- **Durée de rétention effective** vs exigence réglementaire

## Références

- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [CIS Controls v8 — Control 8 (Audit Log Management)](https://www.cisecurity.org/controls/)
- [ANSSI — Recommandations pour la journalisation](https://www.ssi.gouv.fr/guide/recommandations-de-securite-pour-la-mise-en-oeuvre-de-la-journalisation/)
