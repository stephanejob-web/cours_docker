# Cours 4 : Gérer les Conteneurs Comme un Pro 🎯

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Exposer des ports pour accéder à vos conteneurs depuis le navigateur
- Utiliser les variables d'environnement
- Donner des noms intelligents à vos conteneurs
- Lancer une vraie application web avec base de données
- Comprendre les modes de réseau Docker

**Durée : 40 minutes (lecture + pratique)**

---

## 🌐 Partie 1 : Les PORTS - Accéder à vos conteneurs

### Le problème : Mon conteneur est invisible !

**Situation :**

Vous lancez un serveur web nginx dans un conteneur :
```bash
docker run -d nginx
```

Le conteneur tourne bien, mais quand vous ouvrez votre navigateur sur `http://localhost`, rien ne s'affiche ! 😱

**Pourquoi ?**

Le conteneur est **isolé**. Il a son propre réseau interne. Votre navigateur ne peut pas y accéder !

C'est comme une maison sans porte. La maison existe, mais vous ne pouvez pas entrer !

### La solution : Mapper les ports

**Le mapping de ports = Créer une porte d'entrée**

**Syntaxe :**
```bash
docker run -p PORT-DE-VOTRE-PC:PORT-DU-CONTENEUR image
```

### Exemple 1 : Nginx accessible sur le port 8080

**Lancer nginx avec le port mappé :**
```bash
docker run -d -p 8080:80 --name mon-nginx nginx
```

**Explication :**
- `-p 8080:80` : Le port 80 du conteneur est accessible sur le port 8080 de votre PC
- `8080` : Le port sur VOTRE ordinateur
- `80` : Le port DANS le conteneur (nginx écoute sur le port 80)

**Schéma :**
```
Votre navigateur → localhost:8080 → [DOCKER] → Conteneur port 80 → Nginx
```

**Testez maintenant !**

Ouvrez votre navigateur et allez sur :
```
http://localhost:8080
```

🎉 **Vous devriez voir la page d'accueil de Nginx !**

### Exemple 2 : Plusieurs conteneurs avec des ports différents

**Lancer 3 serveurs web sur des ports différents :**
```bash
docker run -d -p 8080:80 --name nginx1 nginx
docker run -d -p 8081:80 --name nginx2 nginx
docker run -d -p 8082:80 --name nginx3 nginx
```

**Maintenant vous pouvez accéder à :**
- `http://localhost:8080` → nginx1
- `http://localhost:8081` → nginx2
- `http://localhost:8082` → nginx3

**3 serveurs web différents en même temps !** 🚀

### Les règles importantes des ports

**Règle 1 : Un port de votre PC = un seul conteneur**

❌ **Ça ne marche PAS :**
```bash
docker run -d -p 8080:80 nginx
docker run -d -p 8080:80 nginx  # ERREUR ! Port 8080 déjà utilisé !
```

✅ **Ça marche :**
```bash
docker run -d -p 8080:80 nginx
docker run -d -p 8081:80 nginx  # Port différent, OK !
```

**Règle 2 : Les ports standards des applications**

Chaque application a un port par défaut :
- **Nginx / Apache** : 80 (HTTP) et 443 (HTTPS)
- **MySQL** : 3306
- **PostgreSQL** : 5432
- **Redis** : 6379
- **MongoDB** : 27017
- **Node.js** : Souvent 3000 ou 8000

**Règle 3 : Mapper plusieurs ports**

Vous pouvez mapper plusieurs ports en même temps :
```bash
docker run -d -p 8080:80 -p 8443:443 nginx
```

### Voir les ports mappés

**Commande :**
```bash
docker ps
```

**Vous verrez :**
```
CONTAINER ID   IMAGE   PORTS                  NAMES
a1b2c3d4e5f6   nginx   0.0.0.0:8080->80/tcp   mon-nginx
```

**Lecture :**
- `0.0.0.0:8080` : Accessible depuis n'importe quelle interface réseau de votre PC sur le port 8080
- `->80/tcp` : Redirige vers le port 80 du conteneur

---

