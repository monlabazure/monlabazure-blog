---

title: "SERIES-3.7 - Comptes Administratifs et Comptes Break Glass"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "Microsoft Entra ID"
* "Break Glass"
* "Administrative Accounts"
* "Security"
* "Conditional Access"
* "RBAC"
* "Azure"

---

# SERIES-3.7 – Comptes Administratifs et Comptes Break Glass

## Introduction

Dans les séries précédentes, nous avons construit les premières briques IAM de MonLabAzure Cloud Platform.

Nous avons :

* étudié les licences Microsoft Entra ID ;
* présenté les scénarios d'identité hybride ;
* créé les groupes Microsoft Entra ID ;
* défini le modèle RBAC cible ;
* affecté les premiers rôles RBAC aux Management Groups.

Nous devons maintenant traiter un sujet plus sensible :

```text
Les comptes administratifs et les comptes Break Glass.
```

Ces comptes ne sont pas des comptes ordinaires.

Ils permettent d'administrer la plateforme, de modifier des accès, de gérer des ressources critiques ou de récupérer l'accès au tenant en cas d'incident.

Une mauvaise gestion de ces comptes peut entraîner :

* compromission de la plateforme ;
* élévation de privilèges ;
* perte de contrôle du tenant ;
* verrouillage administratif ;
* impossibilité d'intervenir en cas d'urgence.

Cet article définit et prépare la stratégie opérationnelle des comptes administratifs et des comptes Break Glass pour MonLabAzure.

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
SERIES-3.7 – Comptes administratifs et comptes Break Glass ⏳
SERIES-3.8 – MFA, authentification forte et méthodes phishing-resistant
SERIES-3.9 – Conditional Access Foundation
SERIES-3.10 – PIM, Access Reviews et accès temporaires
SERIES-3.11 – Comptes invités et gouvernance B2B
SERIES-3.12 – Service Principals, App Registrations et Managed Identities
SERIES-3.13 – Export des logs Entra ID vers Log Analytics
SERIES-3.14 – Validation IAM Foundation
```

---

# Références d'Architecture

Cet article s'appuie principalement sur **ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride**.

ADR-007 définit les principes suivants :

* séparation des comptes standards et administratifs ;
* interdiction des comptes partagés pour l'administration quotidienne ;
* comptes administratifs nominatifs ;
* deux comptes Break Glass cloud-only ;
* comptes Break Glass hors administration quotidienne ;
* surveillance obligatoire des comptes sensibles ;
* test régulier des comptes d'urgence ;
* prise en compte future de PIM, Conditional Access et Access Reviews.

Cet article transforme ces décisions en modèle opérationnel.

---

# Objectif de l'Article

À la fin de cet article, nous aurons défini :

* la différence entre compte standard et compte administratif ;
* la convention de nommage des comptes administratifs ;
* les règles d'utilisation des comptes administratifs ;
* les règles de création des comptes Break Glass ;
* les rôles à attribuer aux comptes Break Glass ;
* les règles Conditional Access associées ;
* les contrôles de surveillance ;
* la procédure de test ;
* les points de validation avant de continuer la Phase 3.

---

# Compte Standard vs Compte Administratif

Dans un environnement sécurisé, un même compte ne doit pas servir à tout faire.

Il faut distinguer :

```text
Compte utilisateur standard
Compte administratif nominatif
Compte Break Glass
Compte de service
Managed Identity
```

Dans cet article, nous traitons les trois premiers.

Les comptes de service, Service Principals, App Registrations et Managed Identities seront traités dans une série dédiée.

---

## Compte Utilisateur Standard

Le compte utilisateur standard sert aux activités quotidiennes.

Exemple :

```text
prenom.nom@monlabazure.fr
```

Usage :

* messagerie ;
* bureautique ;
* consultation ;
* portail utilisateur ;
* collaboration ;
* tâches non privilégiées.

Ce compte ne doit pas disposer de droits administratifs élevés sur Azure.

---

## Compte Administratif Nominatif

Le compte administratif sert uniquement aux actions d'administration.

Exemple :

```text
adm-prenom.nom@monlabazure.fr
```

Usage :

* administration Azure ;
* gestion des ressources ;
* opérations privilégiées ;
* actions sur les Management Groups ;
* gestion des incidents ;
* exploitation contrôlée.

Ce compte est nominatif.

Il ne doit pas être partagé.

Il doit être traçable.

---

## Compte Break Glass

Le compte Break Glass est un compte d'urgence.

Exemple :

```text
bg-admin-01@monlabazure.onmicrosoft.com
bg-admin-02@monlabazure.onmicrosoft.com
```

Usage :

* perte d'accès des administrateurs ;
* erreur Conditional Access ;
* indisponibilité MFA ;
* problème d'identité hybride ;
* verrouillage accidentel du tenant ;
* incident majeur.

Ce compte n'est pas utilisé pour l'administration quotidienne.

Microsoft recommande de définir au moins deux comptes d’accès d’urgence cloud-only, avec le rôle Global Administrator assigné de manière permanente, et limités aux scénarios où les comptes administratifs normaux ne peuvent pas être utilisés.

---

# Pourquoi Séparer les Comptes ?

Un compte utilisateur standard est plus exposé.

Il est utilisé pour :

* recevoir des emails ;
* naviguer ;
* collaborer ;
* accéder à des applications ;
* ouvrir des documents ;
* interagir avec de nombreux services.

Si ce compte possède aussi des droits administratifs, le risque augmente fortement.

Exemple à éviter :

```text
prenom.nom@monlabazure.fr
        │
        ▼
