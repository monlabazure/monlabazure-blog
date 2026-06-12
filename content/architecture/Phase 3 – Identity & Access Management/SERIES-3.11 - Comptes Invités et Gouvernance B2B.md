---

title: "SERIES-3.11 - Comptes Invités et Gouvernance B2B"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "Microsoft Entra ID"
* "B2B"
* "Guest Users"
* "External ID"
* "Access Reviews"
* "Entitlement Management"
* "Governance"
* "Azure"

---

# SERIES-3.11 – Comptes Invités et Gouvernance B2B

## Introduction

Dans les articles précédents, nous avons construit les fondations IAM de MonLabAzure Cloud Platform.

Nous avons traité :

* les groupes Microsoft Entra ID ;
* le modèle RBAC Azure ;
* les comptes administratifs ;
* les comptes Break Glass ;
* MFA et les méthodes phishing-resistant ;
* Conditional Access ;
* PIM ;
* Access Reviews ;
* les accès temporaires.

Nous devons maintenant traiter un sujet souvent sous-estimé :

```text
Les comptes invités.
```

Dans un environnement d'entreprise, il est fréquent de collaborer avec des personnes externes :

* prestataires ;
* consultants ;
* partenaires ;
* fournisseurs ;
* éditeurs ;
* auditeurs ;
* équipes projet externes.

Ces utilisateurs peuvent avoir besoin d'accéder à des applications, des groupes, des espaces collaboratifs ou parfois à certaines ressources Azure.

Sans gouvernance, les comptes invités deviennent rapidement un risque.

Ils peuvent rester actifs après la fin d'une mission, conserver des droits non nécessaires ou être membres de groupes sensibles.

L'objectif de cet article est de définir une stratégie de gouvernance B2B pour MonLabAzure Cloud Platform.

---

# Positionnement dans le Projet

```text
Phase 3 – Identity & Access Management

ADR-004 – Convention de Nommage Azure ✅
ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride ✅

SERIES-3.1 – Introduction à Identity & Access Management dans Azure ✅
SERIES-3.2 – Comprendre les licences Microsoft Entra ID ✅
SERIES-3.3 – Identité hybride : Entra Connect Sync, Cloud Sync et modes d'authentification ✅
SERIES-3.4 – Créer les groupes Microsoft Entra ID ✅
SERIES-3.5 – Modèle RBAC Azure et séparation des rôles ✅
SERIES-3.6 – Affecter les rôles RBAC aux Management Groups ✅
SERIES-3.7 – Comptes administratifs et comptes Break Glass ✅
SERIES-3.8 – MFA, authentification forte et méthodes phishing-resistant ✅
SERIES-3.9 – Conditional Access Foundation ✅
SERIES-3.10 – PIM, Access Reviews et accès temporaires ✅
SERIES-3.11 – Comptes invités et gouvernance B2B ⏳
SERIES-3.12 – Service Principals, App Registrations et Managed Identities
SERIES-3.13 – Export des logs Entra ID vers Log Analytics
SERIES-3.14 – Validation IAM Foundation
```

---

# Références d'Architecture

Cet article applique principalement **ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride**.

ADR-007 impose les principes suivants :

* appliquer le moindre privilège ;
* séparer les responsabilités ;
* éviter les accès permanents inutiles ;
* auditer les comptes sensibles ;
* revoir régulièrement les accès ;
* traiter les comptes invités comme un sujet de gouvernance ;
* préparer Access Reviews et Entitlement Management.

Les comptes invités doivent donc être intégrés au modèle IAM, et non gérés comme des exceptions informelles.

---

# Objectif de l'Article

À la fin de cet article, nous aurons défini :

* ce qu'est un compte invité B2B ;
* les cas d'usage des comptes invités ;
* les risques associés ;
* les règles d'invitation ;
* les règles d'accès ;
* la stratégie Conditional Access pour les invités ;
* la stratégie Access Reviews ;
* le positionnement d'Entitlement Management ;
* les règles de cycle de vie ;
* les contrôles d'audit à mettre en place.

---

# Qu'est-ce qu'un Compte Invité B2B ?

Un compte invité B2B est une identité externe invitée dans un tenant Microsoft Entra ID.

L'utilisateur conserve généralement son identité d'origine, mais il peut accéder à certaines ressources du tenant qui l'invite.

Exemple :

```text
consultant@partenaire.com
        │
        ▼
Invité dans le tenant MonLabAzure
        │
        ▼
Accès à une application ou un groupe
```

