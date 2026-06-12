---

title: "SERIES-3.8 - MFA, Authentification Forte et Méthodes Phishing-Resistant"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "MFA"
* "Authentication"
* "Phishing Resistant"
* "Microsoft Entra ID"
* "Conditional Access"
* "Security"
* "Azure"

---

# SERIES-3.8 – MFA, Authentification Forte et Méthodes Phishing-Resistant

## Introduction

Dans l'article précédent, nous avons traité les comptes administratifs et les comptes Break Glass.

Nous avons défini plusieurs règles importantes :

* les comptes administratifs doivent être séparés des comptes standards ;
* les comptes administratifs doivent être nominatifs ;
* les comptes Break Glass doivent être cloud-only ;
* les comptes Break Glass doivent être utilisés uniquement en situation d'urgence ;
* les comptes sensibles doivent être surveillés.

Nous devons maintenant renforcer l'authentification.

Dans un environnement Azure, un mot de passe seul n'est pas suffisant.

Même un mot de passe complexe peut être :

* volé ;
* réutilisé ;
* intercepté ;
* hameçonné ;
* compromis par malware ;
* exposé dans une fuite.

L'objectif de cette série est donc de définir la stratégie MFA et les méthodes d'authentification forte pour MonLabAzure Cloud Platform.

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
SERIES-3.8 – MFA, authentification forte et méthodes phishing-resistant ⏳
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

ADR-007 impose les principes suivants :

* MFA obligatoire pour les comptes administratifs ;
* protection renforcée des comptes privilégiés ;
* comptes Break Glass séparés ;
* surveillance des comptes sensibles ;
* prise en compte future de Conditional Access ;
* adoption progressive de méthodes d'authentification plus fortes.

Cette série précise ces règles au niveau opérationnel.

---

# Objectif de l'Article

À la fin de cet article, nous aurons défini :

* pourquoi MFA est indispensable ;
* la différence entre MFA classique et authentification phishing-resistant ;
* les principales méthodes d'authentification Microsoft Entra ID ;
* les méthodes recommandées pour les comptes administratifs ;
* les méthodes à éviter ou à limiter ;
* la stratégie spécifique pour les comptes Break Glass ;
* les prérequis pour Conditional Access ;
* les contrôles à réaliser avant la suite de la Phase 3.

---

# Pourquoi MFA est Indispensable

L'authentification par mot de passe seul repose sur un facteur unique :

```text
Quelque chose que l'utilisateur connaît
```

Exemple :

```text
Mot de passe
```

Le problème est simple.

Si ce mot de passe est compromis, l'attaquant peut se connecter.

MFA ajoute un ou plusieurs facteurs supplémentaires :

```text
Quelque chose que l'utilisateur possède
Quelque chose que l'utilisateur est
Quelque chose que l'utilisateur fait
```

Exemples :

* téléphone ;
* application d'authentification ;
* clé de sécurité ;
* biométrie ;
* certificat ;
* passkey.

L'objectif est de rendre la compromission du compte beaucoup plus difficile.

---

# MFA et Comptes Administratifs

Les comptes administratifs sont des cibles prioritaires.

Un attaquant qui compromet un compte administrateur peut :

* modifier des ressources Azure ;
* supprimer des données ;
* créer des accès ;
* désactiver des contrôles ;
* modifier des politiques ;
* créer des identités techniques ;
* exfiltrer des informations sensibles.

Dans MonLabAzure, la règle est donc :

```text
Tout compte administratif doit utiliser MFA.
```

Exemples de comptes concernés :

```text
adm-platform@monlabazure.fr
adm-network@monlabazure.fr
adm-security@monlabazure.fr
adm-operations@monlabazure.fr
```

À terme, ces comptes devront être protégés par Conditional Access et, si possible, par des méthodes phishing-resistant.

---

# MFA Classique vs Phishing-Resistant MFA

Toutes les méthodes MFA ne se valent pas.

Certaines méthodes ajoutent une protection de base.

D'autres protègent aussi contre des attaques de phishing avancées.

---

## MFA Classique

Exemples :

* SMS ;
* appel vocal ;
* code OTP ;
* notification push simple ;
* application d'authentification avec code.

Ces méthodes améliorent la sécurité par rapport au mot de passe seul.

Mais elles peuvent rester exposées à certains risques :

* phishing ;
* fatigue MFA ;
* interception ;
* SIM swapping ;
* attaque de relais ;
* usurpation de session.

---

## Phishing-Resistant MFA

Une méthode phishing-resistant est conçue pour résister aux attaques où l'utilisateur est trompé et redirigé vers une fausse page de connexion.

