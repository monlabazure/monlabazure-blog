---

title: "SERIES-3.6 - Affecter les Rôles RBAC aux Management Groups"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "RBAC"
* "Microsoft Entra ID"
* "Management Groups"
* "Azure CLI"
* "Terraform"
* "Least Privilege"
* "Azure"

---

# SERIES-3.6 – Affecter les Rôles RBAC aux Management Groups

## Introduction

Dans l'article précédent, nous avons défini le modèle RBAC cible de MonLabAzure Cloud Platform.

Nous avons répondu aux questions suivantes :

```text
Quel groupe doit recevoir quel rôle ?
Sur quel périmètre ?
Avec quelle justification ?
```

Nous allons maintenant passer à l'implémentation.

Dans cet article, nous allons affecter des rôles Azure RBAC aux groupes Microsoft Entra ID créés précédemment.

L'objectif est de commencer à appliquer concrètement le modèle IAM défini dans ADR-007.

Le modèle reste le suivant :

```text
Utilisateur
    │
    ▼
Groupe Microsoft Entra ID
    │
    ▼
Rôle Azure RBAC
    │
    ▼
Scope Azure
```

Nous allons volontairement travailler au niveau des **Management Groups**, car ils structurent déjà notre Landing Zone depuis la Phase 1.

---

# Positionnement dans le Projet

```text
Phase 3 – Identity & Access Management

ADR-004 – Convention de Nommage Azure ✅
ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride ✅

SERIES-3.1 – Introduction à Identity & Access Management dans Azure ✅
SERIES-3.2 – Comprendre les licences Microsoft Entra ID ✅
SERIES-3.3 – Identité hybride : Entra Connect Sync, Cloud Sync et modes d'authentification ✅
SERIES-3.4 – Créer les groupes Microsoft Entra ID ✅
SERIES-3.5 – Modèle RBAC Azure et séparation des rôles ✅
SERIES-3.6 – Affecter les rôles RBAC aux Management Groups ⏳
SERIES-3.7 – Comptes administratifs et comptes Break Glass
SERIES-3.8 – MFA, authentification forte et méthodes phishing-resistant
SERIES-3.9 – Conditional Access Foundation
SERIES-3.10 – PIM, Access Reviews et accès temporaires
SERIES-3.11 – Comptes invités et gouvernance B2B
SERIES-3.12 – Service Principals, App Registrations et Managed Identities
SERIES-3.13 – Export des logs Entra ID vers Log Analytics
SERIES-3.14 – Validation IAM Foundation
```

---

# Références d'Architecture

Cet article s'appuie sur les décisions déjà validées.

## ADR-004 – Convention de Nommage Azure

Les groupes Microsoft Entra ID utilisés dans cet article respectent la convention :

```text
AZR-MLA-<ROLE-FONCTIONNEL>
```

Exemples :

```text
AZR-MLA-NETWORK-ADMINS
AZR-MLA-OPERATIONS
AZR-MLA-READERS
```

## ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride

ADR-007 impose les règles suivantes :

* ne pas affecter les rôles directement aux utilisateurs ;
* utiliser des groupes Microsoft Entra ID ;
* appliquer le moindre privilège ;
* limiter les rôles élevés ;
* séparer les responsabilités ;
* éviter les droits permanents trop larges en production ;
* conserver les comptes Break Glass hors administration quotidienne.

Cet article applique ces décisions au niveau des Management Groups.

---

# Objectif de l'Article

À la fin de cet article, nous aurons :

* récupéré les Object IDs des groupes Microsoft Entra ID ;
* construit les scopes RBAC des Management Groups ;
* affecté des rôles Azure RBAC aux groupes ;
* vérifié les affectations ;
* identifié les points de vigilance ;
* préparé l'industrialisation Terraform.

---

# Rappel du Modèle RBAC Cible

Nous allons partir du modèle défini dans SERIES-3.5.

