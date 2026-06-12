---

title: "SERIES-3.1 - Introduction à Identity & Access Management dans Azure"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "Microsoft Entra ID"
* "RBAC"
* "Security"
* "Governance"
* "Azure"

---

# SERIES-3.1 – Introduction à Identity & Access Management dans Azure

## Introduction

Les deux premières phases du projet MonLabAzure Cloud Platform ont permis de construire une base solide.

La **Phase 1 – Landing Zone Foundation** a permis de créer :

* le backend Terraform ;
* la hiérarchie des Management Groups ;
* le rattachement des abonnements Azure.

La **Phase 2 – Governance & Security Foundation** a permis de mettre en place :

* les premières Azure Policies ;
* les contrôles de tags ;
* les régions autorisées ;
* les premiers contrôles de sécurité ;
* une initiative de gouvernance centralisée.

La plateforme est maintenant structurée et gouvernée.

Cependant, une question essentielle reste à traiter :

> Qui peut accéder à quoi, avec quels droits, et dans quelles conditions ?

C'est précisément l'objectif de la **Phase 3 – Identity & Access Management**.

---

# Positionnement dans le Projet

```text
Phase 3 – Identity & Access Management

ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride ✅

SERIES-3.1 – Introduction à Identity & Access Management dans Azure ⏳
SERIES-3.2 – Comprendre les licences Microsoft Entra ID
SERIES-3.3 – Identité hybride : Entra Connect Sync, Cloud Sync et modes d'authentification
SERIES-3.4 – Créer les groupes Microsoft Entra ID
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

# Pourquoi l'IAM est Critique dans Azure ?

Dans Azure, l'identité est au centre de presque toutes les opérations.

Créer une ressource, lire une configuration, modifier une Policy, accéder à une base de données, déployer une infrastructure avec Terraform ou administrer une subscription nécessite une identité et des droits associés.

Sans stratégie IAM claire, une plateforme Azure peut rapidement devenir difficile à sécuriser.

Exemples de dérives fréquentes :

* utilisateurs directement affectés au rôle Owner ;
* trop de comptes Contributor ;
* comptes administratifs non séparés ;
* absence de MFA ;
* comptes de service avec des droits excessifs ;
* invités externes non revus ;
* comptes privilégiés non surveillés ;
* Break Glass inexistants ou mal configurés.

L'objectif de cette phase est d'éviter ces dérives.

---

# Ce que Nous Allons Construire

La Phase 3 va mettre en place les premières briques IAM de MonLabAzure Cloud Platform.

Nous allons traiter :

* Microsoft Entra ID ;
* les groupes de sécurité ;
* les rôles Azure RBAC ;
* les comptes administratifs ;
* les comptes Break Glass ;
* les comptes de service ;
* les Managed Identities ;
* l'identité hybride ;
* les licences Entra ID ;
* les accès privilégiés ;
* les logs et audits d'identité.

L'objectif n'est pas seulement de créer des groupes.

L'objectif est de construire un modèle d'accès cohérent, durable et exploitable.

---

# Les Principes IAM de MonLabAzure

Conformément à ADR-007, notre modèle IAM repose sur plusieurs principes.

---

## Aucun rôle directement affecté aux utilisateurs

Les affectations directes rendent les audits difficiles.

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
* l'automatisation ;
* la révocation ;
* la gestion des équipes.

---

## Séparation des comptes

Un compte utilisateur standard ne doit pas être utilisé pour administrer Azure.

Nous distinguerons :

```text
Compte utilisateur standard
Compte administratif nominatif
Compte de service
Managed Identity
Compte Break Glass
```

Chaque type de compte répond à un usage précis.

---

## Principe du moindre privilège

Un compte ou un groupe ne doit recevoir que les droits nécessaires.

Le rôle :

```text
Owner
```

ne doit pas être utilisé par défaut.

Nous privilégierons les rôles plus précis :

```text
Reader
Contributor
Network Contributor
Security Reader
Monitoring Reader
Log Analytics Reader
```

---

## Séparation des responsabilités

Les équipes ne doivent pas toutes disposer des mêmes droits.

Exemple :

```text
Network Team
    │
    ▼
Network Contributor sur Connectivity

Security Team
    │
    ▼
Security Reader ou Security Admin

Development Team
    │
    ▼
Contributor limité à DEV ou QUALIF
```

Cette séparation réduit les risques opérationnels et améliore la lisibilité du modèle.

---

## Accès d'urgence contrôlé

Les comptes Break Glass seront traités comme des comptes d'urgence.

Ils ne seront pas utilisés pour l'administration quotidienne.

Ils servent uniquement en cas de perte d'accès aux mécanismes normaux d'administration.

---

# Microsoft Entra ID : le Socle d'Identité

Microsoft Entra ID est le service d'identité utilisé par Azure.

Il permet de gérer :

* les utilisateurs ;
* les groupes ;
* les applications ;
* les Service Principals ;
* les rôles ;
* les accès conditionnels ;
* les journaux de connexion ;
* les identités hybrides.

Dans notre plateforme, Microsoft Entra ID servira de base à toute la gestion des accès.

---

# Azure RBAC : le Modèle d'Autorisation

Azure RBAC permet de définir ce qu'une identité peut faire sur une ressource Azure.

Le modèle repose sur trois éléments :

```text
Security Principal
Role Definition
Scope
```

---

## Security Principal

Il peut s'agir :

* d'un utilisateur ;
* d'un groupe ;
* d'un Service Principal ;
* d'une Managed Identity.

---

## Role Definition

Le rôle définit les actions autorisées.

Exemples :

```text
Reader
Contributor
Owner
Network Contributor
Security Reader
Monitoring Contributor
```

---

## Scope

Le scope définit le périmètre d'application du rôle.

Exemples :

```text
Management Group
Subscription
Resource Group
Resource
```

---

# Exemple Simple

```text
AZR-MLA-NETWORK-ADMINS
        │
        ▼
