---

title: "SERIES-3.10 - PIM, Access Reviews et Accès Temporaires"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "PIM"
* "Privileged Identity Management"
* "Access Reviews"
* "Just In Time"
* "Microsoft Entra ID"
* "Governance"
* "Azure"

---

# SERIES-3.10 – PIM, Access Reviews et Accès Temporaires

## Introduction

Dans les articles précédents, nous avons construit une base IAM solide pour MonLabAzure Cloud Platform.

Nous avons traité :

* les licences Microsoft Entra ID ;
* l'identité hybride ;
* les groupes Microsoft Entra ID ;
* le modèle RBAC Azure ;
* les affectations RBAC aux Management Groups ;
* les comptes administratifs ;
* les comptes Break Glass ;
* MFA et les méthodes phishing-resistant ;
* les fondations Conditional Access.

Nous devons maintenant traiter un sujet central dans les environnements Enterprise :

```text
La réduction des privilèges permanents.
```

Dans un modèle RBAC classique, un utilisateur membre d'un groupe administratif dispose souvent de droits en permanence.

Exemple :

```text
adm-user@monlabazure.fr
        │
        ▼
AZR-MLA-PLATFORM-ADMINS
        │
        ▼
Contributor
        │
        ▼
mla-platform
```

Ce modèle est simple, mais il présente un risque.

Si le compte est compromis, les droits sont immédiatement exploitables.

L'objectif de PIM, des Access Reviews et des accès temporaires est de passer d'un modèle permanent à un modèle contrôlé, temporaire et auditable.

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
SERIES-3.6 – Affecter les rôles RBAC aux Management Groups ✅
SERIES-3.7 – Comptes administratifs et comptes Break Glass ✅
SERIES-3.8 – MFA, authentification forte et méthodes phishing-resistant ✅
SERIES-3.9 – Conditional Access Foundation ✅
SERIES-3.10 – PIM, Access Reviews et accès temporaires ⏳
SERIES-3.11 – Comptes invités et gouvernance B2B
SERIES-3.12 – Service Principals, App Registrations et Managed Identities
SERIES-3.13 – Export des logs Entra ID vers Log Analytics
SERIES-3.14 – Validation IAM Foundation
```

---

# Références d'Architecture

Cet article applique principalement **ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride**.

ADR-007 définit les principes suivants :

* limiter les droits permanents ;
* appliquer le moindre privilège ;
* séparer les responsabilités ;
* utiliser des comptes administratifs nominatifs ;
* surveiller les comptes sensibles ;
* préparer PIM pour les rôles privilégiés ;
* préparer les Access Reviews pour les groupes sensibles ;
* documenter les exceptions et les accès temporaires.

Cette série vient renforcer le modèle IAM déjà construit.

---

# Objectif de l'Article

À la fin de cet article, nous aurons défini :

* le rôle de Microsoft Entra Privileged Identity Management ;
* la différence entre rôle actif et rôle éligible ;
* les principes des accès Just-In-Time ;
* les scénarios PIM pour les rôles Microsoft Entra ID ;
* les scénarios PIM pour les rôles Azure RBAC ;
* les scénarios PIM for Groups ;
* le rôle des Access Reviews ;
* la stratégie d'accès temporaire MonLabAzure ;
* les prérequis de licence ;
* les points de vigilance avant une implémentation réelle.

---

# Pourquoi Réduire les Droits Permanents ?

Un droit permanent est un droit disponible à tout moment.

Exemple :

```text
Compte administratif compromis
        │
        ▼
Droit Contributor déjà actif
        │
        ▼
Action malveillante immédiate
```

Plus un droit est permanent, plus il augmente la surface d'attaque.

Les risques sont :

* élévation de privilèges ;
* suppression de ressources ;
* création d'accès non autorisés ;
* désactivation de contrôles de sécurité ;
* exfiltration de données ;
* compromission de production.

La cible Enterprise est donc :

```text
Pas de privilège permanent sans justification.
```

---

# Le Principe Just-In-Time

Le modèle Just-In-Time consiste à accorder un privilège uniquement au moment où il est nécessaire.

Exemple :

```text
Administrateur
        │
        ▼
