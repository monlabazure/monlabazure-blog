---
title: "ADR-0.2 - Hiérarchie des Management Groups Azure"
date: 2026-06-03
draft: false
categories: ["Architecture"]
tags: ["adr", "architecture", "cloud-platform", "gouvernance"]
---

# ADR-002 - Hiérarchie des Management Groups Azure

## Statut

Validé

## Date

Juin 2026

## Projet

MonLabAzure Cloud Platform

---

# 1. Contexte

Dans le cadre du projet MonLabAzure Cloud Platform, l'entreprise fictive MonLabAzure Cloud Services souhaite organiser son environnement Azure selon une structure proche des recommandations Microsoft Enterprise Scale Landing Zone.

L'objectif est de disposer d'une hiérarchie claire permettant de gérer :

* Les abonnements Azure
* Les Azure Policies
* Les rôles RBAC
* La séparation des responsabilités
* Les environnements DEV, QUALIF et PROD
* Les services partagés de connectivité et de supervision

Cette organisation doit rester compatible avec un laboratoire personnel tout en respectant les principes de gouvernance, de sécurité et d'exploitation utilisés dans les environnements professionnels.

---

# 2. Hypothèses

Les décisions décrites dans cet ADR reposent sur les hypothèses suivantes :

* Un seul tenant Microsoft Entra ID.
* Une seule organisation.
* Aucun besoin de séparation réglementaire par pays.
* Aucun scénario de fusion ou acquisition.
* Les abonnements Azure sont rattachés au même tenant.
* L'architecture est déployée principalement dans deux régions Azure :

  * West Europe
  * North Europe
* Le projet vise une architecture Enterprise simplifiée adaptée à un environnement de laboratoire.

Toute évolution remettant en cause ces hypothèses devra faire l'objet d'une révision de cet ADR.

---

# 3. Décision

La plateforme adopte une hiérarchie de Management Groups inspirée du modèle Microsoft Enterprise Scale Landing Zone.

Structure cible :

```text
Tenant Root Group
│
├── mla-platform
│   ├── mla-connectivity
│   └── mla-management
│
├── mla-landingzones
│   ├── mla-dev
│   ├── mla-qualif
│   └── mla-prod
│
└── mla-sandbox
```

Cette structure constitue le socle de gouvernance de la plateforme.

---

# 4. Description des Management Groups

## Tenant Root Group

Niveau racine du tenant Azure.

Responsabilités :

* Gouvernance globale
* Azure Policy globale
* Contrôles de conformité
* Héritage RBAC

Règle :

Les affectations RBAC doivent rester limitées à quelques groupes d'administration de haut niveau.

---

## mla-platform

Regroupe les services techniques partagés.

Objectifs :

* Centraliser les services d'infrastructure.
* Séparer la plateforme des workloads.
* Faciliter la gestion des services communs.

Sous-groupes :

* mla-connectivity
* mla-management

---

## mla-connectivity

Contient l'abonnement réseau partagé.

Abonnement associé :

* sub-mla-connectivity

Services prévus :

* Hub Network
* Azure Firewall
* Azure Bastion
* VPN Gateway
* ExpressRoute (série dédiée)
* Private DNS Zones
* DNS Resolver
* Routage centralisé

---

## mla-management

Contient l'abonnement de gestion et de supervision.

Abonnement associé :

* sub-mla-management

Services prévus :

* Azure Monitor
* Log Analytics Workspace
* Dashboards
* Alertes
* Automation
* Centralisation des logs

---

## mla-landingzones

Regroupe les abonnements hébergeant les charges applicatives.

Sous-groupes :

* mla-dev
* mla-qualif
* mla-prod

Objectifs :

* Héberger les workloads.
* Isoler les environnements.
* Appliquer une gouvernance cohérente.

---

## mla-dev

Abonnement associé :

* sub-mla-dev

Usage :

* Développement
* Tests techniques
* Validation du code Terraform
* Ressources non critiques

---

## mla-qualif

Abonnement associé :

* sub-mla-qualif

Usage :

* Qualification
* Validation fonctionnelle
* Tests d'intégration
* Validation avant production

---

## mla-prod

Abonnement associé :

* sub-mla-prod

Usage :

* Hébergement des charges de travail opérationnelles
* Sécurité renforcée
* Sauvegardes obligatoires
* Supervision obligatoire

---

## mla-sandbox

Zone dédiée aux expérimentations.

Objectifs :

* Tests ponctuels
* Validation de nouveaux services Azure
* POC
* Démonstrations techniques

Règles :

* Durée de vie limitée des ressources
* Budget contrôlé
* Nettoyage obligatoire après utilisation
* Restrictions de sécurité allégées par rapport à PROD

---

# 5. Rattachement des abonnements

