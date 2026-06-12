# SERIES-1.4 – Déployer la Hiérarchie des Management Groups avec Terraform

## Introduction

Dans l'article précédent, nous avons mis en place le backend Terraform qui servira à stocker l'état de l'ensemble de notre plateforme Azure.

Nous disposons désormais :

* d'un backend distant Azure Storage ;
* d'un dépôt Git structuré ;
* d'un environnement Terraform opérationnel.

Nous pouvons maintenant commencer à matérialiser notre architecture Azure.

La première brique de gouvernance que nous allons déployer est la hiérarchie des Management Groups définie dans ADR-002.

---

# Positionnement dans le Projet

```text
Phase 1 - Landing Zone Foundation

SERIES-001-001  Introduction                      ✅
SERIES-001-002  Préparer le tenant Azure          ✅
SERIES-001-003  Bootstrap Terraform               ✅
SERIES-001-004  Management Groups                 ⏳
SERIES-001-005  Rattachement des abonnements
SERIES-001-006  Validation Foundation
```

---

# 1. Pourquoi les Management Groups ?

Les Management Groups permettent d'organiser les abonnements Azure selon une structure hiérarchique.

Ils servent notamment à :

* appliquer des Azure Policies ;
* gérer les accès RBAC ;
* organiser les abonnements ;
* appliquer des standards de gouvernance.

Sans eux, chaque abonnement devrait être administré individuellement.

---

# 2. Architecture Cible

Conformément à ADR-002 :

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

À ce stade :

* aucun abonnement n'est encore rattaché ;
* aucune Policy n'est appliquée ;
* aucun RBAC spécifique n'est configuré.

Nous construisons uniquement la structure.

---

# 3. Prérequis

Avant de commencer :

* Terraform initialisé ;
* Backend Terraform opérationnel ;
* Permissions suffisantes sur le tenant Azure ;
* ADR-002 validé.

---

# 4. Arborescence Terraform

Créer le répertoire :

```text
terraform/
└── 01-management-groups
```

Structure :

```text
01-management-groups
│
├── backend.tf
├── versions.tf
├── providers.tf
├── variables.tf
├── outputs.tf
├── main.tf
└── terraform.tfvars
```

---

# 5. Configuration du Provider

## versions.tf

```hcl
terraform {
  required_version = ">= 1.11"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}
```

---

## providers.tf

```hcl
provider "azurerm" {
  features {}
}
```

---

# 6. Configuration du Backend

## backend.tf

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "management-groups.tfstate"
  }
}
```

---

# 7. Identifier le Tenant Root Group

Avant de créer les Management Groups, nous devons récupérer l'identifiant du Tenant Root Group.

Commande :

```bash
az account management-group show \
  --name TenantRootGroup
```

Exemple :

```json
{
  "id": "/providers/Microsoft.Management/managementGroups/TenantRootGroup",
  "name": "TenantRootGroup"
}
```

Conserver cette valeur.

---

# 8. Variables Terraform

## variables.tf

```hcl
variable "root_management_group_id" {
  type = string
}
```

---

## terraform.tfvars

```hcl
root_management_group_id = "TenantRootGroup"
```

---

# 9. Création des Management Groups

## main.tf

### Platform

```hcl
resource "azurerm_management_group" "platform" {
  display_name               = "mla-platform"
  parent_management_group_id = "/providers/Microsoft.Management/managementGroups/${var.root_management_group_id}"
}
```

---

### Connectivity

```hcl
resource "azurerm_management_group" "connectivity" {
  display_name               = "mla-connectivity"
  parent_management_group_id = azurerm_management_group.platform.id
}
```

---

### Management

```hcl
resource "azurerm_management_group" "management" {
  display_name               = "mla-management"
  parent_management_group_id = azurerm_management_group.platform.id
}
```

---

### Landing Zones

```hcl
resource "azurerm_management_group" "landingzones" {
  display_name               = "mla-landingzones"
  parent_management_group_id = "/providers/Microsoft.Management/managementGroups/${var.root_management_group_id}"
}
```

---

### DEV

```hcl
resource "azurerm_management_group" "dev" {
  display_name               = "mla-dev"
  parent_management_group_id = azurerm_management_group.landingzones.id
}
```

---

### QUALIF

```hcl
resource "azurerm_management_group" "qualif" {
  display_name               = "mla-qualif"
  parent_management_group_id = azurerm_management_group.landingzones.id
}
```

---

### PROD

```hcl
resource "azurerm_management_group" "prod" {
  display_name               = "mla-prod"
  parent_management_group_id = azurerm_management_group.landingzones.id
}
```

---

### Sandbox

```hcl
resource "azurerm_management_group" "sandbox" {
  display_name               = "mla-sandbox"
  parent_management_group_id = "/providers/Microsoft.Management/managementGroups/${var.root_management_group_id}"
}
```

---

# 10. Outputs

## outputs.tf

```hcl
output "platform_management_group_id" {
  value = azurerm_management_group.platform.id
}

output "landingzones_management_group_id" {
  value = azurerm_management_group.landingzones.id
}
```

---

# 11. Initialisation

```bash
terraform init
```

Résultat attendu :

```text
Terraform has been successfully initialized
```

---

# 12. Vérification du Plan

```bash
terraform plan
```

Contrôles :

* 8 ressources à créer ;
* aucune modification ;
* aucune suppression.

---

# 13. Déploiement

```bash
terraform apply
```

Validation :

```text
Apply complete!
Resources: 8 added.
```

---

# 14. Vérification dans Azure

Lister les Management Groups :

```bash
az account management-group list --output table
```

Résultat attendu :

```text
mla-platform
mla-connectivity
mla-management
mla-landingzones
mla-dev
mla-qualif
mla-prod
mla-sandbox
```

---

# 15. Vérification Visuelle

Dans le portail Azure :

```text
Management Groups
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

La hiérarchie doit être identique à celle définie dans ADR-002.

---

# 16. Dépannage

## Permissions insuffisantes

Erreur typique :

```text
AuthorizationFailed
```

Cause :

Le compte ne possède pas les droits nécessaires sur le Tenant Root Group.

---

## Mauvais Parent ID

Erreur :

```text
ManagementGroupNotFound
```

Vérifier :

```bash
az account management-group show \
  --name TenantRootGroup
```

---

## Backend inaccessible

Vérifier :

* Resource Group Bootstrap
* Storage Account
* Container tfstate

---

# 17. Coûts

Les Management Groups sont gratuits.

Coût généré :

```text
0 €
```

Aucune facturation Azure n'est associée aux Management Groups.

---

# 18. Nettoyage

Destruction :

```bash
terraform destroy
```

Attention :

Ne jamais exécuter cette commande sur une plateforme utilisée en production.

---

# 19. Livrables

À l'issue de cet article :

✅ Backend Terraform utilisé

✅ Premier déploiement Terraform réalisé

✅ Hiérarchie de gouvernance créée

✅ Structure conforme à ADR-002

✅ Architecture prête à recevoir les abonnements

---

# 20. Conclusion

Nous avons créé la première brique réelle de gouvernance Azure de la plateforme MonLabAzure Cloud Platform.

La hiérarchie des Management Groups est désormais opérationnelle et servira de socle pour les futures Azure Policies, affectations RBAC et rattachements d'abonnements.

Dans le prochain article, nous rattacherons les abonnements Connectivity, Management, DEV, QUALIF et PROD aux Management Groups appropriés afin de finaliser la Landing Zone Foundation.
