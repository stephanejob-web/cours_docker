# Cours 11 : Debug et Troubleshooting Docker 🔧

## 🎯 Objectifs du cours

À la fin de ce cours, vous saurez :
- Diagnostiquer et résoudre les problèmes Docker courants
- Utiliser les commandes de diagnostic avancées
- Optimiser les performances de vos conteneurs
- Débugger vos applications dans Docker
- Éviter les pièges classiques

**Durée estimée : 40 minutes**

---

## 📖 Introduction : Pourquoi ce cours est important

Docker peut parfois sembler mystérieux quand quelque chose ne fonctionne pas. Ce cours vous donne TOUS les outils pour comprendre et résoudre n'importe quel problème ! 💪

---

## 🚨 Problème 1 : "Mon conteneur démarre puis s'arrête immédiatement"

### Symptôme

```bash
docker run -d mon-image
# Le conteneur apparaît puis disparaît instantanément
docker ps
# Rien ne s'affiche !
```

### Diagnostic

```bash
# 1. Voir TOUS les conteneurs (même arrêtés)
docker ps -a

# 2. Regarder les logs du conteneur
docker logs <container_id>

# 3. Voir pourquoi il s'est arrêté
docker inspect <container_id> | grep -A 10 State
```

### Causes fréquentes et solutions

**Cause 1 : Le processus principal se termine**

Un conteneur Docker s'arrête quand son processus principal se termine.

```bash
# ❌ MAUVAIS : cette commande se termine immédiatement
docker run -d ubuntu echo "Hello"

# ✅ BON : processus qui tourne en continu
docker run -d ubuntu sleep infinity
docker run -d nginx  # nginx tourne en continu
```

**Cause 2 : Erreur dans la commande**

```bash
# Voir l'erreur exacte
docker logs <container_id>

# Exemple : erreur de syntaxe dans le script
# Solution : corriger le Dockerfile ou la commande
```

**Cause 3 : Variables d'environnement manquantes**

```bash
# ❌ L'app crash car DATABASE_URL manque
docker run -d mon-app

# ✅ Ajouter les variables nécessaires
docker run -d -e DATABASE_URL=mysql://... mon-app
```

---

## 🚨 Problème 2 : "Impossible de se connecter au conteneur"

### Symptôme

```bash
docker run -d -p 8080:80 nginx
# Mais http://localhost:8080 ne répond pas !
```

### Diagnostic étape par étape

#### 1. Vérifier que le conteneur tourne

```bash
docker ps
# Le conteneur doit apparaître avec STATUS "Up"
```

#### 2. Vérifier les ports

```bash
# Voir les ports mappés
docker ps
# ou plus détaillé :
docker port <container_name>

# Vérifier que le port est bien écouté
docker exec <container_name> netstat -tlnp
```

#### 3. Tester depuis l'intérieur du conteneur

```bash
# Se connecter au conteneur
docker exec -it <container_name> bash

# Tester en local
curl localhost:80
# Si ça marche, le problème est le mapping de port
```

#### 4. Vérifier les logs

```bash
docker logs <container_name>
# Y a-t-il des erreurs au démarrage ?
```

### Solutions courantes

**Solution 1 : Port déjà utilisé sur l'hôte**

```bash
# Vérifier les ports utilisés
sudo lsof -i :8080
# ou
sudo netstat -tlnp | grep 8080

# Utiliser un autre port
docker run -d -p 8081:80 nginx
```

**Solution 2 : Mauvais mapping de port**

```bash
# ❌ MAUVAIS : ports inversés
docker run -d -p 80:8080 mon-app  # app écoute sur 8080

# ✅ BON : format = hôte:conteneur
docker run -d -p 8080:8080 mon-app
```

**Solution 3 : Firewall qui bloque**

```bash
# Sur Ubuntu
sudo ufw status
sudo ufw allow 8080

# Redémarrer Docker peut aider
sudo systemctl restart docker
```

---

## 🚨 Problème 3 : "docker: Error response from daemon: Conflict"

### Symptôme

```bash
docker run --name mon-app nginx
# Error: container name "mon-app" is already in use
```

### Solutions

```bash
# Voir tous les conteneurs (actifs et arrêtés)
docker ps -a

# Option 1 : Supprimer l'ancien conteneur
docker rm mon-app

# Option 2 : Supprimer même s'il tourne
docker rm -f mon-app

# Option 3 : Utiliser un autre nom
docker run --name mon-app-v2 nginx
```

