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

# 🖥️ Job 11 — Installation et utilisation de Portainer

## 🎯 Objectif

- Installer Portainer (interface Web de gestion Docker)
- Se connecter à l’environnement Docker local
- Refaire les Jobs 2 à 9 via l’interface graphique
- Comprendre la différence entre CLI et GUI

---

# 📦 Installation de Portainer

## Création du volume

```bash
docker volume create portainer_data
```

## Lancement du container Portainer

```bash
docker run -d \
  -p 9000:9000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

## Vérification

```bash
docker ps
```

---

# 🌐 Accès à l’interface

Navigateur :

```
http://IP_DE_LA_VM:9000
```

(ou HTTPS via le port 9443)

---

# 🔐 Configuration initiale

1. Créer un utilisateur administrateur
2. Sélectionner l’environnement **Docker (local)**
3. Se connecter à l’instance Docker

---

# 🔁 Reproduction des Jobs 2 à 9 via l’interface

## ✔ Job 2 — Hello World
- Menu **Containers**
- Add container
- Image : `hello-world`
- Deploy

---

## ✔ Job 3 — Build d’image personnalisée
- Menu **Images**
- Build a new image
- Coller le Dockerfile
- Build

---

## ✔ Job 4 — Container SSH
- Build image SSH
- Exposer le port 2222
- Deploy

---

## ✔ Job 5 — Gestion des containers
- Stop
- Restart
- Remove

---

## ✔ Job 6 — Volumes
- Menu **Volumes**
- Add volume
- Attacher au container

---

## ✔ Job 7 — Docker Compose
- Menu **Stacks**
- Add stack
- Coller le docker-compose.yml
- Deploy

---

## ✔ Job 8 — Image personnalisée
- Build image
- Déployer container

---

## ✔ Job 9 — Registry
- Déployer registry via Stacks
- Vérifier les images

---

# 🧠 Notions apprises

- Installation Portainer
- Gestion Docker via interface graphique
- Différence CLI vs GUI
- Déploiement de containers
- Gestion images, volumes et networks
- Déploiement via Docker Compose (Stacks)

---

# 🛑 Suppression Portainer

```bash
docker stop portainer
docker rm portainer
docker volume rm portainer_data
```

---

## Job XX — Pour aller plus loin : Stack type XAMPP (Nginx + PHP + MariaDB + phpMyAdmin + FTP)

### 🎯 Objectif
Reproduire un environnement type **XAMPP** avec Docker :
- **Nginx** (web)
- **PHP-FPM** (PHP)
- **MariaDB** (DB)
- **phpMyAdmin** (admin DB)
- **FTP** (upload)
- **1 volume partagé** pour les fichiers web (servis par Nginx + exécutés par PHP + upload via FTP)
- **1 volume DB** pour persister les données MariaDB

---

### 📁 Arborescence
```bash
jobXX-xampp-docker/
├─ docker-compose.yml
├─ .env               # ❌ ne pas push
├─ .gitignore
├─ php/
│  └─ Dockerfile
├─ nginx/
│  └─ default.conf
└─ app/
   └─ index.php
```

⚠️ Sécurité (.env)

Le fichier .env contient des identifiants → ne jamais le push sur GitHub.

```bash
echo ".env" >> .gitignore
```

🧱 1) Fichier .env

Créer le fichier suivant :

```bash
MYSQL_DATABASE=appdb
MYSQL_USER=alex
MYSQL_PASSWORD=alex123
MYSQL_ROOT_PASSWORD=root123

FTP_USER=alex
FTP_PASS=alex123
FTP_PASV_MIN=21100
FTP_PASV_MAX=21110
🐘 2) Dockerfile PHP (php/Dockerfile)
FROM php:8.2-fpm-bookworm

RUN apt-get update && apt-get install -y \
    libzip-dev \
 && docker-php-ext-install pdo_mysql mysqli \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html
```

🌐 3) Configuration Nginx (nginx/default.conf)

```bash
server {
    listen 80;
    server_name localhost;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastfastcgi_script_name;
    }
}
```

✅ Remarque : fastcgi_pass php:9000 pointe vers le service php du docker-compose.

✅ 4) Page de test PHP + DB (app/index.php)

```bash
<?php
echo "<h1>✅ OK : Nginx + PHP</h1>";

