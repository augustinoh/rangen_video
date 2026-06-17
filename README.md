## README : Apprentissage par Imitation (Diffusion Policy) sur Robot RanGen

### 🤖 Présentation du Projet

Ce projet implémente un algorithme d'apprentissage par imitation (basé sur l'architecture *Diffusion Policy*) sur le manipulateur mobile autonome RanGen. Le robot RanGen est composé d'une base mobile holonome AgileX Ranger Mini 3.0 et d'un bras collaboratif Kinova Gen3 à 7 degrés de liberté.

Le système est une adaptation de l'architecture logicielle Tidybot2, modifiée pour intégrer un contrôle robuste par manette PS4 et une architecture ROS2 centralisée.

### ⚙️ Architecture Matérielle et Capteurs

L'algorithme de politique de diffusion s'appuie sur des données multimodales (vision et proprioception):


**Vision :** 3 caméras (caméra embarquée sur le poignet, caméra sur la base, caméra en hauteur).



**Proprioception Bras (Kortex API à ~990 Hz) :** Positions, vitesses et couples moteurs des 7 articulations.



**Proprioception Base :** Position et orientation cartésiennes.



### 💻 Architecture Logicielle

L'architecture modulaire repose sur ROS2 et des scripts Python communicant via un serveur RPC:


`manette_PS4.py` : Nœud ROS2 gérant les inputs Bluetooth à 10 Hz et publiant sur les topics de contrôle (vitesses cartésiennes, gripper, base).



`policies.py` : Script central qui intègre les vitesses pour générer les commandes de position et compile le dictionnaire d'actions.



`base_controller.py` : Intègre la bibliothèque **Ruckig** pour générer des trajectoires fluides et optimisées à 250 Hz (4 ms) vers la base AgileX via `/cmd_vel`.



`main.py` : Centralise le thread ROS2 pour éviter les conflits liés aux multiples flux de caméras et de capteurs.



### 🚀 Pipeline d'Utilisation

1. **Collecte de données :** Pilotage intégral du robot (bras, base, et gripper) via la manette PS4 en mode cartésien. La collecte vise entre 50 et 100 démonstrations fluides par tâche (ex: saisir un objet, le déplacer).


2. **Entraînement :** Les vecteurs d'observation (images + états des capteurs filtrés) sont passés au modèle RDT-1B / Diffusion Policy pour l'apprentissage des actions.


3. **Exécution Autonome :** Le modèle infère les actions en boucle fermée et transmet les commandes de position au serveur RPC pour exécution par les contrôleurs.

Video de pick and place avec seulement le bras robotique kinova 7 dof
https://github.com/user-attachments/assets/a0bc388a-f0e7-4a2d-964e-ad181d15c856

Video de pick and place avec la base mobile Rangen Mini scout 3.0 et le bras robotique Kinova gen3 7 dof
https://github.com/user-attachments/assets/b1e04bfb-0b0a-4181-8cac-9f66f6774294

---
## Synthèse : Contrôle en Admittance Intégral avec Pinocchio

* **Problème initial :** L'estimation des forces externes via l'observateur de moment généralisé s'est révélée trop bruitée et instable lors des mouvements dynamiques du robot.
* **Solution technique :** Intégration de la bibliothèque logicielle **Pinocchio** pour calculer le modèle dynamique inverse du bras Kinova Gen3 avec une haute précision.
* **Estimation fiable :** Grâce au calcul rigoureux de la dynamique (et à la compensation de la gravité/friction), le système parvient à isoler et à mesurer précisément la force externe ($F_{ext}$) appliquée directement à l'effecteur.
* **Déclenchement par seuil :** Mise en place d'un seuil de détection. Le système reste inactif jusqu'à ce que la poussée ou la traction de l'opérateur sur l'effecteur dépasse cette valeur limite.
* **Couplage Bras-Base :** Une fois le seuil franchi, le système convertit cet effort en commandes de vitesse qui sont envoyées à la base mobile AgileX, lui permettant d'accompagner le mouvement du bras.
* **Validation expérimentale :** Tests physiques concluants sur le robot RanGen. L'ensemble se comporte comme un système compliant, permettant un guidage manuel complet, fluide et intuitif de la base et du bras par simple contact physique.

Video du déplacement du robot via un seuil de force calculé à l'extrémité du bras robotique Kinova avec la bibliothèque Pinoccio
https://github.com/user-attachments/assets/f408d853-7bc2-4816-b631-c5e470cdfde3



