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

**Structure d'un Dockerfile :**
Un Dockerfile suit généralement cette structure :
1. **FROM** : Définir l'image de base
2. **Métadonnées** : LABEL, MAINTAINER
3. **Variables d'environnement** : ENV, ARG
4. **Installation de dépendances** : RUN
5. **Copie de fichiers** : COPY, ADD
6. **Configuration** : WORKDIR, USER, EXPOSE
7. **Point d'entrée** : CMD, ENTRYPOINT

**Exemple de Dockerfile complet :**
```dockerfile
# Étape 1 : Image de base
FROM node:16-alpine

# Étape 2 : Métadonnées
LABEL maintainer="votre-email@example.com"
LABEL version="1.0"

# Étape 3 : Variables d'environnement
ENV NODE_ENV=production
ENV PORT=3000

# Étape 4 : Arguments de build (peuvent être passés lors du build)
ARG BUILD_DATE
ARG VCS_REF

# Étape 5 : Définir le répertoire de travail
WORKDIR /app

# Étape 6 : Copier les fichiers de dépendances
COPY package*.json ./

# Étape 7 : Installer les dépendances
RUN npm ci --only=production

# Étape 8 : Copier le code source
COPY . .

# Étape 9 : Créer un utilisateur non-root (sécurité)
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Étape 10 : Exposer le port
EXPOSE 3000

# Étape 11 : Point d'entrée
CMD ["node", "server.js"]
```

## Toutes les instructions Dockerfile expliquées

### FROM
Définit l'image de base à utiliser. **Doit être la première instruction** (sauf ARG).

**Syntaxe :**
```dockerfile
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]
```

**Exemples :**
```dockerfile
FROM ubuntu:20.04
FROM node:16-alpine AS builder
FROM python:3.9-slim
FROM nginx:latest
```

**Multi-stage builds :**
```dockerfile
FROM node:16 AS build
WORKDIR /app
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

### RUN
Exécute une commande dans une nouvelle couche et commit le résultat.

**Syntaxe :**
```dockerfile
RUN <command>                    # Shell form
RUN ["executable", "param1", "param2"]  # Exec form
```

**Exemples :**
```dockerfile
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN ["/bin/bash", "-c", "echo hello"]
RUN pip install --no-cache-dir -r requirements.txt
```

**Bonnes pratiques :**
- Combiner plusieurs commandes avec `&&` pour réduire les couches
- Utiliser `--no-cache` pour apt-get quand possible
- Nettoyer les caches après installation

### COPY
Copie des fichiers ou répertoires du contexte de build vers l'image.

**Syntaxe :**
```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

**Exemples :**
```dockerfile
COPY package.json .
COPY package*.json ./
COPY . /app
COPY --chown=node:node app.js /app/
COPY ["src", "dest"]
```

**Différences avec ADD :**
- `COPY` est préféré (plus simple et prévisible)
- `ADD` peut télécharger depuis une URL et extraire des archives (moins recommandé)

### ADD
Copie des fichiers, répertoires ou URLs vers l'image. Peut aussi extraire des archives.

