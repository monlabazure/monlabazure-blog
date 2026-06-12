---

title: "SERIES-3.5 - Modèle RBAC Azure et Séparation des Rôles"
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
* "Least Privilege"
* "Security"
* "Azure"

---

# SERIES-3.5 – Modèle RBAC Azure et Séparation des Rôles

## Introduction

Dans l'article précédent, nous avons créé les groupes Microsoft Entra ID qui serviront de base au modèle IAM de MonLabAzure Cloud Platform.

Nous disposons maintenant des groupes suivants :

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

Ces groupes ne portent pas encore de droits Azure.

Ils constituent uniquement la structure d'identité.

L'étape suivante consiste à définir le modèle RBAC cible.

Autrement dit :

```text
Quel groupe doit recevoir quel rôle, sur quel périmètre, et pourquoi ?
```

Cet article ne réalise pas encore les affectations RBAC.

Il définit le modèle d'autorisation qui sera implémenté dans la série suivante.

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
SERIES-3.5 – Modèle RBAC Azure et séparation des rôles ⏳
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

Cet article s'appuie principalement sur deux ADR.

## ADR-004 – Convention de Nommage Azure

ADR-004 définit les conventions de nommage utilisées dans MonLabAzure Cloud Platform.

Les groupes Microsoft Entra ID créés dans la série précédente suivent cette logique :

```text
AZR-MLA-<ROLE-FONCTIONNEL>
```

Cette convention permet de rendre les groupes lisibles, auditables et facilement identifiables.

## ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride

ADR-007 définit les principes IAM de la plateforme :

* aucun rôle directement attribué aux utilisateurs ;
* affectation des rôles via des groupes Microsoft Entra ID ;
* séparation des comptes standards et administratifs ;
* application du moindre privilège ;
* séparation des responsabilités ;
* limitation des rôles élevés ;
* préparation des accès temporaires avec PIM dans les phases avancées.

Cet article applique ces principes au modèle RBAC Azure.

---

# Objectif de l'Article

À la fin de cet article, nous aurons défini :

* les concepts clés d'Azure RBAC ;
* la différence entre rôle Azure RBAC et rôle Microsoft Entra ID ;
* les scopes utilisés dans MonLabAzure ;
* les rôles intégrés Azure à privilégier ;
* les rôles à éviter ou à limiter ;
* le modèle RBAC cible par groupe ;
* les règles de séparation des responsabilités ;
* les principes à respecter avant les affectations concrètes.

Nous ne ferons pas encore de commande Azure CLI ou Terraform pour affecter les rôles.

Cela sera traité dans l'article suivant.

---

# Rappel : Qu'est-ce qu'Azure RBAC ?

Azure RBAC signifie :

```text
Azure Role-Based Access Control
```

C'est le mécanisme qui permet de contrôler les accès aux ressources Azure.

Une affectation RBAC répond à trois questions :

```text
Qui ?
Quel rôle ?
Sur quel périmètre ?
```

Dans Azure, cela correspond à trois éléments :

```text
Security Principal
Role Definition
Scope
```

---

# Les Trois Composants d'une Affectation RBAC

## Security Principal

Le Security Principal représente l'identité qui reçoit le droit.

Il peut s'agir :

* d'un utilisateur ;
* d'un groupe Microsoft Entra ID ;
* d'un Service Principal ;
* d'une Managed Identity.

Dans MonLabAzure, nous privilégierons les groupes.

Exemple :

```text
AZR-MLA-NETWORK-ADMINS
```

---

## Role Definition

La Role Definition définit les actions autorisées.

Exemples :

```text
Reader
Contributor
Owner
Network Contributor
Monitoring Contributor
Security Reader
```

Un rôle peut être :

* un rôle intégré Microsoft ;
* un rôle personnalisé.

Dans cette phase, nous utiliserons principalement les rôles intégrés.

---

## Scope

Le Scope définit le périmètre sur lequel le rôle s'applique.

