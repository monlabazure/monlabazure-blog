---

title: "SERIES-3.4 - Créer les Groupes Microsoft Entra ID"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "Microsoft Entra ID"
* "Groups"
* "RBAC"
* "Security"
* "Azure CLI"
* "Terraform"
* "Azure"

---

# SERIES-3.4 – Créer les Groupes Microsoft Entra ID

## Introduction

Dans les articles précédents, nous avons posé les bases de la Phase 3 – Identity & Access Management.

Nous avons étudié :

* les principes IAM dans Azure ;
* les licences Microsoft Entra ID ;
* l'identité hybride ;
* les différences entre cloud-only et environnement hybride ;
* les modes d'authentification possibles.

Nous allons maintenant passer à une première implémentation concrète.

Dans cet article, nous allons créer les groupes Microsoft Entra ID qui serviront de base au modèle RBAC de MonLabAzure Cloud Platform.

L'objectif est simple :

```text
Ne jamais attribuer directement les droits Azure aux utilisateurs.
```

Le modèle retenu est :

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

Cette approche facilite :

* l'audit ;
* la délégation ;
* la révocation ;
* l'automatisation ;
* la séparation des responsabilités.

---

# Positionnement dans le Projet

```text
Phase 3 – Identity & Access Management

ADR-004 – Convention de Nommage Azure ✅
ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride ✅

SERIES-3.1 – Introduction à Identity & Access Management dans Azure ✅
SERIES-3.2 – Comprendre les licences Microsoft Entra ID ✅
SERIES-3.3 – Identité hybride : Entra Connect Sync, Cloud Sync et modes d'authentification ✅
SERIES-3.4 – Créer les groupes Microsoft Entra ID ⏳
SERIES-3.5 – Modèle RBAC Azure et séparation des rôles
SERIES-3.6 – Affecter les rôles RBAC aux Management Groups
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

Cet article s'appuie sur deux ADR déjà validés dans MonLabAzure Cloud Platform.

---

## ADR-004 – Convention de Nommage Azure

ADR-004 définit les principes de nommage utilisés dans la plateforme.

Même si les groupes Microsoft Entra ID ne sont pas des ressources Azure classiques comme un Resource Group, un Storage Account ou un Virtual Network, ils doivent respecter une convention claire, lisible et durable.

Dans cet article, la convention suivante sera utilisée :

```text
AZR-MLA-<ROLE-FONCTIONNEL>
```

Exemples :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
```

Cette convention permet d'identifier rapidement :

* le contexte Azure ;
* le projet MonLabAzure ;
* la fonction du groupe ;
* le rôle attendu dans le modèle IAM.

---

## ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride

ADR-007 définit le modèle IAM cible de MonLabAzure.

Il impose notamment les principes suivants :

* les droits Azure doivent être attribués via des groupes Microsoft Entra ID ;
* les affectations directes aux utilisateurs doivent être évitées ;
* les comptes administratifs doivent être séparés des comptes standards ;
* les responsabilités doivent être séparées par équipe ;
* les comptes Break Glass doivent rester hors administration quotidienne ;
* le modèle IAM doit préparer les futures affectations RBAC ;
* les identités techniques doivent être traitées comme des identités sensibles.

Cet article constitue donc la première mise en œuvre concrète des décisions définies dans ADR-007.

---

# Objectif de l'Article

À la fin de cet article, nous aurons :

* défini les groupes Microsoft Entra ID de base ;
* appliqué la convention de nommage définie dans ADR-004 ;
* créé les groupes nécessaires au modèle IAM défini dans ADR-007 ;
* validé la création des groupes ;
* préparé les futures affectations RBAC ;
* identifié les prérequis Azure CLI et Terraform pour gérer les groupes.

Nous ne ferons pas encore les affectations RBAC dans cet article.

Elles seront traitées dans les articles suivants.

---

# Pourquoi Utiliser des Groupes ?

Une erreur fréquente consiste à affecter directement des rôles Azure à des utilisateurs.

Exemple à éviter :

```text
user1@monlabazure.fr
        │
        ▼
Contributor
        │
        ▼
Subscription PROD
```

Ce modèle devient rapidement difficile à maintenir.

Problèmes :

* audit complexe ;
* suppression d'accès manuelle ;
* droits oubliés ;
* absence de modèle d'équipe ;
* difficulté à automatiser ;
* risque élevé en production.

