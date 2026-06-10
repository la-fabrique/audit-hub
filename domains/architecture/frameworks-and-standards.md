---
domain: architecture
category: frameworks-et-standards
severity: medium
applicable-to: [enterprise, web, api, saas, cloud, microservices]
tags: [togaf, archimate, c4, zachman, iso42010, architecture-enterprise, documentation]
related:
  - ./documentation-and-diagrams.md
  - ./coupling-and-dependencies.md
  - ./checklists/architecture-review-checklist.md
ai-prompt-hints: |
  Demander : quel cadre d'architecture est utilisé (ou aucun), qui est responsable
  de l'architecture, à quelle fréquence la documentation est mise à jour.
  Vérifier si des diagrammes existent (ArchiMate, UML, C4, ad hoc) et s'ils
  correspondent à la réalité du code.
  Red flags : aucune documentation d'architecture formelle, diagrammes datés de
  plus de 6 mois sans mise à jour, architecte absent ou rôle non défini.
---

# Frameworks et standards d'architecture

## Cadres d'architecture — comparatif

| Cadre | Usage principal | Complexité | Recommandé pour |
|-------|----------------|------------|-----------------|
| **C4 Model** | Logiciel (Contexte, Conteneurs, Composants, Code) | Faible | Équipes dev ≤ 50 personnes, documentation agile |
| **ArchiMate** | Langage de modélisation EA (métier, application, infra) | Modérée | Complémentaire à TOGAF, grandes orgs |
| **TOGAF (ADM)** | Architecture d'entreprise complète | Élevée | Grandes entreprises, alignement business/IT |
| **Zachman** | Classification des artefacts EA (matrice 6×6) | Modérée | Gouvernance documentaire, audit de complétude |
| **ISO/IEC 42010** | Standard de description architecturale | Modérée | Référence normative, certifications |

## Utilisation en contexte d'audit

### C4 Model (recommandé pour la plupart des projets)

Le C4 fournit 4 niveaux hiérarchiques vérifiables lors d'un audit :

1. **Contexte (L1)** : le système dans son environnement (utilisateurs, systèmes externes) — vérifier que les dépendances externes sont toutes représentées
2. **Conteneurs (L2)** : applications, bases de données, services — vérifier la cohérence avec le déploiement réel
3. **Composants (L3)** : modules internes à un conteneur — vérifier l'alignement avec la structure du code
4. **Code (L4)** : classes/fonctions — généralement pas maintenu, signaler si attendu

Outil recommandé : [Structurizr](https://structurizr.com/) (diagrams-as-code, versionnable).

### TOGAF — ce qu'un auditeur vérifie

Un audit TOGAF ne certifie pas l'adoption du cadre — il vérifie que les artefacts clés existent :

| Artefact TOGAF | Question d'audit |
|----------------|-----------------|
| Architecture Vision | Le système a-t-il une vision documentée avec périmètre et objectifs ? |
| Architecture Repository | Les décisions architecturales sont-elles tracées (ADR) ? |
| Gap Analysis | Les écarts entre architecture cible et actuelle sont-ils identifiés ? |
| Migration Planning | Y a-t-il une roadmap de migration vers l'architecture cible ? |

### Architecture Decision Records (ADR)

Indépendamment du cadre utilisé, les ADR sont la pratique minimale recommandée :

```markdown
# ADR-001 : Choix de PostgreSQL comme base principale

## Statut : Accepté (2024-03)
## Contexte : Besoin de transactions ACID, requêtes complexes, JSON semi-structuré
## Décision : PostgreSQL 15 avec extension JSONB
## Conséquences : Pas de multi-cloud natif, opérationnel à maîtriser
```

- Stockage : `docs/adr/` dans le repo, format MADR ou Nygard
- Revue : les ADR obsolètes doivent être marqués "Superseded" (pas supprimés)

## Indicateurs de maturité architecturale

| Indicateur | Niveau 1 (Initial) | Niveau 3 (Défini) | Niveau 5 (Optimisé) |
|---|---|---|---|
| Documentation | Inexistante ou ad hoc | C4 L1+L2 à jour | Diagrams-as-code auto-générés |
| Décisions | Non tracées | ADR pour les décisions majeures | ADR avec revue périodique |
| Revues d'architecture | Aucune | Revue avant chaque feature majeure | ATAM / RFC structurés |
| Conformité aux standards | Non évaluée | Auto-évaluation annuelle | Audit externe |
| Gouvernance | Informelle | Rôle d'architecte défini | Architecture Board actif |

## Ce que l'agent IA peut vérifier

### Analyse de la documentation

```bash
# Existence de fichiers d'architecture
find . -name "ARCHITECTURE.md" -o -name "architecture.md" -o -name "*.adr.md"
find . -path "*/docs/adr/*.md" -o -path "*/docs/architecture/*.md"

# Existence de diagrammes
find . -name "*.puml" -o -name "*.drawio" -o -name "*.dsl" -o -name "C4*.md"

# Date de la dernière modification des docs d'architecture
find . \( -name "ARCHITECTURE.md" -o -path "*/adr/*.md" \) -exec ls -la {} \;

# ADR dans un repo
ls docs/adr/ 2>/dev/null || echo "Pas de répertoire ADR"
```

### Questions d'audit (entretien)

1. Quel cadre d'architecture utilisez-vous ? Est-il adapté à la taille de l'organisation ?
2. Qui est responsable de l'architecture ? Y a-t-il un rôle d'architecte formellement défini ?
3. Comment les décisions architecturales majeures sont-elles prises et documentées ?
4. À quelle fréquence la documentation d'architecture est-elle mise à jour ?
5. Y a-t-il des revues d'architecture planifiées ? Avec quels participants ?
6. Comment les nouveaux développeurs apprennent-ils l'architecture du système ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Ce que l'IA peut faire avec l'output |
|---|---|---|
| Génération de diagrammes depuis le code | Structurizr, PlantUML, Mermaid | Relire et valider la cohérence des diagrammes générés |
| Analyse de conformité TOGAF | Planview, BiZZdesign | Lire le rapport d'écart produit |
| Modélisation ArchiMate | Archi (open source), Enterprise Architect | Analyser l'export XML ArchiMate |
| Revue formelle ATAM | Facilitation humaine requise | Analyser les scénarios de qualité et les risques identifiés |

## Références

- [C4 Model — Simon Brown](https://c4model.com/)
- [Architecture Decision Records (MADR)](https://adr.github.io/madr/)
- [TOGAF Standard — The Open Group](https://www.opengroup.org/togaf)
- [ISO/IEC 42010:2011](https://www.iso.org/standard/50508.html)
- [Thoughtworks Technology Radar — Architecture](https://www.thoughtworks.com/radar)