## 🔧 Partie 2 : Les Variables d'Environnement

### C'est quoi une variable d'environnement ?

**Définition simple :**

Une variable d'environnement = un réglage que vous donnez au conteneur au démarrage.

**Analogie :**
C'est comme dire à quelqu'un :
- "Parle en français" (LANGUAGE=fr)
- "Travaille en mode silencieux" (MODE=quiet)
- "Utilise ce mot de passe" (PASSWORD=secret123)

### Syntaxe

**Pour passer une variable :**
```bash
docker run -e NOM_VARIABLE=valeur image
```

### Exemple 1 : MySQL avec mot de passe

**Lancer MySQL SANS mot de passe (ne marche pas) :**
```bash
docker run -d mysql:8.0
```

**Ce qui se passe :**
Le conteneur démarre puis s'arrête immédiatement. Pourquoi ?

**Regardons les logs :**
```bash
docker logs nom-du-conteneur
```

**Message d'erreur :**
```
error: database is uninitialized and password option is not specified
```

**Traduction :** "Je ne peux pas démarrer sans mot de passe !"

**Solution : Passer le mot de passe en variable d'environnement**
```bash
docker run -d \
  --name ma-base \
  -e MYSQL_ROOT_PASSWORD=monMotDePasse123 \
  mysql:8.0
```