Microsoft Entra B2B collaboration permet aux organisations de collaborer avec des utilisateurs externes, et ces utilisateurs sont représentés dans le tenant ressource comme des comptes invités.

---

# Exemples de Cas d'Usage

Les comptes invités peuvent être utiles dans plusieurs cas.

## Prestataire Cloud

Un prestataire intervient temporairement sur une architecture Azure.

Besoin :

```text
Accès en lecture à certains environnements
ou
Accès temporaire à un projet spécifique
```

## Auditeur Sécurité

Un auditeur externe doit consulter les configurations.

Besoin :

```text
Reader
Security Reader
Accès aux rapports de conformité
```

## Partenaire Applicatif

Un partenaire doit accéder à une application métier.

Besoin :

```text
Accès applicatif
Pas d'accès Azure RBAC direct
```

## Consultant Projet

Un consultant participe à un projet pendant une période limitée.

Besoin :

```text
Accès temporaire
Approbation obligatoire
Expiration automatique
```

---

# Pourquoi les Comptes Invités sont Sensibles ?

Un compte invité n'est pas un compte interne.

Il peut dépendre :

* d'une autre organisation ;
* d'une autre politique MFA ;
* d'un autre cycle de vie RH ;
* d'un autre niveau de sécurité ;
* d'un autre processus de départ ;
* d'une boîte mail personnelle selon les cas.

Risques fréquents :

```text
Compte invité jamais supprimé
Accès conservé après la fin d'une mission
Membre d'un groupe sensible
Absence de MFA
Invitations non contrôlées
Aucun sponsor interne
Aucune date de fin
Aucune revue d'accès
```

Dans un environnement Enterprise, ces risques doivent être contrôlés.

---

# Principe MonLabAzure

MonLabAzure adopte la règle suivante :

```text
Tout compte invité doit avoir un objectif, un sponsor, un périmètre et une durée.
```

Un invité ne doit jamais être ajouté sans justification.

Il doit être possible de répondre aux questions suivantes :

```text
Pourquoi cet invité existe-t-il ?
Qui l'a invité ?
Qui est son sponsor interne ?
À quoi a-t-il accès ?
Depuis quand ?
Jusqu'à quand ?
Quand son accès sera-t-il revu ?
```

---

# Typologie des Comptes Invités

Dans MonLabAzure, nous distinguons plusieurs types d'invités.

## Invité Projet

Utilisateur externe lié à un projet précis.

Exemple :

```text
consultant projet
éditeur
partenaire technique
```

Durée :

```text
Limitée à la durée du projet
```

## Invité Audit

Utilisateur externe chargé d'une revue ou d'un audit.

Exemple :

```text
auditeur sécurité
auditeur conformité
cabinet externe
```

Durée :

```text
Courte et strictement encadrée
```

## Invité Support

Utilisateur externe intervenant sur un incident ou une expertise.

Exemple :

```text
support éditeur
expert technique
intégrateur
```

Durée :

```text
Temporaire
```

## Invité Applicatif

Utilisateur externe accédant uniquement à une application.

Exemple :

```text
client
partenaire métier
utilisateur externe SaaS
```

Durée :

```text
Selon le contrat ou le besoin métier
```

---

# Convention de Nommage et Identification

Les comptes invités sont visibles dans Microsoft Entra ID avec des propriétés spécifiques.

Leur User Principal Name peut être transformé pour être compatible avec le tenant.

Exemple :

```text
consultant_partenaire.com#EXT#@monlabazure.onmicrosoft.com
```

Il ne faut pas se baser uniquement sur le UPN pour comprendre le contexte d'un invité.

Il faut utiliser des attributs, des groupes ou une documentation complémentaire.

Champs utiles à documenter :

* société externe ;
* sponsor interne ;
* projet ;
* date de début ;
* date de fin ;
* justification ;
* niveau d'accès ;
* propriétaire métier.

---

# Règles d'Invitation

## Règle 1 – Invitation Contrôlée

Tout le monde ne doit pas pouvoir inviter des utilisateurs externes.

Règle cible :

```text
Les invitations externes doivent être contrôlées par un processus IAM ou par des propriétaires autorisés.
```

Microsoft recommande, dans les scénarios gouvernés avec Entitlement Management, de désactiver la possibilité pour les invités d’inviter d’autres invités afin d’éviter les invitations hors gouvernance.

---

## Règle 2 – Sponsor Obligatoire

Chaque invité doit avoir un sponsor interne.

Le sponsor est responsable de :

