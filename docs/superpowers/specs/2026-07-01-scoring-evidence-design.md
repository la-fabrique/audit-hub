# Spec — Renforcement du modèle de scoring et de la traçabilité d'audit-hub

**Date :** 2026-07-01
**Statut :** Approuvé pour planification

## Contexte et objectif

Audit-hub est une base de connaissance markdown pilotée par un LLM (pas un outil
logiciel exécutable) : `CLAUDE.md` définit le workflow, `profiles/schema.md`
définit les règles de sélection et de scoring, `domains/<domain>/checklists/*.md`
contiennent les items d'audit, `templates/reports/audit-report.md` structure la
sortie.

Une analyse critique du framework (comparaison à des référentiels d'audit
établis — ISO 19011, ISO 27001, ASVS, CMMI) a fait ressortir des faiblesses
structurelles récurrentes et indépendamment confirmées :

1. Le score global d'un domaine mélange une **auto-évaluation déclarative**
   (champ `maturity.*` du profil, rempli par le client, non vérifié) et un
   **constat observé** (checklist remplie par l'IA à partir du code/config)
   dans une seule moyenne — un biais méthodologique qui rend le score
   difficile à interpréter : un score de 3/5 peut signifier "le client dit
   que c'est correct" ou "l'IA a vérifié et c'est correct", sans distinction.
2. Tous les items non-blocker d'une checklist ont le même poids dans le calcul
   du pourcentage, qu'ils portent sur un défaut cosmétique ou un risque
   sérieux (mais non bloquant).
3. Le format des findings ne distingue pas ce qui a été **observé
   directement** (code, config, logs) de ce qui a été **déclaré** par le
   client sans vérification, ou **déduit** par l'IA à partir de signaux
   partiels — aucune traçabilité de la preuve.
4. Plusieurs règles du framework, bien que définies noir sur blanc, sont
   régulièrement mal interprétées par des lecteurs attentifs de bonne foi :
   la valeur du plafond en cas d'item blocker, la distinction entre le
   mapping de maturité et le scoring de checklist (deux mécanismes
   différents), le fait que les `ai-selection-criteria` d'une checklist ne
   peuvent qu'ajuster la priorité et jamais réintroduire une checklist déjà
   écartée par les règles générales, et les seuils de conversion
   pourcentage → score. Une règle correcte mais mal lue produit le même
   résultat qu'une règle absente : elle doit donc être rendue plus difficile
   à mal lire.
5. Le framework ne documente nulle part ses propres limites structurelles
   (absence de vérification dynamique/exécutable, absence de garantie de
   reproductibilité d'une exécution à l'autre, envoi des données client à un
   LLM tiers, absence de versionnage des checklists dans le temps). Ces
   limites sont réelles et ne peuvent pas être "corrigées" par une évolution
   du markdown — mais les taire nuit à la crédibilité du framework auprès
   d'un auditeur qui l'évaluerait avant adoption.

**Objectif de cette spec** : faire évoluer `profiles/schema.md`, `CLAUDE.md`,
le format des checklists et `templates/reports/audit-report.md` pour adresser
les points 1 à 4 par des changements concrets de schéma, et adresser le point
5 par une divulgation honnête des limites. Le tout sans introduire de code,
d'outillage ou de dépendance — audit-hub reste un ensemble de fichiers
markdown interprétés par un LLM.

## Non-objectifs

- Ne pas ajouter de vérification dynamique/exécutable (SAST/DAST réels) : hors
  scope, nécessiterait de l'outillage logiciel que le framework n'a pas
  vocation à embarquer.
- Ne pas construire de mécanisme de validation croisée multi-LLM ni de
  garantie de reproductibilité stricte : proposition jugée non actionnable
  sans analyse de coût/risque de boucle, laissée hors scope.
- Ne pas migrer le format des checklists vers un format structuré (YAML/JSON)
  validé par schéma/CI : changerait la nature du projet (markdown lisible par
  un humain et un LLM) sans bénéfice proportionné au coût.
- Ne pas traiter l'indépendance de l'auditeur (mandat, lettre de mission) ni
  la confidentialité de l'envoi de données à un LLM tiers autrement que par
  une mention explicite dans les limites (point 5) — ce sont des sujets de
  gouvernance d'usage, pas de schéma.

## Composants

### 1. Scoring v2 — séparation score déclaratif / score observé

`profiles/schema.md` § Scoring est restructuré :

- Le **score déclaratif** d'un domaine reste le mapping qualitatif existant
  (`profiles/schema.md` § Mapping maturité) appliqué aux champs `maturity.*`
  du profil pertinents pour le domaine. Il est toujours labellisé
  "déclaratif — non vérifié" dans le rapport.
- Le **score observé** d'un domaine est la moyenne pondérée (voir ci-dessous)
  des scores de checklist du domaine uniquement — il ne contient plus aucune
  composante déclarative.
- Le score déclaratif et le score observé sont **rapportés côte à côte**,
  jamais moyennés ensemble. `templates/reports/audit-report.md` § Évaluation
  par domaine affiche désormais deux valeurs (`Score déclaratif : X/5` et
  `Score observé : Y/5`) au lieu d'un score unique.
- Le plafonnement à 2 en cas d'item blocker en échec (`profiles/schema.md`
  § Items éliminatoires) s'applique uniquement au score observé — un item
  blocker est par définition un constat, jamais une déclaration.

