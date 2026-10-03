# Protocole applicatif commun

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C
> **Statut :** Protocole de la phase 1, à valider par les trois équipes

---

## 1. Principes

Toute la communication entre les clients et le serveur passe par des **sockets TCP**. Aucune autre interface n'est utilisée (pas d'API REST). Les clients n'accèdent jamais à la base de données.

| Élément | Choix |
| :--- | :--- |
| Transport | Socket TCP |
| Format | JSON |
| Encodage | UTF-8 |
| Délimiteur | un message par ligne, terminé par `\n` |
| Connexion | une socket par client, ouverte pendant toute la session |
| Taille maximale d'un message | 8192 octets |

### 1.1 Choix possibles et justification

| Question | Options | Choix retenu | Raison |
| :--- | :--- | :--- | :--- |
| Format des messages | Texte séparé par des `;`, JSON, binaire | JSON | Lisible, facile à déboguer, évolutif |
| Découpage des messages | Longueur fixe, longueur préfixée, délimiteur | Délimiteur `\n` | Simple à lire côté Java (`readLine`) et côté C |
| Connexion | Une connexion par demande, une connexion permanente | Permanente | Le serveur doit pouvoir prévenir les clients (battles, résultats) |
| Association réponse / demande | Aucune, ordre d'arrivée, identifiant de demande | Champ `requestId` | Des notifications peuvent arriver entre une demande et sa réponse |

Un message JSON tient sur **une seule ligne** : il ne contient aucun retour à la ligne (dans une description, le retour à la ligne s'écrit `\n` échappé).

---

## 2. Types de messages

| Sorte | Sens | Champ `requestId` | Exemple |
| :--- | :--- | :---: | :--- |
| **Demande** | Client vers serveur | oui | `recommend` |
| **Réponse** | Serveur vers client, suite à une demande | oui (recopié) | `recommend_ok`, `error` |
| **Notification** | Serveur vers client, sans demande | non | `battle_started` |

Chaque message contient un champ `type`.

* `requestId` est une chaîne choisie par le client (ex. `"r_12"`). Le serveur la recopie dans la réponse correspondante.
* Un client ne réutilise pas un `requestId` tant que la réponse n'est pas arrivée.

---

## 3. Conventions

| Sujet | Convention |
| :--- | :--- |
| Identifiants | Chaînes : `u_001` (utilisateur), `c_001` (carte), `b_001` (battle). Générés par le serveur. |
| Dates | Chaîne ISO 8601 en UTC : `"2026-10-05T14:30:00Z"` |
| Booléens | `true` / `false` |
| Valeur absente | `null` |
| Domaine | `Web`, `GameDev`, `DataScience`, `Systems`, `Mobile` |
| Statut de carte | `ACTIVE`, `EN_BATTLE`, `AUTHENTIFIEE`, `INACTIVE` |
| Statut de battle | `EN_COURS`, `TERMINEE` |
| Noms des champs | Les mêmes que dans `regles-metier.md` et `donnees-client.md` |

---

## 4. Objets échangés

### 4.1 Utilisateur (`user`)

```json
{"id":"u_001","username":"alice","isLegitimate":false,"isBot":false,"nbRecommandationsValides":0}
```

### 4.2 Carte (`card`)

```json
{"id":"c_001","nom":"Python","createurHistorique":"Guido van Rossum","anneeCreation":1991,"domaine":"DataScience","description":"Langage généraliste lisible.","proprietaireId":"u_001","createurId":"u_001","valeur":0,"statut":"ACTIVE","estAuthentifiee":false,"authentifieePar":null,"dateAuthentification":null,"winCount":0,"isLegitimate":false,"createdAt":"2026-10-05T14:30:00Z"}
```

### 4.3 Battle (`battle`)

```json
{"id":"b_001","carteAId":"c_001","carteBId":"c_002","domaine":"DataScience","statut":"EN_COURS","dureeSecondes":60,"votesA":0,"votesB":0,"gagnanteId":null}
```

* Pendant la battle, `votesA` et `votesB` valent 0 : les votes ne sont révélés qu'à la fin.
* Dans le contexte envoyé à un client, la battle contient en plus `dejaVote` (booléen propre à cet utilisateur).

### 4.4 Contexte (`context`)

Envoyé dans `login_ok` et `context_ok`. Il est reconstitué par le serveur à partir de la blockchain.

```json
{"cards":[],"recommended":["c_001"],"receivedRecommendations":[{"cardId":"c_001","userId":"u_002"}],"battles":[],"users":[]}
```

| Champ | Contenu |
| :--- | :--- |
| `cards` | Toutes les cartes visibles sur la plateforme (objets `card`) |
| `recommended` | Identifiants des cartes recommandées par l'utilisateur (recommandations actives) |
| `receivedRecommendations` | Recommandations actives reçues par les cartes de l'utilisateur |
| `battles` | Battles en cours ou terminées (objets `battle`) |
| `users` | Utilisateurs actuellement connectés (objets `user`) |

Le client déduit lui-même « cartes possédées » (`proprietaireId`) et « cartes créées » (`createurId`) à partir de `cards`. Si aucune donnée ne concerne l'utilisateur, ses listes sont vides.

---

## 5. Session

### 5.1 États

| État | Description | Messages acceptés |
| :--- | :--- | :--- |
| Connecté (socket ouverte) | Le client n'a pas encore fait `login` | `login` uniquement |
| Identifié | Après `login_ok` | toutes les demandes |
| Fermé | Après `disconnect` ou coupure | aucun |

Toute autre demande avant `login` reçoit `error` avec `NOT_ALLOWED`.

### 5.2 Déroulement

1. Le client ouvre la socket avec l'adresse IP et le port du serveur.
2. Si le serveur a déjà 20 clients (`MAX_CLIENTS`), il envoie `error` avec `SERVER_BUSY`, puis ferme la socket.
3. Le client envoie `login`. Le serveur répond `login_ok` (avec le contexte) ou `error` (`USERNAME_TAKEN`, `INVALID_DATA`). Le serveur envoie ensuite `users_updated` aux autres clients.
4. Le client envoie des demandes. Le serveur répond à chacune. Des notifications peuvent arriver à tout moment.
5. Le client envoie `disconnect`. Le serveur répond `disconnect_ok`, libère la socket et envoie `users_updated` aux autres clients.

### 5.3 Coupure inattendue

* Si la socket est coupée sans `disconnect`, le serveur libère la socket et traite le cas comme une déconnexion.
* Les actions déjà validées restent dans la blockchain. Une action en cours d'envoi est abandonnée, sans modifier la blockchain.
* Si l'utilisateur est en pleine battle (propriétaire d'une carte engagée), la battle continue jusqu'à la fin des 60 secondes.
* Côté client : si le serveur ne répond pas à une demande en **10 secondes**, le client considère la demande comme non réalisée et affiche un message.

