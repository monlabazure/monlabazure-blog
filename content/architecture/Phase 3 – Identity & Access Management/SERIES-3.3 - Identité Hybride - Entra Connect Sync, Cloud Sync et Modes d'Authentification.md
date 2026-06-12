---

title: "SERIES-3.3 - Identité Hybride : Entra Connect Sync, Cloud Sync et Modes d'Authentification"
date: 2026-06-08
draft: false
categories: ["Architecture"]
tags:

* "Phase-3"
* "Identity"
* "IAM"
* "Hybrid Identity"
* "Microsoft Entra ID"
* "Entra Connect Sync"
* "Entra Cloud Sync"
* "Active Directory"
* "Authentication"
* "Azure"

---

# SERIES-3.3 – Identité Hybride : Entra Connect Sync, Cloud Sync et Modes d'Authentification

## Introduction

Dans l'article précédent, nous avons étudié les licences Microsoft Entra ID et leur impact sur les choix IAM.

Avant de créer les groupes Microsoft Entra ID et de construire notre modèle RBAC, il est nécessaire de traiter un sujet majeur dans les architectures Azure Enterprise :

```text
L'identité hybride
```

Dans un Lab personnel, il est possible de travailler avec un tenant Microsoft Entra ID entièrement cloud-only.

Mais dans une grande partie des environnements d'entreprise, les identités ne naissent pas directement dans le cloud.

Elles proviennent souvent d'un Active Directory local existant.

Dans ce cas, Microsoft Entra ID devient une extension cloud du système d'identité interne.

Cette situation influence fortement :

* la stratégie d'authentification ;
* la gestion des comptes ;
* la gouvernance des groupes ;
* les accès Azure RBAC ;
* les comptes administratifs ;
* les comptes de service ;
* la sécurité ;
* la supervision ;
* les scénarios de migration.

Cet article présente donc les concepts essentiels de l'identité hybride avec Microsoft Entra ID.

---

# Positionnement dans le Projet

```text
Phase 3 – Identity & Access Management

ADR-007 – Stratégie IAM, RBAC, Break Glass et Identité Hybride ✅

SERIES-3.1 – Introduction à Identity & Access Management dans Azure ✅
SERIES-3.2 – Comprendre les licences Microsoft Entra ID ✅
SERIES-3.3 – Identité hybride : Entra Connect Sync, Cloud Sync et modes d'authentification ⏳
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

# Pourquoi l'Identité Hybride est Importante

Dans beaucoup d'organisations, Active Directory local reste le référentiel historique des identités.

Il contient souvent :

* les comptes utilisateurs ;
* les groupes de sécurité ;
* les comptes de service ;
* les attributs RH ;
* les unités d'organisation ;
* les règles de gestion internes ;
* les dépendances applicatives historiques.

Lorsqu'une entreprise adopte Azure, elle ne remplace pas toujours immédiatement son Active Directory local.

Elle met généralement en place une synchronisation vers Microsoft Entra ID.

Le modèle devient alors :

```text
Active Directory local
        │
        ▼
Synchronisation d'identités
        │
        ▼
Microsoft Entra ID
        │
        ▼
Azure / Microsoft 365 / Applications SaaS
```

Cette architecture permet de conserver une source d'identité existante tout en donnant accès aux services cloud.

---

# Cloud-Only vs Hybride

## Modèle Cloud-Only

Dans un modèle cloud-only, les identités sont créées directement dans Microsoft Entra ID.

Exemple :

```text
prenom.nom@monlabazure.fr
```

Les utilisateurs, groupes et applications sont gérés directement dans le cloud.

Avantages :

* simplicité ;
* moins de dépendance à l'infrastructure locale ;
* administration centralisée dans Microsoft Entra ID ;
* modèle adapté aux nouveaux environnements cloud-native.

Limites :

* moins représentatif des grandes entreprises historiques ;
* pas de synchronisation avec un Active Directory local ;
* moins adapté aux scénarios de migration d'entreprise.

---

## Modèle Hybride

Dans un modèle hybride, les identités sont généralement créées ou gérées dans Active Directory local, puis synchronisées vers Microsoft Entra ID.

Exemple :

```text
Active Directory local
        │
        ▼
