---
domain: infra-devops
category: observabilité
severity: high
applicable-to: [web, api, saas, cloud, microservices]
tags: [observabilité, monitoring, logs, traces, alerting, slo, opentelemetry, prometheus, grafana]
related:
  - ./cicd-security.md
  - ./infrastructure-hardening.md
  - ./checklists/observability-checklist.md
ai-prompt-hints: |
  Demander : quels outils de monitoring sont en place, comment les alertes sont
  gérées, si des SLO sont définis.
  Chercher : configuration Prometheus/Grafana ou équivalent, dashboards existants,
  présence de logs structurés, correlation IDs dans les logs.
  Red flags : alertes uniquement sur des métriques techniques (CPU), pas de SLO
  orientés utilisateur, logs non structurés (plain text), absence de traces
  distribuées en microservices, alertes qui ne déclenchent jamais d'action.
---

# Observabilité

## Les 3 piliers

| Pilier | Ce qu'il mesure | Sans lui |
|--------|----------------|----------|
| **Métriques** | Compteurs, gauges, histogrammes — état du système à un instant | On sait que quelque chose ne va pas mais pas quoi |
| **Logs** | Événements discrets avec contexte | On ne peut pas reconstituer ce qui s'est passé |
| **Traces** | Parcours d'une requête à travers les services | En microservices, on ne sait pas où la latence est introduite |

## Meilleures pratiques

### Métriques et SLO

Définir des **SLI (Service Level Indicators)** mesurables, puis des **SLO (Service Level Objectives)** :

```
SLI : taux de succès des requêtes HTTP (codes 2xx / total)
SLO : 99,5% des requêtes réussies sur 30 jours glissants
Error Budget : 0,5% d'échecs autorisés = ~3,6h/mois
```

SLO recommandés à définir :
- **Disponibilité** : % requêtes réussies (pas de 5xx)
- **Latence** : p99 < X ms (percentile 99, pas la moyenne)
- **Saturation** : CPU/mémoire/connexions DB < seuil critique

Outillage courant :
- **Prometheus + Grafana** (open source, self-hosted)
- **Datadog / New Relic** (SaaS, tout-en-un)
- **CloudWatch / Azure Monitor** (si mono-cloud)

### Logs structurés

```json
// Bon — log structuré JSON
{"level":"error","timestamp":"2024-03-15T10:23:45Z","trace_id":"abc123","user_id":"u-456","service":"payment","message":"Payment declined","code":"CARD_EXPIRED"}

// Mauvais — log texte non structuré
ERROR 10:23:45 payment declined for user u-456
```

Règles :
- Format JSON obligatoire en production
- Champs standards : `level`, `timestamp`, `service`, `trace_id`, `message`
- Ne jamais logger de données personnelles ou secrets (PII, tokens, mots de passe)
- Niveau de log adapté : DEBUG désactivé en prod (coût + PII), ERROR pour ce qui mérite une alerte

### Traces distribuées

En architecture multi-services, les traces permettent de suivre une requête end-to-end.

- Standard : **OpenTelemetry** (instrumentation) → Jaeger, Tempo, Datadog APM (stockage)
- `trace_id` injecté dans tous les logs et propagé entre services (headers HTTP W3C Trace Context)
- Sampling : 100% en dev, 1-10% en prod (coût), 100% sur les erreurs

### Alertes — qualité vs quantité

| Anti-pattern | Conséquence | Correction |
|---|---|---|
| Alertes sur métriques techniques (CPU > 80%) | Alert fatigue, cause racine floue | Alerter sur les SLO, pas les ressources |
| Alertes sans runbook | On-call perd du temps | Chaque alerte pointe vers un runbook |
| Alertes silencieuses (trop de faux positifs) | On ignore les alertes | Seuil > 95% de précision, révision mensuelle |
| Alertes sans owner | Personne ne la traite | Chaque alerte a un owner d'équipe défini |

## Ce que l'agent IA peut vérifier

### Analyse des configurations

```bash
# Présence d'outils de monitoring
ls docker-compose*.yml | xargs grep -l "prometheus\|grafana\|datadog\|newrelic" 2>/dev/null
find . -name "prometheus.yml" -o -name "alertmanager.yml" -o -name "otel-collector.yaml"

# Logs structurés dans le code
grep -rn "console.log\|print(" --include="*.ts" --include="*.py" | grep -v "\.test\.\|spec\." | head -20
# → signale les logs non structurés potentiels

# Présence de correlation ID / trace_id
grep -rn "trace_id\|traceId\|correlation_id\|x-request-id" --include="*.ts" --include="*.py" | head -10

# Données sensibles dans les logs (red flag)
grep -rn "logger.*password\|log.*token\|log.*secret" --include="*.ts" --include="*.py" -i | head -10

# Configuration des niveaux de log par environnement
grep -rn "LOG_LEVEL\|log_level\|logging.level" --include="*.env*" --include="*.yaml" --include="*.yml" | head -10
```

### Questions d'audit (entretien)

1. Quels SLO sont définis ? Sont-ils suivis avec un error budget ?
2. Comment saviez-vous que le dernier incident de production avait eu lieu ?
3. Les logs sont-ils structurés (JSON) et centralisés ?
4. Peut-on retrouver toutes les actions d'une requête en microservices via un trace_id ?
5. Les alertes déclenchent-elles une action systématique ou sont-elles souvent ignorées ?
6. Combien de temps faut-il pour diagnostiquer un incident de production p50 ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| État des dashboards Grafana | Accès Grafana ou export JSON | Analyser les dashboards configurés, identifier les SLO manquants |
| Qualité des alertes | Export des règles Prometheus/Alertmanager | Identifier les alertes sans runbook ou sans owner |
| Analyse des logs | ELK, Loki, Datadog Logs | Analyser les patterns d'erreurs, détecter les PII dans les logs |
| APM / traces | Jaeger, Datadog APM, Tempo | Analyser les traces de latence, identifier les bottlenecks |

## Références

- [Google SRE Book — SLOs](https://sre.google/sre-book/service-level-objectives/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/)
- [DORA — Monitoring and Observability](https://dora.dev/devops-capabilities/technical/monitoring-and-observability/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