Global Administrator
```

Modèle recommandé :

```text
prenom.nom@monlabazure.fr
        │
        ▼
Usage quotidien

adm-prenom.nom@monlabazure.fr
        │
        ▼
Administration Azure
```

Cette séparation réduit la surface d'attaque.

---

# Convention de Nommage des Comptes

Conformément à ADR-004 et ADR-007, nous appliquons une convention claire.

## Comptes Standards

Format :

```text
prenom.nom@monlabazure.fr
```

Exemple :

```text
jean.dupont@monlabazure.fr
```

## Comptes Administratifs

Format :

```text
adm-prenom.nom@monlabazure.fr
```

Exemple :

```text
adm-jean.dupont@monlabazure.fr
```

## Comptes Break Glass

Format :

```text
bg-admin-XX@monlabazure.onmicrosoft.com
```

Exemples :

```text
bg-admin-01@monlabazure.onmicrosoft.com
bg-admin-02@monlabazure.onmicrosoft.com
```

Pourquoi utiliser le domaine `onmicrosoft.com` ?

Parce que ces comptes doivent rester indépendants d'un domaine personnalisé qui pourrait être affecté par un incident DNS, fédération ou configuration externe.

---

# Règles pour les Comptes Administratifs

## Règle 1 – Compte Nominatif

Chaque administrateur dispose de son propre compte administratif.

Exemple :

```text
adm-prenom.nom@monlabazure.fr
```

Il ne doit pas exister de compte partagé comme :

```text
adminazure@monlabazure.fr
```

ou :

```text
superadmin@monlabazure.fr
```

Les comptes partagés nuisent à la traçabilité.

---

## Règle 2 – MFA Obligatoire

Tous les comptes administratifs doivent utiliser MFA.

Cette règle sera détaillée dans la série suivante sur l'authentification forte.

Pour l'instant, la décision est simple :

```text
Pas de compte administratif sans MFA.
```

---

## Règle 3 – Droits via Groupes

Les comptes administratifs ne doivent pas recevoir directement des rôles Azure RBAC.

Ils doivent être ajoutés aux groupes Microsoft Entra ID créés précédemment.

Exemple :

```text
adm-jean.dupont@monlabazure.fr
        │
        ▼
AZR-MLA-NETWORK-ADMINS
        │
        ▼
Network Contributor
        │
        ▼
