---

title: "ADR-0.7 - Stratégie IAM, RBAC, Break Glass et Identité Hybride"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "ADR"
* "ADR-007"
* "Phase-3"
* "Identity"
* "IAM"
* "RBAC"
* "Microsoft Entra ID"
* "Break Glass"
* "Conditional Access"
* "PIM"
* "Access Reviews"
* "Hybrid Identity"
* "Entra Connect"
* "Cloud Sync"
* "Security"
* "Azure"

---

# ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride

## Statut

Validé

---

# Date

Juin 2026

---

# Projet

MonLabAzure Cloud Platform

---

# 1. Contexte

Les Phases 1 et 2 du projet MonLabAzure Cloud Platform ont permis de construire les premières fondations de la plateforme Azure :

* Management Groups ;
* abonnements Azure ;
* backend Terraform ;
* organisation des states ;
* Azure Policies ;
* initiative de gouvernance ;
* premières règles de conformité.

La plateforme est désormais structurée et gouvernée.

La prochaine étape consiste à sécuriser les identités, les accès et les privilèges.

Dans un environnement Azure Enterprise, la gestion des identités est un sujet central. Une mauvaise gestion IAM peut entraîner :

* des droits excessifs ;
* des accès permanents inutiles ;
* une mauvaise séparation des responsabilités ;
* une exposition de la production ;
* une difficulté à auditer les actions ;
* un risque de verrouillage du tenant ;
* une mauvaise gestion des comptes invités ;
* une mauvaise maîtrise des identités machines ;
* une faiblesse dans les environnements hybrides.

Cet ADR définit le modèle cible d'identité et d'accès pour MonLabAzure Cloud Platform.

Il servira de référence pour la **Phase 3 – Identity & Access Management**.

---

# 2. Problème

Comment organiser les identités et les accès Azure de manière :

* sécurisée ;
* gouvernable ;
* auditable ;
* compatible avec une approche Enterprise ;
* adaptée à un Lab personnel ;
* évolutive vers des scénarios avancés comme PIM, Conditional Access, Access Reviews et identité hybride ?

---

# 3. Objectifs

La stratégie IAM doit permettre de :

* appliquer le principe du moindre privilège ;
* éviter les affectations directes de rôles aux utilisateurs ;
* séparer les responsabilités entre équipes ;
* sécuriser les accès administratifs ;
* mettre en place des comptes Break Glass ;
* préparer les comptes de service et les Managed Identities ;
* intégrer les contraintes de licences Microsoft Entra ID ;
* préparer les scénarios PIM, Conditional Access et Access Reviews ;
* intégrer les scénarios d'identité hybride ;
* permettre l'audit et la supervision des accès ;
* préparer les futures phases réseau, sécurité, supervision, applicative, AKS et APIM.

---

# 4. Décision

MonLabAzure Cloud Platform adopte une stratégie IAM basée sur les principes suivants :

* Microsoft Entra ID comme référentiel d'identité ;
* attribution des rôles Azure via des groupes Microsoft Entra ID ;
* séparation entre comptes standards, comptes administratifs, comptes de service, Managed Identities et comptes Break Glass ;
* application du moindre privilège ;
* utilisation limitée des rôles à privilèges élevés ;
* surveillance obligatoire des comptes sensibles ;
* prise en compte des licences Microsoft Entra ID ;
* prise en compte des environnements hybrides ;
* introduction progressive des fonctionnalités avancées IAM.

---

# 5. Principes de Sécurité

## 5.1 Aucun rôle directement attribué à un utilisateur

Les rôles Azure RBAC devront être attribués à des groupes Microsoft Entra ID.

Modèle retenu :

```text
Utilisateur
    │
    ▼
Groupe Microsoft Entra ID
    │
    ▼
Rôle Azure RBAC
    │
    ▼
Scope Azure
```

Les affectations directes à des utilisateurs seront interdites, sauf exception temporaire, documentée et approuvée.

---

## 5.2 Principe du moindre privilège

Chaque compte et chaque groupe doivent disposer uniquement des droits nécessaires à leur mission.

L'utilisation du rôle :

```text
Owner
```

doit être exceptionnelle.

Les rôles plus spécifiques devront être privilégiés :

```text
Reader
Contributor
Network Contributor
Security Reader
Monitoring Reader
Log Analytics Reader
User Access Administrator
```

Le rôle `Owner` ne doit jamais devenir le rôle par défaut d'administration.

---

## 5.3 Séparation des responsabilités

Les responsabilités d'administration doivent être séparées entre les différentes équipes :

* plateforme ;
* réseau ;
* sécurité ;
* opérations ;
* développement ;
* architecture.

Cette séparation réduit les risques liés aux erreurs humaines, aux compromissions et aux accès trop larges.

---

## 5.4 Traçabilité

Toute action d'administration doit pouvoir être rattachée à une identité identifiable.

Les comptes partagés sont interdits pour l'administration quotidienne.

---

## 5.5 Accès d'urgence contrôlé

Les comptes Break Glass ne sont pas des comptes d'administration quotidienne.

Ils constituent un mécanisme de continuité administrative permettant de récupérer l'accès au tenant en cas d'incident majeur.