Exemples :

```text
Management Group
Subscription
Resource Group
Resource
```

Plus le scope est haut, plus l'impact est large.

Plus le scope est bas, plus l'accès est précis.

Microsoft définit les scopes Azure RBAC à quatre niveaux principaux : Management Group, Subscription, Resource Group et Resource. Les niveaux inférieurs héritent des permissions affectées aux niveaux supérieurs.

---

# Exemple Simple d'Affectation RBAC

Exemple :

```text
AZR-MLA-NETWORK-ADMINS
        │
        ▼
Network Contributor
        │
        ▼
Management Group mla-connectivity
```

Interprétation :

Les membres du groupe `AZR-MLA-NETWORK-ADMINS` pourront administrer les ressources réseau dans le périmètre `mla-connectivity`.

---

# Azure RBAC vs Rôles Microsoft Entra ID

Il faut bien distinguer deux modèles de rôles.

## Azure RBAC

Azure RBAC contrôle l'accès aux ressources Azure.

Exemples :

* Resource Groups ;
* Virtual Networks ;
* Storage Accounts ;
* Azure Firewall ;
* Key Vault ;
* Log Analytics ;
* App Service.

Exemples de rôles Azure RBAC :

```text
Reader
Contributor
Owner
Network Contributor
Monitoring Contributor
Log Analytics Contributor
```

---

## Rôles Microsoft Entra ID

Les rôles Microsoft Entra ID contrôlent l'administration du tenant et des objets d'identité.

Exemples :

* utilisateurs ;
* groupes ;
* applications ;
* rôles administratifs Entra ;
* Conditional Access ;
* identités hybrides.

Exemples de rôles Entra ID :

```text
Global Administrator
User Administrator
Groups Administrator
Application Administrator
Security Administrator
Conditional Access Administrator
```

Un utilisateur peut être `Owner` sur une subscription Azure sans être `Global Administrator` dans Microsoft Entra ID.

Inversement, un `Global Administrator` peut administrer le tenant Entra ID sans avoir automatiquement tous les droits sur toutes les ressources Azure, sauf configuration spécifique.

Cette distinction est essentielle.

---

# Pourquoi Définir un Modèle RBAC Avant les Affectations ?

Il est possible d'affecter des rôles directement depuis le portail Azure.

Mais sans modèle clair, les affectations deviennent vite incohérentes.

Exemples de dérives :

```text
Trop de Contributor
Trop de Owner
Droits directs aux utilisateurs
Accès production non contrôlé
Groupes mal nommés
Scopes trop larges
Absence de revue
```

Le modèle RBAC doit donc être conçu avant d'être appliqué.

Dans MonLabAzure, nous allons d'abord définir la matrice des rôles, puis seulement ensuite l'implémenter.

---

# Hiérarchie Azure Utilisée

La hiérarchie de Management Groups a été définie en Phase 1.

Rappel :

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

Cette hiérarchie servira de base au modèle RBAC.

---

# Principe de Scope

## Scope Haut

Un scope haut donne des droits larges.

Exemple :

```text
Management Group mla-landingzones
```

Un rôle affecté à ce niveau peut s'appliquer à DEV, QUALIF et PROD.

Avantage :

* administration centralisée ;
* héritage automatique ;
* moins d'affectations répétées.

Risque :

* impact trop large ;
* accès non souhaité à PROD ;
* erreur difficile à contenir.

---

## Scope Bas

Un scope bas donne des droits plus précis.

Exemple :

```text
Subscription DEV
```

ou :

```text
Resource Group applicatif DEV
```

Avantage :

* moindre privilège ;
* meilleure séparation ;
* réduction de l'impact.

Risque :

* plus d'affectations à gérer ;
* modèle plus détaillé ;
* besoin d'automatisation.

---

# Règle MonLabAzure

La règle retenue est la suivante :