| Groupe                        | Rôle cible             | Scope cible                     | Objectif                       |
| ----------------------------- | ---------------------- | ------------------------------- | ------------------------------ |
| AZR-MLA-PLATFORM-ADMINS       | Contributor            | mla-platform                    | Administration plateforme      |
| AZR-MLA-NETWORK-ADMINS        | Network Contributor    | mla-connectivity                | Administration réseau          |
| AZR-MLA-SECURITY-ADMINS       | Security Reader        | mla-landingzones                | Lecture sécurité des workloads |
| AZR-MLA-OPERATIONS            | Monitoring Contributor | mla-management                  | Supervision et exploitation    |
| AZR-MLA-DEVELOPERS            | Contributor            | mla-dev                         | Développement                  |
| AZR-MLA-ARCHITECTS            | Reader                 | mla-platform / mla-landingzones | Lecture architecture           |
| AZR-MLA-READERS               | Reader                 | mla-landingzones                | Lecture générale               |
| AZR-MLA-BREAKGLASS-MONITORING | Monitoring Reader      | mla-management                  | Surveillance Break Glass       |

Ce modèle reste volontairement prudent.

Nous n'affectons pas `Owner` dans cet article.

Nous n'affectons pas `Contributor` à la production.

Nous ne donnons pas de droits d'administration aux comptes Break Glass via les groupes standards.

---

# Hiérarchie des Management Groups

Rappel de la hiérarchie créée en Phase 1 :

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

Les affectations RBAC seront appliquées sur certains de ces Management Groups.

---

# Prérequis

Pour suivre cet article, il faut disposer :

* des groupes Microsoft Entra ID créés dans SERIES-3.4 ;
* des Management Groups créés dans la Phase 1 ;
* d'Azure CLI installé ;
* d'un compte disposant des droits nécessaires pour affecter des rôles RBAC ;
* du bon tenant Azure sélectionné ;
* d'une bonne compréhension du modèle RBAC défini dans SERIES-3.5.

Vérifier la connexion :

```bash
az account show
```

Vérifier le tenant courant :

```bash
az account show \
  --query tenantId \
  --output tsv
```

Vérifier les Management Groups :

```bash
az account management-group list \
  --query "[].{Name:name, DisplayName:displayName}" \
  --output table
```

---

# Point Important sur les Permissions

Pour créer une affectation RBAC, le compte utilisé doit disposer d'un rôle permettant de gérer les accès.

Exemples :

```text
Owner
User Access Administrator
Role Based Access Control Administrator
```

Dans un environnement Enterprise, ces permissions doivent être très limitées.

Dans MonLabAzure, elles peuvent être utilisées temporairement pour construire le Lab, mais elles ne doivent pas devenir des droits permanents banalisés.

---

# Convention des Scopes Management Groups

Le scope RBAC d'un Management Group suit le format suivant :

```text
/providers/Microsoft.Management/managementGroups/<management-group-id>
```

Exemples :

```text
/providers/Microsoft.Management/managementGroups/mla-platform
/providers/Microsoft.Management/managementGroups/mla-connectivity
/providers/Microsoft.Management/managementGroups/mla-management
/providers/Microsoft.Management/managementGroups/mla-landingzones
/providers/Microsoft.Management/managementGroups/mla-dev
/providers/Microsoft.Management/managementGroups/mla-qualif
/providers/Microsoft.Management/managementGroups/mla-prod
```

---

# Récupérer les Object IDs des Groupes

Avant de créer les affectations RBAC, nous devons récupérer l'Object ID de chaque groupe Microsoft Entra ID.

## Platform Admins

```bash
AZR_MLA_PLATFORM_ADMINS_ID=$(az ad group show \
  --group "AZR-MLA-PLATFORM-ADMINS" \
  --query id \
  --output tsv)
```

## Network Admins

```bash
AZR_MLA_NETWORK_ADMINS_ID=$(az ad group show \
  --group "AZR-MLA-NETWORK-ADMINS" \
  --query id \
  --output tsv)
```

## Security Admins

```bash
AZR_MLA_SECURITY_ADMINS_ID=$(az ad group show \
  --group "AZR-MLA-SECURITY-ADMINS" \
  --query id \
  --output tsv)
```

## Operations

```bash
AZR_MLA_OPERATIONS_ID=$(az ad group show \
  --group "AZR-MLA-OPERATIONS" \
  --query id \
  --output tsv)
```

