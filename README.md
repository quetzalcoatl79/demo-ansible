# demo-ansible — playbooks de démonstration Ansible-UI

Playbooks appelés depuis l'IHM **Ansible-UI** (module engine-core), avec un
inventaire **dynamique fourni par NetBox** (Source of Truth).

## Inventaire

Aucun fichier d'inventaire statique ici : l'inventaire est synchronisé depuis
NetBox par Ansible-UI. Chaque hôte expose les variables issues de NetBox :
`ansible_host` (primary IP), `netbox_role`, `netbox_site`, `netbox_platform`,
`netbox_tenant`, et les tags comme groupes.

## Playbooks de base

| Fichier | Cible | Rôle |
|---|---|---|
| `ping.yml` | tous | Vérifie la connectivité SSH + Python et affiche les vars NetBox |
| `gather_facts.yml` | tous | Collecte et résume les facts système (OS, CPU, RAM) |
| `deploy_web.yml` | rôle `web` | Déploie une page web statique et la sert en HTTP |
| `manage_user.yml` | tous | Crée/supprime un utilisateur applicatif (extra-vars `app_user`, `app_state`) |

## Scénario applicatif 3-tiers (`app/`)

Transpose le storyboard NetDevOps (intent → déploiement → compliance → connectivité,
+ drift/heal) sur des serveurs Linux jouant web / app / db (rôle issu de NetBox).

| Fichier | Cible | Rôle |
|---|---|---|
| `app/configure.yml` | tous | Déploie le service applicatif selon le rôle (web:8080, app:9000, db:6000) |
| `app/compliance.yml` | tous | Contrôle que chaque service répond `healthy` — **échoue (rouge)** sinon |
| `app/connectivity.yml` | rôle `web` | Vérifie le chemin web → app → db à travers le réseau |
| `app/drift.yml` | rôle `web` | Simule une panne : arrête le service web (→ compliance rouge) |
| `app/heal.yml` | rôle `web` | Répare : relance le service depuis la config désirée (→ compliance verte) |

## Scénario hardening sécurité (`hardening/`)

Même logique appliquée à une baseline de sécurité.

| Fichier | Cible | Rôle |
|---|---|---|
| `hardening/baseline.yml` | tous | Applique la baseline (politique, perms strictes, users autorisés) |
| `hardening/audit.yml` | tous | Audite la conformité — **échoue (rouge)** si un invariant est violé |
| `hardening/drift.yml` | tous | Viole la baseline (compte non autorisé + perms relâchées) |
| `hardening/heal.yml` | tous | Ré-aligne sur la baseline (supprime le rogue, restaure les perms) |

### Logique pédagogique (rouge → vert)

`compliance`/`audit` **échouent volontairement** quand l'état désiré n'est pas
déployé. C'est ce qui permet, dans un workflow DAG, de brancher :
*compliance échoue → lancer configure/heal → re-compliance réussit*.

## Connexion

Les serveurs cibles acceptent l'utilisateur `ansible` par clé SSH. La clé
privée correspondante est fournie à Ansible-UI sous forme de *credential*
de type SSH dans l'IHM.

## Extra-vars utiles

- `manage_user.yml` : `app_user` (défaut `appsvc`), `app_state` (`present`/`absent`)
- `deploy_web.yml` : `listen_port` (défaut `8080`)