**Explication :**
- `-e` : Option pour passer une variable d'environnement
- `MYSQL_ROOT_PASSWORD` : Le nom de la variable (imposé par l'image MySQL)
- `=monMotDePasse123` : La valeur du mot de passe

**Vérifiez que ça marche :**
```bash
docker ps
```

Le conteneur doit être "Up" ! ✅

### Exemple 2 : Créer une base de données automatiquement

**Lancer MySQL avec plusieurs variables :**
```bash
docker run -d \
  --name ma-base \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=ma_boutique \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin123 \
  mysql:8.0
```

**Ce qui est créé automatiquement :**
- ✅ Mot de passe root : `root123`
- ✅ Une base de données : `ma_boutique`
- ✅ Un utilisateur : `admin` avec le mot de passe `admin123`

**Tout ça en une seule commande !** 🎉

### Exemple 3 : PostgreSQL

**Même principe avec PostgreSQL :**
```bash
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_USER=john \
  -e POSTGRES_DB=blog \
  postgres:15
```

### Comment connaître les variables disponibles ?

**Allez sur Docker Hub !**

1. Allez sur https://hub.docker.com
2. Cherchez l'image (ex: "mysql")
3. Cliquez sur l'image
4. Lisez la documentation

**Vous trouverez TOUTES les variables possibles dans la section "Environment Variables"**

### Passer plusieurs variables

**Méthode 1 : Plusieurs fois -e**
```bash
docker run -d \
  -e VAR1=valeur1 \
  -e VAR2=valeur2 \
  -e VAR3=valeur3 \
  nginx
```

**Méthode 2 : Fichier .env (on verra ça plus tard avec Docker Compose)**

---

## 📛 Partie 3 : Nommer intelligemment vos conteneurs

### Sans nom : Le chaos !

**Exemple :**
```bash
docker run -d nginx
docker run -d nginx
docker run -d nginx
```

**Résultat :**
```bash
docker ps
```

```
NAMES
silly_tesla
angry_darwin
peaceful_einstein
```

Docker donne des noms random ! Impossible de s'y retrouver ! 😵

### Avec des noms : L'ordre !

**Exemple :**
```bash
docker run -d --name site-principal nginx
docker run -d --name site-test nginx
docker run -d --name site-backup nginx
```

**Résultat :**
```bash
docker ps
```

```
NAMES
site-principal
site-test
site-backup
```

Clair et facile à gérer ! 😎

### Les règles pour les noms

✅ **Bon noms :**
- `mon-site-web`
- `base-donnees-production`
- `api-backend`
- `redis-cache`

❌ **Mauvais noms :**
- `test` (trop vague)
- `conteneur1` (pas informatif)
- `Mon Site Web` (pas d'espaces ni majuscules)

**Règles :**
- Utilisez des tirets `-` ou underscores `_`
- Pas d'espaces
- Pas de caractères spéciaux
- Soyez descriptif

### Renommer un conteneur existant

**Commande :**
```bash
docker rename ancien-nom nouveau-nom
```

**Exemple :**
```bash
docker rename silly_tesla mon-nginx
```

---

## 🔗 Partie 4 : Faire communiquer des conteneurs

### Le problème

Vous avez :
- Un conteneur avec votre application web (Node.js)
- Un conteneur avec votre base de données (MySQL)

**Question :** Comment l'application web peut-elle parler à la base de données ?

### Solution 1 : Le réseau Docker par défaut (simple)

**Quand vous créez des conteneurs, Docker les met dans un réseau par défaut.**

**Les conteneurs peuvent communiquer en utilisant leurs NOMS.**

**Exemple pratique : WordPress + MySQL**

**Étape 1 : Lancer MySQL**
```bash
docker run -d \
  --name ma-base-mysql \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wpuser \
  -e MYSQL_PASSWORD=wppass \
  mysql:8.0
```

**Étape 2 : Lancer WordPress**
```bash
docker run -d \
  --name mon-wordpress \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=ma-base-mysql \
  -e WORDPRESS_DB_USER=wpuser \
  -e WORDPRESS_DB_PASSWORD=wppass \
  -e WORDPRESS_DB_NAME=wordpress \
  wordpress
```

**Explication de la magie :**
- `WORDPRESS_DB_HOST=ma-base-mysql` : WordPress va chercher la base de données au nom "ma-base-mysql"
- Docker fait le lien automatiquement !

**Testez :**

Ouvrez votre navigateur sur `http://localhost:8080`

🎉 **Vous devriez voir l'installation de WordPress !**

### Solution 2 : Créer un réseau personnalisé (mieux)

**Pourquoi ?**
- Plus de contrôle
- Meilleure isolation
- Plus professionnel

**Créer un réseau :**
```bash
docker network create mon-reseau
```

**Lancer les conteneurs dans ce réseau :**
```bash
# Base de données
docker run -d \
  --name ma-base \
  --network mon-reseau \
  -e MYSQL_ROOT_PASSWORD=root123 \
  mysql:8.0

# Application
docker run -d \
  --name mon-app \
  --network mon-reseau \
  -p 8080:80 \
  mon-image-app
```

**Les conteneurs dans le même réseau peuvent communiquer par leurs noms !**

### Voir les réseaux

**Lister les réseaux :**
```bash
docker network ls
```

**Voir les détails d'un réseau :**
```bash
docker network inspect mon-reseau
```

**Supprimer un réseau :**
```bash
docker network rm mon-reseau
```

---

## 💼 Partie 5 : Projet Pratique - Application Web Complète

### Mission : Lancer un site WordPress complet

**Vous allez créer :**
- Un conteneur MySQL pour la base de données
- Un conteneur WordPress pour le site
- Les faire communiquer ensemble
- Accéder au site depuis votre navigateur

### Étape par étape

**Étape 1 : Créer un réseau personnalisé**
```bash
docker network create reseau-wordpress
```

**Pourquoi ?** Pour isoler notre projet et permettre la communication.

---

**Étape 2 : Lancer MySQL**
```bash
docker run -d \
  --name wp-database \
  --network reseau-wordpress \
  -e MYSQL_ROOT_PASSWORD=root_secret \
  -e MYSQL_DATABASE=wordpress_db \
  -e MYSQL_USER=wp_user \
  -e MYSQL_PASSWORD=wp_password \
  mysql:8.0
```

**Vérifiez que ça tourne :**
```bash
docker ps
```

Vous devez voir `wp-database` avec le statut "Up".

**Regardez les logs pour être sûr :**
```bash
docker logs wp-database
```

Vous devriez voir : "ready for connections" ✅

---

**Étape 3 : Lancer WordPress**
```bash
docker run -d \
  --name wp-site \
  --network reseau-wordpress \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=wp-database \
  -e WORDPRESS_DB_USER=wp_user \
  -e WORDPRESS_DB_PASSWORD=wp_password \
  -e WORDPRESS_DB_NAME=wordpress_db \
  wordpress:latest
```

**Explication ligne par ligne :**
- `--name wp-site` : Notre conteneur s'appelle "wp-site"
- `--network reseau-wordpress` : Même réseau que MySQL
- `-p 8080:80` : Accessible sur localhost:8080
- `-e WORDPRESS_DB_HOST=wp-database` : WordPress parle à MySQL via son nom
- Les autres `-e` : Informations de connexion à la base

**Vérifiez :**
```bash
docker ps
```

Vous devez voir `wp-site` et `wp-database` tous les deux "Up" ! ✅

---

**Étape 4 : Accéder à WordPress**

Ouvrez votre navigateur et allez sur :
```
http://localhost:8080
```

🎉 **Vous devriez voir la page d'installation de WordPress !**

**Remplissez le formulaire :**
- Titre du site : Mon Super Blog
- Nom d'utilisateur : admin
- Mot de passe : (choisissez-en un fort)
- Email : votre@email.com

**Cliquez sur "Installer WordPress"**

🎊 **BRAVO ! Vous avez un WordPress complet qui tourne dans Docker !**

---

**Étape 5 : Tester que tout marche**

**Connectez-vous à WordPress :**
```
http://localhost:8080/wp-admin
```

**Créez un article, changez le thème, installez un plugin...**

**Tout fonctionne comme un vrai site WordPress !**

---

**Étape 6 : Arrêter et redémarrer**

**Arrêter tout :**
```bash
docker stop wp-site wp-database
```

**Redémarrer tout :**
```bash
docker start wp-database
docker start wp-site
```

**Attendez 10 secondes, puis :**
```
http://localhost:8080
```

✅ **Votre site est de retour ! Les données sont conservées !**

---

**Étape 7 : Nettoyer (quand vous avez fini)**

**Arrêter les conteneurs :**
```bash
docker stop wp-site wp-database
```

**Supprimer les conteneurs :**
```bash
docker rm wp-site wp-database
```

**Supprimer le réseau :**
```bash
docker network rm reseau-wordpress
```

**Supprimer les images (optionnel) :**
```bash
docker rmi wordpress mysql
```

---

## 🎓 Partie 6 : Résumé des patterns courants

### Pattern 1 : Serveur web simple

**Nginx pour servir un site statique :**
```bash
docker run -d \
  --name mon-site \
  -p 80:80 \
  nginx
```

**Accès :** `http://localhost`

---

### Pattern 2 : Base de données

**MySQL :**
```bash
docker run -d \
  --name ma-base \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
```

**PostgreSQL :**
```bash
docker run -d \
  --name ma-base \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=secret \
  postgres:15
```

**Connexion depuis votre PC :**
- Host : `localhost`
- Port : `3306` (MySQL) ou `5432` (PostgreSQL)
- Mot de passe : celui que vous avez défini

---

### Pattern 3 : Application + Base de données

```bash
# 1. Créer le réseau
docker network create app-network

# 2. Base de données
docker run -d \
  --name database \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. Application
docker run -d \
  --name app \
  --network app-network \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://postgres:secret@database:5432 \
  mon-app
```

---

### Pattern 4 : Application avec plusieurs services

```bash
# Réseau
docker network create projet-network

# Base de données
docker run -d --name db --network projet-network -e POSTGRES_PASSWORD=secret postgres

# Cache Redis
docker run -d --name cache --network projet-network redis

# Application backend
docker run -d --name api --network projet-network -p 4000:4000 mon-api

# Application frontend
docker run -d --name front --network projet-network -p 3000:3000 mon-front
```

---

## 📝 Partie 7 : Exercice complet (À FAIRE)

### Exercice : Créer votre propre application web avec base de données

**Mission : Lancer une application avec :**
- Une base de données PostgreSQL
- Un outil d'administration de base de données (Adminer)
- Les faire communiquer

**Résultat attendu :** Pouvoir gérer votre base PostgreSQL depuis votre navigateur !

### Instructions pas à pas

**1. Créer un réseau**
```bash
docker network create reseau-db
```

---

**2. Lancer PostgreSQL**
```bash
docker run -d \
  --name postgres \
  --network reseau-db \
  -e POSTGRES_PASSWORD=monsuperpass \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=ma_base \
  postgres:15
```

---

**3. Lancer Adminer (interface web pour gérer la base)**
```bash
docker run -d \
  --name adminer \
  --network reseau-db \
  -p 8080:8080 \
  adminer
```

---

**4. Accéder à Adminer**

Ouvrez votre navigateur :
```
http://localhost:8080
```

**Remplissez le formulaire de connexion :**
- Système : PostgreSQL
- Serveur : `postgres` (le nom du conteneur !)
- Utilisateur : `admin`
- Mot de passe : `monsuperpass`
- Base de données : `ma_base`

**Cliquez sur "Connexion"**

---

**5. Créer une table**

Une fois connecté :
- Cliquez sur "Commande SQL"
- Tapez :
```sql
CREATE TABLE utilisateurs (
  id SERIAL PRIMARY KEY,
  nom VARCHAR(50),
  email VARCHAR(100)
);

INSERT INTO utilisateurs (nom, email) VALUES 
('Alice', 'alice@email.com'),
('Bob', 'bob@email.com'),
('Charlie', 'charlie@email.com');
```
- Cliquez sur "Exécuter"

---

**6. Voir vos données**

- Cliquez sur "ma_base" (à gauche)
- Cliquez sur "utilisateurs"
- Vous voyez vos 3 utilisateurs ! ✅

---

**7. Nettoyer**
```bash
docker stop adminer postgres
docker rm adminer postgres
docker network rm reseau-db
```

---

✅ **Si vous avez réussi, BRAVO ! Vous savez maintenant :**
- Créer des réseaux
- Faire communiquer des conteneurs
- Utiliser des variables d'environnement
- Mapper des ports
- Gérer une vraie application multi-conteneurs !

---

## ✅ Quiz de validation

**Question 1 : Pour rendre un conteneur accessible sur le port 3000 de votre PC, que faites-vous ?**

- A) `docker run -d -port 3000:80 nginx`
- B) `docker run -d -p 3000:80 nginx`
- C) `docker run -d --port=3000 nginx`
- D) `docker run -d 3000:80 nginx`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : `docker run -d -p 3000:80 nginx`**

