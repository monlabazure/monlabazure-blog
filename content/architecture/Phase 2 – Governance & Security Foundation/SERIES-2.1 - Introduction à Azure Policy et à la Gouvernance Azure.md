---

title: "SERIES-2.1 - Introduction à Azure Policy et à la Gouvernance Azure"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-2"
* "Governance"
* "Azure Policy"
* "Landing Zone"
* "Compliance"
* "Terraform"
* "Azure"

---

# SERIES-2.1 – Introduction à Azure Policy et à la Gouvernance Azure

## Introduction

Lors de la Phase 1, nous avons construit les fondations de notre plateforme Azure :

* Backend Terraform
* Management Groups
* Structure des abonnements
* Gouvernance initiale

Nous disposons désormais d'une Landing Zone fonctionnelle.

Cependant, une question importante demeure :

> Comment s'assurer que les futurs déploiements respectent les standards définis pour la plateforme ?

C'est précisément le rôle d'Azure Policy.

Cette phase marque le passage d'une plateforme organisée à une plateforme gouvernée.

---

# Positionnement dans le Projet

```text
Phase 2 – Governance & Security Foundation

ADR-006   Stratégie Azure Policy et Gouvernance      ✅

SERIES-2.1   Introduction Azure Policy              ⏳
SERIES-2.2   Politique de Tags Obligatoires
SERIES-2.3   Politique de Régions Autorisées
SERIES-2.4   Diagnostic Settings
SERIES-2.5   Contrôle des Ressources Publiques
SERIES-2.6   Initiative MonLabAzure Governance
SERIES-2.7   Validation Governance Foundation
```

---

# Pourquoi Azure Policy ?

Dans un environnement Azure, les équipes disposent souvent de droits leur permettant de créer des ressources.

Sans mécanisme de contrôle, cela peut conduire à :

* des ressources sans tags ;
* des régions non autorisées ;
* des ressources non supervisées ;
* des configurations non conformes ;
* une augmentation des coûts.

Azure Policy permet d'automatiser les contrôles de conformité.

---

# Qu'est-ce qu'Azure Policy ?

Azure Policy est un service natif Azure permettant :

* d'évaluer les ressources ;
* de contrôler leur conformité ;
* d'appliquer des règles ;
* d'automatiser certaines configurations.

Azure Policy agit comme un garde-fou de la plateforme.

---

# Fonctionnement Général

Lorsqu'une ressource est créée ou modifiée :

```text
Utilisateur
      │
      ▼
Déploiement Azure
      │
      ▼
Azure Policy
      │
 ┌────┴────┐
 │         │
 ▼         ▼
Autorisé   Refusé
```

Chaque ressource est évaluée selon les règles définies.

---

# Les Modes d'Application

Azure Policy propose plusieurs modes.

---

## Audit

Le déploiement est autorisé.

La ressource est signalée comme non conforme.

Utilisation recommandée :

* phase de découverte ;
* migration ;
* analyse des écarts.

---

## Deny

Le déploiement est bloqué.

Utilisation recommandée :

* standards obligatoires ;
* gouvernance mature.

---

## Append

Ajoute automatiquement certaines propriétés lors du déploiement.

---

## Modify

Modifie automatiquement certains paramètres.

---

## DeployIfNotExists

Déploie automatiquement des ressources complémentaires lorsqu'elles sont absentes.

Exemple :

* Diagnostic Settings
* Extensions VM
* Agents de supervision

---

# Hiérarchie de Gouvernance

Grâce aux Management Groups créés dans la Phase 1, nous pouvons appliquer les Policies à différents niveaux.

Exemple :

```text
mla-landingzones
│
├── mla-dev
├── mla-qualif
└── mla-prod
```

Une Policy affectée à :

```text
mla-landingzones
```

sera automatiquement héritée par :

* DEV
* QUALIF
* PROD

Cette approche simplifie considérablement la gouvernance.

---

# Stratégie MonLabAzure

Conformément à ADR-006, notre stratégie repose sur quatre principes.

---

## Gouvernance Centralisée

Les Policies seront affectées au niveau des Management Groups.

---

## Déploiement Terraform

Toutes les Policies seront gérées avec Terraform.

Aucune création manuelle dans le portail Azure.

---

## Audit Avant Deny

Nous commencerons systématiquement par :

```text
Audit
```

avant d'envisager :

```text
Deny
```

---

## Initiative Unique

Les Policies seront regroupées dans une initiative dédiée :

```text
MonLabAzure Governance Baseline
```

---

# Policies Prioritaires

Les premières Policies qui seront déployées dans cette phase sont :

## Tags Obligatoires

Contrôle de la présence des tags :

* Project
* Environment
* Owner
* CostCenter
* Criticality
* ManagedBy
* DataClassification
* BackupRequired

---

## Régions Autorisées

Limitation aux régions :

* West Europe
* North Europe

---

## Diagnostic Settings

Vérification de la journalisation.

---

## Ressources Publiques

Contrôle des expositions réseau non autorisées.

---

# Pourquoi cette Phase est Importante

Les Management Groups créés précédemment organisent la plateforme.

Azure Policy va maintenant imposer les règles.

Sans Policy :

```text
Organisation
```

Avec Policy :

```text
Organisation
+
Conformité
+
Contrôle
```

C'est une étape majeure dans la construction d'une Landing Zone Enterprise.

---

# Livrables Attendus de la Phase 2

À la fin de cette phase, nous disposerons :

* d'une gouvernance automatisée ;
* d'une initiative Azure Policy ;
* d'un contrôle de conformité centralisé ;
* d'une première base de sécurité Azure.

---

# Coûts

Azure Policy ne génère pas de coût direct.

Cependant, certaines Policies utilisant :

* Log Analytics
* Azure Monitor
* DeployIfNotExists

pourront entraîner des coûts indirects qui seront détaillés dans les séries concernées.

---

# Conclusion

Nous avons désormais défini le rôle d'Azure Policy dans notre plateforme.

Les Management Groups fournissent la structure.

Azure Policy va désormais imposer les règles de gouvernance.

Dans le prochain article, nous déploierons notre première Policy avec Terraform afin d'imposer la présence des tags obligatoires sur les ressources Azure.