Exemples :

* clé de sécurité FIDO2 ;
* passkey ;
* Windows Hello for Business ;
* authentification par certificat.

Ces méthodes lient l'authentification à l'appareil, au domaine ou à un secret cryptographique difficile à réutiliser sur un faux site.

Microsoft documente une force d'authentification intégrée appelée **Phishing-resistant MFA strength**, qui peut être utilisée dans Conditional Access pour exiger des méthodes plus résistantes au hameçonnage.

---

# Les Méthodes d'Authentification Microsoft Entra ID

Microsoft Entra ID prend en charge plusieurs méthodes d'authentification.

Nous allons les classer par niveau de robustesse.

---

# Niveau 1 – Méthodes à Limiter

## SMS

Le SMS est simple à déployer, mais il présente plusieurs faiblesses :

* dépendance au réseau mobile ;
* risque de SIM swapping ;
* interception possible ;
* phishing possible ;
* faible contrôle sur le terminal.

Dans MonLabAzure, le SMS peut être toléré pour un Lab simple, mais il ne doit pas être le standard cible pour les comptes administratifs.

---

## Appel Vocal

L'appel vocal présente des limites similaires au SMS.

Il peut être utile dans certains contextes de secours, mais il ne doit pas être la méthode principale pour les comptes privilégiés.

---

# Niveau 2 – Méthodes Acceptables pour Démarrer

## Microsoft Authenticator – Notification Push

Microsoft Authenticator avec notification push est plus pratique et plus sécurisé qu'un simple SMS.

Cependant, une notification push mal utilisée peut être exposée à la fatigue MFA.

Exemple d'attaque :

```text
L'attaquant tente plusieurs connexions.
L'utilisateur reçoit plusieurs notifications.
L'utilisateur finit par approuver par erreur.
```

Pour limiter ce risque, il faut privilégier :

* number matching ;
* contexte de connexion ;
* localisation ;
* information sur l'application cible ;
* sensibilisation utilisateur.

---

## Microsoft Authenticator – Code OTP

Le code OTP généré dans une application est plus contrôlé qu'un SMS.

Il reste cependant vulnérable au phishing si l'utilisateur saisit son code sur une fausse page.

---

# Niveau 3 – Méthodes Fortes

## Microsoft Authenticator Passwordless

Microsoft Authenticator peut être utilisé en mode passwordless.

L'utilisateur ne saisit plus son mot de passe.

Il approuve la connexion avec son appareil et une interaction locale.

Cette approche réduit l'exposition du mot de passe.

Elle reste plus forte qu'un schéma mot de passe + SMS.

---

## Windows Hello for Business

Windows Hello for Business permet une authentification forte liée à l'appareil.

Il peut utiliser :

* PIN ;
* biométrie ;
* clés cryptographiques ;
* intégration avec l'appareil.

C'est une méthode adaptée aux postes managés d'entreprise.

Microsoft liste Windows Hello for Business parmi les méthodes supportées par Microsoft Entra ID et pouvant satisfaire MFA selon les scénarios.

---

# Niveau 4 – Méthodes Phishing-Resistant

## Passkeys et FIDO2

Les passkeys et clés de sécurité FIDO2 sont des méthodes très fortes.

Elles reposent sur une authentification cryptographique.

Elles réduisent fortement le risque de phishing, car l'authentification est liée au service légitime.

Exemples :

```text
Clé de sécurité FIDO2
Passkey dans Microsoft Authenticator
Passkey synchronisée selon le fournisseur
```

Microsoft documente les passkeys/FIDO2 comme méthode d’authentification Microsoft Entra ID permettant d’améliorer et de sécuriser les événements de connexion.

---

## Authentification par Certificat

L'authentification par certificat peut également être utilisée pour renforcer l'accès.

Elle repose sur un certificat délivré à l'utilisateur ou à l'appareil.

Elle peut être pertinente pour :

* administrateurs ;
* appareils maîtrisés ;
* environnements hautement contrôlés ;
* scénarios réglementés.

Microsoft décrit l’authentification par certificat, les clés FIDO2 et Windows Hello for Business comme des méthodes pouvant être utilisées dans des scénarios phishing-resistant.

---

# Authentication Strengths

Microsoft Entra ID propose le concept d'**Authentication Strength**.

Une Authentication Strength permet de définir quelles méthodes d'authentification sont acceptées dans une politique Conditional Access.

Exemple :

```text
Accès au portail Azure
        │
        ▼
Exiger une force d'authentification phishing-resistant
        │
        ▼
Autoriser uniquement FIDO2, Windows Hello for Business ou certificat
```

