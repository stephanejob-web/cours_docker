# 📄 AIDE-MÉMOIRE DOCKER
## Toutes les commandes essentielles sur une page

**Pour :** Étudiants de la formation Docker
**Usage :** À garder ouvert pendant que vous travaillez !

---

## 🎯 LES 4 CONCEPTS ESSENTIELS

### Rappel rapide
```bash
-p 8080:80              # PORTS : Accéder au conteneur
-v $(pwd):/app          # VOLUMES : Monter des fichiers
--network mon-reseau    # RÉSEAUX : Faire communiquer les conteneurs
-e VAR=valeur           # VARIABLES : Configurer le conteneur
```

---

## 🐳 COMMANDES DE BASE

### Informations système
```bash
docker --version                    # Version de Docker
docker info                         # Informations détaillées
docker system df                    # Espace disque utilisé
```

### Gérer les conteneurs
```bash
docker ps                           # Conteneurs qui tournent
docker ps -a                        # TOUS les conteneurs
docker ps -q                        # Uniquement les IDs

docker run IMAGE                    # Créer et lancer
docker run -d IMAGE                 # En arrière-plan (detached)
docker run -it IMAGE                # Mode interactif
docker run --name NOM IMAGE         # Avec un nom

docker start NOM                    # Démarrer (si arrêté)
docker stop NOM                     # Arrêter proprement
docker restart NOM                  # Redémarrer

docker rm NOM                       # Supprimer (si arrêté)
docker rm -f NOM                    # Forcer la suppression
docker rm $(docker ps -aq)          # Supprimer TOUS
```

### Voir les logs et statistiques
```bash
docker logs NOM                     # Voir les logs
docker logs -f NOM                  # Suivre en temps réel
docker logs --tail 50 NOM           # Les 50 dernières lignes

docker stats                        # Statistiques en temps réel
docker stats NOM                    # Stats d'un conteneur précis
```

### Exécuter des commandes dans un conteneur
```bash
docker exec NOM COMMANDE            # Exécuter une commande
docker exec -it NOM bash            # Ouvrir un shell interactif
docker exec -it NOM sh              # Si bash n'existe pas

# Exemples pratiques
docker exec mon-nginx ls /usr/share/nginx/html
docker exec -it ma-db mysql -uroot -p
```

### Inspecter un conteneur
```bash
docker inspect NOM                  # Toutes les infos (JSON)
docker inspect NOM | grep IPAddress # Trouver l'IP
docker inspect NOM | grep -A 10 Mounts  # Voir les volumes
docker port NOM                     # Voir les ports mappés
```

---

## 🖼️ GÉRER LES IMAGES

### Lister et rechercher
```bash
docker images                       # Lister les images locales
docker images -q                    # Uniquement les IDs
docker search nginx                 # Chercher sur Docker Hub
```

### Télécharger et supprimer
```bash
docker pull IMAGE                   # Télécharger
docker pull IMAGE:TAG               # Version spécifique
docker pull nginx:1.25              # Exemple

docker rmi IMAGE                    # Supprimer une image
docker rmi -f IMAGE                 # Forcer
docker rmi $(docker images -q)      # Supprimer TOUTES
```

### Créer ses images
```bash
docker build -t NOM:TAG .           # Builder depuis Dockerfile
docker build -t mon-app:1.0 .       # Exemple
docker build -t mon-app:latest .    # Tag latest

docker tag SOURCE CIBLE             # Renommer/dupliquer
docker tag mon-app:1.0 mon-app:prod
```

---

## 🔌 LES PORTS (-p)

### Syntaxe
```bash
-p PORT_PC:PORT_CONTENEUR
```

### Exemples
```bash
# Site web sur le port 8080
docker run -d -p 8080:80 nginx

# Plusieurs ports
docker run -d -p 8080:80 -p 8443:443 nginx

# Laisser Docker choisir
docker run -d -p 80 nginx           # Port aléatoire

# IP spécifique
docker run -d -p 127.0.0.1:8080:80 nginx
```

### Voir les ports
```bash
docker port NOM                     # Ports du conteneur
docker ps                           # Colonne PORTS
```