* valider le besoin ;
* confirmer la durée ;
* approuver les accès ;
* participer aux revues ;
* demander la suppression de l'accès.

Exemple :

```text
Invité :
consultant@partner.com

Sponsor :
responsable.projet@monlabazure.fr
```

---

## Règle 3 – Durée Définie

Un invité ne doit pas être créé sans horizon de fin.

Durées recommandées :

| Type d'invité         |                Durée recommandée |
| --------------------- | -------------------------------: |
| Audit                 |                     7 à 30 jours |
| Support éditeur       |                     1 à 14 jours |
| Projet court          |                    30 à 90 jours |
| Projet long           |              Revue trimestrielle |
| Partenaire applicatif | Selon contrat + revue périodique |

---

## Règle 4 – Moindre Privilège

Un invité doit recevoir uniquement les accès nécessaires.

Modèle recommandé :

```text
Invité
        │
        ▼
Groupe dédié
        │
        ▼
Accès applicatif ou RBAC limité
        │
        ▼
Revue régulière
```

---

# Groupes pour Comptes Invités

Il ne faut pas mélanger les invités avec les groupes administratifs internes.

À éviter :

```text
consultant@partenaire.com
        │
        ▼
AZR-MLA-PLATFORM-ADMINS
```

Modèle recommandé :

```text
consultant@partenaire.com
        │
        ▼
AZR-MLA-GUESTS-PROJECT-X-READERS
```

Ou, dans une logique plus générique :

```text
AZR-MLA-GUESTS-READERS
AZR-MLA-GUESTS-AUDITORS
AZR-MLA-GUESTS-PROJECTS
```

Pour MonLabAzure, nous pouvons définir une première convention :

```text
AZR-MLA-GUESTS-<USAGE>
```

Exemples :

```text
AZR-MLA-GUESTS-READERS
AZR-MLA-GUESTS-AUDITORS
AZR-MLA-GUESTS-PROJECT-X
```

---

# Accès Azure RBAC pour les Invités

Par défaut, les invités ne doivent pas recevoir de droits élevés sur Azure.

Rôles acceptables selon scénario :

```text
Reader
Security Reader
Monitoring Reader
```

Rôles à éviter pour les invités :

```text
Owner
User Access Administrator
Contributor sur PROD
Global Administrator
Privileged Role Administrator
```

Exception possible :

```text
Intervention exceptionnelle
Durée limitée
Approbation explicite
Scope réduit
Revue post-intervention
```

---

# Exemple d'Accès Invité Contrôlé

```text
auditeur@cabinet.com
        │
        ▼
AZR-MLA-GUESTS-AUDITORS
        │
        ▼
Reader
        │
        ▼
mla-landingzones
        │
        ▼
Durée : 30 jours
        │
        ▼
Access Review obligatoire
```

Ce modèle est acceptable.

Il est limité, lisible et auditable.

---

# Conditional Access pour les Invités

Les invités doivent être couverts par Conditional Access.

Objectifs :

* exiger MFA ;
* contrôler les accès externes ;
* limiter certains emplacements ;
* protéger les applications sensibles ;
* différencier les utilisateurs internes et externes ;
* appliquer des contrôles adaptés aux risques.

Microsoft documente la possibilité pour l'organisation ressource d’exiger MFA ou appareil conforme pour les utilisateurs B2B via Conditional Access ; Microsoft Entra peut aussi faire confiance aux revendications MFA provenant d’autres tenants Entra selon la configuration cross-tenant.

---

## Policy Recommandée – MFA pour Invités

Nom proposé :

```text
CA-MLA-GUESTS-REQUIRE-MFA-REPORT
```

Configuration logique :

```text
Users:
  Include:
    Guest or external users

Target resources:
  Selected cloud apps
  ou
  All cloud apps selon maturité

Grant:
  Require multifactor authentication

Mode:
  Report-only
```

Comme pour les autres policies Conditional Access, le mode initial doit être :

```text
Report-only
```

---

# Cross-Tenant Access Settings

Les Cross-Tenant Access Settings permettent de contrôler les relations avec d'autres tenants Microsoft Entra.

Ils permettent notamment de gérer :

* l'accès entrant ;
* l'accès sortant ;
* les organisations autorisées ;
* la confiance MFA ;
* la confiance appareil ;
* les paramètres de collaboration B2B.

Microsoft documente les paramètres cross-tenant access pour gérer la collaboration B2B et B2B direct connect avec d’autres organisations Microsoft Entra.

