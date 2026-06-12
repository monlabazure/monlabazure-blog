# SERIES-1.5 – Rattachement des Abonnements aux Management Groups avec Terraform

## Introduction

Dans l'article précédent, nous avons créé la hiérarchie des Management Groups qui constitue le socle de gouvernance de notre plateforme Azure.

Nous disposons désormais de la structure suivante :

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

Cependant, cette hiérarchie reste vide.

Pour qu'elle soit réellement utile, nous devons maintenant rattacher nos abonnements Azure aux Management Groups correspondants.

Cette étape est fondamentale car elle conditionne :

* l'héritage des Azure Policies ;
* l'héritage RBAC ;
* la gouvernance ;
* l'organisation des futures ressources Azure.

---

# Positionnement dans le Projet

```text
Phase 1 - Landing Zone Foundation

SERIES-001-001  Introduction                      ✅
SERIES-001-002  Préparer le tenant Azure          ✅
SERIES-001-003  Bootstrap Terraform               ✅
SERIES-001-004  Management Groups                 ✅
SERIES-001-005  Rattachement abonnements          ⏳
SERIES-001-006  Validation Foundation
```

---

# 1. Objectif

À l'issue de cet article :

* Les abonnements seront rattachés à la bonne branche.
* L'architecture définie dans ADR-002 sera pleinement opérationnelle.
* Les futures Azure Policies pourront être héritées automatiquement.
* Les futures affectations RBAC pourront être centralisées.

---

# 2. Architecture Cible

Nous souhaitons obtenir :

```text
Tenant Root Group
│
├── mla-platform
│   ├── mla-connectivity
│   │   └── sub-mla-connectivity
│   │
│   └── mla-management
│       └── sub-mla-management
│
├── mla-landingzones
│   ├── mla-dev
│   │   └── sub-mla-dev
│   │
│   ├── mla-qualif
│   │   └── sub-mla-qualif
│   │
│   └── mla-prod
│       └── sub-mla-prod
│
└── mla-sandbox
```

---

# 3. Prérequis

Les éléments suivants doivent être disponibles :

* Backend Terraform opérationnel.
* Management Groups créés.
* Abonnements existants.
* Permissions suffisantes sur les abonnements et les Management Groups.

---

# 4. Récupération des Identifiants des Abonnements

Lister les abonnements :

```bash
az account subscription list --output table
```

Exemple :

```text
Name                   SubscriptionId
---------------------------------------------------
sub-mla-connectivity   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
sub-mla-management     xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
sub-mla-dev            xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
sub-mla-qualif         xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
sub-mla-prod           xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Conserver ces identifiants.

---

# 5. Organisation Terraform

Créer le répertoire :

```text
terraform/
└── 02-subscription-associations
```

Structure :

```text
02-subscription-associations
│
├── backend.tf
├── versions.tf
├── providers.tf
├── variables.tf
├── terraform.tfvars
├── main.tf
└── outputs.tf
```

---

# 6. Backend Terraform

## backend.tf

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "subscription-associations.tfstate"
  }
}
```

---

# 7. Variables

## variables.tf

```hcl
variable "connectivity_subscription_id" {
  type = string
}

variable "management_subscription_id" {
  type = string
}

variable "dev_subscription_id" {
  type = string
}

variable "qualif_subscription_id" {
  type = string
}

variable "prod_subscription_id" {
  type = string
}
```

---

## terraform.tfvars