Microsoft Entra ID fournit des forces intégrées, notamment :

* Multifactor authentication strength ;
* Passwordless MFA strength ;
* Phishing-resistant MFA strength.

Ces forces peuvent être utilisées dans Conditional Access pour imposer un niveau d'authentification adapté à la sensibilité de la ressource.

---

# Stratégie MFA MonLabAzure

MonLabAzure adopte une stratégie progressive.

L'objectif n'est pas de tout imposer dès le premier jour.

L'objectif est de construire une trajectoire claire.

---

## Étape 1 – MFA Obligatoire pour les Administrateurs

Tous les comptes administratifs doivent utiliser MFA.

Groupes concernés :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-OPERATIONS
```

Règle :

```text
Aucun compte administratif sans MFA.
```

---

## Étape 2 – Méthodes MFA Fortes pour les Administrateurs

Les méthodes recommandées pour les administrateurs sont :

```text
Microsoft Authenticator avec protections avancées
Windows Hello for Business
Passkey / FIDO2
Certificat selon besoin
```

Méthodes à limiter :

```text
SMS
Appel vocal
Notification push non contextualisée
```

---

## Étape 3 – Phishing-Resistant pour les Rôles Sensibles

Les rôles sensibles devront évoluer vers des méthodes phishing-resistant.

Groupes prioritaires :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-SECURITY-ADMINS
```

Ressources prioritaires :

```text
Portail Azure
Microsoft Entra admin center
Azure Resource Manager
Portails d'administration
```

---

## Étape 4 – Extension Progressive aux Utilisateurs

Une fois les comptes administratifs protégés, la stratégie pourra être étendue :

* aux développeurs ;
* aux architectes ;
* aux comptes invités ;
* aux utilisateurs accédant aux applications critiques.

---

# Cas Spécifique des Comptes Break Glass

Les comptes Break Glass sont particuliers.

Ils doivent rester utilisables en situation d'urgence.

Ils ne doivent donc pas dépendre exactement des mêmes mécanismes que les comptes administratifs standards.

---

## Règles Break Glass

Les comptes Break Glass doivent :

* être cloud-only ;
* utiliser le domaine `onmicrosoft.com` ;
* disposer du rôle Global Administrator ;
* être exclus des Conditional Access bloquants ;
* être protégés par une méthode forte ;
* être surveillés ;
* être testés régulièrement.

---

## Méthode d'Authentification Break Glass

Le choix de la méthode dépend du contexte.

Options possibles :

```text
Clé FIDO2 dédiée
Passkey dédiée
Certificat dédié
Méthode forte indépendante
```

Principe important :

```text
La méthode d'authentification Break Glass doit rester disponible même si le mécanisme MFA standard est indisponible.
```

Exemple :

```text
Comptes administratifs standards
        │
        ▼
Microsoft Authenticator sur téléphone professionnel

Comptes Break Glass
        │
        ▼
Clé FIDO2 stockée dans un coffre sécurisé
```

---

# Pourquoi Ne Pas Forcer Conditional Access Maintenant ?

La série suivante sera dédiée à Conditional Access.

Dans cet article, nous définissons la stratégie MFA.

Nous ne créons pas encore les policies Conditional Access.

Pourquoi ?

Parce qu'une mauvaise politique Conditional Access peut bloquer les administrateurs.

Avant d'activer une policy, il faut avoir :

* comptes Break Glass créés ;
* exclusions Break Glass définies ;
* méthode MFA testée ;
* comptes administratifs prêts ;
* stratégie de rollback ;
* phase report-only si possible.

Microsoft recommande d’utiliser le mode **Report-only** pour tester les politiques Conditional Access avant application effective, notamment pour éviter les blocages involontaires.

---

# Architecture Cible

La cible MonLabAzure est la suivante :

```text
Compte standard
        │
        ▼
MFA classique ou forte selon usage

Compte administratif
        │
        ▼
MFA obligatoire
        │
        ▼
Méthode forte recommandée
        │
        ▼
Phishing-resistant pour rôles sensibles

Compte Break Glass
        │
        ▼
Méthode forte indépendante
        │
        ▼
Exclusion CA bloquant
        │
        ▼
Surveillance critique
```

---

# Contrôles d'Audit

Les contrôles suivants doivent être prévus :

```text
Comptes administratifs sans MFA
Comptes privilégiés utilisant SMS uniquement
Comptes privilégiés utilisant appel vocal uniquement
Comptes Break Glass non testés
Comptes Break Glass sans méthode forte
Comptes Break Glass dépendants d'une méthode MFA standard
Méthodes d'authentification obsolètes
Utilisateurs avec méthodes faibles
Absence de stratégie phishing-resistant pour administrateurs
```