---

# 6. Modèle d'Équipes

## 6.1 Platform Team

Responsable de la plateforme Azure globale :

* Management Groups ;
* abonnements ;
* Terraform ;
* gouvernance ;
* intégration des composants transverses.

---

## 6.2 Network Team

Responsable des composants réseau :

* Hub & Spoke ;
* Azure Firewall ;
* Bastion ;
* VPN ;
* ExpressRoute ;
* Private DNS ;
* routage.

---

## 6.3 Security Team

Responsable de la sécurité :

* Azure Policy ;
* Microsoft Defender for Cloud ;
* recommandations de sécurité ;
* conformité ;
* investigation sécurité ;
* contrôle des accès privilégiés.

---

## 6.4 Operations Team

Responsable de l'exploitation :

* supervision ;
* alertes ;
* incidents ;
* sauvegardes ;
* disponibilité.

---

## 6.5 Development Team

Responsable des workloads applicatifs :

* App Service ;
* API ;
* pipelines ;
* déploiements applicatifs ;
* tests en DEV et QUALIF.

---

## 6.6 Architecture Team

Responsable de la conception et de la validation :

* standards d'architecture ;
* ADR ;
* revues ;
* validation des designs ;
* arbitrages techniques.

---

# 7. Stratégie des Comptes

## 7.1 Comptes utilisateurs standards

Les comptes utilisateurs standards sont utilisés pour les activités quotidiennes non administratives.

Exemple :

```text
prenom.nom@monlabazure.fr
```

Usage :

* messagerie ;
* portail utilisateur ;
* consultation ;
* activités bureautiques.

Ces comptes ne doivent pas disposer de privilèges administratifs élevés.

---

## 7.2 Comptes administratifs nominatifs

Chaque administrateur doit disposer d'un compte administratif dédié.

Exemple :

```text
adm-prenom.nom@monlabazure.fr
```

Usage :

* administration Azure ;
* opérations privilégiées ;
* gestion de la plateforme.

Règles :

* compte nominatif ;
* aucune utilisation quotidienne non administrative ;
* MFA obligatoire ;
* droits attribués via groupes ;
* journalisation obligatoire ;
* accès à la production limité et justifié.

---

## 7.3 Comptes de service

Les comptes de service sont utilisés par des outils ou processus techniques.

Exemples :

```text
svc-terraform
svc-automation
svc-devops
```

Règles :

* usage clairement documenté ;
* permissions limitées ;
* secret stocké dans Key Vault lorsque nécessaire ;
* rotation documentée ;
* remplacement progressif par Managed Identity lorsque possible.

---

## 7.4 Managed Identities

Les Managed Identities seront privilégiées pour les workloads Azure lorsque cela est possible.

Objectifs :

* éviter les secrets statiques ;
* limiter les identifiants stockés ;
* intégrer nativement l'authentification Azure ;
* réduire les risques liés aux secrets applicatifs.

Les Managed Identities seront introduites progressivement dans les futures phases applicatives et sécurité.

---

# 8. Comptes Break Glass

## 8.1 Définition

Les comptes Break Glass sont des comptes d'urgence destinés à garantir un accès administratif minimal au tenant en cas d'incident majeur.

Ils sont utilisés uniquement lorsque les mécanismes d'administration standards ne sont plus disponibles.

Exemples de situations :

* erreur de configuration Conditional Access ;
* indisponibilité MFA ;
* perte d'accès des administrateurs ;
* problème de fédération ou de synchronisation ;
* incident majeur Microsoft Entra ID ;
* verrouillage accidentel du tenant.

---

## 8.2 Décision

Le projet MonLabAzure utilisera deux comptes Break Glass.

Exemples :

```text
bg-admin-01@monlabazure.onmicrosoft.com
bg-admin-02@monlabazure.onmicrosoft.com
```

---

## 8.3 Justification

Un seul compte Break Glass constitue un point de défaillance unique.

Deux comptes permettent :

* redondance ;
* continuité administrative ;
* réduction du risque de perte d'accès ;
* séparation éventuelle des responsabilités de conservation.

---

## 8.4 Caractéristiques obligatoires

Les comptes Break Glass doivent être :

* cloud-only ;
* non synchronisés depuis un annuaire local ;
* indépendants d'un fournisseur fédéré ;
* indépendants d'un SSO externe ;
* protégés par un mot de passe très fort ;
* stockés dans un coffre sécurisé ;
* utilisés uniquement en situation d'urgence.

---

## 8.5 Conditional Access

Les comptes Break Glass doivent être exclus des politiques Conditional Access bloquantes susceptibles d'empêcher leur utilisation en situation d'urgence.

Cette exclusion ne signifie pas absence de contrôle.

Elle doit être compensée par :

* une surveillance renforcée ;
* des alertes critiques ;
* une procédure d'utilisation documentée ;
* des tests réguliers ;
* une revue post-utilisation obligatoire.

---

## 8.6 Rôle attribué

Dans le cadre du projet MonLabAzure, les comptes Break Glass disposeront du rôle :

```text
Global Administrator
```