L'option est `-p` (comme "port"). Format : `-p PORT_PC:PORT_CONTENEUR`

</details>

---

**Question 2 : Comment passer un mot de passe à MySQL ?**

- A) `docker run -d --password=secret mysql`
- B) `docker run -d -e PASSWORD=secret mysql`
- C) `docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql`
- D) `docker run -d -p PASSWORD=secret mysql`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : `docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql`**

On utilise `-e` pour les variables d'environnement. Le nom exact `MYSQL_ROOT_PASSWORD` est imposé par l'image MySQL.

</details>

---

**Question 3 : Comment faire communiquer 2 conteneurs ?**

- A) Ils communiquent automatiquement, toujours
- B) En les mettant dans le même réseau
- C) Avec l'option --connect
- D) Ce n'est pas possible

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : En les mettant dans le même réseau**

Utilisez `--network` pour mettre les conteneurs dans le même réseau. Ils pourront communiquer via leurs noms.

</details>

---

**Question 4 : Pourquoi donner un nom à vos conteneurs avec --name ?**

- A) C'est obligatoire
- B) Pour les retrouver et les gérer facilement
- C) Pour qu'ils aillent plus vite
- D) Pour économiser de la mémoire

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Pour les retrouver et les gérer facilement**

Sans nom, Docker donne des noms random (silly_tesla, angry_darwin...). Avec `--name`, vous choisissez un nom clair.

