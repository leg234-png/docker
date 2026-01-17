# Commandes Docker - Guide de référence

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