---

## 💾 LES VOLUMES (-v)

### Types de volumes

**1. Volume nommé (géré par Docker)**
```bash
docker volume create mon-volume
docker run -d -v mon-volume:/data mysql

# Lister et inspecter
docker volume ls
docker volume inspect mon-volume
docker volume rm mon-volume
```

**2. Bind mount (dossier sur votre PC)**
```bash
# Chemin absolu
docker run -d -v /home/user/projet:/app node

# Chemin relatif avec $(pwd)
docker run -d -v $(pwd):/app node

# ⚠️ IMPORTANT : Vérifiez toujours avec pwd avant !
```

**3. Volume en lecture seule**
```bash
docker run -d -v $(pwd):/app:ro nginx
```

### Gérer les volumes
```bash
docker volume ls                    # Lister
docker volume inspect NOM           # Détails
docker volume rm NOM                # Supprimer
docker volume prune                 # Supprimer tous les volumes inutilisés
```

---

## 🌐 LES RÉSEAUX (--network)

### Créer et gérer
```bash
docker network create NOM           # Créer un réseau
docker network ls                   # Lister
docker network inspect NOM          # Détails
docker network rm NOM               # Supprimer
```

### Utiliser les réseaux
```bash
# Lancer un conteneur dans un réseau
docker run -d --name db --network mon-reseau mysql

# Connecter un conteneur existant
docker network connect mon-reseau mon-conteneur

# Déconnecter
docker network disconnect mon-reseau mon-conteneur
```

### Exemple complet (2 conteneurs qui communiquent)
```bash
# 1. Créer le réseau
docker network create mon-reseau

# 2. Lancer la base de données
docker run -d --name ma-db --network mon-reseau mysql

# 3. Lancer l'application
docker run -d --name mon-app --network mon-reseau php:apache

# Dans le code PHP, connectez-vous avec le nom "ma-db" !
```

---

## 🔧 VARIABLES D'ENVIRONNEMENT (-e)

### Syntaxe
```bash
-e NOM_VARIABLE=valeur
```

### Exemples
```bash
# Une variable
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Plusieurs variables
docker run -d \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=ma_base \
  -e MYSQL_USER=stephane \
  mysql

# Depuis un fichier .env
docker run -d --env-file .env mysql
```

### Fichier .env (exemple)
```bash
# Créer un fichier .env
cat > .env << 'EOF'
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=ma_base
MYSQL_USER=stephane
MYSQL_PASSWORD=password123
EOF

# Utiliser
docker run -d --env-file .env mysql
```

---

## 🎨 COMMANDES COMPOSÉES (tout en un)

### Exemple 1 : Site web PHP complet
```bash
docker run -d \
  --name mon-site \
  --network mon-reseau \
  -p 8080:80 \
  -v $(pwd):/var/www/html \
  -e ENV=development \
  php:8.2-apache
```

### Exemple 2 : Base de données MariaDB
```bash
docker run -d \
  --name ma-db \
  --network mon-reseau \
  -v db-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=ma_base \
  -e MYSQL_USER=user \
  -e MYSQL_PASSWORD=password \
  mariadb:latest
```

---

## 🧹 NETTOYAGE

### Supprimer ce qui ne sert plus
```bash
# Conteneurs arrêtés
docker container prune

# Images non utilisées
docker image prune

# Volumes non utilisés
docker volume prune

# Réseaux non utilisés
docker network prune

# TOUT nettoyer d'un coup ⚠️
docker system prune

# VRAIMENT TOUT (images comprises) ⚠️⚠️
docker system prune -a
```

### Supprimer des éléments spécifiques
```bash
# Arrêter et supprimer un conteneur
docker stop mon-conteneur && docker rm mon-conteneur

# En une seule commande (force)
docker rm -f mon-conteneur

# Supprimer tous les conteneurs arrêtés
docker rm $(docker ps -aq -f status=exited)
```

---

## 🔍 DÉBOGAGE

### Trouver pourquoi un conteneur ne démarre pas
```bash
docker logs NOM                     # Voir les logs
docker logs --tail 100 NOM          # Les 100 dernières lignes
docker inspect NOM                  # Toutes les infos
docker inspect NOM | grep Error     # Chercher les erreurs
```