## Developers

```bash
AZR_MLA_DEVELOPERS_ID=$(az ad group show \
  --group "AZR-MLA-DEVELOPERS" \
  --query id \
  --output tsv)
```

## Architects

```bash
AZR_MLA_ARCHITECTS_ID=$(az ad group show \
  --group "AZR-MLA-ARCHITECTS" \
  --query id \
  --output tsv)
```

## Readers

```bash
AZR_MLA_READERS_ID=$(az ad group show \
  --group "AZR-MLA-READERS" \
  --query id \
  --output tsv)
```

## Break Glass Monitoring

```bash
AZR_MLA_BREAKGLASS_MONITORING_ID=$(az ad group show \
  --group "AZR-MLA-BREAKGLASS-MONITORING" \
  --query id \
  --output tsv)
```

---

# Vérifier les Object IDs

```bash
echo "Platform Admins: $AZR_MLA_PLATFORM_ADMINS_ID"
echo "Network Admins: $AZR_MLA_NETWORK_ADMINS_ID"
echo "Security Admins: $AZR_MLA_SECURITY_ADMINS_ID"
echo "Operations: $AZR_MLA_OPERATIONS_ID"
echo "Developers: $AZR_MLA_DEVELOPERS_ID"
echo "Architects: $AZR_MLA_ARCHITECTS_ID"
echo "Readers: $AZR_MLA_READERS_ID"
echo "Break Glass Monitoring: $AZR_MLA_BREAKGLASS_MONITORING_ID"
```

Chaque variable doit retourner un GUID.

Exemple :

```text
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

---

# Définir les Scopes

Pour simplifier les commandes, nous allons définir les scopes dans des variables.

```bash
SCOPE_PLATFORM="/providers/Microsoft.Management/managementGroups/mla-platform"
SCOPE_CONNECTIVITY="/providers/Microsoft.Management/managementGroups/mla-connectivity"
SCOPE_MANAGEMENT="/providers/Microsoft.Management/managementGroups/mla-management"
SCOPE_LANDINGZONES="/providers/Microsoft.Management/managementGroups/mla-landingzones"
SCOPE_DEV="/providers/Microsoft.Management/managementGroups/mla-dev"
SCOPE_QUALIF="/providers/Microsoft.Management/managementGroups/mla-qualif"
SCOPE_PROD="/providers/Microsoft.Management/managementGroups/mla-prod"
```

Vérifier :

```bash
echo $SCOPE_PLATFORM
echo $SCOPE_CONNECTIVITY
echo $SCOPE_MANAGEMENT
echo $SCOPE_LANDINGZONES
echo $SCOPE_DEV
```

---

# Affectation RBAC – Platform Admins

Le groupe `AZR-MLA-PLATFORM-ADMINS` reçoit le rôle `Contributor` sur `mla-platform`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_PLATFORM_ADMINS_ID" \
  --assignee-principal-type Group \
  --role "Contributor" \
  --scope "$SCOPE_PLATFORM"
```

Justification :

```text
La Platform Team administre les composants de plateforme.
```

Point de vigilance :

```text
Contributor reste un rôle large.
Il doit être attribué uniquement à un groupe contrôlé.
```

---

# Affectation RBAC – Network Admins

Le groupe `AZR-MLA-NETWORK-ADMINS` reçoit le rôle `Network Contributor` sur `mla-connectivity`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_NETWORK_ADMINS_ID" \
  --assignee-principal-type Group \
  --role "Network Contributor" \
  --scope "$SCOPE_CONNECTIVITY"
```

Justification :

```text
L'équipe réseau administre les ressources réseau centrales.
```

Cette affectation respecte le moindre privilège mieux qu'un rôle `Contributor` global.

---

# Affectation RBAC – Security Admins

Le groupe `AZR-MLA-SECURITY-ADMINS` reçoit le rôle `Security Reader` sur `mla-landingzones`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_SECURITY_ADMINS_ID" \
  --assignee-principal-type Group \
  --role "Security Reader" \
  --scope "$SCOPE_LANDINGZONES"
```