afin de garantir un accès d'urgence au tenant.

Dans un environnement Enterprise mature, cette décision devra être réévaluée avec :

* Microsoft Entra ID PIM ;
* les exigences de conformité ;
* les politiques internes de sécurité ;
* les contraintes d'audit.

---

## 8.7 Surveillance

Toute connexion avec un compte Break Glass doit générer une alerte critique.

Surveillance attendue :

* logs de connexion Microsoft Entra ID ;
* alertes Azure Monitor ;
* notification équipe sécurité ;
* revue post-incident obligatoire.

Une connexion Break Glass doit être considérée comme anormale jusqu'à preuve du contraire.

---

## 8.8 Procédure de test

Les comptes Break Glass doivent être testés régulièrement.

Fréquence recommandée :

```text
Tous les 3 à 6 mois
```

Contrôles :

* connexion possible ;
* rôle actif ;
* alerte déclenchée ;
* accès au tenant vérifié ;
* procédure documentée ;
* absence d'utilisation non autorisée.

---

## 8.9 Règles d'utilisation

Les comptes Break Glass ne doivent jamais être utilisés pour :

* administration quotidienne ;
* tests classiques ;
* déploiements Terraform ;
* opérations applicatives ;
* contournement des processus normaux.

Ils doivent être utilisés uniquement pour :

```text
Incident majeur
```

ou :

```text
Perte d'accès administratif standard
```

---

# 9. Licences Microsoft Entra ID

La stratégie IAM de MonLabAzure Cloud Platform dépend des fonctionnalités disponibles dans Microsoft Entra ID.

Certaines fonctionnalités de base sont disponibles dans les tenants Microsoft 365 ou Azure standards, mais les fonctionnalités avancées de sécurité, de gouvernance et d'administration nécessitent des licences Microsoft Entra ID Premium.

Cette distinction influence directement la roadmap de la Phase 3.

---

## 9.1 Microsoft Entra ID Free

Microsoft Entra ID Free fournit les fonctionnalités de base d'identité.

Fonctionnalités typiques :

* gestion des utilisateurs ;
* gestion des groupes ;
* authentification de base ;
* gestion du tenant ;
* intégration avec les services Microsoft Cloud ;
* sécurité de base.

Ce niveau peut suffire pour un Lab simple, mais il reste limité pour une architecture Enterprise.

Limites importantes :

* pas de Conditional Access avancé ;
* pas de PIM ;
* pas d'Access Reviews ;
* gouvernance limitée ;
* automatisation IAM avancée limitée.

Dans MonLabAzure, ce niveau peut permettre de démarrer les premiers exercices IAM, mais il ne couvre pas les besoins avancés d'une plateforme Enterprise.

---

## 9.2 Microsoft Entra ID P1

Microsoft Entra ID P1 ajoute des fonctionnalités importantes pour les environnements professionnels.

Fonctionnalités importantes :

* Conditional Access ;
* groupes dynamiques ;
* fonctionnalités avancées de gestion des groupes ;
* intégration hybride plus avancée ;
* Self-Service Password Reset avancé ;
* accès conditionnel basé sur des règles ;
* meilleure gestion des identités dans les environnements d'entreprise.

Dans MonLabAzure, Entra ID P1 devient pertinent dès que nous souhaitons aborder :

* Conditional Access ;
* groupes dynamiques ;
* automatisation de l'appartenance aux groupes ;
* premiers contrôles d'accès avancés.

Exemple :

```text
Utilisateur membre du département Cloud
        │
        ▼
Ajout automatique dans un groupe dynamique
        │
        ▼
Affectation RBAC ou accès applicatif
```

P1 constitue le premier niveau réellement adapté à une plateforme d'entreprise.

---

## 9.3 Microsoft Entra ID P2

Microsoft Entra ID P2 inclut les fonctionnalités de P1 et ajoute les capacités avancées de sécurité et de gouvernance.

Fonctionnalités importantes :

* Privileged Identity Management ;
* Identity Protection ;
* Access Reviews ;
* accès basé sur le risque ;
* gouvernance avancée des accès ;
* meilleure gestion des accès privilégiés.

Dans MonLabAzure, Entra ID P2 sera nécessaire pour traiter correctement :

* PIM ;
* élévation temporaire des privilèges ;
* revue périodique des accès ;
* détection des risques utilisateurs ;
* gouvernance avancée des administrateurs ;
* réduction des droits permanents.

Exemple :

```text
Administrateur
        │
        ▼
Rôle éligible via PIM
        │
        ▼
Activation temporaire avec justification
        │
        ▼
Expiration automatique du privilège
```

P2 correspond au niveau attendu pour un environnement Enterprise mature.

---

## 9.4 Microsoft Entra ID Governance

Microsoft Entra ID Governance est orienté gouvernance du cycle de vie des identités et des accès.

Il couvre notamment des scénarios comme :

* access packages ;
* entitlement management ;
* lifecycle workflows ;
* gouvernance des invités ;
* automatisation des demandes d'accès ;
* processus d'approbation ;
* revues d'accès avancées selon les scénarios.

Ce niveau devient pertinent lorsque l'organisation souhaite industrialiser :