### Tester la connectivité réseau
```bash
# Entrer dans le conteneur
docker exec -it mon-app bash

# Puis tester
ping ma-db                          # Teste la résolution DNS
curl http://ma-db:3306              # Teste la connexion
```

### Vérifier les ressources
```bash
docker stats                        # CPU, RAM, réseau
docker top NOM                      # Processus dans le conteneur
```

---

## 🐳 DOCKER COMPOSE (si installé)

### Commandes de base
```bash
docker-compose up                   # Lancer tous les services
docker-compose up -d                # En arrière-plan
docker-compose down                 # Tout arrêter et supprimer
docker-compose restart              # Redémarrer tous les services

docker-compose ps                   # Voir les services
docker-compose logs                 # Logs de tous les services
docker-compose logs -f NOM          # Suivre un service précis

docker-compose exec NOM bash        # Ouvrir un shell
docker-compose exec NOM COMMANDE    # Exécuter une commande
```

---

## 📋 WORKFLOW TYPIQUE DE DÉVELOPPEMENT

```bash
# 1. Créer un réseau
docker network create mon-reseau

# 2. Lancer la base de données
docker run -d \
  --name ma-db \
  --network mon-reseau \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql

# 3. Aller dans le dossier du projet
cd mon-projet
pwd  # ⚠️ Vérifier le chemin !

# 4. Lancer l'application
docker run -d \
  --name mon-app \
  --network mon-reseau \
  -p 8080:80 \
  -v $(pwd):/var/www/html \
  php:apache

# 5. Développer
# Modifiez vos fichiers → Sauvegardez → Actualisez le navigateur

# 6. Voir les logs en cas d'erreur
docker logs mon-app

# 7. Arrêter quand terminé
docker stop mon-app ma-db
docker rm mon-app ma-db
docker network rm mon-reseau
```

---

## 🆘 COMMANDES DE SECOURS

### Docker ne répond plus
```bash
sudo systemctl restart docker       # Redémarrer Docker
```

### Problème de permissions
```bash
sudo usermod -aG docker $USER       # Ajouter au groupe docker
# Puis déconnexion/reconnexion
```

### Port déjà utilisé
```bash
docker ps                           # Trouver le coupable
docker stop NOM                     # L'arrêter
# Ou changer de port : -p 8081:80
```

### Espace disque plein
```bash
docker system df                    # Voir l'occupation
docker system prune -a              # Tout nettoyer
```

---

## 💡 ASTUCES PRO

### Alias pratiques (ajoutez dans ~/.bashrc)
```bash
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dex='docker exec -it'
alias dl='docker logs'
alias dcu='docker-compose up'
alias dcd='docker-compose down'
```

### Voir toutes les commandes
```bash
docker --help                       # Aide générale
docker run --help                   # Aide sur run
docker COMMANDE --help              # Aide sur n'importe quelle commande
```

---

## 🎯 LES 10 COMMANDES À RETENIR ABSOLUMENT

```bash
1.  docker ps                       # Que se passe-t-il ?
2.  docker logs NOM                 # Pourquoi ça plante ?
3.  docker run -d --name NOM IMAGE  # Lancer
4.  docker stop NOM                 # Arrêter
5.  docker rm NOM                   # Supprimer
6.  docker exec -it NOM bash        # Entrer dedans
7.  docker images                   # Quelles images j'ai ?
8.  docker pull IMAGE               # Télécharger
9.  docker system prune             # Nettoyer
10. pwd                             # Où suis-je ? (avant -v !)
```

---

## 📚 POUR ALLER PLUS LOIN

**Documentation officielle :**
- https://docs.docker.com/

**Pratiquer en ligne gratuitement :**
- https://labs.play-with-docker.com/

**Référence rapide :**
- https://docs.docker.com/get-started/docker_cheatsheet.pdf

---

**💾 Imprimez cette page et gardez-la à côté de vous !**

**❓ Question ? Relisez les cours 03-08 pour les explications détaillées !**

---

**Version 1.0 - Formation Docker pour débutants**
