# Spécification technique

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C
> **Livrable :** Phase 1, Conception et spécification
> **Date de livraison :** 10 octobre 2026

---

## 1. Introduction

### 1.1 Contexte

Ce document présente la conception de la plateforme SAÉ Recommender, réalisée dans le cadre de la SAÉ du troisième semestre de BUT 2 Informatique (année 2026-2027). Il correspond à la phase 1 du projet, consacrée à la prise en main du concept de blockchain, à la conception de l'architecture et de l'organisation des données, et à la définition du protocole de communication.

Il s'adresse à toutes les parties prenantes du projet, y compris celles qui ne participent pas au développement. Les concepts sont donc expliqués, et les éléments les plus techniques (schémas, diagrammes) sont regroupés dans des annexes. Ce document est complété par les fichiers Markdown du répertoire `docs/`, avec lesquels il reste cohérent.

### 1.2 Objectif du projet

L'objectif est de réaliser une plateforme client-serveur de recommandations structurées autour de cartes. Un serveur unique communique par sockets avec plusieurs clients. Il propose la création de cartes, la recommandation, la répudiation d'une recommandation, la confrontation entre cartes (« battle »), la légitimation, la revendication et l'authentification. Chaque action validée par le serveur est inscrite dans une blockchain privée simplifiée, qui sert de support de traçabilité et de contrôle d'intégrité. Cette blockchain est sauvegardée dans une base de données PostgreSQL, ce qui permet de restaurer le contexte de travail au redémarrage du serveur.

### 1.3 Thème choisi

Le groupe a choisi le thème des **langages de programmation**. Chaque carte représente un langage (Python, Rust, C++, etc.). La communauté peut recommander ses langages préférés et les confronter selon leur domaine d'application : web, développement de jeux, science des données, programmation système ou mobile. Ce thème a l'avantage d'offrir un critère de proximité naturel entre les cartes : le domaine.

### 1.4 Technologies utilisées

Le cahier des charges impose les technologies suivantes, que le groupe respecte : environnement Linux Debian, langage C pour le serveur, langage Java avec JavaFX pour les clients, algorithme de hachage SHA-256, base de données PostgreSQL et gestion de version avec Git. Côté serveur, les bibliothèques autorisées sont OpenSSL (hachage), libpq (PostgreSQL) et les interfaces de programmation système (sockets, threads POSIX). Côté client, JavaFX et JUnit sont utilisés.

Un point reste à trancher avec l'enseignant responsable : le format JSON choisi pour les messages demande de lire et d'écrire du JSON en C et en Java, alors qu'aucune bibliothèque dédiée n'est autorisée. Le groupe écrira soit un petit analyseur JSON dans chaque langage, soit demandera l'autorisation d'utiliser une bibliothèque.

---

## 2. Répartition des rôles

Le groupe est composé de cinq étudiants répartis en trois équipes. L'équipe Client (deux étudiants) développe l'application Java / JavaFX. L'équipe Serveur (deux étudiants) développe le serveur multitâche en C et la communication par sockets. L'équipe Blockchain / BDD (un étudiant) développe la blockchain, la preuve de travail et la persistance dans PostgreSQL.

