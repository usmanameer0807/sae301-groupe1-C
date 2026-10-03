# Cas d'erreur

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C
> **Statut :** Phase 1, à valider par les trois équipes

---

## 1. Principes

* Le **serveur est le seul à décider** : il valide ou refuse chaque demande.
* Une demande refusée **n'est jamais inscrite dans la blockchain** et ne modifie aucune donnée.
* Une erreur est envoyée dans un message `error` contenant le `requestId` de la demande, un `code` (un des 20 codes ci-dessous) et un `message` court.
* La connexion reste ouverte après une erreur, sauf pour `SERVER_BUSY` (le serveur ferme la socket).
* Le client affiche son **propre message en français** selon le `code`. Le champ `message` du serveur sert au débogage.
* Le client évite d'envoyer des demandes qu'il sait invalides (bouton grisé, vérification du formulaire), mais le serveur refait toutes les vérifications.

Format d'un message d'erreur (voir `protocole-applicatif-commun.md`) :

```json
{"type":"error","requestId":"r_3","code":"ALREADY_RECOMMENDED","message":"Carte déjà recommandée"}
```

---

## 2. Les 20 codes d'erreur

### 2.1 Connexion et données

| Code | Cause | Demandes concernées | Message affiché par le client |
| :--- | :--- | :--- | :--- |
| `INVALID_DATA` | Message mal formé, champ manquant ou hors limites, type inconnu, légitimation déjà accordée, même carte choisie deux fois, `cardId` non engagée dans la battle | Toutes | « Les données envoyées sont invalides. » |
| `USERNAME_TAKEN` | Nom d'utilisateur déjà connecté ou déjà utilisé | `login` | « Ce nom d'utilisateur est déjà pris. » |
| `SERVER_BUSY` | Serveur plein (`MAX_CLIENTS = 20`) | `login` (connexion) | « Le serveur est occupé, réessayez plus tard. » |
| `INTERNAL_ERROR` | Erreur du serveur (base de données indisponible, ressource manquante) | Toutes | « Erreur du serveur, réessayez plus tard. » |

### 2.2 Cartes et recommandations

| Code | Cause | Demandes concernées | Message affiché par le client |
| :--- | :--- | :--- | :--- |
| `CARD_NOT_FOUND` | Identifiant de carte inconnu | `recommend`, `repudiate`, `launch_battle`, `claim_card`, `authenticate_card` | « Cette carte n'existe pas. » |
| `CARD_NOT_ACTIVE` | Carte `INACTIVE` ou `EN_BATTLE` selon l'action | `recommend`, `launch_battle`, `authenticate_card` | « Cette carte n'est pas disponible pour cette action. » |
| `DUPLICATE_CARD` | Une carte avec ce nom existe déjà | `create_card` | « Une carte avec ce nom existe déjà. » |
| `MAX_CARDS_REACHED` | L'utilisateur possède déjà 10 cartes actives | `create_card` | « Vous avez atteint la limite de 10 cartes actives. » |
| `ALREADY_RECOMMENDED` | Recommandation active déjà donnée sur cette carte | `recommend` | « Vous avez déjà recommandé cette carte. » |
| `NOT_RECOMMENDED` | Aucune recommandation active de l'utilisateur sur cette carte | `repudiate` | « Vous n'avez pas recommandé cette carte. » |

### 2.3 Battles

| Code | Cause | Demandes concernées | Message affiché par le client |
| :--- | :--- | :--- | :--- |
| `BATTLE_NOT_FOUND` | Battle inconnue ou déjà terminée | `vote` | « Cette battle n'existe pas ou est terminée. » |
| `BATTLE_ALREADY_ACTIVE` | Une des deux cartes est déjà engagée dans une battle | `launch_battle` | « Une de ces cartes est déjà en battle. » |
| `ALREADY_VOTED` | L'utilisateur a déjà voté dans cette battle | `vote` | « Vous avez déjà voté pour cette battle. » |
| `CARDS_NOT_COMPATIBLE` | Cartes de domaines différents | `launch_battle` | « Les deux cartes doivent avoir le même domaine. » |
| `VA_TOO_LOW` | Une des deux cartes a une VA inférieure à 5 | `launch_battle` | « Chaque carte doit avoir une VA d'au moins 5. » |

### 2.4 Légitimation, revendication, authentification

