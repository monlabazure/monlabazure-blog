---

title: "SERIES-2.8 - Validation de la Governance & Security Foundation"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-2"
* "Governance"
* "Azure Policy"
* "Compliance"
* "Validation"
* "Terraform"
* "Landing Zone"
* "Azure"

---

# SERIES-2.8 – Validation de la Governance & Security Foundation

## Introduction

La Phase 2 avait pour objectif de transformer notre Landing Zone Foundation en une plateforme gouvernée.

Lors de cette phase, nous avons mis en œuvre :

* Azure Policy ;
* contrôle des tags ;
* contrôle des régions ;
* contrôle des Diagnostic Settings ;
* contrôle des ressources publiques ;
* initiative de gouvernance centralisée.

Avant de poursuivre vers la gestion des identités et des accès, nous devons vérifier que notre socle de gouvernance est correctement déployé et conforme aux décisions d'architecture.

Cette étape est indispensable dans toute démarche Production Ready.

---

# Positionnement dans le Projet

```text
Phase 2 – Governance & Security Foundation

ADR-006   Stratégie Azure Policy et Gouvernance      ✅

SERIES-2.1   Introduction Azure Policy              ✅
SERIES-2.2   Premier Tag Obligatoire                ✅
SERIES-2.3   Industrialiser les Tags                ✅
SERIES-2.4   Régions Autorisées                     ✅
SERIES-2.5   Diagnostic Settings                    ✅
SERIES-2.6   Ressources Publiques                   ✅
SERIES-2.7   Initiative Governance Baseline         ✅
NOTE-2.1     Terraform Enterprise                   ✅
SERIES-2.8   Validation Governance Foundation       ⏳
```

---

# Objectif

À l'issue de cet article, nous devrons pouvoir affirmer que :

* la gouvernance Azure est opérationnelle ;
* les Policies sont correctement affectées ;
* l'héritage via les Management Groups fonctionne ;
* Terraform est cohérent avec Azure ;
* la plateforme est prête pour la Phase 3.

---

# Architecture Attendue

```text
Tenant Root Group
│
├── mla-platform
│
├── mla-landingzones
│   │
│   ├── mla-dev
│   ├── mla-qualif
│   └── mla-prod
│
└── mla-sandbox
```

avec :

```text
Azure Policy
│
└── MonLabAzure Governance Baseline
```

affectée sur :

```text
mla-landingzones
```

---

# Vérification des Policies

## Afficher les Affectations

```bash
az policy assignment list --output table
```

Vérifier la présence des affectations :

```text
audit-project-tag
audit-environment-tag
audit-owner-tag
audit-costcenter-tag
audit-criticality-tag
audit-managedby-tag
audit-dataclassification-tag
audit-backuprequired-tag

audit-allowed-locations

audit-diagnostic-settings

audit-storage-public-access

mla-governance-baseline
```

---

# Vérification des Initiatives

Lister les initiatives :

```bash
az policy set-definition list --output table
```

Résultat attendu :

```text
MonLabAzure Governance Baseline
```

---

# Vérification des Management Groups

Lister :

```bash
az account management-group list --output table
```

Résultat attendu :

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

# Vérification de l'Héritage

Portail Azure :

```text
Management Groups
│
└── mla-landingzones
      │
      ├── mla-dev
      ├── mla-qualif
      └── mla-prod
```

Vérifier que l'initiative :

```text
MonLabAzure Governance Baseline
```

apparaît également dans :

* DEV ;
* QUALIF ;
* PROD.

Cela confirme le fonctionnement de l'héritage Azure.

---

# Vérification de la Conformité

Accéder :

```text
Azure Policy
│
└── Compliance
```

Contrôler :

* conformité des tags ;
* conformité des régions ;
* conformité des Storage Accounts ;
* conformité des Diagnostic Settings.

---

# Tests Fonctionnels

## Test 1 – Ressource Sans Tag

Créer une ressource sans :

```text
Project
```

Résultat attendu :

```text
Non Compliant
```

---

## Test 2 – Région Non Autorisée

Créer une ressource dans :

