# Répartition des rôles

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C

---

## 1. Composition du groupe

Le groupe est composé de 5 étudiants, répartis en trois équipes selon les parties du projet.

| Équipe | Effectif | Partie | Technologies |
| :--- | :---: | :--- | :--- |
| Client | 2 | Application utilisateur et interface graphique | Java, JavaFX, JUnit |
| Serveur | 2 | Serveur multitâche et communication par sockets | C, sockets, threads POSIX |
| Blockchain / BDD | 1 | Blockchain, preuve de travail, base de données | C, OpenSSL (SHA-256), PostgreSQL (libpq) |

---

## 2. Responsabilités par équipe

### 2.1 Équipe Client

* conception de l'architecture du client et des classes ;
* écrans JavaFX (connexion, cartes, création, battles, profil) ;
* envoi des demandes et traitement des réponses du serveur ;
* tests JUnit du client ;
* éventuellement, client automatique (bot).

### 2.2 Équipe Serveur

* acceptation des clients et limite de connexions (`MAX_CLIENTS = 20`) ;
* traitement des demandes en parallèle (multitâche) ;
* application des règles métier (validation ou refus des actions) ;
* gestion des battles et du chronomètre de vote ;
* tests du serveur en C.

### 2.3 Équipe Blockchain / BDD

* structure des blocs, calcul du hash SHA-256 et minage ;
* vérification de la cohérence de la blockchain ;
* schéma relationnel PostgreSQL, sauvegarde et chargement de la blockchain ;
* restauration du contexte au démarrage ;
* test d'altération d'un bloc.

---

## 3. Rôles transversaux

En plus de leur partie, certains membres ont un rôle pour l'ensemble du groupe.

| Rôle | Mission | Membre |
| :--- | :--- | :--- |
| Chef de projet | Organiser le travail, suivre le planning, faire le lien avec les enseignants | *à compléter* |
| Responsable qualité | Vérifier la cohérence des documents et du code, les commentaires en anglais, la limite de 200 lignes par fonction | *à compléter* |
| Responsable Git | Gérer le dépôt GitLab, les branches et les labels `phase1` à `phase4` | *à compléter* |
| Responsable documentation | Tenir à jour les fichiers de `docs/` et la spécification technique | *à compléter* |

Les rôles de **concepteurs**, **codeurs** et **testeurs** sont partagés : chaque équipe conçoit, code et teste sa propre partie.

---

## 4. Travail en commun

Certains documents concernent tout le groupe et sont rédigés ensemble :

* les règles métier (`regles-metier.md`) ;
* le protocole applicatif (`protocole-applicatif-commun.md`) ;
* les codes d'erreur (`cas-erreur.md`) ;
* le diagramme de déploiement et les diagrammes généraux.

Toute modification d'un de ces documents est validée par les trois équipes, car elle a un impact sur le client, le serveur et la base de données.

---

## 5. Organisation

* **Réunions :** une réunion de groupe par semaine pour faire le point et décider des choix communs.
* **Git :** chaque équipe travaille dans son répertoire (`client-java/`, `server-c/`). Les livrables sont labellisés `phase1`, `phase2`, `phase3` et `phase4`.
* **Répartition du temps :** le temps de SAÉ est partagé entre le projet principal (60 %) et les deux SAÉ ressources (20 % chacune).

---

## 6. Planning des phases

| Phase | Contenu | Période |
| :--- | :--- | :--- |
| Phase 1 | Conception et spécification | Septembre au 10 octobre 2026 |
| Phase 2 | Codage et tests | 12 octobre au 18 décembre 2026 |
| Phase 3 | Qualité et consolidation | 4 au 15 janvier 2027 |
| Phase 4 | Présentation et démonstration | 18 au 21 janvier 2027 |

Un diagramme de Gantt sera ajouté en phase 3, comme demandé par le cahier des charges.

---

## 7. Fichiers liés

* `README.md` : organisation de la documentation
* `regles-metier.md` : règles de la plateforme