</details>

---

**Question 5 : Pour voir les variables d'environnement disponibles d'une image, où allez-vous ?**

- A) Dans le terminal avec docker info
- B) Sur Docker Hub, dans la documentation de l'image
- C) Dans les paramètres de Docker
- D) On ne peut pas les voir

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Sur Docker Hub, dans la documentation de l'image**

Chaque image sur Docker Hub a une page avec la documentation listant toutes les variables d'environnement disponibles.

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Les 4 concepts clés de ce cours :

**1️⃣ Mapper les ports avec `-p`**
```bash
docker run -p 8080:80 nginx
# Port 8080 de votre PC → Port 80 du conteneur
```

**2️⃣ Variables d'environnement avec `-e`**
```bash
docker run -e MYSQL_ROOT_PASSWORD=secret mysql
# Passer des réglages au conteneur
```

**3️⃣ Nommer les conteneurs avec `--name`**
```bash
docker run --name mon-site nginx
# Nom clair au lieu de "silly_tesla"
```

**4️⃣ Réseaux avec `--network`**
```bash
docker network create mon-reseau
docker run --network mon-reseau nginx
# Les conteneurs du même réseau communiquent
```

### Le pattern complet (à réutiliser)

**Pour lancer une application complète :**
```bash
# 1. Créer un réseau
docker network create app-network

# 2. Lancer la base de données
docker run -d \
  --name database \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres

# 3. Lancer l'application
docker run -d \
  --name app \
  --network app-network \
  -p 3000:3000 \
  -e DATABASE_HOST=database \
  mon-application
```

