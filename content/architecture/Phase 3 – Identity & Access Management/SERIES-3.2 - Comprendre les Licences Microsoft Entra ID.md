---

title: "SERIES-3.2 - Comprendre les Licences Microsoft Entra ID"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "Microsoft Entra ID"
* "Licensing"
* "P1"
* "P2"
* "Governance"
* "Security"
* "Azure"

---

# SERIES-3.2 – Comprendre les Licences Microsoft Entra ID

## Introduction

Dans l'article précédent, nous avons introduit la **Phase 3 – Identity & Access Management** du projet MonLabAzure Cloud Platform.

Nous avons vu que l'identité est un pilier central d'une plateforme Azure moderne.

Avant de créer les groupes Microsoft Entra ID, de définir les rôles RBAC ou de parler de Conditional Access, PIM ou Access Reviews, il est indispensable de comprendre un point souvent négligé :

```text
Toutes les fonctionnalités IAM ne sont pas disponibles avec les mêmes licences.
```

Dans Azure, certaines fonctionnalités de base sont disponibles par défaut.

D'autres nécessitent Microsoft Entra ID P1, P2 ou Microsoft Entra ID Governance.

Cette distinction est importante pour trois raisons :

* éviter de concevoir une architecture impossible à appliquer ;
* comprendre les écarts entre un Lab personnel et un environnement Enterprise ;
* planifier correctement la montée en maturité IAM.

Cet article présente donc les principales licences Microsoft Entra ID et leur impact sur le projet MonLabAzure.

---

# Positionnement dans le Projet

