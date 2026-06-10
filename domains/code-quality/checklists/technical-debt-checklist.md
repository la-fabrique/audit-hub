---
domain: code-quality
category: dette-technique
applicable-to: [all]
tags: [dette-technique, complexité, duplication, refactoring, sonarqube, legacy, coverage]
related-knowledge: ../technical-debt.md
ai-selection-criteria: |
  Inclure dans les audits couvrant la qualité de code, notamment si `audit.depth`
  = standard ou comprehensive. Bonus de priorité +3 si l'équipe identifie des
  modules que "personne ne veut toucher" ou si la vélocité a chuté sans raison
  fonctionnelle. +2 si le ratio dette SonarQube est > Grade C ou si la couverture
  de tests est < 40%.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Dette technique

## Mesure et visibilité

- [ ] La dette technique est mesurée par un outil (SonarQube, CodeClimate, NDepend…)
- [ ] Le rapport de dette est accessible à toute l'équipe (pas seulement aux leads)
- [ ] Les métriques clés sont suivies : complexité cyclomatique, duplication, ratio de dette
- [ ] Les hotspots (forte complexité × fort churn) sont identifiés

## Gestion dans le backlog

- [ ] `[B]` La dette technique est visible dans le backlog de l'équipe (pas uniquement dans les TODO du code)
- [ ] Du temps est alloué à la réduction de dette (≥ 15% du sprint ou sprints dédiés trimestriels)
- [ ] La priorisation de la dette suit un critère explicite (impact × fréquence de modification)
- [ ] Le budget de dette est connu du management

## Qualité du code existant

- [ ] La complexité cyclomatique moyenne par méthode est ≤ 10 (seuil d'alerte : > 15)
- [ ] La duplication de code est < 10% (cible : < 3%)
- [ ] Aucun fichier de plus de 500 lignes sans justification architecturale
- [ ] Les TODO/FIXME dans le code sont liés à des tickets dans le backlog

## Code legacy et tests

- [ ] Le code legacy critique est couvert par des tests (même de caractérisation)
- [ ] Les modules sans test sont identifiés et une stratégie de couverture est définie
- [ ] Les modules sans modification depuis > 18 mois sont évalués (code mort ou zone stable ?)

## Pratiques d'équipe

- [ ] La définition of done inclut l'absence de régression sur les métriques de dette
- [ ] Les nouvelles PR ne peuvent pas dégrader le ratio de dette en dessous du seuil cible (quality gate)
- [ ] Un refactoring est systématiquement envisagé avant d'ajouter une feature dans un module à forte dette

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