Ces contrôles seront repris dans les séries Conditional Access, PIM et validation IAM.

---

# Recommandations par Type de Compte

| Type de compte       | Méthode minimale           | Méthode cible                        | Commentaire         |
| -------------------- | -------------------------- | ------------------------------------ | ------------------- |
| Compte standard      | MFA                        | Authenticator / passwordless         | Selon criticité     |
| Compte administratif | MFA obligatoire            | FIDO2 / WHfB / passwordless          | Priorité haute      |
| Compte sécurité      | MFA obligatoire            | Phishing-resistant                   | Très sensible       |
| Compte plateforme    | MFA obligatoire            | Phishing-resistant                   | Très sensible       |
| Compte Break Glass   | Méthode forte indépendante | FIDO2 ou certificat dédié            | Exclu CA bloquant   |
| Compte invité        | MFA selon policy           | MFA trust ou CA dédié                | À traiter plus tard |
| Service Principal    | Secret/certificat          | Workload identity / Managed Identity | Série dédiée        |

---

# Points de Vigilance

## SMS et Appel Vocal

Ces méthodes sont faciles à utiliser, mais ne doivent pas devenir la norme pour les comptes privilégiés.

Elles peuvent être tolérées temporairement dans un Lab, mais elles ne constituent pas une cible Enterprise.

---

## Fatigue MFA

Les notifications push répétées peuvent être exploitées.

Il faut privilégier les mécanismes qui affichent le contexte et limitent l'approbation accidentelle.

---

## Méthodes de Secours

Il faut prévoir des méthodes de secours.

Mais une méthode de secours faible peut affaiblir tout le modèle.

Exemple :

```text
FIDO2 comme méthode principale
SMS comme méthode de secours
```

Dans ce cas, l'attaquant peut chercher à exploiter la méthode la plus faible.

---

## Break Glass

Les comptes Break Glass doivent être protégés, mais pas bloqués.

C'est un équilibre délicat.

Ils doivent être :

```text
Utilisables en urgence
+
Très fortement surveillés
```

---

# Ce que Nous Ne Faisons Pas Encore

Dans cet article, nous ne configurons pas encore :

* les politiques Conditional Access ;
* les Authentication Strengths dans une policy ;
* PIM ;
* Access Reviews ;
* Microsoft Sentinel ;
* export complet des logs Entra ID.

Ces sujets seront traités dans les séries suivantes.

---

# Prérequis pour la Suite

Avant de passer à Conditional Access, vérifier que :

* les comptes administratifs existent ;
* les comptes administratifs sont séparés des comptes standards ;
* les comptes Break Glass existent ;
* les comptes Break Glass sont cloud-only ;
* les méthodes MFA des administrateurs sont enregistrées ;
* les méthodes Break Glass sont documentées ;
* les exclusions Break Glass sont préparées ;
* les risques liés aux méthodes faibles sont compris.

---

# Coûts et Licences

Les fonctionnalités MFA de base peuvent dépendre du contexte tenant et des licences disponibles.

Les fonctionnalités avancées comme :

* Conditional Access ;
* Authentication Strengths ;
* méthodes phishing-resistant imposées par policy ;
* Identity Protection ;
* PIM ;

peuvent nécessiter Microsoft Entra ID P1 ou P2 selon le scénario.

Dans MonLabAzure, la logique reste progressive :

```text
Comprendre la stratégie
        │
        ▼
Préparer les comptes
        │
        ▼
Déployer Conditional Access
        │
        ▼
Renforcer vers phishing-resistant
```

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* défini la stratégie MFA MonLabAzure ;
* distingué MFA classique et phishing-resistant MFA ;
* identifié les méthodes faibles à limiter ;
* identifié les méthodes fortes à privilégier ;
* défini la stratégie pour les comptes administratifs ;
* défini la stratégie pour les comptes Break Glass ;
* préparé la série Conditional Access Foundation.

---

# Conclusion

L'authentification forte est une brique fondamentale de la sécurité IAM.

Pour MonLabAzure Cloud Platform, la priorité est claire :

```text
Protéger d'abord les comptes administratifs.
Séparer les comptes Break Glass.
Limiter les méthodes faibles.
Préparer l'adoption des méthodes phishing-resistant.
```

MFA n'est pas une simple option de sécurité.

C'est une condition de base pour toute plateforme Azure sérieuse.

Dans le prochain article, nous mettrons en place les fondations de Conditional Access afin d'appliquer progressivement ces règles d'authentification.