mla-connectivity
```

---

## Règle 4 – Pas d'Usage Quotidien

Un compte administratif ne doit pas être utilisé pour :

* messagerie quotidienne ;
* navigation Internet ;
* collaboration Teams ;
* lecture de documents externes ;
* usage bureautique.

Il doit être réservé aux tâches administratives.

---

## Règle 5 – Accès Production Limité

Les droits en production doivent rester strictement contrôlés.

Règle MonLabAzure :

```text
Pas de Contributor permanent en PROD pour les développeurs.
```

Les accès production doivent être :

* justifiés ;
* limités ;
* documentés ;
* idéalement temporaires avec PIM dans les phases avancées.

---

# Création des Comptes Administratifs

Dans un environnement réel, la création des comptes administratifs dépend souvent d'un processus IAM ou RH.

Dans MonLabAzure, nous pouvons définir quelques comptes de Lab.

Exemples :

```text
adm-platform@monlabazure.fr
adm-network@monlabazure.fr
adm-security@monlabazure.fr
adm-operations@monlabazure.fr
```

Dans un environnement strictement nominatif, on préférera :

```text
adm-prenom.nom@monlabazure.fr
```

Pour un Lab personnel, des comptes par fonction peuvent être acceptés à des fins pédagogiques, mais ils ne doivent pas être présentés comme modèle Enterprise final.

---

# Ajout des Comptes Administratifs aux Groupes

Exemple avec un compte administrateur réseau.

## Récupérer l'Object ID du compte

```bash
ADMIN_NETWORK_ID=$(az ad user show \
  --id "adm-network@monlabazure.fr" \
  --query id \
  --output tsv)
```

## Ajouter le compte au groupe Network Admins

```bash
az ad group member add \
  --group "AZR-MLA-NETWORK-ADMINS" \
  --member-id "$ADMIN_NETWORK_ID"
```

## Vérifier les membres

```bash
az ad group member list \
  --group "AZR-MLA-NETWORK-ADMINS" \
  --query "[].{DisplayName:displayName, UserPrincipalName:userPrincipalName, ObjectId:id}" \
  --output table
```

Même logique pour les autres équipes :

```text
adm-platform@monlabazure.fr  → AZR-MLA-PLATFORM-ADMINS
adm-network@monlabazure.fr   → AZR-MLA-NETWORK-ADMINS
adm-security@monlabazure.fr  → AZR-MLA-SECURITY-ADMINS
adm-operations@monlabazure.fr → AZR-MLA-OPERATIONS
```

---

# Comptes Break Glass

Les comptes Break Glass doivent être traités différemment des comptes administratifs classiques.

Ils sont conçus pour un scénario extrême :

```text
Les accès normaux ne fonctionnent plus.
```

Exemples :

* mauvaise politique Conditional Access ;
* blocage MFA généralisé ;
* problème de fédération ;
* incident Entra Connect ou Cloud Sync ;
* suppression accidentelle des administrateurs ;
* verrouillage du tenant.

Microsoft indique que les comptes d’accès d’urgence doivent être utilisés uniquement en cas d’urgence ou scénario “break glass”, quand les comptes d’administration normaux ne peuvent pas être utilisés.

---

# Nombre de Comptes Break Glass

MonLabAzure utilise deux comptes Break Glass :

```text
bg-admin-01@monlabazure.onmicrosoft.com
bg-admin-02@monlabazure.onmicrosoft.com
```

Pourquoi deux ?

Parce qu'un seul compte constitue un point de défaillance unique.

Deux comptes permettent :

* redondance ;
* continuité administrative ;
* séparation de conservation ;
* réduction du risque de perte d'accès.

---

# Caractéristiques Obligatoires des Comptes Break Glass

Les comptes Break Glass doivent être :

* cloud-only ;
* non synchronisés depuis Active Directory local ;
* indépendants d'Entra Connect Sync ;
* indépendants d'Entra Cloud Sync ;
* indépendants d'AD FS ;
* indépendants d'un fournisseur SSO externe ;
* protégés par une méthode d'authentification forte ;
* exclus des politiques Conditional Access bloquantes ;
* surveillés ;
* testés régulièrement ;
* documentés.

Microsoft recommande aussi que les informations d'identification ou appareils associés aux comptes d’urgence n’expirent pas et ne soient pas supprimés automatiquement pour cause d’inactivité.

---

# Rôle des Comptes Break Glass

Dans MonLabAzure, les comptes Break Glass reçoivent le rôle :

```text
Global Administrator
```

Ce rôle est volontairement très élevé.

Justification :

```text
Le compte doit pouvoir récupérer l'accès au tenant en situation d'urgence.
```

Dans un environnement avec PIM, Microsoft recommande que le rôle Global Administrator des comptes d’urgence soit actif et permanent, et non simplement éligible, afin que le compte reste utilisable même en situation de panne ou de blocage des mécanismes normaux.

---

# Création des Comptes Break Glass

La création peut être faite depuis le portail Microsoft Entra.

Chemin :

```text
Microsoft Entra admin center
        │
        ▼