Dans MonLabAzure, ce sujet sera positionné comme une évolution avancée.

Pour un environnement Enterprise, il devient important lorsque l'organisation collabore régulièrement avec des partenaires identifiés.

---

# Access Reviews pour Invités

Les comptes invités doivent être revus régulièrement.

Pourquoi ?

Parce que leur cycle de vie échappe souvent à l'organisation qui les invite.

Exemples :

```text
Prestataire parti
Projet terminé
Partenaire sans contrat actif
Compte externe compromis
Sponsor interne parti
```

Microsoft documente l’utilisation des Access Reviews pour gérer l’accès des comptes invités B2B et faciliter la collaboration externe tout en contrôlant les accès.

---

## Fréquence Recommandée

| Type d'invité           |            Fréquence de revue |
| ----------------------- | ----------------------------: |
| Auditeur                |                Fin de mission |
| Support éditeur         |            Après intervention |
| Projet court            |                     Mensuelle |
| Projet long             |                 Trimestrielle |
| Partenaire applicatif   | Trimestrielle ou semestrielle |
| Invité avec accès Azure |    Mensuelle ou trimestrielle |

---

## Responsable de la Revue

La revue doit être réalisée par :

* le sponsor interne ;
* le propriétaire de l'application ;
* le propriétaire du projet ;
* l'équipe sécurité pour les accès sensibles ;
* l'équipe IAM pour les comptes à risque.

---

# Entitlement Management

Entitlement Management permet de gérer les accès via des packages.

Un access package peut contenir :

* groupes ;
* applications ;
* rôles ;
* ressources ;
* durée d'accès ;
* approbation ;
* revue périodique.

Microsoft décrit Entitlement Management comme une fonctionnalité de gouvernance permettant de gérer le cycle de vie des identités et accès à grande échelle, notamment via des access packages et des workflows d’accès.

---

# Exemple d'Access Package

```text
Access Package :
Audit MonLabAzure

Ressources :
- Groupe AZR-MLA-GUESTS-AUDITORS
- Application de reporting
- Accès Reader limité

Règles :
- Approbation par Security Team
- Durée 30 jours
- Revue à la fin de la période
```

Entitlement Management devient très utile lorsque les accès invités deviennent nombreux.

Dans MonLabAzure, il sera positionné comme une cible Enterprise.

---

# Cycle de Vie d'un Invité

Un compte invité doit suivre un cycle de vie clair.

```text
Demande d'accès
        │
        ▼
Validation du sponsor
        │
        ▼
Invitation
        │
        ▼
Ajout au groupe approprié
        │
        ▼
Accès limité
        │
        ▼
Revue périodique
        │
        ▼
Suppression ou renouvellement
```

---

# Processus MonLabAzure

## Étape 1 – Demande

La demande doit contenir :

```text
Nom de l'invité
Adresse email
Organisation externe
Sponsor interne
Objectif
Ressources demandées
Durée demandée
Justification
```

## Étape 2 – Validation

La validation doit être faite par :

* sponsor interne ;
* propriétaire applicatif ;
* sécurité si accès sensible ;
* plateforme si accès Azure RBAC.

## Étape 3 – Invitation

L'invitation est envoyée après validation.

## Étape 4 – Affectation

L'invité est ajouté au groupe adapté.

Exemple :

```text
AZR-MLA-GUESTS-AUDITORS
```

## Étape 5 – Revue

Une revue est planifiée selon la durée et le type d'accès.

## Étape 6 – Suppression

À la fin du besoin, les accès doivent être supprimés.

Si l'utilisateur n'a plus aucun besoin, le compte invité doit être supprimé du tenant.

---

# Commandes Azure CLI Utiles

## Lister les Comptes Invités

```bash
az ad user list \
  --filter "userType eq 'Guest'" \
  --query "[].{DisplayName:displayName, UserPrincipalName:userPrincipalName, Id:id}" \
  --output table
```

## Afficher un Invité

```bash
az ad user show \
  --id "consultant_partenaire.com#EXT#@monlabazure.onmicrosoft.com"
```

## Ajouter un Invité à un Groupe

```bash
GUEST_ID=$(az ad user show \
  --id "consultant_partenaire.com#EXT#@monlabazure.onmicrosoft.com" \
  --query id \
  --output tsv)

az ad group member add \
  --group "AZR-MLA-GUESTS-AUDITORS" \
  --member-id "$GUEST_ID"
```

## Vérifier les Membres du Groupe