```text
Attribuer le rôle au niveau le plus haut uniquement lorsque l'héritage est justifié.
Sinon, attribuer le rôle au niveau le plus bas compatible avec le besoin.
```

Exemple :

```text
Reader global pour audit
        │
        ▼
Possible au niveau Management Group

Contributor pour développeurs
        │
        ▼
À limiter à DEV ou QUALIF
```

---

# Rôles Azure RBAC à Connaître

Azure fournit de nombreux rôles intégrés. Microsoft précise que les rôles intégrés peuvent être affectés à des utilisateurs, groupes, Service Principals et Managed Identities, et que des rôles personnalisés peuvent être créés lorsque les rôles intégrés ne couvrent pas un besoin spécifique.

Dans cette phase, nous utiliserons un nombre limité de rôles pour garder le modèle lisible.

---

## Reader

Le rôle `Reader` permet de lire les ressources sans les modifier.

Usage typique :

* audit ;
* architecture ;
* support ;
* observation ;
* reporting.

Dans MonLabAzure, ce rôle sera utilisé pour les groupes qui doivent consulter sans administrer.

---

## Contributor

Le rôle `Contributor` permet de gérer les ressources, mais ne permet pas de gérer les affectations RBAC.

Usage typique :

* administration technique ;
* déploiement ;
* gestion de ressources.

Point d'attention :

`Contributor` reste un rôle large.

Il ne doit pas être attribué trop haut sans justification.

---

## Owner

Le rôle `Owner` donne tous les droits sur les ressources et permet aussi de gérer les accès RBAC.

C'est un rôle très sensible.

Dans MonLabAzure, `Owner` ne doit pas être le rôle par défaut.

Il doit être réservé à des cas exceptionnels :

* bootstrap initial ;
* comptes ou groupes très contrôlés ;
* actions spécifiques de gouvernance ;
* scénario temporaire documenté.

---

## User Access Administrator

Le rôle `User Access Administrator` permet de gérer les affectations RBAC.

Il ne donne pas nécessairement le droit de modifier toutes les ressources, mais il donne un pouvoir critique : accorder ou retirer des accès.

Ce rôle doit être fortement limité.

Dans MonLabAzure, il ne sera pas attribué largement.

---

## Network Contributor

Le rôle `Network Contributor` permet de gérer les ressources réseau.

Usage typique :

* Virtual Networks ;
* Subnets ;
* NSG ;
* Route Tables ;
* Load Balancers ;
* composants réseau selon le périmètre.

Il est adapté au groupe :

```text
AZR-MLA-NETWORK-ADMINS
```

---

## Security Reader

Le rôle `Security Reader` permet de consulter les informations de sécurité.

Usage typique :

* équipe sécurité ;
* audit ;
* investigation ;
* lecture des recommandations.

Il est adapté à une équipe sécurité qui doit observer sans nécessairement modifier.

---

## Monitoring Reader

Le rôle `Monitoring Reader` permet de consulter les données de supervision.

Usage typique :

* exploitation ;
* support ;
* observation ;
* diagnostic initial.

---

## Monitoring Contributor

Le rôle `Monitoring Contributor` permet de gérer des paramètres liés à la supervision.

Usage typique :

* alertes ;
* métriques ;
* configuration de supervision ;
* dashboards selon les scénarios.

À utiliser pour l'équipe Operations sur le périmètre Management ou Monitoring.

---

## Log Analytics Reader

Le rôle `Log Analytics Reader` permet de consulter les données Log Analytics.

Usage typique :

* lecture des logs ;
* investigation ;
* support ;
* sécurité.

Ce rôle sera utile dans les phases Monitoring et Security.

---

## Log Analytics Contributor

Le rôle `Log Analytics Contributor` permet de gérer les espaces Log Analytics.

Usage typique :

* configuration des workspaces ;
* gestion de certains paramètres ;
* administration de la supervision.

Ce rôle doit être limité à l'équipe Operations ou Platform selon le design.

---

