---
domain: security
category: encryption-data-protection
severity: critical
applicable-to: [web, api, saas, mobile, infra]
tags: [encryption, tls, at-rest, in-transit, cryptography, data-protection, rgpd, hds]
related:
  - ./secrets-management.md
  - ./authentication.md
regulatory-relevance: [ISO27001, HDS, PCI-DSS, RGPD, SOC2]
ai-prompt-hints: |
  Demander : quelles données sensibles sont stockées ? Comment sont-elles chiffrées ?
  Chercher dans le code : algorithmes cryptographiques utilisés, configurations TLS, stockage de données sensibles.
  Red flags : MD5/SHA1 pour intégrité, DES/3DES, RSA < 2048 bits, TLS 1.0/1.1, certificats auto-signés en prod,
  données PII ou de santé non chiffrées au repos.
  Vérifier les configs : nginx/apache TLS config, cloud storage encryption settings, DB encryption at rest.
---

# Chiffrement et protection des données

## Risques principaux

| Risque | Impact | Fréquence |
|---|---|---|
| Données sensibles non chiffrées au repos | Critique | Élevée |
| TLS obsolète (1.0/1.1) ou mal configuré | Critique | Élevée |
| Algorithmes cryptographiques faibles (MD5, DES, RC4) | Critique | Moyenne |
| Certificats expirés ou auto-signés en production | Haute | Élevée |
| Clés de chiffrement stockées avec les données chiffrées | Critique | Faible |
| Pas de chiffrement des backups | Haute | Élevée |
| PII/données de santé dans des champs non chiffrés en DB | Critique | Moyenne |

## Standards cryptographiques actuels (2024)

### Chiffrement symétrique

| Usage | Recommandé | Acceptable | À bannir |
|---|---|---|---|
| Données au repos | **AES-256-GCM** | AES-128-GCM | DES, 3DES, RC4, Blowfish |
| Données en transit | TLS 1.3 (AES-GCM) | TLS 1.2 (AES-GCM) | TLS 1.0/1.1, SSL |

### Chiffrement asymétrique

| Usage | Recommandé | Acceptable | À bannir |
|---|---|---|---|
| Échanges de clés / signatures | **ECDSA P-256**, **Ed25519** | RSA-2048 | RSA < 2048, DSA |
| Certificats TLS | **ECDSA** | RSA-2048 | RSA-1024, MD5, SHA-1 |

### Hachage

| Usage | Recommandé | Acceptable | À bannir |
|---|---|---|---|
| Mots de passe | **Argon2id**, bcrypt (coût ≥12) | scrypt | MD5, SHA-1, SHA-256 seul |
| Intégrité de données | **SHA-256**, SHA-3 | — | MD5, SHA-1 |
| HMAC | **HMAC-SHA256** | HMAC-SHA512 | HMAC-MD5 |

## Ce que l'agent IA peut vérifier

### Analyse statique du code

```bash
# Algorithmes faibles
grep -rn "DES\|RC4\|MD5\|SHA1\b\|SHA-1" \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.java" --include="*.go"

# TLS mal configuré
grep -rn "TLSv1\b\|TLSv1_0\|PROTOCOL_TLSv1\b\|ssl_version.*TLSv1" \
  --include="*.py" --include="*.js" --include="*.conf" --include="*.yaml"

# Vérification des certificats désactivée
grep -rn "verify.*False\|verify_ssl.*false\|InsecureRequestWarning\|rejectUnauthorized.*false" \
  --include="*.py" --include="*.js" --include="*.ts"

# Clés hardcodées (voir aussi secrets-management.md)
grep -rn "private_key\s*=\s*[\"']\|BEGIN.*PRIVATE KEY" --include="*.py" --include="*.js"

# Données potentiellement sensibles non chiffrées en base (recherche de champs)
grep -rn "ssn\|social_security\|carte_vitale\|ipp\|date_naissance\|adresse\b" \
  --include="*.py" --include="*.js" --include="*.sql" -i
```

### Vérification de la configuration TLS

```bash
# Tester la configuration TLS d'un domaine (si accès réseau)
# À faire avec l'outil testssl.sh ou via ssllabs.com
# L'IA peut analyser la sortie fournie par l'auditeur

# Config nginx
grep -n "ssl_protocols\|ssl_ciphers\|ssl_session" /etc/nginx/nginx.conf 2>/dev/null

# Config Apache
grep -rn "SSLProtocol\|SSLCipherSuite" /etc/apache2/ /etc/httpd/ 2>/dev/null
```

### Questions d'audit (entretien)

1. Quelles données sont considérées sensibles dans votre système ?
2. Ces données sont-elles chiffrées au repos ? (base de données, fichiers, backups)
3. Quelle est la configuration TLS ? (version, ciphers)
4. Où sont stockées les clés de chiffrement ? (séparées des données ?)
5. Y a-t-il une politique de rotation des clés ?
6. Les backups sont-ils chiffrés ?
7. Pour HDS : les données de santé sont-elles chiffrées avec une clé gérée par le client (CMK) ?

### Contexte HDS spécifique

Pour les hébergeurs HDS, vérifier :
- Les données de santé (DPS) sont chiffrées au repos ET en transit
- Les clés sont gérées séparément des données (HSM recommandé)
- La gestion des clés est documentée (création, rotation, révocation, destruction)
- Les exports de données sont chiffrés

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Type | Ce que l'IA peut faire avec l'output |
|---|---|---|---|
| Test SSL/TLS complet | **testssl.sh**, **SSLyze**, **SSL Labs** | Network scan | Analyser le rapport fourni, identifier TLS < 1.2, ciphers faibles |
| Scan de vulnérabilités cryptographiques | **SonarQube** (règles crypto), **Semgrep** | SAST | Analyser les rapports JSON exportés |
| Audit des clés dans le cloud | **AWS IAM Access Analyzer**, **Cloud Custodian** | Cloud audit | Vérifier les policies de clés KMS |
| Test de chiffrement DB en transit | Capture réseau (Wireshark) | Network | L'IA peut analyser le rapport de capture |

## Références

- [ANSSI — Recommandations de sécurité relatives à TLS](https://www.ssi.gouv.fr/guide/recommandations-de-securite-relatives-a-tls/)
- [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [NIST SP 800-57 — Key Management](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final)
