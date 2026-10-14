# CLAUDE.md — Référence de travail pour MissionPilot

Ce document fixe les décisions, contraintes et méthodes de travail du projet.
Il s'applique à toute intervention de Claude sur ce dépôt. En cas de conflit avec une
instruction ponctuelle du développeur, l'instruction explicite du développeur prévaut ;
signale alors l'écart.

---

## 1. Le projet en bref

MissionPilot est une application web **pédagogique, personnelle et non commerciale** de
pilotage des demandes informatiques, inspirée des besoins génériques d'un service
informatique administratif.

- Ce n'est **pas** un outil officiel. Ne jamais le présenter comme une application du
  ministère de l'Intérieur, de la gendarmerie ou de tout autre service public.
- Toutes les données sont **fictives**. Aucune donnée opérationnelle ou sensible.
- Priorité : **la qualité réelle du logiciel**, pas la quantité de fonctionnalités.

Fonctionnalités cibles (à construire progressivement) : demandes informatiques,
catégories et priorités, transitions de statut contrôlées, affectation à des agents,
unités fictives, historique des changements, indicateurs et statistiques, accès par
rôles, notifications e-mail, interface d'administration claire et accessible.

**État actuel : phase de fondations documentaires.** Ni Drupal, ni Docker, ni aucun code
applicatif ne sont encore en place. Vérifie toujours l'état réel du dépôt avant de
supposer qu'un élément existe.

---

## 2. Méthode de travail obligatoire

À chaque intervention, Claude doit :

1. **Examiner avant de modifier** : lire les fichiers concernés, l'architecture en place
   et l'état Git (branche courante, fichiers modifiés). Ne jamais écraser un fichier
   sans en avoir lu le contenu.
2. **Vérifier les versions et la compatibilité** des dépendances (Drupal, PHP,
   PostgreSQL, FrankenPHP, Node.js, bibliothèques) au moment de l'implémentation, à
   partir de sources à jour. Ne jamais supposer qu'une combinaison est compatible.
   Vérifier aussi la **licence** et l'état de **maintenance** de chaque dépendance.
