---
title: "ADR-0.1 - Décisions d'Architecture Fondatrices"
date: 2026-06-02
draft: false
categories: ["Architecture"]
tags: ["adr", "architecture", "cloud-platform", "gouvernance"]
---

# ADR-001 - Décisions d'Architecture Fondatrices

## Statut

Validé

## Date

Juin 2026

## Projet

MonLabAzure Cloud Platform

---

# 1. Contexte

MonLabAzure Cloud Services est une entreprise fictive de 500 collaborateurs spécialisée dans l'édition et l'hébergement de solutions SaaS.

L'entreprise souhaite moderniser son système d'information et construire une plateforme Cloud Azure répondant aux exigences suivantes :

* Standardisation des déploiements
* Automatisation de l'infrastructure
* Sécurité by Design
* Gouvernance centralisée
* Maîtrise des coûts
* Exploitabilité
* Évolutivité

La plateforme devra être compatible avec un environnement de laboratoire personnel tout en respectant les bonnes pratiques observées dans les environnements de production.

Cette Architecture Decision Record (ADR) constitue le document de référence définissant les choix structurants de la plateforme. Les décisions présentées s'appuient sur les recommandations du Microsoft Cloud Adoption Framework (CAF), du Well-Architected Framework Azure ainsi que sur les principes de gouvernance, de sécurité et d'exploitation généralement appliqués dans les environnements d'entreprise.

---

# 2. Régions Azure

## Décision

Région principale :

* West Europe

Région secondaire :

* North Europe

## Justification

* Régions situées dans l'Union Européenne facilitant la conformité réglementaire et la localisation des données.
* Disponibilité de la majorité des services Azure stratégiques.
* Faible latence entre les deux régions.
* Support des scénarios de reprise après sinistre (PRA/PCA).
* Architecture proche des déploiements observés en production.

## Normes et règles associées

* Les ressources critiques devront être conçues pour supporter une réplication ou une restauration dans la région secondaire.
* Les services Azure utilisés devront être vérifiés au préalable pour confirmer leur disponibilité dans les deux régions.
* Les sauvegardes devront être stockées dans une région distincte lorsque le service le permet.
* Les architectures devront respecter les recommandations de résilience du Microsoft Well-Architected Framework.

---

# 3. Environnements

## Décision

Trois environnements sont retenus :

* DEV
* QUALIF
* PROD

## Justification

* Séparation claire des cycles de développement, validation et production.
* Réduction des risques liés aux changements.
* Cohérence avec les pratiques DevOps et ITIL.
* Maîtrise des coûts tout en conservant un niveau de gouvernance adapté.

## Règles de gestion

### DEV

* Développement et expérimentations.
* Données non sensibles uniquement.
* Niveau de disponibilité non critique.

### QUALIF

* Validation fonctionnelle et technique.
* Tests d'intégration et de sécurité.
* Configuration proche de la production.

### PROD

* Hébergement des charges de travail opérationnelles.
* Contrôle renforcé des accès.
* Sauvegarde et supervision obligatoires.

---

# 4. Gouvernance Azure

## Décision

Organisation autour de cinq abonnements Azure :

### Connectivity

Services réseau partagés.

### Management

Services d'administration et de supervision.

### DEV

Environnement de développement.

### QUALIF

Environnement de qualification.

### PROD

Environnement de production.

## Justification

Cette approche est inspirée du Cloud Adoption Framework et du modèle Enterprise Scale Landing Zone de Microsoft.

Elle permet :

* Une séparation des responsabilités.
* Une meilleure gestion des quotas Azure.
* Une isolation des coûts.
* Une gouvernance simplifiée.
* Une application cohérente des politiques de sécurité.

## Normes de gouvernance

* Les abonnements seront rattachés à une hiérarchie de Management Groups.
* Les Azure Policies seront appliquées au niveau des Management Groups lorsque cela est pertinent.
* Les rôles RBAC devront être attribués via des groupes Microsoft Entra ID.
* Toute exception de gouvernance devra être documentée et approuvée.

---

# 5. Modèle Réseau

## Décision

Architecture Hub & Spoke.

### Hub

Services communs :

* Azure Firewall
* Bastion
* DNS
* Services de connectivité
* Routage centralisé

### Spoke DEV

Ressources de développement.

### Spoke QUALIF

Ressources de validation.

### Spoke PROD

Ressources de production.

## Justification

* Isolation des environnements.
* Réduction de la surface d'attaque.
* Centralisation des services réseau.
* Scalabilité.
* Standard reconnu dans les architectures Azure d'entreprise.

## Normes réseau

* Les communications entre réseaux virtuels devront être contrôlées.
* Les Network Security Groups (NSG) devront être utilisés pour filtrer les flux.
* Les accès administratifs directs depuis Internet seront interdits.
* Les flux sortants devront être contrôlés via Azure Firewall lorsque cela est possible.
* Les plages d'adressage IP devront être documentées et éviter tout chevauchement.

---

# 6. Infrastructure as Code

## Décision

Terraform est l'outil unique de déploiement.

## Justification

* Standard du marché.
* Reproductibilité.
* Gestion du cycle de vie.
* Contrôle des changements.
* Automatisation complète des déploiements.

## Règle

Aucune ressource ne doit être créée manuellement sauf durant le bootstrap initial.

## Normes