Rôle éligible
        │
        ▼
Demande d'activation
        │
        ▼
MFA + justification
        │
        ▼
Rôle actif pendant 1 heure
        │
        ▼
Expiration automatique
```

Ce modèle réduit fortement le risque.

Le droit existe, mais il n'est pas exploitable en permanence.

---

# Qu'est-ce que Microsoft Entra PIM ?

Microsoft Entra Privileged Identity Management, ou PIM, permet de gérer les accès privilégiés de manière contrôlée.

PIM permet notamment :

* de rendre un rôle éligible au lieu d'actif en permanence ;
* d'exiger une justification lors de l'activation ;
* d'exiger MFA ;
* de limiter la durée d'activation ;
* de notifier les administrateurs ;
* de revoir les accès privilégiés ;
* de tracer les activations.

Microsoft indique que PIM permet de limiter les accès administratifs permanents aux rôles privilégiés, de découvrir qui a accès et de revoir les accès privilégiés.

---

# Actif vs Éligible

## Rôle Actif

Un rôle actif est disponible immédiatement.

Exemple :

```text
adm-platform@monlabazure.fr
        │
        ▼
Global Administrator actif
```

Avantage :

* simple ;
* disponible immédiatement.

Risque :

* exploitable immédiatement en cas de compromission ;
* privilège permanent ;
* surface d'attaque élevée.

---

## Rôle Éligible

Un rôle éligible n'est pas actif par défaut.

L'utilisateur doit l'activer lorsqu'il en a besoin.

Exemple :

```text
adm-platform@monlabazure.fr
        │
        ▼
Global Administrator éligible
        │
        ▼
Activation PIM obligatoire
```

Avantage :

* réduction des droits permanents ;
* justification ;
* durée limitée ;
* meilleure traçabilité.

Microsoft documente l’activation des rôles éligibles dans PIM depuis l’expérience “My Microsoft Entra roles”, où l’utilisateur peut voir ses rôles éligibles et actifs.

---

# Les Trois Grands Scénarios PIM

Dans MonLabAzure, nous distinguons trois scénarios.

```text
PIM pour rôles Microsoft Entra ID
PIM pour rôles Azure RBAC
PIM for Groups
```

---

# PIM pour les Rôles Microsoft Entra ID

Ce scénario concerne les rôles d'administration du tenant.

Exemples :

```text
Global Administrator
Privileged Role Administrator
Security Administrator
Conditional Access Administrator
User Administrator
Groups Administrator
Application Administrator
```

Ces rôles permettent d'administrer Microsoft Entra ID.

Ils sont différents des rôles Azure RBAC utilisés sur les ressources Azure.

---

## Exemple

```text
adm-security@monlabazure.fr
        │
        ▼
Security Administrator éligible
        │
        ▼
Activation PIM pendant 2 heures
        │
        ▼
Travail d'administration sécurité
        │
        ▼
Expiration automatique
```

---

## Usage MonLabAzure

Dans MonLabAzure, PIM pour rôles Entra ID sera utilisé à terme pour :

* Security Administrator ;
* Conditional Access Administrator ;
* Groups Administrator ;
* Application Administrator ;
* Privileged Role Administrator.

Le rôle Global Administrator doit rester extrêmement limité.

---

# PIM pour les Rôles Azure RBAC

Ce scénario concerne les rôles sur les ressources Azure.

Exemples :

```text
Owner
Contributor
User Access Administrator
Network Contributor
Security Reader
Monitoring Contributor
```

PIM peut être utilisé sur des scopes Azure comme :

* Management Group ;
* Subscription ;
* Resource Group.

Microsoft documente l’activation des rôles Azure éligibles depuis la page Access control (IAM) sur les scopes Management Group, Subscription et Resource Group.

---

## Exemple

```text
adm-network@monlabazure.fr
        │
        ▼
Network Contributor éligible
        │
        ▼
Scope mla-connectivity
        │
        ▼
