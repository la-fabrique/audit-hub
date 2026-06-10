---
domain: organizational
category: maturity
severity: high
applicable-to: [startup, scaleup, enterprise, team]
tags: [maturité, gouvernance, processus, culture, organisation, dette-organisationnelle, on-call]
related:
  - ./team-structure.md
  - ./checklists/maturity-checklist.md
  - ../../templates/questionnaires/org-questionnaire.md
ai-prompt-hints: |
  Ce domaine s'évalue par entretiens et observation, pas seulement par le code.
  Demander : organigramme tech, processus de release, gestion des incidents,
  processus d'onboarding, documentation disponible, rétrospectives, postmortems.
  Observer : comment les décisions techniques sont prises, qui peut déployer en prod,
  comment les incidents sont gérés, si la dette est visible et priorisée.
  Piège fréquent : des outils modernes (K8s, microservices) masquent une maturité organisationnelle faible.
---

# Maturité organisationnelle tech

## Modèle de maturité (5 niveaux)

### Niveau 1 — Réactif (chaos géré)
- Pas de processus formalisé, tout dépend des individus clés
- Incidents gérés de manière ad hoc, pas de postmortems
- Documentation absente ou obsolète
- Silos forts entre Dev et Ops

### Niveau 2 — Répétable
- Quelques processus définis mais non systématiquement suivis
- Documentation partielle des systèmes critiques
- Incidents loggés mais pas toujours analysés
- Déploiements manuels mais avec checklist

### Niveau 3 — Défini
- Processus documentés et suivis (PR review, déploiements, incidents)
- On-call structuré avec runbooks
- Postmortems blameless sur les incidents significatifs
- Onboarding documenté (< 2 jours pour être opérationnel)

### Niveau 4 — Géré
- Métriques suivies et utilisées pour les décisions (DORA, SLO, coverage)
- Dette technique visible et budgétée
- Équipes autonomes sur leur domaine (you build it, you run it)
- Revues d'architecture régulières

### Niveau 5 — Optimisé
- Amélioration continue mesurée et pilotée
- Expérimentation systématique (feature flags, A/B testing, chaos engineering)
- Partage de connaissances structuré (guildes, RFCs, ADRs)
- Contribution à l'open source ou à la communauté

## Dimensions d'évaluation

### Gouvernance technique
- Les décisions techniques sont-elles documentées ? (ADR — Architecture Decision Records)
- Y a-t-il un processus de RFC pour les changements structurants ?
- Comment la roadmap technique est-elle priorisée par rapport au produit ?

### Gestion des incidents
- Y a-t-il une astreinte (on-call) formalisée ?
- Les incidents ont-ils des SLA de réponse définis ?
- Les postmortems sont-ils blameless et partagés ?
- Y a-t-il des runbooks pour les incidents courants ?

### Knowledge management
- La documentation est-elle à jour et accessible ?
- Comment le savoir est-il transféré quand quelqu'un quitte l'équipe ?
- Y a-t-il des "bus factor" identifiés (une seule personne sait faire X) ?

### Culture et processus
- Les équipes font-elles des rétrospectives ?
- La dette technique est-elle visible dans le backlog ?
- Y a-t-il du temps alloué à l'amélioration technique (non-feature work) ?

## Ce que l'agent IA peut vérifier

Ce domaine s'évalue surtout par entretiens, mais certains signaux sont
observables dans le dépôt si `access.code_repo` est disponible.

### Signaux observables dans le dépôt

```bash
# Présence d'ADRs / RFCs (décisions techniques documentées)
find . -ipath "*adr*" -o -ipath "*rfc*" -o -ipath "*decisions*" -name "*.md" | head

# Documentation d'onboarding, runbooks, guide de contribution
find . -iname "*onboarding*" -o -iname "*runbook*" -o -iname "CONTRIBUTING*" | head

# Bus factor : concentration des contributions sur 12 mois
git shortlog -sn --since="12 months ago" | head -10

# Fraîcheur de la documentation
git log -1 --format="%ar" -- docs/ README.md

# CODEOWNERS définis (responsabilités explicites)
ls .github/CODEOWNERS CODEOWNERS 2>/dev/null
```

### Questions d'audit (entretien)

1. Combien de temps prend l'onboarding d'un nouveau développeur avant d'être opérationnel ?
2. Qui peut déployer en production ? Y a-t-il un processus ?
3. Comment les incidents sont-ils gérés ? Y a-t-il des postmortems ?
4. Quelle est la documentation disponible sur les systèmes critiques ?
5. Y a-t-il des personnes dont le départ mettrait l'équipe en grande difficulté ?
6. Comment les décisions techniques importantes sont-elles prises et documentées ?
7. Quel % du temps d'ingénierie est alloué à la réduction de la dette technique ?

## Signaux d'alerte organisationnels

- **Bus factor = 1** sur des systèmes critiques
- Déploiements en prod uniquement possibles par 1-2 personnes
- Pas de postmortems, ou postmortems qui cherchent un coupable
- Documentation qui n'a pas été mise à jour depuis > 6 mois
- Rétrospectives qui ne produisent jamais d'actions concrètes
- Équipe qui ne peut pas se permettre de faire du refactoring car "trop de features à livrer"

## Références

- [Team Topologies — Matthew Skelton & Manuel Pais](https://teamtopologies.com/)
- [Accelerate — Nicole Forsgren, Jez Humble, Gene Kim](https://itrevolution.com/accelerate-book/)
- [SRE Book — Google](https://sre.google/sre-book/)
- [Architecture Decision Records (ADR)](https://adr.github.io/)
