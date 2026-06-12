---

title: "SERIES-2.5 - Déployer une Politique de Diagnostic Settings avec Terraform"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-2"
* "Azure Policy"
* "Terraform"
* "Azure Monitor"
* "Log Analytics"
* "Observability"
* "Governance"
* "Compliance"
* "Azure"

---

# SERIES-2.5 – Déployer une Politique de Diagnostic Settings avec Terraform

## Introduction

Dans les articles précédents, nous avons commencé à mettre en œuvre les premiers contrôles de gouvernance :

* contrôle des tags obligatoires ;
* contrôle des régions autorisées.

Ces contrôles permettent d'améliorer la conformité de la plateforme.

Cependant, une question fondamentale reste ouverte :

> Comment superviser efficacement les ressources Azure si aucun journal n'est collecté ?

Une plateforme qui ne génère ni journaux ni métriques est extrêmement difficile à exploiter, sécuriser ou auditer.

Dans cet article, nous allons préparer la gouvernance de l'observabilité en imposant la présence des Diagnostic Settings.

---

# Positionnement dans le Projet

```text
Phase 2 – Governance & Security Foundation

ADR-006   Stratégie Azure Policy et Gouvernance      ✅

SERIES-2.1   Introduction Azure Policy              ✅
SERIES-2.2   Premier Tag Obligatoire                ✅
SERIES-2.3   Industrialiser les Tags                ✅
SERIES-2.4   Régions Autorisées                     ✅
SERIES-2.5   Diagnostic Settings                    ⏳
SERIES-2.6   Contrôle des Ressources Publiques
SERIES-2.7   Initiative MonLabAzure Governance
SERIES-2.8   Validation Governance Foundation
```

---

# Pourquoi les Diagnostic Settings sont Importants ?

Par défaut, de nombreuses ressources Azure produisent :

* des journaux d'activité ;
* des journaux de sécurité ;
* des métriques de performance.

Cependant, ces informations ne sont pas conservées automatiquement dans un espace centralisé.

Sans Diagnostic Settings :

```text
Ressource Azure
        │
        ▼
Logs générés
        │
        ▼
Non exploités
```

Conséquences :

* difficultés d'investigation ;
* perte d'informations ;
* audits compliqués ;
* supervision limitée.

---

# Objectif de la Politique

Nous souhaitons vérifier que les ressources Azure importantes disposent de Diagnostic Settings.

Dans un premier temps :

```text
Audit
```

Dans une phase ultérieure :

```text
DeployIfNotExists
```

---

# Pourquoi Ne Pas Utiliser Directement DeployIfNotExists ?

Parce que notre plateforme ne dispose pas encore :

* d'un Workspace Log Analytics central ;
* d'une architecture Azure Monitor ;
* d'une stratégie de rétention définie.

Ces éléments seront construits dans :

```text
Phase 6 – Monitoring & Observability
```

Nous allons donc commencer par identifier les écarts.

---

# Architecture Future

L'objectif final sera :

```text
Ressource Azure
        │
        ▼
Diagnostic Settings
        │
        ▼
Log Analytics Workspace
        │
        ▼
Azure Monitor
```

Mais aujourd'hui :

```text
Audit uniquement
```

---

# Architecture de Gouvernance

La Policy sera affectée au niveau :

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
└── 05-diagnostic-policies
```

Structure :

```text
05-diagnostic-policies
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
# Backend Terraform dédié aux Policies de supervision.
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "diagnostic-policies.tfstate"
  }
}
```

---

# Utilisation d'une Policy Intégrée Azure

Microsoft fournit plusieurs Policies liées aux Diagnostic Settings.

Pour notre laboratoire, nous utiliserons une Policy d'audit.

---

# Déclaration de la Policy

## main.tf

```hcl
# Policy Microsoft permettant d'identifier les ressources
# ne disposant pas de Diagnostic Settings.
data "azurerm_policy_definition" "diagnostic_settings_audit" {
  display_name = "Audit diagnostic setting for selected resource types"
}
```

---

# Affectation de la Policy

```hcl
# Affectation de la Policy au niveau Landing Zones.
resource "azurerm_management_group_policy_assignment" "diagnostic_settings" {

  name = "audit-diagnostic-settings"

  management_group_id =
    "/providers/Microsoft.Management/managementGroups/mla-landingzones"

  policy_definition_id =
    data.azurerm_policy_definition.diagnostic_settings_audit.id
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
     └── audit-diagnostic-settings
```

---

# Vérification de la Conformité

Après quelques minutes :

```text
Azure Policy
│
└── Compliance
```

Les ressources dépourvues de Diagnostic Settings apparaîtront comme :

```text
Non Compliant
```

---

# Ce que Nous Préparons Réellement

Cette Policy prépare les futures phases :

```text
Phase 6 – Monitoring & Observability
```

où nous déploierons :

* Azure Monitor ;
* Log Analytics ;
* Alertes ;
* Dashboards ;
* Workbooks.

Nous construisons aujourd'hui la gouvernance qui permettra demain d'imposer ces standards.

---

# Dépannage

## Policy Introuvable

Certaines Policies intégrées Microsoft peuvent évoluer.

Lister les Policies disponibles :

```bash
az policy definition list --query "[].displayName"
```

Puis adapter le nom utilisé dans Terraform.

---

## Compliance Vide

Attendre la première évaluation Azure Policy.

Le processus peut prendre plusieurs minutes.

---

## Terraform Sans Modification

Vérifier :

```bash
terraform state list
```

---

# Bonnes Pratiques Enterprise

Dans une entreprise mature :

```text
Audit
      │
      ▼
Correction
      │
      ▼
DeployIfNotExists
```

Cette approche réduit considérablement les risques opérationnels.

---

# Coûts

Cette Policy ne génère aucun coût direct.

Les coûts apparaîtront uniquement lors de la création :

* du Workspace Log Analytics ;
* des Diagnostic Settings ;
* de la rétention des logs.

Ces sujets seront traités dans la Phase 6.

---

# Livrables

À l'issue de cet article :

✅ Première Policy d'observabilité

✅ Contrôle des Diagnostic Settings

✅ Héritage via Management Groups

✅ Préparation de la supervision centralisée

✅ Gouvernance renforcée

---

# Conclusion

Nous avons ajouté un nouveau contrôle de conformité permettant d'identifier les ressources qui ne disposent pas de Diagnostic Settings.

Cette étape prépare directement la future plateforme d'observabilité qui sera construite dans les phases suivantes.

Dans le prochain article, nous mettrons en place un contrôle des ressources exposées publiquement afin de renforcer la sécurité globale de notre Landing Zone Azure.

---

# Revue d'architecte

>>Petit point important- : dans un environnement Enterprise réel, je regrouperais souvent les Policies Tags, Régions et Diagnostic Settings dans une même Initiative très tôt. Mais pédagogiquement, j'ai préféré les déployer séparément afin que le lecteur comprenne clairement le rôle de chaque Policy avant de construire l'initiative globale dans SERIES-2.7
