# 🌐 Job Bonus — Docker Network personnalisé

## 🎯 Objectif

Comprendre et manipuler les réseaux Docker :

- Créer un réseau personnalisé
- Connecter plusieurs containers
- Tester la communication entre containers
- Vérifier l’isolation réseau

---

## 🧠 Voir les réseaux existants

```bash
docker network ls
```

Réseaux par défaut :

- bridge
- host
- none

---

## 🌐 Création d’un réseau personnalisé

```bash
docker network create my-network
```

Vérification :

```bash
docker network ls
```

Inspection détaillée :

```bash
docker network inspect my-network
```

---

## 🐳 Lancer deux containers dans le même réseau

```bash
docker run -dit --name container1 --network my-network debian:stable-slim bash
docker run -dit --name container2 --network my-network debian:stable-slim bash
```

---

## 🔎 Test de communication

Entrer dans container1 :

```bash
docker exec -it container1 bash
```

Installer ping :

```bash
apt update
apt install -y iputils-ping
```

Tester la connexion vers container2 :

```bash
ping container2
```

Résultat : ✅ communication fonctionnelle  
Docker fournit une résolution DNS automatique par nom de container.

---

## 🚫 Test d’isolation réseau

Créer un container hors réseau personnalisé :

```bash
docker run -dit --name container3 debian:stable-slim bash
```

Depuis container1 :

```bash
ping container3
```

Résultat : ❌ échec  
Le container3 n’est pas dans le réseau my-network.

---

## 🛑 Nettoyage

```bash
docker rm -f container1 container2 container3
docker network rm my-network
```

---

# 🗄️ Job Bonus — Stack MySQL + phpMyAdmin avec Docker Compose

## 🎯 Objectif

Mettre en place une stack complète avec :

- MySQL 8.0
- phpMyAdmin
- Réseau personnalisé
- Volume pour persistance des données
- Variables d’environnement

Comprendre la communication entre services via Docker Compose.

---

## 📁 Création du dossier projet

```bash
mkdir job10
cd job10
```

---

## 📝 Création du docker-compose.yml

```bash
nano docker-compose.yml
```

Contenu :

```yaml
version: '3.8'

services:

  db:
    image: mysql:8.0
    container_name: mysql_server
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: testdb
      MYSQL_USER: alex
      MYSQL_PASSWORD: alex123
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - my-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin_server
    restart: always
    ports:
      - "8082:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: root123
    networks:
      - my-network

volumes:
  dbdata:

networks:
  my-network:
```

---

## 🚀 Lancement de la stack

```bash
docker compose up -d
```

Vérification :

```bash
docker ps
```

---

## 🌐 Accès à phpMyAdmin

Dans le navigateur :

```
http://IP_DE_LA_VM:8082
```

Connexion possible avec :

Utilisateur root  
Mot de passe : root123  

OU  

Utilisateur : alex  
Mot de passe : alex123  

---

## 🧪 Test

- Vérifier que la base `testdb` existe
- Créer une table
- Insérer des données
- Redémarrer les containers
- Vérifier que les données persistent (grâce au volume)

---

## 🛑 Arrêt et nettoyage

Arrêter la stack :

```bash
docker compose down
```

Supprimer le volume :

```bash
docker volume rm job10_dbdata
```

---

# 🔐 Job Bonus — Docker Compose avec fichier .env (sécurisation des variables)

## 🎯 Objectif

Utiliser un fichier `.env` pour stocker les variables sensibles (mots de passe, utilisateur, base de données) au lieu de les laisser en clair dans le docker-compose.yml.

Mettre en place :

- MySQL 8.0
- phpMyAdmin
- Variables externalisées
- Réseau personnalisé
- Volume pour persistance

---

## 📁 Création du dossier

```bash
mkdir job11
cd job11
```

---

## 🔐 Création du fichier .env

```bash
nano .env
```

Contenu :

```bash
MYSQL_ROOT_PASSWORD=root123
MYSQL_DATABASE=testdb
MYSQL_USER=alex
MYSQL_PASSWORD=alex123
PMA_PORT=8083
```

⚠️ En production, le fichier `.env` ne doit jamais être push sur GitHub.  
Ajouter `.env` dans un fichier `.gitignore`.

---

## 📝 Création du docker-compose.yml

```bash
nano docker-compose.yml
```

Contenu :

```yaml
version: '3.8'

services:

  db:
    image: mysql:8.0
    container_name: mysql_server_job11
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - my-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin_server_job11
    restart: always
    ports:
      - "${PMA_PORT}:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    networks:
      - my-network

volumes:
  dbdata:

networks:
  my-network:
```

---

## 🚀 Lancement de la stack

```bash
docker compose up -d
```

Vérification :

```bash
docker ps
```

---

## 🌐 Accès à phpMyAdmin

Navigateur :

```
http://IP_DE_LA_VM:8083
```

Connexion :

Utilisateur : root  
Mot de passe : root123  

OU  

Utilisateur : alex  
Mot de passe : alex123  

---

## 🧪 Test

- Vérifier que la base `testdb` existe
- Créer une table
- Insérer des données
- Redémarrer les containers
- Vérifier que les données persistent (grâce au volume)

---

## 🛑 Arrêt et nettoyage

```bash
docker compose down
docker volume rm job11_dbdata
```

---

# 🌍 Job Bonus — WordPress + MySQL avec Docker Compose

## 🎯 Objectif

Déployer une application web complète avec :

- MySQL 8.0
- WordPress
- Réseau personnalisé
- Volumes pour persistance
- Communication inter-container

Architecture 2 tiers : Application + Base de données.

---

## 📁 Création du dossier

```bash
mkdir job12
cd job12
```

---

## 📝 Création du docker-compose.yml

```bash
nano docker-compose.yml
```

Contenu :

```yaml
services:

  db:
    image: mysql:8.0
    container_name: wordpress_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: alex
      MYSQL_PASSWORD: alex123
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - wp-network

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    ports:
      - "8084:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: alex
      WORDPRESS_DB_PASSWORD: alex123
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wpdata:/var/www/html
    depends_on:
      - db
    networks:
      - wp-network

volumes:
  dbdata:
  wpdata:

networks:
  wp-network:
```

---

## 🚀 Lancement de la stack

```bash
docker compose up -d
```

Vérification :

```bash
docker ps
```

---

## 🌐 Accès à WordPress

Navigateur :

```
http://IP_DE_LA_VM:8084
```

Suivre l’installation :

- Choisir la langue
- Nom du site
- Créer un utilisateur admin
- Définir un mot de passe

---

## ⚠️ Important (premier démarrage)

Au premier lancement, MySQL peut mettre quelques secondes à s’initialiser.  
Si l’erreur suivante apparaît :

```
Error establishing a database connection
```

Attendre 20 à 60 secondes puis rafraîchir la page.

Vérification possible :

```bash
docker logs wordpress_db --tail 20
```

Attendre le message :

```
ready for connections
```

---

## 🧪 Test de persistance

1. Créer un article
2. Arrêter la stack :

```bash
docker compose down
```

3. Relancer :

```bash
docker compose up -d
```

Le site et les données sont conservés grâce aux volumes.

---

## 🛑 Nettoyage

```bash
docker compose down -v
```

---