**Syntaxe :**
```dockerfile
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

**Exemples :**
```dockerfile
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/
ADD --chown=user:group file.txt /app/
```

**Note :** Préférez `COPY` sauf si vous avez besoin des fonctionnalités spécifiques d'`ADD`.

### WORKDIR
Définit le répertoire de travail pour les instructions suivantes.

**Syntaxe :**
```dockerfile
WORKDIR /path/to/workdir
```

**Exemples :**
```dockerfile
WORKDIR /app
WORKDIR /usr/src/app
```

**Caractéristiques :**
- Crée le répertoire s'il n'existe pas
- Peut être utilisé plusieurs fois (chemin relatif ou absolu)
- Les commandes `RUN`, `CMD`, `ENTRYPOINT` s'exécutent dans ce répertoire

### ENV
Définit une variable d'environnement qui persiste dans l'image et les conteneurs.

**Syntaxe :**
```dockerfile
ENV <key> <value>
ENV <key>=<value> ...
```

**Exemples :**
```dockerfile
ENV NODE_ENV production
ENV PORT=3000
ENV DB_HOST=localhost DB_PORT=5432
ENV PATH=/usr/local/bin:$PATH
```

**Utilisation :**
- Accessible via `$VARIABLE` ou `${VARIABLE}`
- Persiste dans les conteneurs créés à partir de l'image

### ARG
Définit une variable disponible uniquement pendant le build (pas dans le conteneur).

**Syntaxe :**
```dockerfile
ARG <name>[=<default_value>]
```

**Exemples :**
```dockerfile
ARG VERSION=latest
ARG BUILD_DATE
ARG NODE_VERSION=16
```

**Utilisation :**
```bash
docker build --build-arg VERSION=1.0.0 .
```

**Dans le Dockerfile :**
```dockerfile
ARG VERSION
ENV APP_VERSION=$VERSION
```

**Note :** Les ARG doivent être déclarés avant leur première utilisation.

### EXPOSE
Documente les ports sur lesquels le conteneur écoute. **Ne publie pas réellement les ports.**

**Syntaxe :**
```dockerfile
EXPOSE <port> [<port>/<protocol>]
```

**Exemples :**
```dockerfile
EXPOSE 80
EXPOSE 3000
EXPOSE 8080/tcp
EXPOSE 53/udp
EXPOSE 80 443
```

**Note :** Pour publier les ports, utilisez `docker run -p` ou `docker-compose.yml`.

### CMD
Fournit la commande par défaut à exécuter quand un conteneur démarre. **Peut être surchargée** lors du `docker run`.

**Syntaxe :**
```dockerfile
CMD ["executable","param1","param2"]  # Exec form (recommandé)
CMD command param1 param2              # Shell form
CMD ["param1","param2"]                # Utilisé avec ENTRYPOINT
```

**Exemples :**
```dockerfile
CMD ["node", "server.js"]
CMD ["npm", "start"]
CMD node server.js
CMD ["param1", "param2"]  # Avec ENTRYPOINT
```

**Caractéristiques :**
- Une seule instruction `CMD` par Dockerfile (la dernière est utilisée)
- Peut être surchargée : `docker run <image> <autre-commande>`

### ENTRYPOINT
Configure le conteneur pour qu'il s'exécute comme un exécutable. **Ne peut pas être surchargée** facilement.

**Syntaxe :**
```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]  # Exec form (recommandé)
ENTRYPOINT command param1 param2                # Shell form
```

**Exemples :**
```dockerfile
ENTRYPOINT ["nginx", "-g", "daemon off;"]
ENTRYPOINT ["python", "app.py"]
```

**Différence avec CMD :**
- `ENTRYPOINT` : Commande fixe, arguments de `CMD` passés comme paramètres
- `CMD` : Commande par défaut, peut être complètement remplacée

**Utilisation combinée :**
```dockerfile
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["--help"]  # Arguments par défaut
```

### USER
Définit l'utilisateur (et optionnellement le groupe) pour les instructions suivantes.

**Syntaxe :**
```dockerfile
USER <user>[:<group>]
USER <UID>[:<GID>]
```

**Exemples :**
```dockerfile
USER node
USER 1000:1000
USER www-data
```

**Sécurité :**
- Ne pas utiliser root par défaut
- Créer un utilisateur non-privilégié

**Exemple complet :**
```dockerfile
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser
```

### LABEL
Ajoute des métadonnées à l'image sous forme de paires clé-valeur.

**Syntaxe :**
```dockerfile
LABEL <key>=<value> <key>=<value> ...
LABEL <key>=<value>
```

**Exemples :**
```dockerfile
LABEL maintainer="votre-email@example.com"
LABEL version="1.0"
LABEL description="Mon application"
LABEL "com.example.vendor"="ACME"
LABEL com.example.version="1.0" \
      com.example.release-date="2024-01-01"
```

### VOLUME
Crée un point de montage pour des volumes nommés ou externes.

**Syntaxe :**
```dockerfile
VOLUME ["/path"]
VOLUME /path
```

**Exemples :**
```dockerfile
VOLUME ["/data"]
VOLUME /var/lib/mysql
VOLUME ["/data", "/logs"]
```

**Note :** Les données dans les volumes persistent même si le conteneur est supprimé.

### STOPSIGNAL
Définit le signal système à envoyer au conteneur pour l'arrêter.

**Syntaxe :**
```dockerfile
STOPSIGNAL signal
```

**Exemples :**
```dockerfile
STOPSIGNAL SIGTERM
STOPSIGNAL SIGKILL
```

### HEALTHCHECK
Définit une commande pour vérifier la santé du conteneur.

**Syntaxe :**
```dockerfile
HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE
```

**Options :**
- `--interval=DURATION` : Intervalle entre les vérifications (défaut: 30s)
- `--timeout=DURATION` : Timeout pour chaque vérification (défaut: 30s)
- `--start-period=DURATION` : Période de démarrage (défaut: 0s)
- `--retries=N` : Nombre d'échecs consécutifs avant unhealthy (défaut: 3)

**Exemples :**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

HEALTHCHECK CMD pg_isready -U postgres || exit 1
```

