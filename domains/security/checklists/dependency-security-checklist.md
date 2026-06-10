---
domain: security
category: dependency-security
applicable-to: [web, api, mobile, saas, library]
tags: [dependencies, supply-chain, sbom, cve, sca, vulnerabilities, third-party, rgpd]
related-knowledge: ../dependency-security.md
ai-selection-criteria: |
  Inclure dans tout audit couvrant du code. Toujours inclure avec la règle
  « Tout projet avec du code » (schema.md). Bonus de priorité +3 si CVE critiques
  connues non patchées ou si l'organisation n'a pas de scan SCA automatisé. +2
  si contexte NIS2 (sécurité de la chaîne d'approvisionnement Art. 21) ou RGPD
  (sous-traitants Art. 28).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Sécurité des dépendances

## Inventaire et lockfiles

- [ ] `[B]` Un lockfile est commité pour chaque manifeste de dépendances (package-lock.json, poetry.lock, go.sum…)
- [ ] Les dépendances de production et de développement sont clairement séparées dans le manifeste
- [ ] Un SBOM (CycloneDX ou SPDX) est généré à chaque build et archivé avec l'artefact

## Détection des vulnérabilités

- [ ] `[B]` Un scan SCA bloquant est intégré dans la CI (Snyk, Dependabot, Trivy, Grype…) et bloque les PR en cas de CVE critique dans une dépendance de prod
- [ ] Le scan SCA tourne aussi en continu sur la branche principale (pas uniquement aux PR)
- [ ] Une politique de délai de patch est documentée : critique ≤ 7 jours, haute ≤ 30 jours

## Mises à jour

- [ ] Renovate ou Dependabot est configuré sur tous les manifestes (y compris Dockerfile, GitHub Actions, providers Terraform)
- [ ] Aucune dépendance de production n'est sans mise à jour depuis plus de 24 mois
- [ ] Les images Docker de base utilisent des digests (`sha256:…`) pour les images critiques

## Chaîne d'approvisionnement

- [ ] Les artefacts de build internes sont signés (cosign/sigstore) et la signature est vérifiée au déploiement
- [ ] Les packages récemment publiés (< 7 jours) ne sont pas utilisés comme dépendances de prod sans analyse

## Conformité RGPD (si applicable)

- [ ] `[B]` Tout sous-traitant traitant des données personnelles a un DPA signé (Art. 28 RGPD)
- [ ] La liste des sous-traitants est tenue à jour (incluant les sous-sous-traitants)
- [ ] Les transferts hors UE sont couverts par un mécanisme adéquat (CCT, pays adéquat…)

## Score d'évaluation

Compter le nombre de cases cochées. Les items `[B]` sont éliminatoires :
si un seul est en échec, le score est plafonné à 2.

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
