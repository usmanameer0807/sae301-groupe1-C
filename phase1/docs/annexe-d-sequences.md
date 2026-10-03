# Annexe D : Diagrammes de séquence

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C

Ces diagrammes montrent les échanges entre l'utilisateur, l'interface JavaFX, les classes du client et le serveur. Pour chaque action, le serveur valide ou refuse la demande. Seule une action validée est inscrite dans la blockchain.

## 1. Connexion

![Séquence : connexion](src/diagrammes-client/03-sequence-login.png)

Ouverture de la socket, `login`, puis reconstitution du contexte par le serveur. Erreurs : `SERVER_BUSY`, `USERNAME_TAKEN`.

## 2. Création d'une carte

![Séquence : création d'une carte](src/diagrammes-client/04-sequence-create-card.png)

Le serveur génère l'identifiant de la carte. Erreurs : `INVALID_DATA`, `DUPLICATE_CARD`, `MAX_CARDS_REACHED`.

## 3. Recommandation

![Séquence : recommandation](src/diagrammes-client/05-sequence-recommend.png)

La VA de la carte augmente de 1. Erreurs : `CARD_NOT_FOUND`, `CARD_NOT_ACTIVE`, `ALREADY_RECOMMENDED`.

## 4. Répudiation

![Séquence : répudiation](src/diagrammes-client/06-sequence-repudiate.png)

La VA diminue de 1, l'historique reste dans la blockchain. Erreurs : `CARD_NOT_FOUND`, `NOT_RECOMMENDED`.

## 5. Lancement d'une battle

![Séquence : lancement d'une battle](src/diagrammes-client/07-sequence-battle.png)

Le serveur notifie tous les clients (`battle_started`) et démarre le chrono de 60 secondes. Erreurs : `CARDS_NOT_COMPATIBLE`, `VA_TOO_LOW`, `BATTLE_ALREADY_ACTIVE`, `NOT_ALLOWED`.

## 6. Vote et résultat

![Séquence : vote](src/diagrammes-client/08-sequence-vote.png)

Un utilisateur, un vote. À la fin des 60 secondes, le serveur envoie `battle_finished` : la gagnante récupère les recommandations, la perdante devient `INACTIVE`.

## 7. Légitimation

![Séquence : légitimation](src/diagrammes-client/09-sequence-legitimation.png)

Possible à partir de 3 recommandations valides. Erreur : `THRESHOLD_NOT_REACHED`.

## 8. Authentification

![Séquence : authentification](src/diagrammes-client/10-sequence-authentification.png)

La carte passe à `AUTHENTIFIEE`, avec +10 VA et des champs verrouillés. Erreurs : `NOT_LEGITIMATE`, `ALREADY_AUTHENTICATED`.

## 9. Revendication

![Séquence : revendication](src/diagrammes-client/11-sequence-revendication.png)

Transfert direct de propriété (`ACTION_CLAIM_CARD`) ; `createurId` ne change pas. Erreurs : `NOT_LEGITIMATE`, `NOT_ALLOWED`.

## 10. Déconnexion

![Séquence : déconnexion](src/diagrammes-client/12-sequence-disconnect.png)

Dé