---

## 6. Demandes et réponses

Dans les exemples, `requestId` est présent dans la demande et dans la réponse.

### 6.1 `login`

| Champ | Type | Obligatoire | Description |
| :--- | :--- | :---: | :--- |
| `username` | chaîne | oui | 3 à 20 caractères : lettres, chiffres, `_` |
| `isBot` | booléen | non | `true` pour un client automatique (défaut : `false`) |

Réponse : `login_ok` avec `user` et `context`.
Erreurs : `USERNAME_TAKEN`, `INVALID_DATA`, `SERVER_BUSY`.

```json
{"type":"login","requestId":"r_1","username":"alice"}
{"type":"login_ok","requestId":"r_1","user":{"id":"u_001","username":"alice","isLegitimate":false,"isBot":false,"nbRecommandationsValides":0},"context":{"cards":[],"recommended":[],"receivedRecommendations":[],"battles":[],"users":[]}}
```

### 6.2 `create_card`

| Champ | Type | Contrainte |
| :--- | :--- | :--- |
| `data.nom` | chaîne | non vide, 50 caractères maximum |
| `data.createurHistorique` | chaîne | non vide, 100 caractères maximum |
| `data.anneeCreation` | entier | entier valide |
| `data.domaine` | chaîne | une des 5 valeurs |
| `data.description` | chaîne | 500 caractères maximum |

Vérifications du serveur : données valides, carte pas déjà existante (même `nom`), moins de 10 cartes actives pour l'utilisateur.
Réponse : `card_created` avec `card` (`proprietaireId = createurId`, `valeur = 0`, `statut = ACTIVE`).
Erreurs : `INVALID_DATA`, `DUPLICATE_CARD`, `MAX_CARDS_REACHED`.
Effet : un bloc est ajouté à la blockchain ; `card_added` est envoyé aux autres clients.