---

## 🚀 Et maintenant ?

**FÉLICITATIONS ! Vous êtes maintenant un utilisateur avancé de Docker !** 🎉

### Ce que vous maîtrisez maintenant :

- ✅ Exposer des ports pour accéder aux conteneurs
- ✅ Utiliser des variables d'environnement
- ✅ Nommer intelligemment les conteneurs
- ✅ Créer des réseaux Docker
- ✅ Faire communiquer plusieurs conteneurs
- ✅ Lancer des applications complètes (WordPress, base de données + admin...)

### Dans le prochain cours (Cours 5) :

**Les VOLUMES - Sauvegarder vos données !**

Vous apprendrez :
1. Pourquoi les données disparaissent quand on supprime un conteneur
2. Comment utiliser les volumes pour sauvegarder
3. Partager des fichiers entre votre PC et un conteneur
4. Cas pratique : Base de données avec données persistantes

**C'est un cours CRUCIAL !** Sans volumes, vous perdez toutes vos données ! 😱

---

## 📚 Aide-mémoire ports courants

**Mémorisez ces ports (utilisés tout le temps) :**

| Service | Port par défaut | Commande exemple |
|---------|----------------|------------------|
| HTTP (web) | 80 | `docker run -p 8080:80 nginx` |
| HTTPS (web sécurisé) | 443 | `docker run -p 8443:443 nginx` |
| MySQL | 3306 | `docker run -p 3306:3306 mysql` |
| PostgreSQL | 5432 | `docker run -p 5432:5432 postgres` |
| MongoDB | 27017 | `docker run -p 27017:27017 mongo` |
| Redis | 6379 | `docker run -p 6379:6379 redis` |

---

## 💡 Astuce pro

**Créez-vous un fichier avec vos commandes fréquentes !**

Créez un fichier `mes-commandes-docker.txt` :
```bash
# WordPress complet
docker network create wp-net
docker run -d --name wp-db --network wp-net -e MYSQL_ROOT_PASSWORD=root mysql
docker run -d --name wp --network wp-net -p 8080:80 -e WORDPRESS_DB_HOST=wp-db wordpress

# PostgreSQL + Adminer
docker network create db-net
docker run -d --name postgres --network db-net -e POSTGRES_PASSWORD=pass postgres
docker run -d --name adminer --network db-net -p 8080:8080 adminer

# Nettoyage
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

**Comme ça, vous pouvez copier-coller rapidement !** 📋

---

**🎓 Vous êtes prêt pour le Cours 5 (VOLUMES) !**

Excellent travail ! Vous progressez super bien ! Le prochain cours est très important pour ne jamais perdre vos données. À tout de suite ! 💪🚀