Utilisateur synchronisé
        │
        ▼
Microsoft Entra ID
```

Avantages :

* continuité avec l'existant ;
* compatibilité avec les applications internes ;
* centralisation historique des identités ;
* transition progressive vers le cloud.

Limites :

* dépendance à l'infrastructure locale ;
* complexité de synchronisation ;
* risques liés aux attributs AD ;
* supervision indispensable ;
* gestion plus complexe du cycle de vie des comptes.

---

# Les Deux Outils Principaux

Microsoft propose principalement deux solutions pour synchroniser les identités entre Active Directory local et Microsoft Entra ID :

```text
Microsoft Entra Connect Sync
Microsoft Entra Cloud Sync
```

Ces deux outils répondent au même objectif général :

```text
Synchroniser des identités locales vers Microsoft Entra ID
```

Mais leur architecture, leur gestion et leurs cas d'usage diffèrent.

---

# Microsoft Entra Connect Sync

## Définition

Microsoft Entra Connect Sync est la solution historique de synchronisation entre Active Directory local et Microsoft Entra ID.

Elle repose sur un serveur de synchronisation installé dans l'environnement local.

Ce serveur exécute le moteur de synchronisation, applique les règles de synchronisation et pousse les objets vers Microsoft Entra ID.

Schéma simplifié :

```text
Active Directory local
        │
        ▼
Serveur Entra Connect Sync
        │
        ▼
Microsoft Entra ID
```

---

## Objets Synchronisés

Entra Connect Sync peut synchroniser différents types d'objets, notamment :

* utilisateurs ;
* groupes ;
* contacts ;
* attributs ;
* mots de passe selon le mode choisi ;
* objets nécessaires à certains scénarios hybrides.

---

## Cas d'Usage Typiques

Entra Connect Sync reste pertinent dans les environnements qui disposent :

* d'un Active Directory historique complexe ;
* de règles de synchronisation avancées ;
* de plusieurs forêts avec besoins spécifiques ;
* d'une logique d'attributs personnalisée ;
* de scénarios hybrides déjà en production ;
* de dépendances fortes avec des applications locales ;
* de besoins de personnalisation avancée.

---

## Points Forts

Les points forts d'Entra Connect Sync sont :

* maturité historique ;
* grande richesse fonctionnelle ;
* personnalisation avancée ;
* compatibilité avec de nombreux scénarios existants ;
* forte adoption dans les entreprises.

---

## Points d'Attention

Entra Connect Sync est un composant critique.

Il doit être traité comme une brique d'infrastructure sensible.

Points d'attention :

* disponibilité du serveur ;
* supervision du service de synchronisation ;
* mise à jour régulière ;
* sauvegarde de la configuration ;
* documentation des règles personnalisées ;
* contrôle des erreurs de synchronisation ;
* surveillance des comptes privilégiés synchronisés.

Un problème sur Entra Connect Sync peut provoquer :

* retard de synchronisation ;
* comptes non créés dans Entra ID ;
* groupes obsolètes ;
* erreurs d'accès ;
* offboarding incomplet ;
* incohérence entre AD local et Microsoft Entra ID.

---

# Microsoft Entra Cloud Sync

## Définition

Microsoft Entra Cloud Sync est une approche plus moderne de synchronisation hybride.

La configuration et l'orchestration sont gérées depuis le cloud.

L'environnement local héberge un agent léger qui sert de pont entre Active Directory et Microsoft Entra ID.

Schéma simplifié :

```text
Active Directory local
        │
        ▼
Agent Cloud Sync
        │
        ▼
Service Microsoft Entra Cloud Sync
        │
        ▼
