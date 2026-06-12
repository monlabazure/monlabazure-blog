# SERIES-1.1 - Introduction à la Landing Zone Foundation

## Introduction

Après avoir défini la vision du projet, les décisions d'architecture (ADR) et l'architecture cible de référence, nous entrons maintenant dans la première phase d'implémentation de MonLabAzure Cloud Platform.

Cette phase marque le passage entre la conception et la réalisation.

L'objectif n'est pas encore de déployer des applications, des bases de données ou des clusters Kubernetes.

Avant cela, nous devons construire les fondations de la plateforme Azure.

Dans un projet d'entreprise, cette étape est généralement appelée :

**Landing Zone Foundation**

Elle constitue le socle sur lequel l'ensemble des futures ressources seront déployées.

---

# 1. Pourquoi commencer par une Landing Zone ?

Une erreur fréquente consiste à commencer directement par créer :

* un Resource Group ;
* un réseau virtuel ;
* un App Service ;
* une base de données.

Cette approche fonctionne dans un laboratoire ponctuel mais devient rapidement difficile à maintenir lorsqu'une plateforme évolue.

Dans un environnement professionnel, les équipes commencent généralement par mettre en place :

* la gouvernance ;
* les abonnements ;
* les contrôles de sécurité ;
* les conventions ;
* les mécanismes de supervision.

La Landing Zone fournit ce cadre.

---

# 2. Qu'est-ce qu'une Landing Zone Azure ?

Une Landing Zone Azure est un ensemble de composants permettant d'accueillir les futures charges de travail dans un environnement :

* sécurisé ;
* gouverné ;
* supervisé ;
* évolutif.

Elle définit notamment :

* l'organisation des abonnements ;
* les Management Groups ;
* les Azure Policies ;
* les accès RBAC ;
* les conventions de nommage ;
* les règles de sécurité ;
* les principes FinOps.

Dans notre projet, ces éléments ont déjà été définis dans les ADR précédents.

Nous allons maintenant les transformer en composants réels dans Azure.

---

# 3. Positionnement dans le projet

Nous reprenons l'architecture cible définie dans SERIES-000.

```text
Microsoft Entra ID Tenant
│
└── Tenant Root Group
    │
    ├── Platform
    │   ├── Connectivity
    │   └── Management
    │
    ├── LandingZones
    │   ├── DEV
    │   ├── QUALIF
    │   └── PROD
    │
    └── Sandbox
```

À ce stade, aucune charge de travail métier n'est encore déployée.

Nous construisons uniquement la structure de gouvernance.

---

# 4. Objectifs de la phase Foundation

La phase Foundation doit permettre de mettre en place :

## Gouvernance

* Structure des Management Groups.
* Organisation des abonnements.
* Hiérarchie Azure.

---

## Contrôle

* RBAC.
* Groupes Microsoft Entra ID.
* Azure Policy.

---

## Automatisation

* Terraform.
* Backend Terraform.
* Gestion des états.

---

## Exploitation

* Journalisation.
* Supervision.
* Audit.

---

# 5. Livrables attendus

À l'issue de cette phase, nous devrons disposer :

### Gouvernance

* Tenant structuré.
* Management Groups créés.
* Abonnements rattachés.

### Sécurité

* Premiers groupes Entra ID.
* Principes RBAC définis.

### Terraform

* Backend distant opérationnel.
* Structure Git opérationnelle.
* Premier code Terraform validé.

### Conformité

* Conventions de nommage appliquées.
* Stratégie de tags appliquée.

---

# 6. Ce que nous ne ferons pas encore

Pour éviter de brûler les étapes, les éléments suivants ne seront pas déployés immédiatement :

* Azure Firewall
* Azure Bastion
* Hub Network
* App Service
* Azure SQL
* Key Vault
* AKS
* API Management

Ces composants seront introduits progressivement dans les séries futures.

---

# 7. Architecture cible à l'issue de la phase Foundation

L'objectif est d'obtenir la structure suivante :

```text
Tenant Root Group
│
├── Platform
│   ├── Connectivity Subscription
│   └── Management Subscription
│
├── LandingZones
│   ├── DEV Subscription
│   ├── QUALIF Subscription
│   └── PROD Subscription
│
└── Sandbox
```

Cette structure servira de base à toutes les séries suivantes.

---

# 8. Prérequis

Avant de poursuivre les prochaines étapes de la série, les éléments suivants doivent être disponibles :

## Azure

* Tenant Azure actif.
* Droits suffisants sur le tenant.
* Accès aux abonnements.

## Poste de travail

* Terraform installé.
* Azure CLI installé.
* Git installé.

## Documentation

* INTRO-001
* ADR-001 à ADR-005
* SERIES-000

---

# 9. Validation

À la fin de cet article, aucun déploiement n'est encore réalisé.

L'objectif est uniquement de comprendre :

* le rôle de la Landing Zone ;
* le périmètre de la phase Foundation ;
* les composants qui seront construits.

La validation réelle commencera dans les prochains articles.

---

# 10. Coûts

Aucun coût Azure significatif n'est généré à ce stade.

Les premières dépenses apparaîtront lors de la création des ressources de gouvernance et de supervision.

---

# 11. Conclusion

La Landing Zone Foundation constitue la première étape concrète de construction de MonLabAzure Cloud Platform.

Avant d'héberger des applications ou de déployer des services avancés, nous devons disposer d'un environnement gouverné, sécurisé et automatisé.

Dans le prochain article, nous préparerons le tenant Azure et vérifierons les prérequis nécessaires à la mise en œuvre de notre architecture cible.
