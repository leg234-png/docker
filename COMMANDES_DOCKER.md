# Commandes Docker - Guide de référence

## Introduction à Docker

### Qu'est-ce que Docker ?

Docker est une plateforme open-source de virtualisation au niveau du système d'exploitation qui permet de créer, déployer et exécuter des applications dans des conteneurs. Contrairement aux machines virtuelles traditionnelles qui virtualisent tout le matériel, Docker virtualise uniquement le système d'exploitation, ce qui le rend beaucoup plus léger et rapide.

**Avantages de Docker :**
- **Portabilité** : Une application fonctionne de la même manière sur n'importe quel système (Windows, Linux, Mac)
- **Isolation** : Chaque application s'exécute dans son propre environnement isolé
- **Efficacité** : Utilisation optimale des ressources système
- **Rapidité** : Démarrage des conteneurs en quelques secondes
- **Cohérence** : L'environnement de développement est identique à celui de production

### Qu'est-ce qu'un conteneur ?

Un **conteneur** est une instance exécutable d'une image Docker. C'est un processus isolé qui contient :
- L'application et ses dépendances
- Les bibliothèques système nécessaires
- Les fichiers de configuration
- Un système de fichiers virtuel

**Caractéristiques d'un conteneur :**
- **Isolé** : Chaque conteneur a son propre espace de noms (processus, réseau, fichiers)
- **Léger** : Partage le noyau du système hôte, pas besoin d'un OS complet
- **Éphémère** : Peut être créé, arrêté et supprimé rapidement
- **Portable** : Fonctionne de la même manière sur n'importe quel système Docker

**Exemple :** Un conteneur peut contenir une application Node.js avec toutes ses dépendances npm, sans avoir besoin d'installer Node.js sur la machine hôte.

### Qu'est-ce qu'une couche (Layer) ?

Une **couche** est une modification apportée à une image Docker. Les images Docker sont construites à partir de couches empilées les unes sur les autres, formant une structure en couches (layered filesystem).

**Comment ça fonctionne :**
- Chaque instruction dans un Dockerfile crée une nouvelle couche
- Les couches sont **en lecture seule** (read-only)
- Une couche **écriture** (writable layer) est ajoutée au-dessus lors de l'exécution d'un conteneur
- Les couches sont partagées entre les images et les conteneurs, ce qui économise l'espace disque

**Exemple de couches :**
```
Couche 1 : Base OS (Ubuntu)
Couche 2 : Installation de Node.js
Couche 3 : Copie des fichiers package.json
Couche 4 : Installation des dépendances npm
Couche 5 : Copie du code source
Couche 6 : Commande de démarrage
```

**Avantages :**
- **Réutilisation** : Les couches communes sont partagées entre plusieurs images
- **Efficacité** : Seules les couches modifiées sont reconstruites
- **Taille réduite** : Économie d'espace disque grâce au partage

### Docker Engine

**Docker Engine** est le cœur de Docker, le runtime qui permet de créer et gérer les conteneurs. Il se compose de trois composants principaux :

1. **Docker Daemon (dockerd)** : Le processus en arrière-plan qui gère les conteneurs, images, réseaux et volumes
2. **Docker Client (docker)** : L'interface en ligne de commande qui communique avec le daemon
3. **REST API** : L'API qui permet aux applications de communiquer avec le daemon

**Fonctionnement :**
```
Docker Client → REST API → Docker Daemon → Conteneurs
```

### Docker Client

Le **Docker Client** est l'interface en ligne de commande (CLI) que vous utilisez pour interagir avec Docker. Quand vous tapez `docker run`, `docker build`, etc., vous utilisez le client Docker.

Le client envoie les commandes au Docker Daemon via l'API REST.

### Docker Daemon

Le **Docker Daemon** (dockerd) est le processus serveur qui :
- Écoute les requêtes du client Docker
- Gère les images, conteneurs, réseaux et volumes
- Communique avec le noyau Linux pour créer et gérer les conteneurs

Il s'exécute en arrière-plan sur votre machine.

### Docker Image

Une **image Docker** est un modèle en lecture seule utilisé pour créer des conteneurs. C'est comme un "moule" qui contient :
- Un système d'exploitation minimal (souvent Linux)
- L'application et ses dépendances
- Les fichiers de configuration
- Les métadonnées nécessaires

**Caractéristiques :**
- **Immuable** : Une fois créée, une image ne change pas
- **Empilable** : Construite à partir de couches
- **Partageable** : Peut être poussée vers un registry et réutilisée

**Exemple :** L'image `nginx:latest` contient le serveur web Nginx avec toutes ses dépendances.

### Dockerfile

Un **Dockerfile** est un fichier texte contenant des instructions pour construire une image Docker. Chaque instruction crée une nouvelle couche dans l'image.