* Le code Terraform devra être stocké dans un dépôt Git.
* Toute modification devra passer par une Pull Request.
* Les déploiements devront être exécutés via un pipeline CI/CD.
* Les états Terraform devront être stockés dans un backend distant sécurisé.
* Les modules devront être réutilisables et versionnés.

---

# 7. Convention de Nommage

## Format

----

## Exemple

rg-mla-prod-network-01

## Objectifs

* Lisibilité.
* Recherche simplifiée.
* Cohérence globale.
* Facilitation de l'exploitation et de l'automatisation.

## Règles complémentaires

* Utiliser uniquement des caractères autorisés par Azure.
* Éviter les abréviations ambiguës.
* Respecter les limites de longueur imposées par chaque service Azure.
* Maintenir une nomenclature homogène sur l'ensemble de la plateforme.

---

# 8. Stratégie de Tags

## Tags obligatoires

### Environment

Valeurs :

* dev
* qualif
* prod

### Owner

Équipe responsable.

### Project

MonLabAzure

### CostCenter

Centre de coût.

### Criticality

Valeurs :

* low
* medium
* high

### ManagedBy

Terraform

## Justification

* FinOps.
* Gouvernance.
* Reporting.
* Gestion du cycle de vie des ressources.

## Normes

* Les tags obligatoires devront être contrôlés par Azure Policy.
* Toute ressource dépourvue des tags requis devra être considérée comme non conforme.
* Les valeurs devront être normalisées afin de garantir la qualité des rapports.

---

# 9. Sécurité

## Principes

* Least Privilege.
* RBAC.
* Managed Identity.
* Secrets dans Key Vault.
* Segmentation réseau.
* Chiffrement par défaut.
* Zero Trust.
* Séparation des responsabilités.

## Services prévus

* Azure Policy.
* Azure Key Vault.
* Microsoft Defender for Cloud.
* Azure Firewall.
* Azure Bastion.
* Microsoft Entra ID.

## Normes de sécurité

* L'authentification multifacteur (MFA) devra être activée pour tous les comptes administrateurs.
* Les comptes à privilèges devront être limités et régulièrement audités.
* Les secrets ne devront jamais être stockés dans le code source.
* Les données devront être chiffrées au repos et en transit.
* Les recommandations critiques de Defender for Cloud devront être traitées en priorité.
* Les accès devront être journalisés et conservés conformément aux exigences de supervision.

---

# 10. Supervision

## Services retenus

* Azure Monitor.
* Log Analytics Workspace.
* Alertes Azure.
* Dashboards.
* Application Insights pour les applications.

## Objectif

Centraliser la supervision de la plateforme et améliorer la détection proactive des incidents.

## Normes

* Toutes les ressources compatibles devront envoyer leurs journaux vers Log Analytics.
* Les alertes critiques devront être définies dès la mise en production.
* Les tableaux de bord devront permettre le suivi de la disponibilité, des performances et de la sécurité.
* Les journaux devront être conservés selon une durée adaptée aux besoins opérationnels et budgétaires.

---

# 11. FinOps

## Principes

Chaque série devra inclure :

* Estimation des coûts.
* Optimisation possible.
* Impact budgétaire.
* Procédure de suppression.

## Objectif

Maintenir une plateforme compatible avec un budget de laboratoire tout en appliquant les principes FinOps.

## Bonnes pratiques

* Utilisation des SKU les plus adaptés au besoin réel.
* Suppression des ressources inutilisées.
* Analyse régulière des coûts Azure.
* Utilisation des tags pour la répartition budgétaire.
* Documentation des arbitrages entre coût, performance et disponibilité.

---

# 12. Stratégie Applicative

## Phase 1

Landing Zone Enterprise.

Objectifs :

* Mise en place de la gouvernance.
* Déploiement du socle réseau.
* Mise en œuvre des contrôles de sécurité.

## Phase 2

Application Web classique :

* App Service.
* Azure SQL.
* Key Vault.

Objectifs :

* Hébergement d'une application métier.
* Gestion sécurisée des secrets.
* Mise en œuvre des sauvegardes et de la supervision.

## Phase 3

Modernisation Cloud Native :

* Azure Container Registry.
* AKS.
* API Management.

Objectifs :

* Conteneurisation des applications.
* Automatisation du cycle de vie logiciel.
* Exposition sécurisée des API.
* Préparation à une architecture orientée microservices.

---

# 13. Critères de Réussite

La plateforme devra permettre :

* Déploiement automatisé.
* Sécurisation des accès.
* Supervision centralisée.
* Contrôle des coûts.
* Haute disponibilité des composants critiques.
* Évolution vers une architecture Cloud Native.

## Indicateurs de validation

* 100 % des ressources déployées via Terraform.
* Respect des conventions de nommage et de tags.
* Absence de ressources critiques exposées directement à Internet.
* Couverture de supervision sur l'ensemble des composants principaux.
* Conformité aux politiques de sécurité définies.

---

# 14. Références

Les décisions décrites dans cet ADR s'appuient notamment sur :

* Microsoft Cloud Adoption Framework.
* Azure Well-Architected Framework.
* Azure Landing Zone Architecture.
* Microsoft Defender for Cloud Best Practices.
* Azure Architecture Center.

Tous les articles futurs devront respecter les décisions définies dans ADR-001.

Toute modification majeure devra faire l'objet d'un nouvel ADR.
