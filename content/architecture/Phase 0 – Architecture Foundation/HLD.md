---
title: "HLD - Architecture Globale MonLabAzure Cloud Platform"
date: 2026-06-02
draft: false
categories: ["Architecture"]
tags:
  - "Architecture"
  - "Cloud Platform"
  - "Azure"
  - "Landing Zone"
  - "HLD"
---

# HLD - Architecture Globale MonLabAzure Cloud Platform

## 1. Objectif du HLD

Ce High Level Design présente la vue d'ensemble de l'architecture cible du projet MonLabAzure Cloud Platform.

Il ne décrit pas encore les détails d'implémentation Terraform. Son objectif est de donner une vision claire des grandes briques d'architecture, de leur rôle et de leurs interactions.

---

## 2. Vue d'ensemble

La plateforme repose sur les principes suivants :

* Gouvernance centralisée avec Management Groups.
* Séparation des abonnements par responsabilité.
* Architecture réseau Hub & Spoke.
* Supervision centralisée.
* Sécurité by design.
* Déploiement Infrastructure as Code avec Terraform.
* Séparation des environnements DEV, QUALIF et PROD.
* Préparation à une évolution Cloud Native.

---

## 3. Vue logique globale

```text
Microsoft Entra ID Tenant
│
└── Tenant Root Group
    │
    ├── mla-platform
    │   │
    │   ├── mla-connectivity
    │   │   └── sub-mla-connectivity
    │   │       ├── Hub VNet
    │   │       ├── Azure Firewall
    │   │       ├── Azure Bastion
    │   │       ├── Private DNS Zones
    │   │       └── VPN / ExpressRoute
    │   │
    │   └── mla-management
    │       └── sub-mla-management
    │           ├── Log Analytics Workspace
    │           ├── Azure Monitor
    │           ├── Alerts
    │           ├── Dashboards
    │           └── Automation
    │
    ├── mla-landingzones
    │   │
    │   ├── mla-dev
    │   │   └── sub-mla-dev
    │   │       ├── DEV Spoke VNet
    │   │       ├── App Services
    │   │       ├── Key Vault
    │   │       └── Azure SQL
    │   │
    │   ├── mla-qualif
    │   │   └── sub-mla-qualif
    │   │       ├── QUALIF Spoke VNet
    │   │       ├── App Services
    │   │       ├── Key Vault
    │   │       └── Azure SQL
    │   │
    │   └── mla-prod
    │       └── sub-mla-prod
    │           ├── PROD Spoke VNet
    │           ├── Application Gateway
    │           ├── App Services
    │           ├── Key Vault
    │           ├── Azure SQL
    │           └── Future AKS / APIM
    │
    └── mla-sandbox
        └── Optional Sandbox Subscription
```

---

## 4. Vue réseau cible

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
                        Hub VNet
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
     DEV Spoke         QUALIF Spoke       PROD Spoke
          │                 │                 │
          ▼                 ▼                 ▼
   Workloads DEV     Workloads QUALIF   Workloads PROD
```

Le Hub centralise les services réseau communs.

Les Spokes hébergent les ressources applicatives propres à chaque environnement.

---

## 5. Vue sécurité

```text
Microsoft Entra ID
│
├── Groupes RBAC
│   ├── Platform Admins
│   ├── Network Admins
│   ├── Security Admins
│   ├── Dev Contributors
│   └── Prod Operators
│
├── Azure Policy
│   ├── Régions autorisées
│   ├── Tags obligatoires
│   ├── Diagnostic settings
│   └── Restrictions accès public
│
├── Key Vault
│   ├── Secrets applicatifs
│   ├── Certificats
│   └── Clés
│
└── Defender for Cloud
    ├── Recommandations
    ├── Secure Score
    └── Alertes sécurité