Network Contributor
        │
        ▼
Management Group mla-connectivity
```

Résultat :

Les membres du groupe `AZR-MLA-NETWORK-ADMINS` peuvent administrer les ressources réseau dans le périmètre Connectivity.

---

# Pourquoi Utiliser les Management Groups pour le RBAC ?

Les Management Groups créés en Phase 1 permettent d'appliquer les rôles à un niveau logique.

Exemple :

```text
mla-landingzones
│
├── mla-dev
├── mla-qualif
└── mla-prod
```

Un rôle affecté à `mla-landingzones` peut être hérité par DEV, QUALIF et PROD.

Cela simplifie l'administration.

Mais cela doit être utilisé avec prudence, car un mauvais rôle à un niveau trop haut peut donner trop de droits.

---

# Identité Hybride

Même si le Lab MonLabAzure démarre en mode cloud-only, de nombreuses architectures Enterprise utilisent encore Active Directory local.

Dans ces environnements, l'identité Azure dépend souvent de :

* Microsoft Entra Connect Sync ;
* Microsoft Entra Cloud Sync ;
* Password Hash Synchronization ;
* Pass-through Authentication ;
* Federation.

Cette phase intégrera donc un article dédié à l'identité hybride afin de refléter les réalités des architectures Azure d'entreprise.

---

# Licences Microsoft Entra ID

Toutes les fonctionnalités IAM ne sont pas disponibles avec le même niveau de licence.

Certaines fonctionnalités avancées nécessitent Microsoft Entra ID P1 ou P2.

Exemples :

```text
Conditional Access       → Entra ID P1
Groupes dynamiques       → Entra ID P1
PIM                      → Entra ID P2
Access Reviews           → Entra ID P2 / Governance
Identity Protection      → Entra ID P2
```

La Phase 3 distinguera clairement :

* ce qui est nécessaire pour le Lab ;
* ce qui relève d'un environnement Enterprise mature ;
* ce qui dépend des licences avancées.

---

# Les Comptes Break Glass

Les comptes Break Glass sont des comptes d'urgence.

Ils permettent de récupérer l'accès au tenant si les mécanismes normaux échouent.

Exemples de situations :

* mauvaise configuration Conditional Access ;
* MFA indisponible ;
* perte d'accès des administrateurs ;
* problème de fédération ;
* verrouillage accidentel du tenant.

MonLabAzure prévoit deux comptes Break Glass cloud-only.

Ils seront :

* exclus des règles bloquantes ;
* fortement surveillés ;
* testés régulièrement ;
* utilisés uniquement en situation d'urgence.

---

# Les Identités Machines

Les identités machines sont souvent négligées.

Pourtant, elles sont critiques.

Elles incluent :

* Service Principals ;
* App Registrations ;
* Managed Identities ;
* comptes de service ;
* secrets applicatifs ;
* certificats.

Dans les futures séries, nous verrons comment les intégrer progressivement dans le modèle IAM.

---

# Ce que Nous Ne Ferons Pas Encore

Dans cette première étape, nous ne déploierons pas encore :

* PIM ;
* Access Reviews ;
* Conditional Access avancé ;
* Identity Protection ;
* groupes dynamiques ;
* gouvernance avancée des invités.

Ces sujets seront introduits progressivement.

L'objectif est d'abord de construire une base IAM claire et solide.

---

# Livrables Attendus de la Phase 3

À la fin de la Phase 3, nous devrons disposer :

* d'un modèle IAM documenté ;
* de groupes Microsoft Entra ID cohérents ;
* d'un modèle RBAC aligné sur les Management Groups ;
* de comptes administratifs séparés ;
* de comptes Break Glass définis ;
* d'une première stratégie de comptes de service ;
* d'une vision claire des sujets IAM avancés.

---

# Coûts

Les fonctionnalités de base comme :

* utilisateurs ;
* groupes ;
* RBAC Azure ;

ne génèrent pas de coût direct spécifique dans Azure.

En revanche, certaines fonctionnalités IAM avancées peuvent nécessiter des licences Microsoft Entra ID spécifiques.

Ces coûts seront abordés dans l'article consacré aux licences Microsoft Entra ID.

---

# Conclusion

La Phase 3 marque une étape importante dans la maturité de MonLabAzure Cloud Platform.

Après avoir organisé et gouverné la plateforme, nous allons maintenant sécuriser les accès.

Cette phase permettra de passer d'une plateforme simplement structurée à une plateforme dont les droits, les responsabilités et les identités sont clairement maîtrisés.

Dans le prochain article, nous étudierons les licences Microsoft Entra ID afin de comprendre quelles fonctionnalités IAM sont disponibles selon les niveaux Free, P1, P2 et Governance.