Justification :

```text
L'équipe sécurité doit pouvoir lire les informations de sécurité des environnements DEV, QUALIF et PROD.
```

Nous commençons par un rôle de lecture.

Les droits de modification sécurité seront ajoutés plus tard si nécessaire.

---

# Affectation RBAC – Operations

Le groupe `AZR-MLA-OPERATIONS` reçoit le rôle `Monitoring Contributor` sur `mla-management`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_OPERATIONS_ID" \
  --assignee-principal-type Group \
  --role "Monitoring Contributor" \
  --scope "$SCOPE_MANAGEMENT"
```

Justification :

```text
L'équipe Operations gère les composants de supervision centralisée.
```

Dans les phases Monitoring, ce rôle pourra être complété par des rôles plus spécifiques sur Log Analytics.

---

# Affectation RBAC – Developers

Le groupe `AZR-MLA-DEVELOPERS` reçoit le rôle `Contributor` sur `mla-dev`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_DEVELOPERS_ID" \
  --assignee-principal-type Group \
  --role "Contributor" \
  --scope "$SCOPE_DEV"
```

Justification :

```text
Les développeurs doivent pouvoir travailler librement dans DEV.
```

Point important :

```text
Aucun Contributor permanent n'est accordé sur PROD.
```

---

# Affectation RBAC – Architects

Le groupe `AZR-MLA-ARCHITECTS` reçoit le rôle `Reader` sur `mla-platform`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_ARCHITECTS_ID" \
  --assignee-principal-type Group \
  --role "Reader" \
  --scope "$SCOPE_PLATFORM"
```

Il reçoit également le rôle `Reader` sur `mla-landingzones`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_ARCHITECTS_ID" \
  --assignee-principal-type Group \
  --role "Reader" \
  --scope "$SCOPE_LANDINGZONES"
```

Justification :

```text
L'équipe Architecture doit pouvoir consulter la plateforme et les environnements pour les revues d'architecture.
```

---

# Affectation RBAC – Readers

Le groupe `AZR-MLA-READERS` reçoit le rôle `Reader` sur `mla-landingzones`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_READERS_ID" \
  --assignee-principal-type Group \
  --role "Reader" \
  --scope "$SCOPE_LANDINGZONES"
```

Justification :

```text
Ce groupe fournit une lecture générale contrôlée sur les environnements applicatifs.
```

---

# Affectation RBAC – Break Glass Monitoring

Le groupe `AZR-MLA-BREAKGLASS-MONITORING` reçoit le rôle `Monitoring Reader` sur `mla-management`.

```bash
az role assignment create \
  --assignee-object-id "$AZR_MLA_BREAKGLASS_MONITORING_ID" \
  --assignee-principal-type Group \
  --role "Monitoring Reader" \
  --scope "$SCOPE_MANAGEMENT"
```

Justification :

```text
Ce groupe sert à consulter les éléments de supervision liés aux comptes Break Glass.
```

Point important :

```text
Ce groupe ne donne pas de droits d'administration aux comptes Break Glass.
```

---

# Vérifier les Affectations RBAC

## Vérifier sur mla-platform

```bash
az role assignment list \
  --scope "$SCOPE_PLATFORM" \
  --query "[].{PrincipalName:principalName, Role:roleDefinitionName, Scope:scope}" \
  --output table
```

## Vérifier sur mla-connectivity

```bash
az role assignment list \
  --scope "$SCOPE_CONNECTIVITY" \
  --query "[].{PrincipalName:principalName, Role:roleDefinitionName, Scope:scope}" \
  --output table
```

## Vérifier sur mla-management

```bash
az role assignment list \
  --scope "$SCOPE_MANAGEMENT" \
  --query "[].{PrincipalName:principalName, Role:roleDefinitionName, Scope:scope}" \
  --output table
```

## Vérifier sur mla-landingzones

```bash
az role assignment list \
  --scope "$SCOPE_LANDINGZONES" \
  --query "[].{PrincipalName:principalName, Role:roleDefinitionName, Scope:scope}" \
  --output table