* l'onboarding ;
* l'offboarding ;
* l'accès des prestataires ;
* les accès temporaires ;
* les workflows de validation ;
* la gouvernance des accès applicatifs.

Dans MonLabAzure, ce sujet pourra être abordé plus tard, lorsque la plateforme accueillera davantage d'applications, d'équipes et de rôles.

---

## 9.5 Microsoft Entra Suite

Microsoft Entra Suite regroupe plusieurs capacités avancées autour de l'identité, de l'accès réseau, de la gouvernance et de la vérification d'identité.

Elle devient pertinente pour les organisations qui souhaitent aller au-delà de l'IAM classique et couvrir des scénarios plus larges autour du Zero Trust.

Dans le cadre de MonLabAzure, Microsoft Entra Suite n'est pas nécessaire pour la première version de la plateforme.

Elle pourra être mentionnée dans les phases avancées si le projet évolue vers des scénarios plus complets de sécurité, d'accès réseau ou de gouvernance d'identité.

---

## 9.6 Synthèse des licences

| Fonctionnalité          | Free |  P1 |            P2 | Gouvernance avancée |
| ----------------------- | ---: | --: | ------------: | ------------------: |
| Utilisateurs et groupes |  Oui | Oui |           Oui |                 Oui |
| RBAC Azure via groupes  |  Oui | Oui |           Oui |                 Oui |
| Groupes dynamiques      |  Non | Oui |           Oui |                 Oui |
| Conditional Access      |  Non | Oui |           Oui |                 Oui |
| PIM                     |  Non | Non |           Oui |                 Oui |
| Identity Protection     |  Non | Non |           Oui |                 Oui |
| Access Reviews          |  Non | Non |           Oui |                 Oui |
| Entitlement Management  |  Non | Non | Selon licence |                 Oui |
| Lifecycle Workflows     |  Non | Non | Selon licence |                 Oui |

---

## 9.7 Positionnement pour MonLabAzure

Pour conserver une approche réaliste et accessible, MonLabAzure adoptera la progression suivante :

```text
Phase 3 – IAM Foundation
        │
        ├── Groupes Entra ID
        ├── RBAC Azure
        ├── Comptes administratifs
        ├── Comptes de service
        └── Comptes Break Glass

Phases IAM avancées
        │
        ├── Conditional Access
        ├── Groupes dynamiques
        ├── Access Reviews
        ├── PIM
        └── Identity Protection
```

La Phase 3 initiale ne dépendra pas obligatoirement d'Entra ID P2.

Cela permet de rester compatible avec un Lab personnel, tout en préparant clairement l'évolution vers un modèle Enterprise plus mature.

---

# 10. Identité Hybride : Entra Connect Sync et Entra Cloud Sync

Dans de nombreuses organisations, Microsoft Entra ID n'est pas utilisé seul.

La majorité des environnements Enterprise disposent encore d'un Active Directory local qui reste la source principale des identités utilisateurs, des groupes, des comptes de service et parfois des appareils.

Dans ce contexte, l'identité hybride devient un composant critique de l'architecture IAM.

---

## 10.1 Objectif

L'objectif de cette section est de positionner clairement le rôle de l'identité hybride dans MonLabAzure Cloud Platform.

Même si le Lab MonLabAzure peut être réalisé en mode cloud-only, l'architecture cible doit rester compatible avec les scénarios Enterprise hybrides.

---

## 10.2 Deux modèles principaux

Microsoft propose principalement deux approches pour synchroniser les identités entre Active Directory local et Microsoft Entra ID :

```text
Active Directory local
        │
        ▼
Microsoft Entra ID
```

Les deux solutions principales sont :

* Microsoft Entra Connect Sync ;
* Microsoft Entra Cloud Sync.

---

## 10.3 Microsoft Entra Connect Sync

Microsoft Entra Connect Sync est la solution historique de synchronisation d'identités entre Active Directory local et Microsoft Entra ID.

Elle permet de synchroniser :

* les utilisateurs ;
* les groupes ;
* certains attributs ;
* les objets nécessaires aux scénarios hybrides ;
* les informations utiles à l'authentification selon le mode retenu.

---

## 10.4 Cas d'usage typiques Entra Connect Sync

Microsoft Entra Connect Sync reste pertinent lorsque l'organisation dispose :

* d'un Active Directory historique complexe ;
* de règles de synchronisation avancées ;
* de plusieurs forêts ;
* de besoins de filtrage avancés ;
* de scénarios hybrides déjà en production ;
* d'une forte dépendance aux attributs AD locaux.

---

## 10.5 Points d'attention Entra Connect Sync

Microsoft Entra Connect Sync est un composant critique.

Il doit être :

* supervisé ;
* maintenu à jour ;
* sauvegardé ;
* documenté ;
* audité régulièrement.

Un problème sur Entra Connect Sync peut entraîner :

* retard de synchronisation ;
* comptes non mis à jour ;
* groupes obsolètes ;
* erreurs d'authentification ;
* problème d'onboarding ou d'offboarding.

---

## 10.6 Microsoft Entra Cloud Sync

