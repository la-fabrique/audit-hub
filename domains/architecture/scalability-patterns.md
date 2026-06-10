---
domain: architecture
category: scalability
severity: high
applicable-to: [web, api, saas, cloud, microservices]
tags: [scalabilité, performance, bottleneck, cache, async, database, load-balancing]
related:
  - ./coupling-and-dependencies.md
  - ./checklists/architecture-review-checklist.md
ai-prompt-hints: |
  Demander : diagramme d'architecture actuel, volumes actuels (RPS, users, données),
  points de croissance prévus, incidents de performance récents.
  Chercher : couplage synchrone entre services, single points of failure,
  absence de cache, requêtes N+1, transactions longues, queues saturées.
  Questions clés : "Qu'est-ce qui casse en premier si le trafic x10 ?" et
  "Quel est le composant le plus difficile à scaler horizontalement ?"
---

# Scalabilité et patterns d'architecture

## Anti-patterns fréquents

| Anti-pattern | Symptôme | Impact |
|---|---|---|
| Couplage synchrone en cascade | Latence en cascade, timeout chain | Haut |
| Absence de cache | Requêtes DB répétitives identiques | Haut |
| Requêtes N+1 | Latence exponentiellement liée aux données | Critique |
| Single point of failure (SPOF) | Indisponibilité complète sur panne composant | Critique |
| Base de données monolithique | Goulot d'étranglement à l'écriture | Haut |
| Sessions en mémoire locale | Impossible de scaler horizontalement | Haut |

## Patterns recommandés

### Découplage
- **Async par défaut** pour les opérations non-temps-réel (email, notifs, exports)
- **Message queue** (RabbitMQ, Kafka, SQS) pour absorber les pics
- **Circuit breaker** (Hystrix, Resilience4j) sur les appels inter-services
- **Bulkhead** : isoler les ressources par domaine pour éviter la propagation de panne

### Cache
- Stratégie **read-through** pour les données lues fréquemment
- TTL adapté : données utilisateur (5-15 min), données référentielles (1h+)
- Invalidation : event-driven > polling
- Redis en cluster pour le cache distribué

### Base de données
- **Read replicas** pour les workloads lecture-intensive
- **Connection pooling** (PgBouncer pour PostgreSQL)
- Index sur les colonnes utilisées en `WHERE`, `JOIN`, `ORDER BY`
- Pagination curseur > pagination offset pour les grands datasets

### Stateless
- Sessions externalisées (Redis, JWT)
- Configuration par variables d'environnement
- Artefacts en stockage objet (S3, GCS), jamais en local

## Ce que l'agent IA peut vérifier

### Analyse du code et des configurations

```bash
# Sessions en mémoire locale (bloque le scaling horizontal)
grep -rn "MemoryStore\|new session.MemoryStore" --include="*.js" --include="*.ts"
# Présence d'un cache distribué dans les dépendances / configs
grep -rni "redis\|memcached" --include="package.json" --include="*.yml" --include="*.toml"
# Queues / jobs asynchrones
grep -rni "bullmq\|celery\|sidekiq\|sqs\|rabbitmq\|kafka" --include="package.json" --include="*.yml" --include="*.toml"
# Connection pooling configuré
grep -rni "pgbouncer\|pool_size\|max.*connections\|connectionLimit" --include="*.yml" --include="*.env*" --include="*.ts" --include="*.py"
# Réplication : nombre de replicas dans les manifests
grep -rn "replicas:" k8s/ docker-compose*.yml 2>/dev/null
```

### Revue d'architecture / documentation

- Lire le diagramme d'architecture fourni : identifier les composants sans réplique (SPOF)
- Vérifier la configuration du load balancer et des health checks
- Examiner les requêtes ORM des endpoints les plus sollicités (N+1 : requêtes dans des boucles, lazy loading non maîtrisé)

### Questions d'audit (entretien)

1. Quels sont les 3 composants les plus chargés ? Y a-t-il des métriques ?
2. Comment le système se comporte-t-il à 10x le trafic actuel ?
3. Y a-t-il des SPOF identifiés ? Sont-ils documentés et acceptés ?
4. Comment les sessions sont-elles stockées ? Peut-on scaler les app servers ?
5. Quelle est la stratégie de cache ? Y a-t-il des métriques de hit rate ?
6. Y a-t-il des jobs/tâches asynchrones ? Comment sont-ils monitorés ?

## Matrice de maturité scalabilité

| Niveau | Description |
|--------|-------------|
| 1 - Monolithe | Un seul process, DB locale, pas de cache |
| 2 - Découplage basique | Cache introduit, quelques jobs async |
| 3 - Scalable horizontalement | Stateless, sessions externalisées, queues |
| 4 - Résilient | Circuit breakers, bulkheads, chaos testing |
| 5 - Auto-scalable | Orchestration (K8s), auto-scaling, SLO définis |

## Références

- [AWS Well-Architected Framework — Reliability](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/)
- [Martin Fowler — Patterns of Enterprise Application Architecture](https://martinfowler.com/eaaCatalog/)
- [The Twelve-Factor App](https://12factor.net/)
