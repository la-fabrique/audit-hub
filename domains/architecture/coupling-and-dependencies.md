---
domain: architecture
category: couplage
severity: high
applicable-to: [web, api, saas, microservices, monolith]
tags: [couplage, dépendances, modularité, dette-technique, solid, cohésion]
related:
  - ./scalability-patterns.md
  - ./frameworks-and-standards.md
  - ./checklists/architecture-review-checklist.md
ai-prompt-hints: |
  Demander à voir : diagramme de dépendances entre modules/services, liste des
  bibliothèques tierces, structure du monorepo ou des repos séparés.
  Chercher dans le code : imports circulaires, classes god-object, couches qui
  s'appellent dans les deux sens, logique métier dans les contrôleurs ou la base de données.
  Red flags immédiats : modules avec > 20 dépendances sortantes, cycles de dépendances,
  absence de contrats d'interface (tout en concret), couplage à des implémentations tierces sans abstraction.
---

# Couplage et dépendances

## Risques principaux

| Risque | Impact | Fréquence |
|--------|--------|-----------|
| Imports circulaires entre modules | Haut | Élevée |
| God objects / classes de plus de 500 lignes | Moyen | Très élevée |
| Couplage fort à une technologie tierce sans abstraction | Haut | Élevée |
| Logique métier dans les couches d'infrastructure (ORM, contrôleurs) | Haut | Élevée |
| Absence de séparation des couches (architecture en lasagne) | Critique | Moyenne |
| Dépendances transitives non maîtrisées (supply chain) | Critique | Faible |

## Meilleures pratiques

### Principes SOLID appliqués à l'architecture

- **Responsabilité unique** : chaque module a une raison de changer. Vérifier : si A change, quels modules recompilent ?
- **Inversion de dépendance** : les modules de haut niveau ne dépendent pas des implémentations — utiliser des interfaces/ports
- **Ouvert/fermé** : extensions par ajout, pas modification — limiter les `switch` sur des types

### Séparation des couches

```
Présentation → Application → Domaine ← Infrastructure
```

- La couche Domaine ne connaît pas la DB, le framework web, ni les services externes
- Les appels DB passent par des repositories avec interface définie dans le domaine
- Les services externes (email, paiement) passent par des adaptateurs

### Gestion des dépendances tierces

- Inventaire exhaustif via `npm audit`, `pip-audit`, `mvn dependency:tree`, `cargo audit`
- Politique de mise à jour : patch automatique, minor révisé, major intentionnel
- Abstraction des librairies à fort couplage (ex : ORM, SDK cloud) derrière une interface interne
- Pas de version fixe sans `lock file` versionné (`package-lock.json`, `poetry.lock`)

### Métriques de couplage

| Métrique | Seuil alerte | Outil |
|---|---|---|
| Afférent coupling (Ca) — nb de modules qui dépendent d'un module | > 15 | Structure101, SonarQube |
| Efférent coupling (Ce) — nb de modules dont dépend un module | > 10 | Structure101 |
| Instabilité I = Ce / (Ca + Ce) | Dépend du rôle (stable < 0.3) | Calculé |
| Complexité cyclomatique par méthode | > 10 | SonarQube, Lizard |
| Longueur de classe (LOC) | > 400 | SonarQube |

## Ce que l'agent IA peut vérifier

### Analyse statique du code

```bash
# Détecter les imports circulaires (Node.js)
npx madge --circular --extensions ts src/

# Dépendances Python
pip install pydeps && pydeps <module> --max-bacon 3

# Taille des classes Java
grep -rn "^public class\|^class " --include="*.java" | wc -l

# Modules avec trop d'imports
find . -name "*.ts" -exec sh -c 'echo "$(grep -c "^import" "$1") $1"' _ {} \; | sort -rn | head -20

# Détection des god objects (fichiers > 400 lignes)
find . -name "*.py" -o -name "*.ts" -o -name "*.java" | xargs wc -l | sort -rn | head -20

# Dépendances vulnérables
npm audit --json
pip-audit --format json
```

### Questions d'audit (entretien)

1. Peut-on changer de base de données sans modifier la logique métier ?
2. Comment sont gérées les mises à jour des dépendances tierces ?
3. Y a-t-il un diagramme de dépendances entre services/modules à jour ?
4. Quels modules sont les plus difficiles à modifier sans casser autre chose ?
5. Existe-t-il des couches d'abstraction entre le domaine et les services externes ?
6. Comment est géré le couplage entre microservices (contrats d'API, versioning) ?

### Revue de code / documentation

- Examiner la structure des répertoires : séparation domaine / application / infra ?
- Lire les `package.json`, `pom.xml`, `requirements.txt` : dépendances directes vs transitives
- Vérifier l'existence de tests unitaires sans dépendances externes (tests rapides < 1s)

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Cartographie complète des dépendances | Structure101, Lattix, NDepend | Analyser le rapport exporté, identifier les clusters problématiques |
| Analyse de couplage multilangage | SonarQube avec plugins | Lire les métriques Ca/Ce exportées en JSON |
| Audit de sécurité des dépendances | Snyk, Dependabot, OWASP Dependency-Check | Contextualiser les CVE selon la stack et l'exposition |
| Détection des imports circulaires visuels | Madge (JS), Graphviz | Analyser le graphe exporté |

## Références

- [SOLID Principles — Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html)
- [Coupling and Cohesion — Martin Fowler](https://martinfowler.com/ieeeSoftware/coupling.pdf)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [The Twelve-Factor App — Dependencies](https://12factor.net/dependencies)
