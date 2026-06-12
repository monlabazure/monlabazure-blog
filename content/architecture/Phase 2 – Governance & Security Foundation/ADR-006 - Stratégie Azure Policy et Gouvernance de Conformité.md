---

title: "ADR-0.6 - Stratégie Azure Policy et Gouvernance de Conformité"
date: 2026-05-15
draft: false
categories: ["Architecture"]
tags: ["ADR", "Azure Policy", "Governance", "Compliance", "Landing Zone"]
-------------------------------------------------------------------------

# ADR-006 – Stratégie Azure Policy et Gouvernance de Conformité

---

# Contexte

La Phase 1 a permis de mettre en place les fondations de gouvernance :

* Management Groups
* Structure des abonnements
* Backend Terraform
* Organisation des états
* Standards de nommage
* Stratégie de tags

Cependant, aucune règle de conformité n'est actuellement appliquée.

Les équipes pourraient encore :

* déployer des ressources sans tags ;
* utiliser des régions non autorisées ;
* exposer des ressources publiquement ;
* oublier les paramètres de diagnostic.

Afin de garantir une gouvernance cohérente, une stratégie Azure Policy doit être définie avant tout déploiement de ressources métier.

---

# Problème

Comment imposer les standards de gouvernance définis dans les ADR précédents de manière :

* centralisée ;
* automatisée ;
* reproductible ;
* compatible avec Terraform ?

---

# Décision

Le projet MonLabAzure Cloud Platform utilisera Azure Policy comme mécanisme principal de contrôle de conformité.

Les Azure Policies seront déployées au niveau des Management Groups afin de bénéficier de l'héritage Azure natif.

---

# Principes de Gouvernance

## Principe 1 – Tout est piloté par Azure Policy

Les contrôles de conformité ne doivent pas reposer sur des vérifications manuelles.

Les règles doivent être appliquées automatiquement.

---

## Principe 2 – Déploiement par Terraform

Toutes les Azure Policies seront gérées par Terraform.

Aucune Policy ne sera créée manuellement dans le portail Azure.

---

## Principe 3 – Héritage par Management Groups

Les Policies seront affectées au niveau :

```text
mla-platform
```

ou

```text
mla-landingzones
```

selon leur périmètre.

---

## Principe 4 – Progressivité

Les nouvelles Policies seront introduites progressivement afin de limiter les impacts opérationnels.

---

# Stratégie Audit puis Deny

Afin de limiter les interruptions de service :

## Étape 1

Mode :

```text
Audit
```

Objectif :

Identifier les ressources non conformes.

---

## Étape 2

Correction des écarts.

---

## Étape 3

Passage éventuel en :

```text
Deny
```

pour empêcher la création de ressources non conformes.

---

# Policies Prioritaires

Les premières Policies retenues sont :

## Tags obligatoires

Contrôle :

* Project
* Environment
* Owner
* CostCenter
* Criticality
* ManagedBy
* DataClassification
* BackupRequired

---

## Régions autorisées

Régions autorisées :

```text
West Europe
North Europe
```

---

## Diagnostic Settings

Vérification de la présence des paramètres de diagnostic.

---

## Ressources publiques

Audit puis interdiction progressive des expositions publiques non autorisées.

---

# Initiative MonLabAzure

Les Policies seront regroupées dans une initiative dédiée :

```text
MonLabAzure Governance Baseline
```

Objectifs :

* simplifier les affectations ;
* faciliter les audits ;
* centraliser la gouvernance.

---

# Exceptions

Les exceptions devront être :

* documentées ;
* approuvées ;
* limitées dans le temps.

Aucune exception permanente ne sera autorisée sans justification.

---

# Conséquences

## Avantages

* Gouvernance centralisée.
* Réduction des erreurs humaines.
* Meilleure conformité.
* Contrôle automatisé.

## Inconvénients

* Complexité supplémentaire.
* Risque de blocage en mode Deny.
* Nécessité de tester les Policies avant généralisation.

---

# Alternatives Étudiées

## Contrôle manuel

Rejeté.

Peu fiable et difficile à maintenir.

---

## Contrôle uniquement par Terraform

Rejeté.

Terraform ne permet pas de contrôler les ressources créées en dehors de Terraform.

---

# Validation

Cette décision sera considérée comme appliquée lorsque :

* les Policies prioritaires seront déployées ;
* l'initiative MonLabAzure Governance Baseline sera opérationnelle ;
* les audits de conformité seront disponibles.

---

# Décision Finale

Azure Policy devient le mécanisme officiel de gouvernance et de conformité de MonLabAzure Cloud Platform.

Toutes les futures séries de la Phase 2 devront respecter cette stratégie.