Chaque équipe conçoit, code et teste sa propre partie. En complément, des rôles transversaux sont attribués : chef de projet, responsable qualité, responsable Git et responsable de la documentation. Les documents qui engagent tout le groupe (règles métier, protocole applicatif, codes d'erreur, diagramme de déploiement) sont rédigés ensemble et validés par les trois équipes, car une modification a des conséquences sur le client, le serveur et la base de données.

Le projet est découpé en quatre phases : conception (jusqu'au 10 octobre 2026), codage et tests (12 octobre au 18 décembre 2026), consolidation de la qualité (4 au 15 janvier 2027) et présentation (18 au 21 janvier 2027). Le détail de l'organisation est dans `repartition-roles.md`.

---

## 3. Architecture du serveur

Le serveur est un processus unique, écrit en C, qui joue le rôle de point central de la plateforme. Il est le seul à valider les actions, à écrire dans la blockchain et à accéder à la base de données. Les clients ne font que lui envoyer des demandes et afficher ses réponses.

Au démarrage, le serveur consulte la base de données. S'il existe une sauvegarde de la blockchain, il la charge en mémoire et reconstruit le contexte de la plateforme (cartes, propriétaires, recommandations, battles, légitimations). Sinon, il crée une nouvelle blockchain dont le premier bloc est le bloc « genesis ». Si la base est indisponible, il considère qu'aucune sauvegarde n'existe.

Le serveur attend ensuite les connexions des clients dans une boucle d'attente. Il accepte au plus vingt clients simultanés (`MAX_CLIENTS = 20`) ; au-delà, il répond `SERVER_BUSY` et ferme la connexion. Le serveur est multitâche : chaque client est pris en charge par un fil d'exécution (thread POSIX) dédié, de sorte que le traitement d'une demande, y compris le calcul cryptographique du minage d'un bloc, ne bloque pas les autres clients. Les données partagées (blockchain, état des cartes, battles) sont protégées par des verrous pour éviter les accès concurrents incohérents.

Pour chaque demande, le serveur vérifie les règles métier. Si la demande est valide, il l'exécute, mine un nouveau bloc, le sauvegarde en base puis répond au client. Si elle est refusée, rien n'est inscrit dans la blockchain. Les battles sont suivies par un chronomètre de 60 secondes : à son expiration, le serveur désigne la gagnante, applique le résultat et prévient tous les clients. Enfin, le serveur offre deux fonctions de maintenance : la vérification de la cohérence de la blockchain et l'arrêt propre, qui termine les traitements en cours, sauvegarde l'état, ferme les sockets et libère les ressources.

Le détail de l'architecture du serveur est décrit dans `architecture-serveur.md`, et son diagramme de classes est référencé dans l'**Annexe C**.

---

## 4. Architecture des clients

Le client est une application Java avec interface graphique JavaFX. Il permet de se connecter au serveur (adresse IP et port), de créer des cartes, de recommander ou répudier, de lancer une battle et de voter, de demander une légitimation, de revendiquer et d'authentifier une carte, et de consulter ses cartes, ses recommandations, les utilisateurs connectés et les battles. Le client ne contient aucune règle décisive : il envoie une demande, attend la réponse du serveur et affiche le résultat. Il n'accède jamais à la base de données.

Le client est organisé en trois couches. La couche **Vue** contient les écrans JavaFX et leurs contrôleurs (connexion, écran principal, cartes, création de carte, battles, profil). La couche **Modèle / Services** contient les objets du modèle (`Card`, `User`, `Battle`), l'état courant du client (`ClientState`), le service qui construit et envoie les demandes (`RequestService`) et le gestionnaire qui traite les réponses (`ResponseHandler`). La couche **Réseau** contient la connexion au serveur (`ServerConnection`) et le lecteur de messages (`MessageReader`). La vue ne parle jamais directement à la socket : elle passe par `RequestService`.

Deux types de threads coexistent : le thread JavaFX, qui gère l'affichage et ne doit jamais être bloqué, et un thread de lecture réseau, qui attend en continu les messages du serveur. Ce second thread est nécessaire car le serveur peut envoyer un message sans qu'on le lui ait demandé (début ou résultat d'une battle, nouvel utilisateur connecté). Quand un message arrive, `ResponseHandler` met à jour `ClientState`, puis l'écran est rafraîchi avec `Platform.runLater`, car seul le thread JavaFX peut modifier l'interface.

Le client grise les boutons lorsqu'une action est évidemment impossible (par exemple, lancer une battle avec une VA inférieure à 5), mais le serveur refait toujours les vérifications. En cas de perte de connexion ou d'arrêt du serveur, le client affiche un message et revient à l'écran de connexion. Un client automatique (« bot ») est envisagé mais facultatif ; s'il existe, il réutilise les couches Modèle et Réseau sans interface graphique et se déclare comme bot à la connexion.

Les détails sont dans `architecture-client.md`. Le parcours de l'utilisateur dans l'interface est présenté dans l'**Annexe A**, et le diagramme de classes du client dans l'**Annexe C**.

---

## 5. Organisation des données

### 5.1 Côté serveur

Le serveur conserve en mémoire la blockchain (une liste chaînée de blocs), ainsi qu'un état courant dérivé de celle-ci : la liste des cartes, les utilisateurs connectés, les recommandations actives, les battles en cours avec leurs votes, et les compteurs nécessaires aux règles (par exemple le nombre de cartes actives d'un utilisateur). Chaque client connecté est associé à une structure qui contient sa socket, son identifiant d'utilisateur et son état. Ces structures sont décrites dans `donnees-serveur.md`.

### 5.2 Côté client

Le client ne garde que les données utiles à l'affichage. Une carte contient son identifiant (`c_001`), son nom, le créateur historique du langage, son année de création, son domaine, sa description, son propriétaire et son créateur, sa valeur d'appréciation, son statut, les informations d'authentification (`estAuthentifiee`, `authentifieePar`, `dateAuthentification`), le nombre de victoires (`winCount`), l'indicateur `isLegitimate` et sa date de création. Un utilisateur possède un identifiant (`u_001`), un nom, un indicateur de légitimation, un indicateur de bot et son nombre de recommandations valides. Une battle contient les deux cartes engagées, leur domaine commun, son statut, sa durée, les votes et la carte gagnante. L'ensemble forme l'état du client (`ClientState`), rempli à la connexion avec le contexte envoyé par le serveur. La référence reste toujours le serveur. Le détail est dans `donnees-client.md`.

### 5.3 Base de données

La base PostgreSQL est située sur le serveur `linserv-info-01`, dans le réseau de l'IUT. L'enregistrement de la blockchain dans la base constitue la sauvegarde du contexte de travail : au démarrage, le serveur relit la table des blocs pour reconstruire son état. Pour éviter de parcourir toute la blockchain à chaque consultation, des tables d'état courant (cartes, propriétaires, recommandations actives) peuvent être conservées en complément. Ces tables sont des données dérivées : le serveur les met à jour à chaque action validée, dans la même transaction que l'enregistrement du bloc, afin que les deux écritures soient validées ensemble. La blockchain reste la référence, et les tables d'état peuvent toujours être reconstruites à partir de son historique en cas de perte ou d'incohérence. Les clients n'accèdent jamais à la base. Le schéma relationnel est décrit dans `schema-bdd.md`.

Les diagrammes de classes correspondant à ces trois parties sont dans l'**Annexe C**.

---

## 6. Structures de la blockchain

La blockchain du projet est une version simplifiée de celles des cryptomonnaies : elle est privée, utilisée par un serveur unique (pas de réseau de serveurs) et repose sur une preuve de travail. C'est une liste chaînée de blocs, reliés entre eux par leurs empreintes cryptographiques.

Chaque bloc contient un identifiant unique, un horodatage, les données d'une action validée (type d'action, utilisateur, cartes concernées), le hash du bloc précédent, un nonce et son propre hash. Le premier bloc, « genesis », a un hash précédent nul. Le hash d'un bloc est calculé avec l'algorithme SHA-256 à partir de l'identifiant, de l'horodatage, des données de l'action, du hash précédent et du nonce. En C, un pointeur de chaînage relie les blocs en mémoire, mais il n'entre pas dans le calcul du hash.

Le minage consiste à faire varier le nonce jusqu'à obtenir un hash qui commence par un certain nombre de zéros hexadécimaux : ce nombre est la difficulté, fixée à 3 ou 4 pour obtenir des temps de calcul raisonnables. Comme le hash d'un bloc dépend du hash du précédent, modifier un bloc ancien change son hash et invalide tous les suivants, ce qui rend toute altération détectable.

Le serveur propose une vérification de la cohérence qui contrôle, pour chaque bloc, la validité du hash, la cohérence du hash précédent et le respect de la preuve de travail. Un test d'altération sera réalisé en phase 2 : une donnée d'un bloc sera modifiée volontairement et la vérification devra détecter l'anomalie. Chaque carte possède un identifiant unique dans la blockchain. Lorsqu'une carte perd une battle, ses blocs ne sont jamais supprimés : seul son statut courant change.

Les structures détaillées sont dans `structures-blockchain.md`, et leur diagramme de classes dans l'**Annexe C**.

---

## 7. Règles métier

Les règles métier complètes sont dans `regles-metier.md`. Cette section en explique les principes et les conséquences.

### 7.1 Valeur d'Appréciation (VA)

La valeur d'une carte mesure son attractivité. Elle est égale au nombre de recommandations actives, auquel s'ajoute un bonus fixe de 10 points si la carte est authentifiée. Un utilisateur ne peut avoir qu'une recommandation active par carte. Il n'existe pas de vote négatif : un utilisateur peut seulement annuler (répudier) une recommandation donnée, ce qui retire un point à la carte. L'historique de l'action reste dans la blockchain même après la répudiation.

### 7.2 Battle

Deux cartes peuvent s'affronter si et seulement si elles ont le même domaine : ce critère de proximité évite de comparer des langages qui n'ont pas le même usage (par exemple Rust et PHP). Les deux cartes doivent être actives ou authentifiées, avoir une VA d'au moins 5, et ne participer à aucune autre battle. Le demandeur doit être propriétaire d'au moins une des deux cartes. Pendant 60 secondes, tout utilisateur connecté peut voter, sauf les propriétaires des deux cartes ; chaque utilisateur dispose d'un seul vote. Ce choix « un utilisateur, un vote » est simple et transparent.

À la fin du temps, la carte qui a le plus de votes gagne. En cas d'égalité, la carte qui avait la VA initiale la plus élevée l'emporte, puis, si l'égalité persiste, celle dont l'identifiant est le plus petit : il y a donc toujours un vainqueur. La gagnante récupère toutes les recommandations actives de la perdante et son compteur de victoires augmente. La perdante perd ses recommandations et devient inactive : elle ne peut plus être recommandée ni engagée dans une battle, mais son historique reste dans la blockchain. Si un utilisateur se déconnecte pendant une battle, le vote continue jusqu'au bout.

### 7.3 Légitimation

La légitimation permet à un utilisateur d'être reconnu comme représentant légitime. Elle est accordée à tout utilisateur qui a donné au moins 3 recommandations valides. Dans le cadre de la SAÉ, ce processus est simulé : aucune vérification d'identité réelle n'est faite. L'utilisateur légitimé est enregistré comme tel dans la blockchain. Il peut alors revendiquer des cartes et authentifier les siennes.

La revendication permet à un utilisateur légitimé de prendre le contrôle d'une carte créée par un autre utilisateur non légitimé. Le transfert est direct, validé par le serveur, sans acceptation de l'ancien propriétaire : le propriétaire de la carte change, mais le créateur reste inchangé pour conserver la mémoire de l'auteur d'origine.

### 7.4 Authentification

Le propriétaire légitimé d'une carte peut l'authentifier. La carte reçoit alors le statut « authentifiée », un badge officiel est affiché dans le client, elle gagne définitivement 10 points de VA, et ses caractéristiques principales (nom, domaine, créateur historique, année de création) sont verrouillées : elles ne peuvent plus être modifiées.

### 7.5 Limites

Le serveur accepte au maximum 20 clients connectés simultanément. Un utilisateur peut posséder au maximum 10 cartes actives.

### 7.6 Points à trancher

Certains cas ne sont pas couverts par les règles initiales et seront décidés par le groupe : la légitimation est-elle globale ou propre à un langage (le protocole actuel suppose une légitimation globale), un utilisateur déjà légitimé le reste-t-il si ses recommandations valides repassent sous 3, et quel code d'erreur utiliser dans certains cas limites. Ces points sont listés dans `regles-metier.md` et `cas-erreur.md`.

---

## 8. Protocole applicatif

Toute la communication entre les clients et le serveur passe par des sockets TCP ; aucune autre interface, notamment REST, n'est utilisée. Les messages sont au format JSON, encodés en UTF-8, avec un message par ligne terminé par le caractère `\n`. Le groupe a retenu JSON pour sa lisibilité et sa facilité de débogage, et le délimiteur de fin de ligne pour sa simplicité de lecture en Java comme en C. La connexion est permanente pendant toute la session, car le serveur doit pouvoir prévenir les clients à tout moment.

Il y a trois sortes de messages. Une **demande** va du client au serveur ; une **réponse** revient du serveur au client ; une **notification** est envoyée par le serveur sans demande. Chaque message porte un champ `type`. Les demandes portent un identifiant `requestId`, que le serveur recopie dans la réponse, pour associer chaque réponse à sa demande même si des notifications arrivent entre-temps.

### 8.1 Actions

Le client peut demander les actions suivantes : se connecter (`login`), créer une carte (`create_card`), recommander (`recommend`), répudier (`repudiate`), lancer une battle (`launch_battle`), voter (`vote`), demander la légitimation (`request_legitimation`), revendiquer une carte (`claim_card`), authentifier une carte (`authenticate_card`), redemander son contexte (`get_context`), vérifier la blockchain (`verify_blockchain`) et se déconnecter (`disconnect`). Chaque demande reçoit une réponse de succès (par exemple `recommend_ok`) ou un message d'erreur. Une demande acceptée est inscrite dans la blockchain ; une demande refusée ne l'est jamais.

À la connexion, le serveur parcourt la blockchain pour reconstituer le contexte de l'utilisateur : cartes créées et possédées, recommandations données et reçues, battles, légitimation. Il l'envoie au client dans la réponse `login_ok`. Si rien ne concerne l'utilisateur, son contexte est vide. L'arrêt du serveur n'est pas une demande client : il est déclenché depuis le serveur.

### 8.2 Notifications

Le serveur envoie des notifications pour garder les écrans à jour : `battle_started` et `battle_finished` à tous les clients, `card_added` et `card_updated` lorsqu'une carte apparaît ou change, `claim_ok` à l'ancien propriétaire d'une carte revendiquée, `users_updated` lorsqu'un utilisateur se connecte, se déconnecte ou est légitimé, et `server_stopping` à l'arrêt du serveur.

### 8.3 Déroulement d'une session

Le client ouvre la socket. Si le serveur est plein, il répond `SERVER_BUSY` et ferme la connexion. Sinon, le client envoie `login`, puis ses demandes. Il se déconnecte avec `disconnect`. Si la socket est coupée brutalement, le serveur libère la connexion ; les actions déjà validées restent dans la blockchain et une action en cours est abandonnée sans corrompre l'état du serveur ni la blockchain.

Le protocole complet, avec les champs de chaque message et des exemples, est décrit dans `protocole-applicatif-commun.md`. Les échanges d'une action sont illustrés par les diagrammes de séquence de l'**Annexe D**.

---

## 9. Cas d'erreur

Lorsque le serveur refuse une demande, il répond par un message de type `error` qui contient le `requestId` de la demande, un `code` d'erreur et un court texte. Par exemple : `{"type":"error","requestId":"r_3","code":"ALREADY_RECOMMENDED","message":"Carte déjà recommandée"}`. La connexion reste ouverte, sauf pour `SERVER_BUSY`. Le client affiche à l'utilisateur son propre message en français, selon le code, et ne modifie pas son état. Pour certaines erreurs qui montrent que l'écran n'est plus à jour (carte introuvable, battle terminée), il redemande le contexte au serveur.

Le projet définit 20 codes d'erreur, regroupés ci-dessous par thème.

| Thème | Codes |
| :--- | :--- |
| Connexion et données | `INVALID_DATA`, `USERNAME_TAKEN`, `SERVER_BUSY`, `INTERNAL_ERROR` |
| Cartes et recommandations | `CARD_NOT_FOUND`, `CARD_NOT_ACTIVE`, `DUPLICATE_CARD`, `MAX_CARDS_REACHED`, `ALREADY_RECOMMENDED`, `NOT_RECOMMENDED` |
| Battles | `BATTLE_NOT_FOUND`, `BATTLE_ALREADY_ACTIVE`, `ALREADY_VOTED`, `CARDS_NOT_COMPATIBLE`, `VA_TOO_LOW` |
| Légitimation et authentification | `NOT_LEGITIMATE`, `THRESHOLD_NOT_REACHED`, `ALREADY_AUTHENTICATED` |
| Droits et intégrité | `NOT_ALLOWED`, `BLOCKCHAIN_CORRUPTED` |

Le client gère aussi des erreurs qui ne viennent pas du serveur : formulaire invalide, serveur injoignable, absence de réponse au bout de 10 secondes, connexion perdue ou message illisible. Le tableau complet des codes, leurs causes et les messages affichés sont dans `cas-erreur.md`.

---

## 10. Conclusion

### 10.1 Synthèse

Cette phase de conception a permis de définir l'architecture de la plateforme : un serveur C multitâche, des clients Java / JavaFX, une blockchain privée avec preuve de travail et une sauvegarde dans PostgreSQL. Le thème des langages de programmation, avec le domaine comme critère de proximité, donne des règles de battle claires. Le protocole applicatif et les codes d'erreur sont définis pour que les trois équipes puissent travailler en parallèle. Plusieurs points restent à valider en commun (lecture du JSON, type des identifiants, portée de la légitimation) ; ils sont listés dans les documents concernés.

### 10.2 Prochaines étapes

La phase 2 (12 octobre au 18 décembre 2026) consiste à coder le serveur, les clients et les tests, à mettre à jour cette spécification et les diagrammes (classes et séquences) au fil du développement, et à produire le test d'altération de la blockchain. La phase 3 (janvier 2027) sera consacrée à la consolidation du code, à la qualité et à l'ajout d'un diagramme de Gantt. La phase 4 (18 au 21 janvier 2027) sera la présentation et la démonstration.

---

## Annexes

Les éléments les plus techniques sont regroupés dans cinq annexes, qui correspondent aux cinq types de diagrammes UML retenus.

* **Annexe A : Parcours utilisateur** (`annexe-a-parcours-utilisateur.md`). Elle présente le déplacement de l'utilisateur entre les écrans JavaFX et les échanges entre l'interface, le modèle du client et le serveur. Elle complète la section 4.
* **Annexe B : Cas d'utilisation** (`annexe-b-cas-utilisation.md`). Elle montre ce que l'utilisateur peut demander à la plateforme et le rôle du serveur. Elle complète les sections 7 et 8.
* **Annexe C : Classes** (`annexe-c-classes.md`). Elle regroupe les diagrammes de classes du client, du serveur et de la blockchain / BDD. Elle complète les sections 3 à 6.
* **Annexe D : Séquences** (`annexe-d-sequences.md`). Elle détaille les échanges entre l'utilisateur, le client et le serveur pour chaque action : connexion, création de carte, recommandation, répudiation, battle, vote, légitimation, authentification, revendication et déconnexion. Elle complète les sections 8 et 9.
* **Annexe E : Déploiement** (`annexe-e-deploiement.md`). Elle montre la couche physique du système : les postes clients, le serveur et la base de données sur `linserv-info-01`. Elle complète les sections 3 et 5.