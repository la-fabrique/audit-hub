---
domain: security
category: authentication
severity: critical
applicable-to: [web, api, mobile, saas]
tags: [owasp, authentication, jwt, sessions, mfa, passwords, oauth]
related:
  - ./secrets-management.md
  - ./checklists/authentication-checklist.md
ai-prompt-hints: |
  Demander à voir : mécanisme d'authentification actuel, gestion des sessions,
  politique de mots de passe, présence du MFA, stockage des tokens.
  Chercher dans le code : bibliothèques d'auth utilisées, configuration JWT,
  durée de vie des sessions, gestion des refresh tokens.
  Red flags immédiats : tokens sans expiration, mots de passe en clair,
  absence de rate limiting sur /login, JWT sans vérification de signature.
---

# Authentification

## Risques principaux

| Risque | Impact | Fréquence |
|--------|--------|-----------|
| Credentials faibles / pas de politique de mdp | Critique | Très élevée |
| Absence de MFA sur comptes privilégiés | Critique | Élevée |
| Tokens JWT mal configurés (no expiry, weak secret) | Critique | Élevée |
| Session fixation / session non invalidée à la déconnexion | Haut | Moyenne |
| Absence de rate limiting sur endpoints d'auth | Haut | Élevée |
| Stockage de mots de passe non hashés ou MD5/SHA1 | Critique | Faible (mais catastrophique) |

## Meilleures pratiques

### Mots de passe
- Hashing avec **bcrypt** (coût ≥ 12), **Argon2id** (préféré), ou **scrypt**
- Politique minimale : 12 caractères, pas de rotation forcée périodique (NIST SP 800-63B)
- Vérification contre les listes de mots de passe compromis (HaveIBeenPwned API)

### Sessions
- Identifiants de session cryptographiquement aléatoires (≥ 128 bits)
- Invalidation côté serveur à la déconnexion
- `HttpOnly`, `Secure`, `SameSite=Strict` sur les cookies de session
- Durée de vie adaptée au contexte (15-30 min pour apps sensibles)

### JWT
- Algorithme : **RS256** ou **ES256** (asymétrique), jamais `none`
- `exp` obligatoire — access tokens : 15-60 min max
- Refresh tokens rotatifs avec révocation possible
- Valider `iss`, `aud`, `exp`, `nbf` à chaque requête

### MFA
- Obligatoire pour : admins, accès aux données sensibles, actions irréversibles
- Recommandé : TOTP (Google Authenticator, Authy) ou passkeys (WebAuthn)
- Éviter : SMS (SIM swapping) sauf si aucune autre option

### OAuth / OIDC
- PKCE obligatoire pour les clients publics (mobile, SPA)
- `state` paramètre pour prévenir les attaques CSRF
- Validation de la redirect_uri (liste blanche stricte, pas de wildcards)

## Ce que l'agent IA peut vérifier

### Analyse statique du code

```bash
# Hashing de mots de passe insuffisant
grep -rn "md5\|sha1\|sha256" --include="*.py" --include="*.js" --include="*.ts"
# JWT mal configuré
grep -rn "algorithm.*none\|verify.*False\|verify_signature.*false"
# Secret hardcodé
grep -rn "SECRET_KEY\s*=\s*[\"'][^\"']\|jwt_secret\s*=\s*[\"']"
# Cookie non protégé
grep -rn "httponly.*false\|secure.*false\|samesite.*none" -i
# Durée de session absente ou excessive
grep -rn "expiresIn.*[0-9]\{4,\}\|maxAge.*[0-9]\{8,\}"
```

### Questions d'audit (entretien)

1. Comment les mots de passe sont-ils stockés ? (algorithme, coût)
2. Les sessions sont-elles invalidées côté serveur à la déconnexion ?
3. Quels comptes ont le MFA activé ? Est-il obligatoire pour les admins ?
4. Quelle est la durée de vie des access tokens / refresh tokens ?
5. Y a-t-il du rate limiting sur `/login`, `/register`, `/forgot-password` ?
6. Comment les échecs d'authentification sont-ils loggés et alertés ?

### Revue de configuration / documentation

- Lire la configuration du provider d'auth (Auth0, Keycloak, Cognito…)
- Vérifier les politiques de mot de passe définies dans l'admin
- Examiner les diagrammes de flux d'authentification

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Type | Ce que l'IA peut faire avec l'output |
|---|---|---|---|
| Test de force brute / enumération de comptes | Burp Suite, OWASP ZAP | DAST | Analyser le rapport d'intrusion fourni, identifier les endpoints non protégés |
| Analyse dynamique des tokens en transit | Burp Suite Proxy | DAST | Lire la capture HAR / rapport pour vérifier les headers et flags cookies |
| Scan de vulnérabilités d'authentification connues | Nuclei, Metasploit | Pentest | Contextualiser les CVE identifiées selon la stack |
| Test de session fixation / hijacking | OWASP ZAP active scan | DAST | Analyser le rapport ZAP exporté en JSON/HTML |
| Audit de configuration Keycloak / Auth0 | keycloak-review (skill intégré) | Config scan | Déjà intégré dans ce workflow |

## Références

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [NIST SP 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [OWASP Top 10 — A07:2021 Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)