```

## Vérifier sur mla-dev

```bash
az role assignment list \
  --scope "$SCOPE_DEV" \
  --query "[].{PrincipalName:principalName, Role:roleDefinitionName, Scope:scope}" \
  --output table
```

---

# Résultat Attendu

Les affectations doivent correspondre à la matrice suivante :

| Groupe                        | Rôle                   | Scope            |
| ----------------------------- | ---------------------- | ---------------- |
| AZR-MLA-PLATFORM-ADMINS       | Contributor            | mla-platform     |
| AZR-MLA-NETWORK-ADMINS        | Network Contributor    | mla-connectivity |
| AZR-MLA-SECURITY-ADMINS       | Security Reader        | mla-landingzones |
| AZR-MLA-OPERATIONS            | Monitoring Contributor | mla-management   |
| AZR-MLA-DEVELOPERS            | Contributor            | mla-dev          |
| AZR-MLA-ARCHITECTS            | Reader                 | mla-platform     |
| AZR-MLA-ARCHITECTS            | Reader                 | mla-landingzones |
| AZR-MLA-READERS               | Reader                 | mla-landingzones |
| AZR-MLA-BREAKGLASS-MONITORING | Monitoring Reader      | mla-management   |

---

# Vérification dans le Portail Azure

Il est aussi possible de vérifier les affectations depuis le portail.

Chemin :

```text
Management Groups
        │
        ▼
Sélectionner le Management Group
        │
        ▼
Access control (IAM)
        │
        ▼
Role assignments
```

Vérifier que les groupes apparaissent avec les bons rôles.

---

# Tester l'Héritage

Une affectation au niveau Management Group est héritée par les scopes inférieurs.

Exemple :

```text
AZR-MLA-DEVELOPERS
        │
        ▼
Contributor
        │
        ▼
mla-dev
        │
        ▼
Subscription DEV
```

Les abonnements rattachés à `mla-dev` héritent donc de l'affectation.

Pour vérifier, il est possible de consulter l'IAM au niveau de la subscription DEV.

---

# Points de Vigilance

## Propagation RBAC

Les affectations RBAC peuvent nécessiter quelques minutes avant d'être visibles ou pleinement effectives.

Il ne faut pas conclure trop vite à une erreur si le portail ou les commandes ne montrent pas immédiatement le résultat.

---

## Mauvais Tenant

Une erreur fréquente consiste à créer les groupes dans un tenant, puis à affecter les rôles dans un autre.

Toujours vérifier :

```bash
az account show \
  --query tenantId \
  --output tsv
```

---

## Mauvais Scope

Un scope incorrect peut provoquer une erreur ou une affectation au mauvais périmètre.

Vérifier systématiquement les variables :

```bash
echo $SCOPE_PLATFORM
echo $SCOPE_CONNECTIVITY
echo $SCOPE_MANAGEMENT
echo $SCOPE_LANDINGZONES
echo $SCOPE_DEV
```

---

## Rôle Trop Large

Avant chaque affectation, poser la question :

```text
Le rôle est-il nécessaire ?
Le scope est-il trop large ?
Existe-t-il un rôle plus précis ?
```

---

## Production

Aucune affectation `Contributor` n'est réalisée sur `mla-prod` dans cet article.

Ce choix est volontaire.

La production doit rester plus contrôlée que DEV et QUALIF.

---

# Option Terraform

Les affectations RBAC peuvent être gérées avec Terraform via le provider `azurerm`.

Cette approche sera utile pour industrialiser le modèle.

Exemple simplifié :

```hcl
# Récupération des groupes Microsoft Entra ID existants.
# Les groupes ont été créés dans la série précédente.
data "azuread_group" "platform_admins" {
  display_name = "AZR-MLA-PLATFORM-ADMINS"
}

data "azuread_group" "network_admins" {
  display_name = "AZR-MLA-NETWORK-ADMINS"
}

data "azuread_group" "security_admins" {
  display_name = "AZR-MLA-SECURITY-ADMINS"
}

data "azuread_group" "operations" {
  display_name = "AZR-MLA-OPERATIONS"
}

data "azuread_group" "developers" {
  display_name = "AZR-MLA-DEVELOPERS"
}