Identity
        │
        ▼
Users
        │
        ▼
New user
        │
        ▼
Create new user
```

Créer les deux comptes :

```text
bg-admin-01@monlabazure.onmicrosoft.com
bg-admin-02@monlabazure.onmicrosoft.com
```

Recommandations :

* utiliser le domaine `onmicrosoft.com` ;
* définir un mot de passe très fort ;
* ne pas exiger un changement de mot de passe à la première connexion si cela peut bloquer l'usage d'urgence ;
* stocker les informations d'accès dans un coffre sécurisé ;
* limiter strictement les personnes autorisées à utiliser ces comptes.

---

# Affecter le Rôle Global Administrator

Depuis Microsoft Entra admin center :

```text
Identity
        │
        ▼
Roles & admins
        │
        ▼
Global Administrator
        │
        ▼
Add assignments
```

Ajouter :

```text
bg-admin-01@monlabazure.onmicrosoft.com
bg-admin-02@monlabazure.onmicrosoft.com
```

Point important :

```text
Ces comptes ne doivent pas être ajoutés aux groupes RBAC standards.
```

Ils doivent rester séparés du modèle d'administration quotidien.

---

# Conditional Access et Break Glass

Les comptes Break Glass doivent être exclus des politiques Conditional Access qui pourraient bloquer leur utilisation.

Exemples de politiques pouvant bloquer :

* restriction géographique ;
* exigence d'appareil conforme ;
* exigence d'appareil hybride joint ;
* MFA dépendant d'un téléphone indisponible ;
* blocage de toutes les applications ;
* politique mal configurée.

Microsoft recommande d’exclure les comptes d’urgence ou Break Glass des politiques Conditional Access afin d’éviter un verrouillage complet des administrateurs après une mauvaise configuration.

Cette exclusion ne signifie pas absence de sécurité.

Elle doit être compensée par :

* authentification forte ;
* surveillance ;
* alertes ;
* test régulier ;
* journalisation ;
* revue post-utilisation.

---

# Authentification Forte pour Break Glass

Les comptes Break Glass ne doivent pas dépendre de la même méthode MFA que les comptes administratifs standards.

Exemple :

```text
Comptes administratifs standards
        │
        ▼
Microsoft Authenticator

Comptes Break Glass
        │
        ▼