Activation pendant 1 heure
        │
        ▼
Modification réseau
        │
        ▼
Expiration automatique
```

---

## Usage MonLabAzure

Dans MonLabAzure, les rôles Azure RBAC à prioriser pour PIM sont :

```text
Owner
User Access Administrator
Contributor
Network Contributor
Security Admin
Monitoring Contributor
```

Les rôles de lecture comme `Reader` peuvent rester permanents selon le besoin.

---

# PIM for Groups

PIM for Groups permet de rendre temporaire l'appartenance à un groupe.

Exemple :

```text
adm-user@monlabazure.fr
        │
        ▼
Éligible au groupe AZR-MLA-PLATFORM-ADMINS
        │
        ▼
Activation temporaire du groupe
        │
        ▼
Héritage des rôles RBAC du groupe
```

Microsoft documente PIM for Groups comme une fonctionnalité permettant d’accorder une appartenance ou propriété de groupe Just-In-Time.

---

## Intérêt de PIM for Groups

PIM for Groups est particulièrement intéressant lorsque les rôles sont déjà affectés aux groupes.

C'est précisément notre modèle MonLabAzure :

```text
Utilisateur
        │
        ▼
Groupe Entra ID
        │
        ▼
Rôle Azure RBAC
        │
        ▼
Scope Azure
```

Avec PIM for Groups, on peut garder les rôles RBAC sur les groupes, mais rendre l'appartenance au groupe temporaire.

---

## Exemple MonLabAzure

```text
AZR-MLA-PLATFORM-ADMINS
        │
        ▼
Contributor sur mla-platform

adm-platform@monlabazure.fr
        │
        ▼
Éligible au groupe
        │
        ▼
Activation pendant 2 heures
```

Résultat :

L'administrateur ne devient membre actif du groupe que pendant la durée nécessaire.

---

# PIM et Comptes Break Glass

Les comptes Break Glass ne doivent pas dépendre de PIM.

Pourquoi ?

Parce que PIM est un mécanisme normal de gouvernance des privilèges.

Un compte Break Glass sert quand les mécanismes normaux peuvent être indisponibles ou bloquants.

Dans MonLabAzure :

```text
Comptes Break Glass
        │
        ▼
Global Administrator actif et permanent
        │
        ▼
Utilisation exceptionnelle uniquement
        │
        ▼
Surveillance critique
```

Ce choix est volontaire.

Il ne contredit pas le principe du moindre privilège, car les comptes Break Glass ne sont pas utilisés pour l'administration quotidienne.

---

# Paramètres Recommandés pour PIM

Pour les rôles sensibles, les paramètres PIM doivent être stricts.

Exemples :

```text
MFA obligatoire à l'activation
Justification obligatoire
Durée limitée
Notification aux administrateurs
Approbation pour les rôles critiques
Ticket ou référence de changement si possible
```

Microsoft documente les paramètres de rôles PIM permettant de configurer MFA, approbation, durée maximale d’activation et notifications.

---

# Durées d'Activation Recommandées

| Type de rôle                  |  Durée recommandée |
| ----------------------------- | -----------------: |
| Global Administrator          |       30 min à 1 h |
| Privileged Role Administrator |       30 min à 1 h |
| User Access Administrator     |       30 min à 1 h |
| Contributor PROD              |                1 h |
| Network Contributor           |          1 h à 2 h |
| Security Administrator        |          1 h à 2 h |
| Monitoring Contributor        |          2 h à 4 h |
| Reader                        | Permanent possible |

Ces durées sont indicatives.

Elles doivent être ajustées selon le contexte opérationnel.

---

# Access Reviews

PIM réduit les droits permanents.

Mais il faut aussi vérifier régulièrement qui possède encore un accès.

C'est le rôle des Access Reviews.

Les Access Reviews permettent de revoir périodiquement :

* l'appartenance à un groupe ;
* l'accès à une application ;
* les rôles Microsoft Entra ID ;
* les rôles Azure RBAC ;
* les comptes invités ;
* les accès privilégiés.

Microsoft décrit les Access Reviews comme un moyen de contrôler les appartenances aux groupes et les accès aux applications pour répondre aux besoins de gouvernance, risque et conformité.

---

# Pourquoi les Access Reviews sont Importantes

Les accès changent avec le temps.

Exemples :

* changement d'équipe ;
* fin de mission ;
* prestataire parti ;
* projet terminé ;
* compte administrateur oublié ;
* accès temporaire jamais supprimé ;
* groupe trop large.

Sans revue, les droits s'accumulent.

C'est un problème classique appelé :

```text
Privilege Creep
```

---

# Access Reviews pour Rôles Privilégiés

PIM peut être utilisé pour créer des Access Reviews sur les rôles Microsoft Entra ID et Azure Resource roles.

Microsoft indique que PIM permet de créer des Access Reviews pour les accès privilégiés aux rôles Azure Resources et Microsoft Entra, avec la possibilité de configurer des revues récurrentes.

---

## Exemple

```text
Revue trimestrielle
        │
        ▼
