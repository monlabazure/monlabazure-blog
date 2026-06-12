---
title: "Cahier des Charges et Architecture Cible"
date: 2026-06-02
draft: false
categories: ["Architecture"]
tags:
  - "Architecture"
  - "Cloud Platform"
  - "Azure"
  - "Landing Zone"
---

# SERIES-000 - Cahier des Charges et Architecture Cible

## Introduction

Bienvenue dans la première série technique du projet MonLabAzure Cloud Platform.

Les articles précédents ont permis de définir :

* la vision du projet ;
* les principes d'architecture ;
* la gouvernance Azure ;
* l'organisation Terraform ;
* les conventions de nommage ;
* la stratégie FinOps.

L'objectif de cette série est de formaliser le périmètre fonctionnel et technique de la plateforme qui sera construite tout au long du projet.

Cette architecture servira de référence pour l'ensemble des futures séries.

---

# 1. Présentation de l'entreprise

## Société

MonLabAzure Cloud Services

## Activité

MonLabAzure Cloud Services est un éditeur de solutions SaaS spécialisé dans l'hébergement et l'exploitation d'applications métiers.

L'entreprise fournit plusieurs services numériques à ses clients et souhaite moderniser son système d'information afin d'améliorer :

* l'agilité ;
* la sécurité ;
* l'automatisation ;
* l'observabilité ;
* la maîtrise des coûts.

---

## Taille

* 500 collaborateurs
* 15 personnes dans l'équipe IT
* 4 ingénieurs Cloud
* Plusieurs équipes de développement

---

# 2. Objectifs du projet

La direction souhaite construire une plateforme Cloud Azure répondant aux objectifs suivants :

## Gouvernance

* Standardiser les déploiements.
* Contrôler les accès.
* Garantir la conformité.

## Sécurité

* Appliquer les principes Zero Trust.
* Réduire la surface d'exposition.
* Centraliser les contrôles de sécurité.

## Exploitation

* Superviser l'ensemble de la plateforme.
* Détecter rapidement les incidents.
* Automatiser les opérations récurrentes.

## FinOps

* Contrôler les coûts Azure.
* Identifier les ressources inutilisées.
* Optimiser la consommation.

## Évolutivité

* Préparer l'arrivée de nouvelles applications.
* Préparer la migration vers des architectures Cloud Native.

---

# 3. Contraintes du projet

## Techniques

* Azure uniquement.
* Terraform obligatoire.
* Git comme source de vérité.
* Infrastructure as Code systématique.

## Organisationnelles

* Séparation DEV / QUALIF / PROD.
* Gouvernance centralisée.
* Contrôle des changements.

## Budgétaires

* Compatible avec un laboratoire personnel.
* Utilisation temporaire des services coûteux.
* Destruction des ressources après validation lorsque nécessaire.

---

# 4. Architecture Azure Cible

## Régions

### Région Principale

West Europe

### Région Secondaire

North Europe

---

## Abonnements

### Connectivity

Services réseau partagés.

### Management

Services d'exploitation et de supervision.

### DEV

Développement.

### QUALIF

Qualification.

### PROD

Production.

---

# 5. Hiérarchie de Gouvernance

```text
Tenant Root Group
│
├── Platform
│   ├── Connectivity
│   └── Management
│
├── LandingZones
│   ├── DEV
│   ├── QUALIF
│   └── PROD
│
└── Sandbox
```

---

# 6. Architecture Réseau Cible

```text
Internet
    │
    ▼
Application Gateway
    │
    ▼
Azure Firewall
    │
    ▼
Hub Network
    │
 ┌──┴──────────────┐
 ▼                 ▼
DEV Spoke      QUALIF Spoke
                    │
                    ▼
              PROD Spoke
```

Le modèle retenu est Hub & Spoke.

Objectifs :

* Isolation des environnements.
* Contrôle des flux.
* Réduction de la surface d'attaque.
* Mutualisation des services réseau.

---

# 7. Services Communs

## Connectivity

* Azure Firewall
* Azure Bastion
* VPN Gateway
* ExpressRoute (série dédiée)
* DNS Resolver
* Private DNS Zones

## Management

* Azure Monitor
* Log Analytics
* Alertes
* Dashboards
* Automation

---

# 8. Sécurité

## Principes

* Zero Trust
* Least Privilege
* RBAC
* MFA
* Managed Identity
* Chiffrement par défaut

## Services

* Microsoft Entra ID
* Azure Policy
* Azure Firewall
* Azure Key Vault
* Microsoft Defender for Cloud
* Azure Bastion

---

# 9. Stratégie de Supervision

Tous les composants devront être supervisés.

Objectifs :

* Détection proactive.
* Analyse des performances.
* Analyse des coûts.
* Centralisation des journaux.

Services retenus :

* Azure Monitor
* Log Analytics Workspace
* Application Insights
* Defender for Cloud

---

# 10. Stratégie FinOps

Chaque ressource devra :

* Posséder les tags obligatoires.
* Être justifiée.
* Être dimensionnée correctement.
* Être supprimée lorsqu'elle n'est plus utilisée.

Chaque série inclura :

* Estimation budgétaire.
* Optimisations possibles.
* Procédure de destruction.

---

# 11. Architecture Applicative Cible

Phase 1 :

Landing Zone Enterprise.

Phase 2 :

Application Web moderne :

* App Service
* Azure SQL
* Key Vault
* Managed Identity

Phase 3 :

Modernisation :

* Azure Container Registry
* AKS
* API Management
* Private Endpoints

---

# 12. Roadmap du Projet

## Foundation

* INTRO-001
* ADR-001 à ADR-005
* SERIES-000

## Gouvernance

* Management Groups
* Azure Policy
* RBAC

## Réseau

* Hub & Spoke
* Bastion
* Firewall
* DNS

## Observabilité

* Log Analytics
* Azure Monitor
* Alerting

## Sécurité

* Defender for Cloud
* Key Vault
* Managed Identity

## Applications

* Web App
* Azure SQL

## Cloud Native

* Containers
* ACR
* AKS
* APIM

## Résilience

* Sauvegardes
* PRA
* Tests de restauration

## Optimisation

* FinOps
* Gouvernance avancée
* Durcissement

---

# 13. Critères de Réussite

Le projet sera considéré comme réussi lorsque :

* 100 % de l'infrastructure sera déployée avec Terraform.
* Les environnements DEV, QUALIF et PROD seront opérationnels.
* Les contrôles de sécurité seront en place.
* La supervision sera centralisée.
* Les coûts seront maîtrisés.
* Une application complète fonctionnera sur la plateforme.
* Une version Cloud Native basée sur AKS sera disponible.

---

# Conclusion

Cette architecture constitue la cible officielle du projet MonLabAzure Cloud Platform.

Les futures séries auront pour objectif de construire progressivement chacun des composants décrits dans ce document tout en respectant les décisions d'architecture définies dans les ADR précédents.

La prochaine étape consistera à mettre en œuvre les premiers composants de gouvernance Azure en construisant la Landing Zone Foundation.