| Code | Cause | Demandes concernées | Message affiché par le client |
| :--- | :--- | :--- | :--- |
| `NOT_LEGITIMATE` | L'utilisateur n'est pas légitimé | `claim_card`, `authenticate_card` | « Vous devez d'abord être légitimé. » |
| `THRESHOLD_NOT_REACHED` | Moins de 3 recommandations valides | `request_legitimation` | « Il faut au moins 3 recommandations valides. » |
| `ALREADY_AUTHENTICATED` | Carte déjà authentifiée | `authenticate_card` | « Cette carte est déjà authentifiée. » |

### 2.5 Droits et intégrité

| Code | Cause | Demandes concernées | Message affiché par le client |
| :--- | :--- | :--- | :--- |
| `NOT_ALLOWED` | Action interdite à cet utilisateur : pas propriétaire de la carte, propriétaire d'une carte de la battle qui veut voter, demande avant `login`, propriétaire de la carte déjà légitimé (revendication) | `launch_battle`, `vote`, `claim_card`, `authenticate_card`, toute demande avant `login` | « Vous n'avez pas le droit de faire cette action. » |
| `BLOCKCHAIN_CORRUPTED` | La vérification a trouvé un bloc modifié (le message contient `blockId`) | `verify_blockchain` | « La blockchain est corrompue (bloc n° X). » |

---

## 3. Erreurs possibles par demande

| Demande | Erreurs possibles |
| :--- | :--- |
| `login` | `USERNAME_TAKEN`, `INVALID_DATA`, `SERVER_BUSY` |
| `create_card` | `INVALID_DATA`, `DUPLICATE_CARD`, `MAX_CARDS_REACHED` |
| `recommend` | `CARD_NOT_FOUND`, `CARD_NOT_ACTIVE`, `ALREADY_RECOMMENDED` |
| `repudiate` | `CARD_NOT_FOUND`, `NOT_RECOMMENDED` |
| `launch_battle` | `CARD_NOT_FOUND`, `CARD_NOT_ACTIVE`, `NOT_ALLOWED`, `CARDS_NOT_COMPATIBLE`, `VA_TOO_LOW`, `BATTLE_ALREADY_ACTIVE`, `INVALID_DATA` |
| `vote` | `BATTLE_NOT_FOUND`, `ALREADY_VOTED`, `NOT_ALLOWED`, `INVALID_DATA` |
| `request_legitimation` | `THRESHOLD_NOT_REACHED`, `INVALID_DATA` |
| `claim_card` | `CARD_NOT_FOUND`, `NOT_LEGITIMATE`, `NOT_ALLOWED` |
| `authenticate_card` | `CARD_NOT_FOUND`, `NOT_LEGITIMATE`, `NOT_ALLOWED`, `ALREADY_AUTHENTICATED`, `CARD_NOT_ACTIVE` |
| `get_context` | `INTERNAL_ERROR` |
| `verify_blockchain` | `BLOCKCHAIN_CORRUPTED`, `INTERNAL_ERROR` |
| `disconnect` | aucune |

`INTERNAL_ERROR` et `INVALID_DATA` peuvent aussi répondre à n'importe quelle demande.

---

## 4. Choix pour les cas non couverts par les règles métier

Ces choix complètent `regles-metier.md` et sont à valider par le groupe.

| Situation | Code retenu |
| :--- | :--- |
| Authentifier une carte `EN_BATTLE` ou `INACTIVE` | `CARD_NOT_ACTIVE` |
| Revendiquer une carte dont le propriétaire est déjà légitimé | `NOT_ALLOWED` |
| Revendiquer une carte qu'on possède déjà | `NOT_ALLOWED` |
| Demander la légitimation alors qu'on est déjà légitimé | `INVALID_DATA` |
| Recommander une carte `EN_BATTLE` | accepté (statut `ACTIVE` ou `AUTHENTIFIEE` seulement : `CARD_NOT_ACTIVE`) |
| Voter pour une carte qui n'est pas dans la battle | `INVALID_DATA` |
| Lancer une battle avec deux fois la même carte | `INVALID_DATA` |
| Toute demande avant `login` | `NOT_ALLOWED` |

---

## 5. Erreurs propres au client (hors codes du serveur)

Ces erreurs ne viennent pas d'un message `error` : le client les détecte seul.