# Rôles à Limiter

Certains rôles doivent être utilisés avec prudence.

```text
Owner
User Access Administrator
Contributor à scope élevé
Global Administrator côté Entra ID
```

Ces rôles ne doivent pas être banalisés.

Une erreur fréquente consiste à donner `Owner` ou `Contributor` pour résoudre rapidement un problème.

Cette pratique doit être évitée.

---

# Modèle RBAC Cible MonLabAzure

Le modèle ci-dessous définit les intentions d'affectation.

Les affectations exactes seront réalisées dans l'article suivant.

---

## AZR-MLA-PLATFORM-ADMINS

Responsabilité :

* gouvernance globale ;
* plateforme Azure ;
* Management Groups ;
* organisation des abonnements ;
* Terraform ;
* décisions transverses.

Rôles possibles :

```text
Contributor
Reader
User Access Administrator
```

Scopes possibles :

```text
mla-platform
mla-management
Scopes spécifiques de bootstrap
```

Point de vigilance :

Le rôle `User Access Administrator` doit rester limité.

Il sera utilisé seulement si l'équipe plateforme doit gérer les affectations RBAC.

---

## AZR-MLA-NETWORK-ADMINS

Responsabilité :

* réseau Azure ;
* Hub & Spoke ;
* firewall ;
* bastion ;
* routage ;
* DNS privé ;
* connectivité hybride.

Rôles possibles :

```text
Network Contributor
Reader
```

Scopes possibles :

```text
mla-connectivity
Ressources réseau spécifiques
```

Règle :

Le groupe réseau ne doit pas recevoir `Contributor` global si `Network Contributor` suffit.

---

## AZR-MLA-SECURITY-ADMINS

Responsabilité :

* sécurité ;
* conformité ;
* Defender for Cloud ;
* Azure Policy ;
* recommandations ;
* investigation.

Rôles possibles :

```text
Security Reader
Reader
Security Admin selon besoin
Policy Contributor selon besoin
```

Scopes possibles :

```text
mla-platform
mla-landingzones
Scopes sécurité spécifiques
```

Point de vigilance :

La sécurité doit pouvoir auditer largement, mais modifier uniquement ce qui relève de son périmètre.

---

## AZR-MLA-OPERATIONS

Responsabilité :

* exploitation ;
* supervision ;
* alertes ;
* incidents ;
* sauvegardes ;
* disponibilité.

Rôles possibles :

```text
Monitoring Reader
Monitoring Contributor
Log Analytics Reader
Log Analytics Contributor
Reader
```

Scopes possibles :

```text
mla-management
Subscriptions supervisées
Resource Groups de monitoring
```

Règle :

L'équipe Operations doit pouvoir exploiter et diagnostiquer sans recevoir systématiquement des droits Contributor sur toute la plateforme.

---

## AZR-MLA-DEVELOPERS

Responsabilité :

* développement applicatif ;
* tests ;
* déploiements applicatifs ;
* environnement DEV ;
* contribution contrôlée en QUALIF.

Rôles possibles :

```text
Reader
Contributor
```

Scopes possibles :

```text
mla-dev
Resource Groups applicatifs DEV
Resource Groups applicatifs QUALIF selon besoin
```

Règle :

Aucun accès privilégié permanent à PROD.

Le rôle `Contributor` doit être limité à DEV ou à des périmètres QUALIF clairement définis.

---

## AZR-MLA-ARCHITECTS

Responsabilité :

* conception ;
* revue ;
* validation ;
* analyse ;
* documentation ;
* ADR.

Rôles possibles :

```text
Reader
Security Reader selon besoin
```

Scopes possibles :

```text
mla-platform
mla-landingzones
```

Règle :

L'équipe Architecture doit voir largement, mais ne doit pas nécessairement modifier.

---

## AZR-MLA-READERS

Responsabilité :

* lecture ;
* audit ;
* support ;
* reporting.

Rôles possibles :