Clé de sécurité FIDO2 ou méthode forte distincte
```

Microsoft recommande d’utiliser une authentification forte pour les comptes d’urgence et d’éviter la même méthode d’authentification que celle utilisée par les autres comptes administratifs.

Dans un Lab, la méthode exacte dépendra du matériel et des licences disponibles.

L'important est de retenir le principe :

```text
Le compte Break Glass doit rester utilisable même si le mécanisme MFA standard est indisponible.
```

---

# Surveillance des Comptes Break Glass

Toute connexion à un compte Break Glass doit être considérée comme un événement critique.

Surveillance attendue :

* logs de connexion Microsoft Entra ID ;
* logs d'audit ;
* alertes Azure Monitor ;
* notification à l'équipe sécurité ou plateforme ;
* revue post-utilisation.

Microsoft recommande de surveiller les activités de connexion et les journaux d’audit des comptes d’urgence, puis de déclencher des notifications aux administrateurs.

---

# Exemple de Contrôles à Mettre en Place

Contrôles recommandés :

```text
Alerte si connexion réussie avec bg-admin-01
Alerte si connexion réussie avec bg-admin-02
Alerte si échec répété de connexion
Alerte si modification du rôle Global Administrator
Alerte si modification des méthodes d'authentification
Alerte si ajout du compte à un groupe non prévu
Alerte si changement du mot de passe
```

Ces contrôles seront affinés dans les séries liées aux logs Entra ID et à Azure Monitor.

---

# Test Régulier des Comptes Break Glass

Un compte Break Glass non testé peut donner une fausse impression de sécurité.

Fréquence recommandée :

```text
Tous les 3 à 6 mois
```

Le test doit vérifier :

* connexion possible ;
* rôle Global Administrator actif ;
* méthode d'authentification fonctionnelle ;
* alerte déclenchée ;
* accès au tenant vérifié ;
* procédure documentée ;
* absence d'utilisation non autorisée.

---

# Procédure de Test Break Glass

Exemple de procédure :

```text
1. Planifier un test contrôlé.
2. Informer les personnes autorisées.
3. Se connecter avec bg-admin-01.
4. Vérifier l'accès au centre d'administration Microsoft Entra.
5. Vérifier le rôle Global Administrator.
6. Vérifier que l'alerte de connexion est déclenchée.
7. Se déconnecter.
8. Documenter le test.
9. Répéter avec bg-admin-02.
```

Le test ne doit pas servir à effectuer des tâches administratives normales.

---

# Journal de Test

Chaque test doit être documenté.

Exemple :

| Date       | Compte      | Test réalisé par | Résultat | Alerte reçue | Commentaire   |
| ---------- | ----------- | ---------------- | -------- | ------------ | ------------- |
| 2026-06-08 | bg-admin-01 | Platform Team    | OK       | Oui          | Test contrôlé |
| 2026-06-08 | bg-admin-02 | Platform Team    | OK       | Oui          | Test contrôlé |

---

# Stockage des Identifiants

Les identifiants ou méthodes d'accès Break Glass doivent être stockés dans un coffre sécurisé.

Options possibles :

* coffre-fort physique ;
* coffre numérique d'entreprise ;
* solution PAM ;
* stockage scellé avec double contrôle ;
* séparation entre deux personnes autorisées.

Règles :

* accès limité ;
* traçabilité ;
* procédure d'ouverture ;
* rotation ou vérification périodique ;
* pas de stockage dans un fichier texte ;
* pas de stockage dans un dépôt Git ;
* pas de stockage dans une boîte mail.

---

# Break Glass et Identité Hybride

Dans un environnement hybride, les comptes Break Glass doivent rester indépendants de l'infrastructure locale.

Ils ne doivent pas dépendre de :

* Active Directory local ;
* Entra Connect Sync ;
* Entra Cloud Sync ;
* AD FS ;
* VPN ;
* DNS interne ;
* infrastructure réseau on-premise.

Pourquoi ?

Parce qu'un incident hybride peut être précisément la raison pour laquelle le compte Break Glass doit être utilisé.

Exemple :

```text
AD FS indisponible
        │
        ▼
Les administrateurs fédérés ne peuvent plus se connecter
        │
        ▼
Le compte Break Glass cloud-only reste utilisable
```

---

# Break Glass et Groupes RBAC

Les comptes Break Glass ne doivent pas être membres des groupes suivants :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-OPERATIONS
AZR-MLA-DEVELOPERS
AZR-MLA-ARCHITECTS
AZR-MLA-READERS
```

Ils ne sont pas faits pour participer au modèle RBAC quotidien.

Ils ont un rôle Entra ID d'urgence.

Le groupe :

```text
AZR-MLA-BREAKGLASS-MONITORING
```

sert à organiser la supervision, pas à donner des droits d'urgence.

