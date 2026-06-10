---
domain: infra-devops
category: observabilité
applicable-to: [web, api, saas, cloud, microservices]
tags: [monitoring, logs, traces, alerting, slo, observabilité]
related-knowledge: ../observability.md
ai-selection-criteria: |
  Inclure si l'audit couvre l'infrastructure ou la fiabilité du service. Bonus de
  priorité +2 si l'équipe a eu des incidents difficiles à diagnostiquer. +2 si
  aucun SLO n'est défini. +3 en architecture microservices sans traces distribuées.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Observabilité

## Métriques et SLO

- [ ] Des SLI (indicateurs de niveau de service) sont définis pour chaque service critique
- [ ] Des SLO (objectifs de niveau de service) sont définis et mesurés (disponibilité, latence)
- [ ] Les métriques sont collectées et visualisées (Prometheus, Datadog, CloudWatch…)
- [ ] Des dashboards existent pour les services critiques
- [ ] Les alertes sont basées sur les SLO (pas uniquement sur des métriques techniques)
- [ ] L'error budget est calculé et utilisé pour les décisions de déploiement

## Logs

- [ ] Les logs sont structurés (format JSON)
- [ ] Les logs sont centralisés (ELK, Loki, Datadog Logs, CloudWatch Logs)
- [ ] Chaque log inclut un `trace_id` / `correlation_id`
- [ ] Les logs ne contiennent pas de données personnelles ou de secrets
- [ ] Les niveaux de log sont adaptés par environnement (DEBUG désactivé en prod)
- [ ] La rétention des logs est définie et conforme aux exigences réglementaires

## Traces distribuées

- [ ] Des traces distribuées sont implémentées (OpenTelemetry, Jaeger, Datadog APM…)
- [ ] Le `trace_id` est propagé entre tous les services (headers W3C Trace Context)
- [ ] Les traces permettent d'identifier la latence par service
- [ ] Le sampling est configuré (100% sur les erreurs, échantillonnage en nominal)

## Alertes

- [ ] Chaque alerte critique a un runbook associé
- [ ] Les alertes ont un owner d'équipe défini
- [ ] Les alertes inutiles (trop de faux positifs) sont régulièrement nettoyées
- [ ] Les alertes critiques déclenchent une notification immédiate (PagerDuty, OpsGenie…)
- [ ] Un silence d'alerte planifié existe pour les maintenances programmées

## Health checks

- [ ] Chaque service expose un endpoint de health check (`/health`, `/ready`)
- [ ] Les health checks vérifient les dépendances (DB, cache, services tiers)
- [ ] L'orchestrateur (K8s, ECS) utilise ces health checks pour le routing et le restart
- [ ] Les health checks sont monitorés en dehors du service lui-même

## Score d'évaluation

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