```text
Reader
```

Scopes possibles :

```text
mla-platform
mla-landingzones
Scopes spécifiques
```

Règle :

Ce groupe ne doit jamais recevoir Contributor.

---

## AZR-MLA-BREAKGLASS-MONITORING

Responsabilité :

* surveillance des comptes Break Glass ;
* notifications ;
* revue de sécurité.

Rôles possibles :

```text
Reader
Security Reader
Monitoring Reader
```

Scopes possibles :

```text
Périmètres de logs et supervision
```

Règle :

Ce groupe ne sert pas à donner des droits d'administration aux comptes Break Glass.

Il sert à organiser la surveillance.

---

# Matrice RBAC Cible

| Groupe                        | Rôle cible principal                               | Scope cible                  | Commentaire                    |
| ----------------------------- | -------------------------------------------------- | ---------------------------- | ------------------------------ |
| AZR-MLA-PLATFORM-ADMINS       | Contributor / Reader                               | mla-platform, mla-management | Administration plateforme      |
| AZR-MLA-NETWORK-ADMINS        | Network Contributor                                | mla-connectivity             | Administration réseau          |
| AZR-MLA-SECURITY-ADMINS       | Security Reader / Reader                           | plateforme et landing zones  | Audit et sécurité              |
| AZR-MLA-OPERATIONS            | Monitoring Contributor / Log Analytics Contributor | mla-management               | Supervision et exploitation    |
| AZR-MLA-DEVELOPERS            | Contributor limité                                 | DEV / QUALIF                 | Aucun privilège permanent PROD |
| AZR-MLA-ARCHITECTS            | Reader                                             | plateforme et landing zones  | Revue et conception            |
| AZR-MLA-READERS               | Reader                                             | scopes définis               | Lecture uniquement             |
| AZR-MLA-BREAKGLASS-MONITORING | Security Reader / Monitoring Reader                | logs et supervision          | Surveillance Break Glass       |

---

# Modèle par Environnement

## DEV

En DEV, les développeurs peuvent recevoir des droits plus larges.

Exemple :

```text
AZR-MLA-DEVELOPERS
        │
        ▼
Contributor
        │
        ▼
mla-dev
```

Objectif :

* permettre l'expérimentation ;
* accélérer les tests ;
* faciliter les déploiements applicatifs ;
* maintenir un risque limité.

---

## QUALIF

En QUALIF, les droits doivent être plus contrôlés.

Exemple :

```text
AZR-MLA-DEVELOPERS
        │
        ▼
Contributor ou rôle plus limité
        │
        ▼
Resource Groups applicatifs QUALIF
```

Objectif :

* valider les changements ;
* limiter les modifications non maîtrisées ;
* se rapprocher de la production.

---

## PROD

En PROD, les droits permanents doivent être strictement limités.

Règles :

* aucun Contributor permanent pour les développeurs ;
* accès lecture possible pour Architecture et Sécurité ;
* actions opérationnelles limitées à Operations ;
* élévation temporaire à prévoir avec PIM plus tard ;
* toute exception doit être documentée.

Exemple :

```text
AZR-MLA-OPERATIONS
        │
        ▼
Monitoring Reader / Monitoring Contributor
        │
        ▼
Périmètres PROD nécessaires
```

---

# Modèle par Domaine Technique

## Réseau

Groupe principal :

```text
AZR-MLA-NETWORK-ADMINS
```

Rôle cible :

```text
Network Contributor
```

Scope cible :

```text
mla-connectivity
```

Justification :

Le réseau est une fonction transversale critique.

Les droits réseau doivent être séparés des droits applicatifs.

---

## Sécurité

Groupe principal :

```text
AZR-MLA-SECURITY-ADMINS
```

Rôles cibles :

```text
Security Reader
Reader
Policy Contributor selon besoin
```

Justification :

L'équipe sécurité doit pouvoir auditer largement.

