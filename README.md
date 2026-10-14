# MissionPilot

Application web pédagogique de pilotage des demandes informatiques.

> **Projet personnel, pédagogique et non commercial.**
> MissionPilot s'inspire des besoins génériques d'un service informatique administratif.
> Ce n'est **pas** un outil officiel : il n'est ni produit, ni utilisé, ni validé par le
> ministère de l'Intérieur, la gendarmerie ou tout autre service public.
> Toutes les données manipulées sont **fictives** ; l'application n'est pas destinée à
> traiter des données opérationnelles ou sensibles.

---

## État du développement

**Le projet est en phase d'initialisation.** À ce jour, le dépôt contient uniquement :

- ce `README.md` ;
- `CLAUDE.md`, la référence des décisions et méthodes de travail ;
- `.gitignore`.

Ne sont **pas encore** en place :

- l'environnement Docker (Docker Compose, FrankenPHP, PostgreSQL, Mailpit) ;
- l'installation de Drupal et des dépendances Composer / npm ;
- le code applicatif, les composants Twig et les tests ;
- l'intégration continue.

L'environnement de développement est **encore à construire**. Ce README sera mis à jour
à chaque étape pour refléter l'état réel du projet.

---

## Objectif et contexte

MissionPilot sert de support pour démontrer, sur un cas concret, la capacité à :

- apprendre rapidement Drupal ;
- concevoir une application PHP maintenable ;
- appliquer les principes DDD et CQRS ;
- construire une interface Twig fondée sur des composants réutilisables ;
- mettre en place des tests pertinents ;
- documenter les décisions techniques ;
- maîtriser un environnement Docker ;
- concevoir une application accessible et sécurisée.

La priorité est la **qualité du logiciel**, et non la quantité de fonctionnalités.

---

## Fonctionnalités prévues

Aucune n'est encore implémentée. L'application permettra progressivement de :

- créer et suivre des demandes informatiques ;
- gérer des catégories et des priorités ;
- contrôler les transitions de statut ;
- affecter les demandes à des agents ;
- associer les demandes à des unités fictives ;
- consulter l'historique des changements ;
- afficher des indicateurs et des statistiques ;
- gérer les accès selon les rôles ;
- envoyer des notifications par e-mail ;
- consulter les données dans une interface d'administration claire et accessible.

---

## Stack technique cible

Les versions précises seront arrêtées au moment de l'implémentation, après vérification
de leur compatibilité.

| Domaine | Technologie |
|---|---|
| Backend | Drupal 11, PHP, Composer |
| Base de données | PostgreSQL |
| Serveur applicatif | FrankenPHP (mode classique ; remplace Apache et PHP-FPM) |
| Environnement | Docker Compose, Dev Containers, VS Code |
| Outillage front | Node.js, npm |
| Rendu | Twig, Single Directory Components (SDC) de Drupal |
| Interactions | JavaScript natif, ciblé |
| Styles | CSS fondé sur des variables de design et des composants |
| Graphiques | Chart.js — *envisagé*, licence et compatibilité à vérifier avant intégration |
| Icônes | Jeu d'icônes indépendant sous licence compatible — *choix à venir* |
| E-mails | Templates Twig, transport SMTP configurable, Mailpit en développement |

Choix structurants :

- **Rendu côté serveur** et JavaScript ciblé : pas de React ni de SPA.
- Le **mode worker de FrankenPHP** fera l'objet d'une expérimentation séparée, sur une
  branche dédiée, après vérification de sa compatibilité avec Drupal.

---

## Architecture envisagée

L'architecture s'inspire du **Domain-Driven Design** et du **Command Query
Responsibility Segregation**, appliqués de façon pragmatique.

```
Presentation    contrôleurs Drupal minces, formulaires, préparation de l'affichage, Twig
     │
     ▼
Application     commandes / requêtes et leurs handlers, DTO, ports
     │
     ▼
Domain          demande, statut, priorité, règles de transition, invariants
     ▲
     │ implémente les ports
Infrastructure  persistance Drupal, repositories, cache, transport des notifications
```

- **Domain** : règles métier pures, sans dépendance à Drupal, Twig, PostgreSQL ou HTTP,
  testables indépendamment du framework.
- **Application** : orchestre les cas d'utilisation, sans logique de présentation.
- **Infrastructure** : confine les dépendances techniques et Drupal.
- **Presentation** : délègue aux cas d'utilisation et met en forme les résultats.

Côté CQRS, les **commandes** modifient l'état métier et les **requêtes** lisent et
préparent les données d'affichage. La **persistance reste commune** : pas de base de
lecture séparée, de bus distribué ni de microservices sans besoin démontré.

La structure exacte des dossiers et des namespaces sera définie à l'implémentation, en
tenant compte des conventions Drupal et de la testabilité.

