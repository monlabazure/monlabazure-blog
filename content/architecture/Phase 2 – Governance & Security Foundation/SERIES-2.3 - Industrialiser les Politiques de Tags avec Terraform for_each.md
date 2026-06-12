---

title: "SERIES-2.3 - Industrialiser les Politiques de Tags avec Terraform for_each"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-2"
* "Terraform"
* "Azure Policy"
* "for_each"
* "Governance"
* "FinOps"
* "Azure"

---

# SERIES-2.3 - Industrialiser les Politiques de Tags avec Terraform for_each

## Introduction

Dans l'article précédent, nous avons déployé notre première Azure Policy afin de contrôler la présence du tag :

```text
Project
```

Cette approche nous a permis de valider le fonctionnement complet de la chaîne de gouvernance :

* Terraform
* Azure Policy
* Management Groups
* Héritage Azure

Cependant, notre stratégie MonLabAzure définit huit tags obligatoires.

Créer huit ressources Terraform quasiment identiques ne serait ni maintenable ni évolutif.

Dans cet article, nous allons industrialiser notre code Terraform grâce à `for_each`.

---

# Pourquoi Utiliser for_each ?

Imaginez la solution suivante :

```hcl
project_tag
environment_tag
owner_tag
costcenter_tag
criticality_tag
managedby_tag
dataclassification_tag
backuprequired_tag
```

Cela fonctionne.

Mais chaque nouvelle Policy impose :

* dupliquer du code ;
* maintenir plusieurs ressources ;
* augmenter le risque d'erreur.

Terraform propose une meilleure approche :

```hcl
for_each
```

---

# Objectif

Nous voulons gérer automatiquement :

| Tag                |
| ------------------ |
| Project            |
| Environment        |
| Owner              |
| CostCenter         |
| Criticality        |
| ManagedBy          |
| DataClassification |
| BackupRequired     |

avec une seule ressource Terraform.

---

# Définition des Tags

## locals.tf

```hcl
locals {

  # Liste des tags obligatoires définis dans ADR-005.
  mandatory_tags = [
    "Project",
    "Environment",
    "Owner",
    "CostCenter",
    "Criticality",
    "ManagedBy",
    "DataClassification",
    "BackupRequired"
  ]
}
```

---

# Récupération de la Policy Microsoft

```hcl
# Policy intégrée Microsoft utilisée pour vérifier la présence d'un tag.
data "azurerm_policy_definition" "require_tag" {
  display_name = "Require a tag on resources"
}
```

---

# Affectation Dynamique

## main.tf

```hcl
# Création automatique d'une affectation Azure Policy
# pour chaque tag présent dans la liste mandatory_tags.
resource "azurerm_management_group_policy_assignment" "mandatory_tags" {

  for_each = toset(local.mandatory_tags)

  name = "audit-${lower(each.value)}-tag"

  management_group_id =
    "/providers/Microsoft.Management/managementGroups/mla-landingzones"

  policy_definition_id =
    data.azurerm_policy_definition.require_tag.id

  parameters = jsonencode({
    tagName = {
      value = each.value
    }
  })
}
```

---

# Ce que Terraform Va Créer

Terraform générera automatiquement :

```text
audit-project-tag
audit-environment-tag
audit-owner-tag
audit-costcenter-tag
audit-criticality-tag
audit-managedby-tag
audit-dataclassification-tag
audit-backuprequired-tag
```

sans duplication de code.

---

# Vérification

```bash
terraform plan
```

Résultat attendu :

```text
8 resources to add
```

---

# Déploiement

```bash
terraform apply
```

---

# Vérification dans Azure

Azure Policy :

```text
Assignments
│
├── audit-project-tag
├── audit-environment-tag
├── audit-owner-tag
├── audit-costcenter-tag
├── audit-criticality-tag
├── audit-managedby-tag
├── audit-dataclassification-tag
└── audit-backuprequired-tag
```

---

# Pourquoi cette Approche est Meilleure ?

Ajout d'un nouveau tag :

Avant :

```hcl
Nouvelle ressource Terraform
```

Après :

```hcl
mandatory_tags = [
  ...
  "BusinessUnit"
]
```

Terraform s'occupe du reste.

---

# Livrables

À l'issue de cet article :

✅ 8 tags contrôlés

✅ Premier usage de `locals`

✅ Premier usage de `for_each`

✅ Code Terraform industrialisé

✅ Gouvernance FinOps renforcée

---

# Conclusion

Nous avons transformé notre preuve de concept en une implémentation Terraform réutilisable et évolutive.

Cette approche servira de modèle pour les futures Azure Policies de la plateforme.

Dans le prochain article, nous limiterons les déploiements aux régions autorisées afin de renforcer la gouvernance géographique de notre Landing Zone Azure.