Le modèle recommandé est de passer par des groupes.

Exemple :

```text
adm.user1@monlabazure.fr
        │
        ▼
AZR-MLA-OPERATIONS
        │
        ▼
Monitoring Contributor
        │
        ▼
Management Subscription
```

Le groupe devient le point central de gestion des accès.

---

# Types de Groupes Microsoft Entra ID

Microsoft Entra ID permet d'utiliser différents types de groupes.

Dans notre cas, nous utiliserons des groupes de sécurité.

---

## Groupes de Sécurité

Les groupes de sécurité servent à gérer des accès et des autorisations.

Ils peuvent être utilisés pour :

* affectations RBAC Azure ;
* accès à des applications ;
* règles Conditional Access ;
* délégations administratives ;
* gouvernance des accès.

C'est le type de groupe adapté à notre modèle IAM.

---

## Groupes Microsoft 365

Les groupes Microsoft 365 sont orientés collaboration.

Ils peuvent fournir :

* boîte aux lettres partagée ;
* calendrier ;
* espace SharePoint ;
* équipe Teams ;
* collaboration documentaire.

Ils ne sont pas le bon choix pour structurer les droits RBAC Azure de notre plateforme.

---

## Groupes Dynamiques

Les groupes dynamiques permettent d'ajouter automatiquement des membres selon des règles.

Exemple :

```text
department -eq "Cloud"
```

Dans MonLabAzure, nous ne les utiliserons pas immédiatement.

Pourquoi ?

* ils nécessitent généralement Entra ID P1 ou supérieur ;
* il faut d'abord stabiliser le modèle IAM ;
* une automatisation trop rapide peut masquer les erreurs de conception ;
* la Phase 3 commence volontairement par un modèle simple et maîtrisé.

Ils seront abordés plus tard dans les sujets IAM avancés.

---

# Convention de Nommage des Groupes

Conformément à ADR-004 et ADR-007, les groupes IAM suivront le format suivant :

```text
AZR-MLA-<ROLE-FONCTIONNEL>
```

Exemples :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
```

Signification :

| Élément          | Description                   |
| ---------------- | ----------------------------- |
| AZR              | Indique un groupe lié à Azure |
| MLA              | Abréviation de MonLabAzure    |
| ROLE-FONCTIONNEL | Fonction principale du groupe |

Cette convention permet de distinguer clairement les groupes Azure des autres groupes d'entreprise.

Elle facilite également :

* la recherche dans Microsoft Entra ID ;
* l'audit ;
* les futures affectations RBAC ;
* les revues d'accès ;
* l'automatisation.

---

# Groupes à Créer

Nous allons créer les groupes suivants :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-OPERATIONS
AZR-MLA-DEVELOPERS
AZR-MLA-ARCHITECTS
AZR-MLA-READERS
AZR-MLA-BREAKGLASS-MONITORING
```

---

# Description des Groupes

## AZR-MLA-PLATFORM-ADMINS

Groupe destiné aux administrateurs de la plateforme Azure.

Responsabilités futures :

* Management Groups ;
* abonnements ;
* gouvernance ;
* Terraform ;
* organisation globale de la plateforme.

Ce groupe recevra des droits élevés, mais contrôlés.

---

## AZR-MLA-NETWORK-ADMINS

Groupe destiné aux administrateurs réseau.

Responsabilités futures :

* Hub & Spoke ;
* Azure Firewall ;
* Bastion ;
* Private DNS ;
* VPN ;
* ExpressRoute ;
* routage.

Ce groupe sera utilisé dans les phases réseau.

---

## AZR-MLA-SECURITY-ADMINS

Groupe destiné aux équipes sécurité.

Responsabilités futures :

* Azure Policy ;
* Microsoft Defender for Cloud ;
* contrôle de conformité ;
* recommandations de sécurité ;
* investigation ;
* revue des accès.

---

## AZR-MLA-OPERATIONS

Groupe destiné aux équipes d'exploitation.

Responsabilités futures :

* supervision ;
* alertes ;
* Log Analytics ;
* dashboards ;
* sauvegardes ;
* gestion des incidents.

---

## AZR-MLA-DEVELOPERS

Groupe destiné aux équipes de développement.

Responsabilités futures :

* accès DEV ;
* accès contrôlé à QUALIF ;
* aucun accès privilégié permanent à PROD ;
* déploiements applicatifs selon le modèle défini.

---