```text
mla-platform
│
├── mla-connectivity
│   └── sub-mla-connectivity
│
└── mla-management
    └── sub-mla-management

mla-landingzones
│
├── mla-dev
│   └── sub-mla-dev
│
├── mla-qualif
│   └── sub-mla-qualif
│
└── mla-prod
    └── sub-mla-prod
```

---

# 6. Stratégie Azure Policy

## Niveau Tenant Root Group

Politiques globales :

* Régions autorisées
* Tags obligatoires
* Audit du chiffrement
* Audit des diagnostics
* Contrôle des ressources publiques

---

## Niveau mla-platform

Politiques infrastructure :

* Diagnostic obligatoire
* Restriction des ressources critiques
* Contrôle des accès publics
* Journalisation obligatoire

---

## Niveau mla-landingzones

Politiques workloads :

* Tags obligatoires
* Contrôle des régions
* Contrôle des SKU
* Audit des ressources exposées
* Contrôle des Private Endpoints

---

## Niveau mla-prod

Politiques renforcées :

* Sauvegarde obligatoire
* Diagnostics obligatoires
* Restrictions accrues sur les ressources publiques
* Contrôles de conformité renforcés

---

# 7. Stratégie RBAC

Principes :

* Aucun rôle attribué directement à un utilisateur.
* Utilisation systématique des groupes Microsoft Entra ID.
* Principe du moindre privilège.
* Séparation des responsabilités.
* Revue régulière des accès.

Groupes de référence :

```text
AZ-MLA-Platform-Admins
AZ-MLA-Network-Admins
AZ-MLA-Security-Admins
AZ-MLA-Monitoring-Admins
AZ-MLA-Dev-Contributors
AZ-MLA-Qualif-Contributors
AZ-MLA-Prod-Operators
AZ-MLA-Readers
```

---

# 8. Modèle d'Héritage RBAC

## Tenant Root Group

Accès extrêmement restreints.

Groupes autorisés :

* AZ-MLA-Platform-Admins
* AZ-MLA-Security-Admins

---

## mla-platform

Groupes autorisés :

* AZ-MLA-Platform-Admins
* AZ-MLA-Network-Admins
* AZ-MLA-Monitoring-Admins

---

## mla-landingzones

Groupes autorisés :

* AZ-MLA-Platform-Admins
* AZ-MLA-Dev-Contributors
* AZ-MLA-Qualif-Contributors
* AZ-MLA-Prod-Operators

---

## mla-prod

Les privilèges Contributor doivent être limités.

Les accès de production doivent être contrôlés et documentés.

---

# 9. Matrice des Responsabilités

| Domaine      | Équipe Cloud | Équipe Réseau | Équipe Sécurité |
| ------------ | ------------ | ------------- | --------------- |
| Connectivity | R            | A             | C               |
| Management   | R            | C             | A               |
| DEV          | A            | C             | C               |
| QUALIF       | A            | C             | C               |
| PROD         | R            | C             | A               |

Légende :

* R = Responsable
* A = Approbateur
* C = Consulté

---

# 10. Évolutions Futures

La structure retenue doit permettre les évolutions suivantes :

* Ajout d'un Management Group Identity
* Création d'un abonnement Security dédié
* Création d'un abonnement Shared Services
* Extension vers plusieurs Landing Zones métiers
* Architecture multi-région active/active
* Renforcement des contrôles Zero Trust

Ces évolutions ne remettent pas en cause la structure actuelle.

---

# 11. Justification

Cette architecture :

* Respecte les principes du Cloud Adoption Framework.
* S'aligne sur Enterprise Scale Landing Zone.
* Reste adaptée à un environnement de laboratoire.
* Facilite la gouvernance.
* Prépare les futures séries du projet.
* Simplifie l'application des politiques et des contrôles de sécurité.

---

# 12. Conséquences

## Positives

* Gouvernance claire
* Séparation des responsabilités
* Meilleure visibilité des coûts
* Contrôle centralisé
* Architecture évolutive

## Négatives

* Complexité supérieure à un abonnement unique
* Gestion de plusieurs abonnements
* Multiplication des états Terraform
* Documentation plus importante

---

# 13. Critères de Validation

L'ADR est considéré comme implémenté lorsque :

* Les Management Groups sont créés.
* Les abonnements sont correctement rattachés.
* Les groupes Entra ID de référence existent.
* Les premières Azure Policies sont appliquées.
* Aucun rôle critique n'est attribué directement à un utilisateur.
* La structure est reproductible avec Terraform.

---

# 14. Références

* Microsoft Cloud Adoption Framework
* Azure Landing Zone Architecture
* Azure Management Groups
* Azure Policy
* Azure RBAC
* Microsoft Entra ID
* Azure Well-Architected Framework

---

# 15. Décision Finale

La hiérarchie de Management Groups définie dans cet ADR est approuvée comme structure de gouvernance officielle du projet MonLabAzure Cloud Platform.

Toute modification majeure devra faire l'objet d'un nouvel ADR.