Rôle Contributor sur mla-platform
        │
        ▼
Valider si chaque membre a encore besoin de l'accès
```

---

# Access Reviews pour Groupes

Les groupes IAM de MonLabAzure devront être revus régulièrement.

Groupes prioritaires :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-OPERATIONS
AZR-MLA-DEVELOPERS
```

Fréquence recommandée :

```text
Trimestrielle pour groupes privilégiés
Semestrielle pour groupes non privilégiés
```

---

# Access Reviews pour PIM for Groups

Si PIM for Groups est utilisé, il faut aussi revoir :

* les membres actifs ;
* les membres éligibles ;
* les propriétaires de groupe.

Microsoft documente la création d’Access Reviews pour PIM for Groups, incluant les membres actifs et éligibles.

---

# Access Reviews pour Comptes Invités

Les comptes invités seront traités dans la série suivante.

Cependant, ils devront être intégrés à la stratégie Access Reviews.

Exemples :

```text
Prestataires
Partenaires
Comptes B2B
Consultants externes
```

Les invités doivent être revus plus fréquemment que les utilisateurs internes.

---

# Stratégie d'Accès Temporaire MonLabAzure

MonLabAzure adopte la stratégie suivante.

```text
Accès permanent
        │
        ▼
Réservé aux rôles de lecture ou besoins très stables

Accès éligible
        │
        ▼
Modèle cible pour les rôles privilégiés

Accès temporaire manuel
        │
        ▼
Accepté uniquement en transition

Accès Break Glass
        │
        ▼
Réservé aux incidents majeurs
```

---

# Modèle Cible

```text
Utilisateur standard
        │
        ▼
Aucun rôle privilégié

Compte administratif
        │
        ▼
Rôles éligibles via PIM

Groupe IAM
        │
        ▼
Affectation RBAC contrôlée

Compte Break Glass
        │
        ▼
Global Administrator permanent
        │
        ▼
Surveillance critique
```

---

# Scénario 1 – Élévation Temporaire Platform Admin

Besoin :

```text
Modifier une configuration plateforme.
```

Modèle cible :

```text
adm-platform@monlabazure.fr
        │
        ▼
Activation PIM
        │
        ▼
AZR-MLA-PLATFORM-ADMINS
        │
        ▼
Contributor sur mla-platform
        │
        ▼
Durée : 1 heure
```

Contrôles :

* MFA ;
* justification ;
* notification ;
* expiration automatique.

---

# Scénario 2 – Intervention Réseau

Besoin :

```text
Modifier une route ou un NSG.
```

Modèle cible :

```text
adm-network@monlabazure.fr
        │
        ▼
Activation PIM
        │
        ▼
AZR-MLA-NETWORK-ADMINS
        │
        ▼
Network Contributor sur mla-connectivity
        │
        ▼
Durée : 1 à 2 heures
```

Contrôles :

* MFA ;
* justification ;
* approbation si production ;
* traçabilité.

---

# Scénario 3 – Accès Production Exceptionnel

Besoin :

```text
Incident production.
```

Modèle cible :