```json
{"type":"create_card","requestId":"r_2","data":{"nom":"Python","createurHistorique":"Guido van Rossum","anneeCreation":1991,"domaine":"DataScience","description":"Langage généraliste lisible."}}
{"type":"card_created","requestId":"r_2","card":{"id":"c_001","nom":"Python","createurHistorique":"Guido van Rossum","anneeCreation":1991,"domaine":"DataScience","description":"Langage généraliste lisible.","proprietaireId":"u_001","createurId":"u_001","valeur":0,"statut":"ACTIVE","estAuthentifiee":false,"authentifieePar":null,"dateAuthentification":null,"winCount":0,"isLegitimate":false,"createdAt":"2026-10-05T14:30:00Z"}}
```

### 6.3 `recommend`

| Champ | Type | Description |
| :--- | :--- | :--- |
| `cardId` | chaîne | Carte à recommander |

Vérifications : carte existante, statut `ACTIVE` ou `AUTHENTIFIEE`, pas déjà recommandée par cet utilisateur.
Réponse : `recommend_ok` avec `cardId` et `valeur` (nouvelle VA).
Erreurs : `CARD_NOT_FOUND`, `CARD_NOT_ACTIVE`, `ALREADY_RECOMMENDED`.
Effet : un bloc est ajouté ; `card_updated` est envoyé aux autres clients.

```json
{"type":"recommend","requestId":"r_3","cardId":"c_001"}
{"type":"recommend_ok","requestId":"r_3","cardId":"c_001","valeur":1}
```

### 6.4 `repudiate`

| Champ | Type | Description |
| :--- | :--- | :--- |
| `cardId` | chaîne | Carte dont on retire la recommandation |

Vérifications : carte existante, recommandation active de cet utilisateur sur cette carte.
Réponse : `repudiate_ok` avec `cardId` et `valeur`.
Erreurs : `CARD_NOT_FOUND`, `NOT_RECOMMENDED`.
Effet : un bloc est ajouté, VA − 1, l'historique reste dans la blockchain ; `card_updated` est envoyé aux autres clients.

```json
{"type":"repudiate","requestId":"r_4","cardId":"c_001"}
{"type":"repudiate_ok","requestId":"r_4","cardId":"c_001","valeur":0}
```

### 6.5 `launch_battle`

| Champ | Type | Description |
| :--- | :--- | :--- |
| `cardAId` | chaîne | Première carte |
| `cardBId` | chaîne | Deuxième carte |

Vérifications : cartes existantes, statut `ACTIVE` ou `AUTHENTIFIEE`, demandeur propriétaire d'au moins une des deux cartes, même domaine, VA ≥ 5 pour les deux, aucune des deux déjà en battle.
Réponse : la notification `battle_started`, envoyée à **tous** les clients (y compris le demandeur). Le message `battle_started` du demandeur contient le `requestId` de sa demande.
Erreurs : `CARD_NOT_FOUND`, `CARD_NOT_ACTIVE`, `NOT_ALLOWED`, `CARDS_NOT_COMPATIBLE`, `VA_TOO_LOW`, `BATTLE_ALREADY_ACTIVE`, `INVALID_DATA` (même carte deux fois).
Effet : un bloc est ajouté, les deux cartes passent à `EN_BATTLE`, le chrono de 60 secondes démarre.

```json
{"type":"launch_battle","requestId":"r_5","cardAId":"c_001","cardBId":"c_002"}
{"type":"battle_started","requestId":"r_5","battle":{"id":"b_001","carteAId":"c_001","carteBId":"c_002","domaine":"DataScience","statut":"EN_COURS","dureeSecondes":60,"votesA":0,"votesB":0,"gagnanteId":null},"startedAt":"2026-10-05T14:35:00Z"}
```

### 6.6 `vote`

| Champ | Type | Description |
| :--- | :--- | :--- |
| `battleId` | chaîne | Battle concernée |
| `cardId` | chaîne | Carte soutenue (une des deux cartes engagées) |

