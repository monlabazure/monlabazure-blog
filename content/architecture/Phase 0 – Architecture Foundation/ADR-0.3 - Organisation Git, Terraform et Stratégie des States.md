---
title: "ADR-0.3 - Organisation Git, Terraform et Stratégie des States"
date: 2026-06-03
draft: false
categories: ["Architecture"]
tags: ["adr", "architecture", "cloud-platform", "git", "terraform"]
---

# ADR-003 - Organisation Git, Terraform et Stratégie des States

## Statut

Validé

## Date

Juin 2026

## Projet

MonLabAzure Cloud Platform

---

# 1. Contexte

Le projet MonLabAzure Cloud Platform repose sur le principe fondamental suivant :

> Toute l'infrastructure Azure doit être déployée, modifiée et supprimée via Terraform.

L'objectif est de garantir :

* La reproductibilité des déploiements.
* La traçabilité des changements.
* L'automatisation.
* La standardisation.
* La réduction des erreurs humaines.

Afin de maintenir la plateforme sur le long terme, une organisation Git et Terraform cohérente doit être définie dès le début du projet.

---

# 2. Objectifs

Cette organisation doit permettre :

* Une séparation logique des composants.
* Une gestion indépendante des états Terraform.
* Une maintenance simplifiée.
* Une évolutivité de l'architecture.
* Une compatibilité avec les bonnes pratiques DevOps.
* Une intégration future avec CI/CD.

---

# 3. Décision

Le projet utilisera :

* Un dépôt Git principal.
* Plusieurs couches Terraform.
* Un backend distant Azure Storage.
* Un state Terraform par domaine fonctionnel.
* Des modules Terraform réutilisables.

---

# 4. Organisation Git

## Dépôt principal

Nom recommandé :

```text
monlabazure-cloud-platform
```

Ce dépôt constitue la source de vérité du projet.

---

## Structure cible

```text
monlabazure-cloud-platform
│
├── docs
│   ├── adr
│   ├── architecture
│   ├── diagrams
│   └── runbooks
│
├── terraform
│   ├── 00-bootstrap
│   ├── 01-management-groups
│   ├── 02-policies
│   ├── 03-connectivity
│   ├── 04-management
│   ├── 05-dev
│   ├── 06-qualif
│   ├── 07-prod
│   ├── 08-shared-services
│   ├── 09-security
│   ├── 10-app-platform
│   └── modules
│
├── scripts
│
├── examples
│
└── README.md
```

---

# 5. Principes d'Organisation Terraform

Chaque couche Terraform représente un domaine fonctionnel autonome.

Objectif :

* Réduire la taille des states.
* Limiter les impacts des changements.
* Simplifier les opérations Terraform.

Chaque couche possède :

* Son code.
* Son backend.
* Son cycle de vie.

---

# 6. Couche 00-Bootstrap

## Objectif

Créer les éléments nécessaires à Terraform lui-même.

Ressources typiques :

* Resource Group Terraform
* Storage Account Terraform
* Container Blob Terraform
* Configuration initiale

## Particularité

Cette couche peut être créée manuellement lors du premier déploiement.

C'est la seule exception à la règle Terraform First.

---

# 7. Couche 01-Management-Groups

## Objectif

Déployer :

* Management Groups
* Rattachements des abonnements
* Gouvernance initiale

State dédié :

```text
tfstate-management-groups
```

---

# 8. Couche 02-Policies

## Objectif

Déployer :

* Azure Policies
* Policy Initiatives
* Assignments
* Exemptions

State dédié :

```text
tfstate-policies
```

---

# 9. Couche 03-Connectivity

## Objectif

Déployer :

* Hub VNet
* Azure Firewall
* Bastion
* VPN Gateway
* ExpressRoute
* DNS

State dédié :

```text
tfstate-connectivity
```

---

# 10. Couche 04-Management

## Objectif

Déployer :

* Log Analytics
* Azure Monitor
* Alertes
* Dashboards
* Automation

State dédié :

```text
tfstate-management
```

---

# 11. Environnements Applicatifs

## DEV

Répertoire :

```text
05-dev
```

State :

```text
tfstate-dev
```

---

## QUALIF

Répertoire :

```text
06-qualif
```

State :

```text
tfstate-qualif
```

