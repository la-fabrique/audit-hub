---
type: schema
version: 1.0
---

# Schéma de profil organisation/projet

Ce document définit la structure standard d'un profil d'audit.
Un profil est le point d'entrée du workflow : il permet à l'IA de sélectionner
les checklists pertinentes et d'adapter les questions d'audit au contexte.

## Structure d'un profil

```yaml
---
# ─── IDENTITÉ ─────────────────────────────────────────────────────────────────
name: ""                    # Nom de l'organisation ou du projet
type: ""                    # startup | scaleup | enterprise | agency | nonprofit
date: ""                    # Date de création du profil (YYYY-MM-DD)
auditor: ""                 # Nom de l'auditeur

# ─── CONTEXTE BUSINESS ────────────────────────────────────────────────────────
sector: ""                  # fintech | healthtech | ecommerce | saas | etc.
regulatory:                 # Contraintes réglementaires applicables
  - ""                      # RGPD | PCI-DSS | HIPAA | SOC2 | ISO27001 | HDS | etc.
data_sensitivity: ""        # public | internal | confidential | restricted
users:
  count: ""                 # Ordre de grandeur : <100 | 1k | 10k | 100k | 1M+
  type: ""                  # B2B | B2C | internal

# ─── ÉQUIPE TECHNIQUE ─────────────────────────────────────────────────────────
team:
  size: ""                  # 1-5 | 6-15 | 16-50 | 50+
  structure: ""             # fullstack | frontend+backend | platform+product | etc.
  seniority: ""             # junior | mixed | senior
  distributed: ""           # local | remote-france | remote-international

# ─── STACK TECHNIQUE ──────────────────────────────────────────────────────────
stack:
  frontend: []              # React, Vue, Angular, mobile, etc.
  backend: []               # Node, Python, Java, Go, Ruby, etc.
  database: []              # PostgreSQL, MySQL, MongoDB, Redis, etc.
  infra: []                 # AWS, GCP, Azure, OVH, on-premise, etc.
  deployment: []            # Docker, K8s, serverless, VM, PaaS, etc.
  languages: []             # TypeScript, Python, Java, Go, etc.

# ─── MATURITÉ ACTUELLE ────────────────────────────────────────────────────────
maturity:
  cicd: ""                  # none | manual | partial | automated | advanced
  testing: ""               # none | some | good | extensive
  monitoring: ""            # none | logs-only | metrics | full-observability
  security_practices: ""    # none | basic | standard | advanced
  documentation: ""         # none | partial | good | extensive

# ─── CONTEXTE D'AUDIT ─────────────────────────────────────────────────────────
audit:
  scope:                    # Domaines à auditer
    - ""                    # security | architecture | code-quality | infra-devops | organizational
  trigger: ""               # Pourquoi cet audit ? incident | levée-fonds | conformité | proactif | acquisition
  priority_concerns: []     # Préoccupations principales exprimées par le client
  excluded: []              # Périmètres explicitement hors-scope
  depth: ""                 # quick (2h) | standard (1 jour) | deep (2-5 jours)

# ─── ACCÈS DISPONIBLES ────────────────────────────────────────────────────────
access:
  code_repo: false          # Accès au code source
  ci_cd: false              # Accès aux pipelines
  infra_config: false       # Accès aux configs d'infra
  monitoring: false         # Accès aux dashboards/logs
  interviews: false         # Entretiens avec l'équipe possible
---
```

## Utilisation

### Créer un profil
```bash
cp profiles/example-startup.md profiles/mon-client.md
# Éditer les valeurs selon le contexte
```

### Démarrer l'audit avec un profil

```
Prompt : "Lis profiles/mon-client.md. Sur la base de ce profil,
liste les checklists que tu vas utiliser et pourquoi. Puis commence
l'audit de manière interactive."
```

## Règles de sélection des checklists par l'IA

L'IA doit appliquer cette logique lors de la lecture du profil, dans cet
ordre :

1. **Sélection in/out** : la table § Règles générales (et § Règles
   réglementaires) fait foi pour décider quelles checklists entrent dans le
   périmètre de l'audit. Une checklist non retenue par ces règles est
   écartée.
2. **Ajustement de priorité uniquement** : les champs `ai-selection-criteria`
   des checklists déjà retenues à l'étape 1 sont consultés pour ajuster leur
   score de priorité (bonus `+N`, voir § Échelle de priorité numérique).
   `ai-selection-criteria` ne peut **jamais réintroduire** une checklist déjà
   écartée à l'étape 1, même si son contenu décrit un contexte très
   pertinent pour le profil audité.

**Exemple contre-intuitif** : un profil sans aucune donnée persistante en
production (pas de règle § Règles générales qui matche) ne verra jamais
`backup-recovery-checklist.md` proposée, même si son `ai-selection-criteria`
mentionne un bonus de priorité HDS/ISO27001/PCI-DSS applicable au secteur du
profil — l'étape 2 ne s'exécute jamais pour une checklist qui n'a pas franchi
l'étape 1.

#### Échelle de priorité numérique

