# Documentation — Phase 1

> **Projet :** SAÉ BUT2 S3 (2026-2027), Plateforme SAÉ Recommender
> **Équipe :** Groupe 1-C
> **Thème :** Langages de programmation
> **Livrable :** Phase 1, Conception et spécification (label Git `phase1`)

---

## 1. Présentation

Ce répertoire contient la documentation de conception de la plateforme SAÉ Recommender : une plateforme client-serveur de recommandations de langages de programmation, fondée sur une blockchain privée simplifiée.

La documentation existe sous deux formes :

* les **fichiers Markdown** de ce répertoire, organisés par sujet ;
* le document unique **`specification-technique.md`**, qui présente l'ensemble de manière rédigée pour toutes les parties prenantes.

Les deux formes sont maintenues cohérentes tout au long du projet.

---

## 2. Organisation du projet

Le projet est réalisé par un groupe de 5 étudiants, réparti en trois équipes :

| Équipe | Partie |
| :--- | :--- |
| Client | Client Java / JavaFX |
| Serveur | Serveur C et sockets |
| Blockchain / BDD | Blockchain et PostgreSQL |

Le détail des rôles est dans [repartition-roles.md](repartition-roles.md).

---

## 3. Documents communs

À lire en premier : ils concernent tout le groupe.

| Fichier | Contenu |
| :--- | :--- |
| [repartition-roles.md](repartition-roles.md) | Répartition des rôles dans l'équipe |
| [regles-metier.md](regles-metier.md) | Règles de la plateforme (cartes, VA, battles, légitimation…) |
| [protocole-applicatif-commun.md](protocole-applicatif-commun.md) | Format des messages entre clients et serveur |
| [cas-erreur.md](cas-erreur.md) | Les 20 codes d'erreur et leur traitement |

---

## 4. Documents par partie

### 4.1 Client

| Fichier | Contenu |
| :--- | :--- |
| [architecture-client.md](architecture-client.md) | Organisation du client JavaFX (couches, threads, écrans) |
| [donnees-client.md](donnees-client.md) | Classes et attributs du client |

### 4.2 Serveur

| Fichier | Contenu |
| :--- | :--- |
| [architecture-serveur.md](architecture-serveur.md) | Organisation du serveur C (multitâche, sockets) |
| [donnees-serveur.md](donnees-serveur.md) | Structures de données du serveur |

### 4.3 Blockchain et base de données

| Fichier | Contenu |
| :--- | :--- |
| [structures-blockchain.md](structures-blockchain.md) | Structure des blocs, hachage, preuve de travail |
| [schema-bdd.md](schema-bdd.md) | Schéma relationnel PostgreSQL |

---

## 5. Annexes (diagrammes)

| Fichier | Contenu |
| :--- | :--- |
| [annexe-a-parcours-utilisateur.md](annexe-a-parcours-utilisateur.md) | Parcours utilisateur |
| [annexe-b-cas-utilisation.md](annexe-b-cas-utilisation.md) | Diagrammes de cas d'utilisation |
| [annexe-c-classes.md](annexe-c-classes.md) | Diagrammes de classes |
| [annexe-d-sequences.md](annexe-d-sequences.md) | Diagrammes de séquence |
| [annexe-e-deploiement.md](annexe-e-deploiement.md) | Diagramme de déploiement |

Ces annexes correspondent aux cinq types de diagrammes UML retenus, qui est la limite fixée par le cahier des charges.

Les images et le code source des diagrammes sont dans le dossier [src/](src/) :

* `src/diagrammes-client/`
* `src/diagrammes-serveur/`
* `src/diagrammes-blockchain-bdd/`
* `src/diagrammes-generaux/`

---

## 6. Document de spécification technique

Le fichier [specification-technique.md](specification-technique.md) rassemble de façon rédigée les choix de conception. Il est destiné à toutes les parties prenantes du projet, y compris celles qui ne participent pas au développement. Les éléments les plus techniques sont renvoyés dans les annexes.

Ce document est aussi livré dans le dossier OneDrive partagé avec le professeur.

---

## 7. Ordre de lecture conseillé

1. `regles-metier.md`
2. `protocole-applicatif-commun.md`
3. Le document de votre partie (client, serveur ou blockchain/BDD)
4. Les annexes correspondantes
5. `specification-technique.md`

---

## 8. Livraison

* Dépôt GitLab, label **`phase1`**
* Date de livraison : **10 octobre 2026**