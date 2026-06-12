---

title: "SERIES-2.6 - Contrôler les Ressources Exposées Publiquement avec Azure Policy"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-2"
* "Azure Policy"
* "Security"
* "Governance"
* "Terraform"
* "Landing Zone"
* "Azure"

---

# SERIES-2.6 – Contrôler les Ressources Exposées Publiquement avec Azure Policy

## Introduction

Jusqu'à présent, nous avons renforcé la gouvernance de notre Landing Zone Azure grâce à plusieurs contrôles :

* tags obligatoires ;
* régions autorisées ;
* Diagnostic Settings.

Nous allons maintenant aborder un sujet particulièrement important :

> l'exposition publique des ressources Azure.

Dans de nombreux incidents de sécurité Cloud, le problème n'est pas une vulnérabilité technique mais une mauvaise configuration.

Une ressource exposée publiquement sans justification représente un risque important pour l'entreprise.

Azure Policy permet d'identifier ces situations et de renforcer progressivement la sécurité de la plateforme.

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
SERIES-2.6   Ressources Publiques                   ⏳
SERIES-2.7   Initiative MonLabAzure Governance
SERIES-2.8   Validation Governance Foundation
```

---

# Pourquoi Contrôler les Ressources Publiques ?

Dans Azure, plusieurs services peuvent être exposés directement sur Internet :

* Storage Accounts ;
* Azure SQL ;
* Key Vault ;
* App Services ;
* Machines Virtuelles ;
* Kubernetes ;
* API Management.

Sans gouvernance :

```text
Internet
    │
    ▼
Ressource Azure
```

Le risque est alors :

* fuite de données ;
* exposition involontaire ;
* non-conformité ;
* augmentation de la surface d'attaque.

---

# Stratégie MonLabAzure

Notre architecture cible privilégie :

```text
Internet
    │
    ▼
Application Gateway
    │
    ▼
Services Privés
```

et

```text
Private Endpoint
```

lorsque cela est possible.

---

# Pourquoi Commencer par Audit ?

Dans une Landing Zone existante, certaines ressources publiques peuvent être légitimes :

* site web ;
* API publique ;
* portail client.

Nous devons d'abord :

* identifier ;
* inventorier ;
* analyser ;

avant d'interdire.

---

# Objectif de l'Article

Déployer une Azure Policy permettant :

```text
Audit
```

des configurations publiques non souhaitées.

---

# Architecture de Gouvernance

La Policy sera affectée au niveau :

```text
mla-landingzones
```

et héritée automatiquement par :

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
└── 06-public-access-policies
```

Structure :

```text
06-public-access-policies
│
├── backend.tf
├── versions.tf
├── providers.tf
├── main.tf
├── outputs.tf
└── terraform.tfvars
```

---

# Backend Terraform

## backend.tf

```hcl
# Backend Terraform dédié aux Policies de sécurité.
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "public-access-policies.tfstate"
  }
}
```

---

# Première Policy de Sécurité

Pour cet article, nous allons nous concentrer sur les Storage Accounts.

Pourquoi ?

Parce qu'ils représentent l'une des sources les plus fréquentes d'exposition involontaire de données.

---

# Récupération de la Policy Intégrée Azure

## main.tf

```hcl
# Policy Microsoft permettant d'auditer
# l'accès public aux Storage Accounts.
data "azurerm_policy_definition" "storage_public_access" {
  display_name = "Storage accounts should prevent public blob access"
}
```

---

# Affectation de la Policy

```hcl
# Affectation de la Policy au niveau Landing Zones.
resource "azurerm_management_group_policy_assignment" "storage_public_access" {

  name = "audit-storage-public-access"

  management_group_id =
    "/providers/Microsoft.Management/managementGroups/mla-landingzones"

  policy_definition_id =
    data.azurerm_policy_definition.storage_public_access.id
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
1 resource to add
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
└── Assignments
     │
     └── audit-storage-public-access
```

---

# Cas Conforme

Storage Account :

```text
Allow Blob Public Access = Disabled
```

Résultat :

```text
Compliant
```

---

# Cas Non Conforme

Storage Account :

```text
Allow Blob Public Access = Enabled
```

Résultat :

```text
Non Compliant
```

---

# Évolution Future

Aujourd'hui :

```text
Audit
```

Demain :

```text
Deny
```

Une fois la plateforme stabilisée, certaines Policies pourront être renforcées.

---

# Préparation des Futures Phases

Cette Policy prépare directement :

```text
Phase 4 – Hub & Spoke Networking
Phase 5 – Connectivity & Security Services
Phase 7 – Secrets & Platform Security
```

où nous mettrons en œuvre :

* Azure Firewall ;
* Private Endpoints ;
* Bastion ;
* Key Vault privé ;
* Azure SQL privé.

---

# Dépannage

## Policy Introuvable

Lister les Policies disponibles :

```bash
az policy definition list --query "[].displayName"
```

---

## Résultats de Conformité Absents

Attendre la première évaluation Azure Policy.

Le délai peut varier de quelques minutes à plusieurs dizaines de minutes.

---

# Coûts

Cette Policy n'entraîne aucun coût direct.

Coût estimé :

```text
0 €
```

---

# Livrables

À l'issue de cet article :

✅ Première Policy de sécurité

✅ Contrôle des Storage Accounts publics

✅ Gouvernance renforcée

✅ Préparation des futures architectures privées

✅ Landing Zone plus sécurisée

---

# Conclusion

Nous avons ajouté un premier contrôle de sécurité permettant d'identifier les Storage Accounts exposés publiquement.

Cette approche s'inscrit dans une stratégie globale visant à réduire progressivement la surface d'attaque de la plateforme.

Dans le prochain article, nous regrouperons toutes les Policies déployées jusqu'à présent dans une initiative unique : **MonLabAzure Governance Baseline**.
