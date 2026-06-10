---
domain: organizational
category: gouvernance
applicable-to: [startup, scaleup, enterprise, team]
tags: [gouvernance, raci, conformité, cobit, processus, documentation, rgpd]
related-knowledge: ../team-structure.md
ai-selection-criteria: |
  Inclure si l'audit couvre la gouvernance ou la conformité réglementaire. Bonus
  de priorité +3 si l'organisation est soumise à des obligations réglementaires
  (RGPD, ISO 27001, SOC2, HDS). +2 si une croissance rapide a créé des flous
  de responsabilité.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Gouvernance IT et organisation

## Responsabilités et décisions

- [ ] Un organigramme technique est disponible et à jour
- [ ] Les rôles et responsabilités sont documentés (RACI ou équivalent)
- [ ] Il est clair qui est responsable de chaque système en production
- [ ] Le processus de prise de décision technique est défini (qui décide quoi)
- [ ] Les décisions architecturales majeures sont documentées (ADR)

## Conformité réglementaire

- [ ] Les exigences réglementaires applicables sont identifiées (RGPD, NIS2, HDS, PCI-DSS…)
- [ ] Un responsable de la conformité est désigné (DPO pour le RGPD si applicable)
- [ ] Les données personnelles traitées sont inventoriées (registre des traitements RGPD)
- [ ] Les sous-traitants accédant à des données personnelles ont des DPA signés
- [ ] Les délais de conservation des données sont définis et appliqués

## Gestion des risques

- [ ] Un registre des risques IT existe et est maintenu
- [ ] Les risques critiques ont un plan de mitigation documenté
- [ ] Un plan de continuité d'activité (PCA) ou de reprise (PRA) est défini
- [ ] Le PCA/PRA a été testé dans les 12 derniers mois
- [ ] Les accès privilégiés sont revus régulièrement (au moins trimestriellement)

## Processus et procédures

- [ ] Les procédures de déploiement sont documentées
- [ ] Les procédures de gestion des incidents sont documentées
- [ ] Les procédures de gestion des accès (onboarding/offboarding) sont définies
- [ ] Les changements en production font l'objet d'un processus de validation
- [ ] Un processus de gestion des vulnérabilités est en place (scan → triage → correction)

## Audit et traçabilité

- [ ] Les actions des comptes admin sont journalisées
- [ ] Les accès aux données sensibles sont tracés
- [ ] Les logs d'audit sont conservés selon les exigences légales (6 mois minimum, 1 an recommandé)
- [ ] Les logs ne peuvent pas être supprimés par les utilisateurs qu'ils concernent
- [ ] Des revues de logs sont effectuées régulièrement

## Score d'évaluation

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
