---
domain: security
category: encryption-data-protection
applicable-to: [web, api, saas, mobile, infra]
tags: [encryption, tls, at-rest, in-transit, cryptography, data-protection, rgpd, hds, pci-dss]
related-knowledge: ../encryption-data-protection.md
ai-selection-criteria: |
  Inclure si `data_sensitivity` ∈ {confidential, restricted} ou si des données
  personnelles / de santé / de paiement sont traitées. Bonus de priorité +3 si
  contexte HDS (chiffrement obligatoire des DPS) ou PCI-DSS (PAN chiffré). +2 si
  contexte RGPD avec données sensibles (santé, biométrie, etc.).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Chiffrement et protection des données

## Chiffrement en transit (TLS)

- [ ] `[B]` TLS 1.2 minimum sur toutes les communications (TLS 1.3 recommandé)
- [ ] `[B]` TLS 1.0 et TLS 1.1 désactivés
- [ ] `[B]` La vérification des certificats n'est jamais désactivée (pas de `verify=False`)
- [ ] Seules les cipher suites AEAD sont autorisées (pas de RC4, 3DES, CBC avec hash faibles)
- [ ] HSTS activé sur tous les domaines applicatifs (`max-age ≥ 31536000`)
- [ ] Le renouvellement des certificats est automatisé (aucun renouvellement manuel en production)

## Algorithmes cryptographiques

- [ ] `[B]` MD5 et SHA-1 ne sont pas utilisés pour des fonctions de sécurité (signatures, intégrité)
- [ ] `[B]` DES, 3DES, RC4 ne sont pas utilisés pour le chiffrement des données
- [ ] Les mots de passe sont hashés avec bcrypt, Argon2id ou scrypt (pas SHA-256 seul)
- [ ] AES-256-GCM ou AES-128-GCM pour le chiffrement symétrique des données sensibles

## Chiffrement au repos

- [ ] Les données sensibles (PII, données de santé, données de paiement) sont chiffrées en base
- [ ] Les backups sont chiffrés
- [ ] Les clés de chiffrement sont stockées séparément des données chiffrées
- [ ] Une politique de rotation des clés est définie et appliquée

## Contexte HDS (si applicable)

- [ ] `[B]` Les données de santé (DPS) sont chiffrées au repos ET en transit
- [ ] Les clés sont gérées séparément des données (HSM recommandé)
- [ ] La gestion des clés est documentée (création, rotation, révocation, destruction)
- [ ] Les exports de données sont chiffrés

## Contexte PCI-DSS (si applicable)

- [ ] `[B]` Les PAN (numéros de carte) sont chiffrés au repos (AES-256) ou tokenisés
- [ ] Les PAN ne sont jamais loggés en clair
- [ ] Les clés cryptographiques sont gérées conformément à la Req. 3.5 PCI-DSS

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