### SHELL
Remplace le shell par défaut pour la forme shell des commandes.

**Syntaxe :**
```dockerfile
SHELL ["executable", "parameters"]
```

**Exemples :**
```dockerfile
SHELL ["/bin/bash", "-c"]
SHELL ["powershell", "-command"]
```

### MAINTAINER (déprécié)
Définit l'auteur de l'image. **Utilisez LABEL à la place.**

**Syntaxe :**
```dockerfile
MAINTAINER <name>
```

**Remplacement recommandé :**
```dockerfile
LABEL maintainer="votre-email@example.com"
```

### ONBUILD
Ajoute une instruction à exécuter quand l'image est utilisée comme base pour un autre build.

**Syntaxe :**
```dockerfile
ONBUILD <INSTRUCTION>
```

**Exemples :**
```dockerfile
ONBUILD COPY . /app/src
ONBUILD RUN npm install
```

**Utilisation :** Utile pour créer des images de base réutilisables.

### .dockerignore
Fichier (similaire à .gitignore) qui exclut des fichiers du contexte de build.

**Exemple .dockerignore :**
```
node_modules
npm-debug.log
.git
.gitignore
.env
*.md
.DS_Store
```

**Avantages :**
- Réduit la taille du contexte de build
- Accélère le build
- Évite d'inclure des fichiers sensibles

## Structure complète d'un Dockerfile optimisé

**Exemple pour une application Node.js :**
```dockerfile
# Multi-stage build pour optimiser la taille
FROM node:16-alpine AS builder

# Métadonnées
LABEL maintainer="dev@example.com"
LABEL version="1.0.0"

# Arguments de build
ARG NODE_ENV=production
ARG BUILD_DATE
ARG VCS_REF

# Variables d'environnement
ENV NODE_ENV=${NODE_ENV}
ENV NPM_CONFIG_LOGLEVEL=warn

# Répertoire de travail
WORKDIR /app

# Copier et installer les dépendances
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage de production
FROM node:16-alpine

# Créer un utilisateur non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copier depuis le stage builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .

# Utilisateur non-root
USER nodejs

# Exposer le port
EXPOSE 3000

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Point d'entrée
ENTRYPOINT ["node"]
CMD ["server.js"]
```

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

## Structure complète de docker-compose.yml

**Docker Compose** est un outil pour définir et exécuter des applications Docker multi-conteneurs. Il utilise un fichier YAML (`docker-compose.yml`) pour configurer les services, réseaux et volumes.

### Structure de base d'un docker-compose.yml

**Exemple minimal :**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
```

### Toutes les options de docker-compose.yml expliquées

#### version
Définit la version du format du fichier Compose.

**Syntaxe :**
```yaml
version: '3.8'
```

**Versions courantes :**
- `'3.8'` : Version moderne (recommandée)
- `'3.7'`, `'3.6'`, etc. : Versions antérieures
- `'2'` : Format legacy

**Note :** Depuis Compose v1.27.0+, le champ `version` est optionnel.

#### services
Définit les services (conteneurs) de votre application.

**Syntaxe :**
```yaml
services:
  service-name:
    # Configuration du service
```

**Exemple :**
```yaml
services:
  web:
    build: .
    ports:
      - "3000:3000"
  db:
    image: postgres:13
```

### Options de service - Images

#### image
Spécifie l'image Docker à utiliser.

**Syntaxe :**
```yaml
image: <image>[:<tag>]
```

**Exemples :**
```yaml
image: nginx:latest
image: postgres:13-alpine
image: myregistry.com/myapp:v1.0
```

#### build
Définit comment construire l'image à partir d'un Dockerfile.

**Syntaxe simple :**
```yaml
build: .
build: ./path/to/dockerfile
```

**Syntaxe complète :**
```yaml
build:
  context: .
  dockerfile: Dockerfile
  args:
    - NODE_ENV=production
  target: production
  cache_from:
    - myapp:latest
```

**Exemples :**
```yaml
build:
  context: ./app
  dockerfile: Dockerfile.prod
  args:
    BUILD_DATE: ${BUILD_DATE}
    VERSION: ${VERSION}
```

### Options de service - Ports

#### ports
Mappe les ports du conteneur vers l'hôte.

**Syntaxe :**
```yaml
ports:
  - "<host_port>:<container_port>"
  - "<host_port>:<container_port>/<protocol>"
```

**Exemples :**
```yaml
ports:
  - "8080:80"
  - "3000:3000"
  - "53:53/udp"
  - "127.0.0.1:8080:80"  # Spécifier l'interface
