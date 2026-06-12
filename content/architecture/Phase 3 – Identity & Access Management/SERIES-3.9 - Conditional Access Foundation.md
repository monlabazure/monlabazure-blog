---

title: "SERIES-3.9 - Conditional Access Foundation"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "Conditional Access"
* "Microsoft Entra ID"
* "MFA"
* "Security"
* "Zero Trust"
* "Break Glass"
* "Azure"

---

# SERIES-3.9 – Conditional Access Foundation

## Introduction

Dans l'article précédent, nous avons défini la stratégie MFA, l'authentification forte et les méthodes phishing-resistant pour MonLabAzure Cloud Platform.

Nous avons notamment retenu les principes suivants :

* MFA obligatoire pour les comptes administratifs ;
* méthodes faibles à limiter ;
* méthodes phishing-resistant à privilégier pour les rôles sensibles ;
* comptes Break Glass protégés mais utilisables en urgence ;
* adoption progressive des contrôles avancés.

Nous allons maintenant traiter un composant central de Microsoft Entra ID :

```text
Conditional Access
```

Conditional Access permet de contrôler les accès selon le contexte.

Il ne s'agit pas seulement de demander MFA.

Il s'agit de décider :

```text
Qui peut accéder à quoi ?
Depuis où ?
Avec quel niveau de risque ?
Avec quel type d'appareil ?
Avec quelle méthode d'authentification ?
Dans quelles conditions ?
```

Cet article pose les fondations Conditional Access pour MonLabAzure.

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
SERIES-3.9 – Conditional Access Foundation ⏳
SERIES-3.10 – PIM, Access Reviews et accès temporaires
SERIES-3.11 – Comptes invités et gouvernance B2B
SERIES-3.12 – Service Principals, App Registrations et Managed Identities
SERIES-3.13 – Export des logs Entra ID vers Log Analytics
SERIES-3.14 – Validation IAM Foundation
```

---

# Références d'Architecture

Cet article applique principalement **ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride**.

ADR-007 impose notamment :

* MFA obligatoire pour les comptes administratifs ;
* protection renforcée des comptes privilégiés ;
* exclusion contrôlée des comptes Break Glass des politiques bloquantes ;
* surveillance des comptes sensibles ;
* adoption progressive de Conditional Access ;
* audit des authentifications legacy ;
* évolution future vers PIM, Access Reviews et Identity Protection.

Conditional Access devient ici le mécanisme technique permettant d'appliquer une partie de ces décisions.

---

# Objectif de l'Article

À la fin de cet article, nous aurons défini :

* le rôle de Conditional Access dans Microsoft Entra ID ;
* les composants d'une policy Conditional Access ;
* les prérequis de licence ;
* les règles de sécurité avant déploiement ;
* la stratégie MonLabAzure ;
* les premières policies de fondation ;
* la méthode de validation en Report-only ;
* les points de vigilance pour éviter un verrouillage du tenant.

---

# Qu'est-ce que Conditional Access ?

Conditional Access est un moteur de décision d'accès dans Microsoft Entra ID.

Il analyse différents signaux, puis applique une décision.

Exemple :

```text
Signal :
Utilisateur administrateur
+
Application Azure Management
+
Connexion depuis un emplacement non approuvé

Décision :
Exiger MFA
ou
Bloquer
ou
Exiger une méthode d'authentification plus forte
```

Conditional Access s'inscrit dans une logique Zero Trust.

Le principe n'est pas :

```text
Utilisateur connecté = utilisateur de confiance
```

Le principe devient :

```text
Chaque accès doit être évalué selon son contexte.
```

---

# Les Composants d'une Policy Conditional Access

Une policy Conditional Access repose généralement sur quatre blocs.

```text
Assignments
Conditions
Access controls
Enable policy
```

---

# Assignments

Les assignments définissent qui et quoi est ciblé.

Ils peuvent inclure :

* utilisateurs ;
* groupes ;
* rôles administratifs ;
* utilisateurs invités ;
* applications cloud ;
* workload identities selon scénario.

Exemples :

```text
Tous les administrateurs
Tous les utilisateurs
Groupe AZR-MLA-PLATFORM-ADMINS
Application Microsoft Azure Management
```

---

# Conditions

Les conditions permettent de préciser le contexte.

Exemples :

* emplacement ;
* plateforme de l'appareil ;
* type de client ;
* risque utilisateur ;
* risque de connexion ;
* appareil conforme ;
* filtre sur les appareils.

Certaines conditions avancées nécessitent des licences spécifiques.

---

# Access Controls

Les contrôles définissent l'action à appliquer.

Exemples :

```text
Block access
Grant access
Require multifactor authentication
Require authentication strength
Require compliant device
Require hybrid Azure AD joined device
Require approved client app
```

Dans MonLabAzure, nous commencerons avec des contrôles simples :

* exiger MFA ;
* bloquer l'authentification legacy ;
* préparer les authentication strengths pour les administrateurs.

---

# Enable Policy

Une policy peut être configurée selon plusieurs états :

```text
Off
Report-only
On
```

Pour MonLabAzure, la règle est claire :

```text
Toute nouvelle policy Conditional Access doit d'abord être testée en Report-only.
```

Microsoft recommande d’utiliser le mode Report-only pour tester les politiques avant application effective et d’évaluer leur impact avant de les activer.

---

# Pourquoi Conditional Access est Sensible ?

Conditional Access peut bloquer les accès.

Une mauvaise policy peut provoquer :

* blocage des administrateurs ;
* blocage des comptes Break Glass ;
* blocage des utilisateurs ;
* perte d'accès au portail Azure ;
* interruption d'applications ;
* incident de production ;
* verrouillage du tenant.

Exemple de mauvaise configuration :

```text
Policy :
Bloquer tous les utilisateurs
sur toutes les applications
sans exclusion

