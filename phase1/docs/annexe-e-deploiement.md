# Annexe E : Diagramme de déploiement

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C

Ce diagramme montre la couche physique du système : où s'exécute chaque programme et comment ils communiquent.

## Diagramme de déploiement

![Diagramme de déploiement](src/diagrammes-generaux/20-deploiement.png)

* **Clients :** jusqu'à 20 clients simultanés (`MAX_CLIENTS = 20`), sous Linux Debian. Chaque client Java / JavaFX (ou bot) ouvre une socket TCP vers le serveur.
* **Serveur :** un seul processus en C, multitâche (threads POSIX), qui contient la blockchain en mémoire. Il est le seul à accéder à la base de données.
* **Base de données :** PostgreSQL sur `linserv-info-01` (LAN de l'IUT), accessible uniquement par le serveur (libpq).
* **Échanges :** JSON, UTF-8, un message par ligne (`\n`). Aucune API REST.