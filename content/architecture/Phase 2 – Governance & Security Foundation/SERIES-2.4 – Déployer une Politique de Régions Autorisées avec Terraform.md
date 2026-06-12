---

title: "SERIES-2.4 - Déployer une Politique de Régions Autorisées avec Terraform"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-2"
* "Azure Policy"
* "Terraform"
* "Governance"
* "Compliance"
* "Landing Zone"
* "Azure"

---

# SERIES-2.4 - Déployer une Politique de Régions Autorisées avec Terraform

## Introduction

Après avoir mis en place les contrôles de tags obligatoires, nous allons maintenant renforcer la gouvernance de notre Landing Zone Azure en contrôlant les régions autorisées.

Dans de nombreuses entreprises, les ressources Azure ne peuvent pas être déployées librement dans n'importe quelle région du monde.

Les raisons sont multiples :

* conformité réglementaire ;
* souveraineté des données ;
* réduction des risques ;
* optimisation des coûts ;
* standardisation des architectures.

Azure Policy permet d'imposer cette règle de manière centralisée.

---

# Positionnement dans le Projet

```text
Phase 2 – Governance & Security Foundation

ADR-006   Stratégie Azure Policy et Gouvernance      ✅

SERIES-2.1   Introduction Azure Policy              ✅
SERIES-2.2   Premier Tag Obligatoire                ✅
SERIES-2.3   Industrialiser les Tags                ✅
SERIES-2.4   Régions Autorisées                     ⏳
SERIES-2.5   Diagnostic Settings
SERIES-2.6   Contrôle des Ressources Publiques
SERIES-2.7   Initiative MonLabAzure Governance
SERIES-2.8   Validation Governance Foundation
```

---

# Pourquoi Contrôler les Régions ?

Sans gouvernance :

```text
West Europe
North Europe
East US
Japan East
Australia East
Brazil South
```

peuvent être utilisés simultanément.

Cela entraîne :

* dispersion des ressources ;
* complexité réseau ;
* difficultés de conformité ;
* augmentation des coûts opérationnels.

---

# Stratégie MonLabAzure

Conformément à notre architecture :

## Région Principale

```text
West Europe
```

---

## Région de Secours

```text
North Europe
```

---

Toutes les autres régions devront être signalées comme non conformes.

---

# Approche Retenue

Comme pour les tags :

Nous commencerons en :

```text
Audit
```

et non :

```text
Deny
```

afin d'éviter les blocages pendant les premières phases du projet.

---

# Architecture de Gouvernance

La Policy sera affectée au niveau :

```text
mla-landingzones
```

Elle sera automatiquement héritée par :

```text
mla-dev
mla-qualif
mla-prod
```

---

# Arborescence Terraform

Créer :

```text
terraform/
└── 04-region-policies
```

Structure :

```text
04-region-policies
│
├── backend.tf
├── versions.tf
├── providers.tf
├── variables.tf
├── locals.tf
├── main.tf
├── outputs.tf
└── terraform.tfvars
```

---

# Backend Terraform

## backend.tf

```hcl
# Backend Terraform stocké dans Azure Storage.
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "region-policies.tfstate"
  }
}
```

---

# Définition des Régions Autorisées

## locals.tf

```hcl
locals {

  # Régions approuvées pour la plateforme MonLabAzure.
  allowed_locations = [
    "westeurope",
    "northeurope"
  ]
}
```

---

# Récupération de la Policy Intégrée Azure

## main.tf

```hcl
# Policy Microsoft permettant de limiter les régions de déploiement.
data "azurerm_policy_definition" "allowed_locations" {
  display_name = "Allowed locations"
}
```

---

# Affectation de la Policy

```hcl
# Affectation de la Policy au niveau des Landing Zones.
resource "azurerm_management_group_policy_assignment" "allowed_locations" {

  name = "audit-allowed-locations"

  management_group_id =
    "/providers/Microsoft.Management/managementGroups/mla-landingzones"

  policy_definition_id =
    data.azurerm_policy_definition.allowed_locations.id

  parameters = jsonencode({
    listOfAllowedLocations = {
      value = local.allowed_locations
    }
  })
}
```

---

# Vérification

```bash
terraform init
```

```bash
terraform validate
```

```bash
terraform plan
```

Résultat attendu :

```text
1 resource to add
```

---

# Déploiement

```bash
terraform apply
```

---

# Validation dans Azure

Portail Azure :

```text
Azure Policy
│
└── Assignments
     │
     └── audit-allowed-locations
```

---

# Test de Conformité

## Cas Conforme

Création :

```text
West Europe
```

Résultat :

```text
Compliant
```

---

## Cas Conforme

Création :

```text
North Europe
```

Résultat :

```text
Compliant
```

---

## Cas Non Conforme

Création :

```text
East US
```

Résultat :

```text
Non Compliant
```

---

# Pourquoi Commencer par Audit ?

Avant d'interdire une région :

* il faut connaître l'existant ;
* identifier les écarts ;
* mesurer les impacts.

La transition vers :

```text
Deny
```

sera étudiée dans une phase ultérieure.

---

# Dépannage

## Policy Non Visible

Attendre quelques minutes.

La propagation Azure Policy peut nécessiter un délai.

---

## Mauvaise Région Déclarée

Vérifier :

```hcl
locals {
  allowed_locations = [
    "westeurope",
    "northeurope"
  ]
}
```

Les noms doivent correspondre aux noms Azure et non aux noms affichés dans le portail.

---

## Terraform Sans Changement

Vérifier :

```bash
terraform state list
```

---

# Coûts

Azure Policy ne génère aucun coût direct.

Coût estimé :

```text
0 €
```

---

# Livrables

À l'issue de cet article :

✅ Contrôle des régions activé

✅ Héritage via Management Groups

✅ Première gouvernance géographique

✅ Terraform industrialisé

✅ Landing Zone renforcée

---

# Conclusion

Nous avons ajouté un nouveau contrôle de gouvernance permettant de limiter les déploiements aux régions approuvées de la plateforme.

Cette Policy constitue une étape importante dans la maîtrise des risques et la standardisation de notre environnement Azure.

Dans le prochain article, nous mettrons en place une Policy destinée à contrôler la présence des Diagnostic Settings afin d'améliorer l'observabilité et la conformité de la plateforme.