Résultat :
Plus personne ne peut se connecter.
```

C'est pour cette raison que Conditional Access doit être déployé progressivement.

---

# Prérequis de Licence

Conditional Access nécessite Microsoft Entra ID P1 ou supérieur.

Pour des scénarios plus avancés, d'autres licences peuvent être nécessaires :

```text
Conditional Access classique      → Entra ID P1
Authentication Strengths          → Entra ID P1 selon scénario
Identity Protection Risk Policies → Entra ID P2
PIM                               → Entra ID P2
Access Reviews                    → Entra ID P2 / Governance selon scénario
```

Dans MonLabAzure, nous traiterons Conditional Access comme un sujet nécessitant au minimum Entra ID P1.

---

# Prérequis de Sécurité Avant Déploiement

Avant d'activer une policy Conditional Access, il faut vérifier plusieurs points.

## 1. Comptes Break Glass Disponibles

Les comptes suivants doivent exister :

```text
bg-admin-01@monlabazure.onmicrosoft.com
bg-admin-02@monlabazure.onmicrosoft.com
```

Ils doivent être :

* cloud-only ;
* Global Administrator ;
* testés ;
* documentés ;
* accessibles en urgence ;
* surveillés.

Microsoft recommande d’exclure les comptes d’urgence ou Break Glass des politiques Conditional Access afin d’éviter un verrouillage complet après une mauvaise configuration.

---

## 2. Méthodes MFA Enregistrées

Les comptes administratifs doivent avoir une méthode MFA prête.

Exemples :

```text
Microsoft Authenticator
Passkey / FIDO2
Windows Hello for Business
Certificat
```

Sans méthode enregistrée, une policy MFA peut bloquer les utilisateurs ciblés.

---

## 3. Groupes IAM Créés

Les groupes suivants doivent être disponibles :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-OPERATIONS
AZR-MLA-DEVELOPERS
AZR-MLA-ARCHITECTS
AZR-MLA-READERS
```

Ils permettront de cibler progressivement les policies.

---

## 4. Pilotage Progressif

Ne jamais commencer avec :

```text
Tous les utilisateurs
+
Toutes les applications
+
Policy activée immédiatement
```

Approche recommandée :

```text
Groupe pilote
        │
        ▼
Report-only
        │
        ▼
Analyse des impacts
        │
        ▼
Activation progressive
```

Microsoft recommande de tester les policies avec un groupe pilote avant un déploiement large.

---

# Stratégie Conditional Access MonLabAzure

MonLabAzure adopte une stratégie progressive en plusieurs niveaux.

```text
Niveau 1 – Sécuriser les administrateurs
Niveau 2 – Bloquer l'authentification legacy
Niveau 3 – Protéger Azure Management
Niveau 4 – Étendre aux utilisateurs
Niveau 5 – Préparer les contrôles avancés
```

Dans cet article, nous allons définir les fondations, sans tout activer brutalement.

---

# Convention de Nommage des Policies

Les policies Conditional Access doivent suivre une convention lisible.

Format proposé :

```text
CA-MLA-<SCOPE>-<OBJECTIF>-<MODE>
```

Exemples :

```text
CA-MLA-ADMINS-REQUIRE-MFA-REPORT
CA-MLA-ALL-BLOCK-LEGACY-AUTH-REPORT
CA-MLA-AZURE-MGMT-REQUIRE-MFA-REPORT
CA-MLA-ADMINS-PHISHING-RESISTANT-REPORT
```