## AZR-MLA-ARCHITECTS

Groupe destiné aux architectes.

Responsabilités futures :

* lecture globale ;
* revue d'architecture ;
* validation des designs ;
* analyse des écarts ;
* préparation des ADR.

Ce groupe ne doit pas recevoir par défaut de droits opérationnels élevés.

---

## AZR-MLA-READERS

Groupe destiné aux accès en lecture.

Responsabilités futures :

* audit ;
* reporting ;
* support ;
* observation ;
* consultation des configurations.

---

## AZR-MLA-BREAKGLASS-MONITORING

Groupe ou mécanisme logique destiné au suivi des comptes Break Glass.

Ce groupe ne sert pas à donner des droits aux comptes Break Glass.

Il sert à organiser la surveillance et les notifications liées aux comptes d'urgence.

---

# Prérequis Techniques

Pour suivre cet article, il faut disposer :

* d'un tenant Microsoft Entra ID ;
* d'un compte disposant des droits nécessaires pour créer des groupes ;
* d'Azure CLI installé ;
* d'une session Azure CLI connectée ;
* du bon tenant sélectionné dans Azure CLI.

Les commandes Azure CLI utilisées dans cet article s'appuient sur le groupe de commandes :

```bash
az ad group
```

Ce groupe de commandes permet notamment de créer, lister, afficher et gérer les membres des groupes Microsoft Entra ID.

Vérifier la connexion Azure CLI :

```bash
az account show
```

Si nécessaire, se connecter :

```bash
az login
```

Vérifier le tenant courant :

```bash
az account show \
  --query tenantId \
  --output tsv
```

Lister les tenants disponibles :

```bash
az account tenant list \
  --output table
```

---

# Prérequis pour l'Option Terraform

Si la création des groupes est réalisée avec Terraform, le provider `azuread` doit être utilisé.

Contrairement aux ressources Azure classiques gérées par le provider `azurerm`, les objets Microsoft Entra ID sont gérés par le provider `azuread`.

Le compte ou le Service Principal exécutant Terraform doit disposer des permissions Microsoft Graph nécessaires pour créer ou gérer les groupes.

Exemples de permissions courantes selon le scénario :

```text
Group.ReadWrite.All
Directory.ReadWrite.All
Group.Create
```

Dans un environnement Enterprise, ces permissions doivent être validées par l'équipe IAM ou sécurité avant d'être accordées.

Elles donnent une capacité significative sur l'annuaire et ne doivent pas être attribuées sans contrôle.

---

# Méthode Retenue

Dans cet article, nous allons créer les groupes avec Azure CLI.

Pourquoi ne pas commencer directement avec Terraform ?

Parce que cette étape sert d'abord à comprendre le modèle IAM.

Terraform sera utilisé progressivement pour industrialiser la gestion des identités et des affectations RBAC.

Pour l'instant, nous allons rester simples :

```text
Comprendre
    │
    ▼
Créer
    │
    ▼
Valider
    │
    ▼
Industrialiser plus tard
```

---

# Création des Groupes avec Azure CLI

## Groupe Platform Admins

```bash
# Création du groupe de sécurité pour les administrateurs de la plateforme Azure.
az ad group create \
  --display-name "AZR-MLA-PLATFORM-ADMINS" \
  --mail-nickname "AZR-MLA-PLATFORM-ADMINS"
```

---

## Groupe Network Admins

```bash
# Création du groupe de sécurité pour les administrateurs réseau Azure.
az ad group create \
  --display-name "AZR-MLA-NETWORK-ADMINS" \
  --mail-nickname "AZR-MLA-NETWORK-ADMINS"
```

---

## Groupe Security Admins

```bash
# Création du groupe de sécurité pour les administrateurs sécurité Azure.
az ad group create \
  --display-name "AZR-MLA-SECURITY-ADMINS" \
  --mail-nickname "AZR-MLA-SECURITY-ADMINS"
```

---

## Groupe Operations

```bash
# Création du groupe de sécurité pour l'équipe d'exploitation.
az ad group create \
  --display-name "AZR-MLA-OPERATIONS" \
  --mail-nickname "AZR-MLA-OPERATIONS"
```

---

## Groupe Developers

```bash
# Création du groupe de sécurité pour les équipes de développement.
az ad group create \
  --display-name "AZR-MLA-DEVELOPERS" \
  --mail-nickname "AZR-MLA-DEVELOPERS"
```