---

## 🚨 Problème 4 : "No space left on device"

### Symptôme

```bash
docker build -t mon-image .
# Error: no space left on device
```

### Diagnostic

```bash
# Voir l'espace disque utilisé par Docker
docker system df

# Résultat exemple :
# TYPE            TOTAL     ACTIVE    SIZE
# Images          42        10        15.2GB
# Containers      25        2         1.3GB
# Local Volumes   8         2         2.5GB
# Build Cache     0         0         0B
```

### Solutions

```bash
# 1. Nettoyer les conteneurs arrêtés
docker container prune

# 2. Nettoyer les images non utilisées
docker image prune

# 3. Nettoyer les volumes non utilisés
docker volume prune

# 4. TOUT NETTOYER (attention !)
docker system prune -a --volumes

# Confirmation requise !
# ATTENTION : supprime TOUT ce qui n'est pas utilisé
```

---

## 🚨 Problème 5 : "Mon application est TRÈS lente dans Docker"

### Diagnostic des performances

```bash
# 1. Voir l'utilisation CPU et RAM en temps réel
docker stats

# 2. Voir les stats d'un conteneur spécifique
docker stats mon-conteneur

# 3. Inspecter les limites de ressources
docker inspect mon-conteneur | grep -i memory
```

### Solutions d'optimisation

#### Limiter les ressources (éviter qu'un conteneur monopolise tout)

```bash
# Limiter la RAM
docker run -d --memory="512m" mon-app

# Limiter le CPU (50% d'un core)
docker run -d --cpus="0.5" mon-app

# Combiner les deux
docker run -d \
  --memory="1g" \
  --cpus="2" \
  --name mon-app-optimise \
  mon-app
```

#### Problème de volumes sur Windows/Mac

```bash
# ❌ LENT : volumes montés depuis Windows/Mac
docker run -v /chemin/local:/app mon-app

# ✅ RAPIDE : utiliser un volume Docker nommé
docker volume create mon-volume
docker run -v mon-volume:/app mon-app
```

---

## 🔍 Commandes de diagnostic avancées

### Inspecter tout le détail d'un conteneur

```bash
# Toutes les infos en JSON
docker inspect mon-conteneur

# Filtrer une info précise
docker inspect mon-conteneur --format='{{.State.Status}}'
docker inspect mon-conteneur --format='{{.NetworkSettings.IPAddress}}'
```

### Voir les processus dans un conteneur

```bash
# Liste des processus
docker top mon-conteneur

# Version détaillée
docker exec mon-conteneur ps aux
```

### Analyser les événements Docker

```bash
# Voir tous les événements en temps réel
docker events

# Filtrer par conteneur
docker events --filter container=mon-conteneur

# Filtrer par type d'événement
docker events --filter event=start
```

### Débugger le réseau

```bash
# Lister les réseaux
docker network ls

# Inspecter un réseau
docker network inspect bridge

# Voir les conteneurs dans un réseau
docker network inspect mon-reseau --format='{{range .Containers}}{{.Name}} {{end}}'
```

---

## 🐛 Techniques de debugging avancées

### Technique 1 : Entrer dans un conteneur qui crash

```bash
# Si le conteneur crash au démarrage, empêcher le crash
docker run -it --entrypoint /bin/bash mon-image

# Maintenant vous êtes dedans, vous pouvez investiguer !
```

### Technique 2 : Copier des fichiers pour analyser

```bash
# Copier UN fichier depuis le conteneur
docker cp mon-conteneur:/var/log/app.log ./app.log

# Copier un dossier complet
docker cp mon-conteneur:/var/log ./logs
```

### Technique 3 : Débugger avec des outils

```bash
# Installer des outils de debug dans le conteneur
docker exec -it mon-conteneur bash
apt-get update
apt-get install -y curl vim net-tools

# Tester depuis l'intérieur
curl http://localhost:8080
netstat -tlnp
```

### Technique 4 : Analyser les logs en détail

```bash
# Voir les 100 dernières lignes
docker logs --tail 100 mon-conteneur

# Suivre en temps réel avec timestamp
docker logs -f --timestamps mon-conteneur

# Filtrer par date
docker logs --since 2024-01-01T10:00:00 mon-conteneur
docker logs --since 1h mon-conteneur  # dernière heure
```

---

