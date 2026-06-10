---
domain: security
category: regulatory-mapping
type: reference
tags: [iso27001, hds, rgpd, pci-dss, nis2, soc2, compliance]
ai-prompt-hints: |
  Lire ce fichier dès que le profil contient un champ `regulatory` non vide.
  Identifier quels topics deviennent obligatoires vs recommandés.
  Pour chaque norme applicable, élever la priorité des items marqués (M) à Critique.
  Signaler dans le rapport : "Audit de conformité [NORME] — items suivants obligatoires".
---

# Mapping réglementaire — Sécurité

## Comment utiliser ce fichier

Après lecture du profil (`profiles/`), identifier les normes dans le champ `regulatory`.
Pour chaque norme applicable :
1. Les topics marqués **(M)** deviennent des **exigences** (non-conformité = finding Critique)
2. Les topics marqués **(R)** sont des **recommandations renforcées** (non-conformité = finding Haute)
3. Ajouter une section "Conformité [NORME]" dans le rapport final

---

## ISO/IEC 27001:2022

**Contexte** : certification internationale de management de la sécurité de l'information (SMSI). Applicable sur demande, souvent requis pour les contrats grands comptes ou appels d'offres publics.

**Déclencheur dans le profil** : `regulatory` contient `ISO27001`

| Topic de sécurité | Contrôles ISO 27001 associés | Statut |
|---|---|---|
| `authentication.md` | A.8.2 (droits d'accès), A.8.5 (auth sécurisée) | **(M)** |
| `access-control.md` | A.8.2, A.8.3 (gestion des accès), A.5.15 | **(M)** |
| `secrets-management.md` | A.8.13 (sauvegarde info), A.8.24 (cryptographie) | **(M)** |
| `encryption-data-protection.md` | A.8.24 (utilisation de la cryptographie) | **(M)** |
| `logging-monitoring.md` | A.8.15 (journalisation), A.8.16 (surveillance) | **(M)** |
| `network-security.md` | A.8.20, A.8.21 (sécurité réseau) | **(M)** |
| `vulnerability-management.md` | A.8.8 (gestion des vulnérabilités techniques) | **(M)** |
| `dependency-security.md` | A.8.30 (externalisation), A.8.8 | **(R)** |

**Questions spécifiques ISO 27001** :
- Existe-t-il un SMSI formalisé (périmètre, politique, objectifs) ?
- Y a-t-il une analyse de risques documentée (selon ISO 27005) ?
- Les audits internes et revues de direction sont-ils réalisés ?
- Le registre des actifs informationnels est-il à jour ?

---

## HDS (Hébergement de Données de Santé)

**Contexte** : certification française obligatoire (ANS/ANSSI) pour tout hébergeur traitant des données de santé à caractère personnel (DMP, dossiers médicaux, résultats d'analyses…). Basée sur ISO 27001 + exigences additionnelles secteur santé. Certification requise pour exercer en France.

**Déclencheur dans le profil** : `regulatory` contient `HDS` ou `sector` = `healthtech`

| Topic de sécurité | Exigence HDS associée | Statut |
|---|---|---|
| `authentication.md` | Traçabilité des accès aux données de santé, MFA obligatoire sur accès DPS | **(M)** |
| `access-control.md` | Cloisonnement par patient/praticien, principe du moindre privilège strict | **(M)** |
| `encryption-data-protection.md` | Chiffrement des données de santé au repos ET en transit (TLS 1.2 min) | **(M)** |
| `logging-monitoring.md` | Journaux d'accès aux données de santé, conservation 5 ans minimum | **(M)** |
| `secrets-management.md` | Gestion des clés de chiffrement (HSM recommandé), rotation | **(M)** |
| `network-security.md` | Segmentation réseau entre zones hébergement santé et autres | **(M)** |
| `vulnerability-management.md` | Tests d'intrusion annuels obligatoires, gestion des CVE sous 72h pour critique | **(M)** |

**Questions spécifiques HDS** :
- L'hébergeur est-il certifié HDS (vérifier sur le site ANS) ? Quelle activité (1a, 1b, 2, 3, 4, 5, 6) ?
- Les données de santé sont-elles hébergées en France/UE ?
- Existe-t-il un contrat d'hébergement HDS avec les sous-traitants ?
- La politique de sauvegarde garantit-elle la RPO/RTO requis ?
- Y a-t-il un plan de continuité d'activité (PCA) testé ?

**Ressource** : [Référentiel HDS v1.1 — ANS](https://esante.gouv.fr/produits-services/hds)

---

## RGPD (Règlement Général sur la Protection des Données)

**Contexte** : règlement européen obligatoire pour tout traitement de données personnelles de résidents UE. Applicable à toute organisation, sans certification formelle mais avec obligation de conformité sous peine d'amendes (jusqu'à 4% du CA mondial).

**Déclencheur dans le profil** : `regulatory` contient `RGPD` ou `data_sensitivity` ≠ `public`

| Topic de sécurité | Article RGPD associé | Statut |
|---|---|---|
| `authentication.md` | Art. 32 (sécurité du traitement), Art. 25 (privacy by design) | **(M)** |
| `access-control.md` | Art. 32 (accès limité aux personnes autorisées) | **(M)** |
| `encryption-data-protection.md` | Art. 32 (pseudonymisation, chiffrement) | **(M)** |
| `logging-monitoring.md` | Art. 33 (notification violation — nécessite logs pour détecter) | **(M)** |
| `vulnerability-management.md` | Art. 32 (processus de test et évaluation réguliers) | **(R)** |
| `dependency-security.md` | Art. 28 (sous-traitants, DPA requis) | **(M)** |

**Questions spécifiques RGPD** :
- Existe-t-il un registre des traitements (Art. 30) ?
- Un DPO est-il désigné (obligatoire si traitement à grande échelle de données sensibles) ?
- Les violations de données peuvent-elles être détectées et notifiées dans les 72h (Art. 33) ?
- Les droits des personnes (accès, effacement, portabilité) sont-ils implémentés ?
- Les durées de conservation sont-elles définies et appliquées ?

---

## PCI-DSS v4.0

**Contexte** : norme de sécurité pour les organisations qui stockent, traitent ou transmettent des données de cartes de paiement. Certification annuelle par QSA (Qualified Security Assessor) ou auto-évaluation (SAQ) selon le niveau.

**Déclencheur dans le profil** : `regulatory` contient `PCI-DSS` ou `sector` = `fintech`/`ecommerce`

| Topic de sécurité | Exigence PCI-DSS associée | Statut |
|---|---|---|
| `authentication.md` | Req. 8 (MFA obligatoire CDE, mdp complexité) | **(M)** |
| `access-control.md` | Req. 7 (moindre privilège sur CDE) | **(M)** |
| `network-security.md` | Req. 1 (firewall CDE), Req. 4 (TLS) | **(M)** |
| `encryption-data-protection.md` | Req. 3 (PAN chiffré au repos), Req. 4 (transit) | **(M)** |
| `logging-monitoring.md` | Req. 10 (logs CDE, 12 mois rétention) | **(M)** |
| `vulnerability-management.md` | Req. 6 (patch <1 mois critique), Req. 11 (scan trimestriel, pentest annuel) | **(M)** |
| `secrets-management.md` | Req. 3.5 (gestion clés cryptographiques) | **(M)** |

---

## NIS2 (Network and Information Security Directive 2)

**Contexte** : directive européenne (transposée en droit français en 2024) imposant des exigences de cybersécurité aux entités essentielles et importantes. Périmètre élargi par rapport à NIS1 (santé, énergie, transport, numérique, eau, espace…).

**Déclencheur dans le profil** : `regulatory` contient `NIS2` ou `sector` critique (énergie, santé, transport, finance, numérique)

| Topic de sécurité | Mesures NIS2 associées | Statut |
|---|---|---|
| `vulnerability-management.md` | Art. 21 — gestion des risques, politiques de correctifs | **(M)** |
| `logging-monitoring.md` | Art. 21 — détection d'incidents, journalisation | **(M)** |
| `authentication.md` | Art. 21 — contrôle d'accès, authentification renforcée | **(M)** |
| `network-security.md` | Art. 21 — sécurité des réseaux, segmentation | **(M)** |
| `secrets-management.md` | Art. 21 — cryptographie et chiffrement | **(R)** |
| `dependency-security.md` | Art. 21 — sécurité de la chaîne d'approvisionnement | **(M)** |

**Questions spécifiques NIS2** :
- L'organisation est-elle dans le périmètre NIS2 (entité essentielle ou importante) ?
- Un plan de réponse aux incidents cyber est-il formalisé ?
- Les incidents significatifs peuvent-ils être notifiés à l'ANSSI dans les 24h (alerte précoce) puis 72h (notification) ?
- La gouvernance cyber est-elle portée au niveau de la direction (obligation NIS2) ?

---

## SOC 2 Type II

**Contexte** : framework américain (AICPA) pour les prestataires de services SaaS/cloud. Audit indépendant sur les Trust Service Criteria : Sécurité, Disponibilité, Intégrité de traitement, Confidentialité, Vie privée. Souvent demandé par les clients entreprises américains.

**Déclencheur dans le profil** : `regulatory` contient `SOC2` ou `users.type` = `B2B` avec clients US

| Topic de sécurité | Critère SOC2 associé | Statut |
|---|---|---|
| `authentication.md` | CC6.1 (accès logique), CC6.3 (révocation) | **(M)** |
| `access-control.md` | CC6.2, CC6.3 (gestion des accès) | **(M)** |
| `logging-monitoring.md` | CC7.2 (surveillance), A1.2 (disponibilité) | **(M)** |
| `vulnerability-management.md` | CC7.1 (détection des défauts), CC4.1 (COSO) | **(M)** |
| `encryption-data-protection.md` | C1.1 (confidentialité), CC6.7 (chiffrement) | **(M)** |
| `network-security.md` | CC6.6 (périmètre logique), CC6.7 (communications) | **(R)** |

---

## Tableau de synthèse — Couverture par norme

| Topic | ISO27001 | HDS | RGPD | PCI-DSS | NIS2 | SOC2 |
|---|---|---|---|---|---|---|
| `authentication` | M | M | M | M | M | M |
| `access-control` | M | M | M | M | — | M |
| `encryption-data-protection` | M | M | M | M | R | M |
| `logging-monitoring` | M | M | M | M | M | M |
| `secrets-management` | M | M | — | M | R | — |
| `network-security` | M | M | — | M | M | R |
| `vulnerability-management` | M | M | R | M | M | M |
| `dependency-security` | R | — | M | — | M | — |
