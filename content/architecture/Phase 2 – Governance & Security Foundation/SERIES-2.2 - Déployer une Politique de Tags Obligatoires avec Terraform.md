---

title: "SERIES-2.2 - Déployer une Politique de Tags Obligatoires avec Terraform"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-2"
* "Governance"
* "Azure Policy"
* "Tags"
* "Terraform"
* "FinOps"
* "Compliance"
* "Azure"

---

# SERIES-2.2 – Déployer une Politique de Tags Obligatoires avec Terraform

## Introduction

Dans l'article précédent, nous avons présenté Azure Policy et son rôle dans la gouvernance Azure.

Conformément à ADR-005 et ADR-006, notre première règle de conformité concerne les tags.

Les tags constituent l'un des piliers d'une plateforme Azure moderne.

Ils permettent notamment :

* l'identification des ressources ;
* la répartition des coûts ;
* l'automatisation ;
* les rapports FinOps ;
* la classification des données ;
* les contrôles de conformité.

Dans cet article, nous allons déployer notre première Azure Policy avec Terraform afin d'auditer la présence des tags obligatoires.

---

# Positionnement dans le Projet

```text
Phase 2 – Governance & Security Foundation

ADR-006   Stratégie Azure Policy et Gouvernance      ✅

SERIES-2.1   Introduction Azure Policy              ✅
SERIES-2.2   Politique de Tags Obligatoires         ⏳
SERIES-2.3   Politique de Régions Autorisées
SERIES-2.4   Diagnostic Settings
SERIES-2.5   Contrôle des Ressources Publiques
SERIES-2.6   Initiative MonLabAzure Governance
SERIES-2.7   Validation Governance Foundation
```

---

# Objectif

À la fin de cet article :

* Une Azure Policy sera déployée.
* Les ressources sans tags seront identifiées.
* Les écarts de conformité seront visibles.
* La gouvernance FinOps commencera à être appliquée.

Nous utiliserons volontairement le mode :

```text
Audit
```

afin d'éviter tout blocage durant les premières phases du projet.

---

# Pourquoi les Tags sont Importants ?

Sans tags :

```text
VM01
Storage01
SQL01
```

Il devient difficile de répondre aux questions suivantes :

* Quel projet utilise cette ressource ?
* Qui en est responsable ?
* Quel environnement est concerné ?
* Quelle est sa criticité ?
* Faut-il la sauvegarder ?
* À quel centre de coût appartient-elle ?

---

# Stratégie de Tags MonLabAzure

Conformément à ADR-005 :

Les tags obligatoires sont :

| Tag                | Description                 |
| ------------------ | --------------------------- |
| Project            | Nom du projet               |
| Environment        | DEV, QUALIF ou PROD         |
| Owner              | Responsable de la ressource |
| CostCenter         | Centre de coût              |
| Criticality        | Niveau de criticité         |
| ManagedBy          | Équipe responsable          |
| DataClassification | Classification des données  |
| BackupRequired     | Sauvegarde requise          |

---

# Architecture de Déploiement

La Policy sera affectée au niveau :

```text
mla-landingzones
```

Ce qui permettra l'héritage automatique vers :

```text
mla-dev
mla-qualif
mla-prod
```

---

# Arborescence Terraform

Créer un nouveau répertoire :

```text
terraform/
└── 03-tag-policies
```

Structure :

```text
03-tag-policies
│
├── backend.tf
├── versions.tf
├── providers.tf
├── variables.tf
├── main.tf
├── outputs.tf
└── terraform.tfvars
```

---

# Backend Terraform

## backend.tf

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "tag-policies.tfstate"
  }
}
```

---

# Utilisation d'une Policy Intégrée Azure

Microsoft fournit déjà une Policy intégrée permettant de vérifier la présence d'un tag.

Nous allons la réutiliser.

---

# Déclaration de la Policy

## main.tf

### Policy Project

```hcl
data "azurerm_policy_definition" "require_project_tag" {
  display_name = "Require a tag on resources"
}
```

---

# Affectation de la Policy

```hcl
resource "azurerm_management_group_policy_assignment" "project_tag" {

  name                 = "audit-project-tag"
  management_group_id  = "/providers/Microsoft.Management/managementGroups/mla-landingzones"

  policy_definition_id = data.azurerm_policy_definition.require_project_tag.id

  parameters = jsonencode({
    tagName = {
      value = "Project"
    }
  })
}
```

---

# Pourquoi Commencer par un Seul Tag ?

Une erreur fréquente consiste à déployer immédiatement :

* 8 tags ;
* plusieurs initiatives ;
* plusieurs modes Deny.

Nous allons procéder progressivement.

Notre premier objectif est :

```text
Valider le fonctionnement complet
de la chaîne Azure Policy.
```

Une fois cette étape validée, nous étendrons la gouvernance.

---

# Initialisation

```bash
terraform init
```

---

# Vérification

```bash
terraform validate
```

---

# Plan Terraform

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
     └── audit-project-tag
```

---

# Vérification de la Conformité

Créer volontairement une ressource :

```text
Sans le tag Project
```

Puis consulter :

```text
Azure Policy
│
└── Compliance
```

Résultat attendu :

```text
Non Compliant
```

---

# Pourquoi Utiliser Audit ?

Nous ne voulons pas encore bloquer les déploiements.

Mode Audit :

```text
Création autorisée
+
Écart signalé
```

Mode Deny :

```text
Création refusée
```

Dans un projet réel, il est généralement recommandé de commencer par Audit.

---

# Dépannage

## Policy Non Visible

Attendre quelques minutes.

La propagation Azure Policy peut prendre du temps.

---

## Terraform Plan Vide

Vérifier :

```bash
terraform state list
```

---

## Mauvais Management Group

Vérifier :

```text
mla-landingzones
```

---

# Coûts

Cette Azure Policy n'entraîne aucun coût direct.

Coût estimé :

```text
0 €
```

---

# Limites de la Solution Actuelle

À ce stade :

* Un seul tag est contrôlé.
* Les autres tags ne sont pas encore vérifiés.
* Aucune initiative n'est utilisée.

Cette limitation est volontaire.

---

# Livrables

À l'issue de cet article :

✅ Première Azure Policy déployée

✅ Terraform utilisé pour la gouvernance

✅ Contrôle du tag Project

✅ Héritage via Management Groups

✅ Première conformité Azure opérationnelle

---

# Conclusion

Nous avons déployé notre première Azure Policy et commencé à appliquer concrètement la gouvernance définie dans ADR-005 et ADR-006.

Cette étape valide l'ensemble de la chaîne :

* Terraform
* Azure Policy
* Management Groups
* Héritage de gouvernance

Dans le prochain article, nous étendrons cette approche afin de contrôler l'ensemble des tags obligatoires définis pour la plateforme MonLabAzure Cloud Platform.