Vérifications : battle existante et en cours, utilisateur non propriétaire d'une des deux cartes, un seul vote par utilisateur, `cardId` engagée dans la battle.
Réponse : `vote_ok` avec `battleId`.
Erreurs : `BATTLE_NOT_FOUND`, `ALREADY_VOTED`, `NOT_ALLOWED`, `INVALID_DATA`.
Effet : le vote est compté (1 utilisateur = 1 vote).

```json
{"type":"vote","requestId":"r_6","battleId":"b_001","cardId":"c_001"}
{"type":"vote_ok","requestId":"r_6","battleId":"b_001"}
```

### 6.7 `request_legitimation`

Aucun champ. La légitimation est **globale** : un seul `isLegitimate` par utilisateur.

Vérifications : au moins 3 recommandations valides, utilisateur pas déjà légitimé.
Réponse : `legitimation_ok` avec `user`.
Erreurs : `THRESHOLD_NOT_REACHED`, `INVALID_DATA` (déjà légitimé).
Effet : un bloc est ajouté, `isLegitimate = true` ; `users_updated` est envoyé aux autres clients.

```json
{"type":"request_legitimation","requestId":"r_7"}
{"type":"legitimation_ok","requestId":"r_7","user":{"id":"u_001","username":"alice","isLegitimate":true,"isBot":false,"nbRecommandationsValides":3}}
```

### 6.8 `claim_card`

| Champ | Type | Description |
| :--- | :--- | :--- |
| `cardId` | chaîne | Carte à revendiquer |

Vérifications : carte existante, demandeur légitimé, propriétaire actuel non légitimé, carte pas déjà à lui.
Réponse : `claim_ok` avec `card` (nouveau `proprietaireId`). Le transfert est direct, sans acceptation de l'ancien propriétaire.
Erreurs : `CARD_NOT_FOUND`, `NOT_LEGITIMATE`, `NOT_ALLOWED`.
Effet : un bloc `ACTION_CLAIM_CARD` est ajouté, `proprietaireId` change, `createurId` ne change pas. L'ancien propriétaire (s'il est connecté) reçoit `claim_ok` sans `requestId`.

```json
{"type":"claim_card","requestId":"r_8","cardId":"c_003"}
{"type":"claim_ok","requestId":"r_8","card":{"id":"c_003","nom":"Rust","domaine":"Systems","proprietaireId":"u_001","createurId":"u_004","valeur":6,"statut":"ACTIVE"}}
```

