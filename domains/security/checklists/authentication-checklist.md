---
domain: security
category: authentication
applicable-to: [web, api, mobile, saas]
tags: [owasp, authentication, mfa, jwt, sessions]
related-knowledge: ../authentication.md
ai-selection-criteria: |
  Inclure cette checklist si : l'application a des comptes utilisateurs, une API
  avec authentification, ou des données sensibles. Bonus de priorité +2 si présence
  de données personnelles, +3 si contexte réglementaire (RGPD, HIPAA, PCI-DSS).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Authentification

## Stockage des mots de passe

- [ ] `[B]` Les mots de passe sont hashés avec bcrypt (coût ≥ 12), Argon2id, ou scrypt
- [ ] `[B]` MD5, SHA1, SHA256 seuls ne sont PAS utilisés pour les mots de passe
- [ ] `[B]` Les mots de passe ne sont jamais loggés ni stockés en clair (même temporairement)
- [ ] Vérification contre les listes de credentials compromis

## Politique de mots de passe

- [ ] Longueur minimale : 12 caractères
- [ ] Pas de règles de complexité arbitraires (NIST recommande contre la rotation forcée)
- [ ] Les mots de passe courants sont rejetés

## Sessions

- [ ] Identifiants de session cryptographiquement aléatoires (≥ 128 bits)
- [ ] Sessions invalidées côté serveur à la déconnexion
- [ ] `HttpOnly` sur les cookies de session
- [ ] `Secure` sur les cookies de session (HTTPS uniquement)
- [ ] `SameSite=Strict` ou `SameSite=Lax` sur les cookies de session
- [ ] Durée de session adaptée (≤ 30 min pour apps sensibles, re-auth pour actions critiques)
- [ ] Pas de session ID dans les URLs

## JWT (si applicable)

- [ ] Algorithme RS256 ou ES256 (jamais `none`, jamais HS256 avec secret faible)
- [ ] `exp` présent et court (≤ 60 min pour access tokens)
- [ ] `iss` et `aud` validés à chaque requête
- [ ] Refresh tokens rotatifs avec possibilité de révocation
- [ ] Blacklist des tokens révoqués (déconnexion, changement de mot de passe)

## MFA

- [ ] MFA disponible pour tous les utilisateurs
- [ ] MFA obligatoire pour les comptes admin/privilégiés
- [ ] MFA obligatoire pour les actions sensibles (changement email/mdp, export de données)
- [ ] Méthode recommandée : TOTP ou WebAuthn (pas SMS seul)
- [ ] Codes de secours disponibles et sécurisés

## Rate limiting

- [ ] Rate limiting sur `/login` (ex: 5 tentatives / 15 min par IP)
- [ ] Rate limiting sur `/forgot-password`
- [ ] Rate limiting sur `/register`
- [ ] Délai exponentiel ou lockout après échecs répétés
- [ ] Alertes sur les tentatives d'authentification anormales

## OAuth / OIDC (si applicable)

- [ ] PKCE implémenté pour les clients publics
- [ ] Paramètre `state` validé pour prévenir le CSRF
- [ ] `redirect_uri` validée par liste blanche stricte (pas de wildcards)
- [ ] Tokens d'accès des providers tiers jamais stockés en clair

## Logging et monitoring

- [ ] Échecs d'authentification loggés (sans le mot de passe)
- [ ] Connexions réussies loggées (IP, user-agent, timestamp)
- [ ] Alertes sur les pics d'échecs d'authentification
- [ ] Revue régulière des connexions inhabituelles

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