```text
Phase 3 – Identity & Access Management

ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride ✅

SERIES-3.1 – Introduction à Identity & Access Management dans Azure ✅
SERIES-3.2 – Comprendre les licences Microsoft Entra ID ⏳
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

# Pourquoi Parler des Licences Maintenant ?

Dans un projet IAM, la tentation est de commencer directement par la création des groupes et des rôles.

Cependant, certaines décisions d'architecture dépendent fortement des licences disponibles.

Exemples :

```text
Conditional Access       → nécessite Entra ID P1 ou supérieur
Groupes dynamiques       → nécessite Entra ID P1 ou supérieur
PIM                      → nécessite Entra ID P2
Identity Protection      → nécessite Entra ID P2
Access Reviews           → nécessite Entra ID P2 ou Entra ID Governance selon le scénario
Entitlement Management   → relève d'Entra ID Governance
Lifecycle Workflows      → relève d'Entra ID Governance
```

Sans cette compréhension, on risque de concevoir une architecture théorique mais non déployable dans un Lab ou chez un client.

L'objectif n'est donc pas de choisir la licence la plus chère.

L'objectif est de comprendre :

* ce qui est nécessaire ;
* ce qui est utile ;
* ce qui est optionnel ;
* ce qui relève d'un environnement Enterprise mature.

---

# Microsoft Entra ID : Rappel

Microsoft Entra ID est le service d'identité cloud utilisé par Azure et les services Microsoft.

Il permet notamment de gérer :

* les utilisateurs ;
* les groupes ;
* les applications ;
* les Service Principals ;
* les rôles administratifs ;
* les accès aux ressources ;
* les journaux de connexion ;
* les accès conditionnels ;
* les scénarios hybrides.

Historiquement, ce service était connu sous le nom Azure Active Directory.

Aujourd'hui, il fait partie de la famille Microsoft Entra.

---

# Les Grandes Familles de Licences

Pour simplifier, nous allons retenir cinq niveaux principaux :

```text
Microsoft Entra ID Free
Microsoft Entra ID P1
Microsoft Entra ID P2
Microsoft Entra ID Governance
Microsoft Entra Suite
```

Chaque niveau apporte des fonctionnalités différentes.

---

# Microsoft Entra ID Free

## Présentation

Microsoft Entra ID Free fournit les fonctionnalités d'identité de base.

Il est généralement inclus avec les abonnements cloud Microsoft comme Azure ou Microsoft 365.

Il permet de démarrer un tenant, de créer des utilisateurs, de gérer des groupes et d'utiliser les bases de l'authentification cloud.

---

## Fonctionnalités Typiques

Avec Entra ID Free, on peut généralement travailler sur :

* les utilisateurs ;
* les groupes de base ;
* les rôles administratifs de base ;
* l'authentification cloud ;
* l'accès aux services Azure ;
* les premiers journaux ;
* les affectations RBAC Azure via groupes.

Dans le contexte MonLabAzure, cela suffit pour démarrer les premiers sujets IAM.

---

## Ce que l'on Peut Faire dans MonLabAzure

Avec Entra ID Free, nous pouvons déjà construire une première base :

```text
Créer des utilisateurs
Créer des groupes
Affecter des rôles RBAC Azure
Séparer les comptes standards et administratifs
Préparer les comptes Break Glass
Utiliser des groupes pour les affectations RBAC
```

Cela permet de réaliser une partie importante de l'IAM Foundation.

---

## Limites

Entra ID Free ne suffit pas pour les scénarios IAM avancés.

Limites importantes :

* pas de Conditional Access avancé ;
* pas de groupes dynamiques ;
* pas de PIM ;
* pas d'Access Reviews avancées ;
* pas d'Identity Protection ;
* gouvernance des accès limitée.

Ce niveau est donc suffisant pour apprendre les bases, mais insuffisant pour une stratégie Enterprise complète.

---

# Microsoft Entra ID P1

## Présentation

Microsoft Entra ID P1 est le premier niveau réellement adapté à un environnement professionnel structuré.

Il ajoute des capacités importantes pour contrôler les accès et automatiser une partie de la gestion des identités.

---

## Fonctionnalités Importantes

Entra ID P1 apporte notamment :

* Conditional Access ;
* groupes dynamiques ;
* fonctionnalités avancées de gestion des groupes ;
* Self-Service Password Reset avancé ;
* fonctionnalités hybrides avancées selon les scénarios ;
* contrôles d'accès plus fins.

---

# Conditional Access

Conditional Access est l'un des apports majeurs d'Entra ID P1.

Il permet de contrôler l'accès en fonction de plusieurs signaux.

Exemples de signaux :

* utilisateur ;
* groupe ;
* application ;
* emplacement réseau ;
* appareil ;
* niveau de risque selon les licences ;
* état de conformité ;
* méthode d'authentification.

Exemple de logique :

```text
Si l'utilisateur est administrateur
ET qu'il accède au portail Azure
ALORS exiger MFA
```

Conditional Access devient rapidement indispensable dans une plateforme Azure sérieuse.

---

# Groupes Dynamiques

Les groupes dynamiques permettent d'ajouter automatiquement des utilisateurs ou des appareils à un groupe selon des règles.

Exemple :

```text
Si department = Cloud
Alors ajouter l'utilisateur au groupe AZR-MLA-CLOUD-USERS
```

Autre exemple :

```text
Si jobTitle contient Developer
Alors ajouter l'utilisateur au groupe AZR-MLA-DEVELOPERS
```

Cela permet d'automatiser une partie du cycle de vie des accès.

Dans MonLabAzure, nous ne commencerons pas directement par les groupes dynamiques.

Nous créerons d'abord les groupes manuellement afin de bien comprendre le modèle IAM.

Les groupes dynamiques seront étudiés plus tard comme évolution d'automatisation.

---

## Ce que P1 Apporte à MonLabAzure

Avec Entra ID P1, MonLabAzure pourra aborder :

* Conditional Access Foundation ;
* MFA ciblé sur les administrateurs ;
* protection renforcée du portail Azure ;
* blocage progressif de l'authentification legacy ;
* groupes dynamiques ;
* automatisation simple de l'appartenance aux groupes.

P1 est donc le niveau minimal pour commencer à parler sérieusement de contrôle d'accès conditionnel.

---

# Microsoft Entra ID P2

## Présentation

Microsoft Entra ID P2 inclut les capacités de P1 et ajoute des fonctionnalités avancées pour la sécurité, les risques et les accès privilégiés.

C'est un niveau généralement attendu dans les environnements Enterprise matures.

---

## Fonctionnalités Importantes

Entra ID P2 apporte notamment :

* Privileged Identity Management ;
* Identity Protection ;
* accès basé sur le risque ;
* détection des utilisateurs à risque ;
* détection des connexions à risque ;
* capacités avancées de gouvernance des accès selon les scénarios.

---

# Privileged Identity Management

PIM permet de réduire les privilèges permanents.

Au lieu d'accorder un rôle administrateur de manière permanente, l'utilisateur devient éligible.

Il active ensuite le rôle uniquement lorsqu'il en a besoin.

Exemple :

```text
Utilisateur administrateur
        │
        ▼
Rôle éligible
        │
        ▼
Activation temporaire
        │
        ▼
Justification
        │
        ▼