```bash
az ad group member list \
  --group "AZR-MLA-GUESTS-AUDITORS" \
  --query "[].{DisplayName:displayName, UserPrincipalName:userPrincipalName, Id:id}" \
  --output table
```

Ces commandes servent à la vérification. Dans un environnement Enterprise, l'onboarding des invités devrait passer par un processus gouverné.

---

# Ce que Nous Ne Faisons Pas Encore

Dans cet article, nous ne mettons pas encore en œuvre :

* Entitlement Management complet ;
* access packages ;
* workflow d'approbation ;
* expiration automatique avancée ;
* cross-tenant access settings détaillés ;
* automatisation complète du cycle de vie invité.

Ces sujets relèvent d'une maturité IAM avancée.

L'objectif ici est de définir le cadre de gouvernance.

---

# Contrôles d'Audit

Les contrôles suivants doivent être prévus :

```text
Nombre total d'invités
Invités sans sponsor
Invités sans groupe
Invités dans des groupes privilégiés
Invités avec rôles Azure RBAC
Invités avec accès PROD
Invités inactifs
Invités sans revue récente
Invités créés manuellement hors processus
Invités provenant de domaines non approuvés
Invités avec MFA non satisfait
```

---

# Points de Vigilance

## Ne Pas Confondre Collaboration et Administration

Un invité peut collaborer sans avoir de droits Azure élevés.

L'accès administratif à un invité doit rester exceptionnel.

## Ne Pas Laisser les Invités Sans Expiration

Un invité sans date de fin est un risque.

## Ne Pas Ajouter les Invités aux Groupes Internes Sensibles

Les groupes administratifs internes ne doivent pas contenir d'invités sauf exception validée.

## Ne Pas Oublier le Sponsor

Sans sponsor, personne ne porte la responsabilité de l'accès.

## Ne Pas Négliger Conditional Access

Les invités doivent être protégés par MFA ou contrôles adaptés.

---

# Dépannage

## L'Invité ne Peut Pas Accéder à une Ressource

Causes possibles :

* invitation non acceptée ;
* mauvais groupe ;
* rôle RBAC non hérité ;
* Conditional Access bloquant ;
* MFA non configuré ;
* application non assignée ;
* cross-tenant settings restrictifs.

## L'Invité Apparaît mais n'a Aucun Accès

Vérifier :

* appartenance aux groupes ;
* affectations RBAC ;
* accès applicatifs ;
* scope ;
* état de l'invitation.

## L'Invité est Encore Présent Après la Fin du Projet

Action :

```text
Retirer les groupes
Révoquer les accès
Supprimer le compte invité si aucun besoin restant
Documenter la suppression
```

---

# Coûts et Licences

Les comptes invités B2B peuvent être utilisés dans Microsoft Entra External ID.

Certaines fonctionnalités avancées de gouvernance peuvent nécessiter des licences spécifiques :

```text
Access Reviews
Entitlement Management
Lifecycle Workflows
Cross-tenant access avancé
Conditional Access selon scénario
```

Microsoft documente des règles de licence spécifiques pour Microsoft Entra ID Governance, notamment Entitlement Management, Access Reviews et Lifecycle Workflows.

Dans MonLabAzure, la première étape reste conceptuelle et organisationnelle.

Les fonctions avancées seront utilisées selon les licences disponibles.

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* défini ce qu'est un compte invité B2B ;
* identifié les cas d'usage ;
* identifié les risques ;
* défini les règles d'invitation ;
* défini le rôle du sponsor ;
* proposé une convention de groupes invités ;
* cadré l'utilisation Azure RBAC pour les invités ;
* défini la stratégie Conditional Access ;
* positionné Access Reviews ;
* positionné Entitlement Management ;
* défini un cycle de vie invité.

---

# Conclusion

Les comptes invités sont indispensables dans les environnements modernes, mais ils doivent être gouvernés.

Un compte invité mal géré peut devenir un accès oublié, puis un risque de sécurité.

La stratégie MonLabAzure repose sur des principes simples :

```text
Un invité doit avoir un sponsor.
Un invité doit avoir une justification.
Un invité doit avoir un périmètre.
Un invité doit avoir une durée.
Un invité doit être revu.
```

Les comptes invités ne doivent pas être gérés comme des exceptions permanentes.

Ils doivent être intégrés au modèle IAM, aux Access Reviews, à Conditional Access et, à terme, à Entitlement Management.

Dans le prochain article, nous traiterons les identités techniques : Service Principals, App Registrations et Managed Identities.
