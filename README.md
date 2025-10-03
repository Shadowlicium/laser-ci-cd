# laser-ci-cd

CI/CD – Appli 1 (HTML via Nginx) & Appli 2 (Image Docker)

Ce dépôt met en place deux pipelines GitHub Actions :

Appli 1 – Déploiement pré-prod d’un site statique sur un serveur public via SSH + Nginx (.github/workflows/deploy.yml).

Appli 2 – Mise à disposition d’une image sur Docker Hub (retag/push de l’image officielle WordPress) (.github/workflows/dep-stack.yml).

Hébergement : la VM (Nginx). GitHub stocke le code et exécute le CI/CD ; le job SCP copie les fichiers du dépôt vers /var/www/app sur la Machine Virtuelle.

Arborescence
```
.
├─ .github/
│  └─ workflows/
│     ├─ deploy.yml       # Appli 1 : déploiement HTML via SSH/Nginx
│     └─ dep-stack.yml      # Appli 2 : retag/push image sur Docker Hub
├─ public/
│  └─ index.html          # Le site statique à déployer
└─ README.md
```


---

## ⚙️ Workflows

### 1) Appli 1 — `deploy.yml` (SSH → Nginx)
```
**Pipeline :**
1. Prépare la cible `/var/www/app` **avant** la copie (création + droits).
2. Copie `public/**` sur le serveur (SCP), en déposant le **contenu** dans `/var/www/app`.
3. Installe/Configure Nginx puis restart.

**Secrets utilisés :**
- `SSH_HOST` : IP publique
- `SSH_PORT` : port externe
- `SSH_USER` : utilisateur SSH
- `SSH_PRIVATE_KEY` : clé privée OpenSSH

**Host Nginx appliqué :**
nginx
server {
  listen 80;
  server_name _;
  root /var/www/app;
  index index.html;
  location / { try_files $uri $uri/ =404; }
}
```

### 2) Appli 2 — `dep-stack.yml`

```
Objectif demandé : mettre l’appli 2 en accès téléchargement via un Docker Registry.
Ici : retag de l’image WordPress officielle + smoke test + push Docker Hub.

Étapes :

1. docker pull wordpress:latest
2. docker run --rm wordpress:latest php -v
3. docker tag wordpress:latest $DOCKER_IMAGE
4. docker push $DOCKER_IMAGE

Secrets utilisés :

DOCKER_HUB_USERNAME
DOCKER_HUB_ACCESS_TOKEN
DOCKER_IMAGE
```
Les secrets mise en place pour l'accès a mon DockerHub

| Secret                         | Rôle                 |
| ------------------------------ | -------------------- | 
| `DOCKER_HUB_USERNAME`          | Login Docker Hub     |
| `DOCKER_HUB_ACCESS_TOKEN`      | PAT Docker Hub       |
| `DOCKER_IMAGE`                 | Repo/Tag cible       |

## L'exposition

La machine virtuelle est exposé depuis un réseau NAT avec plusieurs ouverture de port via Virtualbox

HTTP : Host 80 ➜ Guest 10.0.2.15:80

SSH : Host 22 ➜ Guest 10.0.2.15:22

Ainsi que part ma box internet


Livebox (WAN → PC hôte) :

HTTP : WAN 80 ➜ LAN 80 (PC hôte)

SSH : WAN 22 ➜ LAN 22 (PC hôte)

Une sécurité a bien été mise en place du coté SSH et n'autorise une connexion que via une clé SSH.

## Problème rencontré

Le problème que j'ai eu est dû a mes machines virtuelles ayant un réseau très faible en accès par pont pour une raison que j'ignore mais fonctionne très bien par le nat ou en privé, j'ai donc opté sur la solution du nat pour pallier a mon problème

## Livrable

URL publique du site (appli 1) : http://90.25.253.205

Lien Docker Hub (appli 2) : https://hub.docker.com/repository/docker/shadowlicium/wordpress-ci/

Ainsi que le fonctionnement des workflows.
<img width="1273" height="164" alt="image" src="https://github.com/user-attachments/assets/f2a00d69-d07c-45ce-882d-5240320d1086" />

