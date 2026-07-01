---
domain: infra-devops
category: backup-recovery
applicable-to: [web, api, saas, cloud, on-premise, infra]
tags: [backup, recovery, rpo, rto, disaster-recovery]
related-knowledge: ../backup-recovery.md
ai-selection-criteria: |
  Inclure cette checklist si l'organisation opère une infrastructure avec
  des données persistantes. Bonus de priorité +3 si regulatory contient HDS,
  ISO27001 ou PCI-DSS (voir domains/infra-devops/backup-recovery.md
  regulatory-relevance).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Sauvegarde et reprise d'activité

## Politique de sauvegarde

- [ ] `[B]` Une politique de sauvegarde existe (fréquence, rétention, périmètre)
- [ ] `[B]` Les backups ne sont pas stockés sur le même support que les données sources
- [ ] Les backups sont chiffrés au repos

## RPO / RTO

- [ ] Un RPO (Recovery Point Objective) cible est défini
- [ ] Un RTO (Recovery Time Objective) cible est défini
- [ ] Le RPO/RTO réel a été mesuré lors d'un exercice de restauration

## Tests de restauration

- [ ] `[B]` Une restauration complète a été testée avec succès dans les 6 derniers mois
- [ ] Le processus de restauration est documenté et accessible sans dépendre de la
      personne qui l'a écrit

## Score d'évaluation

Compter le nombre de cases cochées. Les items `[B]` sont éliminatoires : si
un seul est en échec, le score est plafonné à 2.

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
