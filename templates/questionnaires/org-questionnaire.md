---
type: template
domain: organizational
usage: |
  Ce template est utilisé par l'IA pour générer un questionnaire d'audit
  organisationnel personnalisé. Remplir le profil (profiles/schema.md)
  puis demander : "Génère un questionnaire d'audit organisationnel basé
  sur profiles/[mon-profil].md en utilisant ce template."
---

# Questionnaire d'audit organisationnel — [NOM ORGANISATION]

> Généré le : [DATE]
> Auditeur : [AUDITEUR]
> Contexte : [TRIGGER AUDIT]
> Périmètre : [SCOPE]

---

## Instructions pour l'auditeur

Ce questionnaire est conçu pour des entretiens de 45-60 min avec les responsables techniques.
Format recommandé : semi-directif — poser la question, laisser la personne répondre librement,
puis approfondir avec les sous-questions si nécessaire.

**Interlocuteurs recommandés :**
- [ ] CTO / Lead Tech (toutes les sections)
- [ ] Lead DevOps / SRE (sections Infrastructure et Incidents)
- [ ] Développeur senior (sections Qualité et Processus)
- [ ] Product Manager (section Organisation et Priorisation)

---

## Section 1 — Organisation et gouvernance technique

### 1.1 Structure de l'équipe
1. **Décrivez l'organisation de l'équipe technique.** Qui fait quoi ? Comment êtes-vous organisés ?
   - Combien de personnes dans l'équipe tech ?
   - Y a-t-il des équipes / squads ? Comment sont-elles découpées ?

2. **Comment les décisions techniques importantes sont-elles prises ?**
   - Qui peut prendre quelle décision (architecture, choix techno, sécurité) ?
   - Y a-t-il des comités techniques, des RFCs, des ADRs ?

3. **Quels sont les "single points of knowledge" dans l'équipe ?** (personnes qui seules maîtrisent un système critique)
   - Qu'est-ce qui se passe si cette personne part demain ?

### 1.2 Priorisation et dette technique
4. **Comment la roadmap technique est-elle priorisée par rapport aux features produit ?**
   - Quel % du temps est alloué à la dette technique / l'amélioration ?

5. **Comment la dette technique est-elle identifiée et trackée ?**
   - Est-elle visible dans le backlog ? Qui peut la prioriser ?

---

## Section 2 — Processus de développement

### 2.1 Cycle de développement
6. **Décrivez le cycle de développement, de l'idée à la production.**
   - Combien de temps entre une décision et le déploiement en prod ?
   - Qui peut déployer en production ?

7. **Comment la qualité est-elle assurée avant un merge ?**
   - Y a-t-il des revues de code ? Par qui ? Combien de reviewers ?
   - Les tests sont-ils requis pour merger ?

### 2.2 Tests et qualité
8. **Quel est l'état des tests automatisés ?**
   - Quel est le taux de coverage approximatif ?
   - Y a-t-il des tests d'intégration et/ou end-to-end ?

9. **Y a-t-il un linter / formatter configuré ? Est-il bloquant en CI ?**

---

## Section 3 — Infrastructure et déploiements

### 3.1 Déploiements
10. **Comment les déploiements en production se passent-ils ?**
    - Fréquence ? Qui les déclenche ? Y a-t-il une approbation ?
    - Y a-t-il un processus de rollback ?

11. **Les configurations d'infrastructure sont-elles dans du code (IaC) ?**
    - Terraform, Ansible, CloudFormation ?
    - L'infra est-elle reproductible depuis le code ?

### 3.2 Secrets et sécurité
12. **Comment les secrets (clés API, mots de passe) sont-ils gérés ?**
    - Vault, variables d'environnement, fichiers de config ?
    - Comment les secrets sont-ils rotés ?

---

## Section 4 — Incidents et résilience

13. **Comment les incidents en production sont-ils gérés ?**
    - Y a-t-il une astreinte (on-call) ?
    - Quel est le processus quand quelque chose casse à 2h du matin ?

14. **Y a-t-il des postmortems après les incidents ?**
    - Sont-ils blameless ? Partagés avec l'équipe ?
    - Y a-t-il des runbooks pour les incidents courants ?

15. **Quel est le dernier incident significatif ?** (s'ils acceptent d'en parler)
    - Comment a-t-il été résolu ?
    - Qu'est-ce qui a changé après ?

---

## Section 5 — Monitoring et observabilité

16. **Comment savez-vous que votre système fonctionne correctement en ce moment ?**
    - Logs, métriques, alertes, dashboards ?
    - Recevez-vous des alertes avant que les utilisateurs se plaignent ?

17. **Y a-t-il des SLO (Service Level Objectives) définis ?**

---

## Section 6 — Documentation et onboarding

18. **Si un nouveau développeur rejoint l'équipe demain, combien de temps avant qu'il soit opérationnel ?**
    - Où est la documentation ? Est-elle à jour ?

19. **Comment le savoir est-il transféré dans l'équipe ?**
    - Pair programming, documentation, sessions de partage ?

---

## Grille de scoring

Après l'entretien, noter chaque section de 1 à 5 :

| Section | Score (1-5) | Notes |
|---------|-------------|-------|
| Organisation et gouvernance | | |
| Processus de développement | | |
| Infrastructure et déploiements | | |
| Incidents et résilience | | |
| Monitoring et observabilité | | |
| Documentation et onboarding | | |
| **Score global** | **/30** | |

**Légende :** 1 = Absent / Chaotique | 2 = Émergent | 3 = Défini | 4 = Géré | 5 = Optimisé

---

## Observations et points d'action

### Points forts identifiés
-

### Risques prioritaires
1.
2.
3.

### Recommandations (court terme < 3 mois)
-

### Recommandations (moyen terme 3-12 mois)
-
