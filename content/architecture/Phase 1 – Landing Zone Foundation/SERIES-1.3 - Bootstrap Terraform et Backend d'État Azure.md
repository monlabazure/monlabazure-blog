# SERIES-1.3 – Bootstrap Terraform et Backend d'État Azure

## Introduction

Dans les articles précédents, nous avons préparé notre environnement Azure et validé les prérequis nécessaires à la mise en œuvre de la Landing Zone Foundation.

Nous sommes maintenant prêts à commencer les premiers déploiements.

Cependant, avant de créer les Management Groups, les Azure Policies ou toute autre ressource de gouvernance, nous devons résoudre un problème fondamental :

> Où Terraform va-t-il stocker son état ?

Cette étape est appelée Bootstrap Terraform.

Elle consiste à mettre en place l'infrastructure minimale nécessaire au fonctionnement de Terraform lui-même.

---

# Positionnement dans le Projet

```text
Phase 1 - Landing Zone Foundation

SERIES-001-001  Introduction                 ✅
SERIES-001-002  Préparation du tenant        ✅
SERIES-001-003  Bootstrap Terraform          ⏳
SERIES-001-004  Management Groups
SERIES-001-005  Abonnements
SERIES-001-006  Validation Foundation
```

---

# 1. Pourquoi un Backend Terraform ?

Terraform conserve un fichier appelé :

```text
terraform.tfstate
```

Ce fichier contient :

* les ressources créées ;
* leurs identifiants Azure ;
* leur configuration ;
* les dépendances.

Sans ce fichier, Terraform ne peut pas savoir ce qu'il a déjà déployé.

---

## Backend local

Par défaut :

```text
terraform.tfstate
```

est stocké localement.

Cette méthode est acceptable pour :

* des démonstrations ;
* des tests ponctuels.

Elle n'est pas adaptée à une plateforme Azure.

---

## Backend distant

Dans un environnement professionnel :

```text
Terraform
    │
    ▼
Azure Storage Account
    │
    ▼
tfstate
```

Avantages :

* stockage centralisé ;
* protection contre la perte de données ;
* collaboration ;
* verrouillage des états ;
* historisation.

---

# 2. Le paradoxe du Bootstrap

Nous rencontrons un problème classique.

Pour utiliser Terraform, nous avons besoin :

* d'un Storage Account ;
* d'un container Blob.

Mais ces ressources doivent elles-mêmes être créées.

Cette étape constitue l'unique exception au principe :

> Tout est déployé avec Terraform.

---

# 3. Architecture Bootstrap

Infrastructure cible :

```text
Management Subscription
│
└── rg-mla-management-tfstate-weu-01
    │
    └── stmlatfstate01
        │
        └── Container
            └── tfstate
```

---

# 4. Choix d'Architecture

## Pourquoi dans l'abonnement Management ?

Selon ADR-002 :

```text
Management Subscription
```

héberge :

* la supervision ;
* l'administration ;
* les composants de plateforme.

Le backend Terraform est un composant de plateforme.

---

## Pourquoi West Europe ?

Selon ADR-001 :

* Région primaire : West Europe
* Région secondaire : North Europe

Le backend principal sera hébergé en West Europe.

---

# 5. Ressources à Créer

## Resource Group

Nom :

```text
rg-mla-management-tfstate-weu-01
```

---

## Storage Account

Nom :

```text
stmlatfstate01
```

Conforme à ADR-004 :

* minuscules ;
* sans tirets ;
* unique globalement.

---

## Blob Container

Nom :

```text
tfstate
```

---

# 6. Création du Resource Group

Azure CLI :

```bash
az group create \
  --name rg-mla-management-tfstate-weu-01 \
  --location westeurope
```

Validation :

```bash
az group show \
  --name rg-mla-management-tfstate-weu-01
```

---

# 7. Création du Storage Account

```bash
az storage account create \
  --name stmlatfstate01 \
  --resource-group rg-mla-management-tfstate-weu-01 \
  --location westeurope \
  --sku Standard_LRS \
  --kind StorageV2
```

---

## Choix du SKU

Pour notre laboratoire :

```text
Standard_LRS
```

est largement suffisant.

Dans une entreprise réelle :

* GRS ;
* RA-GRS ;

pourraient être envisagés.

---

# 8. Activation des Bonnes Pratiques

## Soft Delete

```bash
az storage blob service-properties delete-policy update \
  --account-name stmlatfstate01 \
  --enable true \
  --days-retained 30
```

---

## Versioning

```bash
az storage account blob-service-properties update \
  --account-name stmlatfstate01 \
  --enable-versioning true
```

---

Pourquoi ?

Pour pouvoir récupérer un état Terraform supprimé ou corrompu.

---

# 9. Création du Container

```bash
az storage container create \
  --name tfstate \
  --account-name stmlatfstate01 \
  --auth-mode login
```

Validation :

```bash
az storage container list \
  --account-name stmlatfstate01 \
  --auth-mode login \
  --output table
```

---

# 10. Structure Git

Création du répertoire :

```text
terraform/
└── 00-bootstrap
```

Structure :

```text
00-bootstrap
│
├── versions.tf
├── providers.tf
├── backend.tf
├── variables.tf
├── outputs.tf
└── main.tf
```

Même si le bootstrap a été réalisé avec Azure CLI, nous préparons déjà la structure Terraform.

---

# 11. Configuration du Backend

Exemple :

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-mla-management-tfstate-weu-01"
    storage_account_name = "stmlatfstate01"
    container_name       = "tfstate"
    key                  = "bootstrap.tfstate"
  }
}
```

---

# 12. Initialisation Terraform

Commande :

```bash
terraform init
```

Résultat attendu :

```text
Terraform has been successfully initialized
```

---

# 13. Validation

Contrôles :

### Azure

* Resource Group créé
* Storage Account créé
* Container créé

### Terraform

* Backend initialisé
* Connexion au Storage Account validée

### Git

* Structure créée
* Premier commit réalisé

---

# 14. Dépannage

## Erreur de nom Storage Account

Cause :

Nom déjà utilisé globalement.

Solution :

Ajouter un suffixe :

```text
stmlatfstate01
stmlatfstate02
```

---

## Erreur d'authentification

Vérifier :

```bash
az login
```

et :

```bash
az account show
```

---

## Erreur backend

Vérifier :

* Resource Group
* Storage Account
* Container

---

# 15. Coûts

Ressource principale :

```text
Storage Account Standard_LRS
```

Coût estimé :

Très faible.

Dans un laboratoire :

quelques centimes par mois.

---

# 16. Nettoyage

Si nécessaire :

```bash
az group delete \
  --name rg-mla-management-tfstate-weu-01
```

Attention :

La suppression entraîne la perte des états Terraform.

À ne jamais faire sur une plateforme en exploitation.

---

# 17. Livrables

À la fin de cet article :

✅ Resource Group Bootstrap créé

✅ Storage Account Terraform créé

✅ Container tfstate créé

✅ Versioning activé

✅ Soft Delete activé

✅ Backend Terraform opérationnel

✅ Premier dépôt Terraform structuré

---

# 18. Conclusion

Nous avons mis en place le socle indispensable à tous les futurs déploiements Terraform.

Le backend distant est désormais opérationnel et conforme aux bonnes pratiques définies dans nos ADR.

À partir de maintenant, toutes les ressources de la plateforme seront déployées et gérées via Terraform.

Dans le prochain article, nous créerons notre première véritable brique de gouvernance Azure :

la hiérarchie des Management Groups définie dans ADR-002.
