---
domain: organizational
category: gestion-des-incidents
applicable-to: [startup, scaleup, enterprise, team]
tags: [incidents, postmortem, on-call, runbook, sla, slo, mttr, blameless]
related-knowledge: ../incident-management.md
ai-selection-criteria: |
  Inclure si `audit.trigger` = incident (priorité Haute, règle schema.md). Toujours
  inclure dans les audits de maturité organisationnelle. Bonus de priorité +3 si
  l'organisation a subi plusieurs incidents P1 récents sans postmortem. +2 si le
  processus on-call est informel ou si le MTTR moyen est > 4h.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Gestion des incidents

## Détection et triage

- [ ] Des niveaux de sévérité sont définis (P0→P3 ou équivalent) avec des critères explicites
- [ ] Les SLA de réponse sont définis par niveau de sévérité
- [ ] Des alertes automatiques déclenchent le processus on-call (pas de détection manuelle)
- [ ] Le taux de faux positifs des alertes est suivi et maîtrisé (< 2 alertes actionnables par nuit de garde)

## Organisation on-call

- [ ] `[B]` Un planning d'astreinte est formalisé et connu à l'avance (au moins 2 semaines)
- [ ] Une chaîne d'escalade est définie et testée (si on-call ne répond pas sous N minutes → qui ?)
- [ ] Les handoffs de rotation sont documentés (incidents en cours transmis)
- [ ] L'astreinte est reconnue (financièrement ou en récupération)

## Runbooks

- [ ] Des runbooks existent pour les incidents les plus fréquents
- [ ] Chaque runbook est lié à l'alerte qui le déclenche
- [ ] Les runbooks sont tenus à jour (revue au moins trimestrielle)
- [ ] Les runbooks sont accessibles pendant un incident (pas uniquement dans le wiki interne)

## Résolution et communication

- [ ] Un rôle de coordinateur (Incident Commander) est désigné sur les incidents P0/P1
- [ ] Les parties prenantes (management, clients) sont notifiées dans les délais définis
- [ ] Un canal de communication dédié est ouvert pour chaque incident significatif

## Postmortems

- [ ] `[B]` Tout incident P0/P1 fait l'objet d'un postmortem dans les 5 jours ouvrés
- [ ] Les postmortems sont blameless (causes systémiques, pas de désignation de coupable)
- [ ] Les actions issues des postmortems sont suivies jusqu'à résolution (pas de postmortem sans suites)
- [ ] Les postmortems sont partagés avec toute l'équipe technique
- [ ] Les incidents P2 récurrents (3× le même mois) déclenchent un postmortem

## Amélioration continue

- [ ] Une revue mensuelle des actions de postmortems est organisée
- [ ] Les métriques d'incidents (MTTR, MTTD, fréquence par sévérité) sont suivies
- [ ] Les apprentissages sont intégrés dans les runbooks et les pratiques d'équipe

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