3. **Présenter les décisions importantes avant toute modification structurelle**
   (arborescence, namespaces, nouvelle dépendance, changement d'architecture) et
   attendre la validation du développeur.
4. **Limiter chaque tâche à un périmètre cohérent.** Ne pas élargir le périmètre sans
   accord ; signaler les améliorations repérées plutôt que les appliquer d'office.
5. **Livrer les tests et la documentation avec la fonctionnalité**, y compris la mise à
   jour du README si l'état du projet change.
6. **Exécuter les vérifications réellement disponibles** (tests, analyse statique,
   linters) et rapporter leur résultat fidèlement.
7. **Examiner le diff** (`git status`, `git diff`) avant de conclure.
8. **Signaler explicitement** les tests non exécutés, les vérifications impossibles et
   les limitations connues.
9. **Ne jamais créer de commit** sans autorisation explicite. Ne jamais pousser une
   branche sans instruction explicite.
10. **Ne jamais présenter une fonctionnalité comme terminée** si elle n'a pas été
    vérifiée. Ne jamais inventer de résultat de test.

Compte rendu attendu en fin de tâche : fichiers modifiés et leur rôle, décisions prises,
vérifications réellement effectuées, points restant à arbitrer, `git status`, résumé du
diff.

---

## 3. Stack technique cible

| Domaine | Choix | Remarques |
|---|---|---|
| Backend | Drupal 11, PHP, Composer | Version mineure et versions précises à vérifier à l'implémentation |
| Base de données | PostgreSQL | Version à aligner sur les exigences de Drupal retenu |
| Serveur | FrankenPHP | Remplace Apache et PHP-FPM ; **mode classique** au départ |
| Environnement | Docker Compose, Dev Containers, VS Code | |
| Outillage front | Node.js, npm | |
| Rendu | Twig, Single Directory Components (SDC) | SDC après vérification de compatibilité |
| JavaScript | JavaScript natif, ciblé | Pas de React, pas de SPA |
| CSS | Variables de design + composants réutilisables | |
| Graphiques | Chart.js (**envisagé**) | Licence, compatibilité et mode de distribution à vérifier avant intégration |
| Icônes | Jeu d'icônes indépendant sous licence compatible | Choix ultérieur |
| E-mails | Templates Twig + transport SMTP configurable | Mailpit en développement |

Règles associées :

- **Mode worker FrankenPHP** : expérimentation distincte, sur une branche
  `experiment/...` dédiée, seulement après vérification de la compatibilité avec Drupal
  et des mécanismes de réinitialisation d'état entre requêtes. Jamais sur le flux
  principal sans décision explicite.
- **Rendu côté serveur privilégié**, interactions JavaScript ciblées.
- **Graphiques accessibles** : chaque graphique s'accompagne, lorsque c'est pertinent,
  d'une alternative textuelle ou d'un tableau de données.

---

## 4. Identité visuelle — pas de DSFR

Décision ferme : **aucune bibliothèque officielle du Système de Design de l'État** dans
les dépendances du projet. Cela inclut le DSFR, DSFR Chart, les icônes DSFR et DSFR
Mail. **Ne pas proposer de les réintroduire.**

- Le projet développe **son propre système de composants Twig** et sa propre identité
  visuelle, documentée dans son propre système de design.
- Ne pas reproduire les éléments distinctifs réservés de l'identité graphique de l'État
  (Marianne, bloc-marque, typographie et charte réservées, etc.) ni laisser croire à un
  service public officiel.
- Ne copier aucune ressource graphique (icônes, logos, polices, images) sans avoir
  vérifié les droits applicables.

Principes d'ergonomie retenus : hiérarchie visuelle claire, navigation prévisible,
formulaires lisibles, tableaux structurés, états et priorités identifiables (pas
uniquement par la couleur), couleurs sobres, contrastes suffisants, navigation complète
au clavier, messages d'erreur compréhensibles.

---

## 5. Architecture : DDD et CQRS

Appliquer ces principes de façon **cohérente et justifiée**, pas de manière théorique
exhaustive.

### Couches

- **Domain** — concepts et règles métier : demande, statut, priorité, règles de
  transition, identifiants métier, invariants. **Aucune dépendance** vers Drupal, Twig,
  PostgreSQL ou HTTP. Testable sans framework.
- **Application** — orchestration des cas d'utilisation : commandes et leurs handlers,
  requêtes et leurs handlers, DTO, ports (interfaces) nécessaires. Aucune logique de
  présentation ; aucune dépendance aux contrôleurs Drupal.
- **Infrastructure** — implémentations techniques : persistance Drupal, repositories,
  cache, services techniques, transports de notification. Les dépendances Drupal y sont
  confinées autant que possible (avec les adaptateurs nécessaires).
- **Presentation** — contrôleurs, formulaires, points d'entrée HTTP, préparation des
  données d'affichage, templates Twig et composants. **Contrôleurs minces** qui délèguent
  aux cas d'utilisation.

### Règle de dépendance

Les dépendances pointent vers le domaine : Presentation → Application → Domain ;
Infrastructure implémente les ports définis par Application/Domain. Le domaine ne
dépend de rien d'extérieur.

La structure exacte des dossiers et namespaces sera **proposée au développeur** au moment
de l'implémentation, en tenant compte des conventions Drupal (modules, PSR-4) et de la
testabilité.

### CQRS

- Les **commandes** modifient l'état métier.
- Les **requêtes** lisent et préparent les résultats pour la présentation.
- **Persistance commune** tant qu'aucun besoin démontré ne justifie davantage.
- **Interdit sans justification concrète** : microservices, Kafka, bus de messages
  distribué, bases de lecture séparées, event sourcing.

### Sobriété des abstractions

Ne pas créer de classes, d'interfaces ou de dossiers uniquement pour remplir une
arborescence. Chaque abstraction répond à un besoin identifiable (testabilité,
substitution réelle, frontière de couche).

---

## 6. Composants Twig

Bibliothèque de composants réutilisables construite progressivement, en privilégiant les
**SDC natifs de Drupal** lorsque c'est adapté.

Composants envisagés : AppShell, navigation latérale, en-tête de page, boutons, badges,
cartes, indicateurs statistiques, tableaux, champs de formulaire, alertes, pagination,
états vides, conteneurs de graphiques, modales de confirmation (seulement si un besoin
le justifie).

Règles :

- Chaque composant a un **rôle précis** et des **paramètres explicites**.
- **Aucune règle métier dans les templates.** Un composant reçoit des données déjà
  préparées ; il n'interroge jamais la base de données.
- Accessibilité intégrée dès la conception (sémantique HTML, focus visible, ARIA
  seulement si nécessaire).
- Chaque composant implémenté est documenté.

---

## 7. Notifications par e-mail

Séparer explicitement :

1. la **décision applicative** de notifier ;
2. la **préparation du contenu** du message ;
3. le **rendu** du template Twig ;
4. le **transport** SMTP ;
5. la **configuration du transport** selon l'environnement.

- Le domaine ne dépend jamais d'un transport SMTP.
- **Mailpit** capture les messages en développement : aucun envoi réel vers de vrais
  destinataires, aucun service d'envoi externe pour les tests locaux.
- Le système doit permettre de tester le contenu, les destinataires attendus et les
  erreurs pertinentes.
- Templates d'e-mail responsives, lisibles, accessibles, avec l'identité visuelle du
  projet.
- Les notifications n'exposent aucune donnée non autorisée pour le destinataire.
- Gestion des erreurs, reprise et cohérence transactionnelle : à définir et à faire
  valider lors de l'implémentation.

---

## 8. Stratégie de tests

| Niveau | Cible | Outil pressenti |
|---|---|---|
| Unitaires | Règles métier, transitions de statut, validations, objets-valeurs, calculs statistiques, cas limites | PHPUnit, sans démarrer Drupal |
| Cas d'utilisation | Commandes, handlers, interactions avec les repositories, erreurs métier, résultats de requêtes | PHPUnit avec doublures de test |
| Intégration | Intégration Drupal, persistance, services techniques | Tests Kernel Drupal |
| Fonctionnels | Connexion/déconnexion, permissions, création de demandes, transitions, listes et détails, parcours principaux, notifications | Tests fonctionnels Drupal ; tests navigateur si nécessaire |
| Qualité | Analyse statique, style de code | À choisir et vérifier |
| Accessibilité | Contrôles automatisés pertinents + vérifications manuelles des parcours critiques | À choisir et vérifier |

Règles :

- L'accessibilité n'est **jamais** considérée comme validée par les seuls tests
  automatisés.
- Aucun test n'est déclaré réussi sans avoir été réellement exécuté.
- Les tests non exécutés sont signalés explicitement, avec la raison.

---

## 9. Sécurité

- **Aucun secret commité.** Les fichiers `.env` réels sont exclus de Git ; les
  `.env.example` restent versionnés et ne contiennent que des valeurs fictives.
- Les identifiants de base de données et autres secrets sont lus depuis l'environnement,
  jamais écrits en dur dans un fichier suivi par Git.
- Données de démonstration **fictives** uniquement.
- Contrôle d'accès **côté serveur**, permission vérifiée pour chaque opération protégée.
- Les caches ne divulguent pas de données entre utilisateurs (contextes et tags de cache
  Drupal corrects).
- Formulaires protégés contre les soumissions non autorisées (jetons CSRF de Drupal).
- Les erreurs techniques ne révèlent aucune information sensible.
- Authentification : celle de Drupal, adaptée aux besoins réels. **Pas de JWT** sans API
  indépendante qui le justifie.
- Dépendances choisies en tenant compte de leur licence et de leur maintenance.

---

## 10. Git et commits

Branches (kebab-case) :

- `main` : branche stable ;
- `develop` : intégration ;
- `feature/<nom>` : nouvelles fonctionnalités ;
- `fix/<nom>` : corrections ;
- `experiment/<nom>` : expérimentations (ex. mode worker FrankenPHP).

Messages au format **Conventional Commits** :

```
feat(infrastructure): add FrankenPHP development environment
feat(notifications): add local email delivery
test(domain): cover request status transitions
docs(architecture): document application boundaries
```

Règle : **un commit = une unité de travail cohérente**, avec ses tests, sa documentation
et la mise à jour du README si nécessaire. Commits lisibles et ciblés.

Rappel : aucun commit sans autorisation explicite, aucun push sans instruction explicite.

---

## 11. Documentation

La documentation fait partie du développement :

- README (toujours fidèle à l'état réel du projet) ;
- ce CLAUDE.md (à mettre à jour quand une décision change) ;
- environnement, installation, commandes de développement et de test ;
- décisions d'architecture importantes sous forme d'**ADR** ;
- conventions de contribution ;
- documentation des composants réutilisables une fois implémentés.

Ne pas créer de documents vides pour remplir une arborescence.
