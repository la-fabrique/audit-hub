---
domain: security
category: dependency-security
severity: critical
applicable-to: [web, api, mobile, saas, library]
tags: [dependencies, supply-chain, sbom, cve, sca, vulnerabilities, third-party]
related:
  - ./vulnerability-management.md
  - ./secrets-management.md
  - ../infra-devops/cicd-security.md
regulatory-relevance: [ISO27001, RGPD, NIS2]
ai-prompt-hints: |
  Demander à voir : manifestes de dépendances (package.json, requirements.txt,
  go.mod, pom.xml, Gemfile, Cargo.toml), lockfiles, configuration des scans
  de dépendances en CI, politique de mise à jour, sous-traitants/DPA (RGPD).
  Chercher dans le code : versions épinglées vs flottantes, dépendances non
  maintenues (last commit > 2 ans), CVE connues non patchées, packages
  typosquattés.
  Red flags immédiats : lockfile manquant, `npm install` sans `--package-lock`,
  CVE critiques non traitées depuis > 30 jours, dépendances inutilisées qui
  trainent dans le manifeste.
---

# Sécurité des dépendances et chaîne d'approvisionnement

## Risques principaux

| Risque | Impact | Fréquence |
|---|---|---|
| CVE critiques dans des dépendances de production non patchées | Critique | Très élevée |
| Dépendance compromise (typosquatting, takeover de package) | Critique | Moyenne |
| Pas de lockfile / versions non reproductibles | Haute | Élevée |
| Dépendances abandonnées (pas de mise à jour > 24 mois) | Haute | Très élevée |
| Builds non déterministes (récupération de packages au build) | Haute | Élevée |
| Pas de SBOM (Software Bill of Materials) | Moyenne | Très élevée |
| Sous-traitant traitant des données sans DPA (RGPD Art. 28) | Critique | Élevée |
| Images de base Docker non scannées / non mises à jour | Haute | Très élevée |

## Meilleures pratiques

### Inventaire et traçabilité

- **SBOM généré à chaque build** (CycloneDX ou SPDX) et archivé avec l'artefact.
- Lockfile obligatoire et **commité** : `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `poetry.lock`, `Pipfile.lock`, `go.sum`, `Cargo.lock`, `Gemfile.lock`.
- Pas de récupération de packages au moment du build prod — utiliser un registry interne (Artifactory, Nexus, GitHub Packages) qui mirrore et fige.

### Détection des vulnérabilités

- Scan SCA dans la CI à chaque PR : **bloquer** sur CVE critique connue dans une dépendance de prod.
- Scan continu de la branche principale (pas seulement aux PR) — de nouvelles CVE apparaissent sur du code stable.
- Distinction nette **deps prod / deps dev** : la CVE dans une lib de test mérite un ticket, pas un blocage.
- Politique documentée : critique = patch sous 7 jours, haute sous 30 jours, moyenne au prochain cycle.

### Mises à jour

- Renovate / Dependabot configurés sur tous les manifestes (y compris Dockerfile, GitHub Actions, Terraform providers).
- Grouper les patchs mineurs (less noise) mais traiter individuellement les majors et les CVE.
- Au moins une **passe mensuelle** sur les dépendances non bloquantes pour éviter l'accumulation.
- Tests automatisés suffisants pour absorber les bumps mineurs sans peur.

### Chaîne d'approvisionnement

- Vérifier la **provenance** des packages : provenance attestations (npm 9+, pip 25+, SLSA).
- Bloquer l'installation de packages publiés < 7 jours auparavant pour les deps prod (mitige le typosquatting / takeover récent).
- Signature des artefacts internes (cosign, sigstore) ; vérification en déploiement.
- Pour les images Docker : utiliser des **digests** (`sha256:...`) pour les images de base critiques, pas seulement des tags mutables.

### Conformité (RGPD / sous-traitants)

- Tout sous-traitant traitant des données personnelles a un **DPA** signé (Art. 28 RGPD).
- Liste des sous-traitants tenue à jour (incluant sous-sous-traitants si applicable).
- Évaluation du niveau de protection (UE / pays adéquat / clauses contractuelles types pour transferts hors UE).

## Ce que l'agent IA peut vérifier

### Analyse statique

```bash
# Lockfiles présents ?
find . -maxdepth 3 -name "package.json" -exec dirname {} \; | while read d; do
  ls "$d"/package-lock.json "$d"/yarn.lock "$d"/pnpm-lock.yaml 2>/dev/null || echo "MISSING lock: $d"
done

# Dépendances avec versions flottantes (^/~) en prod
grep -E '"[^"]+":\s*"[\^~]' package.json

# Images Docker sans digest pin
grep -rnE "^FROM\s+[^@]+$" --include="Dockerfile*"

# Dépendances abandonnées (Python)
pip list --outdated 2>/dev/null | head

# Audit npm/yarn rapide
npm audit --production --json 2>/dev/null | jq '.metadata.vulnerabilities'

# SBOM existant
find . -maxdepth 2 -iname "sbom*" -o -iname "*.cdx.json" -o -iname "*.spdx*"
```

### Vérification de configuration CI/CD

- Pipeline contient une étape SCA bloquante (Snyk, Dependabot, Trivy, OWASP Dependency-Check, Grype).
- Renovate/Dependabot config présente (`.github/dependabot.yml`, `renovate.json`).
- Politique de seuil documentée (par qui sont approuvées les PR de bump majeur ?).

### Questions d'audit (entretien)

1. Comment savez-vous quand une CVE critique impacte vos dépendances ?
2. Quel est le délai cible entre publication d'un patch critique et déploiement en prod ?
3. Le build de prod est-il reproductible à partir du code commité ?
4. Qui approuve les montées de versions majeures ? Y a-t-il une politique ?
5. Combien de dépendances de prod n'ont pas reçu de mise à jour depuis 12+ mois ?
6. Avez-vous un SBOM ? À quelle fréquence est-il généré ?
7. Liste des sous-traitants traitant des données personnelles : à jour ? DPA signés ?

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Type | Ce que l'IA peut faire avec l'output |
|---|---|---|---|
| Scan SCA complet | **Snyk**, **OWASP Dependency-Check**, **Trivy**, **Grype** | SCA | Lire le rapport JSON, prioriser par exploitabilité (EPSS) |
| Génération de SBOM | **Syft**, **CycloneDX CLI**, **cdxgen** | SBOM | Comparer deux SBOM (drift de dépendances entre releases) |
| Audit de la chaîne d'appro | **deps.dev**, **OSV-Scanner**, **Sigstore policy-controller** | Supply chain | Analyser le rapport de provenance |
| Scan d'images conteneur | **Trivy**, **Grype**, **Docker Scout** | Container scan | Différencier vulnérabilités OS vs application |
| Surveillance des publications suspectes | **Phylum**, **Socket.dev** | Supply chain monitoring | Analyser les alertes de comportement (postinstall scripts, etc.) |

## Références

- [OWASP — Software Component Verification Standard (SCVS)](https://owasp.org/www-project-software-component-verification-standard/)
- [NIST SP 800-218 — Secure Software Development Framework (SSDF)](https://csrc.nist.gov/publications/detail/sp/800-218/final)
- [SLSA — Supply-chain Levels for Software Artifacts](https://slsa.dev/)
- [CycloneDX — SBOM Specification](https://cyclonedx.org/)
- [CNIL — Modèle de DPA et guide sous-traitants](https://www.cnil.fr/fr/sous-traitant)
