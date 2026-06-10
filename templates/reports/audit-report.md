---
type: template
usage: |
  Template de rapport d'audit final. L'IA remplit ce template à l'issue
  de l'audit interactif. Prompt : "Génère le rapport final de l'audit
  en utilisant templates/reports/audit-report.md et les résultats
  de notre session."
---

# Rapport d'audit technique — [NOM ORGANISATION]

**Date :** [DATE]
**Auditeur :** [AUDITEUR]
**Version :** 1.0

---

## Résumé exécutif

> Synthèse en 3-5 phrases destinée à un lecteur non-technique (dirigeant, investisseur).
> Niveau de maturité global, principaux risques identifiés, priorités d'action.

**Niveau de maturité global : [1-5] / 5** — [Label : Réactif | Répétable | Défini | Géré | Optimisé]

### Synthèse des findings

| Sévérité (FR) | Sévérité (EN, frontmatter) | Nombre |
|---|---|---|
| Critique | `critical` | |
| Haute | `high` | |
| Moyenne | `medium` | |
| Basse | `low` | |
| Info | `info` | |

---

## Périmètre de l'audit

- **Organisation :** [NOM]
- **Date :** [DATE]
- **Durée :** [DURÉE]
- **Domaines audités :** [LISTE]
- **Accès disponibles :** [LISTE]
- **Interlocuteurs :** [LISTE]
- **Exclusions :** [LISTE]

---

## Findings

### Findings critiques — Action immédiate requise

#### [FINDING-001] [Titre court et descriptif]

**Domaine :** [security | architecture | code-quality | infra-devops | organizational]
**Sévérité :** Critique
**Effort de correction :** [Faible | Moyen | Élevé]

**Observation**
[Description factuelle du problème constaté — ce qui a été vu, pas ce qui devrait être.]

**Risque**
[Impact si non corrigé — conséquence concrète pour le business ou la sécurité.]

**Recommandation**
[Action spécifique, actionnable, avec si possible un exemple ou une référence.]

**Références**
- [Lien vers l'entrée de knowledge base domains/...]
- [Lien externe si pertinent]

---

#### [FINDING-002] ...

---

### Findings hauts

#### [FINDING-003] ...

---

### Findings moyens

#### [FINDING-004] ...

---

### Findings bas / améliorations recommandées

#### [FINDING-005] ...

---

## Conformité réglementaire

> Une sous-section par norme listée dans `profil.regulatory`. Reprendre les items
> marqués **(M)** dans `domains/security/regulatory-mapping.md` et statuer
> conforme / non conforme / partiel pour chacun. Supprimer ou marquer **N/A**
> si aucune contrainte réglementaire n'est applicable.

### [NORME — ex. RGPD] — Statut global : [Conforme | Partiel | Non conforme]

| Topic | Exigence | Statut | Finding lié |
|---|---|---|---|
| | | | |

---

## Évaluation par domaine

> Marquer **N/A — hors scope** pour les domaines listés dans `profil.audit.excluded`.
> Score 1–5 obtenu par conversion (voir `profiles/schema.md` § Mapping maturité).

### Sécurité — [Score] / 5

**Points forts :**
-

**Points faibles :**
-

**Checklists utilisées :** [LISTE]

---

### Architecture — [Score] / 5

**Points forts :**
-

**Points faibles :**
-

---

### Qualité de code — [Score] / 5

**Points forts :**
-

**Points faibles :**
-

---

### Infra / DevOps — [Score] / 5

**Points forts :**
-

**Points faibles :**
-

---

### Organisation — [Score] / 5

**Points forts :**
-

**Points faibles :**
-

---

## Plan d'action recommandé

### Court terme (0-30 jours) — Risques critiques

| # | Action | Responsable | Effort | Impact |
|---|--------|-------------|--------|--------|
| 1 | | | | |
| 2 | | | | |

### Moyen terme (1-3 mois) — Risques hauts

| # | Action | Responsable | Effort | Impact |
|---|--------|-------------|--------|--------|
| 1 | | | | |

### Long terme (3-12 mois) — Améliorations structurelles

| # | Action | Responsable | Effort | Impact |
|---|--------|-------------|--------|--------|
| 1 | | | | |

---

## Annexes

### A. Méthodologie

Cet audit a été conduit par analyse :
- [x] Du code source
- [x] Des configurations CI/CD
- [x] Des configurations d'infrastructure
- [x] Des entretiens avec l'équipe
- [ ] Des logs et métriques de production

### B. Références

- [Liste des entrées de la base de connaissance utilisées]
- [Standards et frameworks de référence]

### C. Historique des versions

| Version | Date | Auteur | Changements |
|---------|------|--------|-------------|
| 1.0 | [DATE] | [AUDITEUR] | Version initiale |
