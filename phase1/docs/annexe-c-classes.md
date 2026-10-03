# Annexe C : Diagramme de classes

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C

Ce diagramme montre les classes du client Java / JavaFX et leurs relations. Le détail des attributs est dans `donnees-client.md`.

## Diagramme de classes du client

![Diagramme de classes du client](src/diagrammes-client/02-diagramme-classes.png)

* **Modèle :** `Card`, `User`, `Battle` et `ClientState` (état courant du client) avec les énumérations `Domaine`, `StatutCarte`, `StatutBattle` et `ErrorCode`.
* **Services :** `RequestService` envoie les demandes, `ResponseHandler` traite les réponses et notifications puis met à jour `ClientState`.
* **Réseau :** `ServerConnection` gère la socket TCP, `MessageReader` est le thread qui lit les messages du serveur.
* **Vue :** `MainApp`, `SceneManager` et les six contrôleurs JavaFX, qui implémentent `ViewListener` pour être prévenus des changements.
* **Autres parties :** les diagrammes de classes du serveur et de la blockchain / BDD sont dans `src/diagrammes-serveur/` et `src/diagrammes-blockchain-bdd/`.