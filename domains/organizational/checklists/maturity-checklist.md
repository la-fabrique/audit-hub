---
domain: organizational
category: maturity
applicable-to: [startup, scaleup, enterprise, team]
tags: [maturité, gouvernance, processus, culture, on-call, documentation]
related-knowledge: ../tech-maturity.md
ai-selection-criteria: |
  Inclure dans tous les audits organisationnels. Bonus de priorité +2 si l'équipe
  a grandi rapidement (> 50% en 12 mois). +3 si des incidents récents ont révélé
  des lacunes organisationnelles (bus factor, manque de runbooks).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Maturité organisationnelle

## Processus de développement

- [ ] Un processus de revue de code est défini et systématiquement suivi
- [ ] Les déploiements en production sont traçables (qui, quand, quoi)
- [ ] Les déploiements sont réversibles (rollback documenté et testé)
- [ ] La definition of done inclut les tests et la documentation
- [ ] Des rétrospectives régulières sont conduites et produisent des actions concrètes

## Gestion des incidents

- [ ] Un niveau de sévérité des incidents est défini (P0→P3 avec SLA)
- [ ] Un processus on-call est formalisé (planning, escalade, compensation)
- [ ] Des runbooks existent pour les incidents les plus fréquents
- [ ] Les incidents P1+ donnent lieu à un postmortem dans les 5 jours
- [ ] Les postmortems sont blameless et partagés avec toute l'équipe
- [ ] Les actions des postmortems sont suivies jusqu'à résolution

## Knowledge management

- [ ] L'onboarding d'un nouveau développeur est documenté (< 1 jour pour être opérationnel)
- [ ] Le bus factor des systèmes critiques est > 2
- [ ] La documentation technique est accessible et à jour
- [ ] Les décisions techniques majeures sont tracées (ADR ou équivalent)
- [ ] Le savoir n'est pas concentré sur 1-2 personnes clés

## Gouvernance technique

- [ ] La dette technique est visible dans le backlog et priorisée
- [ ] Du temps est alloué à l'amélioration technique (non-feature work : ≥ 15-20%)
- [ ] Les choix technologiques majeurs font l'objet d'une validation explicite
- [ ] Des métriques de performance d'équipe sont suivies (DORA ou équivalent)
- [ ] Une roadmap technique est définie et partagée avec le management

## Culture et collaboration

- [ ] Les équipes Dev et Ops collaborent sur les incidents (pas de blame croisé)
- [ ] La sécurité est intégrée dans le flux de développement (pas seulement en fin de cycle)
- [ ] Les personnes peuvent signaler des problèmes sans crainte de répercussions
- [ ] L'expérimentation est encouragée (feature flags, spikes, POC)
- [ ] La montée en compétences est soutenue (formation, conférences, temps d'apprentissage)

## Score d'évaluation

| % items cochés | Score | Niveau de maturité |
|---|---|---|
| ≥ 90 % | 5 | Niveau 4-5 — Gérant et optimisant |
| 70–89 % | 4 | Niveau 3 — Processus définis et suivis |
| 50–69 % | 3 | Niveau 2-3 — Défini mais fragile |
| 30–49 % | 2 | Niveau 2 — Répétable |
| < 30 % | 1 | Niveau 1 — Réactif, dépendant des individus |