Expiration automatique
```

Cela permet de réduire fortement la surface d'attaque liée aux comptes privilégiés.

Dans MonLabAzure, PIM sera abordé après les bases RBAC et les comptes administratifs.

---

# Identity Protection

Identity Protection permet de détecter certains risques liés aux utilisateurs et aux connexions.

Exemples :

* utilisateur à risque ;
* connexion depuis un emplacement inhabituel ;
* fuite potentielle d'identifiants ;
* comportement de connexion suspect.

Ces signaux peuvent ensuite être utilisés dans des stratégies de sécurité avancées.

---

## Ce que P2 Apporte à MonLabAzure

Avec Entra ID P2, MonLabAzure pourra aborder :

* PIM ;
* accès administratifs temporaires ;
* réduction des droits permanents ;
* Identity Protection ;
* politiques basées sur le risque ;
* contrôles avancés des comptes privilégiés.

P2 correspond au niveau cible pour une plateforme Enterprise plus mature.

---

# Microsoft Entra ID Governance

## Présentation

Microsoft Entra ID Governance est orienté gouvernance du cycle de vie des identités et des accès.

Son objectif est de répondre à une question plus large que le simple RBAC :

```text
Qui doit avoir accès à quoi, pourquoi, pendant combien de temps, et avec quelle validation ?
```

---

## Fonctionnalités Importantes

Microsoft Entra ID Governance couvre notamment :

* Access Reviews ;
* Entitlement Management ;
* Access Packages ;
* Lifecycle Workflows ;
* gouvernance des invités ;
* demandes d'accès ;
* processus d'approbation ;
* automatisation de l'onboarding et de l'offboarding selon les scénarios.

---

# Access Reviews

Les Access Reviews permettent de revoir périodiquement les accès accordés aux utilisateurs.

Elles sont utiles pour :

* les groupes sensibles ;
* les accès à la production ;
* les accès invités ;
* les rôles privilégiés ;
* les applications critiques.

Exemple :

```text
Tous les trimestres
        │
        ▼
Revue du groupe AZR-MLA-PROD-OPERATORS
        │
        ▼
Validation ou suppression des membres
```

Dans un environnement Enterprise, les Access Reviews sont très importantes pour les audits et la conformité.

---

# Entitlement Management

Entitlement Management permet de gérer des packages d'accès.

Un package d'accès peut regrouper :

* groupes ;
* applications ;
* rôles ;
* ressources ;
* règles d'approbation ;
* durée d'accès.

Exemple :

```text
Package "Prestataire Projet X"
        │
        ├── Accès application
        ├── Groupe Teams
        ├── Groupe Entra ID
        └── Expiration automatique après 30 jours