Les priorités qualitatives de la table ci-dessous correspondent à un score de
base. Les `ai-selection-criteria` de chaque checklist ajoutent un bonus
conditionnel. La priorité finale détermine l'ordre de traitement et le tri
pour `depth: quick`.

| Priorité qualitative | Score de base |
|---|---|
| Critique | 10 |
| Haute | 5 |
| Moyenne | 2 |

Un bonus `+N` dans `ai-selection-criteria` s'ajoute au score de base.
Exemple : une checklist à priorité Haute (5) avec un bonus contextuel de +3
obtient un score final de 8.

### Règles générales

| Condition | Fiche / checklist ajoutée | Priorité |
|-----------|--------------------------|----------|
| `regulatory` non vide | **Lire `domains/security/regulatory-mapping.md` en priorité** | Critique |
| `data_sensitivity` ∈ {confidential, restricted} | `domains/security/checklists/authentication-checklist.md` | Haute |
| `maturity.cicd` ∈ {none, manual} | `domains/infra-devops/checklists/cicd-checklist.md` | Haute |
| `audit.trigger` = incident | `domains/organizational/incident-management.md` | Haute |
| `audit.trigger` = levée-fonds | `domains/architecture/scalability-patterns.md` + `domains/organizational/checklists/maturity-checklist.md` | Haute |
| `team.size` = `1-5` | `domains/organizational/checklists/maturity-checklist.md` (adapté startup) | Moyenne |
| `deployment` contient `K8s` | `domains/security/network-security.md` + `domains/infra-devops/infrastructure-hardening.md` (sections K8s) | Haute |
| Tout projet avec du code | `domains/security/checklists/vulnerability-management-checklist.md` + `domains/security/dependency-security.md` | Haute |
| Tout projet avec des utilisateurs | `domains/security/checklists/authentication-checklist.md` + `domains/security/access-control.md` | Haute |
| Tout projet avec des données persistantes en production | `domains/infra-devops/checklists/backup-recovery-checklist.md` | Haute |

### Règles réglementaires (élèvent la priorité des items marqués **M** dans regulatory-mapping.md)