### 2. Pondération des items de checklist

Chaque item de checklist peut porter un poids optionnel, en plus du marqueur
`[B]` existant :

```
- [ ] `[W2]` Les tokens de session ne sont jamais visibles dans les logs
- [ ] Les mots de passe communs sont rejetés
```

- Poids par défaut si non précisé : `1`.
- Poids déclarable : `2` (élevé) ou `3` (critique non-bloquant). Un item déjà
  marqué `[B]` n'a pas besoin de poids : son échec plafonne déjà le score.
- Le score observé d'une checklist devient : `(somme des poids des items
  cochés) / (somme des poids de tous les items applicables) × 100`, converti
  ensuite via la même table de seuils qu'aujourd'hui (`<30/30-49/50-69/70-89/
  ≥90`).
- Les checklists existantes restent valides sans modification (tous les items
  ont un poids implicite de 1, donc le calcul actuel est un cas particulier
  du nouveau).

### 3. Champ de preuve sur les findings

`CLAUDE.md` § Format des findings et `templates/reports/audit-report.md`
gagnent un champ obligatoire **Preuve**, avec trois valeurs possibles :

- `Observé` — constaté directement dans le code, la config, les logs ou un
  entretien avec démonstration (ex. capture d'écran, extrait de config).
- `Déclaré` — rapporté par le client sans vérification indépendante possible
  dans le cadre de l'audit (accès non disponible, ou point purement
  organisationnel).
- `Déduit` — inféré par l'IA à partir de signaux indirects ou partiels (ex.
  absence de fichier de config habituellement présent, structure de code
  suggérant une pratique sans la confirmer).

Ce champ est positionné juste après **Observation** dans chaque finding. Il
ne remplace aucun champ existant.

### 4. Clarification documentaire de `profiles/schema.md`

Sans changer les règles elles-mêmes (elles sont correctes), leur présentation
est renforcée aux quatre endroits où des lecteurs indépendants les ont
mal interprétées de façon répétée :

- **Plafond blocker** : la phrase "le score du domaine est plafonné à 2" est
  mise en évidence (bloc encadré) et un exemple chiffré est ajouté
  (checklist à 85% cochée mais un item `[B]` en échec → score observé = 2,
  pas 4).
- **Mapping maturité vs scoring checklist** : un paragraphe explicite ouvre
  la section pour nommer les deux mécanismes et dire clairement qu'ils ne se
  combinent jamais dans un même calcul (renforcé par le point 1 ci-dessus,
  qui rend cette règle vraie par construction plutôt que seulement énoncée).
- **Hiérarchie `ai-selection-criteria`** : la phrase "ne peuvent pas
  réintroduire une checklist écartée par les règles ci-dessous" est reformulée
  en règle numérotée séparée avec un exemple contre-intuitif (une checklist
  exclue par les règles générales reste exclue même si son
  `ai-selection-criteria` décrit un contexte très pertinent).
- **Table de seuils pourcentage → score** : réaffichée sous forme de règle
  résumée en une ligne juste avant la table ("moins de 30% = 1, chaque palier
  suivant +20 points sauf le dernier") pour réduire le risque de
  transcription erronée.

### 5. Section "Limites du framework"

Nouvelle section dans `CLAUDE.md`, après "Principes d'audit", intitulée
**Limites connues**. Elle énonce sans ambiguïté :

- Vérification **statique et déclarative uniquement** : l'IA lit du code, des
  configs et des réponses d'entretien, elle n'exécute rien (pas de SAST/DAST
  réel, pas de mesure de performance en conditions réelles).
- **Risque d'hallucination** : comme tout usage de LLM, les constats "Déduit"
  doivent être revus par un humain avant d'être communiqués comme définitifs.
- **Reproductibilité non garantie** : deux exécutions du même audit sur le
  même périmètre peuvent produire des findings ou une sélection de
  checklists légèrement différents.
- **Confidentialité** : les informations du profil et du code analysé
  transitent par un LLM tiers ; pour un contexte réglementé (HDS, PCI-DSS,
  secret professionnel), valider en amont les conditions d'usage du LLM
  utilisé.
- **Pas de garantie de comparabilité dans le temps** avant l'introduction du
  versionnage des checklists (point 6) : un score peut changer entre deux
  audits simplement parce que la checklist a été modifiée entre-temps.

Cette section est un pointeur, pas une réécriture — elle renvoie vers les
mécanismes concernés (`profiles/schema.md`, `domains/`) plutôt que de dupliquer
leur contenu.

### 6. Versionnage léger des checklists

Le frontmatter YAML des checklists (`domains/<domain>/checklists/*.md`) gagne
deux clés optionnelles :

```yaml
version: 1.0
last-updated: 2026-07-01
```

- `templates/reports/audit-report.md` § Évaluation par domaine liste, pour
  chaque checklist utilisée, sa version au moment de l'audit
  (`Checklists utilisées : authentication-checklist.md (v1.0)`).
- Aucune règle de compatibilité n'est imposée entre versions ; l'objectif est
  la traçabilité ("quelle version de la checklist a produit ce score"), pas
  un système de migration.
- Les checklists sans `version` déclarée sont traitées comme `version: 1.0`
  par défaut, pour rester rétro-compatible sans réécriture immédiate de tout
  le dossier `domains/`.

## Flux de données (impact sur le workflow existant)

Le workflow en 4 étapes de `CLAUDE.md` n'est pas restructuré, seules les
étapes 3 et 4 changent de contenu :

- **Étape 3 (Audit interactif)** : à chaque finding documenté, l'IA renseigne
  désormais le champ Preuve en plus des champs existants. Aucune étape
  supplémentaire n'est ajoutée au processus d'entretien/vérification.
- **Étape 4 (Génération du rapport)** : le template rempli produit deux
  scores par domaine (déclaratif + observé) au lieu d'un seul, et liste la
  version des checklists utilisées.

## Gestion des cas limites

- **Domaine sans checklist observée mais avec des axes de maturité
  déclarés** (ex. domaine entièrement en mode entretien, aucun accès) : le
  score observé est marqué `N/A — aucune checklist vérifiable`, seul le
  score déclaratif est affiché, avec la mention "non vérifié" en évidence.
- **Item de checklist non applicable** (ex. section JWT d'une checklist
  d'authentification pour une app qui n'utilise pas JWT) : il continue à être
  exclu du calcul du pourcentage comme aujourd'hui, poids compris — un item
  non applicable de poids 3 n'entre ni au numérateur ni au dénominateur.
- **Checklist modifiée après un audit déjà réalisé** : le rapport déjà généré
  référence la version au moment de l'audit ; il n'est pas rétroactivement
  invalidé, mais un nouvel audit utilisant la checklist mise à jour verra sa
  version différer dans son propre rapport.
- **Finding avec preuve mixte** (ex. code montre une pratique correcte mais
  le client déclare un contournement en production) : documenter deux
  findings distincts plutôt que de forcer une valeur unique de Preuve — un
  finding = une preuve.

## Validation

Le repo ne contient ni code ni CI : la validation se fait par relecture et
par un audit à blanc.

- **Relecture de cohérence** : vérifier que `profiles/schema.md`,
  `CLAUDE.md` et `templates/reports/audit-report.md` restent mutuellement
  cohérents après modification (mêmes noms de champs, mêmes valeurs
  possibles pour Preuve, même table de seuils référencée partout).
- **Audit à blanc** : dérouler le workflow des 4 étapes sur un profil
  d'exemple existant (`profiles/example-startup.md` ou équivalent) et
  vérifier que le rapport généré affiche bien deux scores par domaine, le
  champ Preuve sur chaque finding, et la version des checklists utilisées —
  sans erreur d'interprétation des nouvelles règles.
- **Non-régression** : vérifier qu'une checklist existante sans `weight` ni
  `version` déclarés produit exactement le même score qu'avant (poids 1
  implicite, version 1.0 implicite).
