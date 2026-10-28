# Collection CS2 pour Lan ETS

## Installation

## Déploiement du serveur avec Docker Compose

Le playbook `events/lanets/cs2-server` installe Docker, télécharge le fichier
Compose inclus dans le rôle `lanets.cs2.csm`, puis démarre l'image publiée
`git.etsmtl.club/lan_ets/cs2-csm-server`. Les pushes sur `main` publient le tag
`dev`; les tags de version publient aussi `latest`. Le tag se règle avec
`csm_image_tag`.

Le mot de passe RCON vient de `cs2_rcon_password` dans l'inventaire et est écrit
dans `/opt/cs2/.env` avec les permissions `0600`. Si le registre Forgejo exige
une authentification, définissez `csm_registry_username` et
`csm_registry_password` dans les variables chiffrées de l'inventaire.
