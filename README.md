# demo-ansible — playbooks de démonstration Ansible-UI

Playbooks appelés depuis l'IHM **Ansible-UI** (module engine-core), avec un
inventaire **dynamique fourni par NetBox** (Source of Truth).

## Inventaire

Aucun fichier d'inventaire statique ici : l'inventaire est synchronisé depuis
NetBox par Ansible-UI. Chaque hôte expose les variables issues de NetBox :
`ansible_host` (primary IP), `netbox_role`, `netbox_site`, `netbox_platform`,
`netbox_tenant`, et les tags comme groupes.

## Playbooks

| Fichier | Cible | Rôle |
|---|---|---|
| `ping.yml` | tous | Vérifie la connectivité SSH + Python et affiche les vars NetBox |
| `gather_facts.yml` | tous | Collecte et résume les facts système (OS, CPU, RAM) |
| `deploy_web.yml` | rôle `web` | Déploie une page web statique et la sert en HTTP |
| `manage_user.yml` | tous | Crée/supprime un utilisateur applicatif (extra-vars `app_user`, `app_state`) |

## Connexion

Les serveurs cibles acceptent l'utilisateur `ansible` par clé SSH. La clé
privée correspondante est fournie à Ansible-UI sous forme de *credential*
de type SSH dans l'IHM.

## Extra-vars utiles

- `manage_user.yml` : `app_user` (défaut `appsvc`), `app_state` (`present`/`absent`)
- `deploy_web.yml` : `listen_port` (défaut `8080`)
