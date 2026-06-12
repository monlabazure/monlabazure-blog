# SERIES-1.2 – Préparer le Tenant Azure et Vérifier les Prérequis de la Landing Zone Foundation

## Introduction

Dans l'article précédent, nous avons présenté les objectifs de la Landing Zone Foundation et son rôle dans la construction de MonLabAzure Cloud Platform.

Avant de commencer à déployer les premiers composants de gouvernance avec Terraform, il est nécessaire de vérifier que notre environnement Azure répond aux prérequis définis dans les ADR et dans l'architecture cible.

Cette étape est souvent négligée dans les laboratoires techniques. Pourtant, dans un projet réel, elle permet d'éviter de nombreux problèmes liés aux permissions, aux abonnements ou aux outils d'administration.

L'objectif de cet article est donc de préparer l'environnement de travail et de valider que nous sommes prêts à démarrer les déploiements.

---

# 1. Objectifs

À l'issue de cet article, nous devons être capables de :

* Vérifier l'accès au tenant Azure.
* Vérifier la présence des abonnements nécessaires.
* Contrôler les permissions Azure.
* Préparer le poste d'administration.
* Vérifier l'installation des outils.
* Préparer le dépôt Git du projet.
* Préparer Terraform.

Aucune ressource Azure ne sera créée à ce stade.

---

# 2. Hypothèses du Projet

Pour conserver une approche cohérente et reproductible, nous retenons les hypothèses suivantes :

## Tenant

Le tenant Microsoft Entra ID existe déjà.

Exemple :

```text
monlabazure.onmicrosoft.com
```

---

## Abonnements

Les abonnements Azure suivants existent déjà :

```text
sub-mla-connectivity
sub-mla-management
sub-mla-dev
sub-mla-qualif
sub-mla-prod
```

---

## Gouvernance

À ce stade :

* Aucun Management Group n'est créé.
* Aucune Azure Policy n'est déployée.
* Aucun RBAC spécifique n'est configuré.
* Aucun backend Terraform n'existe.

Nous partons donc d'un environnement quasiment vierge.

---

# 3. Note sur les Contrats Azure

Dans un environnement réel, la création des abonnements dépend du contrat Azure utilisé.

Les procédures peuvent varier selon l'organisation.

## Azure Free Account

Destiné aux démonstrations et aux laboratoires personnels.

Limites :

* Nombre réduit de ressources.
* Crédit temporaire.
* Peu adapté aux architectures Enterprise.

---

## Pay-As-You-Go (PAYG)

Très utilisé dans les laboratoires et petites structures.

Caractéristiques :

* Facturation à la consommation.
* Création simple des abonnements.
* Grande flexibilité.

---

## Microsoft Customer Agreement (MCA)

Modèle actuellement privilégié par Microsoft.

Caractéristiques :

* Gouvernance moderne.
* Gestion hiérarchique de la facturation.
* Création simplifiée des abonnements.

---

## Enterprise Agreement (EA)

Modèle historiquement utilisé par les grandes entreprises.

Caractéristiques :

* Gouvernance avancée.
* Facturation consolidée.
* Gestion d'un grand nombre d'abonnements.

---

Dans le cadre de ce projet, nous considérons que les abonnements nécessaires existent déjà afin de nous concentrer sur l'architecture et l'Infrastructure as Code.

---

# 4. Vérification du Poste d'Administration

## Git

Vérifier la version :

```bash
git --version
```

Résultat attendu :

```text
git version 2.x
```

---

## Terraform

Vérifier la version :

```bash
terraform version
```

Résultat attendu :

```text
Terraform v1.x
```

La version devra respecter les contraintes définies dans ADR-003.

---

## Azure CLI

Vérifier l'installation :

```bash
az version
```

Résultat attendu :

```text
azure-cli 2.x
```

---

# 5. Connexion à Azure

Connexion interactive :

```bash
az login
```

Après authentification :

```bash
az account show
```

Résultat attendu :

* Nom du compte.
* Tenant ID.
* Subscription active.

---

# 6. Vérification du Tenant

Afficher les informations du tenant :

```bash
az account tenant list --output table
```

Contrôles :

* Présence du tenant attendu.
* Tenant actif.
* Accès valide.

---

# 7. Vérification des Abonnements

Lister les abonnements :

```bash
az account subscription list --output table
```

Vérifier la présence des abonnements :

```text
sub-mla-connectivity
sub-mla-management
sub-mla-dev
sub-mla-qualif
sub-mla-prod
```

Si les noms diffèrent dans votre environnement, adaptez simplement les conventions utilisées dans les exemples.

---

# 8. Vérification des Permissions

Contrôler les rôles disponibles.

Exemple :

```bash
az role assignment list \
  --assignee <votre_compte> \
  --all \
  --output table
```

Objectif :

Vérifier que le compte dispose des droits nécessaires pour :

* Créer des Management Groups.
* Affecter des abonnements.
* Déployer des ressources Terraform.

---

# 9. Préparation du Dépôt Git

Créer le dépôt principal :

```text
monlabazure-cloud-platform
```

Structure minimale :

```text
monlabazure-cloud-platform
│
├── docs
├── terraform
├── scripts
└── README.md
```

Cette structure sera enrichie dans les prochaines séries conformément à ADR-003.

---

# 10. Préparation de Terraform

Créer l'arborescence initiale :

```text
terraform
│
├── 00-bootstrap
├── 01-management-groups
├── 02-policies
├── modules
└── environments
```

À ce stade, les dossiers peuvent être vides.

L'objectif est simplement de préparer le socle du projet.

---

# 11. Vérification des Conventions

Relire les documents suivants :

* ADR-001
* ADR-002
* ADR-003
* ADR-004
* ADR-005

Points à valider :

* Convention de nommage.
* Organisation Git.
* Stratégie Terraform.
* Tags obligatoires.
* Structure de gouvernance.

---

# 12. Checklist de Validation

Avant de poursuivre, les éléments suivants doivent être validés :

| Contrôle                  | Statut |
| ------------------------- | ------ |
| Git installé              | ☐      |
| Terraform installé        | ☐      |
| Azure CLI installé        | ☐      |
| Connexion Azure validée   | ☐      |
| Tenant accessible         | ☐      |
| Abonnements identifiés    | ☐      |
| Permissions vérifiées     | ☐      |
| Dépôt Git créé            | ☐      |
| Structure Terraform créée | ☐      |
| ADR relus                 | ☐      |

Tous les éléments doivent être validés avant de passer à la suite.

---

# 13. Coûts

Aucun coût Azure significatif n'est généré durant cette étape.

Seules des opérations de consultation et de préparation sont réalisées.

---

# 14. Conclusion

Nous disposons maintenant d'un environnement prêt à accueillir les premiers composants de gouvernance de notre Landing Zone Azure.

Les outils sont installés, les abonnements sont identifiés et les conventions du projet sont connues.

Dans le prochain article, nous commencerons la première implémentation réelle de la plateforme en créant la hiérarchie des Management Groups conformément à ADR-002.