data "azuread_group" "architects" {
  display_name = "AZR-MLA-ARCHITECTS"
}

data "azuread_group" "readers" {
  display_name = "AZR-MLA-READERS"
}

data "azuread_group" "breakglass_monitoring" {
  display_name = "AZR-MLA-BREAKGLASS-MONITORING"
}
```

---

# Variables de Scopes Terraform

```hcl
# Définition locale des scopes Management Groups.
# Ces scopes correspondent à la hiérarchie créée en Phase 1.
locals {
  scope_platform     = "/providers/Microsoft.Management/managementGroups/mla-platform"
  scope_connectivity = "/providers/Microsoft.Management/managementGroups/mla-connectivity"
  scope_management   = "/providers/Microsoft.Management/managementGroups/mla-management"
  scope_landingzones = "/providers/Microsoft.Management/managementGroups/mla-landingzones"
  scope_dev          = "/providers/Microsoft.Management/managementGroups/mla-dev"
}
```

---

# Affectations RBAC Terraform

```hcl
# Affectation du rôle Contributor aux administrateurs plateforme.
# Cette affectation s'applique au Management Group mla-platform.
resource "azurerm_role_assignment" "platform_admins_contributor" {
  scope                = local.scope_platform
  role_definition_name = "Contributor"
  principal_id         = data.azuread_group.platform_admins.object_id
}

# Affectation du rôle Network Contributor aux administrateurs réseau.
# Cette affectation s'applique au Management Group mla-connectivity.
resource "azurerm_role_assignment" "network_admins_network_contributor" {
  scope                = local.scope_connectivity
  role_definition_name = "Network Contributor"
  principal_id         = data.azuread_group.network_admins.object_id
}

# Affectation du rôle Security Reader à l'équipe sécurité.
# Cette affectation s'applique aux Landing Zones DEV, QUALIF et PROD.
resource "azurerm_role_assignment" "security_admins_security_reader" {
  scope                = local.scope_landingzones
  role_definition_name = "Security Reader"
  principal_id         = data.azuread_group.security_admins.object_id
}

# Affectation du rôle Monitoring Contributor à l'équipe Operations.
# Cette affectation s'applique au Management Group mla-management.
resource "azurerm_role_assignment" "operations_monitoring_contributor" {
  scope                = local.scope_management
  role_definition_name = "Monitoring Contributor"
  principal_id         = data.azuread_group.operations.object_id
}

# Affectation du rôle Contributor aux développeurs sur DEV uniquement.
# Aucun droit Contributor n'est donné sur PROD.
resource "azurerm_role_assignment" "developers_dev_contributor" {
  scope                = local.scope_dev
  role_definition_name = "Contributor"
  principal_id         = data.azuread_group.developers.object_id
}

# Affectation du rôle Reader aux architectes sur la plateforme.
resource "azurerm_role_assignment" "architects_platform_reader" {
  scope                = local.scope_platform
  role_definition_name = "Reader"
  principal_id         = data.azuread_group.architects.object_id
}

# Affectation du rôle Reader aux architectes sur les Landing Zones.
resource "azurerm_role_assignment" "architects_landingzones_reader" {
  scope                = local.scope_landingzones
  role_definition_name = "Reader"
  principal_id         = data.azuread_group.architects.object_id
}

# Affectation du rôle Reader au groupe de lecture général.
resource "azurerm_role_assignment" "readers_landingzones_reader" {
  scope                = local.scope_landingzones
  role_definition_name = "Reader"
  principal_id         = data.azuread_group.readers.object_id
}

