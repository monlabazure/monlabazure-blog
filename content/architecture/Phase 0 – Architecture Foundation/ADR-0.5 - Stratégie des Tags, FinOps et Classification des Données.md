---
title: "ADR-0.5 - Stratégie des Tags, FinOps et Classification des Données"
date: 2026-06-03
draft: false
categories: ["Architecture"]
tags: ["adr", "architecture", "cloud-platform", "finops", "tag"]
---

# ADR-005 - Stratégie des Tags, FinOps et Classification des Données

## Statut

Validé

## Date

Juin 2026

## Projet

MonLabAzure Cloud Platform

---

# 1. Contexte

Le projet MonLabAzure Cloud Platform repose sur les principes de gouvernance, d'automatisation, de sécurité et de maîtrise des coûts définis dans les ADR précédents.

Afin de garantir une gestion cohérente des ressources Azure, il est nécessaire de définir :

* une stratégie de tagging commune ;
* une classification des données ;
* des règles de gouvernance FinOps ;
* des mécanismes de contrôle et de conformité.

Cette stratégie s'applique à l'ensemble des ressources déployées dans les abonnements du projet.

---

# 2. Objectifs

La présente décision vise à :

* Identifier clairement les ressources Azure.
* Faciliter l'exploitation.
* Améliorer le suivi des coûts.
* Permettre l'automatisation des contrôles.
* Renforcer la gouvernance.
* Préparer les futures Azure Policies.
* Faciliter les audits et les rapports.

---

# 3. Principes Généraux

## Principe 1 : Tagging obligatoire

Toute ressource Azure doit être créée avec les tags obligatoires définis dans cet ADR.

Une ressource sans tags est considérée comme non conforme.

---

## Principe 2 : Normalisation

Les valeurs de tags doivent être standardisées.

Les variations telles que :

```text
Prod
PROD
production
Production
```

sont interdites.

Une seule valeur officielle est autorisée.

---

## Principe 3 : Terraform First

Les tags doivent être appliqués automatiquement par Terraform.

Aucune gestion manuelle n'est autorisée.

---

## Principe 4 : Gouvernance continue

La conformité des tags doit être contrôlée par :

* Azure Policy
* Revues de code
* Terraform
* Audits réguliers

---

# 4. Tags Obligatoires

Les tags suivants sont obligatoires pour toutes les ressources compatibles Azure.

| Tag                | Description                 |
| ------------------ | --------------------------- |
| Project            | Projet propriétaire         |
| Environment        | Environnement               |
| Owner              | Responsable de la ressource |
| CostCenter         | Centre de coût              |
| Criticality        | Niveau de criticité         |
| ManagedBy          | Outil de gestion            |
| DataClassification | Classification des données  |
| BackupRequired     | Sauvegarde requise          |

---

# 5. Définition des Tags

## Project

Valeur officielle :

```text
MonLabAzure
```

Objectif :

Identifier les ressources appartenant au projet.

---

## Environment

Valeurs autorisées :

```text
dev
qualif
prod
```

Aucune autre valeur n'est autorisée.

---

## Owner

Valeurs recommandées :

```text
CloudTeam
NetworkTeam
SecurityTeam
PlatformTeam
```

Objectif :

Identifier l'équipe responsable.

---

## CostCenter

Valeurs recommandées :

```text
IT
PLATFORM
SECURITY
NETWORK
```

Objectif :

Faciliter l'analyse des coûts.

---

## Criticality

Valeurs autorisées :

```text
low
medium
high
```

Définition :

### low

Impact limité.

Exemple :

* Ressources de développement.

### medium

Impact modéré.

Exemple :

* Qualification.

### high

Impact important sur l'activité.

Exemple :

* Production.

---

## ManagedBy

Valeur officielle :

```text
Terraform
```

Objectif :

Identifier l'origine du déploiement.

---

## DataClassification

Classification officielle des données.

Valeurs autorisées :

```text
Public
Internal
Confidential
Restricted
```

Définition détaillée dans la section 6.

---

## BackupRequired

Valeurs autorisées :

```text
Yes
No
```

Objectif :

Identifier les ressources devant être intégrées à une stratégie de sauvegarde.

---

# 6. Classification des Données

## Public

Données librement diffusables.

Exemples :

* Documentation publique
* Site vitrine
* Informations marketing

Impact en cas de divulgation :

Faible.

---

## Internal

Données réservées à l'entreprise.

Exemples :

* Journaux techniques
* Documentation interne
* Informations opérationnelles

Impact en cas de divulgation :