---

## Interface et composants Twig

Le projet développe **sa propre bibliothèque de composants Twig** et sa propre identité
visuelle. Il n'utilise pas les bibliothèques officielles du Système de Design de l'État
(DSFR, DSFR Chart, icônes DSFR, DSFR Mail) et ne reproduit pas les éléments réservés de
l'identité graphique de l'État.

Composants envisagés : structure d'application, navigation latérale, en-tête de page,
boutons, badges, cartes, indicateurs statistiques, tableaux, champs de formulaire,
alertes, pagination, états vides, conteneurs de graphiques.

Principes :

- composants à rôle précis et à paramètres explicites, en SDC lorsque c'est adapté ;
- aucune règle métier dans les templates : les composants reçoivent des données déjà
  préparées ;
- documentation de chaque composant une fois implémenté.

---

## Accessibilité

Objectifs poursuivis dès la conception :

- structure sémantique et hiérarchie visuelle claire ;
- navigation complète au clavier et focus visible ;
- contrastes suffisants ;
- états et priorités identifiables sans dépendre uniquement de la couleur ;
- formulaires lisibles et messages d'erreur compréhensibles ;
- graphiques accompagnés d'une alternative textuelle ou d'un tableau de données.

Des contrôles automatisés seront ajoutés, complétés par des vérifications manuelles des
parcours critiques : les tests automatisés seuls ne suffisent pas à valider
l'accessibilité.

---

## Notifications et Mailpit

Les notifications par e-mail sépareront la décision de notifier, la préparation du
contenu, le rendu du template Twig, le transport SMTP et sa configuration par
environnement.

En développement, **Mailpit** sera ajouté à Docker Compose pour intercepter tous les
messages : aucun e-mail ne sera transmis à de véritables destinataires et aucun service
d'envoi externe ne sera nécessaire. Mailpit fournira une interface web pour consulter
les messages capturés.

---

## Stratégie de tests

| Niveau | Contenu |
|---|---|
| Unitaires | Règles métier, transitions de statut, validations, objets-valeurs, calculs statistiques, cas limites — sans démarrer Drupal lorsque c'est pertinent |
| Cas d'utilisation | Commandes, handlers, interactions avec les repositories, erreurs métier, résultats de requêtes — avec doublures de test |
| Intégration | Intégration avec Drupal, persistance et services techniques (tests Kernel Drupal) |
| Fonctionnels | Connexion, permissions, création de demandes, transitions, listes et détails, notifications ; tests navigateur si nécessaire |
| Qualité | Analyse statique, style de code, contrôles d'accessibilité automatisés |

Aucun test n'existe encore.

---

## Démarrage

*Non disponible.* Les instructions d'installation et de lancement seront ajoutées avec
l'environnement Docker.

## Commandes de test

*Non disponibles.* Elles seront documentées une fois les outils de test configurés.

---

## Sécurité

- Aucun secret n'est versionné : les fichiers `.env` réels sont ignorés, seuls les
  fichiers `.env.example` (valeurs fictives) sont suivis.
- Contrôle d'accès vérifié côté serveur pour chaque opération protégée.
- Formulaires protégés contre les soumissions non autorisées.
- Caches sans fuite de données entre utilisateurs.
- Erreurs techniques sans divulgation d'informations sensibles.
- Dépendances sélectionnées selon leur licence et leur maintenance.

---

## Conventions de contribution

Branches, en kebab-case :

| Branche | Usage |
|---|---|
| `main` | Branche stable |
| `develop` | Intégration |
| `feature/<nom>` | Nouvelle fonctionnalité |
| `fix/<nom>` | Correction |
| `experiment/<nom>` | Expérimentation |

Messages de commit au format [Conventional Commits](https://www.conventionalcommits.org/),
par exemple :

```
feat(notifications): add local email delivery
test(domain): cover request status transitions
docs(architecture): document application boundaries
```

Un commit correspond à une unité de travail cohérente, accompagnée de ses tests, de sa
documentation et, si nécessaire, de la mise à jour de ce README.

Les décisions d'architecture importantes seront consignées sous forme d'ADR.

---

## Limites connues

- Aucun environnement exécutable : Docker, Drupal et PostgreSQL ne sont pas installés.
- Aucune fonctionnalité implémentée, aucun test, aucune intégration continue.
- Les versions de Drupal, PHP, PostgreSQL, FrankenPHP et Node.js restent à arrêter après
  vérification de compatibilité.
- Chart.js et le jeu d'icônes restent à valider (licence, compatibilité, distribution).
- La compatibilité du mode worker de FrankenPHP avec Drupal n'est pas évaluée.
- Le projet n'a pas vocation à être déployé en production.
