---
title: "ADR-0.4 - Convention de Nommage Azure"
date: 2026-06-03
draft: false
categories: ["Architecture"]
tags: ["adr", "architecture", "cloud-platform", "finops", "tag"]
---

# ADR-004 - Convention de Nommage Azure

## Statut

Validé

## Date

Juin 2026

## Projet

MonLabAzure Cloud Platform

---

# 1. Contexte

La plateforme MonLabAzure Cloud Platform est destinée à évoluer progressivement et à intégrer un nombre important de ressources Azure réparties sur plusieurs abonnements, environnements et services.

Afin de garantir :

* la cohérence de l'architecture ;
* la lisibilité des ressources ;
* la facilité d'exploitation ;
* l'automatisation ;
* la gouvernance ;

une convention de nommage unique est définie et devra être appliquée à l'ensemble des ressources du projet.

Cette convention s'inspire des recommandations Microsoft Cloud Adoption Framework tout en restant adaptée à un environnement de laboratoire.

---

# 2. Principes Généraux

## Règle 1 : Uniformité

Toutes les ressources doivent respecter la convention définie dans cet ADR.

Aucune convention locale ou spécifique à une série ne doit être introduite.

---

## Règle 2 : Lisibilité

Le nom d'une ressource doit permettre d'identifier rapidement :

* le type de ressource ;
* le projet ;
* l'environnement ;
* la fonction ;
* la région ;
* l'instance.

---

## Règle 3 : Compatibilité Azure

Les conventions doivent respecter les contraintes Azure :

* longueur maximale ;
* caractères autorisés ;
* unicité globale lorsque nécessaire.

---

## Règle 4 : Automatisation

Les conventions doivent être compatibles avec Terraform et les mécanismes d'automatisation.

---

# 3. Préfixe Projet

Préfixe officiel :

```text
mla
```

Signification :

```text
MonLabAzure
```

Ce préfixe devra apparaître dans l'ensemble des ressources lorsque cela est possible.

---

# 4. Convention des Environnements

Valeurs autorisées :

```text
dev
qualif
prod
```

Règles :

* Minuscules uniquement.
* Aucune autre valeur autorisée.
* Utilisation systématique dans les ressources liées aux environnements.

---

# 5. Convention des Régions

Abréviations officielles :

| Région Azure | Abréviation |
| ------------ | ----------- |
| West Europe  | weu         |
| North Europe | neu         |

Exemples :

```text
vnet-mla-prod-weu-01
vnet-mla-prod-neu-01
```

---

# 6. Structure Générale

Format standard :

```text
<type>-<projet>-<environnement>-<fonction>-<région>-<instance>
```

Exemple :

```text
rg-mla-prod-network-weu-01
```

Lorsque le service Azure impose des contraintes spécifiques, la convention sera adaptée.

---

# 7. Resource Groups

## Format

```text
rg-mla-<env>-<fonction>-<region>-<instance>
```

## Exemples

```text
rg-mla-dev-network-weu-01
rg-mla-qualif-app-weu-01
rg-mla-prod-monitoring-weu-01
```

---

# 8. Réseau

## Virtual Networks

### Format

```text
vnet-mla-<fonction>-<region>-<instance>
```

### Exemples

```text
vnet-mla-hub-weu-01
vnet-mla-dev-weu-01
vnet-mla-qualif-weu-01
vnet-mla-prod-weu-01
```

---

## Subnets

### Format

```text
snet-<fonction>
```

### Exemples

```text
snet-firewall
snet-bastion
snet-app
snet-data
snet-private-endpoints
snet-aks
```

---

## Network Security Groups

### Format

```text
nsg-mla-<env>-<fonction>-<region>-<instance>
```

### Exemple

```text
nsg-mla-prod-app-weu-01
```

---

## Route Tables

### Format

```text
rt-mla-<env>-<fonction>-<region>-<instance>
```

### Exemple

```text
rt-mla-prod-app-weu-01
```

---

# 9. Sécurité

## Azure Firewall

### Format

```text
afw-mla-hub-<region>-<instance>
```

### Exemple

```text
afw-mla-hub-weu-01
```