```hcl
connectivity_subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

management_subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

dev_subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

qualif_subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

prod_subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

---

# 8. Création des Associations

Terraform utilise la ressource :

```hcl
azurerm_management_group_subscription_association
```

---

## main.tf

### Connectivity

```hcl
resource "azurerm_management_group_subscription_association" "connectivity" {
  management_group_id = "/providers/Microsoft.Management/managementGroups/mla-connectivity"
  subscription_id     = "/subscriptions/${var.connectivity_subscription_id}"
}
```

---

### Management

```hcl
resource "azurerm_management_group_subscription_association" "management" {
  management_group_id = "/providers/Microsoft.Management/managementGroups/mla-management"
  subscription_id     = "/subscriptions/${var.management_subscription_id}"
}
```

---

### DEV

```hcl
resource "azurerm_management_group_subscription_association" "dev" {
  management_group_id = "/providers/Microsoft.Management/managementGroups/mla-dev"
  subscription_id     = "/subscriptions/${var.dev_subscription_id}"
}
```

---

### QUALIF

```hcl
resource "azurerm_management_group_subscription_association" "qualif" {
  management_group_id = "/providers/Microsoft.Management/managementGroups/mla-qualif"
  subscription_id     = "/subscriptions/${var.qualif_subscription_id}"
}
```

---

### PROD

```hcl
resource "azurerm_management_group_subscription_association" "prod" {
  management_group_id = "/providers/Microsoft.Management/managementGroups/mla-prod"
  subscription_id     = "/subscriptions/${var.prod_subscription_id}"
}
```

---

# 9. Initialisation

```bash
terraform init
```

Résultat attendu :

```text
Terraform has been successfully initialized
```

---

# 10. Vérification du Plan

```bash
terraform plan
```

Contrôles :

* 5 ressources à créer.
* Aucun abonnement supprimé.
* Aucun déplacement inattendu.

---

# 11. Déploiement

```bash
terraform apply
```

Validation :

```text
Apply complete!
Resources: 5 added.
```

---

# 12. Vérification dans Azure

Lister les abonnements :

```bash
az account subscription list --output table
```

Puis vérifier dans le portail Azure :

```text
Management Groups
│
├── mla-platform
│   ├── mla-connectivity
│   │   └── sub-mla-connectivity
│   │
│   └── mla-management
│       └── sub-mla-management
│
├── mla-landingzones
│   ├── mla-dev
│   │   └── sub-mla-dev
│   │
│   ├── mla-qualif
│   │   └── sub-mla-qualif
│   │
│   └── mla-prod
│       └── sub-mla-prod
│
└── mla-sandbox
```

---

# 13. Pourquoi ce Rattachement est Important

À partir de maintenant :

## Azure Policies

Les futures policies appliquées sur :

```text
mla-landingzones
```

seront héritées automatiquement par :

* DEV
* QUALIF
* PROD

---

## RBAC

Les futures affectations RBAC pourront être réalisées au niveau :

```text
mla-platform
```

ou

```text
mla-landingzones
```

et être héritées automatiquement.

---

## Gouvernance

Nous disposons désormais d'une structure cohérente et évolutive.

---

# 14. Dépannage

## SubscriptionNotFound

Vérifier :

```bash
az account subscription list
```

---

## AuthorizationFailed

Le compte doit disposer :

* des droits sur le Management Group ;
* des droits sur l'abonnement.

---

## Management Group Not Found

Vérifier :

```bash
az account management-group list --output table
```

---

# 15. Coûts

Le rattachement d'un abonnement à un Management Group :

```text
Coût Azure = 0 €
```

Aucune facturation n'est associée à cette opération.

---

# 16. Nettoyage

Suppression :

```bash
terraform destroy
```

Attention :

Cette opération détache les abonnements mais ne supprime aucun abonnement Azure.

---

# 17. Livrables

À la fin de cet article :

✅ Hiérarchie des Management Groups créée

✅ Cinq abonnements rattachés

✅ Héritage prêt pour Azure Policy

✅ Héritage prêt pour RBAC

✅ Gouvernance Azure opérationnelle

---

# 18. Conclusion

Nous avons terminé la construction de la structure de gouvernance principale de notre Landing Zone Azure.

Les abonnements sont désormais organisés conformément aux décisions définies dans ADR-002 et prêts à recevoir les futures politiques de gouvernance.

Dans le prochain article, nous réaliserons la validation complète de la Landing Zone Foundation et préparerons le déploiement des premières Azure Policies de la plateforme.
