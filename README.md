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

---

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