| Condition | Action | Impact |
|-----------|--------|--------|
| `regulatory` contient `HDS` ou `sector` = healthtech | Activer toutes les checklists sécu en mode **M** ; ajouter section "Conformité HDS" au rapport | Critique |
| `regulatory` contient `ISO27001` | Activer security/* + vérifier existence SMSI, registre actifs, analyse de risques | Critique |
| `regulatory` contient `RGPD` ou `data_sensitivity` ≠ public | security/authentication + security/access-control + security/logging-monitoring | Haute |
| `regulatory` contient `PCI-DSS` ou `sector` = fintech/ecommerce | Toutes checklists sécu en mode **M** ; scope CDE à identifier | Critique |
| `regulatory` contient `NIS2` ou sector critique | security/vulnerability-management + security/logging-monitoring | Critique |
| `regulatory` contient `SOC2` | security/authentication + security/access-control + security/logging-monitoring | Haute |

### Règles d'accès disponibles (détermine ce que l'IA peut faire seule)

| `access.*` disponible | Ce que l'IA peut faire | Ce qui nécessite des outils tiers |
|---|---|---|
| `code_repo: true` | SAST via grep, revue de code, analyse des dépendances | Outils SAST formels (SonarQube, Semgrep), DAST |
| `ci_cd: true` | Revue des pipelines, détection de secrets CI | Scans automatisés intégrés |
| `infra_config: true` | Revue IaC, configs K8s/Docker | Scanners infra (Trivy, kube-bench, ScoutSuite) |
| `monitoring: true` | Analyse des logs et métriques fournis | SIEM, corrélation automatisée |
| `interviews: true` | Questions d'audit approfondies | — |
| Aucun accès | Questions d'audit uniquement (mode questionnaire) | Tous les outils de scan |

### Mode d'audit par point (auto vs interview)

Pour chaque point d'une checklist, l'IA choisit le mode :

1. **Vérification automatique** si `access.*` correspondant est `true` et que la fiche
   `domains/` fournit une commande / méthode de vérification dans sa section
   « Ce que l'agent IA peut vérifier ». Pas de question à l'humain dans ce cas.
2. **Question d'entretien** si pas d'accès, ou si le point demande explicitement
   du contexte humain (processus, gouvernance, postmortem…).
3. **Croisé** quand utile : vérifier dans le code/IaC + valider le contexte avec
   l'équipe (typique pour les findings sensibles).

## Profondeur de l'audit (`audit.depth`)

Le champ `audit.depth` du profil pilote le scope retenu :

| `depth` | Durée cible | Comportement de l'IA |
|---|---|---|
| `quick` | 2 h | **Top 5 checklists** retenues par les règles ci-dessus, triées par priorité (Critique > Haute > Moyenne). Ignorer les findings de priorité Basse. Rapport synthétique, pas de section « Évaluation par domaine ». |
| `standard` | 1 jour | **Toutes les checklists retenues** par les règles. Mode questionnaire + vérifs automatiques si accès. Rapport complet. |
| `deep` | 2-5 jours | Standard + revue de code ciblée (zones critiques : auth, paiement, accès aux données), tests de bout-en-bout des contrôles documentés, entretiens étendus. |

## Scoring

Le framework distingue deux mécanismes qui **ne se combinent jamais dans un
même calcul** : le **mapping maturité** convertit une auto-évaluation
déclarative du profil (`maturity.*`, non vérifiée) en score, et le
**scoring de checklist** convertit un constat observé (items cochés par
l'IA à partir du code/config/entretien) en score. Un domaine produit donc
toujours deux scores séparés — voir § Score déclaratif et score observé
d'un domaine.

### Scoring des checklists

Toutes les checklists utilisent une **unique échelle à 5 paliers** pour
convertir le pourcentage (ou le pourcentage pondéré, voir § Pondération)
d'items cochés en un score 1–5. Règle résumée : **moins de 30 % = 1, chaque
palier suivant +20 points, sauf le dernier** (70–89 % au lieu de 70–90 %) :

| % items cochés | Score | Label |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |

#### Pondération des items de checklist

Un item de checklist peut porter un poids optionnel, en plus du marqueur
`[B]` :

```
- [ ] `[W2]` Les tokens de session ne sont jamais visibles dans les logs
- [ ] Les mots de passe communs sont rejetés
```

- Poids par défaut si non précisé : `1`.
- Poids déclarable : `[W2]` (élevé) ou `[W3]` (critique non-bloquant). Un
  item déjà marqué `[B]` n'a pas besoin de poids : son échec plafonne déjà
  le score (voir ci-dessous), un poids serait sans effet.
- Le score d'une checklist devient : `(somme des poids des items cochés) /
  (somme des poids de tous les items applicables) × 100`, converti ensuite
  via la table ci-dessus.
- Un item non applicable au contexte audité (ex. section JWT d'une checklist
  d'authentification pour une app qui n'utilise pas JWT) est exclu du
  calcul, poids compris — il n'entre ni au numérateur ni au dénominateur.
- Les checklists existantes restent valides sans modification : tous les
  items ont un poids implicite de `1`, donc le calcul ci-dessus est
  identique au calcul précédent pour une checklist sans poids déclaré.

#### Items éliminatoires (blocker)

> **Règle** : si un seul item marqué `[B]` (blocker) est en échec, le
> **score observé du domaine est plafonné à 2** (Critique), quel que soit
> le pourcentage global obtenu par ailleurs.

Ces items représentent des failles dont la gravité rend le score global non
représentatif (ex. mots de passe en clair, absence totale de backup,
secrets en dur dans le code).

**Exemple** : une checklist cochée à 85 % (score brut = 4, Acceptable) mais
contenant un item `[B]` en échec (ex. secrets en dur détectés dans le code)
→ le score observé du domaine est **2**, pas 4. Le plafond blocker
s'applique uniquement au **score observé** (voir ci-dessous) : un item
blocker est par définition un constat, jamais une déclaration, donc il n'a
aucun effet sur le score déclaratif.

### Mapping maturité (profil → score)

Le profil utilise une échelle qualitative par axe. Le rapport (et les
templates) utilise un score 1–5. Conversion :

| Valeur profil | Score (1-5) | Label CMM |
|---|---|---|
| `none` / `chaotique` | 1 | Réactif / Initial |
| `manual` / `some` / `basic` / `logs-only` | 2 | Répétable |
| `partial` / `good` / `metrics` / `standard` | 3 | Défini |
| `automated` / `extensive` / `full-observability` | 4 | Géré |
| `advanced` | 5 | Optimisé |

### Score déclaratif et score observé d'un domaine

Un domaine produit **deux scores rapportés côte à côte, jamais moyennés
ensemble** (`templates/reports/audit-report.md` § Évaluation par domaine
affiche `Score déclaratif : X/5` et `Score observé : Y/5`) :

- **Score déclaratif** : moyenne arrondie des scores par axe de maturité
  applicable au domaine (§ Mapping maturité), calculée uniquement à partir
  des champs `maturity.*` du profil. Toujours labellisé "déclaratif — non
  vérifié" dans le rapport.
- **Score observé** : moyenne arrondie des scores des checklists associées
  au domaine uniquement (§ Scoring des checklists) — aucune composante
  déclarative. Si un item blocker d'une checklist du domaine est en échec,
  le score observé est plafonné à 2 (§ Items éliminatoires).
- **Domaine sans checklist vérifiable** (aucun accès, mode entretien
  uniquement) : le score observé est marqué `N/A — aucune checklist
  vérifiable`, seul le score déclaratif est affiché, avec la mention "non
  vérifié" en évidence.

### Versionnage des checklists

Le frontmatter YAML d'une checklist peut déclarer `version: 1.0` et
`last-updated: YYYY-MM-DD`. Ces clés sont optionnelles : une checklist qui
ne les déclare pas est traitée comme `version: 1.0`. Le rapport final liste,
pour chaque checklist utilisée, sa version au moment de l'audit (ex.
`authentication-checklist.md (v1.0)`) — objectif de traçabilité ("quelle
version de la checklist a produit ce score"), sans système de migration ni
règle de compatibilité entre versions.