Microsoft Entra Cloud Sync est une approche plus moderne, légère et pilotée depuis le cloud.

Elle repose sur un agent installé dans l'environnement local.

Elle permet de synchroniser les utilisateurs, groupes et contacts depuis Active Directory vers Microsoft Entra ID.

---

## 10.7 Cas d'usage typiques Entra Cloud Sync

Microsoft Entra Cloud Sync est particulièrement intéressant pour :

* les environnements plus simples ;
* les nouvelles forêts Active Directory ;
* les scénarios multi-forêts ;
* les forêts déconnectées ;
* les organisations souhaitant réduire la complexité locale ;
* les migrations progressives depuis Entra Connect Sync.

---

## 10.8 Intérêt architectural de Cloud Sync

Cloud Sync réduit la dépendance à un serveur de synchronisation lourdement personnalisé.

Il facilite une approche plus cloud-managed de l'identité hybride.

Cependant, il ne remplace pas automatiquement tous les scénarios avancés de Microsoft Entra Connect Sync.

Le choix doit être réalisé selon les besoins réels de l'organisation.

---

## 10.9 Comparaison synthétique

| Critère                   | Entra Connect Sync         | Entra Cloud Sync             |
| ------------------------- | -------------------------- | ---------------------------- |
| Maturité historique       | Très élevée                | Plus moderne                 |
| Gestion cloud             | Limitée                    | Plus forte                   |
| Agent léger               | Non                        | Oui                          |
| Personnalisation avancée  | Plus complète              | Plus limitée selon scénarios |
| Scénarios complexes AD    | Adapté                     | À valider                    |
| Multi-forêts simples      | Possible                   | Très pertinent               |
| Migration progressive     | Source existante fréquente | Cible possible               |
| Complexité opérationnelle | Plus élevée                | Plus faible                  |

---

## 10.10 Modes d'authentification hybride

Dans une architecture hybride, la synchronisation des identités ne suffit pas.

Il faut également définir le mode d'authentification.

Les principaux modèles sont :

* Password Hash Synchronization ;
* Pass-through Authentication ;
* Federation.

---

## 10.11 Password Hash Synchronization

Les hash de mot de passe sont synchronisés vers Microsoft Entra ID.

Avantages :

* simplicité ;
* résilience ;
* moins de dépendance au réseau local ;
* recommandé dans de nombreux scénarios cloud-first.

---

## 10.12 Pass-through Authentication

L'authentification est validée par des agents dans l'environnement local.

Avantages :

* validation locale ;
* pas de dépendance à une infrastructure AD FS.

Points d'attention :

* dépendance aux agents ;
* dépendance à la disponibilité locale ;
* supervision obligatoire.

---

## 10.13 Federation

L'authentification repose sur une infrastructure fédérée, par exemple AD FS.

Avantages :

* contrôle fort local ;
* scénarios historiques complexes.

Points d'attention :

* complexité ;
* dépendance forte à l'infrastructure AD FS ;
* supervision critique ;
* modèle souvent à réévaluer dans les architectures modernes.

---

## 10.14 Décision pour MonLabAzure

Dans la première version du Lab MonLabAzure, l'identité sera traitée en mode cloud-only afin de conserver une mise en œuvre simple, accessible et reproductible.

Cependant, l'architecture cible reconnaît officiellement que les environnements Enterprise utilisent fréquemment un modèle hybride.

La Phase 3 inclura donc un article dédié à l'identité hybride afin de présenter :

* Microsoft Entra Connect Sync ;
* Microsoft Entra Cloud Sync ;
* les modes d'authentification hybrides ;
* les critères de choix ;
* les risques opérationnels ;
* les points de supervision ;
* les implications de sécurité.

---

## 10.15 Contrôles d'audit hybride

Dans un environnement hybride, les contrôles suivants doivent être prévus :

* version de Microsoft Entra Connect Sync ;
* état de synchronisation ;
* erreurs de synchronisation ;
* fraîcheur des dernières synchronisations ;
* mode d'authentification utilisé ;
* statut des agents Cloud Sync ou Pass-through Authentication ;
* dépendance à AD FS le cas échéant ;
* comptes synchronisés à privilèges ;
* groupes synchronisés utilisés pour le RBAC Azure ;
* cohérence du cycle de vie des comptes entre AD local et Microsoft Entra ID.

---

# 11. Groupes Microsoft Entra ID

