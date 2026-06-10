---
domain: security
category: access-control
applicable-to: [web, api, saas, infra, mobile]
tags: [rbac, abac, iam, least-privilege, authorization, zero-trust, idor]
related-knowledge: ../access-control.md
ai-selection-criteria: |
  Inclure si l'application a des utilisateurs ou des rôles distincts. Toujours
  inclure si `data_sensitivity` ∈ {confidential, restricted} (règle obligatoire
  schema.md). Bonus de priorité +3 si contexte HDS (cloisonnement patient/praticien)
  ou PCI-DSS (moindre privilège sur le CDE). +2 si l'équipe a connu des incidents
  liés à des accès non autorisés.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Contrôle d'accès

## Modèle d'autorisation

- [ ] `[B]` Les vérifications d'autorisation sont faites côté serveur sur chaque requête
- [ ] `[B]` Aucune logique de contrôle d'accès n'est présente uniquement côté client
- [ ] Un modèle d'autorisation explicite est défini (RBAC, ABAC, policies)
- [ ] La matrice des rôles et permissions est documentée

## Principe du moindre privilège

- [ ] `[B]` Chaque utilisateur/service a uniquement les permissions nécessaires à sa fonction
- [ ] Les comptes de service sont distincts par application et environnement
- [ ] Aucun compte partagé entre plusieurs personnes ou services
- [ ] Les accès à la production nécessitent une justification explicite (JIT ou processus formel)
- [ ] Les accès IAM cloud ne contiennent pas de permissions `*:*` non justifiées

## Protection contre les IDOR

- [ ] `[B]` Les ressources d'un utilisateur ne sont pas accessibles depuis un autre compte en changeant l'ID dans l'URL
- [ ] Les identifiants exposés dans l'API sont non-séquentiels (UUID v4 ou équivalent)
- [ ] L'appartenance d'une ressource à l'utilisateur courant est vérifiée avant toute opération

## Séparation des accès

- [ ] Les environnements prod / staging / dev ont des accès distincts
- [ ] Les développeurs n'ont pas d'accès direct en écriture sur la production
- [ ] Les actions critiques (export massif, suppression) nécessitent une validation à deux personnes ou MFA re-auth
- [ ] Les accès temporaires sont révoqués à l'issue de leur usage

## Revue des accès

- [ ] Les accès sont revus au moins une fois par trimestre (access review)
- [ ] Les accès sont révoqués automatiquement lors d'un départ ou changement de rôle
- [ ] Un registre des accès privilégiés est maintenu et à jour

## Contexte HDS (si applicable)

- [ ] `[B]` Les accès aux données de santé sont cloisonnés par contexte de soin (patient/praticien)
- [ ] Les accès aux données de santé sont tracés individuellement (voir `logging-monitoring.md`)
- [ ] Les accès sont révoqués à la fin de la prise en charge

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
