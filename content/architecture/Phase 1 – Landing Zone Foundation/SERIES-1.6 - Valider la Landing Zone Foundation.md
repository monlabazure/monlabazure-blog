# SERIES-1.6 – Valider la Landing Zone Foundation

## Introduction

Dans les articles précédents, nous avons construit les premières briques concrètes de MonLabAzure Cloud Platform.

Nous avons mis en place :

* le backend Terraform ;
* la hiérarchie des Management Groups ;
* le rattachement des abonnements Azure.

À ce stade, nous disposons d'une première Landing Zone Foundation opérationnelle.

Avant de continuer vers Azure Policy, RBAC, réseau ou supervision, nous devons valider que la fondation est correctement construite.

Cette étape est indispensable dans une démarche Production Ready.

---

# Positionnement dans le Projet

```text
Phase 1 - Landing Zone Foundation

SERIES-001-001  Introduction                      ✅
SERIES-001-002  Préparer le tenant Azure          ✅
SERIES-001-003  Bootstrap Terraform               ✅
SERIES-001-004  Management Groups                 ✅
SERIES-001-005  Rattachement abonnements          ✅
SERIES-001-006  Validation Foundation             ⏳
```

---

# 1. Objectif de l'Article

L'objectif est de vérifier que la structure de gouvernance initiale est conforme à l'architecture cible.

Nous allons contrôler :

* le backend Terraform ;
* les fichiers state ;
* la hiérarchie des Management Groups ;
* le rattachement des abonnements ;
* la cohérence avec ADR-002 ;
* la cohérence avec ADR-003 ;
* la préparation aux futures Azure Policies.

---

# 2. Architecture Attendue

La structure attendue est la suivante :

```text
Tenant Root Group
│
├── mla-platform
│   ├── mla-connectivity
│   │   └── sub-mla-connectivity
│   │
│   └── mla-management
│       └── sub-mla-management
│
├── mla-landingzones
│   ├── mla-dev
│   │   └── sub-mla-dev
│   │
│   ├── mla-qualif
│   │   └── sub-mla-qualif
│   │
│   └── mla-prod
│       └── sub-mla-prod
│
└── mla-sandbox
```

---

# 3. Vérifier le Backend Terraform

## Vérifier le Resource Group

```bash
az group show \
  --name rg-mla-management-tfstate-weu-01 \
  --output table
```

Résultat attendu :

```text
Name                                Location
----------------------------------  ----------
rg-mla-management-tfstate-weu-01    westeurope
```

---

## Vérifier le Storage Account

```bash
az storage account show \
  --name stmlatfstate01 \
  --resource-group rg-mla-management-tfstate-weu-01 \
  --output table
```

Contrôles :

* le Storage Account existe ;
* il est dans la région `westeurope` ;
* il est dans le Resource Group attendu.

---

## Vérifier le Container tfstate

```bash
az storage container list \
  --account-name stmlatfstate01 \
  --auth-mode login \
  --output table
```

Résultat attendu :

```text
Name
--------
tfstate
```

---

# 4. Vérifier les Fichiers d'État Terraform

Lister les fichiers présents dans le container :

```bash
az storage blob list \
  --account-name stmlatfstate01 \
  --container-name tfstate \
  --auth-mode login \
  --output table
```

Résultat attendu :

```text
Name
--------------------------------
management-groups.tfstate
subscription-associations.tfstate
```

Selon votre progression, le fichier `bootstrap.tfstate` peut également être présent.

---

# 5. Vérifier les Management Groups

Lister les Management Groups :

```bash
az account management-group list --output table
```

Les éléments suivants doivent être présents :

```text
mla-platform
mla-connectivity
mla-management
mla-landingzones
mla-dev
mla-qualif
mla-prod
mla-sandbox
```

---

# 6. Vérifier la Hiérarchie

Afficher un Management Group avec ses enfants :

```bash
az account management-group show \
  --name mla-platform \
  --expand \
  --recurse
```

Vérifier que `mla-platform` contient :

```text
mla-connectivity
mla-management
```

Puis :

```bash
az account management-group show \
  --name mla-landingzones \
  --expand \
  --recurse
```

Vérifier que `mla-landingzones` contient :

```text
mla-dev
mla-qualif
mla-prod
```

---

# 7. Vérifier le Rattachement des Abonnements

Vérifier le Management Group Connectivity :

```bash
az account management-group show \
  --name mla-connectivity \
  --expand \
  --recurse
```

Résultat attendu :

```text
sub-mla-connectivity
```

Vérifier le Management Group Management :