```

**Note :** `ports` publie les ports. Pour exposer sans publier, utilisez `expose`.

#### expose
Expose des ports sans les publier sur l'hôte (pour communication inter-conteneurs).

**Syntaxe :**
```yaml
expose:
  - "3000"
  - "5432"
```

### Options de service - Variables d'environnement

#### environment
Définit les variables d'environnement.

**Syntaxe :**
```yaml
environment:
  - VAR=value
  - VAR2=value2
```

**Ou :**
```yaml
environment:
  VAR: value
  VAR2: value2
```

**Exemples :**
```yaml
environment:
  - NODE_ENV=production
  - DATABASE_URL=postgres://user:pass@db:5432/mydb
  - REDIS_HOST=redis
```

#### env_file
Charge les variables d'environnement depuis un fichier.

**Syntaxe :**
```yaml
env_file:
  - .env
  - .env.production
```

**Exemple :**
```yaml
env_file:
  - .env
  - .env.local
```

### Options de service - Volumes

#### volumes
Monte des volumes ou des bind mounts.

**Syntaxe :**
```yaml
volumes:
  - <host_path>:<container_path>
  - <volume_name>:<container_path>
  - <container_path>  # Volume anonyme
```

**Exemples :**
```yaml
volumes:
  - ./data:/app/data
  - db_data:/var/lib/postgresql/data
  - /app/logs
  - type: bind
    source: ./config
    target: /app/config
```

**Types de volumes :**
- **bind mount** : `./path:/container/path`
- **named volume** : `volume_name:/container/path`
- **anonymous volume** : `/container/path`

**Options avancées :**
```yaml
volumes:
  - type: volume
    source: mydata
    target: /data
    volume:
      nocopy: true
  - type: bind
    source: ./src
    target: /app
    bind:
      propagation: rshared
```

#### volumes (niveau racine)
Définit les volumes nommés partagés.

**Syntaxe :**
```yaml
volumes:
  volume_name:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /path
```

**Exemples :**
```yaml
volumes:
  db_data:
    driver: local
  cache:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
```

### Options de service - Réseau

#### networks
Définit les réseaux auxquels le service se connecte.

**Syntaxe :**
```yaml
networks:
  - network_name
  - default
```

**Exemples :**
```yaml
networks:
  - frontend
  - backend
```

**Avec configuration :**
```yaml
networks:
  frontend:
    aliases:
      - web
      - app
  backend:
    ipv4_address: 172.20.0.10
```

#### networks (niveau racine)
Définit les réseaux personnalisés.

**Syntaxe :**
```yaml
networks:
  network_name:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br0
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

**Exemples :**
```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # Réseau isolé
```

### Options de service - Dépendances

#### depends_on
Définit l'ordre de démarrage des services.

**Syntaxe :**
```yaml
depends_on:
  - service1
  - service2
```

**Exemples :**
```yaml
services:
  web:
    depends_on:
      - db
      - redis
  db:
    image: postgres
```

**Avec conditions :**
```yaml
depends_on:
  db:
    condition: service_healthy
  redis:
    condition: service_started
```

### Options de service - Commandes

#### command
Remplace la commande par défaut du conteneur.

**Syntaxe :**
```yaml
command: ["executable", "param1", "param2"]
command: sh -c "echo hello"
```

**Exemples :**
```yaml
command: ["node", "server.js"]
command: npm start
command: >
  sh -c "npm install && npm start"
```

#### entrypoint
Remplace le point d'entrée par défaut.

**Syntaxe :**
```yaml
entrypoint: ["executable", "param"]
entrypoint: /entrypoint.sh
```

### Options de service - Ressources

#### deploy
Options de déploiement (pour Docker Swarm).

**Syntaxe :**
```yaml
deploy:
  replicas: 3
  resources:
    limits:
      cpus: '0.5'
      memory: 512M
    reservations:
      cpus: '0.25'
      memory: 256M
  restart_policy:
    condition: on-failure
    delay: 5s
    max_attempts: 3
```

**Exemples :**
```yaml
deploy:
  mode: replicated
  replicas: 2
  resources:
    limits:
      memory: 1G
    reservations:
      cpus: '0.5'
```

#### mem_limit / cpus (legacy)
Limites de ressources (format legacy).

**Syntaxe :**
```yaml
mem_limit: 512m
cpus: 0.5
```

### Options de service - Restart

#### restart
Politique de redémarrage du conteneur.

**Syntaxe :**
```yaml
restart: always
restart: unless-stopped
restart: on-failure
restart: no
```

**Exemples :**
```yaml
restart: always  # Toujours redémarrer
restart: on-failure:3  # Redémarrer en cas d'échec, max 3 fois
```

### Options de service - Healthcheck

