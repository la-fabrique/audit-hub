---
domain: architecture
category: documentation
severity: medium
applicable-to: [web, api, saas, cloud, microservices, monolith]
tags: [documentation, diagrammes, uml, c4, archimate, drift, dette-documentaire]
related:
  - ./frameworks-and-standards.md
  - ./checklists/documentation-checklist.md
ai-prompt-hints: |
  Demander : où est stockée la documentation d'architecture, quand a-t-elle été
  mise à jour pour la dernière fois, qui la maintient.
  Chercher : présence d'un ARCHITECTURE.md, de diagrammes dans /docs, d'ADR,
  de README avec contexte système.
  Red flags : documentation uniquement dans les têtes des développeurs seniors,
  diagrammes dans des outils déconnectés du repo (Confluence non lié, PPT),
  pas de description des flux de données.
---

# Documentation et diagrammes d'architecture

## Risques principaux

| Risque | Impact | Fréquence |
|--------|--------|-----------|
| Documentation absente — connaissance dans les têtes | Haut | Très élevée |
| Diagrammes en décalage avec la réalité du code (doc drift) | Haut | Élevée |
| Flux de données non documentés (impact RGPD, debug) | Critique | Élevée |
| Décisions architecturales non tracées | Moyen | Très élevée |
| Documentation uniquement dans des outils déconnectés du repo | Moyen | Élevée |
| Aucune documentation d'onboarding pour les nouveaux devs | Moyen | Élevée |

## Niveaux de documentation recommandés

### Niveau minimum (tout projet)

- `README.md` : description du système, prérequis, lancement local
- `ARCHITECTURE.md` (ou `docs/architecture.md`) : vue d'ensemble, choix techniques majeurs
- Diagramme de contexte C4 L1 : le système et ses dépendances externes
- `docs/adr/` : décisions architecturales majeures (3+ minimum)

### Niveau recommandé (équipes > 3 personnes)

- Diagramme de conteneurs C4 L2 : services, bases de données, interfaces
- Flux de données sensibles documentés (pour RGPD : données personnelles)
- Runbook de déploiement et de rollback
- Diagramme de séquence pour les flux critiques (auth, paiement…)

### Niveau avancé (systèmes critiques)

- Diagrammes-as-code versionables (Structurizr DSL, PlantUML, Mermaid)
- Architecture testée (fitness functions, ArchUnit)
- Documentation auto-générée depuis le code (Swagger/OpenAPI, AsyncAPI)
- Revue d'architecture périodique documentée

## Bonne pratique : docs-as-code

Les diagrammes et documentation stockés dans le repo Git permettent :
- Revue de code des changements architecturaux
- Historique des évolutions
- Cohérence avec le code (même PR)
- Génération automatique en CI

```bash
# Exemple : valider les diagrammes PlantUML en CI
plantuml -checkonly docs/**/*.puml

# Générer les diagrammes depuis Structurizr DSL
docker run --rm -v $(pwd):/data structurizr/cli export -workspace docs/architecture.dsl -format png
```

## Détection du doc drift

Le décalage entre documentation et réalité est un problème structurel sans outil dédié.

### Signaux de drift à vérifier

```bash
# Documentation modifiée il y a plus de 90 jours alors que le code a changé
git log --since="90 days ago" --oneline -- "*.md" docs/
git log --since="90 days ago" --oneline -- "src/" "lib/"

# Comparer les services documentés vs services déployés
# (à faire manuellement ou avec un script)

# Vérifier si les ADR couvrent les technologies détectées dans le code
grep -rn "import.*redis\|require.*redis" --include="*.ts" --include="*.py"
# → Redis est-il documenté dans l'architecture ?
```

### Questions pour détecter le drift

1. Ce diagramme a-t-il été mis à jour lors de la dernière refactorisation majeure ?
2. Le service X visible dans le code apparaît-il dans les diagrammes de conteneurs ?
3. Les flux de données documentés correspondent-ils aux logs de production ?

## Ce que l'agent IA peut vérifier

### Analyse de la structure documentaire

```bash
# Inventaire de la documentation existante
find . -name "*.md" | grep -i "arch\|design\|adr\|decision\|overview" | head -20
find . -name "*.puml" -o -name "*.dsl" -o -name "*.drawio" 2>/dev/null

# Présence d'une documentation minimale
for f in README.md ARCHITECTURE.md docs/architecture.md; do
  [ -f "$f" ] && echo "✓ $f" || echo "✗ $f manquant"
done

# Fraîcheur de la documentation
git log --format="%ar %s" -1 -- README.md ARCHITECTURE.md docs/

# Nombre d'ADR
ls docs/adr/*.md 2>/dev/null | wc -l
```

### Questions d'audit (entretien)

1. Où trouve-t-on la documentation d'architecture ? Est-elle dans le repo ?
2. Qui est responsable de maintenir les diagrammes à jour ?
3. Les nouveaux développeurs comprennent-ils le système en lisant la documentation seule ?
4. Y a-t-il des flux de données impliquant des données personnelles documentés ?
5. Comment les décisions d'architecture majeures sont-elles tracées ?
6. La documentation est-elle révisée lors des code reviews ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Génération de diagrammes depuis le code | Structurizr, Mermaid, PlantUML | Valider la cohérence des diagrammes générés avec la description du projet |
| Détection de doc drift automatique | Swimm, Docusaurus avec linting | Analyser le rapport de drift produit |
| Documentation API | Swagger UI, Redoc (sur fichier OpenAPI fourni) | Lire et auditer la spec OpenAPI |
| Cartographie des flux de données | Radar (ThoughtWorks), Datamap | Analyser le rapport de flux pour identifier les données personnelles |

## Références

- [C4 Model — Documentation as Code](https://c4model.com/)
- [Architecture Decision Records (ADR) — Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Docs as Code — Write the Docs](https://www.writethedocs.org/guide/docs-as-code/)
- [Fitness Functions — Building Evolutionary Architectures](https://www.thoughtworks.com/books/building-evolutionary-architectures)