## 🚨 Problèmes Docker Compose spécifiques

### Problème : "Service xyz is unhealthy"

```bash
# Voir les détails du healthcheck
docker inspect <container_name> | grep -A 20 Health

# Tester manuellement le healthcheck
docker exec mon-conteneur curl http://localhost/health
```

### Problème : "Cannot connect to service"

```bash
# Les conteneurs doivent utiliser le nom du service, pas localhost
# ❌ MAUVAIS dans app :
# DATABASE_URL=mysql://localhost:3306

# ✅ BON dans app :
# DATABASE_URL=mysql://db:3306  # "db" = nom du service
```

### Problème : "depends_on not working"

```yaml
# ❌ depends_on ne garantit PAS que le service est PRÊT
services:
  app:
    depends_on:
      - db  # Démarre après db, mais db peut ne pas être prête

# ✅ Utiliser un healthcheck
services:
  db:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 5
  
  app:
    depends_on:
      db:
        condition: service_healthy  # Attend que db soit healthy !
```

---

## ⚡ Guide d'optimisation des performances

### 1. Optimiser les images

```dockerfile
# ❌ LENT : image lourde
FROM ubuntu
RUN apt-get update && apt-get install -y python3

# ✅ RAPIDE : image légère
FROM python:3.11-alpine
```

### 2. Utiliser le cache de build intelligemment

```dockerfile
# ❌ INEFFICACE : copie tout avant d'installer
COPY . /app
RUN pip install -r requirements.txt

# ✅ OPTIMISÉ : copie d'abord requirements, installe, puis copie le code
COPY requirements.txt /app/
RUN pip install -r requirements.txt
COPY . /app
# Comme ça, si le code change, on ne réinstalle pas les dépendances !
```

### 3. Multi-stage builds pour réduire la taille

```dockerfile
# Stage 1 : Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2 : Production (image finale plus petite !)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/index.js"]
```

### 4. Nettoyer dans le Dockerfile

```dockerfile
# ❌ Laisse des fichiers temporaires
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get clean

# ✅ Nettoie dans la même couche
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

---

## 🏋️ Exercice Pratique : Troubleshooting Challenge

### Mission
Débugger une application qui ne fonctionne pas !

```bash
# 1. Créer un fichier Dockerfile avec une erreur volontaire
cat > Dockerfile << 'EOF'
FROM python:3.11
WORKDIR /app
COPY . .
CMD ["python", "app.py"]
EOF

# 2. Créer app.py avec une erreur
cat > app.py << 'EOF'
import os
database_url = os.environ['DATABASE_URL']  # Variable manquante !
print(f"Connecting to {database_url}")
EOF

# 3. Build et run
docker build -t debug-exercise .
docker run --name test-debug debug-exercise
```

### Questions à résoudre :
1. Pourquoi le conteneur crash ?
2. Comment voir l'erreur ?
3. Comment la corriger ?

### Solutions :

```bash
# 1. Voir l'erreur
docker logs test-debug
# KeyError: 'DATABASE_URL'

# 2. Corriger en ajoutant la variable
docker run --name test-debug-fixed \
  -e DATABASE_URL=postgresql://localhost/mydb \
  debug-exercise