```

Ce type de fonctionnalité devient pertinent lorsque l'organisation gère de nombreux accès temporaires ou externes.

---

# Lifecycle Workflows

Lifecycle Workflows permet d'automatiser certaines étapes du cycle de vie des identités.

Exemples :

* arrivée d'un collaborateur ;
* changement de poste ;
* départ d'un utilisateur ;
* suppression ou désactivation d'accès ;
* notification des équipes concernées.

Dans MonLabAzure, ce sujet sera positionné comme évolution avancée.

---

## Ce que Governance Apporte à MonLabAzure

Microsoft Entra ID Governance sera utile lorsque nous traiterons :

* les comptes invités ;
* les accès temporaires ;
* les workflows d'approbation ;
* les revues régulières ;
* les accès applicatifs ;
* les scénarios de prestataires.

Ce niveau dépasse l'IAM Foundation.

Il correspond davantage à une gouvernance Enterprise avancée.

---

# Microsoft Entra Suite

## Présentation

Microsoft Entra Suite regroupe plusieurs capacités avancées autour de l'identité, de la gouvernance, de l'accès réseau et du Zero Trust.

Elle ne se limite pas à Microsoft Entra ID.

Elle couvre une vision plus globale de l'accès sécurisé.

---

## Positionnement

Microsoft Entra Suite peut devenir pertinente pour les organisations qui veulent aller plus loin sur :

* l'accès sécurisé aux applications privées ;
* l'accès Internet sécurisé ;
* la gouvernance d'identité ;
* la vérification d'identité ;
* l'approche Zero Trust étendue.

Dans MonLabAzure, Microsoft Entra Suite ne sera pas nécessaire pour la première version de la plateforme.

Elle sera simplement positionnée comme une évolution possible si le projet couvre plus tard des scénarios Zero Trust plus avancés.

---

# Tableau de Synthèse

| Fonctionnalité                       |   Free |  P1 |                   P2 |             Governance |
| ------------------------------------ | -----: | --: | -------------------: | ---------------------: |
| Utilisateurs                         |    Oui | Oui |                  Oui |                    Oui |
| Groupes de base                      |    Oui | Oui |                  Oui |                    Oui |
| RBAC Azure via groupes               |    Oui | Oui |                  Oui |                    Oui |
| Comptes administratifs séparés       |    Oui | Oui |                  Oui |                    Oui |
| Comptes Break Glass                  |    Oui | Oui |                  Oui |                    Oui |
| Groupes dynamiques                   |    Non | Oui |                  Oui |                    Oui |
| Conditional Access                   |    Non | Oui |                  Oui |                    Oui |
| Blocage avancé legacy authentication | Limité | Oui |                  Oui |                    Oui |
| PIM                                  |    Non | Non |                  Oui | Selon licence/scénario |
| Identity Protection                  |    Non | Non |                  Oui | Selon licence/scénario |
| Access Reviews                       |    Non | Non | Oui / selon scénario |                    Oui |
| Entitlement Management               |    Non | Non |       Selon scénario |                    Oui |
| Lifecycle Workflows                  |    Non | Non |       Selon scénario |                    Oui |

---

# Lecture Architecte

Le tableau précédent doit être lu avec prudence.

Il ne faut pas raisonner uniquement en fonctionnalités.

Il faut raisonner en scénarios.

---

## Scénario 1 – Lab IAM Foundation

Objectif :

```text
Comprendre les bases IAM Azure.
```

Fonctionnalités nécessaires :

* utilisateurs ;
* groupes ;
* Azure RBAC ;
* comptes administratifs ;
* comptes Break Glass ;
* Service Principals ;
* Managed Identities.

Licence minimale :

```text
Microsoft Entra ID Free
```

Ce scénario est compatible avec le démarrage de MonLabAzure.

---

## Scénario 2 – Lab Sécurité IAM Intermédiaire

Objectif :

```text
Renforcer les accès et commencer le contrôle conditionnel.
```

Fonctionnalités utiles :

* Conditional Access ;
* groupes dynamiques ;
* blocage de l'authentification legacy ;
* MFA ciblé ;
* politiques d'accès administrateur.

Licence cible :

```text
Microsoft Entra ID P1
```

Ce scénario est adapté à une plateforme plus proche d'un environnement professionnel.

---

## Scénario 3 – Enterprise Privileged Access

Objectif :

```text
Réduire les droits permanents et sécuriser les comptes privilégiés.
```

Fonctionnalités utiles :

* PIM ;
* Identity Protection ;
* accès basés sur le risque ;
* activation temporaire des rôles ;
* justification des élévations.

Licence cible :

```text
Microsoft Entra ID P2
```

Ce scénario correspond à un environnement Enterprise mature.

---

## Scénario 4 – Enterprise Identity Governance

Objectif :

```text
Industrialiser la gouvernance des accès.
```

Fonctionnalités utiles :

* Access Reviews ;
* Entitlement Management ;
* Access Packages ;
* Lifecycle Workflows ;
* gouvernance des invités ;
* workflows d'approbation.

Licence cible :

```text
Microsoft Entra ID Governance
```

Ce scénario concerne les organisations avec de nombreux utilisateurs, prestataires, applications et accès temporaires.

---

# Positionnement pour MonLabAzure

Pour garder une approche réaliste, MonLabAzure ne va pas imposer toutes les fonctionnalités avancées dès le début.

La progression retenue est la suivante :

```text
Étape 1 – IAM Foundation
        │
        ├── Utilisateurs
        ├── Groupes Entra ID
        ├── Azure RBAC
        ├── Comptes administratifs
        ├── Comptes Break Glass
        ├── Service Principals
        └── Managed Identities

Étape 2 – IAM Security Foundation
        │
        ├── MFA
        ├── Conditional Access
        ├── Blocage legacy authentication
        └── Groupes dynamiques

Étape 3 – Privileged Access
        │
        ├── PIM
        ├── Accès temporaires
        ├── Identity Protection
        └── Réduction des privilèges permanents

Étape 4 – Identity Governance
        │
        ├── Access Reviews
        ├── Gouvernance des invités
        ├── Entitlement Management
        └── Lifecycle Workflows