(Les exemples abrègent l'objet `card` ; le message réel contient tous les champs du § 4.2.)

### 6.9 `authenticate_card`

| Champ | Type | Description |
| :--- | :--- | :--- |
| `cardId` | chaîne | Carte à authentifier |

Vérifications : carte existante, utilisateur légitimé, utilisateur propriétaire de la carte, carte pas déjà authentifiée, carte `ACTIVE` (pas `EN_BATTLE` ni `INACTIVE`).
Réponse : `authenticate_ok` avec `card` (statut `AUTHENTIFIEE`, +10 VA, `authentifieePar`, `dateAuthentification`).
Erreurs : `CARD_NOT_FOUND`, `NOT_LEGITIMATE`, `NOT_ALLOWED`, `ALREADY_AUTHENTICATED`, `CARD_NOT_ACTIVE`.
Effet : un bloc est ajouté, `nom`, `domaine`, `createurHistorique` et `anneeCreation` sont verrouillés ; `card_updated` est envoyé aux autres clients.

```json
{"type":"authenticate_card","requestId":"r_9","cardId":"c_001"}
{"type":"authenticate_ok","requestId":"r_9","card":{"id":"c_001","nom":"Python","domaine":"DataScience","valeur":11,"statut":"AUTHENTIFIEE","estAuthentifiee":true,"authentifieePar":"u_001","dateAuthentification":"2026-10-05T15:00:00Z"}}
```

### 6.10 `get_context`

Aucun champ. Le client redemande son contexte complet (utile après une erreur d'affichage).
Réponse : `context_ok` avec `context`.
Erreurs : `INTERNAL_ERROR`.

```json
{"type":"get_context","requestId":"r_10"}
{"type":"context_ok","requestId":"r_10","context":{"cards":[],"recommended":[],"receivedRecommendations":[],"battles":[],"users":[]}}
```

### 6.11 `verify_blockchain`

Aucun champ. Le serveur contrôle les hash, les hash précédents et la preuve de travail de chaque bloc.
Réponse : `verify_ok` avec `valid` (booléen) et `blocks` (nombre de blocs vérifiés).
Erreurs : `BLOCKCHAIN_CORRUPTED` (avec `blockId` du premier bloc incohérent), `INTERNAL_ERROR`.

```json
{"type":"verify_blockchain","requestId":"r_11"}
{"type":"verify_ok","requestId":"r_11","valid":true,"blocks":42}
{"type":"error","requestId":"r_11","code":"BLOCKCHAIN_CORRUPTED","message":"Bloc 17 modifié","blockId":17}
```

### 6.12 `disconnect`

Aucun champ.
Réponse : `disconnect_ok`. Le serveur libère ensuite la socket.

```json
{"type":"disconnect","requestId":"r_12"}
{"type":"disconnect_ok","requestId":"r_12"}
```

### 6.13 Arrêt du serveur

L'arrêt propre du serveur n'est **pas** une demande client : il est déclenché depuis le serveur (commande ou signal). Le serveur envoie alors la notification `server_stopping`, termine les traitements en cours, sauvegarde la blockchain en base, ferme les sockets et libère ses ressources.

---

## 7. Réponse d'erreur

```json
{"type":"error","requestId":"r_3","code":"ALREADY_RECOMMENDED","message":"Carte déjà recommandée"}
```

| Champ | Description |
| :--- | :--- |
| `code` | Un des 20 codes ci-dessous |
| `message` | Texte court pour le débogage. Le client affiche son propre message en français selon le `code`. |

Une demande refusée n'est **jamais** inscrite dans la blockchain.

### 7.1 Les 20 codes d'erreur

| Code | Cas d'utilisation |
| :--- | :--- |
| `INVALID_DATA` | Message mal formé, champ manquant ou hors limites, type inconnu, légitimation déjà accordée |
| `USERNAME_TAKEN` | Nom d'utilisateur déjà connecté ou déjà utilisé |
| `CARD_NOT_FOUND` | Identifiant de carte inconnu |
| `CARD_NOT_ACTIVE` | Carte `INACTIVE` ou `EN_BATTLE` (selon l'action) |
| `DUPLICATE_CARD` | Une carte avec ce nom existe déjà |
| `ALREADY_RECOMMENDED` | L'utilisateur a déjà une recommandation active sur cette carte |
| `NOT_RECOMMENDED` | Aucune recommandation active de l'utilisateur à répudier |
| `BATTLE_NOT_FOUND` | Battle inconnue ou déjà terminée |
| `BATTLE_ALREADY_ACTIVE` | Une des cartes est déjà engagée dans une battle |
| `ALREADY_VOTED` | L'utilisateur a déjà voté dans cette battle |
| `NOT_LEGITIMATE` | L'utilisateur n'est pas légitimé |
| `THRESHOLD_NOT_REACHED` | Moins de 3 recommandations valides |
| `CARDS_NOT_COMPATIBLE` | Cartes de domaines différents |
| `VA_TOO_LOW` | Une des cartes a une VA inférieure à 5 |
| `ALREADY_AUTHENTICATED` | Carte déjà authentifiée |
| `SERVER_BUSY` | Serveur plein (`MAX_CLIENTS = 20`) |
| `INTERNAL_ERROR` | Erreur du serveur (base de données indisponible, etc.) |
| `MAX_CARDS_REACHED` | L'utilisateur possède déjà 10 cartes actives |
| `NOT_ALLOWED` | Action interdite à cet utilisateur (pas propriétaire, propriétaire d'une carte de la battle, pas connecté, propriétaire déjà légitimé…) |
| `BLOCKCHAIN_CORRUPTED` | La vérification a détecté une altération |

Le détail du traitement côté client est dans `cas-erreur.md`.

---

## 8. Notifications

Les notifications n'ont pas de `requestId`. Elles sont envoyées uniquement aux clients identifiés.

| Type | Destinataires | Champs | Quand |
| :--- | :--- | :--- | :--- |
| `battle_started` | Tous | `battle`, `startedAt` | Une battle est acceptée |
| `battle_finished` | Tous | `battle`, `cards` | Les 60 secondes sont écoulées |
| `card_added` | Tous sauf le créateur | `card` | Une carte est créée |
| `card_updated` | Tous sauf l'auteur de l'action | `card` | VA, statut ou propriétaire modifié |
| `claim_ok` | Ancien propriétaire | `card` | Sa carte a été revendiquée |
| `users_updated` | Tous | `users` | Connexion, déconnexion ou légitimation d'un utilisateur |
| `server_stopping` | Tous | aucun | Arrêt du serveur |

### 8.1 `battle_finished`

Le serveur compte les votes. En cas d'égalité : VA initiale la plus élevée, puis `id` le plus petit. La gagnante récupère les recommandations actives de la perdante, qui passe à `INACTIVE`. `cards` contient les deux cartes mises à jour.

```json
{"type":"battle_finished","battle":{"id":"b_001","carteAId":"c_001","carteBId":"c_002","domaine":"DataScience","statut":"TERMINEE","dureeSecondes":60,"votesA":3,"votesB":1,"gagnanteId":"c_001"},"cards":[{"id":"c_001","valeur":14,"statut":"ACTIVE","winCount":1},{"id":"c_002","valeur":0,"statut":"INACTIVE","winCount":0}]}
```

(Les cartes sont abrégées ici ; le message réel contient tous les champs du § 4.2.)

### 8.2 `users_updated`

```json
{"type":"users_updated","users":[{"id":"u_001","username":"alice","isLegitimate":false,"isBot":false,"nbRecommandationsValides":2}]}
```

### 8.3 `server_stopping`

```json
{"type":"server_stopping"}
```

À sa réception, le client affiche un message et revient à l'écran de connexion.

---

## 9. Règles de traitement

* Le serveur est le **seul** à décider : il valide ou refuse chaque demande.
* Le serveur traite les demandes de plusieurs clients en parallèle. Le minage d'un bloc ne bloque pas les autres clients.
* Le serveur répond toujours à une demande, par un message de succès ou par `error`.
* Un message mal formé, trop long ou de type inconnu reçoit `error` avec `INVALID_DATA`. La connexion reste ouverte.
* Le client ignore les champs inconnus d'un message et les types de notification qu'il ne connaît pas.
* Un client automatique envoie `"isBot":true` dans `login`. Ses actions sont ainsi distinguées de celles d'un humain, notamment dans la blockchain.
* Après `battle_finished`, les cartes redeviennent utilisables : la gagnante repasse `ACTIVE` (ou `AUTHENTIFIEE`), la perdante est `INACTIVE` et ne peut plus être recommandée ni engagée.

---

## 10. Tableau récapitulatif

| Demande | Réponse de succès | Notifications déclenchées |
| :--- | :--- | :--- |
| `login` | `login_ok` | `users_updated` |
| `create_card` | `card_created` | `card_added` |
| `recommend` | `recommend_ok` | `card_updated` |
| `repudiate` | `repudiate_ok` | `card_updated` |
| `launch_battle` | `battle_started` | `battle_started` (tous), puis `battle_finished` |
| `vote` | `vote_ok` | aucune |
| `request_legitimation` | `legitimation_ok` | `users_updated` |
| `claim_card` | `claim_ok` | `claim_ok` (ancien propriétaire), `card_updated` |
| `authenticate_card` | `authenticate_ok` | `card_updated` |
| `get_context` | `context_ok` | aucune |
| `verify_blockchain` | `verify_ok` | aucune |
| `disconnect` | `disconnect_ok` | `users_updated` |

---

## 11. Points restant à valider par le groupe

| Point | Question |
| :--- | :--- |
| Lecture du JSON | Bibliothèque non autorisée par le cahier des charges : écrire un petit analyseur JSON (en C et en Java) ou demander l'autorisation au responsable de la SAÉ ? |
| Nom unique de carte | `DUPLICATE_CARD` repose sur le `nom` : à confirmer avec l'équipe serveur |
| Légitimation | Choix « globale » retenu ici ; à confirmer dans `regles-metier.md` |
| Arrêt du serveur | Déclenché depuis la console du serveur ; à confirmer avec l'équipe serveur |

---

## 12. Fichiers liés

* `cas-erreur.md` : codes d'erreur
* `regles-metier.md` : règles de la plateforme
* `architecture-client.md` : traitement des messages côté client
* `annexe-d-sequences.md` : diagrammes de séquence