Microsoft Entra ID
```

Microsoft décrit Cloud Sync comme une synchronisation moderne, gérée depuis le cloud, avec une approche légère basée sur un agent.

---

## Objets Synchronisés

Cloud Sync permet de synchroniser notamment :

* utilisateurs ;
* groupes ;
* contacts.

Selon les scénarios, il peut être utilisé pour des environnements multi-forêts ou des forêts déconnectées.

---

## Cas d'Usage Typiques

Cloud Sync est particulièrement intéressant pour :

* nouveaux environnements hybrides ;
* environnements simples ;
* scénarios multi-forêts ;
* filiales ;
* forêts Active Directory séparées ;
* organisations souhaitant réduire la complexité locale ;
* migration progressive depuis Entra Connect Sync.

Microsoft indique que, pour la plupart des scénarios, Cloud Sync est l'outil recommandé, tandis que Connect Sync reste utilisé pour les scénarios avancés ou complexes.

---

## Points Forts

Les points forts de Cloud Sync sont :

* agent léger ;
* configuration pilotée depuis le cloud ;
* réduction de la complexité locale ;
* mise à jour simplifiée des agents ;
* bonne approche pour certains scénarios multi-forêts ;
* modernisation progressive de l'identité hybride.

---

## Points d'Attention

Cloud Sync ne remplace pas automatiquement tous les scénarios Entra Connect Sync.

Avant de le choisir, il faut vérifier :

* les fonctionnalités nécessaires ;
* les attributs à synchroniser ;
* les besoins de personnalisation ;
* les topologies Active Directory ;
* les dépendances applicatives ;
* les scénarios hybrides déjà en place.

Dans une architecture Enterprise, le choix ne doit pas être fait uniquement parce qu'une solution est plus moderne.

Il doit être fait en fonction des besoins réels.

---

# Comparaison Entra Connect Sync vs Cloud Sync

| Critère                   | Entra Connect Sync         | Entra Cloud Sync             |
| ------------------------- | -------------------------- | ---------------------------- |
| Positionnement            | Historique, riche, mature  | Moderne, cloud-managed       |
| Composant local           | Serveur de synchronisation | Agent léger                  |
| Configuration             | Principalement locale      | Principalement cloud         |
| Personnalisation avancée  | Plus adaptée               | Plus limitée selon scénarios |
| Complexité opérationnelle | Plus élevée                | Plus faible                  |
| Scénarios simples         | Possible                   | Très adapté                  |
| Scénarios complexes       | Très adapté                | À valider                    |
| Multi-forêts              | Possible                   | Très pertinent selon cas     |
| Migration progressive     | Source fréquente           | Cible possible               |
| Supervision               | Locale + cloud             | Cloud + agents               |

---

# Peut-on Utiliser les Deux ?

Oui, certains scénarios permettent de faire coexister Cloud Sync et Connect Sync.

Microsoft documente des scénarios où Cloud Sync et Connect Sync peuvent fonctionner en parallèle, par exemple pour traiter une majorité de cas avec Cloud Sync tout en conservant Connect Sync pour des cas plus spécifiques.

Cependant, cette coexistence doit être strictement maîtrisée.

Il faut éviter :

* la double synchronisation du même objet ;
* les conflits de source d'autorité ;
* les erreurs de filtrage ;
* les incohérences entre forêts ;
* les effets de bord sur les groupes ou les attributs.

Dans un vrai projet client, ce point doit faire l'objet d'une analyse détaillée.

---

# Modes d'Authentification Hybride

Synchroniser les identités ne suffit pas.

Il faut aussi décider comment les utilisateurs s'authentifient.

Microsoft distingue principalement trois grands modèles :

```text
Password Hash Synchronization
Pass-through Authentication
Federation
```

Ces modèles répondent à une question simple :

```text
Où et comment le mot de passe de l'utilisateur est-il validé ?
```

---

# Password Hash Synchronization

## Principe

Password Hash Synchronization, ou PHS, synchronise vers Microsoft Entra ID une version dérivée du hash du mot de passe provenant d'Active Directory local.

L'utilisateur peut ensuite s'authentifier directement auprès de Microsoft Entra ID avec le même mot de passe que celui utilisé en local.

Microsoft décrit PHS comme l'une des méthodes de connexion utilisées pour l'identité hybride.

Schéma simplifié :

```text
Active Directory local
        │
        ▼
Synchronisation du hash de mot de passe
        │
        ▼
Microsoft Entra ID
        │
        ▼
