---
domain: organizational
category: gestion-des-incidents
severity: high
applicable-to: [startup, scaleup, enterprise, team]
tags: [incidents, postmortem, on-call, runbook, sla, slo, mttr, blameless]
related:
  - ./tech-maturity.md
  - ./team-structure.md
  - ./checklists/governance-checklist.md
ai-prompt-hints: |
  Demander : comment le dernier incident significatif a été géré, qui a été
  alerté, combien de temps il a fallu pour résoudre, si un postmortem a été fait.
  Chercher : existence de runbooks, configuration des alertes et on-call, historique
  des incidents, postmortems disponibles.
  Red flags : aucun postmortem sur les incidents, on-call informel (tout le monde
  reçoit des alertes), MTTR > 4h sur des incidents récurrents, incidents résolus
  sans analyse de cause racine.
---

# Gestion des incidents

## Cycle de vie d'un incident

```
Détection → Triage → Résolution → Communication → Postmortem → Amélioration
```

Chaque étape doit être documentée, avec un responsable désigné.

## Rôles dans un incident

| Rôle | Responsabilité | Obligatoire à partir de |
|------|---------------|------------------------|
| **Incident Commander (IC)** | Coordonne la réponse, prend les décisions | Incident P1/P2 |
| **Tech Lead** | Diagnostique et applique les corrections | Tout incident technique |
| **Comms Lead** | Communication vers les parties prenantes | Incident avec impact utilisateurs |
| **Scribe** | Documente la timeline en temps réel | Incident P1/P2 |

En petite équipe, une seule personne peut tenir plusieurs rôles — mais le rôle IC doit être explicite.

## Sévérité des incidents

Définir des niveaux clairs avec des SLA de réponse :

| Sévérité | Définition | SLA détection | SLA résolution |
|----------|-----------|---------------|----------------|
| **P0 — Critique** | Service totalement indisponible, impact majeur | < 5 min | < 2h |
| **P1 — Élevée** | Fonctionnalité majeure dégradée, > X% utilisateurs impactés | < 15 min | < 4h |
| **P2 — Modérée** | Fonctionnalité secondaire dégradée, workaround possible | < 1h | < 24h |
| **P3 — Basse** | Impact mineur, pas d'urgence | < 4h | < 1 semaine |

## On-call (astreinte)

### Structure minimale

- Rotation explicite : planning connu à l'avance, au moins 2 semaines
- Escalade définie : si l'on-call ne répond pas dans N minutes → qui est alerté ensuite
- Handoff documenté : à chaque changement de rotation, les incidents en cours sont transmis
- Compensation : la garde est reconnue (financièrement ou en récupération)

### Qualité des alertes on-call

Une alerte on-call doit être :
- **Actionnable** : déclenche une action humaine, pas juste informationnelle
- **Urgente** : ne peut pas attendre le lendemain matin
- **Précise** : pointe vers la cause probable et le runbook

Seuil de tolérance : **< 2 alertes actionnables par nuit de garde** en moyenne. Au-delà, revoir les seuils.

## Runbooks

Un runbook est une procédure pas-à-pas pour répondre à un type d'incident spécifique.

### Structure minimale d'un runbook

```markdown
# Runbook : [Nom de l'alerte]

## Contexte
Ce que surveille cette alerte et pourquoi c'est important.

## Diagnostic
1. Vérifier [X] : `kubectl get pods -n production`
2. Consulter le dashboard : [lien Grafana]
3. Chercher dans les logs : `grep "ERROR" /var/log/app.log | tail -50`

## Actions correctives
- Si [symptôme A] : `kubectl rollout restart deployment/app`
- Si [symptôme B] : augmenter la capacité du pool de connexions DB

## Escalade
Si non résolu en 30 min : contacter [Nom] via [canal].

## Dernière mise à jour
2024-03 — [auteur]
```

## Postmortems blameless

### Principes

- **Blameless** : l'objectif est d'améliorer le système, pas de sanctionner des personnes. Les personnes ont fait de leur mieux avec les informations qu'elles avaient.
- **Causaux** : chercher la cause racine (5 Pourquoi) et les facteurs systémiques, pas les erreurs humaines de surface
- **Actionnables** : chaque postmortem produit des actions concrètes avec propriétaire et date

### Structure d'un postmortem

```markdown
# Postmortem — [Nom de l'incident] — [Date]

## Impact
- Durée : de HH:MM à HH:MM (Xh)
- Utilisateurs impactés : ~N (X% de la base)
- Services affectés : [liste]

## Timeline
- HH:MM — Première alerte reçue par [qui]
- HH:MM — Diagnostic initial : [hypothèse]
- HH:MM — Action corrective appliquée
- HH:MM — Résolution confirmée

## Causes racines (5 Pourquoi)
1. Pourquoi l'incident s'est-il produit ? → [réponse]
2. Pourquoi [réponse] s'est-il produit ? → ...

## Facteurs contributifs (pas des coupables)
- Monitoring insuffisant sur [composant]
- Runbook absent pour ce scénario

## Actions
| Action | Propriétaire | Date cible |
|--------|-------------|------------|
| Ajouter alerte sur [métrique] | @alice | 2024-04-15 |
| Créer runbook [X] | @bob | 2024-04-20 |
```

### Cadence recommandée

- P0/P1 : postmortem obligatoire dans les 5 jours ouvrés
- P2 récurrents (3x le même mois) : postmortem recommandé
- Revue des actions de postmortems : mensuelle

## Ce que l'agent IA peut vérifier

```bash
# Existence de runbooks
find . -path "*/runbooks/*.md" -o -path "*/docs/runbooks/*.md" | head -10
ls docs/runbooks/ ops/runbooks/ 2>/dev/null

# Postmortems disponibles
find . -name "*postmortem*" -o -name "*post-mortem*" -o -name "*incident*" | \
  grep -i "\.md$" | head -10

# Configuration des alertes (Prometheus alertmanager)
find . -name "alertmanager.yml" -o -name "alerts.yaml" -o -name "*.rules.yaml"
```

### Questions d'audit (entretien)

1. Décrivez la gestion du dernier incident P1. Qui a été alerté ? Comment ?
2. Y a-t-il un postmortem écrit ? Est-il accessible à toute l'équipe ?
3. Les actions des postmortems sont-elles suivies jusqu'à résolution ?
4. Qui est on-call cette semaine ? Comment le sait-on ?
5. Combien d'alertes reçoit l'on-call en moyenne par nuit ?
6. Y a-t-il des runbooks pour les incidents les plus fréquents ?

## Références

- [Google SRE Book — Incident Management](https://sre.google/sre-book/managing-incidents/)
- [Google SRE Book — Postmortem Culture](https://sre.google/sre-book/postmortem-culture/)
- [PagerDuty Incident Response Guide](https://response.pagerduty.com/)
- [Etsy — Blameless PostMortems](https://www.etsy.com/codeascraft/blameless-postmortems/)