Les groupes suivants sont définis comme base IAM de la plateforme.

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-OPERATIONS
AZR-MLA-DEVELOPERS
AZR-MLA-ARCHITECTS
AZR-MLA-READERS
AZR-MLA-BREAKGLASS-MONITORING
```

---

# 12. Description des Groupes

## 12.1 AZR-MLA-PLATFORM-ADMINS

Responsables de l'administration globale de la plateforme Azure.

Scopes possibles :

* Management Groups ;
* subscriptions de plateforme ;
* ressources Terraform.

---

## 12.2 AZR-MLA-NETWORK-ADMINS

Responsables de l'administration réseau.

Scopes possibles :

* Connectivity Subscription ;
* Hub VNet ;
* Firewall ;
* DNS ;
* Bastion ;
* VPN.

---

## 12.3 AZR-MLA-SECURITY-ADMINS

Responsables de la sécurité et de la conformité.

Scopes possibles :

* Azure Policy ;
* Defender for Cloud ;
* Security Center ;
* audit ;
* recommandations de sécurité.

---

## 12.4 AZR-MLA-OPERATIONS

Responsables de l'exploitation.

Scopes possibles :

* Monitoring ;
* Log Analytics ;
* alertes ;
* dashboards ;
* sauvegardes.

---

## 12.5 AZR-MLA-DEVELOPERS

Équipes de développement.

Scopes possibles :

* DEV ;
* certains périmètres QUALIF ;
* aucun accès privilégié direct en PROD.

---

## 12.6 AZR-MLA-ARCHITECTS

Équipe architecture.

Scopes possibles :

* lecture globale ;
* validation des designs ;
* revue d'architecture ;
* aucun droit opérationnel permanent par défaut.

---

## 12.7 AZR-MLA-READERS

Accès en lecture.

Usage :

* audit ;
* observation ;
* support ;
* reporting.

---

## 12.8 AZR-MLA-BREAKGLASS-MONITORING

Groupe ou mécanisme logique utilisé pour la surveillance des comptes Break Glass.

Usage :

* alerting ;
* revue sécurité ;
* procédures d'urgence.

---

# 13. Groupes Dynamiques

Les groupes dynamiques Microsoft Entra ID permettent d'ajouter automatiquement des utilisateurs ou des appareils à des groupes selon des règles d'appartenance.

Exemples d'usage futur :

* ajout automatique des utilisateurs d'une équipe ;
* séparation des populations DEV, OPS et SEC ;
* regroupement automatique d'appareils conformes ;
* automatisation de certaines affectations d'accès.

Dans le cadre de MonLabAzure, les groupes dynamiques ne seront pas utilisés dans la première version opérationnelle de la Phase 3 afin de conserver une approche simple et contrôlée.

Ils seront étudiés lorsque le modèle IAM nécessitera plus d'automatisation.

---

# 14. Modèle RBAC Azure

## 14.1 Règle générale

Les rôles seront affectés selon la structure suivante :

```text
Management Group
    │
    ▼
Subscription
    │
    ▼
Resource Group
    │
    ▼
Resource
```

L'affectation au niveau le plus haut doit être limitée aux cas justifiés.

---

## 14.2 Niveau Tenant Root Group

Accès très limité.

Groupes autorisés :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-SECURITY-ADMINS
```

Rôles possibles :

```text
Reader
Security Reader
```

Les rôles élevés à ce niveau doivent rester exceptionnels.

---

## 14.3 Niveau mla-platform

