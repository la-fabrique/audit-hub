---
domain: code-quality
category: dette-technique
severity: medium
applicable-to: [all]
tags: [dette-technique, complexité, duplication, refactoring, sonarqube, codecoverage, legacy]
related:
  - ./code-review.md
  - ./static-analysis.md
  - ./checklists/code-review-checklist.md
ai-prompt-hints: |
  Demander : rapport SonarQube ou CodeClimate disponible, estimation de la dette
  en jours, politique de remboursement (time box, ratio).
  Chercher : fichiers jamais modifiés depuis > 1 an, classes > 500 lignes,
  TODO/FIXME sans tickets, complexité cyclomatique > 15, duplication > 5%.
  Red flags : dette technique jamais trackée, pas de budget alloué, refactoring
  absent de la définition of done, code legacy sans tests.
---

# Dette technique

## Définition et types

La dette technique est le coût futur causé par des choix de conception ou d'implémentation sous-optimaux. Elle se manifeste sous plusieurs formes :

| Type | Symptômes visibles | Coût si non traité |
|------|-------------------|--------------------|
| **Dette de code** | Complexité élevée, code dupliqué, nommage obscur | Ralentissement des développements |
| **Dette de tests** | Coverage faible, tests fragiles | Régressions fréquentes, peur des changements |
| **Dette d'architecture** | Couplage fort, monolithe non modulaire | Impossibilité de scaler ou migrer |
| **Dette de dépendances** | Versions obsolètes, vulnérabilités connues | Failles de sécurité, incompatibilités |
| **Dette de documentation** | Connaissance tacite, diagrammes obsolètes | Bus factor élevé, onboarding long |

## Métriques clés

### Métriques SonarQube

| Métrique | Cible | Seuil d'alerte |
|----------|-------|----------------|
| **Complexité cyclomatique** par méthode | ≤ 10 | > 15 |
| **Duplication** | < 3% | > 10% |
| **Dette technique totale** | < 5% du coût de dev | > 20% |
| **Ratio dette** (dette / effort total estimé) | Grade A (< 5%) | Grade D/E (> 20%) |
| **Code smells** par KLOC | < 5 | > 20 |
| **Cognitive complexity** | ≤ 15 | > 25 |

### Indicateurs git

```bash
# Fichiers les plus souvent modifiés (churn élevé = instabilité)
git log --name-only --format="" | grep -v "^$" | sort | uniq -c | sort -rn | head -20

# Fichiers jamais touchés depuis longtemps (code mort potentiel)
git log --diff-filter=M --name-only --format="" -- "*.py" "*.ts" | sort | uniq -c | sort -n | head -20

# Auteurs qui ont travaillé sur chaque fichier (bus factor)
git log --format="%aN" -- src/core/critical-module.ts | sort -u | wc -l

# TODO et FIXME sans ticket associé
grep -rn "TODO\|FIXME\|HACK\|XXX" --include="*.py" --include="*.ts" --include="*.java" | \
  grep -v "#[0-9]\{3,\}\|JIRA\|GH-" | wc -l
```

## Gestion de la dette

### Stratégies de remboursement

**Boy Scout Rule** (continu) : chaque PR laisse le code légèrement meilleur. Efficace pour la dette diffuse, nécessite une culture d'équipe.

**Time box** (planifié) : allouer 20% du sprint à la réduction de dette. Mesurable, prévisible, mais peut être sacrifié sous pression.

**Refactoring opportuniste** : refactoriser avant d'ajouter une feature dans un module donné. Naturel, mais couvre inégalement la base de code.

**Sprints dédiés** : sprints techniques 1 fois par trimestre. Visible pour le management, mais crée une séparation artificielle.

### Priorisation de la dette

Prioriser par : **impact × fréquence de modification**

```
Impact élevé + modifié souvent = dette prioritaire
Impact élevé + jamais modifié = acceptable (zone stable)
Impact faible + modifié souvent = à traiter en boy scout
Impact faible + jamais modifié = ignorer
```

### Tracking minimal recommandé

```markdown
<!-- Dans chaque TODO/FIXME -->
# TODO(2024-03): Remplacer par le pattern Repository — ticket #1234
# FIXME: Contournement pour le bug de l'API tierce — supprimer après migration v3
```

- Tag les fichiers à forte dette dans SonarQube (labels, projets)
- Backlog de dette technique visible (pas enterré dans les TODO du code)
- Revue trimestrielle des métriques de dette

## Ce que l'agent IA peut vérifier

### Détection directe dans le code

```bash
# Complexité par fichier (Lizard)
pip install lizard && lizard src/ --CCN 10 --length 50 --warnings_only

# Duplication (CPD — Copy-Paste Detector PMD)
# Java
pmd cpd --minimum-tokens 100 --dir src/

# Fichiers trop longs
find src -name "*.py" -o -name "*.ts" -o -name "*.java" | \
  xargs wc -l | sort -rn | awk '$1 > 400' | head -20

# TODO sans tickets
grep -rn "TODO\|FIXME" --include="*.py" --include="*.ts" --include="*.java" | head -30

# Fonctions trop longues (Python)
grep -n "def " src/**/*.py | head -5  # puis lire les fonctions > 30 lignes
```

### Questions d'audit (entretien)

1. La dette technique est-elle mesurée et visible (SonarQube, CodeClimate) ?
2. Quel budget est alloué à la réduction de dette par sprint/trimestre ?
3. Y a-t-il des modules que personne ne veut toucher ? Pourquoi ?
4. La définition of done inclut-elle l'absence de régression sur la dette ?
5. Les nouvelles features introduisent-elles systématiquement de la dette ?
6. Y a-t-il du code legacy sans tests ? Quelle est la stratégie pour le couvrir ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Rapport de dette global | SonarQube, CodeClimate, NDepend | Analyser le rapport JSON/CSV, identifier les hotspots |
| Visualisation du churn vs complexité | CodeScene | Analyser le rapport de risque produit |
| Analyse des "hotspots" (churn × complexité) | CodeScene, SonarQube | Prioriser les modules à refactoriser en premier |
| Coverage de tests sur le legacy | JaCoCo, istanbul, pytest-cov | Identifier les modules legacy non couverts |

## Références

- [Ward Cunningham — Technical Debt Metaphor](https://martinfowler.com/bliki/TechnicalDebt.html)
- [Martin Fowler — Refactoring](https://martinfowler.com/books/refactoring.html)
- [CodeScene — Behavioral Code Analysis](https://codescene.com/)
- [SonarQube — Technical Debt Ratio](https://docs.sonarsource.com/sonarqube/latest/user-guide/metric-definitions/)
- [ISO/IEC 25010 — Software Quality Model](https://www.iso.org/standard/35720.html)
