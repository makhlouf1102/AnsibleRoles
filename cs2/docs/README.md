# Collection CS2 pour Lan ETS

## Installation

## Préparer une machine avec CS2 Server Manager

Le rôle `lanets.cs2.server_manager` installe le binaire officiel `csm`, exécute
`csm setup-host`, démarre un serveur avec `csm bootstrap` et installe Ready Up
depuis le canal beta sous le compte `cs2servermanager`. Fournissez
`cs2_server_manager_rcon_password` via Ansible Vault. Le rôle configure aussi
le choix de licence noncommercial, comme dans la configuration actuelle de
l'image. Une licence commerciale utilise `cs2_server_manager_license_key`.
Pour un hôte géré par les instances de la plateforme, désactivez le bootstrap
et activez
`cs2_server_manager_configure_instance_backend`. Après le déploiement, liez la
machine avec la commande à usage unique fournie par **Servers → Machines → Add
machine**. Voir
`roles/server_manager/README.md` pour les variables et un exemple de playbook.