Authentification cloud
```

---

## Avantages

PHS présente plusieurs avantages :

* simplicité ;
* bonne résilience ;
* moins de dépendance à l'infrastructure locale lors de l'authentification ;
* expérience utilisateur cohérente ;
* approche adaptée à beaucoup de scénarios cloud-first.

---

## Points d'Attention

Points à prendre en compte :

* nécessite d'accepter la synchronisation de hashs vers le cloud ;
* doit être expliqué aux équipes sécurité ;
* nécessite une bonne gouvernance des mots de passe locaux ;
* doit être combiné avec MFA et Conditional Access lorsque possible.

---

# Pass-through Authentication

## Principe

Pass-through Authentication, ou PTA, permet à Microsoft Entra ID de transmettre la validation du mot de passe à des agents installés dans l'environnement local.

Le mot de passe est validé contre Active Directory local.

Schéma simplifié :

```text
Utilisateur
        │
        ▼
Microsoft Entra ID
        │
        ▼
Agent PTA local
        │
        ▼
Active Directory local
```

Microsoft présente PTA comme une alternative à Password Hash Synchronization pour bénéficier d'une authentification cloud tout en validant le mot de passe localement.

---

## Avantages

PTA peut être pertinent lorsque l'organisation veut :

* valider les mots de passe directement contre AD local ;
* éviter le modèle PHS pour des raisons internes ;
* conserver une dépendance forte à Active Directory local pour l'authentification.

---

## Points d'Attention

PTA introduit une dépendance aux agents locaux.

Il faut donc prévoir :

* plusieurs agents ;
* haute disponibilité ;
* supervision ;
* tests réguliers ;
* plan de secours ;
* documentation de la configuration.

Si les agents ou l'infrastructure locale sont indisponibles, l'authentification peut être impactée.

---

# Federation / AD FS

## Principe

Dans un modèle fédéré, Microsoft Entra ID délègue l'authentification à une infrastructure de fédération, par exemple AD FS.

Schéma simplifié :

```text
Utilisateur
        │
        ▼
Microsoft Entra ID
        │
        ▼
AD FS / Fédération
        │
        ▼
