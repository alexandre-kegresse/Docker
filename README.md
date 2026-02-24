# Docker
# 🐳 Projet Docker — La Plateforme

## 🎯 Objectif

Mettre en place un environnement Docker complet sous Debian (console uniquement), puis créer des images personnalisées via Dockerfile.

---

# 🖥️ Environnement

- VM Debian 13 (mode console)
- 1 vCPU
- 1 Go RAM
- 8 Go disque
- Installation Docker via dépôt officiel

---

# ✅ Job 01 — Installation Docker (CLI)

## Mise à jour système

```bash
apt update
apt install -y ca-certificates curl gnupg lsb-release

Ajout clé GPG Docker

install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

Ajout dépôt Docker

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

Installation Docker

apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Activation du service

systemctl enable --now docker
systemctl status docker --no-pager
docker --version
```

---

# ✅ Job 02 — Test hello-world
## Test fonctionnement

```bash
docker run hello-world
```

Résultat attendu :
" Hello from Docker!
This message shows that your installation appears to be working correctly. "

Commandes essentielles Docker

```bash
docker ps
docker ps -a
docker images
docker pull debian:stable-slim
docker run -it debian:stable-slim bash
docker stop <container_id>
docker rm <container_id>
docker rmi <image_id>
docker logs <container_id>
docker exec -it <container_id> bash
```

---

# ✅ Job 03 — Dockerfile personnalisé (Hello World)

## 🎯 Objectif

Recréer un conteneur équivalent à hello-world en utilisant une image Debian minimale.

--

Dockerfile

FROM debian:stable-slim

RUN apt-get update \
 && apt-get install -y --no-install-recommends cowsay \
 && ln -sf /usr/games/cowsay /usr/local/bin/cowsay \
 && rm -rf /var/lib/apt/lists/*

CMD ["/bin/sh","-lc","echo 'Hello from my custom Docker container!' && cowsay 'Docker Job 03 - Alexandre'"]

--

Build Image

```bash
docker build --no-cache -t my-hello .
```
Lancement conteneur

```bash
docker run --rm my-hello
```

Résultat :

"Hello from my custom Docker container!
 ______________________
< Docker Job 03 - Alexandre >
 ----------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||"

---

# ✅ Job 04 — Image SSH personnalisée

## 🎯 Objectif

Créer une image Debian avec serveur SSH :

Accès root

Mot de passe : root123

Port SSH différent de 22

Sans utiliser d’image SSH existante

#

Dockerfile
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

Build
```bash
docker build -t my-ssh .
```

Run
```bash
docker run -d --name ssh-test -p 2222:2222 my-ssh
docker ps
```

Test connexion SSH
```bash
ssh -p 2222 root@localhost
```

Mot de passe:
```bash
root123
```

Stop / Remove
```bash
docker stop ssh-test
docker rm ssh-test
```

---

# ✅ Job 05 — Alias Docker dans ~/.bashrc

## Ajout des alias

```bash
nano ~/.bashrc
```
Ajout en fin de fichier :

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

# ✅ Job 06 — Volumes Docker

## 1️⃣ Bind Mount

Création dossier local :

```bash
mkdir ~/volume-test
echo "Bonjour depuis l'hôte Debian" > ~/volume-test/index.html
```
Lancement nginx avec bind mount :
```bash
docker run -d --name nginx-bind -p 8080:80 -v ~/volume-test:/usr/share/nginx/html nginx
```
Test :
```bash
curl http://localhost:8080
```
## 2️⃣ Volume nommé

Création :
```bash
docker volume create myvolume
```
Utilisation :
```bash
docker run -d --name nginx-volume -p 8081:80 -v myvolume:/usr/share/nginx/html nginx
```
Inspection :
```bash
docker volume inspect myvolume
```
## 3️⃣ Partage entre conteneurs

Écriture dans le volume :
```bash
docker run -it --rm -v myvolume:/data debian:stable-slim bash
echo "Fichier écrit depuis un autre conteneur" > /data/test.txt
exit
```
Lecture via nginx :
```bash
curl http://localhost:8081/test.txt
```

# ✅ Job 07 – Docker Compose (Nginx + FTP + Volume partagé)

## 🎯 Objectif

Mettre en place une infrastructure Docker composée de :

- Un serveur **Nginx**
- Un serveur **FTP**
- Un **volume partagé**
- Upload d’un fichier via FTP visible sur Nginx

---

📁 Structure du projet
```bash
mkdir job07
cd job07
```
📝 docker-compose.yml
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
      - PASV_ADDRESS=192.168.X.X   # IP de la VM
      - PASV_MIN_PORT=21100
      - PASV_MAX_PORT=21110
    volumes:
      - webdata:/home/vsftpd/alex
    restart: always

volumes:
  webdata:
```
🚀 Lancement des services
```bash
docker compose up -d
```
Vérification :
```bash
docker ps
```
🌐 Test Nginx

Navigateur :

http://IP_DE_LA_VM:8080
📂 Test FTP (FileZilla)

Hôte : IP_DE_LA_VM

Port : 21

Utilisateur : alex

Mot de passe : alex123

Mode : Passif

🧪 Test final

- Créer un fichier index.html

- Upload via FTP

- Rafraîchir le navigateur

- Le fichier est visible via Nginx

🧠 Notions apprises

- Docker Compose

- Multi-containers

- Volume nommé partagé

- Mode passif FTP

- Orchestration de services

🛠 Commandes utiles

Arrêter les containers :
```bash
docker compose down
```
Voir les logs :
```bash
docker logs ftp_server
docker logs nginx_server
```
✅ Résultat

Infrastructure fonctionnelle permettant :

- Upload de fichiers via FTP

- Hébergement automatique via Nginx

- Partage de données via volume Docker