```text
adm-operations@monlabazure.fr
        │
        ▼
Activation temporaire
        │
        ▼
Scope PROD limité
        │
        ▼
Durée courte
        │
        ▼
Revue post-incident
```

Règle :

```text
Pas de Contributor permanent en PROD.
```

---

# Scénario 4 – Break Glass

Besoin :

```text
Perte d'accès administratif standard.
```

Modèle :

```text
bg-admin-01@monlabazure.onmicrosoft.com
        │
        ▼
Global Administrator actif
        │
        ▼
Utilisation exceptionnelle
        │
        ▼
Alerte critique
        │
        ▼
Revue post-utilisation
```

Règle :

```text
Break Glass ne passe pas par PIM.
```

---

# Prérequis de Licence

PIM nécessite généralement Microsoft Entra ID P2.

Access Reviews peuvent nécessiter Microsoft Entra ID P2 ou Microsoft Entra ID Governance selon le scénario.

Microsoft indique que la gestion des Access Reviews requiert Microsoft Entra ID P2 ou Microsoft Entra ID Governance.

Dans MonLabAzure, cette série est donc positionnée comme :

```text
Fondation conceptuelle
+
Préparation Enterprise
+
Implémentation selon licences disponibles
```

---

# Ce que Nous Implémentons Maintenant

Dans le cadre du Lab, nous ne sommes pas obligés d'activer immédiatement PIM.

Nous allons d'abord :

* documenter le modèle cible ;
* identifier les rôles à rendre éligibles ;
* définir les durées d'activation ;
* préparer les revues d'accès ;
* garder les comptes Break Glass hors PIM ;
* prévoir l'implémentation lorsque la licence sera disponible.

---

# Ce que Nous N'Implémentons Pas Encore

Dans cet article, nous ne déployons pas encore :

* configuration complète PIM ;
* workflows d'approbation ;
* Access Reviews automatiques ;
* PIM for Groups en production ;
* intégration ITSM ;
* intégration Sentinel.

Ces éléments peuvent être ajoutés dans des séries avancées ou dans une phase Enterprise Ready.

---

# Matrice PIM Cible

| Groupe ou rôle          | Modèle actuel    | Modèle cible                |
| ----------------------- | ---------------- | --------------------------- |
| AZR-MLA-PLATFORM-ADMINS | Membre actif     | Éligible via PIM for Groups |
| AZR-MLA-NETWORK-ADMINS  | Membre actif     | Éligible via PIM for Groups |
| AZR-MLA-SECURITY-ADMINS | Membre actif     | Éligible via PIM for Groups |
| AZR-MLA-OPERATIONS      | Membre actif     | Éligible selon criticité    |
| AZR-MLA-DEVELOPERS      | Membre actif DEV | Revue régulière             |
| AZR-MLA-READERS         | Membre actif     | Permanent possible          |
| Global Administrator    | Très limité      | PIM sauf Break Glass        |
| Break Glass             | Actif permanent  | Hors PIM, surveillé         |

---

# Matrice Access Reviews Cible

| Périmètre       |                  Fréquence | Responsable             |
| --------------- | -------------------------: | ----------------------- |
| Platform Admins |              Trimestrielle | Architecture / Security |
| Network Admins  |              Trimestrielle | Network Lead            |
| Security Admins |              Trimestrielle | Security Lead           |
| Operations      |              Trimestrielle | Operations Lead         |
| Developers DEV  |               Semestrielle | Project Owner           |
| Accès PROD      | Mensuelle ou trimestrielle | Operations / Security   |
| Comptes invités | Mensuelle ou trimestrielle | Owner métier            |
| Break Glass     |              Trimestrielle | Security / Platform     |

---

# Règles d'Accès Temporaires

Toute demande d'accès temporaire doit préciser :

```text
Demandeur
Compte utilisé
Rôle demandé
Scope demandé
Justification
Durée
Approbateur
Date de début
Date de fin
Numéro de ticket ou changement
```

Sans ces informations, l'accès ne doit pas être accordé.

---

# Journal des Accès Temporaires