```

Principes appliqués :

* Aucun rôle critique directement affecté à un utilisateur.
* RBAC via groupes Microsoft Entra ID.
* Secrets stockés dans Key Vault.
* Accès administratifs via Bastion.
* Réduction des expositions publiques.
* Supervision des événements de sécurité.

---

## 6. Vue observabilité

```text
Ressources Azure
│
├── Logs d'activité
├── Logs de diagnostic
├── Métriques
└── Logs applicatifs
        │
        ▼
Log Analytics Workspace
        │
        ├── Azure Monitor
        ├── Alertes
        ├── Dashboards
        ├── Application Insights
        └── Defender for Cloud
```

Objectifs :

* Centraliser les logs.
* Détecter les incidents.
* Suivre les performances.
* Faciliter les diagnostics.
* Alimenter les tableaux de bord d'exploitation.

---

## 7. Vue Terraform

```text
monlabazure-cloud-platform
│
└── terraform
    │
    ├── 00-bootstrap
    ├── 01-management-groups
    ├── 02-policies
    ├── 03-connectivity
    ├── 04-management
    ├── 05-dev
    ├── 06-qualif
    ├── 07-prod
    └── modules
```

Chaque couche Terraform possède son propre state.

Objectif :

* Limiter les impacts.
* Séparer les responsabilités.
* Faciliter les déploiements.
* Préparer l'intégration CI/CD.

---

## 8. Vue applicative cible

### Phase 1 : Web App classique

```text
Utilisateur
    │
    ▼
Application Gateway
    │
    ▼
App Service
    │
    ├── Key Vault
    └── Azure SQL
```

Objectif :

Déployer une première application métier simple, sécurisée et supervisée.

---

### Phase 2 : Modernisation Cloud Native

```text
Utilisateur
    │
    ▼
Application Gateway
    │
    ▼
API Management
    │
    ▼
AKS
    │
    ├── Azure Container Registry
    ├── Key Vault
    ├── Azure SQL
    └── Log Analytics
```

Objectif :

Faire évoluer progressivement l'architecture vers une plateforme Cloud Native.

---

## 9. Vue FinOps

```text
Ressources Azure
│
├── Tags obligatoires
│   ├── Project
│   ├── Environment
│   ├── Owner
│   ├── CostCenter
│   ├── Criticality
│   ├── ManagedBy
│   ├── DataClassification
│   └── BackupRequired
│
└── Cost Management
    ├── Analyse par environnement
    ├── Analyse par équipe
    ├── Analyse par criticité
    └── Optimisation continue
```

Chaque série devra inclure :

* une estimation des coûts ;
* une stratégie de nettoyage ;
* une alternative Lab économique ;
* une analyse des arbitrages coût / sécurité / disponibilité.

---

## 10. Flux principaux

### Flux d'administration

```text
Administrateur
    │
    ▼
Microsoft Entra ID
    │
    ▼
RBAC
    │
    ▼
Azure Bastion
    │
    ▼
Ressources privées
```

---

### Flux applicatif

```text
Utilisateur
    │
    ▼
Application Gateway
    │
    ▼
Application Backend
    │
    ├── Key Vault
    └── Azure SQL
```

---

### Flux supervision

```text
Ressources Azure
    │
    ▼
Diagnostic Settings
    │
    ▼
Log Analytics
    │
    ▼
Alertes / Dashboards
```

---

## 11. Décisions structurantes associées

Ce HLD s'appuie sur les documents suivants :

* ADR-001 : Décisions d'architecture fondatrices.
* ADR-002 : Hiérarchie des Management Groups.
* ADR-003 : Organisation Git, Terraform et states.
* ADR-004 : Convention de nommage.
* ADR-005 : Tags, FinOps et classification des données.

---

## 12. Conclusion

Ce HLD constitue la vue d'architecture globale du projet MonLabAzure Cloud Platform.

Il servira de référence visuelle et fonctionnelle pour les futures séries techniques.

Les prochaines étapes consisteront à transformer cette architecture cible en implémentation Terraform progressive, en commençant par la Landing Zone Foundation.
