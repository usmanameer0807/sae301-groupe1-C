# Annexe A : Parcours utilisateur

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C

Ce diagramme montre le parcours de l'utilisateur dans l'interface JavaFX et les échanges entre l'interface, le modèle du client et le serveur.

## Diagramme de navigation

![Navigation JavaFX](src/diagrammes-client/13-navigation-javafx.png)

* L'utilisateur arrive sur l'écran de connexion, puis navigue entre les écrans avec le menu de l'écran principal.
* Chaque action passe par `RequestService`, est validée par le serveur, puis `ResponseHandler` met à jour `ClientState` et l'écran.
* Une déconnexion ou une connexion perdue ramène à l'écran de connexion.