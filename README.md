# 👨‍💻 Projet Docker — La Plateforme

## 📌 Objectif

Installer et utiliser Docker sur Debian, puis apprendre à créer des images personnalisées, utiliser Docker Compose, et orchestrer des services.

---

## 🖥️ Environnement

- VM Debian 13 (console)
- 1 vCPU | 1 Go RAM | 8 Go disque
- Installation de Docker via dépôt officiel

---

# 🚀 Job 01 — Installation Docker (CLI)

## Mise à jour & installation

```bash
apt update
apt install -y ca-certificates curl gnupg lsb-release

mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg \
  | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
  | tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

systemctl enable --now docker
systemctl status docker --no-pager
docker --version
```

---

# 🧪 Job 02 — Test hello-world

## Test

```bash
docker run hello-world
```

## Commandes essentielles

```bash
docker ps
docker ps -a
docker images
docker pull debian:stable-slim
docker run -it debian:stable-slim bash
docker stop <container>
docker rm <container>
docker rmi <image>
docker logs <container>
docker exec -it <container> bash
```

---

# 🐳 Job 03 — Dockerfile personnalisé (Hello World)

## Objectif

Créer une image personnalisée équivalente à hello-world en utilisant Debian minimale.

## Dockerfile

```dockerfile
FROM debian:stable-slim

RUN apt-get update \
 && apt-get install -y --no-install-recommends cowsay \
 && ln -sf /usr/games/cowsay /usr/local/bin/cowsay \
 && rm -rf /var/lib/apt/lists/*

CMD ["/bin/sh","-lc","echo 'Hello from my custom Docker container!' && cowsay 'Docker Job 03 - Alexandre'"]
```

## Build & Run

```bash
docker build --no-cache -t my-hello .
docker run --rm my-hello
```

---

# 🛠️ Job 04 — Image SSH personnalisée

## Dockerfile SSH

```bash
FROM debian:stable-slim

RUN apt-get update \
 && apt-get install -y --no-install-recommends openssh-server \
 && mkdir -p /run/sshd \
 && echo "root:root123" | chpasswd \
 && sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin yes/' /etc/ssh/sshd_config \
 && sed -i 's/^#\?PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config \
 && rm -rf /var/lib/apt/lists/*

EXPOSE 2222
CMD ["/usr/sbin/sshd","-D","-p","2222"]
```

## Build & Test

```bash
docker build -t my-ssh .
docker run -d --name ssh-test -p 2222:2222 my-ssh
```

Test SSH :

```bash
ssh -p 2222 root@localhost
# Mot de passe : root123
```

Stop & Cleanup :

```bash
docker stop ssh-test
docker rm ssh-test
```

---

# 🧠 Job 05 — Alias Docker dans ~/.bashrc

```bash
nano ~/.bashrc
```

Ajout :

```bash
# ---- Docker aliases ----
alias d='docker'
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dr='docker run'
alias dri='docker run -it --rm'
alias dst='docker stop'
alias drm='docker rm'
alias drmi='docker rmi'
alias dlog='docker logs'
alias dex='docker exec -it'
alias dprune='docker system prune -af'
# -------------------------------
```

---

# 📦 Job 06 — Volumes Docker

## 1️⃣ Bind Mount

```bash
mkdir ~/volume-test
echo "Bonjour depuis l'hôte Debian" > ~/volume-test/index.html

docker run -d --name nginx-bind -p 8080:80 \
  -v ~/volume-test:/usr/share/nginx/html nginx

curl http://localhost:8080
```

## 2️⃣ Volume nommé

```bash
docker volume create myvolume

docker run -d --name nginx-volume -p 8081:80 \
  -v myvolume:/usr/share/nginx/html nginx

docker volume inspect myvolume
```

## 3️⃣ Partage entre conteneurs

```bash
docker run -it --rm -v myvolume:/data debian:stable-slim bash
echo "Fichier écrit depuis un autre conteneur" > /data/test.txt
exit

curl http://localhost:8081/test.txt
```

---

# 📁 Job 07 — Docker Compose (Nginx + FTP + Volume)

## Objectif

Créer une stack avec :

- Nginx
- FTP
- Volume partagé
- Upload via FTP visible sur Nginx

## docker-compose.yml

```bash
version: '3.8'

services:

  web:
    image: nginx:latest
    container_name: nginx_server
    ports:
      - "8080:80"
    volumes:
      - webdata:/usr/share/nginx/html
    restart: always

  ftp:
    image: fauria/vsftpd
    container_name: ftp_server
    ports:
      - "21:21"
      - "21100-21110:21100-21110"
    environment:
      - FTP_USER=alex
      - FTP_PASS=alex123
      - PASV_ADDRESS=192.168.X.X
      - PASV_MIN_PORT=21100
      - PASV_MAX_PORT=21110
    volumes:
      - webdata:/home/vsftpd/alex
    restart: always

volumes:
  webdata:
```

## Lancement

```bash
docker compose up -d
docker ps
```

## Test Nginx

```
http://IP_DE_LA_VM:8080
```

## Test FTP (FileZilla)

- Hôte : IP_DE_LA_VM
- Port : 21
- Utilisateur : alex
- Mot de passe : alex123
- Mode : Passif