---

# Ce que Nous Ne Faisons Pas Encore

Dans cet article, nous ne configurons pas encore complètement :

* Conditional Access ;
* PIM ;
* Access Reviews ;
* export des logs Entra ID ;
* alertes Azure Monitor détaillées ;
* Microsoft Sentinel.

Ces sujets arrivent dans les séries suivantes.

Cependant, nous définissons déjà les règles qui devront être respectées quand ces mécanismes seront mis en place.

---

# Points de Contrôle

Avant de passer à l'article suivant, vérifier que :

* les comptes administratifs sont séparés des comptes standards ;
* les comptes administratifs sont ajoutés aux bons groupes ;
* les comptes administratifs ne reçoivent pas de rôles directs inutiles ;
* deux comptes Break Glass sont créés ;
* les comptes Break Glass utilisent le domaine `onmicrosoft.com` ;
* les comptes Break Glass sont cloud-only ;
* les comptes Break Glass disposent du rôle Global Administrator ;
* les comptes Break Glass ne sont pas membres des groupes RBAC standards ;
* les comptes Break Glass sont exclus des politiques Conditional Access bloquantes ;
* les méthodes d'authentification sont documentées ;
* une procédure de test est définie ;
* une surveillance minimale est prévue.

---

# Dépannage

## Le Compte Break Glass Dépend d'AD Local

Problème :

```text
Le compte est synchronisé depuis Active Directory local.
```

Correction :

```text
Créer un compte cloud-only dans Microsoft Entra ID.
```

Un compte Break Glass ne doit pas dépendre d'une synchronisation hybride.

---

## Le Compte Break Glass est Bloqué par Conditional Access

Problème :

```text
Le compte ne peut pas se connecter en raison d'une policy Conditional Access.
```

Correction :

```text
Exclure explicitement les comptes Break Glass des politiques bloquantes.
```

Cette exclusion doit être contrôlée et compensée par la surveillance.

---

## Le Compte Break Glass est Utilisé pour l'Administration Courante

Problème :

```text
Le compte sert à administrer Azure au quotidien.
```

Correction :

```text
Utiliser un compte administratif nominatif.
Réserver Break Glass aux urgences.
```

---

## Le Compte Break Glass n'est Jamais Testé

Problème :

```text
Personne ne sait si le compte fonctionne réellement.
```

Correction :

```text
Mettre en place un test contrôlé tous les 3 à 6 mois.
```

---

# Coûts

La création de comptes administratifs ou Break Glass ne génère pas de coût Azure direct.

Cependant, certaines protections peuvent dépendre de licences ou solutions complémentaires :

* Conditional Access ;
* PIM ;
* Identity Protection ;
* Access Reviews ;
* solutions PAM ;
* Microsoft Sentinel ;
* stockage sécurisé d'entreprise.

Dans MonLabAzure, nous commençons par la stratégie et les comptes.

Les mécanismes avancés seront traités progressivement.

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* distingué les comptes standards, administratifs et Break Glass ;
* défini la convention de nommage ;
* défini les règles d'utilisation des comptes administratifs ;
* défini les règles des comptes Break Glass ;
* créé le cadre pour deux comptes Break Glass cloud-only ;
* défini leur rôle Global Administrator ;
* cadré leur exclusion Conditional Access ;
* cadré leur surveillance ;
* défini une procédure de test ;
* préparé les prochaines séries sur MFA, Conditional Access et PIM.

---

# Conclusion

Les comptes administratifs et les comptes Break Glass sont des composants critiques de la stratégie IAM.

Les comptes administratifs permettent d'exploiter la plateforme de manière contrôlée et traçable.

Les comptes Break Glass garantissent une continuité administrative en cas de perte d'accès aux mécanismes standards.

La règle principale est simple :

```text
Un compte administratif sert à administrer.
Un compte Break Glass sert uniquement à récupérer l'accès en urgence.
```

Dans le prochain article, nous renforcerons ce modèle avec l'authentification forte, le MFA et les méthodes résistantes au phishing.