**Exemple de Dockerfile :**
```dockerfile
FROM node:16              # Image de base
WORKDIR /app              # Définir le répertoire de travail
COPY package.json .        # Copier les fichiers
RUN npm install           # Installer les dépendances
COPY . .                  # Copier le code source
EXPOSE 3000               # Exposer le port
CMD ["node", "app.js"]    # Commande de démarrage
```

**Instructions courantes :**
- `FROM` : Image de base
- `RUN` : Exécuter une commande
- `COPY` / `ADD` : Copier des fichiers
- `WORKDIR` : Définir le répertoire de travail
- `EXPOSE` : Documenter les ports
- `CMD` / `ENTRYPOINT` : Commande de démarrage
- `ENV` : Variables d'environnement

### Docker Hub

**Docker Hub** est le registry public officiel de Docker, une sorte de "GitHub pour les images Docker". C'est un service cloud qui permet de :
- **Télécharger** des images pré-construites (pull)
- **Partager** vos propres images (push)
- **Rechercher** des images disponibles
- **Gérer** vos images publiques et privées

**Exemples d'images populaires sur Docker Hub :**
- `nginx` : Serveur web
- `mysql` : Base de données MySQL
- `node` : Runtime Node.js
- `python` : Runtime Python
- `redis` : Base de données Redis

**URL :** https://hub.docker.com

### Docker Registry

Un **Docker Registry** est un service qui stocke et distribue des images Docker. Docker Hub est le registry public par défaut, mais vous pouvez utiliser :
- **Docker Hub** (public/privé)
- **Amazon ECR** (Elastic Container Registry)
- **Google Container Registry**
- **Azure Container Registry**
- **Registry privé** (self-hosted)

Le registry permet de centraliser et partager les images Docker.

### API Docker

L'**API Docker** est une interface REST qui permet aux applications de communiquer avec le Docker Daemon. Elle permet de :
- Créer, démarrer, arrêter des conteneurs
- Gérer les images
- Gérer les réseaux et volumes
- Inspecter l'état du système

**Utilisation :**
- Le Docker Client utilise cette API
- Les outils comme Docker Compose l'utilisent
- Les applications peuvent l'utiliser directement

**Endpoint par défaut :** `unix:///var/run/docker.sock` (Linux) ou `npipe:////./pipe/docker_engine` (Windows)

## Pourquoi déployer une application dans un conteneur ?

### Problèmes résolus par Docker

1. **"Ça marche sur ma machine"**
   - L'environnement de développement est identique à celui de production
   - Plus de problèmes de versions incompatibles

2. **Dépendances complexes**
   - Toutes les dépendances sont encapsulées dans le conteneur
   - Pas de conflits entre différentes versions de bibliothèques