Active Directory local
```

---

## Cas d'Usage

La fédération peut être rencontrée dans des environnements historiques ou complexes.

Elle peut répondre à certains besoins spécifiques :

* règles d'authentification très personnalisées ;
* intégration avec des infrastructures existantes ;
* scénarios historiques déjà en production ;
* contraintes organisationnelles spécifiques.

---

## Points d'Attention

La fédération apporte une complexité importante.

Elle nécessite :

* serveurs AD FS ;
* proxy ou Web Application Proxy ;
* certificats ;
* haute disponibilité ;
* supervision ;
* sécurité renforcée ;
* procédures de renouvellement ;
* documentation complète.

Une panne de l'infrastructure fédérée peut bloquer les connexions.

Microsoft documente aussi la possibilité d'utiliser Password Hash Synchronization comme mécanisme de secours lorsque la fédération AD FS est utilisée.

---

# Seamless Single Sign-On

Seamless Single Sign-On permet aux utilisateurs connectés à un poste joint au domaine d'accéder plus facilement aux services cloud sans ressaisir systématiquement leurs identifiants.

Il peut être combiné avec Password Hash Synchronization ou Pass-through Authentication.

Microsoft précise que Seamless SSO peut être combiné avec PHS ou PTA, mais ne s'applique pas à AD FS.

---

# Comparaison des Modes d'Authentification

| Critère                                                      | PHS         | PTA                      | Federation              |
| ------------------------------------------------------------ | ----------- | ------------------------ | ----------------------- |
| Complexité                                                   | Faible      | Moyenne                  | Élevée                  |
| Dépendance à l'infrastructure locale pour l'authentification | Faible      | Forte                    | Très forte              |
| Résilience cloud                                             | Bonne       | Dépend des agents        | Dépend d'AD FS          |
| Expérience utilisateur                                       | Bonne       | Bonne                    | Variable selon design   |
| Scénarios modernes                                           | Très adapté | Adapté selon contraintes | À réévaluer             |
| Supervision locale nécessaire                                | Limitée     | Oui                      | Forte                   |
| Recommandation générale Lab                                  | Oui         | Optionnel                | Non recommandé pour Lab |

---

# Critères de Choix Architecturaux

Le choix entre cloud-only, Connect Sync, Cloud Sync, PHS, PTA ou Federation dépend de plusieurs critères.

## Critère 1 – Existant Active Directory

Questions à poser :

* Active Directory local existe-t-il ?
* Combien de forêts ?
* Combien de domaines ?
* Quels objets doivent être synchronisés ?
* Y a-t-il des règles de synchronisation spécifiques ?

---

## Critère 2 – Complexité des Attributs

Questions à poser :

* Les attributs AD sont-ils standards ?
* Des attributs personnalisés sont-ils utilisés ?
* Des applications dépendent-elles de certains attributs ?
* Des règles de transformation sont-elles nécessaires ?

---

## Critère 3 – Mode d'Authentification Souhaité

Questions à poser :

* L'organisation accepte-t-elle PHS ?
* L'authentification doit-elle être validée localement ?
* Une infrastructure AD FS existe-t-elle déjà ?
* Le modèle fédéré est-il encore justifié ?

---

## Critère 4 – Résilience

Questions à poser :

* Que se passe-t-il si le site local est indisponible ?
* Les utilisateurs doivent-ils continuer à accéder aux services cloud ?
* Existe-t-il un plan de secours ?
* PHS peut-il être utilisé comme mécanisme de résilience ?

---

## Critère 5 – Exploitabilité

Questions à poser :

* Qui supervise la synchronisation ?
* Où sont les alertes ?
* Qui traite les erreurs de synchronisation ?
* Les procédures sont-elles documentées ?
* Les versions sont-elles maintenues ?

---

## Critère 6 – Sécurité

Questions à poser :

* Les comptes privilégiés sont-ils synchronisés ?
* Les groupes RBAC Azure viennent-ils d'AD local ?
* Les comptes désactivés localement sont-ils bien désactivés côté cloud ?
* Le cycle de vie des comptes est-il maîtrisé ?
* Les comptes de service sont-ils correctement séparés ?

---

# Risques Fréquents

## Synchroniser des Comptes Privilégiés sans Contrôle

Un compte privilégié local synchronisé vers Entra ID peut devenir un risque important si ses droits cloud ne sont pas maîtrisés.

Les comptes disposant de rôles Entra ID ou Azure RBAC doivent être strictement contrôlés.

---

## Utiliser des Groupes AD Locaux pour Trop de Droits Cloud

Il est courant de réutiliser des groupes AD existants.

Cela peut sembler pratique, mais c'est risqué si les groupes n'ont pas été conçus pour Azure.

Exemple de risque :

```text
Groupe AD historique
        │
        ▼
Synchronisé vers Entra ID
        │
        ▼
Affecté Contributor sur PROD
```

Si l'appartenance au groupe est mal contrôlée, l'accès Azure devient vulnérable.

---

## Ne Pas Superviser la Synchronisation

Une synchronisation cassée peut provoquer des effets silencieux :

* nouvel utilisateur absent du cloud ;
* départ non répercuté ;
* groupe non mis à jour ;
* accès non supprimé ;
* erreurs d'authentification.

La synchronisation doit être surveillée comme un service critique.

---

## Garder AD FS sans Réévaluation

Certaines organisations conservent AD FS uniquement par héritage.

Dans une architecture moderne, il faut régulièrement se demander si la fédération est toujours nécessaire.

Un modèle PHS ou PTA peut parfois réduire la complexité.

---

## Négliger les Comptes Break Glass

Dans un environnement hybride, les comptes Break Glass doivent rester cloud-only.

Ils ne doivent dépendre ni d'AD local, ni d'AD FS, ni d'un outil de synchronisation.

C'est un point critique.

---

# Contrôles d'Audit Hybride

Dans un environnement hybride, les contrôles suivants doivent être prévus :

```text
Version d'Entra Connect Sync
État de synchronisation
Dernière synchronisation réussie
Erreurs de synchronisation
Statut des agents Cloud Sync
Statut des agents Pass-through Authentication
Mode d'authentification utilisé
Présence d'AD FS
Certificats AD FS
Comptes synchronisés à privilèges
Groupes synchronisés utilisés pour RBAC Azure
Comptes désactivés localement mais encore actifs dans le cloud
Comptes cloud privilégiés rattachés à une source locale
```

Ces contrôles seront importants dans les audits IAM.

---

# Positionnement pour MonLabAzure

Pour le Lab MonLabAzure, la première implémentation IAM restera cloud-only.

Pourquoi ?

* simplicité ;
* coût réduit ;
* reproductibilité ;
* moins de dépendances locales ;
* meilleure accessibilité pour le lecteur.

Cependant, le projet MonLabAzure doit rester aligné avec les réalités Enterprise.

C'est pourquoi l'identité hybride est officiellement documentée dans ADR-007 et dans cet article.

Le positionnement est donc le suivant :

```text
Lab MonLabAzure
        │
        ▼