Lorsque la policy est activée, le suffixe peut évoluer :

```text
REPORT
ON
```

Exemple :

```text
CA-MLA-ADMINS-REQUIRE-MFA-ON
```

Cette convention permet de lire rapidement :

* le contexte ;
* la population ciblée ;
* l'objectif ;
* l'état de déploiement.

---

# Policy 1 – Exiger MFA pour les Administrateurs

## Objectif

Protéger les comptes administratifs.

Groupes concernés :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-NETWORK-ADMINS
AZR-MLA-SECURITY-ADMINS
AZR-MLA-OPERATIONS
```

Application ciblée :

```text
All cloud apps
```

Contrôle :

```text
Require multifactor authentication
```

État initial :

```text
Report-only
```

Nom proposé :

```text
CA-MLA-ADMINS-REQUIRE-MFA-REPORT
```

Microsoft documente une policy type pour exiger MFA sur les rôles administratifs ; les organisations plus matures peuvent ensuite évoluer vers une policy exigeant une méthode phishing-resistant pour les administrateurs.

---

## Configuration Logique

```text
Users or groups:
Include:
  AZR-MLA-PLATFORM-ADMINS
  AZR-MLA-NETWORK-ADMINS
  AZR-MLA-SECURITY-ADMINS
  AZR-MLA-OPERATIONS

Exclude:
  bg-admin-01
  bg-admin-02

Target resources:
  All cloud apps

Grant:
  Require multifactor authentication

Enable policy:
  Report-only
```

---

# Policy 2 – Bloquer l'Authentification Legacy

## Objectif

Bloquer les protocoles d'authentification anciens qui ne supportent pas correctement MFA.

Exemples :

* anciens clients Office ;
* protocoles POP ;
* IMAP ;
* SMTP AUTH selon contexte ;
* clients utilisant Basic Authentication.

Ces protocoles peuvent contourner MFA.

Microsoft précise que l’authentification legacy ne supporte pas MFA et peut permettre à un attaquant de contourner une policy MFA.

Nom proposé :

```text
CA-MLA-ALL-BLOCK-LEGACY-AUTH-REPORT
```

---

## Configuration Logique

```text
Users or groups:
Include:
  All users

Exclude:
  bg-admin-01
  bg-admin-02

Target resources:
  All cloud apps

Conditions:
  Client apps:
    Legacy authentication clients

Access controls:
  Block access

Enable policy:
  Report-only
```

Microsoft documente une procédure Conditional Access pour bloquer l’authentification legacy et recommande de démarrer cette policy en Report-only.

---

# Policy 3 – Protéger Azure Management

## Objectif

Renforcer l'accès aux portails et API d'administration Azure.

Application ciblée :

```text
Microsoft Azure Management
```

Population ciblée :

```text
Administrateurs Azure
```

Contrôle :

```text
Require multifactor authentication
```

Nom proposé :

```text
CA-MLA-AZURE-MGMT-REQUIRE-MFA-REPORT
```

---

## Configuration Logique

```text
Users or groups:
Include:
  AZR-MLA-PLATFORM-ADMINS
  AZR-MLA-NETWORK-ADMINS
  AZR-MLA-SECURITY-ADMINS
  AZR-MLA-OPERATIONS

Exclude:
  bg-admin-01
  bg-admin-02

Target resources:
  Microsoft Azure Management

Grant:
  Require multifactor authentication

Enable policy:
  Report-only
```

Cette policy permet de protéger en priorité l'administration Azure sans appliquer immédiatement des contraintes à toutes les applications.

---

# Policy 4 – Préparer le Phishing-Resistant MFA pour les Administrateurs

## Objectif

Préparer l'évolution vers des méthodes MFA plus fortes.

Population cible initiale :

```text
AZR-MLA-PLATFORM-ADMINS
AZR-MLA-SECURITY-ADMINS
```

Contrôle cible :

```text
Require authentication strength
Phishing-resistant MFA
```

Nom proposé :

```text
CA-MLA-ADMINS-PHISHING-RESISTANT-REPORT
```

Microsoft documente une policy Conditional Access pour exiger une authentification MFA phishing-resistant sur les rôles hautement privilégiés.

---

## Configuration Logique

```text
Users or groups:
Include:
  AZR-MLA-PLATFORM-ADMINS
  AZR-MLA-SECURITY-ADMINS

Exclude:
  bg-admin-01
  bg-admin-02

