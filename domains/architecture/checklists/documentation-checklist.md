---
domain: architecture
category: documentation
applicable-to: [web, api, saas, cloud, microservices, monolith]
tags: [documentation, diagrammes, adr, onboarding, drift]
related-knowledge: ../documentation-and-diagrams.md
ai-selection-criteria: |
  Inclure cette checklist si : l'audit couvre l'architecture ou la maintenabilité.
  Bonus de priorité +2 si l'équipe a eu des difficultés d'onboarding récentes ou
  des incidents difficiles à débugger. +2 si contexte réglementaire (RGPD exige
  de documenter les flux de données personnelles).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Documentation d'architecture

## Documentation minimale

- [ ] Un `README.md` à la racine décrit le système, ses prérequis et le lancement local
- [ ] Un fichier d'architecture (`ARCHITECTURE.md` ou `docs/architecture.md`) existe
- [ ] Les choix technologiques majeurs sont justifiés (dans l'architecture ou les ADR)
- [ ] Les dépendances externes (services tiers, APIs) sont toutes référencées

## Diagrammes

- [ ] Un diagramme de contexte (C4 L1 ou équivalent) montre le système et ses acteurs/systèmes externes
- [ ] Un diagramme de conteneurs (C4 L2 ou équivalent) liste les services, bases de données et interfaces
- [ ] Les diagrammes sont stockés dans le repo (pas uniquement dans Confluence ou PPT)
- [ ] Les diagrammes ont été mis à jour lors du dernier changement architectural majeur (< 6 mois)
- [ ] Pour les flux critiques (auth, paiement), des diagrammes de séquence existent

## Architecture Decision Records (ADR)

- [ ] Un répertoire `docs/adr/` (ou équivalent) existe
- [ ] Les décisions majeures sont documentées (choix de framework, base de données, pattern d'auth…)
- [ ] Chaque ADR indique : contexte, décision, alternatives considérées, conséquences
- [ ] Les ADR obsolètes sont marqués "Superseded" (pas supprimés)
- [ ] Les ADR sont revus lors des code reviews quand une décision est remise en question

## Flux de données

- [ ] Les flux impliquant des données personnelles sont documentés (exigence RGPD)
- [ ] Les données en transit et au repos sont identifiées dans la documentation
- [ ] Les intégrations avec des tiers recevant des données sensibles sont documentées

## Onboarding et maintenabilité

- [ ] Un nouveau développeur peut lancer l'environnement de développement en moins de 30 minutes en suivant la doc seule
- [ ] Les concepts métier clés sont expliqués dans la documentation (glossaire ou section dédiée)
- [ ] La documentation est révisée lors des code reviews (au moins pour les changements architecturaux)
- [ ] Un runbook de déploiement et de rollback existe

## Freshness (fraîcheur)

- [ ] La documentation a été modifiée dans les 3 derniers mois (ou lors de la dernière évolution majeure)
- [ ] Il n'existe pas de sections marquées "TODO" ou "à compléter" sans date de résolution
- [ ] Les versions des composants mentionnées dans la doc correspondent aux versions déployées

## Score d'évaluation

Compter le nombre de cases cochées :

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
