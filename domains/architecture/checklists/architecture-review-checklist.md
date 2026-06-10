---
domain: architecture
category: review
applicable-to: [web, api, saas, cloud, microservices, monolith, enterprise]
tags: [architecture, revue, couplage, scalabilité, résilience, solid, modularité]
related-knowledge: ../
ai-selection-criteria: |
  Inclure cette checklist si : le périmètre d'audit couvre l'architecture technique.
  Bonus de priorité +2 si l'organisation a subi des incidents de production liés à
  l'architecture (panne en cascade, impossibilité de scaler). +3 si refonte ou
  migration planifiée dans les 6 mois.
  Voir profiles/schema.md § Échelle de priorité numérique.
---

# Checklist — Revue d'architecture

## Séparation des responsabilités

- [ ] Les couches Présentation, Application, Domaine et Infrastructure sont distinctes
- [ ] La logique métier ne réside pas dans les contrôleurs ou les requêtes SQL
- [ ] Les entités de domaine ne dépendent pas de frameworks externes (ORM, HTTP)
- [ ] Les interfaces/ports sont définis dans le domaine, les implémentations dans l'infrastructure

## Couplage et dépendances

- [ ] Aucun import circulaire détecté entre modules
- [ ] Les modules ont une responsabilité unique identifiable
- [ ] Les dépendances tierces critiques sont abstraites derrière une interface interne
- [ ] Un inventaire des dépendances directes existe et est versionné (lockfile)
- [ ] Les dépendances vulnérables sont identifiées et suivies (npm audit, pip-audit…)

## Scalabilité et résilience

- [ ] Les app servers sont stateless (pas de sessions en mémoire locale)
- [ ] Les SPOF (Single Points of Failure) sont identifiés et documentés
- [ ] Un mécanisme de circuit breaker existe pour les appels inter-services
- [ ] Les opérations non-temps-réel passent par une queue asynchrone
- [ ] La stratégie de cache est définie avec TTL et politique d'invalidation

## Sécurité de l'architecture

- [ ] Le principe du moindre privilège est appliqué entre services (pas de service avec accès DB root)
- [ ] Les flux de données sensibles sont chiffrés en transit (TLS) et au repos
- [ ] Les secrets ne sont pas partagés entre services via des variables d'environnement en clair
- [ ] Le réseau est segmenté (les services ne communiquent pas directement si inutile)
- [ ] Les entrées venant d'autres services sont validées (pas de confiance implicite)

## Observabilité

- [ ] Chaque service expose des métriques (health check, latence, erreurs)
- [ ] Les logs incluent un correlation ID pour tracer les requêtes inter-services
- [ ] Des alertes sont définies sur les SLI/SLO critiques
- [ ] Les traces distribuées sont disponibles pour le debug (OpenTelemetry, Jaeger…)

## Évolutivité (evolutionary architecture)

- [ ] L'ajout d'un nouveau service/module ne nécessite pas de modifier les modules existants
- [ ] Les contrats d'API sont versionnés (pas de breaking change sans nouveau endpoint)
- [ ] Les décisions architecturales majeures sont tracées dans des ADR
- [ ] La roadmap d'architecture (dette technique, migrations) est documentée

## Gouvernance

- [ ] Un rôle d'architecte est formellement défini (personne ou comité)
- [ ] Les choix technologiques majeurs font l'objet d'une validation formelle (RFC, ADR)
- [ ] Des revues d'architecture sont planifiées (au moins une fois par trimestre ou avant chaque feature majeure)
- [ ] Les nouveaux développeurs peuvent comprendre l'architecture depuis la documentation seule

## Score d'évaluation

Compter le nombre de cases cochées :

| % items cochés | Score | Niveau |
|---|---|---|
| ≥ 90 % | 5 | Exemplaire |
| 70–89 % | 4 | Acceptable |
| 50–69 % | 3 | Insuffisant |
| 30–49 % | 2 | Critique |
| < 30 % | 1 | Défaillant |