# Affectation du rôle Monitoring Reader au groupe de surveillance Break Glass.
resource "azurerm_role_assignment" "breakglass_monitoring_reader" {
  scope                = local.scope_management
  role_definition_name = "Monitoring Reader"
  principal_id         = data.azuread_group.breakglass_monitoring.object_id
}
```

---

# Pourquoi ne pas Utiliser for_each Maintenant ?

Nous pourrions optimiser ce code avec une map et `for_each`.

Exemple futur :

```hcl
locals {
  role_assignments = {
    platform_admins = {
      group = "AZR-MLA-PLATFORM-ADMINS"
      role  = "Contributor"
      scope = "/providers/Microsoft.Management/managementGroups/mla-platform"
    }
  }
}
```

Mais pour cet article, nous gardons une écriture explicite.

Objectif :

```text
Comprendre chaque affectation RBAC.
```

L'industrialisation avec `for_each`, modules et maps sera plus adaptée lorsque le modèle sera stabilisé.

---

# Prérequis Terraform

Pour gérer ces affectations avec Terraform, le compte ou le Service Principal utilisé doit disposer :

* du droit de lire les groupes Microsoft Entra ID via le provider `azuread` ;
* du droit de créer des affectations RBAC via Azure Resource Manager ;
* des permissions nécessaires sur les scopes ciblés ;
* d'un accès au state Terraform distant.

Dans un environnement Enterprise, ces droits doivent être validés par l'équipe IAM ou sécurité.

---

# Dépannage

## Erreur Authorization Failed

Cause probable :

```text
Le compte utilisé n'a pas le droit de créer des affectations RBAC sur le scope ciblé.
```

Solution :

* vérifier les rôles du compte ;
* vérifier le scope ;
* vérifier si le rôle `User Access Administrator` ou équivalent est nécessaire.

---

## Groupe Introuvable

Cause probable :

```text
Le groupe n'existe pas dans le tenant actif.
```

Vérifier :

```bash
az ad group show \
  --group "AZR-MLA-NETWORK-ADMINS"
```

---

## Rôle Introuvable

Cause probable :

```text
Nom du rôle mal orthographié.
```

Vérifier les rôles disponibles :

```bash
az role definition list \
  --query "[].roleName" \
  --output table
```

---

## Affectation Déjà Existante

Si l'affectation existe déjà, Azure peut retourner une erreur.

Vérifier :

```bash
az role assignment list \
  --assignee "$AZR_MLA_NETWORK_ADMINS_ID" \
  --scope "$SCOPE_CONNECTIVITY" \
  --output table
```

---

# Points de Contrôle

Avant de passer à la série suivante, vérifier que :

* les affectations correspondent à la matrice cible ;
* aucun rôle `Owner` n'a été attribué ;
* aucun rôle `Contributor` n'a été attribué à `mla-prod` ;
* les rôles sont affectés aux groupes et non aux utilisateurs ;
* les scopes sont corrects ;
* les groupes Break Glass ne sont pas utilisés pour l'administration quotidienne ;
* les affectations sont visibles dans le portail Azure ;
* les affectations sont documentées.

---

# Coûts

Azure RBAC ne génère pas de coût direct.

Cependant, un mauvais modèle RBAC peut entraîner des coûts indirects :

* création de ressources non contrôlées ;
* ressources oubliées ;
* mauvaises pratiques FinOps ;
* accès trop larges en DEV ;
* erreurs de suppression ou de modification ;
* mauvaise séparation entre environnements.

Le RBAC contribue donc indirectement à la maîtrise des coûts.

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* récupéré les Object IDs des groupes Microsoft Entra ID ;
* construit les scopes des Management Groups ;
* affecté les rôles RBAC de base ;
* appliqué le modèle défini dans SERIES-3.5 ;
* respecté ADR-004 et ADR-007 ;
* évité les droits directs aux utilisateurs ;
* évité les droits Contributor permanents sur PROD ;
* préparé l'industrialisation Terraform.

---

# Conclusion

Nous avons appliqué concrètement le modèle RBAC de MonLabAzure Cloud Platform.

Les groupes Microsoft Entra ID disposent maintenant de premiers rôles Azure RBAC alignés avec leurs responsabilités.

Cette étape est importante, car elle transforme notre modèle IAM en contrôle opérationnel réel.

Nous avons volontairement gardé une approche prudente :

```text
Pas d'Owner
Pas de Contributor en PROD
Pas d'affectation directe aux utilisateurs
Pas d'utilisation des comptes Break Glass pour l'administration quotidienne
```

Dans le prochain article, nous traiterons les comptes administratifs et les comptes Break Glass, qui constituent une partie sensible du modèle IAM.