Les droits de modification doivent être limités aux composants de sécurité.

---

## Supervision

Groupe principal :

```text
AZR-MLA-OPERATIONS
```

Rôles cibles :

```text
Monitoring Contributor
Log Analytics Contributor
Monitoring Reader
```

Justification :

L'équipe Operations doit pouvoir exploiter la plateforme sans posséder systématiquement des droits Contributor globaux.

---

## Applications

Groupe principal :

```text
AZR-MLA-DEVELOPERS
```

Rôles cibles :

```text
Contributor
Reader
```

Scopes cibles :

```text
DEV
QUALIF selon besoin
```

Justification :

Les équipes applicatives doivent pouvoir travailler dans DEV et QUALIF, mais pas administrer librement PROD.

---

# Cas du Rôle Owner

Le rôle `Owner` doit être traité comme un rôle exceptionnel.

Il cumule :

```text
Gestion des ressources
+
Gestion des accès RBAC
```

Risques :

* élévation de privilèges ;
* modification des accès ;
* perte de contrôle ;
* erreurs à large impact.

Dans MonLabAzure :

```text
Owner ne doit pas être attribué aux groupes standards.
```

Il peut être utilisé uniquement pour :

* bootstrap initial ;
* intervention exceptionnelle ;
* compte ou groupe très contrôlé ;
* scénario temporaire documenté.

À terme, l'usage d'Owner devra être remplacé par :

* rôles plus précis ;
* PIM ;
* élévation temporaire ;
* rôles personnalisés si nécessaire.

---

# Cas du Rôle Contributor

Le rôle `Contributor` est très pratique, mais il reste large.

Il permet de gérer de nombreuses ressources.

Dans MonLabAzure :

```text
Contributor est acceptable en DEV.
Contributor est contrôlé en QUALIF.
Contributor est fortement limité en PROD.
```

Règle :

```text
Pas de Contributor permanent sur PROD pour les développeurs.
```

---

# Cas du Rôle User Access Administrator

Le rôle `User Access Administrator` permet de gérer les affectations RBAC.

Il est donc sensible.

Dans MonLabAzure :

```text
User Access Administrator doit être limité à la Platform Team ou à un processus contrôlé.
```

Il ne doit jamais être accordé largement.

---

# Rôles Personnalisés

Les rôles personnalisés ne seront pas créés immédiatement.

Pourquoi ?

* les rôles intégrés suffisent pour démarrer ;
* il faut d'abord comprendre les besoins ;
* les rôles personnalisés ajoutent de la complexité ;
* ils doivent être maintenus dans le temps.

Ils seront envisagés lorsque :

* un rôle intégré est trop large ;
* un besoin est stable ;
* le besoin est récurrent ;
* la sécurité impose une granularité plus fine.

Exemple futur :

```text
Rôle personnalisé permettant de redémarrer une VM
sans modifier le réseau, le disque ou les accès.
```

---

# Séparation des Responsabilités

La séparation des responsabilités est un principe clé d'ADR-007.

Exemple de séparation cible :

```text
Platform Team
    → gouvernance et socle Azure

Network Team
    → réseau et connectivité

Security Team
    → sécurité et conformité

Operations Team
    → supervision et exploitation

Development Team
    → workloads applicatifs

Architecture Team
    → lecture, conception, validation
```

Cette séparation évite qu'une seule équipe dispose de tous les droits sur tous les composants.

---

# Règles de Gouvernance RBAC

MonLabAzure adopte les règles suivantes.

## Règle 1 – Pas d'affectation directe aux utilisateurs

Les rôles doivent être affectés aux groupes.

Exception possible :

```text
Temporaire
Documentée
Approuvée
Revue
```

---

## Règle 2 – Pas de droits élevés permanents sans justification

Les rôles élevés doivent être limités.

Exemples :

```text
Owner
User Access Administrator
Contributor à scope élevé
```

---

## Règle 3 – Pas de Contributor permanent en PROD pour les développeurs