---

## PROD

Répertoire :

```text
07-prod
```

State :

```text
tfstate-prod
```

---

# 12. Modules Terraform

Tous les composants réutilisables seront stockés dans :

```text
terraform/modules
```

Exemple :

```text
modules
│
├── resource-group
├── vnet
├── subnet
├── nsg
├── route-table
├── firewall
├── bastion
├── keyvault
├── log-analytics
├── app-service
├── sql
├── aks
└── apim
```

---

# 13. Règles de Développement des Modules

Chaque module doit contenir :

```text
main.tf
variables.tf
outputs.tf
README.md
versions.tf
```

Objectifs :

* Réutilisabilité
* Documentation
* Maintenabilité

---

# 14. Backend Terraform

## Solution retenue

Azure Storage Account.

Structure :

```text
Storage Account
│
└── tfstate
    ├── management-groups.tfstate
    ├── policies.tfstate
    ├── connectivity.tfstate
    ├── management.tfstate
    ├── dev.tfstate
    ├── qualif.tfstate
    └── prod.tfstate
```

---

# 15. Sécurisation du Backend

Règles :

* Chiffrement activé.
* Soft Delete activé.
* Versioning activé.
* Accès RBAC uniquement.
* Secrets interdits dans le code.

Évolutions futures possibles :

* Private Endpoint.
* Customer Managed Key.
* Replication GRS.

---

# 16. Gestion des Branches Git

## Branche principale

```text
main
```

Contient uniquement du code validé.

---

## Branche de développement

```text
develop
```

Contient les travaux en cours.

---

## Branches de fonctionnalités

Format :

```text
feature/nom-fonctionnalite
```

Exemple :

```text
feature/hub-network
feature/azure-firewall
feature/aks-platform
```

---

# 17. Workflow Git

Principe :

```text
feature
    ↓
pull request
    ↓
develop
    ↓
validation
    ↓
main
```

Objectifs :

* Contrôle qualité
* Relecture
* Historisation

---

# 18. Intégration Continue

Même si le projet débute sans pipeline complet, les contrôles suivants devront être prévus :

* terraform fmt
* terraform validate
* terraform plan

Évolutions futures :

* GitHub Actions
* Azure DevOps
* Contrôles de sécurité
* Scans Terraform

---

# 19. Gestion des Versions Terraform

Version Terraform imposée :

Exemple :

```text
>= 1.11
```

Version AzureRM :

Exemple :

```text
~> 4.0
```

Toutes les versions devront être figées dans :

```text
versions.tf
```

---

# 20. Gestion des Secrets

Interdictions :

* Secrets dans Git.
* Secrets dans variables.tf.
* Secrets dans tfvars versionnés.

Solution retenue :

* Azure Key Vault.
* Variables d'environnement.
* Managed Identities.

---

# 21. Justification

Cette architecture présente plusieurs avantages :

* Compatible Enterprise.
* Compatible Lab.
* Facile à comprendre.
* Facile à maintenir.
* Évolutive.
* Adaptée aux futures séries.

Elle permet également de limiter les impacts liés aux erreurs Terraform en cloisonnant les states.

---

# 22. Conséquences

## Positives

* États Terraform isolés.
* Réduction des risques.
* Réutilisation des modules.
* Architecture évolutive.
* Déploiements plus sûrs.

## Négatives

* Nombre de states plus important.
* Gestion des dépendances entre couches.
* Complexité initiale légèrement supérieure.

---

# 23. Critères de Validation

ADR-003 sera considéré comme implémenté lorsque :

* Le dépôt Git principal sera créé.
* L'arborescence Terraform sera mise en place.
* Le backend Azure Storage sera opérationnel.
* Les premiers states seront séparés.
* Les modules Terraform de base existeront.
* Les conventions Git seront appliquées.

---

# 24. Références

* Terraform Best Practices
* Azure Landing Zone
* Microsoft Cloud Adoption Framework
* Azure Well-Architected Framework
* HashiCorp Terraform Documentation

---

# 25. Décision Finale

Le projet MonLabAzure Cloud Platform adopte une organisation Git centralisée, des couches Terraform indépendantes et une séparation stricte des états Terraform.

Cette structure devient la référence officielle pour tous les futurs développements Terraform du projet.