#### healthcheck
Configure le healthcheck du service.

**Syntaxe :**
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

**Exemples :**
```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Options de service - Autres

#### container_name
Définit un nom personnalisé pour le conteneur.

**Syntaxe :**
```yaml
container_name: my-app
```

#### working_dir
Définit le répertoire de travail.

**Syntaxe :**
```yaml
working_dir: /app
```

#### user
Définit l'utilisateur pour exécuter les commandes.

**Syntaxe :**
```yaml
user: "1000:1000"
user: node
```

#### labels
Ajoute des métadonnées au service.

**Syntaxe :**
```yaml
labels:
  - "com.example.description=Web server"
  - "com.example.version=1.0"
```

**Ou :**
```yaml
labels:
  com.example.description: "Web server"
  com.example.version: "1.0"
```

#### logging
Configure le logging du service.

**Syntaxe :**
```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

**Exemples :**
```yaml
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

#### dns
Définit des serveurs DNS personnalisés.

**Syntaxe :**
```yaml
dns:
  - 8.8.8.8
  - 8.8.4.4
```

#### extra_hosts
Ajoute des entrées au fichier /etc/hosts.

**Syntaxe :**
```yaml
extra_hosts:
  - "somehost:162.242.195.82"
  - "otherhost:50.31.209.229"
```

#### security_opt
Options de sécurité.

**Syntaxe :**
```yaml
security_opt:
  - apparmor:profile_name
  - seccomp:unconfined
```

#### cap_add / cap_drop
Ajoute ou supprime des capabilities Linux.

**Syntaxe :**
```yaml
cap_add:
  - ALL
cap_drop:
  - NET_ADMIN
```

#### devices
Accès aux périphériques de l'hôte.

**Syntaxe :**
```yaml
devices:
  - /dev/ttyUSB0:/dev/ttyUSB0
```

#### tmpfs
Monte un tmpfs dans le conteneur.

**Syntaxe :**
```yaml
tmpfs:
  - /tmp
  - /run:size=100m
```

#### stdin_open / tty
Alloue un pseudo-TTY.

**Syntaxe :**
```yaml
stdin_open: true
tty: true
```

#### privileged
Donne des privilèges étendus au conteneur.

**Syntaxe :**
```yaml
privileged: true
```

### Variables d'environnement et substitution

#### Utilisation de variables
Vous pouvez utiliser des variables d'environnement dans docker-compose.yml.

**Syntaxe :**
```yaml
environment:
  - VAR=${VAR}
  - VAR2=${VAR2:-default_value}
```

**Exemples :**
```yaml
services:
  web:
    image: ${IMAGE_NAME:-nginx}:${IMAGE_TAG:-latest}
    ports:
      - "${PORT:-8080}:80"
    environment:
      - NODE_ENV=${NODE_ENV:-production}
```

#### Fichier .env
Créez un fichier `.env` pour définir les variables :

```env
IMAGE_NAME=myapp
IMAGE_TAG=v1.0
PORT=3000
NODE_ENV=production
DATABASE_URL=postgres://user:pass@db:5432/mydb
```

### Exemple complet de docker-compose.yml

**Application complète avec web, base de données et cache :**
```yaml
version: '3.8'

services:
  # Service web
  web:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
    image: myapp:latest
    container_name: myapp-web
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:password@db:5432/mydb
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    env_file:
      - .env
    volumes:
      - ./src:/app/src
      - app_logs:/app/logs
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - frontend
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    labels:
      - "com.example.description=Web application"
      - "com.example.version=1.0.0"

  # Service base de données
  db:
    image: postgres:13-alpine
    container_name: myapp-db
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432"

  # Service Redis
  redis:
    image: redis:6-alpine
    container_name: myapp-redis
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - backend
    restart: unless-stopped
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # Service Nginx (reverse proxy)
  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - web
    networks:
      - frontend
    restart: always

# Volumes nommés
volumes:
  db_data:
    driver: local
  redis_data:
    driver: local
  app_logs:
    driver: local

# Réseaux
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: false
```

### Bonnes pratiques pour docker-compose.yml

1. **Utiliser des variables d'environnement** pour la configuration
2. **Séparer les services** par responsabilité
3. **Utiliser des healthchecks** pour les dépendances
4. **Définir des restart policies** appropriées
5. **Utiliser des volumes nommés** pour la persistance
6. **Créer des réseaux séparés** pour isoler les services
7. **Utiliser des images spécifiques** (éviter `latest` en production)
8. **Documenter avec des labels**
9. **Utiliser depends_on** pour l'ordre de démarrage
10. **Versionner le fichier** dans Git

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