Les développeurs peuvent avoir des droits en DEV.

Ils peuvent avoir des droits contrôlés en QUALIF.

Ils ne doivent pas disposer de droits Contributor permanents en PROD.

---

## Règle 4 – Reader avant Contributor

Lorsqu'un besoin est de consulter, analyser ou auditer, `Reader` doit être préféré à `Contributor`.

---

## Règle 5 – Scope le plus bas possible

Le scope doit être choisi selon le besoin réel.

```text
Besoin global
    → Management Group possible

Besoin spécifique
    → Subscription, Resource Group ou Resource
```

---

## Règle 6 – Toute exception doit être documentée

Une exception RBAC doit préciser :

* bénéficiaire ;
* rôle ;
* scope ;
* justification ;
* durée ;
* approbateur ;
* date de revue.

---

# Points d'Audit

Le modèle RBAC devra être régulièrement audité.

Contrôles à prévoir :

```text
Qui possède Owner ?
Qui possède User Access Administrator ?
Quels groupes ont Contributor ?
Quels droits existent sur PROD ?
Existe-t-il des affectations directes à des utilisateurs ?
Existe-t-il des Service Principals avec droits élevés ?
Les comptes Break Glass ont-ils été utilisés ?
Les accès sont-ils alignés avec ADR-007 ?
```

Ces contrôles seront repris dans les séries de validation et dans les phases avancées avec Access Reviews.

---

# Ce que Nous Ne Faisons Pas Encore

Dans cet article, nous ne faisons pas encore :

* affectation RBAC avec Azure CLI ;
* affectation RBAC avec Terraform ;
* création de rôles personnalisés ;
* activation de PIM ;
* Access Reviews ;
* Conditional Access ;
* modification des comptes Break Glass.

L'objectif ici est de concevoir proprement le modèle.

---

# Préparation de l'Implémentation

Dans l'article suivant, nous utiliserons ce modèle pour réaliser les premières affectations RBAC.

Nous aurons besoin :

* des Object IDs des groupes ;
* des IDs des Management Groups ;
* des rôles Azure ciblés ;
* des scopes d'affectation ;
* d'une méthode d'implémentation.

Exemple de future affectation :

```text
Groupe :
AZR-MLA-NETWORK-ADMINS

Rôle :
Network Contributor

Scope :
/providers/Microsoft.Management/managementGroups/mla-connectivity
```

---

# Coûts

Azure RBAC ne génère pas de coût direct spécifique.

Cependant, un mauvais modèle RBAC peut générer des coûts indirects :

* ressources créées sans contrôle ;
* erreurs de déploiement ;
* mauvaise gouvernance ;
* ressources non supprimées ;
* accès trop larges en DEV ou PROD ;
* difficulté à identifier les responsables.

Le RBAC est donc un sujet de sécurité, mais aussi de gouvernance et de FinOps.

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* clarifié le fonctionnement d'Azure RBAC ;
* distingué Azure RBAC et rôles Microsoft Entra ID ;
* défini les scopes utilisés dans MonLabAzure ;
* identifié les rôles Azure à privilégier ;
* identifié les rôles sensibles à limiter ;
* défini une matrice RBAC cible ;
* posé les règles de séparation des responsabilités ;
* préparé l'implémentation de SERIES-3.6.

---

# Conclusion

Nous avons défini le modèle RBAC cible de MonLabAzure Cloud Platform.

Cette étape est essentielle.

Sans modèle clair, les affectations de rôles deviennent rapidement incohérentes, difficiles à auditer et dangereuses en production.

Le modèle retenu repose sur des principes simples :

```text
Groupes plutôt qu'utilisateurs
Moindre privilège
Scopes maîtrisés
Séparation des responsabilités
Pas de droits élevés permanents sans justification
```

Dans le prochain article, nous passerons à l'implémentation concrète avec l'affectation des rôles RBAC aux Management Groups.