# Audit Hub — Base de connaissance d'audit tech assisté par IA

Référentiel de meilleures pratiques pour conduire des audits tech avec l'IA comme outil central.

## Structure

```
audit-hub/
├── domains/                  # Connaissances par domaine (le "quoi" et le "pourquoi")
│   ├── security/
│   │   └── checklists/       # Checklists du domaine (sélection dynamique par l'IA)
│   ├── architecture/
│   │   └── checklists/
│   ├── code-quality/
│   │   └── checklists/
│   ├── infra-devops/
│   │   └── checklists/
│   └── organizational/
│       └── checklists/
├── templates/                # Templates de rapports et questionnaires
│   ├── reports/
│   └── questionnaires/
├── profiles/                 # Définitions de contexte org/projet (entrée du workflow)
```

Les checklists sont **imbriquées dans chaque domaine** (`domains/<domain>/checklists/`)
et non à la racine — elles vivent au plus près des fiches de connaissance qu'elles
référencent (frontmatter `related-knowledge: ../<fichier>.md`).

## Workflow d'audit

```
1. PROFIL        → Décrire l'organisation/projet (profiles/)
       ↓
2. CHECKLISTS    → L'IA sélectionne les checklists pertinentes (checklists/)
       ↓
3. AUDIT         → Audit interactif appuyé sur la base de connaissances (domains/)
       ↓
4. RAPPORT       → Génération du rapport final (templates/reports/)
```

## Démarrer un audit

Prompt de démarrage (à copier dans Claude Code) :

```
Lis le profil dans profiles/[mon-profil].md et sélectionne les checklists
pertinentes depuis domains/<domain>/checklists/, en appliquant les règles
de sélection de profiles/schema.md. Conduis l'audit de manière interactive,
en t'appuyant sur domains/ pour chaque point. Génère le rapport final
avec le template templates/reports/audit-report.md.
```

## Conventions des entrées

Trois sous-schémas de frontmatter YAML coexistent.

### Fiches de connaissance (`domains/<domain>/*.md`)

- `domain` : security | architecture | code-quality | infra-devops | organizational
- `category` : sous-catégorie thématique
- `severity` : critical | high | medium | low | info  *(mapping FR utilisé dans le rapport : Critique | Haute | Moyenne | Basse | Info)*
- `applicable-to` : contextes (web, api, mobile, cloud, startup, enterprise…)
- `tags` : mots-clés pour filtrage et sélection dynamique
- `related` : liens vers entrées et checklists associées
- `regulatory-relevance` *(optionnel)* : normes auxquelles ce topic se rattache
- `ai-prompt-hints` : guidance pour l'IA lors de l'analyse de ce point

### Checklists (`domains/<domain>/checklists/*.md`)

- `domain`, `category`, `applicable-to`, `tags` *(mêmes valeurs que ci-dessus)*
- `related-knowledge` : chemin **relatif** vers la fiche de connaissance, toujours `../<fichier>.md`
- `ai-selection-criteria` : critères en texte libre que l'IA utilise pour décider d'inclure (ou non) la checklist dans un audit donné, **en complément** de la table de règles centrale de `profiles/schema.md`

### Templates et fichiers de référence (`templates/`, `regulatory-mapping.md`)

- `type` : template | reference | schema
- `usage` *(templates)* : description de l'usage et prompt suggéré

## Domaines couverts

| Domaine | Focus |
|---|---|
| Security | OWASP, authentification, secrets, CVE, pentest |
| Architecture | Scalabilité, couplage, dette technique, cloud patterns |
| Code Quality | Revue de code, tests, conventions, maintenabilité |
| Infra / DevOps | CI/CD, IaC, observabilité, SLO, incidents |
| Organizational | Maturité tech, gouvernance, processus, culture |
