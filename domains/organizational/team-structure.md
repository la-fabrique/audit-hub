---
domain: organizational
category: structure-équipe
severity: medium
applicable-to: [startup, scaleup, enterprise, team]
tags: [organisation, équipe, topologie, raci, bus-factor, silos, collaboration, team-topologies]
related:
  - ./tech-maturity.md
  - ./incident-management.md
  - ./checklists/governance-checklist.md
ai-prompt-hints: |
  Demander : organigramme technique, comment les équipes sont organisées (feature
  teams, plateformes, composants), qui est responsable de quoi en production.
  Observer : silos entre dev et ops, dépendances bloquantes entre équipes, bus
  factor sur les systèmes critiques.
  Red flags : une seule personne peut déployer en production, les développeurs
  n'ont pas accès aux logs de prod, une équipe ops est le goulot d'étranglement
  de toutes les mises en production.
---

# Structure des équipes

## Modèles d'organisation

### Team Topologies — 4 types fondamentaux

| Type | Rôle | Interaction principale | Risque si mal configuré |
|------|------|----------------------|------------------------|
| **Stream-aligned** | Livrer de la valeur sur un flux produit | Reçoit le support de la plateforme | Trop de dépendances bloquantes vers d'autres équipes |
| **Platform** | Fournir des capacités en self-service (CI/CD, observabilité, DB) | X-as-a-Service vers les stream-aligned | Devient un goulot d'étranglement si trop contrôlante |
| **Enabling** | Aider les équipes à acquérir de nouvelles compétences (temporaire) | Facilitating vers les stream-aligned | Crée de la dépendance si elle reste trop longtemps |
| **Complicated subsystem** | Gérer une composante technique spécialisée (ML, paiement, crypto) | Collaboration ou X-as-a-Service | Sur-spécialisation, isolement |

### Modes d'interaction

- **Collaboration** : deux équipes travaillent ensemble — coût élevé, à utiliser pour découvrir des interfaces, pas en continu
- **X-as-a-Service** : une équipe consomme un service d'une autre — scalable, faible friction
- **Facilitating** : une équipe aide une autre à monter en compétences — temporaire

## Bus factor

Le bus factor d'un système est le nombre minimum de personnes dont le départ simultané mettrait le système en danger.

```bash
# Bus factor par fichier (nb d'auteurs distincts)
git log --format="%aN" -- src/critical-module/ | sort -u | wc -l

# Fichiers touchés par une seule personne
git log --format="%aN %H" --name-only -- "src/" | \
  awk '/^[a-zA-Z]/{author=$0} /\./{print author, $0}' | \
  sort -k2 | uniq -f1 -u | head -20
```

Bus factor = 1 sur un système critique est un risque opérationnel et un risque RH.

**Remèdes** : pair programming, rotation des on-calls, documentation, mob reviews.

## RACI — matrice de responsabilité

La matrice RACI clarifie qui est **R**esponsable, **A**ccountable, **C**onsulté, **I**nformé pour les décisions clés :

| Décision | Dev Lead | Ops/SRE | RSSI | CTO |
|----------|----------|---------|------|-----|
| Déploiement en production | R | R/A | C | I |
| Choix technologique majeur | R | C | C | A |
| Réponse à un incident critique | R | A | I | I |
| Validation d'architecture | R | C | C | A |
| Accès à la production | R | A | C | I |

Signaux d'une RACI défaillante :
- Plusieurs « A » (accountable) sur une même décision → personne n'est vraiment responsable
- Personne « A » non disponible lors des incidents → délais de décision
- Décisions prises sans consultation des « C » → surprises et rollbacks

## Collaboration Dev/Sec/Ops

### Anti-patterns à détecter

| Anti-pattern | Symptôme | Impact |
|---|---|---|
| Silo Dev/Ops | Dev livre, Ops déploie — pas de communication | Incidents fréquents, MTTR élevé |
| Security as gatekeeper | La sécurité bloque les livraisons en fin de cycle | Contournements, dette sécurité |
| On-call = Ops only | Les développeurs ne font pas de garde | Pas de skin in the game, bugs non priorisés |
| Feature factory | Pas de temps alloué à la fiabilité et la dette | Dégradation progressive |

### Modèle cible : équipes autonomes

Une équipe stream-aligned idéale peut :
- Développer, tester, déployer et monitorer son service sans dépendre d'une autre équipe
- Répondre aux incidents sur son périmètre
- Prendre des décisions techniques dans son domaine

## Ce que l'agent IA peut vérifier

```bash
# Nombre d'auteurs distincts par module (bus factor proxy)
git log --format="%aN" -- src/ | sort | uniq -c | sort -rn | head -10

# Commits concentrés sur peu d'auteurs
git shortlog -sn --no-merges | head -10

# Dernière activité par auteur (identifier les départs récents)
git log --format="%aN %ad" --date=short | sort -k2 -r | awk '!seen[$1]++' | head -20
```

### Questions d'audit (entretien)

1. Qui peut déployer en production ? Combien de personnes ?
2. Si [X] quittait l'équipe demain, quels systèmes seraient en danger ?
3. Comment une équipe peut-elle déployer sans dépendre d'une autre équipe ?
4. Y a-t-il des équipes qui attendent régulièrement une autre équipe pour avancer ?
5. Comment la sécurité est-elle intégrée dans le flux de développement ?
6. Qui est on-call ? Les développeurs participent-ils à la garde ?

## Références

- [Team Topologies — Matthew Skelton & Manuel Pais](https://teamtopologies.com/)
- [Accelerate — Forsgren, Humble, Kim](https://itrevolution.com/accelerate-book/)
- [Google SRE — Team Organization](https://sre.google/sre-book/)
- [DORA — Team Experimentation](https://dora.dev/devops-capabilities/cultural/team-experimentation/)
