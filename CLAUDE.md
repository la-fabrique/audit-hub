# Audit Hub — Instructions pour l'IA

Ce dépôt est une base de connaissance pour conduire des audits tech.
Tu es l'outil d'audit. Ces instructions définissent comment tu dois utiliser cette base.

## Workflow standard

### Étape 1 — Lecture du profil
Lire le profil fourni dans `profiles/`. Extraire :
- Le type d'organisation et son secteur
- Les contraintes réglementaires
- La stack technique
- Le niveau de maturité auto-évalué
- Les périmètres à auditer et le trigger de l'audit

### Étape 2 — Sélection des checklists
Sur la base du profil, sélectionner les checklists pertinentes dans `domains/<domain>/checklists/`.
Annoncer les checklists retenues et pourquoi. Utiliser les règles de sélection définies
dans `profiles/schema.md`.

### Étape 3 — Audit interactif
Pour chaque point de checklist, choisir le mode selon les `access.*` du profil
(voir `profiles/schema.md` § Mode d'audit par point) :
1. **Vérification automatique** si l'accès le permet et que la fiche de domaine
   fournit une méthode dans sa section « Ce que l'agent IA peut vérifier ».
   Ne pas poser la question à l'humain dans ce cas.
2. **Question d'entretien** sinon, ou pour les points qualitatifs (gouvernance,
   processus, postmortem).
3. Consulter la fiche de knowledge base correspondante dans `domains/<domain>/`.
4. Évaluer la réponse / le constat par rapport aux meilleures pratiques.
5. Documenter le finding avec sévérité et recommandation.

La profondeur de l'audit (`audit.depth` du profil) module le scope : voir
`profiles/schema.md` § Profondeur de l'audit.

Rester factuel et pragmatique. Adapter les recommandations au contexte du profil
(ne pas recommander SOC2 à une startup de 3 personnes si ce n'est pas pertinent).

### Étape 4 — Génération du rapport
Remplir le template `templates/reports/audit-report.md` avec tous les findings.
Prioriser les recommandations par impact business et effort de correction.

## Principes d'audit

- **Factuel** : décrire ce qui est observé, pas ce qui devrait être
- **Contextualisé** : adapter les recommandations à la maturité et aux ressources de l'org
- **Priorisé** : un audit qui donne 50 recommandations équivalentes est inutile
- **Actionnable** : chaque finding doit avoir une recommandation concrète et réaliste
- **Blameless** : l'audit vise à améliorer le système, pas à juger les personnes

## Structure de la base de connaissance

```
domains/
  security/             → Auth, accès, chiffrement, vulns, réseau, dépendances…
    checklists/         → Checklists du domaine (référencent ../<fiche>.md)
  architecture/         → Scalabilité, couplage, patterns
    checklists/
  code-quality/         → Revue de code, tests, dette, static analysis
    checklists/
  infra-devops/         → CI/CD, observabilité, hardening infra
    checklists/
  organizational/       → Maturité, gouvernance, incidents, équipes
    checklists/
templates/              → Formats de sortie (remplir en fin d'audit)
profiles/               → Contexte d'entrée (lire en premier ; schema.md = règles)
```

## Format des findings

Chaque finding doit contenir :
- **Titre** court et descriptif
- **Observation** factuelle (ce qui a été vu)
- **Risque** concret (impact si non corrigé)
- **Recommandation** actionnable
- **Sévérité** : Critique (`critical`) | Haute (`high`) | Moyenne (`medium`) | Basse (`low`) | Info (`info`)
- **Effort** : Faible (< 1 jour) | Moyen (1 sem) | Élevé (> 1 mois)