---

## Groupe Architects

```bash
# Création du groupe de sécurité pour l'équipe architecture.
az ad group create \
  --display-name "AZR-MLA-ARCHITECTS" \
  --mail-nickname "AZR-MLA-ARCHITECTS"
```

---

## Groupe Readers

```bash
# Création du groupe de sécurité pour les accès en lecture.
az ad group create \
  --display-name "AZR-MLA-READERS" \
  --mail-nickname "AZR-MLA-READERS"
```

---

## Groupe Break Glass Monitoring

```bash
# Création du groupe logique utilisé pour organiser la surveillance des comptes Break Glass.
az ad group create \
  --display-name "AZR-MLA-BREAKGLASS-MONITORING" \
  --mail-nickname "AZR-MLA-BREAKGLASS-MONITORING"
```

---

# Variante Optimisée avec Boucle Bash

Pour éviter de répéter plusieurs fois la même commande, il est possible d'utiliser une boucle.

```bash
# Liste des groupes Microsoft Entra ID à créer pour MonLabAzure.
groups=(
  "AZR-MLA-PLATFORM-ADMINS"
  "AZR-MLA-NETWORK-ADMINS"
  "AZR-MLA-SECURITY-ADMINS"
  "AZR-MLA-OPERATIONS"
  "AZR-MLA-DEVELOPERS"
  "AZR-MLA-ARCHITECTS"
  "AZR-MLA-READERS"
  "AZR-MLA-BREAKGLASS-MONITORING"
)

# Création de chaque groupe.
for group in "${groups[@]}"
do
  echo "Création du groupe : $group"

  az ad group create \
    --display-name "$group" \
    --mail-nickname "$group"
done
```

Cette variante est plus rapide, mais elle est moins pédagogique pour un premier passage.

---

# Variante PowerShell

Si vous travaillez depuis Windows avec PowerShell, vous pouvez utiliser la logique suivante.

```powershell
# Liste des groupes Microsoft Entra ID à créer pour MonLabAzure.
$Groups = @(
    "AZR-MLA-PLATFORM-ADMINS",
    "AZR-MLA-NETWORK-ADMINS",
    "AZR-MLA-SECURITY-ADMINS",
    "AZR-MLA-OPERATIONS",
    "AZR-MLA-DEVELOPERS",
    "AZR-MLA-ARCHITECTS",
    "AZR-MLA-READERS",
    "AZR-MLA-BREAKGLASS-MONITORING"
)

# Création des groupes.
foreach ($Group in $Groups) {
    Write-Host "Création du groupe : $Group"

    az ad group create `
        --display-name $Group `
        --mail-nickname $Group
}
```

Cette méthode est pratique pour le Lab, surtout si vous travaillez depuis un poste Windows.

---

# Vérification des Groupes

Après la création, vérifier la liste des groupes.

```bash
az ad group list \
  --filter "startswith(displayName,'AZR-MLA')" \
  --query "[].{DisplayName:displayName, ObjectId:id}" \
  --output table
```

Résultat attendu :

```text
DisplayName                         ObjectId
----------------------------------  ------------------------------------
AZR-MLA-PLATFORM-ADMINS             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZR-MLA-NETWORK-ADMINS              xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZR-MLA-SECURITY-ADMINS             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZR-MLA-OPERATIONS                  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZR-MLA-DEVELOPERS                  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZR-MLA-ARCHITECTS                  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZR-MLA-READERS                     xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZR-MLA-BREAKGLASS-MONITORING       xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

---

# Vérification d'un Groupe Spécifique

Exemple avec le groupe Platform Admins :

```bash
az ad group show \
  --group "AZR-MLA-PLATFORM-ADMINS" \
  --query "{DisplayName:displayName, ObjectId:id}" \
  --output table
```

---

# Ajouter un Membre à un Groupe

Pour ajouter un utilisateur à un groupe, il faut connaître son User Principal Name ou son Object ID.

Exemple :

```bash
# Récupération de l'Object ID de l'utilisateur.
USER_OBJECT_ID=$(az ad user show \
  --id "adm-user1@monlabazure.fr" \
  --query id \
  --output tsv)

# Ajout de l'utilisateur au groupe Platform Admins.
az ad group member add \
  --group "AZR-MLA-PLATFORM-ADMINS" \
  --member-id "$USER_OBJECT_ID"