Modéré.

---

## Confidential

Données sensibles.

Exemples :

* Données applicatives
* Données métiers
* Informations clients

Impact en cas de divulgation :

Important.

---

## Restricted

Données critiques.

Exemples :

* Secrets Key Vault
* Clés de chiffrement
* Comptes privilégiés
* Secrets applicatifs

Impact en cas de divulgation :

Très élevé.

---

# 7. Classification Recommandée des Ressources

| Ressource               | Classification |
| ----------------------- | -------------- |
| Documentation publique  | Public         |
| Azure Monitor Logs      | Internal       |
| Log Analytics           | Internal       |
| Base SQL métier         | Confidential   |
| Storage applicatif      | Confidential   |
| App Service métier      | Confidential   |
| Azure Key Vault         | Restricted     |
| Secrets Terraform       | Restricted     |
| Comptes administrateurs | Restricted     |

---

# 8. Modèle de Tags Standard

Exemple pour une ressource de production :

```text
Project            = MonLabAzure
Environment        = prod
Owner              = PlatformTeam
CostCenter         = IT
Criticality        = high
ManagedBy          = Terraform
DataClassification = Confidential
BackupRequired     = Yes
```

---

# 9. Implémentation Terraform

Tous les modules Terraform devront utiliser un bloc commun de tags.

Exemple logique :

```hcl
local.common_tags = {
  Project            = "MonLabAzure"
  Environment        = var.environment
  Owner              = var.owner
  CostCenter         = var.cost_center
  Criticality        = var.criticality
  ManagedBy          = "Terraform"
  DataClassification = var.data_classification
  BackupRequired     = var.backup_required
}
```

Les ressources devront utiliser :

```hcl
tags = local.common_tags
```

---

# 10. Stratégie FinOps

## Objectif

Optimiser les dépenses Azure tout en maintenant un niveau de qualité compatible avec les exigences du projet.

---

## Principes

### Consommation maîtrisée

Chaque ressource doit répondre à un besoin identifié.

---

### Dimensionnement adapté

Les SKU doivent être choisies selon les besoins réels.

---

### Suppression des ressources inutiles

Les ressources temporaires doivent être détruites après validation.

---

### Analyse régulière

Les coûts doivent être analysés régulièrement.

---

### Traçabilité

Les tags doivent permettre d'identifier :

* l'environnement ;
* le propriétaire ;
* le centre de coût ;
* la criticité.

---

# 11. Exigences pour les Séries du Blog

Chaque série devra inclure :

## Coût estimatif

Présentation du coût prévisionnel.

---

## Alternatives économiques

Présentation d'une version adaptée à un laboratoire.

---

## Impact FinOps

Analyse des conséquences financières.

---

## Nettoyage

Procédure de suppression des ressources.

---

# 12. Azure Policy

Les politiques suivantes devront être étudiées dans les futures séries :

## Contrôle des Tags

* Audit des tags manquants.
* Refus des ressources non conformes.

---

## Contrôle de Classification

* Audit des classifications absentes.

---

## Contrôle des Sauvegardes

* Audit des ressources critiques sans sauvegarde.

---

## Contrôle des Régions

* Restriction aux régions autorisées.

---

# 13. Contrôle de Conformité

Une ressource est considérée conforme lorsque :

* Les tags obligatoires sont présents.
* Les valeurs respectent les nomenclatures officielles.
* La classification des données est définie.
* Le besoin de sauvegarde est identifié.

---

# 14. Justification

Cette stratégie :

* Renforce la gouvernance.
* Facilite l'exploitation.
* Prépare l'automatisation.
* Simplifie le suivi des coûts.
* Améliore la conformité.
* Prépare les futures Azure Policies.

---

# 15. Critères de Validation

ADR-005 sera considéré comme implémenté lorsque :

* Les modules Terraform utilisent les tags standard.
* Les Azure Policies de contrôle sont déployées.
* Les ressources critiques possèdent une classification.
* Les rapports de coûts utilisent les tags définis.
* Les nouvelles ressources respectent la stratégie de tagging.

---

# 16. Références

* Microsoft Cloud Adoption Framework
* Azure Well-Architected Framework
* Azure FinOps Guidance
* Azure Policy Documentation
* Azure Resource Tagging Best Practices

---

# 17. Décision Finale

La stratégie de tags, de classification des données et de gouvernance FinOps définie dans cet ADR devient la référence officielle du projet MonLabAzure Cloud Platform.

Tous les futurs développements Terraform et toutes les futures séries devront respecter les règles définies dans ce document.