Target resources:
  Microsoft Azure Management

Grant:
  Require authentication strength:
    Phishing-resistant MFA

Enable policy:
  Report-only
```

Cette policy ne doit pas être activée trop tôt.

Elle nécessite que les administrateurs aient déjà enregistré des méthodes compatibles, par exemple :

* passkey / FIDO2 ;
* Windows Hello for Business ;
* certificat.

---

# Pourquoi Exclure les Comptes Break Glass ?

Les comptes Break Glass sont faits pour récupérer l'accès en situation d'urgence.

Une policy Conditional Access peut dépendre :

* d'un téléphone ;
* d'un appareil conforme ;
* d'un emplacement réseau ;
* d'une méthode MFA ;
* d'un service externe ;
* d'une configuration qui peut être défaillante.

Si les comptes Break Glass sont bloqués par cette policy, ils ne servent plus à rien.

La règle MonLabAzure est donc :

```text
Les comptes Break Glass doivent être exclus des policies Conditional Access bloquantes.
```

Cette exclusion doit être compensée par :

* mots de passe très forts ;
* méthode d'authentification forte indépendante ;
* surveillance critique ;
* alertes ;
* tests réguliers ;
* revue post-utilisation.

Microsoft précise que les policies en Report-only ne bloquent pas l’accès et n’ont pas besoin d’exclure les comptes d’urgence, mais qu’une policy activée pouvant affecter ces comptes doit prévoir leur exclusion pour éviter de les bloquer.

---

# Mode Report-only

Le mode Report-only permet de simuler l'application d'une policy.

La policy n'est pas appliquée réellement.

Elle permet de répondre à la question :

```text
Que se passerait-il si cette policy était activée ?
```

Avantages :

* limiter les risques ;
* observer l'impact ;
* identifier les utilisateurs bloqués ;
* corriger avant activation ;
* préparer la communication.

Règle MonLabAzure :

```text
Toute nouvelle policy Conditional Access démarre en Report-only.
```

---

# Validation des Policies en Report-only

Avant d'activer une policy, vérifier :

```text
Qui serait bloqué ?
Qui serait autorisé ?
Qui serait invité à MFA ?
Quels comptes n'ont pas de méthode MFA ?
Quels anciens clients utilisent encore legacy authentication ?
Les comptes Break Glass sont-ils exclus des policies activées ?
```

Chemin portail :

```text
Microsoft Entra admin center
        │
        ▼
Protection
        │
        ▼
Conditional Access
        │
        ▼
Insights and reporting
```

Ou via les logs de connexion :

```text
Microsoft Entra admin center
        │
        ▼
Identity
        │
        ▼
Monitoring & health
        │
        ▼
Sign-in logs
```

---

# Ordre d'Activation Recommandé

L'ordre recommandé est le suivant :

```text
1. Créer les policies en Report-only.
2. Vérifier les impacts pendant une période d'observation.
3. Corriger les comptes sans MFA.
4. Identifier les clients legacy.
5. Tester avec un groupe pilote.
6. Activer la policy MFA administrateurs.
7. Activer la protection Azure Management.
8. Activer le blocage legacy authentication.
9. Préparer le phishing-resistant MFA.
```

Ne pas activer toutes les policies en même temps.

---

# Cas des Environnements Hybrides

Dans un environnement hybride, Conditional Access doit être étudié avec plus de prudence.

Points à vérifier :

* mode d'authentification : PHS, PTA ou Federation ;
* présence d'AD FS ;
* état d'Entra Connect Sync ou Cloud Sync ;
* dépendances aux appareils hybrides ;
* appareils conformes ou non ;
* accès depuis le réseau interne ;
* méthodes MFA disponibles ;
* comptes synchronisés privilégiés.

Exemple de risque :

```text
Policy :
Require hybrid joined device

Problème :
Les administrateurs cloud-only ne possèdent pas d'appareil hybride joint