Groupes principaux :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-OPERATIONS
```

Rôles possibles :

```text
Contributor
Network Contributor
Monitoring Contributor
Reader
```

---

## 14.4 Niveau mla-connectivity

Groupe principal :

```text
AZR-MLA-NETWORK-ADMINS
```

Rôles possibles :

```text
Network Contributor
Reader
```

---

## 14.5 Niveau mla-management

Groupes principaux :

```text
AZR-MLA-OPERATIONS
AZR-MLA-SECURITY-ADMINS
```

Rôles possibles :

```text
Monitoring Contributor
Log Analytics Contributor
Security Reader
Reader
```

---

## 14.6 Niveau mla-landingzones

Groupes principaux :

```text
AZR-MLA-DEVELOPERS
AZR-MLA-OPERATIONS
AZR-MLA-READERS
```

Rôles possibles :

```text
Reader
Contributor
Monitoring Reader
```

---

## 14.7 Niveau mla-prod

Règles spécifiques :

* aucun accès développeur privilégié permanent ;
* accès production limité aux opérations ;
* accès en lecture possible pour architecture et sécurité ;
* toute élévation de privilège doit être documentée.

Groupes possibles :

```text
AZR-MLA-OPERATIONS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-ARCHITECTS
```

---

# 15. Rôles Personnalisés

Les rôles Azure intégrés seront privilégiés au démarrage.

Les rôles personnalisés seront créés uniquement si :

* aucun rôle intégré ne répond correctement au besoin ;
* le périmètre est stable ;
* le besoin est récurrent ;
* la règle est documentée.

Les rôles personnalisés seront traités progressivement dans les articles de la Phase 3.

---

# 16. Comptes Terraform et Automatisation

Terraform doit être exécuté avec une identité dédiée.

Options possibles :

```text
Service Principal
Managed Identity
Federated Identity
```

Pour le Lab MonLabAzure, l'approche initiale pourra utiliser un Service Principal dédié :

```text
svc-terraform
```

À terme, l'approche recommandée sera :

```text
Federated Identity
```

ou :

```text
Managed Identity
```

selon le contexte CI/CD.

Règles :

* pas d'utilisation d'un compte personnel pour les pipelines ;
* permissions limitées ;
* secret stocké de manière sécurisée ;
* rotation documentée ;
* préférer les identités managées lorsque possible.

---

# 17. Fonctionnalités IAM Avancées

Le modèle IAM défini dans cet ADR constitue la première version de la stratégie d'identité et d'accès de MonLabAzure Cloud Platform.

Certaines fonctionnalités avancées de Microsoft Entra ID seront étudiées dans des phases ultérieures, lorsque leur usage deviendra nécessaire dans le projet.

---

## 17.1 Privileged Identity Management

Microsoft Entra Privileged Identity Management permet de gérer les rôles privilégiés de manière temporaire et contrôlée.

Objectifs futurs :

* réduire les droits permanents ;
* activer les rôles à la demande ;
* imposer une justification ;
* limiter la durée des privilèges ;
* auditer les élévations de privilèges.

Dans le cadre du projet MonLabAzure, PIM sera étudié lorsque nous aborderons la sécurisation avancée des accès administratifs.

---

## 17.2 Conditional Access

Les politiques Conditional Access permettent de contrôler les conditions d'accès aux services Microsoft et Azure.

Objectifs futurs :

* imposer le MFA ;
* contrôler les accès selon le risque ;
* limiter les accès depuis des emplacements non approuvés ;
* renforcer les accès administratifs ;
* protéger les portails d'administration.

Les comptes Break Glass devront rester exclus des politiques Conditional Access bloquantes susceptibles d'empêcher leur usage en situation d'urgence.

---

## 17.3 Access Reviews

Les Access Reviews permettent de revoir périodiquement les accès accordés aux utilisateurs et aux groupes.

Objectifs futurs :

* détecter les accès obsolètes ;
* réduire les droits excessifs ;
* valider les accès à la production ;
* auditer les groupes sensibles ;
* améliorer la conformité.

Les Access Reviews seront particulièrement utiles pour les groupes administratifs, les accès production et les rôles élevés.

---

## 17.4 Identity Protection

Microsoft Entra ID Protection permet d'identifier certains risques liés aux utilisateurs et aux connexions.

Objectifs futurs :

* détecter les connexions risquées ;
* identifier les utilisateurs à risque ;
* déclencher des contrôles adaptés ;
* renforcer la stratégie Zero Trust.

---

# 18. Contrôles IAM Complémentaires

ADR-007 intègre également les contrôles suivants dans la roadmap IAM.

---

## 18.1 Couverture MFA des comptes administrateurs

Tous les comptes administratifs doivent utiliser une authentification multifacteur.

Les comptes privilégiés sans MFA doivent être considérés comme un risque critique.

---

## 18.2 Blocage ou audit de l'authentification legacy

Les protocoles d'authentification legacy doivent être audités puis bloqués lorsque cela est possible.

Exemples :

* IMAP ;
* POP ;
* SMTP AUTH ancien ;
* anciens clients Office ;
* protocoles ne supportant pas correctement MFA ou Conditional Access.

Ces protocoles peuvent contourner certains contrôles modernes.

---

## 18.3 Méthodes MFA phishing-resistant

Toutes les méthodes MFA n'offrent pas le même niveau de sécurité.

Les méthodes suivantes seront étudiées dans les phases avancées :

* FIDO2 ;
* passkeys ;
* Windows Hello for Business ;
* certificats ;
* méthodes résistantes au phishing.

Les méthodes plus faibles comme SMS ou appels vocaux devront être limitées lorsque cela est possible.

---

## 18.4 Export des logs Entra ID

Les logs Microsoft Entra ID devront être exportés vers une solution centralisée.

Cible future :

```text
Microsoft Entra ID Logs
        │
        ▼
Log Analytics Workspace
        │
        ▼
