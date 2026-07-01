# Spec — Rééquilibrage de la couverture des domaines d'audit-hub

**Date :** 2026-07-01
**Statut :** Approuvé pour planification

## Contexte et objectif

Chaque domaine d'audit-hub (`domains/<domain>/`) est censé suivre la même
structure : des fiches de connaissance decrivant les bonnes pratiques, et des
checklists qui opérationnalisent ces fiches en items vérifiables, reliées par
le champ `related-knowledge` du frontmatter de chaque checklist (convention
1:1 : une checklist référence la fiche qu'elle opérationnalise).

Un état des lieux quantitatif des cinq domaines révèle un déséquilibre
structurel :

| Domaine | Fiches | Checklists | Items de checklist | Items `[B]` (blocker) |
|---|---|---|---|---|
| security | 9 | 8 | 197 | 32 |
| code-quality | 4 | 4 | 91 | 3 |
| infra-devops | 4 | 3 | 86 | 5 |
| organizational | 3 | 3 | 74 | 3 |
| architecture | 4 | 2 | 55 | **0** |

Ce déséquilibre n'est pas seulement quantitatif, il correspond à trois défauts
concrets et vérifiables dans le repo :

1. **`domains/architecture/frameworks-and-standards.md`** (fiche sur les
   cadres d'architecture — C4, TOGAF, ArchiMate, ADR) référence bien
   `architecture-review-checklist.md` dans son propre champ `related:`, mais
   **aucune checklist ne référence cette fiche en retour** via
   `related-knowledge`. Le lien n'existe que dans un sens : un audit qui suit
   le workflow standard (checklist → fiche) ne découvre jamais cette fiche,
   alors qu'un audit qui partirait de la fiche la relierait à la checklist.
2. **`domains/infra-devops/backup-recovery.md`** (fiche `severity: high`,
   `regulatory-relevance: [HDS, ISO27001, PCI-DSS]`, sur les politiques de
   sauvegarde, RPO/RTO, tests de restauration) **n'est référencée par aucune
   checklist et ne référence elle-même aucune checklist**. C'est un point
   d'audit à fort enjeu réglementaire qui n'apparaît dans aucun item
   vérifiable du workflow standard.
3. **`architecture-review-checklist.md`** déclare
   `related-knowledge: ../` (le dossier entier) au lieu d'une fiche
   précise — seule checklist du repo à ne pas suivre la convention 1:1
   utilisée par les 19 autres checklists.
4. **Le domaine architecture ne contient aucun item `[B]`** alors que les
   quatre autres domaines en ont. Un item `[B]` en échec plafonne le score
   observé du domaine à 2 (`profiles/schema.md` § Items éliminatoires) : dans
   les autres domaines, un défaut catastrophique (mot de passe en clair,
   absence de backup, secret en dur) force ce plafond. En architecture,
   aucun défaut — même une absence totale de redondance sur un système
   critique, ou zéro SPOF documenté — ne peut produire cet effet, alors que
   `architecture-review-checklist.md` contient déjà des items décrivant des
   risques de sévérité comparable (ex. "Les SPOF sont identifiés et
   documentés", "Un mécanisme de circuit breaker existe pour les appels
   inter-services").

**Objectif de cette spec** : corriger ces quatre défauts précis, sans
réduire la richesse du domaine security (qui est justifiée par l'ampleur du
domaine sécurité en pratique d'audit) et sans inventer de nouveau contenu
non fondé sur ce qui existe déjà dans les fiches.

## Non-objectifs

- Ne pas réduire le nombre d'items ou de checklists du domaine security pour
  "équilibrer" artificiellement les chiffres — le déséquilibre volumétrique
  entre security et les autres domaines n'est pas en soi un défaut tant que
  chaque domaine a une couverture plancher cohérente.
- Ne pas créer de nouveaux domaines ni de nouvelles fiches de connaissance à
  partir de zéro — cette spec ne fait que relier et compléter l'existant
  (fiches déjà écrites, checklists déjà écrites).
- Ne pas revoir l'intégralité du contenu de chaque checklist item par item —
  seuls les items `[B]` manquants en architecture, justifiés par du contenu
  déjà présent dans la checklist, sont ajoutés.

## Composants

### 1. Rattacher `frameworks-and-standards.md` à une checklist

Ajouter une section "Cadre et gouvernance d'architecture" dans
`architecture-review-checklist.md` avec des items dérivés directement des
red flags déjà documentés dans la fiche (absence de documentation
d'architecture formelle, absence d'ADR, diagrammes non maintenus depuis plus
de 6 mois) :

```
## Cadre et gouvernance d'architecture

- [ ] Un cadre ou une convention de documentation architecturale est en
      place (C4, ArchiMate, TOGAF, ou convention interne équivalente)
- [ ] Les décisions architecturales significatives sont tracées (ADR ou
      équivalent)
- [ ] Les diagrammes d'architecture existants ont été mis à jour dans les
      6 derniers mois
```

### 2. Rattacher `backup-recovery.md` à une checklist

Créer `domains/infra-devops/checklists/backup-recovery-checklist.md`, sur le
modèle des checklists existantes du domaine, avec `related-knowledge:
../backup-recovery.md`. Les items sont dérivés des `ai-prompt-hints` déjà
présents dans la fiche (politique de sauvegarde, RPO/RTO définis et mesurés,
test de restauration récent, chiffrement des backups, séparation du support
de backup et des données sources) :

```yaml
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
```

Ajouter cette checklist à `profiles/schema.md` § Règles générales de
sélection : ligne `Tout projet avec des données persistantes en production →
domains/infra-devops/checklists/backup-recovery-checklist.md, priorité Haute`.

### 3. Corriger `related-knowledge` d'`architecture-review-checklist.md`

Remplacer `related-knowledge: ../` par la liste explicite des fiches
couvertes par les sections existantes de cette checklist :

```yaml
related-knowledge:
  - ../coupling-and-dependencies.md
  - ../scalability-patterns.md
  - ../frameworks-and-standards.md
```

C'est la seule checklist du repo à référencer plusieurs fiches ; les 19
autres gardent le format à fiche unique déjà en place — cette spec introduit
la forme liste uniquement là où elle reflète la réalité (une checklist qui
couvre plusieurs fiches), sans changer la convention pour les checklists à
fiche unique.

### 4. Ajouter des items `[B]` en architecture

Marquer `[B]` deux items déjà existants dans
`architecture-review-checklist.md`, sélectionnés parce qu'ils décrivent des
risques de gravité comparable aux blockers des autres domaines (panne totale
non maîtrisée, absence de tout garde-fou) :

- `Les SPOF (Single Points of Failure) sont identifiés et documentés` → `[B]`
- Le nouvel item de la section 1 ci-dessus,
  `Un cadre ou une convention de documentation architecturale est en place`,
  reste sans `[B]` (défaut de qualité, pas de risque catastrophique).

Pas d'ajout d'item entièrement nouveau marqué `[B]` sans base dans le
contenu déjà présent — seule la requalification de l'existant est faite ici.

## Flux de données (impact sur le workflow existant)

Aucun changement du workflow en 4 étapes de `CLAUDE.md`. Impact uniquement
sur l'étape 2 (sélection des checklists, qui inclut désormais
`backup-recovery-checklist.md` selon la nouvelle règle) et sur les scores
observés du domaine architecture (désormais soumis au même mécanisme de
plafonnement blocker que les autres domaines).

## Gestion des cas limites

- **Projet sans infrastructure persistante** (ex. librairie sans backend) :
  `backup-recovery-checklist.md` n'est pas sélectionnée, la règle
  `applicable-to` s'applique normalement — aucun changement de comportement
  pour ce cas.
- **Audit déjà réalisé avant cette évolution** : un rapport existant qui ne
  mentionne pas de score architecture plafonné reste valide pour la version
  de checklist utilisée à l'époque (voir versionnage des checklists,
  spec scoring/preuve) — cette spec ne rétro-affecte pas les audits passés.

## Validation

- **Relecture de cohérence** : vérifier que chaque fiche de `domains/*/` a
  bien au moins une checklist qui la référence via `related-knowledge`
  après application de cette spec (les deux gaps identifiés — 
  `frameworks-and-standards.md`, `backup-recovery.md` — sont fermés).
- **Audit à blanc** : dérouler l'étape 2 (sélection des checklists) sur un
  profil avec `stack.infra` non vide et vérifier que
  `backup-recovery-checklist.md` est bien proposée.
- **Non-régression** : vérifier que les checklists non modifiées
  (code-quality, organizational, security, `documentation-checklist.md`)
  gardent un `related-knowledge` à fiche unique inchangé.
