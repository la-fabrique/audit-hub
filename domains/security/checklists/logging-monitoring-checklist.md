---
domain: security
category: logging-monitoring
applicable-to: [web, api, saas, infra]
tags: [logs, monitoring, siem, alerting, audit-trail]
related-knowledge: ../logging-monitoring.md
ai-selection-criteria: |
  Inclure pour tout projet avec données sensibles ou contexte réglementaire.
  Bonus de priorité +3 si HDS (traçabilité accès données de santé obligatoire).
  +2 si PCI-DSS (rétention 12 mois, logs CDE).
  +1 si audit.trigger = incident (les logs sont souvent le premier artefact d'investigation).
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Journalisation et monitoring de sécurité

## Événements loggés

- [ ] Connexions réussies et échouées (avec IP, user-agent, timestamp)
- [ ] Déconnexions et expirations de session
- [ ] Accès aux données sensibles (données personnelles, données de santé, données financières)
- [ ] Modifications de données sensibles (création, mise à jour, suppression)
- [ ] Changements de permissions et de rôles
- [ ] Créations/modifications/suppressions de comptes
- [ ] Accès aux fonctions d'administration
- [ ] Déploiements et changements de configuration
- [ ] Erreurs système et exceptions non gérées

## Format et qualité des logs

- [ ] Les logs contiennent : timestamp (UTC), user ID, IP source, action, ressource affectée, résultat
- [ ] Les logs NE contiennent PAS : mots de passe, tokens, données de cartes, données de santé en clair
- [ ] Les timestamps sont synchronisés (NTP) entre tous les composants
- [ ] Les logs sont structurés (JSON) pour faciliter l'analyse automatique

## Stockage et rétention

- [ ] Les logs sont centralisés (pas uniquement sur le serveur applicatif)
- [ ] Rétention conforme aux exigences réglementaires (voir tableau ci-dessous)
- [ ] Les logs sont protégés contre la modification et la suppression non autorisées
- [ ] Les backups de logs sont chiffrés
- [ ] L'espace de stockage des logs est monitoré (pas de rotation silencieuse)

### Rétention requise par norme

| Norme | Rétention minimale |
|---|---|
| HDS | 5 ans (accès aux données de santé) |
| PCI-DSS | 12 mois (3 mois accessibles en ligne) |
| SOC 2 | 6 à 12 mois |
| ISO 27001 | Selon politique interne documentée |

## Alertes et détection

- [ ] Alertes sur les pics d'échecs d'authentification (brute force)
- [ ] Alertes sur les connexions depuis des IPs ou pays inhabituels
- [ ] Alertes sur les accès hors-heures (si applicable)
- [ ] Alertes sur les erreurs système critiques
- [ ] Alertes sur la détection de patterns d'attaque connus
- [ ] Les alertes ont des responsables et des procédures de réponse définies
- [ ] Le bruit d'alertes est maîtrisé (ratio signal/bruit acceptable)

## SIEM et corrélation (si applicable)

- [ ] Un SIEM ou équivalent centralise les événements de sécurité
- [ ] Des règles de corrélation identifient les chaînes d'attaque
- [ ] Les logs système, applicatifs et réseau sont corrélés

## Contexte réglementaire

**HDS uniquement** :
- [ ] Les accès aux données de santé (DPS) sont loggés individuellement (qui a accédé, quand, à quoi)
- [ ] Ces logs sont conservés 5 ans minimum
- [ ] Les logs d'accès aux données de santé sont séparés et protégés
- [ ] Un rapport d'accès par patient peut être généré (droit RGPD)

**PCI-DSS uniquement** :
- [ ] Tous les accès au CDE (Cardholder Data Environment) sont loggés
- [ ] Les logs sont revus quotidiennement pour le CDE
- [ ] Synchronisation NTP documentée et vérifiable

## Score d'évaluation

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