Azure Monitor / Sentinel
```

Objectifs :

* investigation ;
* audit ;
* détection ;
* conservation ;
* corrélation avec les autres logs Azure.

---

## 18.5 Gouvernance des comptes invités

Les comptes invités B2B devront être surveillés.

Contrôles futurs :

* volume de comptes invités ;
* ancienneté ;
* accès actifs ;
* appartenance aux groupes ;
* accès à la production ;
* revue périodique.

---

## 18.6 Audit des identités machines

Les identités machines devront être auditées régulièrement.

Objets concernés :

* Service Principals ;
* App Registrations ;
* Managed Identities ;
* comptes de service ;
* secrets applicatifs ;
* certificats expirants.

Les identités machines sont souvent sur-permissionnées et doivent être traitées comme des identités sensibles.

---

# 19. Surveillance et Audit

Les éléments suivants doivent être surveillés :

* connexions administratives ;
* affectations RBAC ;
* modifications de rôles ;
* élévations de privilèges ;
* utilisation des comptes Break Glass ;
* création ou suppression de comptes de service ;
* modifications Conditional Access ;
* changements sur les groupes sensibles ;
* création de secrets sur App Registrations ;
* changements de configuration Entra Connect ou Cloud Sync.

Les événements critiques devront être intégrés progressivement à Azure Monitor, Log Analytics et éventuellement Microsoft Sentinel.

---

# 20. Exceptions

Toute exception doit être :

* documentée ;
* justifiée ;
* limitée dans le temps ;
* validée par l'équipe Architecture ou Sécurité ;
* revue régulièrement.

Exemples d'exceptions :

* attribution temporaire Contributor ;
* accès d'urgence à PROD ;
* rôle élevé pour déploiement exceptionnel ;
* contournement contrôlé d'une règle Conditional Access ;
* accès invité temporaire ;
* compte de service avec privilège élevé.

---

# 21. Conséquences

## 21.1 Conséquences positives

* meilleure sécurité ;
* meilleure traçabilité ;
* séparation claire des responsabilités ;
* réduction des droits excessifs ;
* préparation aux audits ;
* continuité administrative via Break Glass ;
* prise en compte des environnements hybrides ;
* trajectoire claire vers PIM, Conditional Access et Access Reviews.

---

## 21.2 Conséquences négatives

* complexité plus élevée qu'un modèle simple ;
* besoin de documentation ;
* nécessité de maintenir les groupes ;
* besoin de surveiller les comptes sensibles ;
* gestion plus stricte des accès temporaires ;
* dépendance possible à des licences Microsoft Entra ID avancées ;
* complexité supplémentaire dans les environnements hybrides.

---

# 22. Alternatives Étudiées

## 22.1 Attribution directe aux utilisateurs

Rejetée.

Elle rend l'audit difficile et augmente les risques d'erreur.

---

## 22.2 Un seul groupe Administrateurs Azure

Rejeté.

Cette approche ne respecte pas la séparation des responsabilités.

---

## 22.3 Un seul compte Break Glass

Rejeté.

Il constitue un point de défaillance unique.

---

## 22.4 Aucun compte Break Glass

Rejeté.

Cela expose l'organisation à un risque de perte totale d'accès administratif.

---

## 22.5 Ignorer les environnements hybrides

Rejeté.

Même si MonLabAzure démarre en cloud-only, les environnements Enterprise utilisent très fréquemment Active Directory local avec Entra Connect Sync ou Cloud Sync.

---

## 22.6 Imposer PIM dès le départ

Rejeté pour la première version du Lab.

PIM reste une cible importante, mais son usage dépend des licences et de la maturité IAM.

---

# 23. Critères de Validation

ADR-007 sera considéré comme appliqué lorsque :

* les groupes Microsoft Entra ID seront créés ;
* les rôles seront attribués aux groupes et non aux utilisateurs ;
* les comptes administratifs seront séparés des comptes standards ;
* deux comptes Break Glass seront créés ;
* les comptes Break Glass seront surveillés ;
* une procédure de test Break Glass sera documentée ;
* les premières affectations RBAC seront validées ;
* les licences Entra ID nécessaires seront identifiées ;
* les scénarios hybrides seront documentés ;
* les contrôles MFA, legacy authentication, invités et identités machines seront intégrés à la roadmap.

---

# 24. Roadmap Phase 3

La Phase 3 sera organisée comme suit :

```text
Phase 3 – Identity & Access Management

ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride

SERIES-3.1 – Introduction à Identity & Access Management dans Azure

SERIES-3.2 – Comprendre les licences Microsoft Entra ID

SERIES-3.3 – Identité hybride : Entra Connect Sync, Cloud Sync et modes d'authentification

SERIES-3.4 – Créer les groupes Microsoft Entra ID

SERIES-3.5 – Modèle RBAC Azure et séparation des rôles

SERIES-3.6 – Affecter les rôles RBAC aux Management Groups

SERIES-3.7 – Comptes administratifs et comptes Break Glass

SERIES-3.8 – MFA, authentification forte et méthodes phishing-resistant

SERIES-3.9 – Conditional Access Foundation

SERIES-3.10 – PIM, Access Reviews et accès temporaires

SERIES-3.11 – Comptes invités et gouvernance B2B

SERIES-3.12 – Service Principals, App Registrations et Managed Identities

SERIES-3.13 – Export des logs Entra ID vers Log Analytics

SERIES-3.14 – Validation IAM Foundation
```

---

# 25. Références

* Microsoft Entra ID
* Azure RBAC
* Microsoft Entra Conditional Access
* Microsoft Entra Privileged Identity Management
* Microsoft Entra Access Reviews
* Microsoft Entra ID Governance
* Microsoft Entra Connect Sync
* Microsoft Entra Cloud Sync
* Microsoft Entra ID Protection
* Microsoft Cloud Adoption Framework
* Azure Well-Architected Framework
* Azure Landing Zone Architecture

---

# 26. Décision Finale

MonLabAzure Cloud Platform adopte une stratégie IAM basée sur Microsoft Entra ID, Azure RBAC, la séparation des rôles, les groupes de sécurité, les comptes administratifs dédiés, les comptes Break Glass, les comptes de service, les Managed Identities et la prise en compte des environnements hybrides.

La première version de la Phase 3 sera implémentée en mode IAM Foundation afin de rester compatible avec un Lab personnel.

Les fonctionnalités avancées comme PIM, Conditional Access, Access Reviews, groupes dynamiques, Identity Protection et gouvernance avancée seront traitées progressivement selon les besoins, les licences disponibles et la maturité du projet.

Cette stratégie devient la référence officielle pour la Phase 3 – Identity & Access Management.

Toute évolution majeure du modèle IAM devra faire l'objet d'un nouvel ADR.
