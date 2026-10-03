# Données du client

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C
> **Partie :** Client Java / JavaFX

---

## 1. Objectif de ce document

Ce document décrit les classes du client et les données qu'elles contiennent. Il complète `architecture-client.md` et le diagramme de classes :

![Diagramme de classes du client](src/diagrammes-client/02-diagramme-classes.png)

Le client ne garde que les données utiles à l'affichage. La référence reste le serveur (et la blockchain). Au moment de la connexion, le serveur envoie au client le contexte de l'utilisateur, et `ClientState` est rempli avec ces informations.

---

## 2. Énumérations

### 2.1 `Domaine`

Valeurs possibles : `Web`, `GameDev`, `DataScience`, `Systems`, `Mobile`.

Le domaine sert de critère de proximité pour les battles.

### 2.2 `StatutCarte`

| Valeur | Signification |
| :--- | :--- |
| `ACTIVE` | Carte normale, peut être recommandée |
| `EN_BATTLE` | Carte engagée dans une battle |
| `AUTHENTIFIEE` | Carte certifiée (badge affiché dans l'interface) |
| `INACTIVE` | Carte archivée, ne peut plus être recommandée ni engagée |

### 2.3 `StatutBattle`

Valeurs possibles : `EN_COURS`, `TERMINEE`.

### 2.4 `ErrorCode`

Les 20 codes d'erreur du projet : `INVALID_DATA`, `USERNAME_TAKEN`, `CARD_NOT_FOUND`, `CARD_NOT_ACTIVE`, `DUPLICATE_CARD`, `ALREADY_RECOMMENDED`, `NOT_RECOMMENDED`, `BATTLE_NOT_FOUND`, `BATTLE_ALREADY_ACTIVE`, `ALREADY_VOTED`, `NOT_LEGITIMATE`, `THRESHOLD_NOT_REACHED`, `CARDS_NOT_COMPATIBLE`, `VA_TOO_LOW`, `ALREADY_AUTHENTICATED`, `SERVER_BUSY`, `INTERNAL_ERROR`, `MAX_CARDS_REACHED`, `NOT_ALLOWED`, `BLOCKCHAIN_CORRUPTED`.

Chaque code est associé à un message en français affiché dans une boîte de dialogue (voir `cas-erreur.md`).

---

## 3. Classes du modèle

### 3.1 `Card`

Représente un langage de programmation.

| Attribut | Type | Description |
| :--- | :--- | :--- |
| `id` | String | Identifiant unique (ex. `c_001`) |
| `nom` | String | Nom du langage (ex. `Python`) |
| `createurHistorique` | String | Concepteur d'origine (ex. `Guido van Rossum`) |
| `anneeCreation` | int | Année de première parution (ex. `1991`) |
| `domaine` | Domaine | Domaine du langage |
| `description` | String | Présentation du langage |
| `proprietaireId` | String | Propriétaire courant (ex. `u_001`) |
| `createurId` | String | Utilisateur qui a créé la carte |
| `valeur` | int | Valeur d'Appréciation (VA) |
| `statut` | StatutCarte | État courant de la carte |
| `estAuthentifiee` | boolean | `true` si la carte est authentifiée |
| `authentifieePar` | String | Utilisateur qui a authentifié la carte |
| `dateAuthentification` | String | Date de l'authentification |
| `winCount` | int | Nombre de battles gagnées |
| `isLegitimate` | boolean | `true` si le propriétaire est légitimé |
| `createdAt` | String | Date de création |

Méthodes utiles pour l'affichage : `estRecommandable()` (statut `ACTIVE` ou `AUTHENTIFIEE`) et `peutEntrerEnBattle()` (statut `ACTIVE` ou `AUTHENTIFIEE` et `valeur >= 5`).

Ces méthodes servent uniquement à activer ou désactiver les boutons. La décision finale appartient au serveur.

### 3.2 `User`

| Attribut | Type | Description |
| :--- | :--- | :--- |
| `id` | String | Identifiant unique (ex. `u_001`) |
| `username` | String | Nom d'utilisateur choisi à la connexion |
| `isLegitimate` | boolean | `true` si l'utilisateur est légitimé |
| `isBot` | boolean | `true` si c'est un client automatique |
| `nbRecommandationsValides` | int | Nombre de recommandations actives données |

### 3.3 `Battle`

| Attribut | Type | Description |
| :--- | :--- | :--- |
| `id` | String | Identifiant de la battle |
| `carteAId` | String | Première carte engagée |
| `carteBId` | String | Deuxième carte engagée |
| `domaine` | Domaine | Domaine commun aux deux cartes |
| `statut` | StatutBattle | `EN_COURS` ou `TERMINEE` |
| `dureeSecondes` | int | Durée du vote (60) |
| `votesA` | int | Votes pour la carte A |
| `votesB` | int | Votes pour la carte B |
| `gagnanteId` | String | Carte gagnante (vide tant que la battle est en cours) |
| `dejaVote` | boolean | `true` si l'utilisateur local a déjà voté |

### 3.4 `ClientState`

Contient l'état courant du client.

| Attribut | Type | Description |
| :--- | :--- | :--- |
| `utilisateurCourant` | User | Utilisateur connecté sur ce client |
| `cartesVisibles` | List&lt;Card&gt; | Cartes de la plateforme |
| `cartesPossedees` | List&lt;Card&gt; | Cartes dont l'utilisateur est propriétaire |
| `cartesCreees` | List&lt;Card&gt; | Cartes créées par l'utilisateur |
| `cartesRecommandees` | List&lt;String&gt; | Identifiants des cartes recommandées par l'utilisateur |
| `recommandationsRecues` | List&lt;String&gt; | Recommandations reçues par ses cartes |
| `utilisateursConnectes` | List&lt;User&gt; | Utilisateurs actuellement connectés |
| `battles` | List&lt;Battle&gt; | Battles en cours ou terminées |

---

## 4. Constantes du client

Ces valeurs reprennent les règles métier. Le client les utilise seulement pour guider l'utilisateur (message, bouton grisé).

| Constante | Valeur | Usage |
| :--- | :--- | :--- |
| `MAX_CARTES_ACTIVES` | 10 | Avertir avant de créer une carte |
| `VA_MIN_BATTLE` | 5 | Griser le bouton « Lancer une battle » |
| `DUREE_BATTLE` | 60 | Compte à rebours affiché |
| `SEUIL_LEGITIMATION` | 3 | Afficher la progression vers la légitimation |

---

## 5. Classes réseau et services

### 5.1 `ServerConnection`

| Attribut | Type | Description |
| :--- | :--- | :--- |
| `host` | String | Adresse IP du serveur |
| `port` | int | Port du serveur |
| `socket` | Socket | Socket TCP |
| `out` | PrintWriter | Flux d'écriture (UTF-8) |
| `in` | BufferedReader | Flux de lecture (UTF-8) |

Méthodes : `connect()`, `send(String json)`, `close()`, `isConnected()`.

### 5.2 `MessageReader`

Thread qui lit les messages du serveur ligne par ligne (délimiteur `\n`).

| Attribut | Type | Description |
| :--- | :--- | :--- |
| `connection` | ServerConnection | Connexion utilisée pour lire |
| `handler` | ResponseHandler | Reçoit chaque message lu |
| `running` | boolean | Indique si le thread doit continuer |

Méthodes : `run()`, `stop()`.

### 5.3 `RequestService`

Construit les demandes JSON et les envoie. Méthodes principales :

* `login(username)`
* `createCard(nom, createurHistorique, anneeCreation, domaine, description)`
* `recommend(cardId)`
* `repudiate(cardId)`
* `launchBattle(cardAId, cardBId)`
* `vote(battleId, cardId)`
* `requestLegitimation()`
* `claimCard(cardId)`
* `authenticateCard(cardId)`
* `disconnect()`

### 5.4 `ResponseHandler`

| Attribut | Type | Description |
| :--- | :--- | :--- |
| `state` | ClientState | État à mettre à jour |
| `listeners` | List&lt;ViewListener&gt; | Contrôleurs à prévenir |

Méthodes : `onMessage(String json)`, `handleResponse(...)`, `handleNotification(...)`, `handleError(ErrorCode)`.

La mise à jour de l'écran passe par `Platform.runLater(...)`.

---

## 6. Contrôleurs JavaFX

| Contrôleur | Principaux éléments |
| :--- | :--- |
| `LoginController` | champs IP, port, nom d'utilisateur ; bouton « Se connecter » |
| `MainController` | menu de navigation, liste des utilisateurs connectés |
| `CardsController` | tableau des cartes, boutons « Recommander » et « Répudier » |
| `CreateCardController` | champs nom, créateur historique, année, domaine, description |
| `BattlesController` | liste des battles, boutons de vote, compte à rebours |
| `ProfileController` | cartes de l'utilisateur, boutons légitimation, revendication, authentification |

Chaque contrôleur utilise `RequestService` pour envoyer une demande et lit `ClientState` pour s'afficher.

---

## 7. Vérifications faites par le client avant l'envoi

Pour éviter des échanges inutiles, le formulaire de création vérifie :

* `nom` : non vide, 50 caractères maximum ;
* `createurHistorique` : non vide, 100 caractères maximum ;
* `anneeCreation` : nombre entier ;
* `domaine` : une des 5 valeurs de la liste ;
* `description` : 500 caractères maximum.

Le serveur refait toutes ces vérifications et reste le seul à décider (`INVALID_DATA` en cas d'erreur).

---

## 8. Fichiers liés

* `architecture-client.md` : organisation du client
* `regles-metier.md` : règles de la plateforme
* `protocole-applicatif-commun.md` : format des messages
* `annexe-c-classes.md` : diagramme de classes