```

Vérifier les membres :

```bash
az ad group member list \
  --group "AZR-MLA-PLATFORM-ADMINS" \
  --query "[].{DisplayName:displayName, UserPrincipalName:userPrincipalName, ObjectId:id}" \
  --output table
```

---

# Retirer un Membre d'un Groupe

```bash
# Suppression d'un membre du groupe.
az ad group member remove \
  --group "AZR-MLA-PLATFORM-ADMINS" \
  --member-id "$USER_OBJECT_ID"
```

---

# Bonnes Pratiques de Membership

Les groupes créés dans cet article ne doivent pas être remplis au hasard.

Règles recommandées :

* ajouter uniquement des comptes administratifs aux groupes privilégiés ;
* ne pas ajouter de comptes utilisateurs standards aux groupes d'administration ;
* éviter les comptes partagés ;
* documenter les membres des groupes sensibles ;
* revoir régulièrement les groupes administratifs ;
* limiter les membres permanents en production ;
* préparer PIM pour les phases avancées.

---

# Cas Particulier des Comptes Break Glass

Les comptes Break Glass ne doivent pas être ajoutés dans tous les groupes administratifs.

Ils disposent d'un rôle d'urgence spécifique.

Ils doivent rester séparés du modèle d'administration quotidien.

Le groupe :

```text
AZR-MLA-BREAKGLASS-MONITORING
```

sert à organiser la surveillance des comptes Break Glass, pas à leur donner plus de droits.

Rappel :

```text
Compte Break Glass
        │
        ▼
Utilisation exceptionnelle uniquement
        │
        ▼
Alerte critique obligatoire
```

---

# Préparer les Futures Affectations RBAC

Les groupes créés ici seront utilisés dans les prochains articles pour affecter des rôles Azure.

Exemples futurs :

```text
AZR-MLA-NETWORK-ADMINS
        │
        ▼
Network Contributor
        │
        ▼
Management Group mla-connectivity
```

```text
AZR-MLA-READERS
        │
        ▼
Reader
        │
        ▼
Management Group mla-landingzones
```

```text
AZR-MLA-OPERATIONS
        │
        ▼
Monitoring Contributor
        │
        ▼
Management Subscription
```

---

# Option Terraform

La création des groupes peut aussi être gérée avec Terraform via le provider `azuread`.

Exemple simplifié :

```hcl
# Déclaration du provider AzureAD.
# Ce provider permet de gérer les objets Microsoft Entra ID comme les groupes,
# les utilisateurs, les applications et les Service Principals.
provider "azuread" {}

# Liste locale des groupes Microsoft Entra ID à créer.
locals {
  entra_groups = [
    "AZR-MLA-PLATFORM-ADMINS",
    "AZR-MLA-NETWORK-ADMINS",
    "AZR-MLA-SECURITY-ADMINS",
    "AZR-MLA-OPERATIONS",
    "AZR-MLA-DEVELOPERS",
    "AZR-MLA-ARCHITECTS",
    "AZR-MLA-READERS",
    "AZR-MLA-BREAKGLASS-MONITORING"
  ]
}

# Création des groupes Microsoft Entra ID.
# for_each permet de créer un groupe par entrée dans la liste.
resource "azuread_group" "mla_groups" {
  for_each = toset(local.entra_groups)

  # Nom visible du groupe dans Microsoft Entra ID.
  display_name = each.value

  # Groupe de sécurité utilisé pour les autorisations.
  security_enabled = true
}
```

Cette approche sera utile pour industrialiser le modèle IAM.

Cependant, elle nécessite de bien gérer les permissions du compte ou du Service Principal utilisé par Terraform.

---

# Points de Vigilance Terraform

La gestion des groupes Entra ID avec Terraform nécessite une attention particulière.

Contrairement aux ressources Azure classiques gérées par le provider `azurerm`, les groupes Microsoft Entra ID sont gérés via le provider `azuread`.

Cela implique des permissions Microsoft Graph spécifiques.

Dans un Lab, il est possible de tester avec un compte disposant des droits nécessaires.

Dans un environnement Enterprise, il est préférable de définir :

* qui est autorisé à créer les groupes ;
* quel Service Principal exécute Terraform ;
* quelles permissions Microsoft Graph sont accordées ;
* qui valide les changements IAM ;
* comment les changements sont audités.

La création de groupes IAM ne doit pas être considérée comme une simple tâche technique.

Elle impacte directement le modèle d'accès de la plateforme.

---

# Pourquoi Ne Pas Tout Faire avec Terraform Maintenant ?

Nous pourrions tout automatiser dès maintenant.

Mais ce n'est pas le meilleur choix pédagogique pour cette étape.

La Phase 3 doit d'abord clarifier :

* les types de groupes ;
* les responsabilités ;
* les memberships ;
* les rôles futurs ;
* les risques ;
* les exceptions.

Terraform sera très utile ensuite pour fiabiliser les affectations RBAC, les rôles personnalisés ou certaines configurations répétables.

La logique reste la même que dans les phases précédentes :

```text
Comprendre d'abord
Industrialiser ensuite
```

---

# Vérification dans le Portail Azure

Il est aussi possible de vérifier les groupes dans le portail.

Chemin :

```text
Microsoft Entra ID
        │
        ▼