Exemple de journal :

| Date       | Demandeur      | Rôle                   | Scope            | Durée | Justification        | Approbateur     |
| ---------- | -------------- | ---------------------- | ---------------- | ----: | -------------------- | --------------- |
| 2026-06-08 | adm-network    | Network Contributor    | mla-connectivity |    2h | Modification route   | Platform Team   |
| 2026-06-08 | adm-operations | Monitoring Contributor | mla-management   |    4h | Incident supervision | Operations Lead |

---

# Points d'Audit

Les contrôles suivants doivent être réalisés régulièrement :

```text
Qui possède des rôles actifs ?
Qui possède des rôles éligibles ?
Quels rôles ont été activés ?
Quelle était la justification ?
Combien de temps l'accès est resté actif ?
Quels accès sont permanents ?
Quels accès n'ont pas été revus ?
Quels comptes invités ont encore accès ?
Les comptes Break Glass ont-ils été utilisés ?
```

---

# Points de Vigilance

## Ne Pas Mettre Break Glass dans PIM

Les comptes Break Glass doivent rester utilisables même si les mécanismes normaux sont indisponibles.

## Ne Pas Tout Basculer en PIM Sans Test

Commencer par un groupe pilote.

## Ne Pas Garder Trop de Rôles Actifs

Les rôles privilégiés doivent devenir éligibles lorsque possible.

## Ne Pas Ignorer les Groupes

PIM for Groups peut être plus cohérent avec notre modèle RBAC basé sur les groupes.

## Ne Pas Oublier les Revues

PIM sans Access Reviews reste incomplet.

---

# Dépannage

## L'Utilisateur ne Voit Pas Son Rôle Éligible

Causes possibles :

* rôle non assigné comme éligible ;
* mauvais tenant ;
* mauvais compte ;
* délai de propagation ;
* licence manquante.

---

## L'Activation PIM Échoue

Causes possibles :

* MFA non configuré ;
* approbation requise non validée ;
* durée demandée supérieure à la durée autorisée ;
* condition d'activation non satisfaite.

---

## L'Access Review ne Couvre Pas le Bon Scope

Causes possibles :

* mauvaise sélection du groupe ;
* mauvais rôle ;
* mauvais scope Azure ;
* revue configurée sur une application au lieu d'un rôle ;
* propriétaires de groupe non définis.

---

# Coûts et Licences

PIM et Access Reviews sont des fonctionnalités avancées.

Elles peuvent nécessiter :

```text
Microsoft Entra ID P2
Microsoft Entra ID Governance
Enterprise Mobility + Security E5 selon contexte
```

Dans un Lab personnel, il faut vérifier les licences disponibles avant de commencer.

Dans MonLabAzure, l'article sert d'abord à définir la cible d'architecture.

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* compris le rôle de PIM ;
* distingué rôle actif et rôle éligible ;
* défini l'intérêt des accès Just-In-Time ;
* identifié les scénarios PIM pour Entra ID, Azure RBAC et Groups ;
* positionné les comptes Break Glass hors PIM ;
* défini une matrice PIM cible ;
* défini une matrice Access Reviews cible ;
* cadré les accès temporaires ;
* préparé la gouvernance avancée des accès.

---

# Conclusion

PIM, Access Reviews et les accès temporaires permettent de faire évoluer MonLabAzure vers un modèle IAM plus mature.

Le modèle initial RBAC reste nécessaire.

Mais il ne suffit pas dans une architecture Enterprise.

La trajectoire cible est claire :

```text
RBAC via groupes
        │
        ▼
MFA et Conditional Access
        │
        ▼
PIM pour réduire les privilèges permanents
        │
        ▼
Access Reviews pour supprimer les accès obsolètes
        │
        ▼
Accès temporaires documentés et audités
```

Les comptes Break Glass restent volontairement à part.

Ils ne sont pas des comptes d'administration quotidienne.

Ils constituent un mécanisme d'urgence contrôlé, surveillé et testé.

Dans le prochain article, nous traiterons les comptes invités et la gouvernance B2B.