# Ça marche ! 🎉
```

---

## 📊 Checklist de troubleshooting

Quand quelque chose ne marche pas, suivez cette checklist dans l'ordre :

### ✅ Niveau 1 : Les bases
- [ ] Le conteneur tourne-t-il ? (`docker ps`)
- [ ] Y a-t-il des erreurs dans les logs ? (`docker logs`)
- [ ] Les ports sont-ils correctement mappés ? (`docker port`)
- [ ] L'image est-elle à jour ? (`docker pull`)

### ✅ Niveau 2 : Configuration
- [ ] Les variables d'environnement sont-elles définies ?
- [ ] Les volumes sont-ils montés correctement ?
- [ ] Les permissions des fichiers sont-elles correctes ?
- [ ] Le réseau est-il configuré ? (`docker network inspect`)

### ✅ Niveau 3 : Ressources
- [ ] Y a-t-il assez d'espace disque ? (`docker system df`)
- [ ] Le conteneur a-t-il assez de RAM ? (`docker stats`)
- [ ] Le CPU est-il surchargé ?
- [ ] Y a-t-il des conflits de ports ?

### ✅ Niveau 4 : Avancé
- [ ] Le healthcheck fonctionne-t-il ?
- [ ] Les dépendances entre services sont-elles respectées ?
- [ ] Les DNS résolvent-ils correctement ?
- [ ] Y a-t-il des problèmes de firewall ?

---

## 🎯 Quiz de validation

### Question 1
Un conteneur démarre puis s'arrête immédiatement. Quelle est la PREMIÈRE commande à utiliser ?

**A)** `docker restart`  
**B)** `docker logs`  
**C)** `docker rm`  
**D)** `docker system prune`

<details>
<summary>Voir la réponse</summary>

**Réponse : B) `docker logs`**

Les logs vous diront POURQUOI le conteneur s'est arrêté. C'est toujours la première chose à vérifier !
</details>

### Question 2
Comment libérer de l'espace disque en supprimant TOUT ce qui n'est pas utilisé ?

**A)** `docker rm -a`  
**B)** `docker prune`  
**C)** `docker system prune -a --volumes`  
**D)** `docker clean --all`

<details>
<summary>Voir la réponse</summary>

**Réponse : C) `docker system prune -a --volumes`**

Cette commande supprime :
- Tous les conteneurs arrêtés
- Toutes les images non utilisées
- Tous les volumes non utilisés
- Tout le cache de build

ATTENTION : utilisez avec précaution !
</details>

### Question 3
Votre application ne peut pas se connecter à la base de données. Dans docker-compose, quel est le problème ?

```python
# Dans l'application :
DATABASE_URL = "mysql://localhost:3306/mydb"
```

**A)** Le port est incorrect  
**B)** Le mot de passe manque  
**C)** Il faut utiliser le nom du service, pas "localhost"  
**D)** MySQL n'est pas compatible avec Docker

<details>
<summary>Voir la réponse</summary>

**Réponse : C) Il faut utiliser le nom du service**

Dans Docker Compose, les services communiquent via leurs noms de service :
```python
# ✅ BON :
DATABASE_URL = "mysql://db:3306/mydb"  # "db" = nom du service
```
</details>

---

## 📝 Récapitulatif des commandes essentielles

### Diagnostic

```bash
docker ps -a                    # Tous les conteneurs
docker logs <container>         # Voir les logs
docker logs -f <container>      # Suivre les logs
docker inspect <container>      # Toutes les infos
docker stats                    # Stats en temps réel
docker top <container>          # Processus
docker system df                # Espace disque
```

### Debugging

```bash
docker exec -it <container> bash    # Entrer dans le conteneur
docker cp <container>:/file ./      # Copier un fichier
docker port <container>             # Voir les ports
docker events                       # Événements en temps réel
```

### Nettoyage

```bash
docker container prune          # Conteneurs arrêtés
docker image prune              # Images non utilisées
docker volume prune             # Volumes non utilisés
docker system prune -a          # TOUT nettoyer
```

### Performance

```bash
docker run --memory="1g" ...    # Limiter RAM
docker run --cpus="2" ...       # Limiter CPU
docker stats                    # Monitorer
```

---

## 🚀 Points clés à retenir

1. **Toujours vérifier les logs en premier** (`docker logs`)
2. **Un conteneur s'arrête si son processus principal se termine**
3. **Format des ports : `-p hôte:conteneur`**
4. **Dans docker-compose, utiliser les noms de services, pas localhost**
5. **Nettoyer régulièrement** (`docker system prune`)
6. **Utiliser `docker stats` pour surveiller les performances**
7. **`docker inspect` donne TOUTES les infos**
8. **Les volumes nommés sont plus rapides que les bind mounts sur Win/Mac**

---

## 🎓 Prochaines étapes

Félicitations ! Vous savez maintenant :
- ✅ Diagnostiquer n'importe quel problème Docker
- ✅ Utiliser les commandes avancées
- ✅ Optimiser les performances
- ✅ Débugger efficacement

Dans le **Cours 12 (Projet Final)**, vous allez :
- Créer une application complète de A à Z
- Gérer plusieurs services
- Appliquer toutes les bonnes pratiques
- Déployer en production

**Vous êtes maintenant un expert du troubleshooting Docker !** 🎉

---

## 📚 Ressources complémentaires

- [Docker Troubleshooting Guide](https://docs.docker.com/config/daemon/troubleshoot/)
- [Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Performance Optimization](https://docs.docker.com/config/containers/resource_constraints/)

---

**Prêt pour le projet final ?** 💪
