---

title: "SERIES-2.7 - Créer l'Initiative Azure Policy MonLabAzure Governance Baseline"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:
  * "Phase-2"
  * "Azure Policy"
  * "Initiative"
  * "Governance"
  * "Landing Zone"
  * "Terraform"
  * "Compliance"
  * "Azure"

---

# SERIES-2.7 – Créer l'Initiative Azure Policy MonLabAzure Governance Baseline

## Introduction

Dans les articles précédents, nous avons déployé plusieurs Azure Policies afin de renforcer la gouvernance de notre Landing Zone :

* Tags obligatoires
* Régions autorisées
* Diagnostic Settings
* Contrôle des ressources publiques

Cette approche est idéale pour comprendre chaque mécanisme individuellement.

Cependant, dans un environnement Enterprise, les Policies sont généralement regroupées dans une Initiative Azure Policy.

Cette approche simplifie :

* le déploiement ;
* l'administration ;
* les audits ;
* les rapports de conformité.

Dans cet article, nous allons construire notre première initiative de gouvernance.

---

# Positionnement dans le Projet

```text
Phase 2 – Governance & Security Foundation

ADR-006   Stratégie Azure Policy et Gouvernance      ✅

SERIES-2.1   Introduction Azure Policy              ✅
SERIES-2.2   Premier Tag Obligatoire                ✅
SERIES-2.3   Industrialiser les Tags                ✅
SERIES-2.4   Régions Autorisées                     ✅
SERIES-2.5   Diagnostic Settings                    ✅
SERIES-2.6   Ressources Publiques                   ✅
SERIES-2.7   Initiative Governance Baseline         ⏳
SERIES-2.8   Validation Governance Foundation
```

---

# Qu'est-ce qu'une Initiative Azure Policy ?

Une Initiative est un regroupement logique de plusieurs Azure Policies.

Au lieu de gérer :

```text
Policy A
Policy B
Policy C
Policy D
```

nous gérons :

```text
MonLabAzure Governance Baseline
│
├── Policy A
├── Policy B
├── Policy C
└── Policy D
```

Cette approche est utilisée dans la majorité des Landing Zones Enterprise.

---

# Pourquoi Utiliser une Initiative ?

## Simplification

Une seule affectation.

---

## Cohérence

Toutes les règles sont regroupées.

---

## Reporting

Vue consolidée de la conformité.

---

## Évolutivité

Ajout de nouvelles Policies sans modifier les affectations existantes.

---

# Composition de l'Initiative

Notre initiative contiendra :

```text
MonLabAzure Governance Baseline
│
├── Tags Obligatoires
├── Régions Autorisées
├── Diagnostic Settings
└── Contrôle Ressources Publiques
```

---

# Architecture de Gouvernance

L'initiative sera affectée au niveau :

```text
mla-landingzones
```

Elle sera héritée automatiquement par :

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
└── 07-governance-initiative
```

Structure :

```text
07-governance-initiative
│
├── backend.tf
├── versions.tf
├── providers.tf
├── locals.tf
├── main.tf
├── outputs.tf
└── terraform.tfvars
```

---

# Backend Terraform

## backend.tf

```hcl
# Backend Terraform dédié aux initiatives de gouvernance.
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "governance-initiative.tfstate"
  }
}
```

---

# Récupération des Policies

## main.tf

```hcl
# Policy intégrée pour les régions autorisées.
data "azurerm_policy_definition" "allowed_locations" {
  display_name = "Allowed locations"
}

# Policy intégrée pour les Storage Accounts publics.
data "azurerm_policy_definition" "storage_public_access" {
  display_name = "Storage accounts should prevent public blob access"
}
```

---

# Création de l'Initiative

```hcl
# Initiative principale de gouvernance MonLabAzure.
resource "azurerm_policy_set_definition" "governance_baseline" {

  name         = "mla-governance-baseline"
  policy_type  = "Custom"
  display_name = "MonLabAzure Governance Baseline"

  metadata = jsonencode({
    category = "Governance"
  })

  # Politique des régions autorisées.
  policy_definition_reference {

    policy_definition_id =
      data.azurerm_policy_definition.allowed_locations.id

    reference_id = "AllowedLocations"
  }

  # Politique d'accès public Storage.
  policy_definition_reference {

    policy_definition_id =
      data.azurerm_policy_definition.storage_public_access.id

    reference_id = "StoragePublicAccess"
  }
}
```

---

# Affectation de l'Initiative

```hcl
# Affectation de l'initiative au niveau Landing Zones.
resource "azurerm_management_group_policy_assignment" "governance_baseline" {

  name = "mla-governance-baseline"

  management_group_id =
    "/providers/Microsoft.Management/managementGroups/mla-landingzones"

  policy_definition_id =
    azurerm_policy_set_definition.governance_baseline.id
}
```

---

# Vérification Terraform

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
2 resources to add
```

---

# Déploiement

```bash
terraform apply
```

Validation :

```text
Apply complete!
```

---

# Vérification dans Azure

Portail Azure :

```text
Azure Policy
│
└── Initiatives
     │
     └── MonLabAzure Governance Baseline
```

---

# Résultat Attendu

```text
MonLabAzure Governance Baseline
│
├── Allowed Locations
├── Storage Public Access
├── Tags (futur enrichissement)
└── Diagnostic Settings (futur enrichissement)
```

---

# Pourquoi "Future Enrichissement" ?

Pour simplifier les premiers articles, nous avons créé certaines Policies séparément.

Dans un projet réel, l'initiative sera progressivement enrichie :

```text
Version 1
│
├── Régions
└── Sécurité

Version 2
│
├── Régions
├── Sécurité
├── Tags
└── Diagnostic Settings
```

---

# Bonnes Pratiques Enterprise

Une initiative doit rester :

* cohérente ;
* documentée ;
* versionnée ;
* alignée sur les standards d'entreprise.

Éviter les initiatives "fourre-tout" contenant des dizaines de Policies sans logique métier.

---

# Dépannage

## Policy Set Introuvable

Vérifier :

```bash
terraform state list
```

---

## Affectation Vide

Vérifier :

```bash
az policy set-definition list --output table
```

---

## Conformité Non Visible

Attendre l'évaluation Azure Policy.

---

# Coûts

L'initiative elle-même n'entraîne aucun coût direct.

Coût estimé :

```text
0 €
```

---

# Livrables

À l'issue de cet article :

✅ Première Initiative Azure Policy

✅ Gouvernance centralisée

✅ Affectation unique

✅ Héritage via Management Groups

✅ Alignement avec les Azure Landing Zones

---

# Conclusion

Nous avons transformé plusieurs Policies indépendantes en une approche de gouvernance centralisée et évolutive.

Cette initiative constitue désormais la référence de conformité de notre plateforme.

Dans le prochain article, nous réaliserons l'audit complet de la Phase 2 afin de valider la Governance & Security Foundation avant de passer à la Phase 3 – Identity & Access Management.
