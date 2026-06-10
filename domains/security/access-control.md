---
domain: security
category: access-control
severity: critical
applicable-to: [web, api, saas, infra, mobile]
tags: [rbac, abac, iam, least-privilege, authorization, zero-trust]
related:
  - ./authentication.md
  - ./logging-monitoring.md
regulatory-relevance: [ISO27001, HDS, PCI-DSS, NIS2, SOC2, RGPD]
ai-prompt-hints: |
  Demander : comment les permissions sont-elles définies ? RBAC ? ABAC ? Policies IAM ?
  Chercher dans le code : vérifications d'autorisation (ou leur absence), logique de contrôle d'accès,
  endpoints sans middleware d'auth.
  Red flags : autorisation côté client uniquement, IDOR (accès aux ressources d'autres users par ID),
  comptes admin partagés, accès prod pour tous les devs.
  Vérifier : matrice des rôles, revue des accès IAM cloud, accès aux données de santé (HDS : par patient/praticien).
---

# Contrôle d'accès

## Risques principaux

| Risque | Impact | Fréquence |
|---|---|---|
| IDOR (Insecure Direct Object Reference) | Critique | Très élevée |
| Absence de vérification d'autorisation côté serveur | Critique | Élevée |
| Principe du moindre privilège non appliqué | Haute | Très élevée |
| Comptes admin/service partagés | Critique | Élevée |
| Accès à la production pour toute l'équipe | Haute | Élevée |
| Élévation de privilèges non contrôlée | Critique | Moyenne |
| Pas de revue régulière des accès (access review) | Haute | Élevée |
| Accès aux données de santé non cloisonné par patient | Critique (HDS) | Selon architecture |

## Modèles de contrôle d'accès

| Modèle | Description | Usage typique |
|---|---|---|
| **RBAC** (Role-Based) | Permissions basées sur des rôles | La majorité des SaaS |
| **ABAC** (Attribute-Based) | Permissions basées sur attributs contextuels | Apps complexes, HDS |
| **PBAC** (Policy-Based) | Policies déclaratives (ex. OPA, Cedar) | Microservices, cloud-native |
| **Zero Trust** | Vérification systématique, pas de confiance implicite | Infra moderne, cloud |

## Meilleures pratiques

### Principe du moindre privilège

- Chaque utilisateur/service a uniquement les permissions nécessaires à sa fonction
- Accès temporaires et révocables pour les besoins ponctuels (JIT access)
- Comptes de service différents par application et environnement
- Revue des accès trimestrielle (ou à chaque changement de rôle)

### Séparation des duties (SoD)

- Pas de compte unique avec tous les privilèges (y compris pour les admins)
- Validation à deux personnes pour les actions critiques (déploiements prod, exports massifs)
- Séparation des environnements dev/staging/prod avec accès distincts

### Contrôle d'accès en HDS

- Accès aux données de santé cloisonné par contexte de soin (patient/praticien)
- Traçabilité obligatoire de chaque accès (voir `logging-monitoring.md`)
- Accès aux données de santé révoqué immédiatement en fin de prise en charge

## Ce que l'agent IA peut vérifier

### Analyse statique du code

```bash
# Endpoints sans middleware d'authentification/autorisation (Express.js)
grep -n "app\.get\|app\.post\|router\." --include="*.js" --include="*.ts" -r routes/

# Vérification d'autorisation absente sur les routes (Python Flask)
grep -rn "@app.route" --include="*.py" | grep -v "login_required\|requires_auth"

# IDOR potentiel : accès par ID sans vérification ownership
grep -rn "findById\|getById\|\.find.*id\s*=" --include="*.js" --include="*.ts" --include="*.py" \
  | grep -v "userId\|currentUser\|req\.user"

# Vérification du rôle côté serveur
grep -rn "role\|permission\|hasPermission\|can(" --include="*.js" --include="*.ts" -r src/ | head -30

# Admin routes protégées ?
grep -rn "admin\|/admin" --include="*.js" --include="*.ts" -r routes/ | head -20
```

### Questions d'audit (entretien)

1. Quels rôles existent dans le système ? Qui peut créer/modifier des rôles ?
2. Comment un développeur obtient-il un accès à la production ?
3. Y a-t-il des comptes de service partagés ? Pour quels usages ?
4. Les accès sont-ils revus régulièrement ? (access review, onboarding/offboarding)
5. Que se passe-t-il lors du départ d'un employé ? (révocation des accès)
6. Pour HDS : comment est géré le cloisonnement des accès par patient/praticien ?
7. Qui a accès aux secrets de production (base de données, API keys) ?

### Revue des accès cloud (IAM)

- Exporter et analyser les policies IAM AWS/GCP/Azure
- Identifier les rôles avec permissions trop larges (`*:*`, `AdministratorAccess`)
- Vérifier les comptes de service avec des clés permanentes vs OIDC/IRSA
- Repérer les utilisateurs IAM encore actifs d'anciens employés

## Ce qui nécessite un outil tiers

| Vérification | Outil recommandé | Type | Ce que l'IA peut faire avec l'output |
|---|---|---|---|
| Analyse IAM cloud complète | **Cloudsplaining** (AWS), **Cartography**, **IAM Access Analyzer** | Cloud IAM audit | Analyser le rapport HTML/JSON, identifier les permissions excessives |
| Test IDOR automatisé | **Burp Suite** (IDOR extensions), **OWASP ZAP** | DAST | Analyser les résultats de scan |
| Revue des policies OPA/Cedar | Revue manuelle + **OPA Conftest** | Policy audit | L'IA peut lire et analyser directement les fichiers `.rego` |
| Audit des droits Active Directory / LDAP | **BloodHound**, **PingCastle** | AD audit | Analyser les graphes de permissions exportés |

## Références

- [OWASP Top 10 — A01:2021 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [NIST SP 800-162 — Guide to Attribute Based Access Control (ABAC)](https://csrc.nist.gov/publications/detail/sp/800-162/final)