## Résultat

Upload via FTP → visible sur Nginx.

---

# 📦 Job 08 — Image Docker personnalisée

## Objectif

Créer une image Docker avec Nginx qui embarque un index.html personnalisé.

## Création

```bash
mkdir job08
cd job08
```

### index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Job 08</title>
</head>
<body style="background:black;color:lime;text-align:center;padding-top:100px;">
    <h1>🔥 Job 08 Docker Image Custom</h1>
    <p>Image créée par Alex</p>
</body>
</html>
```

### Dockerfile

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

## Build & Run

```bash
docker build -t alex-nginx .
docker run -d -p 8081:80 --name job08 alex-nginx
```

## Test

```
http://IP_DE_LA_VM:8081
```

## Commandes utiles

```bash
docker stop job08
docker rm job08
docker rmi alex-nginx
```

---

# 📦 Job 09 — Docker Registry Local + UI

## 🎯 Objectif

Mettre en place :

- Un Docker Registry local
- Une interface Web (Docker Registry UI)
- Stockage persistant via volume
- Push d’une image dans le registry
- Vérification du catalogue

---

## 📁 Création du dossier

```bash
mkdir job09
cd job09
```

---

## 📝 Création du docker-compose.yml

```bash
nano docker-compose.yml
```

Contenu :

```yaml
services:
  registry:
    image: registry:2
    container_name: local_registry
    restart: always
    ports:
      - "5000:5000"
    environment:
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Origin: '["http://192.168.58.142:8089"]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Methods: '["HEAD","GET","OPTIONS"]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Credentials: '["true"]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Headers: '["Authorization","Accept"]'
    volumes:
      - registrydata:/var/lib/registry

  ui:
    image: joxit/docker-registry-ui:latest
    container_name: registry_ui
    restart: always
    ports:
      - "8089:80"
    environment:
      - SINGLE_REGISTRY=true
      - REGISTRY_TITLE=Local Docker Registry
      - REGISTRY_URL=http://registry:5000
    depends_on:
      - registry

volumes:
  registrydata:
```

⚠️ Adapter l’IP si nécessaire (ici : 192.168.58.142).

---

## 🚀 Lancement des services

```bash
docker compose up -d
docker ps
```

---

## 🌐 Accès à l’interface Web

Navigateur :

```
http://IP_DE_LA_VM:8089
```

---

## 📤 Push d’une image dans le registry

Télécharger une image :

```bash
docker pull nginx:latest
```

Tag vers le registry local :

```bash
docker tag nginx:latest 127.0.0.1:5000/nginx:latest
```

Push vers le registry :

```bash
docker push 127.0.0.1:5000/nginx:latest
```

---

## 🔎 Vérification du catalogue

```bash
curl http://127.0.0.1:5000/v2/_catalog
```

Résultat attendu :

```json
{"repositories":["nginx"]}
```

---

## 📦 Vérification via UI

Actualiser la page :

```
http://IP_DE_LA_VM:8089
```

Le repository `nginx` doit apparaître.

---

## 🛑 Arrêt et nettoyage

```bash
docker compose down
```

Supprimer les volumes :

```bash
docker compose down -v
```

---

# 🛠️ Job 10 — Scripts Bash (Désinstallation + Installation Docker)

## 🎯 Objectif

Créer deux scripts Bash :

1. Script de suppression complète de Docker
2. Script d’installation automatique de Docker

Automatiser totalement la gestion de Docker sur Debian.

---

# 📜 Script 1 — Désinstallation complète Docker

## Création du script

```bash
nano uninstall_docker.sh
```

Contenu :

```bash
#!/bin/bash

echo "Stopping Docker service..."
systemctl stop docker 2>/dev/null

echo "Removing Docker containers..."
docker rm -f $(docker ps -aq) 2>/dev/null

echo "Removing Docker images..."
docker rmi -f $(docker images -q) 2>/dev/null

echo "Removing Docker volumes..."
docker volume rm $(docker volume ls -q) 2>/dev/null

echo "Removing Docker networks..."
docker network rm $(docker network ls -q | grep -v "bridge\|host\|none") 2>/dev/null

echo "Purging Docker packages..."
apt purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 2>/dev/null

echo "Removing Docker directories..."
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
rm -rf /etc/docker

echo "Docker completely removed."
```

## Rendre exécutable

```bash
chmod +x uninstall_docker.sh
```

## Exécution

```bash
sudo ./uninstall_docker.sh
```

---

# 📦 Script 2 — Installation automatique Docker

## Création du script

```bash
nano install_docker.sh
```

Contenu :

```bash
#!/bin/bash

echo "Updating system..."
apt update

echo "Installing dependencies..."
apt install -y ca-certificates curl gnupg lsb-release

echo "Adding Docker GPG key..."
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg \
  | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

chmod a+r /etc/apt/keyrings/docker.gpg

echo "Adding Docker repository..."
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
  | tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "Installing Docker..."
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "Enabling Docker service..."
systemctl enable --now docker

echo "Docker installation completed."
docker --version
```

## Rendre exécutable

```bash
chmod +x install_docker.sh
```

## Exécution

```bash
sudo ./install_docker.sh
```

---