Résultat :
Blocage possible
```

Les comptes Break Glass doivent rester indépendants de ces dépendances hybrides.

---

# Cas des Comptes Invités

Les comptes invités ne sont pas traités en détail dans cet article.

Ils feront l'objet d'une série dédiée.

Cependant, Conditional Access devra plus tard couvrir :

* invités B2B ;
* prestataires ;
* accès externes ;
* MFA invité ;
* conditions d'accès aux applications sensibles ;
* accès temporaires.

---

# Ce que Nous Ne Configurons Pas Encore

Dans cet article, nous ne configurons pas encore :

* policies basées sur le risque utilisateur ;
* policies basées sur le risque de connexion ;
* Identity Protection ;
* PIM ;
* Access Reviews ;
* règles spécifiques aux invités ;
* politiques complètes d'appareils conformes ;
* intégration Sentinel.

Ces sujets seront traités dans les séries suivantes ou dans les phases avancées.

---

# Points de Vigilance

## Ne Pas Bloquer les Administrateurs

Toujours garder au moins deux comptes Break Glass utilisables.

## Ne Pas Cibler Tous les Utilisateurs Directement

Commencer avec des groupes pilotes.

## Ne Pas Activer Sans Analyse

Le mode Report-only est obligatoire dans notre démarche.

## Ne Pas Oublier les Applications

Une policy peut cibler toutes les applications ou une application spécifique.

Le choix doit être explicite.

## Ne Pas Oublier les Clients Legacy

L'authentification legacy reste un risque majeur.

Elle doit être identifiée puis bloquée progressivement.

## Ne Pas Confondre MFA et Phishing-Resistant

MFA n'est pas toujours phishing-resistant.

Une stratégie mature doit évoluer vers des méthodes plus fortes pour les rôles sensibles.

---

# Points de Contrôle

Avant de passer à la série suivante, vérifier que :

* les comptes Break Glass existent ;
* les comptes Break Glass sont cloud-only ;
* les comptes Break Glass sont exclus des policies activées ;
* les comptes administratifs disposent d'une méthode MFA ;
* les policies sont d'abord en Report-only ;
* les impacts sont analysés ;
* les clients legacy sont identifiés ;
* les policies suivent une convention de nommage ;
* aucune policy bloquante globale n'est activée sans validation ;
* le modèle respecte ADR-007.

---

# Dépannage

## Tous les Administrateurs Sont Bloqués

Cause probable :

```text
Policy trop large ou mauvaise exclusion.
```

Actions :

```text
Utiliser un compte Break Glass.
Désactiver ou corriger la policy.
Analyser les logs.
Documenter l'incident.
```

---

## Les Utilisateurs ne Reçoivent pas de Demande MFA

Causes possibles :

* policy en Report-only ;
* utilisateur non ciblé ;
* application non ciblée ;
* exclusion active ;
* autre policy prioritaire ;
* méthode déjà satisfaite.

---

## La Policy ne Bloque pas l'Authentification Legacy

Vérifier :

* la condition Client apps ;
* les logs de connexion ;
* les exclusions ;
* l'état de la policy ;
* la présence réelle de clients legacy.

---

## Le Phishing-Resistant MFA Bloque Certains Admins

Cause probable :

```text
Méthode compatible non enregistrée.
```

Actions :

* repasser en Report-only ;
* enregistrer les méthodes compatibles ;
* tester avec un groupe pilote ;
* réactiver progressivement.

---

# Coûts et Licences

Conditional Access nécessite Microsoft Entra ID P1 ou supérieur.

Les scénarios avancés peuvent nécessiter P2 ou Governance :

```text
Conditional Access classique       → Entra ID P1
Authentication Strengths           → Entra ID P1 selon scénario
Identity Protection Risk Policies  → Entra ID P2
PIM                                → Entra ID P2
Access Reviews                     → Entra ID P2 / Governance selon scénario
```

Dans un Lab personnel, il faut vérifier les licences disponibles avant d'appliquer les policies.

---

# Livrables de cet Article

À l'issue de cet article, nous avons :

* compris le rôle de Conditional Access ;
* défini une convention de nommage des policies ;
* défini une première baseline Conditional Access ;
* préparé MFA pour les administrateurs ;
* préparé le blocage de l'authentification legacy ;
* préparé la protection Azure Management ;
* préparé le phishing-resistant MFA ;
* confirmé l'exclusion contrôlée des comptes Break Glass ;
* défini l'usage obligatoire du mode Report-only.

---

# Conclusion

Conditional Access est une brique majeure de la sécurité Microsoft Entra ID.

Mais c'est aussi une brique sensible.

Une bonne policy peut renforcer fortement la sécurité.

Une mauvaise policy peut bloquer tout le tenant.

La stratégie MonLabAzure est donc progressive :

```text
Préparer les comptes Break Glass
        │
        ▼
Créer les policies en Report-only
        │
        ▼
Analyser les impacts
        │
        ▼
Activer progressivement
        │
        ▼
Renforcer vers phishing-resistant MFA
```

Cette approche permet de sécuriser la plateforme sans prendre de risque excessif.

Dans le prochain article, nous aborderons PIM, Access Reviews et les accès temporaires afin de réduire les privilèges permanents et améliorer la gouvernance des accès.
