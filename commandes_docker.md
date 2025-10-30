# Guide des Commandes Docker

## Table des matières
- [Commandes de base](#commandes-de-base)
- [Gestion des images](#gestion-des-images)
- [Gestion des conteneurs](#gestion-des-conteneurs)
- [Gestion des volumes](#gestion-des-volumes)
- [Gestion des réseaux](#gestion-des-réseaux)
- [Docker Compose](#docker-compose)
- [Nettoyage et maintenance](#nettoyage-et-maintenance)
- [Inspection et logs](#inspection-et-logs)clear

---

## Commandes de base

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker version` | Affiche la version de Docker | `docker version` |
| `docker info` | Affiche les informations système de Docker | `docker info` |
| `docker --help` | Affiche l'aide générale | `docker --help` |
| `docker <commande> --help` | Affiche l'aide pour une commande spécifique | `docker run --help` |

---

## Gestion des images

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker images` | Liste toutes les images locales | `docker images` |
| `docker images -a` | Liste toutes les images (y compris intermédiaires) | `docker images -a` |
| `docker pull <image>` | Télécharge une image depuis Docker Hub | `docker pull nginx` |
| `docker pull <image>:<tag>` | Télécharge une version spécifique | `docker pull nginx:1.21` |
| `docker build -t <nom> .` | Construit une image depuis un Dockerfile | `docker build -t mon-app .` |
| `docker build -t <nom> -f <fichier> .` | Construit avec un Dockerfile spécifique | `docker build -t app -f Dockerfile.prod .` |
| `docker tag <image> <nouveau-nom>` | Renomme une image | `docker tag mon-app:latest mon-app:v1.0` |
| `docker rmi <image>` | Supprime une image | `docker rmi nginx` |
| `docker rmi -f <image>` | Force la suppression d'une image | `docker rmi -f mon-app` |
| `docker rmi $(docker images -q)` | Supprime toutes les images | `docker rmi $(docker images -q)` |
| `docker save -o <fichier.tar> <image>` | Sauvegarde une image dans un fichier | `docker save -o nginx.tar nginx` |
| `docker load -i <fichier.tar>` | Charge une image depuis un fichier | `docker load -i nginx.tar` |
| `docker history <image>` | Affiche l'historique d'une image | `docker history nginx` |
| `docker search <terme>` | Recherche des images sur Docker Hub | `docker search python` |

---

## Gestion des conteneurs

### Création et exécution

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker run <image>` | Crée et démarre un conteneur | `docker run nginx` |
| `docker run -d <image>` | Exécute en arrière-plan (detached) | `docker run -d nginx` |
| `docker run -it <image>` | Mode interactif avec terminal | `docker run -it ubuntu bash` |
| `docker run --name <nom> <image>` | Nomme le conteneur | `docker run --name mon-nginx nginx` |
| `docker run -p <hôte>:<conteneur> <image>` | Map un port | `docker run -p 8080:80 nginx` |
| `docker run -P <image>` | Map tous les ports exposés automatiquement | `docker run -P nginx` |
| `docker run -v <hôte>:<conteneur> <image>` | Monte un volume | `docker run -v /data:/app/data nginx` |
| `docker run -e <VAR>=<valeur> <image>` | Définit une variable d'environnement | `docker run -e ENV=prod nginx` |
| `docker run --rm <image>` | Supprime le conteneur après l'arrêt | `docker run --rm nginx` |
| `docker run --restart=always <image>` | Redémarre automatiquement | `docker run --restart=always nginx` |
| `docker run -w <répertoire> <image>` | Définit le répertoire de travail | `docker run -w /app node npm start` |
| `docker run --network <réseau> <image>` | Connecte à un réseau spécifique | `docker run --network mon-reseau nginx` |
| `docker run --memory <limite> <image>` | Limite la mémoire | `docker run --memory 512m nginx` |
| `docker run --cpus <nombre> <image>` | Limite les CPUs | `docker run --cpus 2 nginx` |
| `docker create <image>` | Crée un conteneur sans le démarrer | `docker create nginx` |

### Gestion de l'état

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker ps` | Liste les conteneurs en cours d'exécution | `docker ps` |
| `docker ps -a` | Liste tous les conteneurs (actifs et arrêtés) | `docker ps -a` |
| `docker ps -q` | Liste uniquement les IDs des conteneurs | `docker ps -q` |
| `docker start <conteneur>` | Démarre un conteneur arrêté | `docker start mon-nginx` |
| `docker stop <conteneur>` | Arrête un conteneur en cours | `docker stop mon-nginx` |
| `docker restart <conteneur>` | Redémarre un conteneur | `docker restart mon-nginx` |
| `docker pause <conteneur>` | Met en pause un conteneur | `docker pause mon-nginx` |
| `docker unpause <conteneur>` | Reprend un conteneur en pause | `docker unpause mon-nginx` |
| `docker kill <conteneur>` | Tue immédiatement un conteneur | `docker kill mon-nginx` |
| `docker rm <conteneur>` | Supprime un conteneur arrêté | `docker rm mon-nginx` |
| `docker rm -f <conteneur>` | Force la suppression d'un conteneur actif | `docker rm -f mon-nginx` |
| `docker rm $(docker ps -aq)` | Supprime tous les conteneurs | `docker rm $(docker ps -aq)` |
| `docker container prune` | Supprime tous les conteneurs arrêtés | `docker container prune` |

### Interaction avec les conteneurs

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker exec -it <conteneur> <commande>` | Exécute une commande dans un conteneur actif | `docker exec -it mon-nginx bash` |
| `docker attach <conteneur>` | Attache au processus principal | `docker attach mon-nginx` |
| `docker cp <conteneur>:<chemin> <hôte>` | Copie des fichiers depuis le conteneur | `docker cp mon-nginx:/app/log.txt ./` |
| `docker cp <hôte> <conteneur>:<chemin>` | Copie des fichiers vers le conteneur | `docker cp ./config.json mon-nginx:/app/` |
| `docker port <conteneur>` | Affiche les ports mappés | `docker port mon-nginx` |
| `docker top <conteneur>` | Affiche les processus d'un conteneur | `docker top mon-nginx` |
| `docker stats` | Affiche les statistiques en temps réel | `docker stats` |
| `docker stats <conteneur>` | Stats d'un conteneur spécifique | `docker stats mon-nginx` |
| `docker diff <conteneur>` | Affiche les modifications du système de fichiers | `docker diff mon-nginx` |
| `docker wait <conteneur>` | Attend qu'un conteneur s'arrête | `docker wait mon-nginx` |
| `docker rename <ancien> <nouveau>` | Renomme un conteneur | `docker rename ancien nouveau` |
| `docker update --memory <limite> <conteneur>` | Met à jour les ressources | `docker update --memory 1g mon-nginx` |

---

## Gestion des volumes

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker volume ls` | Liste tous les volumes | `docker volume ls` |
| `docker volume create <nom>` | Crée un volume | `docker volume create mon-volume` |
| `docker volume inspect <volume>` | Affiche les détails d'un volume | `docker volume inspect mon-volume` |
| `docker volume rm <volume>` | Supprime un volume | `docker volume rm mon-volume` |
| `docker volume prune` | Supprime tous les volumes non utilisés | `docker volume prune` |
| `docker run -v <volume>:<chemin> <image>` | Monte un volume nommé | `docker run -v mon-volume:/data nginx` |
| `docker run --mount source=<vol>,target=<chemin> <image>` | Monte avec syntaxe mount | `docker run --mount source=mon-volume,target=/data nginx` |

---

## Gestion des réseaux

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker network ls` | Liste tous les réseaux | `docker network ls` |
| `docker network create <nom>` | Crée un réseau | `docker network create mon-reseau` |
| `docker network create --driver bridge <nom>` | Crée avec un driver spécifique | `docker network create --driver bridge mon-reseau` |
| `docker network inspect <réseau>` | Affiche les détails d'un réseau | `docker network inspect mon-reseau` |
| `docker network connect <réseau> <conteneur>` | Connecte un conteneur à un réseau | `docker network connect mon-reseau nginx` |
| `docker network disconnect <réseau> <conteneur>` | Déconnecte un conteneur | `docker network disconnect mon-reseau nginx` |
| `docker network rm <réseau>` | Supprime un réseau | `docker network rm mon-reseau` |
| `docker network prune` | Supprime tous les réseaux non utilisés | `docker network prune` |

---

## Docker Compose

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker-compose up` | Crée et démarre les services | `docker-compose up` |
| `docker-compose up -d` | Démarre en arrière-plan | `docker-compose up -d` |
| `docker-compose down` | Arrête et supprime les conteneurs | `docker-compose down` |
| `docker-compose down -v` | Supprime aussi les volumes | `docker-compose down -v` |
| `docker-compose start` | Démarre les services existants | `docker-compose start` |
| `docker-compose stop` | Arrête les services | `docker-compose stop` |
| `docker-compose restart` | Redémarre les services | `docker-compose restart` |
| `docker-compose ps` | Liste les conteneurs du compose | `docker-compose ps` |
| `docker-compose logs` | Affiche les logs | `docker-compose logs` |
| `docker-compose logs -f <service>` | Suit les logs d'un service | `docker-compose logs -f web` |
| `docker-compose exec <service> <cmd>` | Exécute une commande dans un service | `docker-compose exec web bash` |
| `docker-compose build` | Construit les images | `docker-compose build` |
| `docker-compose pull` | Télécharge les images | `docker-compose pull` |
| `docker-compose config` | Valide et affiche la configuration | `docker-compose config` |
| `docker-compose top` | Affiche les processus | `docker-compose top` |
| `docker-compose pause <service>` | Met en pause un service | `docker-compose pause web` |
| `docker-compose unpause <service>` | Reprend un service | `docker-compose unpause web` |

---

## Nettoyage et maintenance

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker system df` | Affiche l'utilisation du disque | `docker system df` |
| `docker system prune` | Supprime les ressources non utilisées | `docker system prune` |
| `docker system prune -a` | Supprime tout (y compris images non utilisées) | `docker system prune -a` |
| `docker system prune --volumes` | Inclut les volumes dans le nettoyage | `docker system prune --volumes` |
| `docker image prune` | Supprime les images non utilisées | `docker image prune` |
| `docker container prune` | Supprime les conteneurs arrêtés | `docker container prune` |
| `docker volume prune` | Supprime les volumes non utilisés | `docker volume prune` |
| `docker network prune` | Supprime les réseaux non utilisés | `docker network prune` |

---

## Inspection et logs

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker logs <conteneur>` | Affiche les logs d'un conteneur | `docker logs mon-nginx` |
| `docker logs -f <conteneur>` | Suit les logs en temps réel | `docker logs -f mon-nginx` |
| `docker logs --tail <n> <conteneur>` | Affiche les n dernières lignes | `docker logs --tail 100 mon-nginx` |
| `docker logs --since <temps> <conteneur>` | Logs depuis un moment | `docker logs --since 1h mon-nginx` |
| `docker logs -t <conteneur>` | Affiche avec timestamps | `docker logs -t mon-nginx` |
| `docker inspect <ressource>` | Affiche tous les détails | `docker inspect mon-nginx` |
| `docker inspect --format '<template>' <res>` | Format personnalisé | `docker inspect --format '{{.State.Status}}' nginx` |
| `docker events` | Affiche les événements en temps réel | `docker events` |
| `docker events --since <temps>` | Événements depuis un moment | `docker events --since 1h` |

---

## Registry et partage

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker login` | Se connecte à Docker Hub | `docker login` |
| `docker login <registry>` | Se connecte à un registry privé | `docker login registry.exemple.com` |
| `docker logout` | Se déconnecte | `docker logout` |
| `docker push <image>` | Pousse une image vers le registry | `docker push mon-user/mon-app:latest` |
| `docker pull <image>` | Télécharge une image | `docker pull nginx:latest` |
| `docker commit <conteneur> <image>` | Crée une image depuis un conteneur | `docker commit mon-nginx mon-nginx-custom` |
| `docker export <conteneur> > <fichier.tar>` | Exporte un conteneur | `docker export mon-nginx > backup.tar` |
| `docker import <fichier.tar>` | Importe un conteneur | `docker import backup.tar` |

---

## Options courantes utiles

### Options de `docker run`

| Option | Description | Exemple |
|--------|-------------|---------|
| `-d, --detach` | Mode arrière-plan | `docker run -d nginx` |
| `-it` | Mode interactif + TTY | `docker run -it ubuntu bash` |
| `--name` | Nom du conteneur | `docker run --name web nginx` |
| `-p, --publish` | Mapping de port | `docker run -p 8080:80 nginx` |
| `-v, --volume` | Montage de volume | `docker run -v /data:/app nginx` |
| `-e, --env` | Variable d'environnement | `docker run -e DB_HOST=localhost nginx` |
| `--rm` | Supprime après arrêt | `docker run --rm nginx` |
| `--restart` | Politique de redémarrage | `docker run --restart=always nginx` |
| `--network` | Réseau à utiliser | `docker run --network host nginx` |
| `--link` | Lie à un autre conteneur | `docker run --link db:database nginx` |
| `--env-file` | Charge les variables depuis un fichier | `docker run --env-file .env nginx` |
| `-w, --workdir` | Répertoire de travail | `docker run -w /app node npm start` |
| `--user` | Utilisateur à utiliser | `docker run --user 1000:1000 nginx` |
| `--memory` | Limite de mémoire | `docker run --memory 512m nginx` |
| `--cpus` | Limite de CPU | `docker run --cpus 2 nginx` |

---

## Astuces et raccourcis

### Combinaisons utiles

```bash
# Arrêter tous les conteneurs
docker stop $(docker ps -q)

# Supprimer tous les conteneurs
docker rm $(docker ps -aq)

# Supprimer toutes les images
docker rmi $(docker images -q)

# Supprimer les images non taguées
docker rmi $(docker images -f "dangling=true" -q)

# Voir les logs des 5 dernières minutes
docker logs --since 5m <conteneur>

# Nettoyer complètement le système
docker system prune -a --volumes -f

# Afficher uniquement les IDs
docker ps -q

# Afficher avec format personnalisé
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Exécuter et supprimer automatiquement
docker run --rm -it ubuntu bash

# Build sans cache
docker build --no-cache -t mon-app .

# Inspecter le réseau d'un conteneur
docker inspect -f '{{.NetworkSettings.IPAddress}}' <conteneur>
```

---

## Codes de sortie

| Code | Signification |
|------|---------------|
| 0 | Succès |
| 1 | Erreur d'application |
| 125 | Erreur Docker daemon |
| 126 | Commande non exécutable |
| 127 | Commande introuvable |
| 130 | Arrêt par Ctrl+C |
| 137 | Kill (SIGKILL) |
| 143 | Arrêt gracieux (SIGTERM) |

---

## Ressources supplémentaires

- Documentation officielle: https://docs.docker.com
- Docker Hub: https://hub.docker.com
- Docker Compose: https://docs.docker.com/compose