3. **Déploiement simplifié**
   - Un seul artefact (l'image) contient tout
   - Déploiement rapide et reproductible

4. **Scalabilité**
   - Facile de lancer plusieurs instances d'une application
   - Orchestration simplifiée avec Kubernetes, Docker Swarm, etc.

5. **Isolation**
   - Chaque application fonctionne indépendamment
   - Un problème dans une application n'affecte pas les autres

6. **Ressources optimisées**
   - Plus léger qu'une machine virtuelle
   - Meilleure utilisation des ressources serveur

### Comment déployer une application dans un conteneur ?

**Étapes typiques :**

1. **Créer un Dockerfile**
   ```dockerfile
   FROM node:16
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["node", "server.js"]
   ```

2. **Construire l'image**
   ```bash
   docker build -t mon-app .
   ```

3. **Tester localement**
   ```bash
   docker run -p 3000:3000 mon-app
   ```

4. **Pousser vers un registry**
   ```bash
   docker tag mon-app username/mon-app:latest
   docker push username/mon-app:latest
   ```

5. **Déployer en production**
   ```bash
   docker pull username/mon-app:latest
   docker run -d -p 3000:3000 username/mon-app:latest
   ```

## Comment utiliser Docker ?

### Installation

**Windows/Mac :**
- Télécharger Docker Desktop depuis https://www.docker.com/products/docker-desktop
- Installer et démarrer Docker Desktop

**Linux :**
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

### Vérifier l'installation

```bash
docker --version
docker run hello-world
```

### Workflow de base

1. **Télécharger une image** (ou la construire)
   ```bash
   docker pull nginx
   ```

2. **Créer et démarrer un conteneur**
   ```bash
   docker run -d -p 8080:80 --name mon-nginx nginx
   ```

3. **Vérifier que ça fonctionne**
   ```bash
   docker ps
   curl http://localhost:8080
   ```

4. **Voir les logs**
   ```bash
   docker logs mon-nginx
   ```

5. **Arrêter le conteneur**
   ```bash
   docker stop mon-nginx
   ```

6. **Supprimer le conteneur**
   ```bash
   docker rm mon-nginx
   ```

### Bonnes pratiques

- **Utiliser des images officielles** quand c'est possible
- **Créer des images légères** (utiliser des images de base minimales)
- **Ne pas stocker de données** dans les conteneurs (utiliser des volumes)
- **Une application par conteneur** (séparation des responsabilités)
- **Utiliser des tags spécifiques** plutôt que `latest` en production
- **Optimiser les Dockerfiles** (mettre les couches qui changent rarement en premier)

## Commandes de base

### Gestion des conteneurs

#### Démarrer un conteneur
```bash
docker start <container_id>
docker start <container_name>
```

#### Arrêter un conteneur
```bash
docker stop <container_id>
docker stop <container_name>
```

#### Redémarrer un conteneur
```bash
docker restart <container_id>
docker restart <container_name>
```

#### Créer et démarrer un conteneur
```bash
docker run <image_name>
docker run -d <image_name>  # En mode détaché (background)
docker run -it <image_name>  # Mode interactif avec terminal
docker run -p 8080:80 <image_name>  # Mapper le port 8080 (host) vers 80 (conteneur)
docker run --name <container_name> <image_name>  # Donner un nom au conteneur
docker run -v /chemin/host:/chemin/conteneur <image_name>  # Monter un volume
docker run -e VAR=value <image_name>  # Définir une variable d'environnement
```

#### Exécuter une commande dans un conteneur en cours d'exécution
```bash
docker exec -it <container_id> <command>
docker exec -it <container_id> /bin/bash  # Ouvrir un shell bash
docker exec -it <container_id> /bin/sh   # Ouvrir un shell sh
```

#### Supprimer un conteneur
```bash
docker rm <container_id>
docker rm -f <container_id>  # Forcer la suppression même si le conteneur est en cours d'exécution
docker rm $(docker ps -a -q)  # Supprimer tous les conteneurs arrêtés
```

### Gestion des images

#### Lister les images
```bash
docker images
docker image ls
```

#### Télécharger une image
```bash
docker pull <image_name>
docker pull <image_name>:<tag>  # Télécharger une version spécifique
```

#### Construire une image
```bash
docker build .
docker build -t <image_name> .
docker build -t <image_name>:<tag> .
docker build -f <dockerfile_path> .  # Spécifier un Dockerfile
```

#### Supprimer une image
```bash
docker rmi <image_id>
docker rmi <image_name>
docker rmi -f <image_id>  # Forcer la suppression
docker image prune  # Supprimer les images non utilisées
```

#### Taguer une image
```bash
docker tag <image_id> <new_name>:<tag>
docker tag <old_name>:<tag> <new_name>:<tag>
```

### Informations et statut

#### Lister les conteneurs
```bash
docker ps  # Conteneurs en cours d'exécution
docker ps -a  # Tous les conteneurs (y compris arrêtés)
docker ps -l  # Dernier conteneur créé
```

#### Voir les logs
```bash
docker logs <container_id>
docker logs -f <container_id>  # Suivre les logs en temps réel
docker logs --tail 100 <container_id>  # Dernières 100 lignes
docker logs --since 1h <container_id>  # Logs depuis 1 heure
```

#### Inspecter un conteneur ou une image
```bash
docker inspect <container_id>
docker inspect <image_id>
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container_id>  # IP du conteneur
```

#### Voir l'utilisation des ressources
```bash
docker stats  # Tous les conteneurs
docker stats <container_id>  # Un conteneur spécifique
```

#### Voir les processus dans un conteneur
```bash
docker top <container_id>
```

### Réseau

#### Lister les réseaux
```bash
docker network ls
```

#### Créer un réseau
```bash
docker network create <network_name>
docker network create --driver bridge <network_name>
```

#### Connecter un conteneur à un réseau
```bash
docker network connect <network_name> <container_id>
```

#### Déconnecter un conteneur d'un réseau
```bash
docker network disconnect <network_name> <container_id>
```

#### Inspecter un réseau
```bash
docker network inspect <network_name>
```

#### Supprimer un réseau
```bash
docker network rm <network_name>
docker network prune  # Supprimer les réseaux non utilisés
```

### Volumes

#### Lister les volumes
```bash
docker volume ls
```

#### Créer un volume
```bash
docker volume create <volume_name>
```

#### Inspecter un volume
```bash
docker volume inspect <volume_name>
```

#### Supprimer un volume
```bash
docker volume rm <volume_name>
docker volume prune  # Supprimer les volumes non utilisés
```

### Docker Compose

#### Démarrer les services
```bash
docker-compose up
docker-compose up -d  # En mode détaché
docker-compose up --build  # Reconstruire les images avant de démarrer
```

#### Arrêter les services
```bash
docker-compose down
docker-compose down -v  # Supprimer aussi les volumes
docker-compose stop
```

#### Voir les logs
```bash
docker-compose logs
docker-compose logs -f  # Suivre en temps réel
docker-compose logs <service_name>  # Logs d'un service spécifique
```

#### Exécuter une commande dans un service
```bash
docker-compose exec <service_name> <command>
docker-compose exec <service_name> /bin/bash
```

#### Reconstruire les images
```bash
docker-compose build
docker-compose build --no-cache  # Sans utiliser le cache
```

#### Voir le statut des services
```bash
docker-compose ps
```

#### Redémarrer un service
```bash
docker-compose restart <service_name>
```

### Nettoyage

#### Nettoyer le système
```bash
docker system prune  # Supprimer les conteneurs, réseaux et images non utilisés
docker system prune -a  # Supprimer aussi les images non utilisées
docker system prune -a --volumes  # Supprimer aussi les volumes
```

#### Nettoyer les conteneurs
```bash
docker container prune  # Supprimer les conteneurs arrêtés
```

### Commandes avancées

#### Copier des fichiers
```bash
docker cp <container_id>:<chemin_conteneur> <chemin_local>  # Du conteneur vers local
docker cp <chemin_local> <container_id>:<chemin_conteneur>  # Du local vers conteneur
```

#### Commiter un conteneur en image
```bash
docker commit <container_id> <new_image_name>:<tag>
```

#### Sauvegarder une image
```bash
docker save -o <fichier.tar> <image_name>
docker save <image_name> > <fichier.tar>
```

#### Charger une image
```bash
docker load -i <fichier.tar>
docker load < <fichier.tar>
```

#### Exporter un conteneur
```bash
docker export <container_id> > <fichier.tar>
```

#### Importer un conteneur
```bash
docker import <fichier.tar> <image_name>:<tag>
```

### Docker Hub et registries

#### Se connecter à Docker Hub
```bash
docker login
docker login <registry_url>
```

#### Se déconnecter
```bash
docker logout
```

#### Pousser une image
```bash
docker push <image_name>:<tag>
docker push <username>/<image_name>:<tag>
```

#### Rechercher une image
```bash
docker search <image_name>
```

### Commandes utiles pour le développement

#### Exécuter un conteneur temporaire
```bash
docker run --rm -it <image_name> /bin/bash  # --rm supprime le conteneur à l'arrêt
```

#### Monter le répertoire courant
```bash
docker run -v $(pwd):/app <image_name>  # Linux/Mac
docker run -v %cd%:/app <image_name>  # Windows CMD
docker run -v ${PWD}:/app <image_name>  # Windows PowerShell
```

#### Exposer plusieurs ports
```bash
docker run -p 8080:80 -p 3306:3306 <image_name>
```

#### Définir plusieurs variables d'environnement
```bash
docker run -e VAR1=value1 -e VAR2=value2 <image_name>
docker run --env-file .env <image_name>  # Depuis un fichier .env
```

#### Lier des conteneurs (legacy, utiliser les réseaux maintenant)
```bash
docker run --link <container_name>:<alias> <image_name>
```

## Commandes Git pour ce projet

### Configuration du dépôt distant
```bash
git remote add origin https://github.com/leg234-png/docker.git
```

### Autres commandes Git utiles
```bash
git remote -v  # Voir les dépôts distants configurés
git remote remove origin  # Supprimer un dépôt distant
git push -u origin main  # Pousser vers le dépôt distant
git pull origin main  # Récupérer depuis le dépôt distant
```

## Exemples pratiques

### Exemple 1 : Lancer un serveur web Nginx
```bash
docker run -d -p 8080:80 --name mon-nginx nginx
```

### Exemple 2 : Lancer une base de données MySQL
```bash
docker run -d -p 3306:3306 --name ma-mysql -e MYSQL_ROOT_PASSWORD=monpassword mysql:latest
```

### Exemple 3 : Lancer un conteneur Node.js avec volume
```bash
docker run -it -v $(pwd):/app -p 3000:3000 node:latest /bin/bash
```

### Exemple 4 : Construire et lancer une application
```bash
docker build -t mon-app .
docker run -d -p 8080:8080 --name mon-app mon-app
```

### Exemple 5 : Docker Compose avec plusieurs services
```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:80"
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: password
```

```bash
docker-compose up -d
```

## Notes importantes

- Remplacez `<container_id>`, `<image_name>`, etc. par les valeurs réelles
- Utilisez `docker ps` pour voir les IDs et noms des conteneurs
- Utilisez `docker images` pour voir les images disponibles
- Les commandes avec `-f` ou `--force` peuvent être destructives
- Toujours vérifier avant de supprimer des ressources importantes

