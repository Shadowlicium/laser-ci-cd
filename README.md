# laser-ci-cd

CI/CD – Appli 1 (HTML via Nginx) & Appli 2 (Image Docker)

Ce dépôt met en place deux pipelines GitHub Actions :

Appli 1 – Déploiement pré-prod d’un site statique sur un serveur public via SSH + Nginx (.github/workflows/deploy.yml).

Appli 2 – Mise à disposition d’une image sur Docker Hub (retag/push de l’image officielle WordPress) (.github/workflows/lamp-ci.yml).

Hébergement : la VM (Nginx). GitHub stocke le code et exécute le CI/CD ; le job SCP copie les fichiers du dépôt vers /var/www/app sur la Machine Virtuelle.

Arborescence
```
.
├─ .github/
│  └─ workflows/
│     ├─ deploy.yml       # Appli 1 : déploiement HTML via SSH/Nginx
│     └─ lamp-ci.yml      # Appli 2 : retag/push image sur Docker Hub
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

2) Appli 2 — `dep-stack.yml`

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
Lien du Dockerhub pour télécharger l'image :

https://hub.docker.com/repository/docker/shadowlicium/wordpress-ci/general

Commande pour pull le conteneur

docker pull shadowlicium/wordpress-ci:tagname