Groups
        │
        ▼
All groups
```

Rechercher :

```text
AZR-MLA
```

Les huit groupes doivent être visibles.

---

# Points de Contrôle

Avant de passer à l'article suivant, vérifier que :

* les huit groupes sont créés ;
* les noms respectent la convention ADR-004 ;
* les groupes correspondent au modèle IAM ADR-007 ;
* aucun rôle RBAC n'est encore affecté par erreur ;
* aucun utilisateur standard n'est ajouté aux groupes privilégiés ;
* les comptes Break Glass restent séparés ;
* les Object IDs des groupes sont récupérables ;
* la liste des groupes est documentée ;
* les permissions Terraform éventuelles sont validées avant automatisation.

---

# Dépannage

## Le Groupe Existe Déjà

Si un groupe existe déjà, Azure CLI peut retourner une erreur.

Vérifier avec :

```bash
az ad group show \
  --group "AZR-MLA-PLATFORM-ADMINS"
```

Si le groupe existe, il n'est pas nécessaire de le recréer.

---

## Erreur de Permission

Si la création échoue, vérifier que le compte utilisé dispose des droits nécessaires dans Microsoft Entra ID.

Selon l'organisation, seuls certains rôles peuvent créer ou administrer des groupes.

Dans un environnement Enterprise, la création de groupes peut être restreinte à l'équipe IAM.

---

## Mauvais Tenant

Si les groupes ne sont pas visibles, vérifier le tenant actif.

```bash
az account show \
  --query tenantId \
  --output tsv
```

Puis vérifier la liste des tenants disponibles :

```bash
az account tenant list \
  --output table
```

---

## Problème avec Terraform

Si Terraform échoue lors de la création des groupes, vérifier :

* le provider `azuread` ;
* l'authentification utilisée ;
* les permissions Microsoft Graph ;
* les consentements administrateur ;
* le tenant ciblé ;
* les droits du Service Principal.

Les erreurs Terraform liées à Microsoft Entra ID sont souvent causées par un manque de permissions Microsoft Graph.

---

# Coûts

La création de groupes de sécurité Microsoft Entra ID ne génère pas de coût Azure direct spécifique.

Cependant, certaines fonctionnalités avancées liées aux groupes peuvent dépendre des licences.

Exemples :

* groupes dynamiques ;
* Access Reviews ;
* PIM for Groups ;
* gouvernance avancée.

Ces sujets seront traités plus tard.

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* créé les groupes Microsoft Entra ID de base ;
* appliqué la convention de nommage issue d'ADR-004 ;
* commencé l'implémentation concrète du modèle IAM défini dans ADR-007 ;
* préparé le modèle RBAC ;
* séparé les responsabilités par équipe ;
* conservé les comptes Break Glass hors administration quotidienne ;
* identifié les prérequis Azure CLI ;
* identifié les prérequis Terraform et Microsoft Graph ;
* préparé l'industrialisation future avec Terraform.

---

# Conclusion

Nous avons maintenant créé les groupes Microsoft Entra ID qui serviront de base à notre modèle IAM.

Cette étape est volontairement simple, mais structurante.

À partir de maintenant, les rôles Azure ne devront plus être pensés utilisateur par utilisateur.

Ils devront être pensés à travers des groupes, des responsabilités et des scopes.

Cette série applique directement deux décisions d'architecture importantes :

```text
ADR-004 – Convention de Nommage Azure
ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride
```

Dans le prochain article, nous définirons le modèle RBAC Azure et la séparation des rôles entre les équipes MonLabAzure.