Implémentation cloud-only

Architecture Enterprise
        │
        ▼
Compatibilité avec identité hybride
        │
        ├── Entra Connect Sync
        ├── Entra Cloud Sync
        ├── Password Hash Synchronization
        ├── Pass-through Authentication
        └── Federation / AD FS
```

---

# Recommandation Architecte

Pour un nouveau projet Azure Enterprise, je recommande généralement cette logique d'analyse :

```text
1. Vérifier si Active Directory local est encore la source d'autorité.
2. Identifier les forêts, domaines et groupes critiques.
3. Évaluer Cloud Sync en priorité pour les scénarios simples ou modernes.
4. Conserver ou choisir Connect Sync pour les scénarios complexes.
5. Éviter AD FS sauf besoin clairement justifié.
6. Favoriser PHS lorsque les contraintes de sécurité le permettent.
7. Prévoir des comptes Break Glass cloud-only.
8. Superviser la synchronisation comme un service critique.
```

Ce n'est pas une règle absolue.

C'est une grille de lecture.

Chaque contexte client doit être analysé.

---

# Ce que Nous Ne Déploierons Pas dans cet Article

Cet article est volontairement conceptuel.

Nous ne déploierons pas encore :

* Entra Connect Sync ;
* Entra Cloud Sync ;
* AD FS ;
* agents PTA ;
* infrastructure Active Directory locale.

Ces sujets peuvent faire l'objet de travaux pratiques dédiés plus tard.

Ici, l'objectif est de comprendre le rôle de l'identité hybride avant de construire le modèle IAM cloud de MonLabAzure.

---

# Coûts

L'identité hybride peut introduire des coûts indirects.

Exemples :

* serveurs Windows pour Entra Connect Sync ;
* supervision ;
* sauvegarde ;
* haute disponibilité ;
* maintenance ;
* licences éventuelles ;
* temps d'exploitation ;
* infrastructure AD FS si fédération.

Dans un Lab personnel, ces coûts peuvent devenir inutiles.

C'est une raison supplémentaire pour conserver MonLabAzure en cloud-only dans la première implémentation.

---

# Livrables de cet Article

À ce stade, nous avons :

* compris la différence entre cloud-only et hybride ;
* identifié Entra Connect Sync ;
* identifié Entra Cloud Sync ;
* comparé les deux approches ;
* compris les modes PHS, PTA et Federation ;
* identifié les principaux risques ;
* défini les contrôles d'audit hybride ;
* confirmé le positionnement cloud-only du Lab MonLabAzure.

---

# Conclusion

L'identité hybride est un sujet structurant dans les architectures Azure Enterprise.

Même si MonLabAzure démarre en cloud-only, il serait incomplet d'ignorer Active Directory local, Entra Connect Sync, Cloud Sync et les modes d'authentification hybrides.

Dans les environnements réels, ces composants conditionnent souvent la stratégie IAM, la sécurité, les accès Azure et l'exploitation quotidienne.

Nous avons donc posé le cadre nécessaire.

Dans le prochain article, nous passerons à l'implémentation concrète de notre modèle IAM avec la création des groupes Microsoft Entra ID.