| Situation | Détection | Traitement |
| :--- | :--- | :--- |
| Champ de connexion vide, port non numérique | Vérification du formulaire | Message sous le champ, rien n'est envoyé |
| Formulaire de carte invalide (longueurs, année, domaine) | Vérification du formulaire | Message sous le champ, rien n'est envoyé |
| Serveur injoignable (adresse ou port incorrect, serveur arrêté) | Exception à l'ouverture de la socket | Boîte de dialogue « Impossible de se connecter au serveur », on reste sur l'écran de connexion |
| Pas de réponse en 10 secondes | Délai écoulé sur une demande | Demande considérée comme non réalisée, message « Le serveur ne répond pas » |
| Connexion perdue en cours de session | Fin de flux dans `MessageReader` | Message « Connexion au serveur perdue », vidage de l'état, retour à l'écran de connexion |
| Arrêt du serveur | Notification `server_stopping` | Message « Le serveur s'arrête », retour à l'écran de connexion |
| Message reçu illisible (JSON invalide) | Erreur d'analyse | Message ignoré et noté dans le journal du client, la connexion reste ouverte |
| Code d'erreur inconnu | Code absent de `ErrorCode` | Message générique « Erreur inattendue », avec le texte du champ `message` |
| Réponse sans demande correspondante (`requestId` inconnu) | `requestId` absent de la liste des demandes en attente | Réponse ignorée |

Une demande en cours au moment d'une coupure est considérée comme **non réalisée** (règle métier de déconnexion). Après reconnexion, le serveur reconstitue le contexte depuis la blockchain et le client affiche l'état réel.

---

## 6. Traitement côté client

### 6.1 Affichage

| Type d'erreur | Affichage |
| :--- | :--- |
| Erreur de saisie (formulaire) | Message rouge sous le champ concerné |
| Code d'erreur du serveur | Boîte de dialogue avec le message en français |
| Erreur réseau | Boîte de dialogue, puis retour à l'écran de connexion si la connexion est perdue |

L'affichage passe par `Platform.runLater(...)` : le thread de lecture ne touche jamais directement l'interface.

### 6.2 Resynchronisation

Certaines erreurs montrent que l'écran du client n'est plus à jour (une autre personne a modifié la carte entre-temps). Après la boîte de dialogue, le client envoie `get_context` pour rafraîchir l'état dans ces cas :

* `CARD_NOT_FOUND`
* `CARD_NOT_ACTIVE`
* `BATTLE_NOT_FOUND`
* `BATTLE_ALREADY_ACTIVE`
* `ALREADY_RECOMMENDED`
* `NOT_RECOMMENDED`
* `ALREADY_AUTHENTICATED`

### 6.3 Ce que le client ne fait jamais

* Il ne modifie pas `ClientState` après une erreur (sauf par un nouveau contexte).
* Il ne renvoie pas automatiquement la demande refusée.
* Il n'affiche pas de données techniques (identifiants de bloc, trace d'erreur) à l'utilisateur, sauf le numéro du bloc pour `BLOCKCHAIN_CORRUPTED`.

---

## 7. Traitement côté serveur

Rappel de ce que le serveur garantit (détail dans `architecture-serveur.md`) :

* une demande invalide est refusée avant tout calcul de bloc : pas de minage inutile ;
* une demande refusée ne laisse aucune trace dans la blockchain ni dans la base de données ;
* si la base de données est indisponible, le serveur répond `INTERNAL_ERROR` et n'enregistre pas l'action ;
* une coupure de socket pendant une action abandonne l'action sans corrompre la mémoire du serveur ni la blockchain ;
* `BLOCKCHAIN_CORRUPTED` est renvoyé uniquement par `verify_blockchain` (altération détectée sur le hash, le hash précédent ou la preuve de travail).

---

## 8. Tests prévus (client)

Les tests JUnit du client vérifient notamment :

* chaque code d'erreur du serveur est reconnu et associé à un message ;
* un code inconnu donne le message générique ;
* une erreur ne modifie pas `ClientState` ;
* les erreurs de resynchronisation déclenchent bien `get_context` ;
* un message JSON invalide est ignoré sans arrêter `MessageReader` ;
* le délai de 10 secondes est détecté.

---



## 9. Fichiers liés

* `protocole-applicatif-commun.md` : format des messages et des erreurs
* `regles-metier.md` : règles à l'origine des refus
* `architecture-client.md` : traitement des messages côté client
* `donnees-client.md` : énumération `ErrorCode`