```bash
az account management-group show \
  --name mla-management \
  --expand \
  --recurse
```

Résultat attendu :

```text
sub-mla-management
```

Vérifier les Landing Zones :

```bash
az account management-group show \
  --name mla-dev \
  --expand \
  --recurse
```

```bash
az account management-group show \
  --name mla-qualif \
  --expand \
  --recurse
```

```bash
az account management-group show \
  --name mla-prod \
  --expand \
  --recurse
```

Résultats attendus :

```text
sub-mla-dev
sub-mla-qualif
sub-mla-prod
```

---

# 8. Vérifier Terraform

Dans chaque répertoire Terraform utilisé jusqu'ici :

```text
terraform/01-management-groups
terraform/02-subscription-associations
```

exécuter :

```bash
terraform init
terraform validate
terraform plan
```

Résultat attendu :

```text
No changes. Your infrastructure matches the configuration.
```

Ce résultat est important.

Il signifie que l'état Terraform correspond bien à ce qui existe dans Azure.

---

# 9. Vérification de Conformité avec les ADR

## ADR-002

Contrôles :

| Élément                               | Résultat attendu |
| ------------------------------------- | ---------------- |
| Management Group Platform             | Présent          |
| Management Group Connectivity         | Présent          |
| Management Group Management           | Présent          |
| Management Group LandingZones         | Présent          |
| Management Groups DEV / QUALIF / PROD | Présents         |
| Management Group Sandbox              | Présent          |
| Abonnements rattachés                 | Oui              |

---

## ADR-003

Contrôles :

| Élément                                    | Résultat attendu |
| ------------------------------------------ | ---------------- |
| Backend distant                            | Présent          |
| State séparé pour Management Groups        | Oui              |
| State séparé pour rattachement abonnements | Oui              |
| Dépôt Git structuré                        | Oui              |

---

## ADR-004

Contrôles :

| Élément              | Résultat attendu |
| -------------------- | ---------------- |
| Préfixe projet       | `mla`            |
| Nommage cohérent     | Oui              |
| Ressources conformes | Oui              |

---

# 10. Points d'Attention

## Propagation Azure

Les changements de Management Groups peuvent prendre quelques minutes à apparaître dans le portail Azure.

Si la hiérarchie ne semble pas immédiatement correcte, attendre quelques minutes puis relancer les commandes de vérification.

---

## Permissions

Certaines commandes peuvent échouer si le compte utilisé ne possède pas les droits nécessaires au niveau du Tenant Root Group ou des Management Groups.

---

## Abonnements déjà rattachés

Un abonnement ne peut être rattaché qu'à un seul Management Group à la fois.

Si un abonnement est déjà positionné ailleurs, Terraform déplacera l'association selon la configuration définie.

---

# 11. Coûts

Les composants validés dans cet article ne génèrent pas de coût significatif.

| Élément                           | Coût        |
| --------------------------------- | ----------- |
| Management Groups                 | 0 €         |
| Rattachement abonnements          | 0 €         |
| Backend Terraform Storage Account | Très faible |
| Azure CLI / Terraform validation  | 0 €         |

---

# 12. Checklist de Validation

Avant de considérer la Landing Zone Foundation comme prête, vérifier :

| Contrôle                         | Statut |
| -------------------------------- | ------ |
| Backend Terraform opérationnel   | ☐      |
| Container `tfstate` présent      | ☐      |
| States Terraform présents        | ☐      |
| Management Groups créés          | ☐      |
| Hiérarchie conforme à ADR-002    | ☐      |
| Abonnements rattachés            | ☐      |
| `terraform plan` sans changement | ☐      |
| Structure Git conforme à ADR-003 | ☐      |
| Nommage conforme à ADR-004       | ☐      |

---

# 13. Livrables

À l'issue de cet article :

✅ Backend Terraform validé

✅ Management Groups validés

✅ Abonnements rattachés

✅ Hiérarchie conforme à ADR-002

✅ States Terraform cohérents

✅ Landing Zone Foundation prête pour les Azure Policies

---

# 14. Conclusion

Nous avons terminé la première étape majeure de la construction de MonLabAzure Cloud Platform.

La Landing Zone Foundation est désormais en place.

Elle ne contient pas encore de réseau, de policies avancées, de supervision ou de workloads applicatifs, mais elle fournit le socle de gouvernance indispensable à toutes les futures séries.

Dans le prochain chapitre, nous pourrons commencer à renforcer cette fondation avec les premières Azure Policies, afin d'appliquer concrètement les règles de gouvernance définies dans nos ADR.