```text
East US
```

Résultat attendu :

```text
Non Compliant
```

---

## Test 3 – Storage Public

Activer :

```text
Allow Blob Public Access
```

Résultat attendu :

```text
Non Compliant
```

---

## Test 4 – Diagnostic Settings

Créer une ressource sans Diagnostic Settings.

Résultat attendu :

```text
Non Compliant
```

---

# Vérification Terraform

Pour chaque répertoire Terraform de la Phase 2 :

```text
03-tag-policies
04-region-policies
05-diagnostic-policies
06-public-access-policies
07-governance-initiative
```

Exécuter :

```bash
terraform init
terraform validate
terraform plan
```

Résultat attendu :

```text
No changes.
Your infrastructure matches the configuration.
```

---

# Vérification des States

Lister les blobs :

```bash
az storage blob list \
  --account-name stmlatfstate01 \
  --container-name tfstate \
  --auth-mode login \
  --output table
```

Vérifier la présence :

```text
tag-policies.tfstate

region-policies.tfstate

diagnostic-policies.tfstate

public-access-policies.tfstate

governance-initiative.tfstate
```

---

# Vérification de Conformité avec les ADR

## ADR-005

Contrôles :

| Élément                     | Statut |
| --------------------------- | ------ |
| Stratégie de tags définie   | Oui    |
| Tags obligatoires contrôlés | Oui    |
| Gouvernance FinOps préparée | Oui    |

---

## ADR-006

Contrôles :

| Élément                          | Statut |
| -------------------------------- | ------ |
| Azure Policy utilisée            | Oui    |
| Affectation par Management Group | Oui    |
| Audit avant Deny                 | Oui    |
| Initiative centralisée           | Oui    |

---

# Checklist de Validation

Avant de clôturer la Phase 2 :

| Contrôle                        | Statut |
| ------------------------------- | ------ |
| Tags obligatoires contrôlés     | ☐      |
| Régions contrôlées              | ☐      |
| Diagnostic Settings contrôlés   | ☐      |
| Ressources publiques contrôlées | ☐      |
| Initiative créée                | ☐      |
| Héritage validé                 | ☐      |
| Terraform cohérent              | ☐      |
| States présents                 | ☐      |
| Conformité visible dans Azure   | ☐      |

---

# Coûts

Les composants déployés pendant la Phase 2 génèrent :

| Ressource         | Coût |
| ----------------- | ---- |
| Azure Policy      | 0 €  |
| Policy Initiative | 0 €  |
| Affectations      | 0 €  |

Coût total estimé :

```text
0 €
```

Les coûts liés à Azure Monitor et Log Analytics seront traités ultérieurement.

---

# Livrables

À l'issue de cet article :

✅ Azure Policy opérationnel

✅ Contrôle des tags

✅ Contrôle des régions

✅ Contrôle des ressources publiques

✅ Contrôle des Diagnostic Settings

✅ Initiative de gouvernance centralisée

✅ Conformité Azure visible

✅ Landing Zone gouvernée

---

# Ce que Nous Avons Construit

À la fin de la Phase 2, notre plateforme dispose désormais :

```text
Landing Zone Foundation
        │
        ▼
Governance Foundation
        │
        ├── Azure Policies
        ├── Initiative Governance
        ├── Contrôle des Tags
        ├── Contrôle des Régions
        ├── Contrôle Sécurité
        └── Contrôle Observabilité
```

Nous avons franchi une étape majeure.

La plateforme n'est plus seulement organisée.

Elle est désormais gouvernée.

---

# Conclusion

La Governance & Security Foundation est maintenant opérationnelle.

Les règles de conformité sont définies, déployées et appliquées de manière centralisée grâce à Azure Policy.

Cette gouvernance servira de base à toutes les futures phases du projet.

Dans la prochaine phase, nous nous attaquerons à un sujet fondamental de toute plateforme Azure Enterprise :

**Phase 3 – Identity & Access Management**, avec la mise en place de Microsoft Entra ID, des groupes de sécurité, du RBAC et du principe du moindre privilège.