---

## Azure Bastion

### Format

```text
bas-mla-hub-<region>-<instance>
```

### Exemple

```text
bas-mla-hub-weu-01
```

---

## Key Vault

Azure impose une contrainte d'unicité globale.

### Format

```text
kvmla<env><fonction><instance>
```

### Exemples

```text
kvmlaprodshared01
kvmladevshared01
kvmlaqualifshared01
```

Règles :

* Minuscules uniquement.
* Sans tirets.
* Nom unique globalement.

---

# 10. Monitoring

## Log Analytics Workspace

### Format

```text
law-mla-<fonction>-<region>-<instance>
```

### Exemple

```text
law-mla-platform-weu-01
```

---

## Application Insights

### Format

```text
appi-mla-<env>-<application>-<instance>
```

### Exemple

```text
appi-mla-prod-webapp-01
```

---

# 11. Stockage

## Storage Accounts

Azure impose :

* unicité globale ;
* minuscules uniquement ;
* pas de tirets.

### Format

```text
stmla<env><fonction><instance>
```

### Exemple

```text
stmlaprodtfstate01
stmlamanagementlogs01
```

---

# 12. Azure SQL

## SQL Server

### Format

```text
sqlmla<env><fonction><instance>
```

### Exemple

```text
sqlmlaprodbusiness01
```

---

## Base de données

### Format

```text
sqldb-mla-<env>-<application>
```

### Exemple

```text
sqldb-mla-prod-ecommerce
```

---

# 13. App Service

## App Service Plan

### Format

```text
asp-mla-<env>-<application>-<instance>
```

### Exemple

```text
asp-mla-prod-webapp-01
```

---

## Web App

### Format

```text
app-mla-<env>-<application>-<instance>
```

### Exemple

```text
app-mla-prod-webapp-01
```

---

# 14. Kubernetes

## Azure Container Registry

### Format

```text
acrmla<env><instance>
```

### Exemple

```text
acrmlaprod01
```

---

## AKS

### Format

```text
aks-mla-<env>-<region>-<instance>
```

### Exemple

```text
aks-mla-prod-weu-01
```

---

# 15. API Management

## Format

```text
apim-mla-<env>-<region>-<instance>
```

### Exemple

```text
apim-mla-prod-weu-01
```

---

# 16. Terraform

## Variables

Convention :

```text
snake_case
```

Exemples :

```hcl
resource_group_name
location
environment
application_name
```

---

## Locals

Convention :

```hcl
local.naming_prefix
local.common_tags
local.environment
```

---

## Outputs

Convention :

```hcl
resource_group_id
vnet_id
key_vault_uri
```

---

# 17. Exceptions

Certaines ressources Azure imposent leurs propres contraintes.

Dans ce cas :

* Les contraintes Azure priment.
* La logique métier de la convention doit être conservée autant que possible.
* Toute exception devra être documentée.

---

# 18. Contrôle de Conformité

Les conventions de nommage devront être contrôlées par :

* Revues de code.
* Pull Requests.
* Azure Policy lorsque possible.
* Contrôles Terraform.

Toute ressource non conforme devra être corrigée avant mise en production.

---

# 19. Justification

Cette convention :

* Facilite l'exploitation.
* Facilite la recherche des ressources.
* Améliore la gouvernance.
* Réduit les erreurs.
* Simplifie les scripts d'automatisation.
* Reste compatible avec les contraintes Azure.

---

# 20. Critères de Validation

ADR-004 est considéré comme appliqué lorsque :

* Toutes les nouvelles ressources respectent la convention.
* Les modules Terraform utilisent les conventions définies.
* Les revues de code contrôlent les noms.
* Les exemples publiés sur le blog utilisent exclusivement cette nomenclature.

---

# 21. Références

* Microsoft Cloud Adoption Framework
* Azure Naming and Tagging Strategy
* Azure Well-Architected Framework
* Terraform Best Practices

---

# 22. Décision Finale

La convention de nommage définie dans cet ADR devient la référence officielle du projet MonLabAzure Cloud Platform.

Toute évolution majeure devra faire l'objet d'un nouvel ADR.