```

Cette progression permet de construire une base solide sans rendre le Lab dépendant immédiatement de licences avancées.

---

# Ce que Nous Allons Faire dans la Phase 3

Dans la Phase 3, nous allons distinguer deux catégories.

---

## Les Fondations à Implémenter

Ces éléments seront traités comme base IAM :

* groupes Microsoft Entra ID ;
* modèle RBAC ;
* affectations RBAC ;
* comptes administratifs ;
* comptes Break Glass ;
* Service Principals ;
* Managed Identities ;
* logs Entra ID.

Ces sujets restent compatibles avec une approche Lab.

---

## Les Fonctionnalités à Étudier ou Préparer

Ces éléments seront présentés, mais pas forcément implémentés immédiatement :

* Conditional Access ;
* groupes dynamiques ;
* PIM ;
* Access Reviews ;
* Identity Protection ;
* gouvernance avancée des invités ;
* Entitlement Management ;
* Lifecycle Workflows.

Ils seront traités selon les licences disponibles et la maturité du Lab.

---

# Vigilance sur les Coûts

Certaines licences Microsoft Entra ID peuvent représenter un coût significatif selon le nombre d'utilisateurs.

Dans un Lab personnel, il faut donc être prudent.

Bonnes pratiques :

* vérifier les licences disponibles avant de déployer ;
* utiliser les essais ou environnements de test lorsque possible ;
* éviter d'imposer P2 ou Governance dès le début ;
* documenter clairement les prérequis ;
* séparer les exercices obligatoires des exercices optionnels.

Dans MonLabAzure, chaque article concerné précisera les prérequis de licence.

---

# Erreurs Fréquentes

## Concevoir une architecture P2 sans licence P2

C'est une erreur fréquente.

Exemple :

```text
Prévoir PIM dans l'architecture
mais disposer uniquement d'Entra ID Free.
```

La conséquence est simple : l'architecture ne peut pas être appliquée.

---

## Ignorer les licences dans les dossiers d'architecture

Un HLD ou un ADR IAM doit toujours mentionner les dépendances de licence.

Sinon, l'architecture semble complète sur le papier, mais incomplète en réalité.

---

## Confondre RBAC Azure et rôles Entra ID

Azure RBAC contrôle l'accès aux ressources Azure.

Les rôles Microsoft Entra ID contrôlent l'administration du tenant et des services d'identité.

Exemple :

```text
Owner sur une subscription Azure
≠
Global Administrator dans Microsoft Entra ID
```

Les deux modèles sont liés, mais ils ne sont pas équivalents.

---

## Activer trop tôt les fonctionnalités avancées

PIM, Conditional Access, Access Reviews et groupes dynamiques sont importants.

Mais ils doivent être introduits progressivement.

Une mise en place trop rapide sans modèle IAM clair peut créer de la complexité et des erreurs.

---

# Synthèse

Pour MonLabAzure, nous retenons la lecture suivante :

```text
Free
    → base IAM, utilisateurs, groupes, RBAC, Break Glass

P1
    → Conditional Access, groupes dynamiques, sécurité d'accès renforcée

P2
    → PIM, Identity Protection, accès privilégiés avancés

Governance
    → Access Reviews avancées, Entitlement Management, Lifecycle Workflows

Suite
    → approche Zero Trust plus large avec accès réseau et gouvernance avancée
```

Cette lecture nous permet de construire une roadmap IAM réaliste.

---

# Livrables de cet Article

À ce stade, nous avons :

* identifié les principales licences Microsoft Entra ID ;
* compris l'impact des licences sur l'architecture IAM ;
* distingué les fonctionnalités Lab et Enterprise ;
* préparé les futurs articles sur Conditional Access, PIM, Access Reviews et groupes dynamiques ;
* confirmé que la première partie de la Phase 3 peut commencer sans dépendre immédiatement d'Entra ID P2.

---

# Conclusion

Les licences Microsoft Entra ID ne sont pas un détail administratif.

Elles influencent directement les choix d'architecture IAM.

Pour MonLabAzure Cloud Platform, l'approche retenue est progressive.

Nous allons commencer par les fondations :

* groupes ;
* RBAC ;
* comptes administratifs ;
* Break Glass ;
* identités techniques.

Puis nous intégrerons progressivement les fonctionnalités avancées selon leur pertinence et leurs prérequis de licence.

Dans le prochain article, nous aborderons l'identité hybride avec Microsoft Entra Connect Sync, Microsoft Entra Cloud Sync et les différents modes d'authentification.