$host = "db";
$db   = getenv("MYSQL_DATABASE") ?: "appdb";
$user = getenv("MYSQL_USER") ?: "alex";
$pass = getenv("MYSQL_PASSWORD") ?: "alex123";

try {
  $pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8mb4", $user, $pass);
  echo "<p>✅ Connexion DB OK</p>";
  echo "<p>Version DB: " . $pdo->query("SELECT VERSION()")->fetchColumn() . "</p>";
} catch (Exception $e) {
  echo "<p>❌ DB KO: " . htmlspecialchars($e->getMessage()) . "</p>";
}
```

🧩 5) Docker Compose (docker-compose.yml)

```bash
services:
  nginx:
    image: nginx:alpine
    container_name: xampp_nginx
    ports:
      - "8080:80"
    volumes:
      - web_data:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - php
    networks:
      - xnet

  php:
    build: ./php
    container_name: xampp_php
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - web_data:/var/www/html
    networks:
      - xnet

  db:
    image: mariadb:11
    container_name: xampp_db
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - xnet

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: xampp_phpmyadmin
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: ${MYSQL_USER}
      PMA_PASSWORD: ${MYSQL_PASSWORD}
    depends_on:
      - db
    networks:
      - xnet

  ftp:
    image: fauria/vsftpd
    container_name: xampp_ftp
    ports:
      - "2121:21"
      - "${FTP_PASV_MIN}-${FTP_PASV_MAX}:${FTP_PASV_MIN}-${FTP_PASV_MAX}"
    environment:
      FTP_USER: ${FTP_USER}
      FTP_PASS: ${FTP_PASS}
      PASV_ADDRESS: 127.0.0.1
      PASV_MIN_PORT: ${FTP_PASV_MIN}
      PASV_MAX_PORT: ${FTP_PASV_MAX}
      LOCAL_UMASK: "022"
    volumes:
      - web_data:/home/vsftpd
    networks:
      - xnet

networks:
  xnet:

volumes:
  web_data:
  db_data:
```

✅ Ici, web_data est le volume partagé entre Nginx + PHP + FTP.

🚀 6) Lancement
```bash
docker compose up -d --build
docker compose ps
```
📦 7) Copier le fichier web dans le volume partagé

Comme web_data est un volume Docker (pas un dossier local), on injecte index.php dedans :
```bash
docker run --rm \
  -v xampp-docker_web_data:/data \
  -v "$(pwd)/app:/src" \
  alpine sh -c "cp /src/index.php /data/index.php"
```
Si le nom du volume diffère :
```bash
docker volume ls
```
✅ 8) Tests
🌐 Test web PHP

Ouvrir :
```bash
http://localhost:8080
```
Résultat attendu :

✅ OK : Nginx + PHP

✅ Connexion DB OK

🛠 Test phpMyAdmin

Ouvrir :
```bash
http://localhost:8081
```
Connexion :

Serveur : db

User : alex

MDP : alex123

📂 Test FTP (FileZilla)

Hôte : 127.0.0.1

Port : 2121

User : alex

Pass : alex123

Uploader un fichier (ex: index.html) puis vérifier :
```bash
http://localhost:8080/index.html
```
🛠 Commandes utiles
```bash
docker compose logs -f --tail=100
docker compose down
docker compose down -v   # ⚠️ supprime aussi les volumes (web + db)
```
🔧 Fix DNS (si apt-get update échoue dans le build)

Symptôme : Temporary failure resolving 'deb.debian.org'

✅ Solution : forcer des DNS dans Docker
```bash
sudo nano /etc/docker/daemon.json
{
  "dns": ["8.8.8.8", "1.1.1.1"]
}
sudo systemctl restart docker
docker run --rm alpine ping -c 2 deb.debian.org
```
---

## 👨‍💻 Auteur

Alexandre Kegresse  
Formation Administrateur d’Infrastructures Sécurisées  
La Plateforme – Cannes
