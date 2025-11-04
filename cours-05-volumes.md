# Cours 5 : Les Volumes - Ne Plus Perdre vos Données ! 💾

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Pourquoi les données disparaissent quand on supprime un conteneur
- C'est quoi un volume Docker
- Les 3 types de volumes et quand les utiliser
- Sauvegarder les données d'une base de données
- Partager des fichiers entre votre PC et un conteneur

**Durée : 35 minutes (lecture + pratique)**

---

## 😱 Partie 1 : Le GROS problème - Les données disparaissent !

### Expérience : Perdre ses données

**Faites ce test (MAINTENANT) :**

**Étape 1 : Lancer MySQL avec des données**
```bash
docker run -d \
  --name test-mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=ma_base \
  mysql:8.0
```

**Attendez que MySQL démarre (environ 20 secondes).** Pendant ce temps, MySQL s'initialise et crée ses fichiers de données.

**Étape 2 : Entrer dans le conteneur et créer une table**
```bash
docker exec -it test-mysql mysql -uroot -ppassword ma_base
```

**Explication détaillée de la commande :**
- `docker exec` : Exécuter une commande dans un conteneur qui tourne déjà
- `-i` : **Interactive** = garder l'entrée standard ouverte (vous pouvez taper des commandes)
- `-t` : **TTY** = allouer un terminal (affichage correct avec curseur, couleurs, etc.)
- `test-mysql` : Le nom du conteneur dans lequel on va entrer
- `mysql` : La commande à exécuter (le client MySQL)
- `-uroot` : User = root (utilisateur MySQL)
- `-ppassword` : Password = password (mot de passe MySQL)
- `ma_base` : Le nom de la base de données à ouvrir

**🤔 POURQUOI `-uroot` et `-ppassword` sont COLLÉS (sans espace) ?**

**C'est une particularité de MySQL/MariaDB !** Deux syntaxes sont possibles :

**Syntaxe 1 : COLLÉ (recommandée dans ce cours)**
```bash
mysql -uroot -ppassword
```
- `-u` suivi **directement** du nom d'utilisateur (pas d'espace)
- `-p` suivi **directement** du mot de passe (pas d'espace)

**Syntaxe 2 : SÉPARÉ (fonctionne aussi)**
```bash
mysql -u root -p password
```
- `-u` puis **espace** puis le nom d'utilisateur
- `-p` puis **espace** puis le mot de passe

**Les DEUX fonctionnent !** On utilise la syntaxe collée dans ce cours car elle est plus courante dans la documentation MySQL.

**⚠️ Attention : Sécurité du mot de passe**

En production, ne JAMAIS mettre le mot de passe dans la ligne de commande ! Utilisez plutôt :
```bash
mysql -uroot -p
```
Sans rien après `-p`, MySQL vous demandera le mot de passe de manière sécurisée (il ne s'affichera pas à l'écran).

**Dans Docker, pourquoi on met le mot de passe dans la commande ?**
- C'est pour l'apprentissage (plus simple)
- Dans un conteneur de test, ce n'est pas grave
- En production, on utiliserait des variables d'environnement ou des secrets

**En résumé :** "Docker, entre dans le conteneur test-mysql de manière interactive, et lance le client MySQL connecté à la base ma_base"

Vous êtes maintenant dans MySQL. Tapez :
```sql
CREATE TABLE clients (
  id INT PRIMARY KEY,
  nom VARCHAR(50)
);

INSERT INTO clients VALUES (1, 'Alice');
INSERT INTO clients VALUES (2, 'Bob');

SELECT * FROM clients;
```

Vous devriez voir Alice et Bob ! ✅

**Sortez de MySQL :**
```sql
exit
```

**Étape 3 : Vérifier que les données sont là**
```bash
docker exec -it test-mysql mysql -uroot -ppassword ma_base -e "SELECT * FROM clients;"
```

**Explication de la commande :**
- `docker exec -it` : Exécuter de manière interactive dans le conteneur
- `test-mysql` : Nom du conteneur
- `mysql -uroot -ppassword ma_base` : Se connecter à MySQL (comme avant)
- `-e "SELECT * FROM clients;"` : **Execute** = exécuter cette requête SQL puis sortir automatiquement

**Différence avec la commande précédente :** Ici on n'entre PAS dans MySQL de manière permanente, on exécute juste UNE commande puis on ressort.

Les données sont là ! Alice et Bob existent ! ✅

**Étape 4 : Supprimer le conteneur**
```bash
docker stop test-mysql
docker rm test-mysql
```

**Étape 5 : Recréer le même conteneur**
```bash
docker run -d \
  --name test-mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=ma_base \
  mysql:8.0
```

**Attendez que MySQL démarre (environ 20 secondes).**

**Étape 6 : Chercher Alice et Bob**
```bash
docker exec -it test-mysql mysql -uroot -ppassword ma_base -e "SELECT * FROM clients;"
```

**Rappel de la commande :**
- `docker exec -it test-mysql` : Entrer dans le conteneur de manière interactive
- `mysql -uroot -ppassword ma_base` : Client MySQL connecté à la base
- `-e "SELECT * FROM clients;"` : Exécuter cette requête et sortir

**Résultat :**
```
ERROR 1146: Table 'ma_base.clients' doesn't exist
```

😱 **LES DONNÉES ONT DISPARU !**

Alice et Bob n'existent plus ! La table n'existe plus !

### Pourquoi ça arrive ?

**Explication simple :**

Quand vous créez un conteneur :
- Docker crée un **système de fichiers temporaire** pour ce conteneur
- Tout ce que vous créez dedans (fichiers, données de base de données...) est stocké dans ce système temporaire
- Quand vous **supprimez le conteneur** → Le système de fichiers temporaire est supprimé aussi !

**C'est comme :**
- Écrire sur un tableau blanc
- Effacer le tableau
- Tout disparaît ! 🧽

### Le problème en entreprise

**Imaginez en production :**

```
Lundi : Vous lancez votre base de données
Mardi : Vos clients créent des comptes
Mercredi : Le serveur redémarre (mise à jour)
Jeudi : PANIQUE ! Tous les comptes ont disparu ! 😱
```

**C'est CATASTROPHIQUE !**

### La solution : Les VOLUMES

**Un volume = Un disque dur externe pour vos conteneurs** 💾

Les données sont stockées **en dehors** du conteneur, donc :
- ✅ Si vous supprimez le conteneur → Les données restent
- ✅ Si vous créez un nouveau conteneur → Les données sont toujours là
- ✅ Plusieurs conteneurs peuvent partager les mêmes données

---

## 📦 Partie 2 : C'est quoi un volume ?

### Définition simple

**Un volume Docker = Un dossier spécial qui survit à la suppression du conteneur**

**Analogie :**
- **Sans volume** : Écrire sur un Post-it collé sur le conteneur → Si vous jetez le conteneur, vous jetez le Post-it
- **Avec volume** : Écrire dans un cahier séparé → Même si vous jetez le conteneur, le cahier reste

### Schéma

```
SANS VOLUME :
┌─────────────────┐
│   CONTENEUR     │
│  ┌───────────┐  │
│  │  Données  │  │ ← À l'intérieur du conteneur
│  └───────────┘  │
└─────────────────┘
     ⬇️ docker rm
    ❌ TOUT DISPARAÎT


AVEC VOLUME :
┌─────────────────┐         ┌──────────────┐
│   CONTENEUR     │ ◄─────► │   VOLUME     │
│                 │         │  (Dossier    │
│                 │         │   externe)   │
└─────────────────┘         └──────────────┘
     ⬇️ docker rm               ⬇️
    ❌ Supprimé               ✅ RESTE !
```

### Les 3 types de volumes

Docker propose 3 façons de gérer les données :

**Type 1 : Volumes nommés** (le plus utilisé) ⭐
- Géré par Docker
- Facile à utiliser
- **RECOMMANDÉ pour les bases de données**

**Type 2 : Bind mounts** (montage de dossiers)
- Vous choisissez le dossier sur votre PC
- Pratique pour le développement
- **RECOMMANDÉ pour partager du code**

**Type 3 : tmpfs** (temporaire en RAM)
- Données en mémoire RAM
- Rapide mais disparaît au redémarrage
- Rarement utilisé

**On va se concentrer sur les 2 premiers : Volumes nommés et Bind mounts**

---

## ⭐ Partie 3 : Les Volumes Nommés (pour les bases de données)

### Créer un volume

**Commande :**
```bash
docker volume create nom-du-volume
```

**Exemple :**
```bash
docker volume create mysql-data
```

**Ce qui se passe :**
Docker crée un espace de stockage permanent sur votre disque dur.

### Voir tous vos volumes

**Commande :**
```bash
docker volume ls
```

**Résultat :**
```
DRIVER    VOLUME NAME
local     mysql-data
```

### Utiliser un volume dans un conteneur

**Syntaxe :**
```bash
docker run -v nom-volume:/chemin/dans/conteneur image
```

**Exemple avec MySQL :**
```bash
docker run -d \
  --name mysql-avec-volume \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=ma_base \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

**Explication :**
- `-v mysql-data:/var/lib/mysql` : Le volume "mysql-data" est monté sur `/var/lib/mysql`
- `/var/lib/mysql` : C'est là que MySQL stocke ses données

### 🤔 MAIS POURQUOI /var/lib/mysql ? Pourquoi PAS ailleurs ?

**Question importante :** Pourquoi exactement `/var/lib/mysql` et pas `/data` ou `/mysql` ou n'importe quoi d'autre ?

**Réponse : Parce que c'est là que MySQL a DÉCIDÉ de stocker ses données !**

#### Explication détaillée :

**1. Chaque application choisit OÙ elle stocke ses données**

Quand les développeurs de MySQL ont créé MySQL, ils ont programmé le logiciel pour qu'il stocke ses fichiers de base de données dans un dossier spécifique : `/var/lib/mysql`

C'est une **convention Linux** :
- `/var/lib/` = dossier pour les données des applications
- `/var/lib/mysql` = données de MySQL
- `/var/lib/postgresql` = données de PostgreSQL
- `/var/lib/mongodb` = données de MongoDB

**2. Que se passe-t-il dans /var/lib/mysql ?**

Dans ce dossier, MySQL crée plein de fichiers :
- Les fichiers `.ibd` (données des tables)
- Les fichiers de logs
- Les fichiers de configuration de la base
- Les métadonnées

**Exemple concret - Regardons dedans :**

Regardons ce qu'il y a dans le conteneur `mysql-avec-volume` qu'on vient de créer !

**Attendez que MySQL démarre (environ 20 secondes).**

```bash
# Voir ce qu'il y a dans /var/lib/mysql
docker exec mysql-avec-volume ls -la /var/lib/mysql
```

**Résultat :**
```
-rw-r----- 1 mysql mysql  196608 Nov  4 10:00 #ib_16384_0.dblwr
-rw-r----- 1 mysql mysql 8585216 Nov  4 10:00 #ib_16384_1.dblwr
drwxr-x--- 2 mysql mysql    4096 Nov  4 10:00 #innodb_redo
drwxr-x--- 2 mysql mysql    4096 Nov  4 10:00 #innodb_temp
-rw-r----- 1 mysql mysql      56 Nov  4 10:00 auto.cnf
-rw-r----- 1 mysql mysql 12582912 Nov  4 10:00 ibdata1
drwxr-x--- 2 mysql mysql    4096 Nov  4 10:00 mysql
-rw-r----- 1 mysql mysql 16777216 Nov  4 10:00 undo_001
-rw-r----- 1 mysql mysql 16777216 Nov  4 10:00 undo_002
```

**Tous ces fichiers = vos données MySQL !**

**Nettoyage :**
```bash
docker stop mysql-avec-volume
docker rm mysql-avec-volume
docker volume rm mysql-data
```

**3. Que se passerait-il si vous mettiez le volume ailleurs ?**

**Scénario 1 : Volume au MAUVAIS endroit**
```bash
# MAUVAIS ❌
docker run -d \
  --name mysql-mauvais-volume \
  -e MYSQL_ROOT_PASSWORD=pass \
  -v mysql-data:/data \
  mysql:8.0
```

**Résultat :**
- Le volume `mysql-data` est monté sur `/data`
- Mais MySQL écrit toujours dans `/var/lib/mysql` (à l'intérieur du conteneur)
- Donc les données NE SONT PAS dans le volume !
- Si vous supprimez le conteneur → **Les données disparaissent quand même !** 😱

**Scénario 2 : Volume au BON endroit**
```bash
# BON ✅
docker run -d \
  --name mysql-bon-volume \
  -e MYSQL_ROOT_PASSWORD=pass \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

**Résultat :**
- Le volume `mysql-data` est monté sur `/var/lib/mysql`
- MySQL écrit dans `/var/lib/mysql`
- Les données SONT dans le volume !
- Si vous supprimez le conteneur → **Les données restent !** ✅

**Ces exemples ne sont pas à taper**, c'est juste pour montrer la différence entre un volume au bon et au mauvais endroit.

#### Résumé : Comment connaître le bon chemin ?

**Pour chaque image Docker, vous devez connaître le chemin où l'application stocke ses données.**

**Comment le trouver ?**

**👉 Méthode simple : Lire la documentation sur Docker Hub**

Allez sur https://hub.docker.com et cherchez votre image.

**Exemples :**

**Pour MySQL :**
1. Allez sur https://hub.docker.com/_/mysql
2. Cherchez la section "Where to Store Data"
3. Vous verrez : `/var/lib/mysql`

**Pour PostgreSQL :**
1. Allez sur https://hub.docker.com/_/postgres
2. Cherchez "PGDATA"
3. Vous verrez : `/var/lib/postgresql/data`

**Pour MongoDB :**
1. Allez sur https://hub.docker.com/_/mongo
2. Cherchez "Where to Store Data"
3. Vous verrez : `/data/db`

**C'est tout !** Pas besoin de commandes compliquées. La documentation vous dit toujours où mettre le volume.

#### Tableau récapitulatif : Les chemins les plus courants

| Application | Chemin du volume | Pourquoi ce chemin ? |
|-------------|------------------|----------------------|
| **MySQL** | `/var/lib/mysql` | Convention Linux + configuration par défaut de MySQL |
| **PostgreSQL** | `/var/lib/postgresql/data` | Convention Linux + PostgreSQL stocke dans un sous-dossier `data` |
| **MongoDB** | `/data/db` | Choix des développeurs MongoDB (pas la convention Linux standard) |
| **Redis** | `/data` | Choix des développeurs Redis (simple et court) |
| **Nginx** (logs) | `/var/log/nginx` | Convention Linux pour les logs |
| **Apache** (site) | `/var/www/html` | Convention Linux pour les sites web |

**Conclusion importante :**
🎯 **Le chemin du volume n'est PAS arbitraire. Il doit correspondre EXACTEMENT au chemin où l'application stocke ses données !**

### Test : Les données survivent !

**Étape 1 : Créer le volume**
```bash
docker volume create mysql-data
```

**Étape 2 : Lancer MySQL avec le volume**
```bash
docker run -d \
  --name mysql-permanent \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=ma_base \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

**Attendez que MySQL démarre (environ 20 secondes).**

**Étape 3 : Créer des données**
```bash
docker exec -it mysql-permanent mysql -uroot -ppassword ma_base
```

**Explication (rappel) :**
- `docker exec` : Exécuter une commande dans un conteneur qui tourne
- `-i` : Mode **i**nteractif (vous pouvez taper)
- `-t` : Allouer un **t**erminal (affichage propre)
- `mysql-permanent` : Nom du conteneur
- `mysql -uroot -ppassword ma_base` : Lancer le client MySQL

Dans MySQL :
```sql
CREATE TABLE produits (
  id INT PRIMARY KEY,
  nom VARCHAR(100)
);

INSERT INTO produits VALUES (1, 'Ordinateur');
INSERT INTO produits VALUES (2, 'Souris');
INSERT INTO produits VALUES (3, 'Clavier');

SELECT * FROM produits;
exit
```

Vous voyez vos 3 produits ! ✅

**Étape 4 : SUPPRIMER le conteneur**
```bash
docker stop mysql-permanent
docker rm mysql-permanent
```

Le conteneur est détruit ! 💥

**Étape 5 : Recréer un conteneur avec le MÊME volume**
```bash
docker run -d \
  --name mysql-nouveau \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=ma_base \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

**Attendez que MySQL démarre (environ 20 secondes).**

**Étape 6 : Vérifier les données**
```bash
docker exec -it mysql-nouveau mysql -uroot -ppassword ma_base -e "SELECT * FROM produits;"
```

**Explication (rappel) :**
- `docker exec -it mysql-nouveau` : Exécuter de manière interactive dans le conteneur
- `mysql -uroot -ppassword ma_base` : Client MySQL
- `-e "SELECT * FROM produits;"` : Exécuter juste cette requête puis sortir

**Résultat :**
```
+----+-------------+
| id | nom         |
+----+-------------+
|  1 | Ordinateur  |
|  2 | Souris      |
|  3 | Clavier     |
+----+-------------+
```

🎉 **LES DONNÉES SONT LÀ !**

Ordinateur, Souris, Clavier... tout est revenu ! MAGIE ! ✨

**Pourquoi ?** Parce que les données sont dans le volume `mysql-data`, pas dans le conteneur !

### Les chemins importants pour chaque base de données

**MySQL / MariaDB :**
```bash
-v mon-volume:/var/lib/mysql
```

**PostgreSQL :**
```bash
-v mon-volume:/var/lib/postgresql/data
```

**MongoDB :**
```bash
-v mon-volume:/data/db
```

**Redis :**
```bash
-v mon-volume:/data
```

**Retenez ces chemins !** 📝

### Gérer les volumes

**Voir les détails d'un volume :**
```bash
docker volume inspect nom-volume
```

**Supprimer un volume :**
```bash
docker volume rm nom-volume
```

⚠️ **ATTENTION :** Supprimer un volume = supprimer les données définitivement !

**Nettoyer les volumes inutilisés :**
```bash
docker volume prune
```

⚠️ **DANGER :** Cette commande supprime tous les volumes non utilisés !

---

## 📂 Partie 4 : Les Bind Mounts (partager des dossiers)

### C'est quoi un Bind Mount ?

**Définition simple :**

Connecter un dossier de votre PC directement dans le conteneur.

**Cas d'usage :**
- Développer un site web et voir les changements en temps réel
- Partager des fichiers de configuration
- Modifier du code sans recréer le conteneur

### Syntaxe

**Format :**
```bash
docker run -v /chemin/sur/votre/pc:/chemin/dans/conteneur image
```

**Important :** Utilisez un **chemin absolu** (complet) !

### Exemple 1 : Partager un site web HTML

**Étape 1 : Créer un dossier sur votre PC**
```bash
mkdir ~/mon-site
cd ~/mon-site
```

**Étape 2 : Créer un fichier HTML**
```bash
echo "<h1>Bonjour Docker !</h1>" > index.html
```

**Étape 3 : Lancer nginx avec ce dossier**
```bash
docker run -d \
  --name mon-serveur \
  -p 8080:80 \
  -v ~/mon-site:/usr/share/nginx/html \
  nginx
```

**Explication :**
- `-v ~/mon-site:/usr/share/nginx/html` : Votre dossier `~/mon-site` est accessible dans le conteneur au chemin `/usr/share/nginx/html`
- Nginx sert les fichiers de ce dossier

**Étape 4 : Tester dans le navigateur**

Ouvrez :
```
http://localhost:8080
```

Vous voyez : **"Bonjour Docker !"** ✅

**Étape 5 : Modifier le fichier (SANS redémarrer le conteneur)**

Sur votre PC, modifiez `~/mon-site/index.html` :
```bash
echo "<h1>Docker c'est génial !</h1><p>Les modifications sont en temps réel !</p>" > ~/mon-site/index.html
```

**Étape 6 : Rafraîchir le navigateur**

Appuyez sur F5 dans le navigateur.

🎉 **Le texte a changé !** Sans redémarrer le conteneur !

**Pourquoi ?** Le conteneur lit directement votre dossier !

### Exemple 2 : Développer un site avec plusieurs pages

**Étape 1 : Créer une structure de site**
```bash
mkdir -p ~/mon-site
cd ~/mon-site
```

**Créer index.html :**
```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Mon Site</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Bienvenue sur mon site !</h1>
    <p>Ceci est la page d'accueil.</p>
    <a href="contact.html">Contactez-moi</a>
</body>
</html>
EOF
```

**Créer contact.html :**
```bash
cat > contact.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Contact</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Me contacter</h1>
    <p>Email : contact@monsite.com</p>
    <a href="index.html">Retour</a>
</body>
</html>
EOF
```

**Créer style.css :**
```bash
cat > style.css << 'EOF'
body {
    font-family: Arial, sans-serif;
    margin: 50px;
    background-color: #f0f0f0;
}
h1 {
    color: #2196F3;
}
a {
    color: #4CAF50;
    text-decoration: none;
    font-weight: bold;
}
EOF
```

**Étape 2 : Lancer le serveur**
```bash
docker run -d \
  --name mon-site-complet \
  -p 8080:80 \
  -v ~/mon-site:/usr/share/nginx/html \
  nginx
```

**Étape 3 : Tester**

Ouvrez `http://localhost:8080`

Vous avez un site complet avec plusieurs pages ! 🎨

**Modifiez le CSS, ajoutez des pages... Tout en temps réel !**

### Exemple 3 : Exercice pratique avec Ubuntu - Comprendre les volumes en profondeur

**Cet exercice va vous montrer EXACTEMENT comment fonctionnent les volumes !**

Nous allons utiliser un conteneur Ubuntu simple pour créer des fichiers et voir où ils sont stockés.

#### Scénario 1 : SANS volume (les fichiers disparaissent)

**Étape 1 : Créer un conteneur Ubuntu et entrer dedans**
```bash
docker run -it --name ubuntu-test ubuntu bash
```

**Explication détaillée :**
- `docker run` : **Créer et démarrer** un nouveau conteneur (différent de `docker exec` qui entre dans un conteneur existant)
- `-i` : Mode **i**nteractif (garder l'entrée standard ouverte)
- `-t` : Allouer un pseudo-**t**erminal (TTY - pour avoir un terminal propre)
- `--name ubuntu-test` : Donner le nom "ubuntu-test" au conteneur
- `ubuntu` : L'image à utiliser (Ubuntu Linux)
- `bash` : La commande à lancer (le terminal bash)

**En résumé :** "Docker, crée un nouveau conteneur Ubuntu nommé ubuntu-test et lance bash en mode interactif"

**Différence importante :**
- `docker run -it` = **Créer** un nouveau conteneur + entrer dedans
- `docker exec -it` = **Entrer** dans un conteneur qui existe déjà

Vous êtes maintenant DANS le conteneur Ubuntu ! 🐧

**Étape 2 : Créer des fichiers**

Dans le conteneur, tapez :
```bash
# Créer un dossier
mkdir /mes-donnees

# Créer quelques fichiers
echo "Fichier important numéro 1" > /mes-donnees/fichier1.txt
echo "Document secret" > /mes-donnees/fichier2.txt
echo "Liste de courses" > /mes-donnees/fichier3.txt

# Voir ce qu'on a créé
ls -la /mes-donnees/
cat /mes-donnees/fichier1.txt
```

**Résultat :**
```
Fichier important numéro 1
```

✅ Les fichiers sont là !

**Étape 3 : Sortir du conteneur**
```bash
exit
```

**Étape 4 : Supprimer le conteneur**
```bash
docker rm ubuntu-test
```

**Étape 5 : Recréer un conteneur et chercher les fichiers**
```bash
docker run -it --name ubuntu-nouveau ubuntu bash
```

**Rappel :** `docker run -it` = créer un NOUVEAU conteneur et entrer dedans en mode interactif

Dans le nouveau conteneur :
```bash
ls /mes-donnees/
```

**Résultat :**
```
ls: cannot access '/mes-donnees/': No such file or directory
```

😱 **LES FICHIERS ONT DISPARU !** Le dossier n'existe même plus !

Sortez :
```bash
exit
```

Nettoyez :
```bash
docker rm ubuntu-nouveau
```

#### Scénario 2 : AVEC volume (les fichiers survivent !)

**Étape 1 : Créer un volume**
```bash
docker volume create mes-fichiers
```

**Étape 2 : Créer un conteneur Ubuntu AVEC le volume**
```bash
docker run -it --name ubuntu-avec-volume -v mes-fichiers:/mes-donnees ubuntu bash
```

**Explication détaillée de la commande :**
- `docker run` : Créer et démarrer un nouveau conteneur
- `-i` : Mode **i**nteractif
- `-t` : Allouer un **t**erminal
- `--name ubuntu-avec-volume` : Nom du conteneur
- `-v mes-fichiers:/mes-donnees` : **V**olume = attacher le volume "mes-fichiers" au dossier `/mes-donnees` dans le conteneur
  - `mes-fichiers` = nom du volume (créé précédemment)
  - `/mes-donnees` = où ce volume sera accessible dans le conteneur
- `ubuntu` : Image Ubuntu
- `bash` : Lancer le terminal bash

**En résumé :** "Crée un conteneur Ubuntu avec le volume mes-fichiers monté sur /mes-donnees, et ouvre un terminal interactif"

**Étape 3 : Créer des fichiers dans le volume**

Dans le conteneur :
```bash
# Le dossier /mes-donnees existe déjà (créé par Docker pour le volume)
ls -la /mes-donnees/

# Créer des fichiers
echo "Ce fichier va SURVIVRE" > /mes-donnees/important.txt
echo "Alice, Bob, Charlie" > /mes-donnees/utilisateurs.txt
echo "Version 1.0" > /mes-donnees/version.txt

# Créer un sous-dossier
mkdir /mes-donnees/archives
echo "Données archivées" > /mes-donnees/archives/backup.txt

# Voir tout ce qu'on a créé
ls -R /mes-donnees/
cat /mes-donnees/important.txt
```

**Résultat :**
```
/mes-donnees/:
archives  important.txt  utilisateurs.txt  version.txt

/mes-donnees/archives:
backup.txt
```

**Étape 4 : Sortir et SUPPRIMER le conteneur**
```bash
exit
```

```bash
docker rm ubuntu-avec-volume
```

Le conteneur est DÉTRUIT ! 💥

**Étape 5 : Créer un NOUVEAU conteneur avec le MÊME volume**
```bash
docker run -it --name ubuntu-nouveau-2 -v mes-fichiers:/mes-donnees ubuntu bash
```

**Rappel :** On crée un conteneur DIFFÉRENT (ubuntu-nouveau-2) mais on attache le MÊME volume (mes-fichiers)

**Étape 6 : Vérifier que les fichiers sont TOUJOURS là**

Dans le nouveau conteneur :
```bash
# Lister les fichiers
ls -R /mes-donnees/

# Lire le contenu
cat /mes-donnees/important.txt
cat /mes-donnees/archives/backup.txt
```

**Résultat :**
```
/mes-donnees/:
archives  important.txt  utilisateurs.txt  version.txt

/mes-donnees/archives:
backup.txt

Ce fichier va SURVIVRE
Données archivées
```

🎉 **TOUS LES FICHIERS SONT LÀ !** Même le sous-dossier `archives` !

**Étape 7 : Modifier un fichier existant**

Dans le conteneur :
```bash
# Modifier un fichier
echo "Version 2.0 - Mise à jour" > /mes-donnees/version.txt

# Ajouter un nouveau fichier
echo "Nouvelle fonctionnalité" > /mes-donnees/feature.txt

# Vérifier
cat /mes-donnees/version.txt
```

Sortez :
```bash
exit
```

**Étape 8 : Supprimer et recréer ENCORE**
```bash
docker rm ubuntu-nouveau-2

docker run -it --name ubuntu-final -v mes-fichiers:/mes-donnees ubuntu bash
```

**Rappel :** Encore un nouveau conteneur (ubuntu-final), toujours le même volume (mes-fichiers)

Dans le conteneur :
```bash
cat /mes-donnees/version.txt
ls /mes-donnees/
```

**Résultat :**
```
Version 2.0 - Mise à jour

archives  feature.txt  important.txt  utilisateurs.txt  version.txt
```

✅ **Toutes les modifications sont PRÉSERVÉES !**

Sortez et nettoyez :
```bash
exit
docker rm ubuntu-final
```

#### Scénario 3 : Accéder au même volume depuis PLUSIEURS conteneurs

**Étape 1 : Créer 2 conteneurs qui partagent le MÊME volume**

Terminal 1 :
```bash
docker run -it --name ubuntu-writer -v mes-fichiers:/mes-donnees ubuntu bash
```

**Explication :** Créer un conteneur "ubuntu-writer" avec le volume mes-fichiers (celui qui va écrire des données)

Dans ce conteneur :
```bash
# Écrire un message
echo "Message du conteneur 1 à $(date)" > /mes-donnees/partage.txt
cat /mes-donnees/partage.txt
```

**NE SORTEZ PAS** de ce conteneur ! Laissez-le tourner.

**Étape 2 : Dans un AUTRE terminal, créer un deuxième conteneur**

Terminal 2 (nouveau terminal) :
```bash
docker run -it --name ubuntu-reader -v mes-fichiers:/mes-donnees ubuntu bash
```

**Explication :** Créer un DEUXIÈME conteneur "ubuntu-reader" avec le MÊME volume mes-fichiers!

**Important :** Les deux conteneurs tournent EN MÊME TEMPS et partagent le même volume!

Dans ce deuxième conteneur :
```bash
# Lire le message créé par le premier conteneur
cat /mes-donnees/partage.txt

# Ajouter quelque chose
echo "Réponse du conteneur 2" >> /mes-donnees/partage.txt
cat /mes-donnees/partage.txt
```

**Résultat :**
```
Message du conteneur 1 à [date]
Réponse du conteneur 2
```

**Étape 3 : Retourner dans le terminal 1**

Dans le premier conteneur (ubuntu-writer) :
```bash
# Relire le fichier
cat /mes-donnees/partage.txt
```

🎉 **Vous voyez la réponse du conteneur 2 !**

**C'est magique !** Deux conteneurs qui partagent les mêmes données en temps réel !

Sortez des deux conteneurs :
```bash
exit  # Dans chaque terminal
```

Nettoyez :
```bash
docker rm ubuntu-writer ubuntu-reader
```

#### Scénario 4 : Où sont VRAIMENT stockées les données du volume ?

**Question : Les données du volume "mes-fichiers", elles sont OÙ sur mon ordinateur ?**

**Réponse : Docker les stocke dans un dossier spécial !**

**Étape 1 : Inspecter le volume**
```bash
docker volume inspect mes-fichiers
```

**Résultat :**
```json
[
    {
        "CreatedAt": "2025-11-04T10:30:00Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mes-fichiers/_data",
        "Name": "mes-fichiers",
        "Options": null,
        "Scope": "local"
    }
]
```

**Regardez la ligne "Mountpoint" !**
```
"Mountpoint": "/var/lib/docker/volumes/mes-fichiers/_data"
```

C'est là que Docker stocke physiquement vos fichiers !

**Étape 2 : Voir les fichiers (nécessite les droits root)**

Sur Linux :
```bash
sudo ls -la /var/lib/docker/volumes/mes-fichiers/_data/
```

Sur Mac/Windows avec Docker Desktop :
```bash
# Les volumes sont dans une VM, utilisez un conteneur pour y accéder
docker run --rm -v mes-fichiers:/data alpine ls -la /data
```

**Résultat :**
```
total 24
drwxr-xr-x    3 root     root          4096 Nov  4 10:35 .
drwxr-xr-x    3 root     root          4096 Nov  4 10:30 ..
drwxr-xr-x    2 root     root          4096 Nov  4 10:32 archives
-rw-r--r--    1 root     root            45 Nov  4 10:35 feature.txt
-rw-r--r--    1 root     root            25 Nov  4 10:31 important.txt
-rw-r--r--    1 root     root            50 Nov  4 10:34 partage.txt
-rw-r--r--    1 root     root            19 Nov  4 10:31 utilisateurs.txt
-rw-r--r--    1 root     root            27 Nov  4 10:33 version.txt
```

🎯 **Ce sont VOS fichiers ! Ils existent vraiment sur votre disque dur !**

#### Nettoyage final

**Supprimer le volume (ATTENTION : cela supprime les données !)**
```bash
docker volume rm mes-fichiers
```

#### Ce que vous avez appris avec cet exercice Ubuntu :

✅ **Sans volume** : Les fichiers disparaissent quand vous supprimez le conteneur

✅ **Avec volume** : Les fichiers survivent à la suppression du conteneur

✅ **Plusieurs conteneurs** : Peuvent partager le même volume

✅ **Persistance** : Les modifications sont toujours sauvegardées dans le volume

✅ **Emplacement réel** : Docker stocke les volumes dans `/var/lib/docker/volumes/`

**Maintenant vous comprenez VRAIMENT comment fonctionnent les volumes !** 🎓

### Différence Volume nommé vs Bind mount

| Critère | Volume nommé | Bind mount |
|---------|--------------|------------|
| **Géré par** | Docker | Vous |
| **Emplacement** | Docker choisit | Vous choisissez |
| **Utilisation** | Bases de données | Code, développement |
| **Portabilité** | ✅ Fonctionne partout | ⚠️ Dépend de votre PC |
| **Performance** | ✅ Rapide | ✅ Rapide |
| **Recommandé pour** | Production | Développement |

---

## 🎓 Partie 5 : Projet Pratique - Site WordPress avec données permanentes

### Mission : WordPress qui ne perd jamais ses données

**On va créer :**
- Un volume pour la base de données MySQL
- Un volume pour les fichiers WordPress (thèmes, plugins, uploads)
- Un site qui survit aux redémarrages !

### Étape par étape

**Étape 1 : Créer les volumes**
```bash
docker volume create wordpress-db
docker volume create wordpress-files
```

**Étape 2 : Créer le réseau**
```bash
docker network create wordpress-network
```

**Étape 3 : Lancer MySQL avec volume**
```bash
docker run -d \
  --name wp-database \
  --network wordpress-network \
  -e MYSQL_ROOT_PASSWORD=root_password \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wp_user \
  -e MYSQL_PASSWORD=wp_password \
  -v wordpress-db:/var/lib/mysql \
  mysql:8.0
```

**Étape 4 : Lancer WordPress avec volume**
```bash
docker run -d \
  --name wp-site \
  --network wordpress-network \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=wp-database \
  -e WORDPRESS_DB_USER=wp_user \
  -e WORDPRESS_DB_PASSWORD=wp_password \
  -e WORDPRESS_DB_NAME=wordpress \
  -v wordpress-files:/var/www/html \
  wordpress:latest
```

**Étape 5 : Installer WordPress**

Ouvrez `http://localhost:8080` et installez WordPress :
- Titre : Mon Blog Docker
- Utilisateur : admin
- Mot de passe : (choisissez-en un)
- Email : votre@email.com

**Étape 6 : Créer du contenu**

Dans WordPress :
- Créez 2-3 articles
- Changez le thème
- Installez un plugin
- Ajoutez une image

**Étape 7 : TEST - Supprimer TOUT**
```bash
docker stop wp-site wp-database
docker rm wp-site wp-database
```

Les conteneurs sont détruits ! 💥

**Étape 8 : Recréer TOUT (mêmes commandes)**
```bash
docker run -d \
  --name wp-database \
  --network wordpress-network \
  -e MYSQL_ROOT_PASSWORD=root_password \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wp_user \
  -e MYSQL_PASSWORD=wp_password \
  -v wordpress-db:/var/lib/mysql \
  mysql:8.0

docker run -d \
  --name wp-site \
  --network wordpress-network \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=wp-database \
  -e WORDPRESS_DB_USER=wp_user \
  -e WORDPRESS_DB_PASSWORD=wp_password \
  -e WORDPRESS_DB_NAME=wordpress \
  -v wordpress-files:/var/www/html \
  wordpress:latest
```

**Attendez que WordPress et MySQL démarrent (environ 30 secondes).**

**Étape 9 : Ouvrir le site**

`http://localhost:8080`

🎉 **TOUT EST LÀ !**
- Vos articles
- Votre thème
- Vos plugins
- Vos images

**RIEN n'a été perdu !** ✨

**Pourquoi ?** Les volumes `wordpress-db` et `wordpress-files` ont conservé toutes les données !

---

## 💼 Partie 6 : Exercice Pratique

### Exercice : Base de données PostgreSQL permanente

**Mission : Créer une base PostgreSQL dont les données survivent**

**Étape 1 : Créer un volume**
```bash
docker volume create postgres-data
```

**Étape 2 : Lancer PostgreSQL avec le volume**
```bash
docker run -d \
  --name postgres-permanent \
  -e POSTGRES_PASSWORD=secret123 \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=entreprise \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15
```

**Attendez que PostgreSQL démarre (environ 10 secondes).**

**Étape 3 : Créer des données**
```bash
docker exec -it postgres-permanent psql -U admin -d entreprise
```

**Explication de la commande :**
- `docker exec -it` : Exécuter de manière interactive dans le conteneur
- `postgres-permanent` : Nom du conteneur
- `psql` : Client PostgreSQL (équivalent du client `mysql` pour MySQL)
- `-U admin` : **U**ser = admin (utilisateur PostgreSQL)
- `-d entreprise` : **D**atabase = entreprise (nom de la base de données)

**En résumé :** "Entre dans PostgreSQL en mode interactif sur la base entreprise"

Dans PostgreSQL :
```sql
CREATE TABLE employes (
  id SERIAL PRIMARY KEY,
  nom VARCHAR(100),
  poste VARCHAR(100),
  salaire INTEGER
);

INSERT INTO employes (nom, poste, salaire) VALUES
('Alice Dupont', 'Développeuse', 45000),
('Bob Martin', 'Designer', 40000),
('Charlie Durand', 'Manager', 55000);

SELECT * FROM employes;
```

Vous voyez vos 3 employés ! ✅

**Sortez :**
```sql
\q
```

**Étape 4 : Supprimer le conteneur**
```bash
docker stop postgres-permanent
docker rm postgres-permanent
```

**Étape 5 : Recréer le conteneur (même volume !)**
```bash
docker run -d \
  --name postgres-nouveau \
  -e POSTGRES_PASSWORD=secret123 \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=entreprise \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15
```

**Attendez que PostgreSQL démarre (environ 10 secondes).**

**Étape 6 : Vérifier les données**
```bash
docker exec -it postgres-nouveau psql -U admin -d entreprise -c "SELECT * FROM employes;"
```

**Explication de la commande :**
- `docker exec -it postgres-nouveau` : Exécuter dans le conteneur
- `psql -U admin -d entreprise` : Client PostgreSQL connecté à la base
- `-c "SELECT * FROM employes;"` : **C**ommand = exécuter cette commande SQL puis sortir

**Différence avec MySQL :** PostgreSQL utilise `-c` (command) alors que MySQL utilise `-e` (execute), mais c'est la même chose!

**Résultat :**
```
 id |      nom       |    poste     | salaire
----+----------------+--------------+---------
  1 | Alice Dupont   | Développeuse |   45000
  2 | Bob Martin     | Designer     |   40000
  3 | Charlie Durand | Manager      |   55000
```

✅ **Les données sont intactes !** Alice, Bob et Charlie sont toujours là !

**Nettoyage :**
```bash
docker stop postgres-nouveau
docker rm postgres-nouveau
docker volume rm postgres-data
```

---

## 📊 Partie 7 : Les bonnes pratiques

### ✅ À FAIRE

**1. Toujours utiliser des volumes pour les bases de données**
```bash
# BON ✅
docker run -d -v mysql-data:/var/lib/mysql mysql

# MAUVAIS ❌
docker run -d mysql  # Les données vont disparaître !
```

**2. Nommer vos volumes explicitement**
```bash
# BON ✅
docker volume create projet-prod-db
docker run -v projet-prod-db:/var/lib/mysql mysql

# MOINS BON ⚠️
docker run -v /var/lib/mysql mysql  # Volume anonyme
```

**3. Documenter vos volumes**

Créez un fichier `README.md` :
```markdown
# Volumes du projet

- `wordpress-db` : Base de données MySQL
- `wordpress-files` : Fichiers WordPress (thèmes, plugins)
- `nginx-config` : Configuration Nginx
```

**4. Sauvegarder vos volumes régulièrement**
```bash
# Sauvegarder un volume dans un fichier tar
docker run --rm -v mysql-data:/data -v $(pwd):/backup ubuntu tar czf /backup/backup.tar.gz /data
```

### ❌ À ÉVITER

**1. Ne pas oublier les volumes lors des tests**

Si vous testez et supprimez tout sans volumes, vous perdez vos données de test !

**2. Ne pas mélanger données et code dans le même volume**

Séparez :
- Volume pour la base de données
- Bind mount pour le code

**3. Ne pas supprimer les volumes sans vérifier**
```bash
# DANGEREUX ❌
docker volume prune -f  # Supprime TOUS les volumes inutilisés !
```

Vérifiez d'abord :
```bash
docker volume ls  # Voir ce qui sera supprimé
```

---

## ✅ Quiz de validation

**Question 1 : Que se passe-t-il quand vous supprimez un conteneur SANS volume ?**

- A) Les données restent
- B) Les données disparaissent
- C) Les données vont dans un dossier temporaire
- D) Docker fait une sauvegarde automatique

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Les données disparaissent**

Sans volume, toutes les données créées dans le conteneur sont supprimées avec lui !

</details>

---

**Question 2 : Quelle option pour attacher un volume à un conteneur ?**

- A) `-volume`
- B) `-mount`
- C) `-v`
- D) `-data`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : `-v`**

L'option `-v` (ou `--volume`) permet d'attacher un volume ou un bind mount à un conteneur.

</details>

---

**Question 3 : Quel chemin pour le volume MySQL ?**

- A) `-v mysql-data:/data`
- B) `-v mysql-data:/var/lib/mysql`
- C) `-v mysql-data:/mysql`
- D) `-v mysql-data:/database`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : `-v mysql-data:/var/lib/mysql`**

MySQL stocke ses données dans `/var/lib/mysql`. C'est le chemin à utiliser pour le volume.

</details>

---

**Question 4 : Quelle différence entre volume nommé et bind mount ?**

- A) Aucune différence
- B) Volume nommé = géré par Docker, Bind mount = vous choisissez le dossier
- C) Bind mount est plus rapide
- D) Volume nommé ne fonctionne que sur Linux

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B**

Volume nommé : Docker gère l'emplacement (recommandé pour les bases de données)
Bind mount : Vous choisissez un dossier de votre PC (pratique pour le développement)

</details>

---

**Question 5 : Comment voir tous vos volumes ?**

- A) `docker ps -v`
- B) `docker volume list`
- C) `docker volume ls`
- D) `docker show volumes`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : `docker volume ls`**

Cette commande liste tous les volumes Docker sur votre système.

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Le problème

❌ **Sans volume :** Données dans le conteneur → Suppression du conteneur = perte des données

### La solution

✅ **Avec volume :** Données dans un volume externe → Le conteneur peut être supprimé, les données restent

### Les 2 types de volumes

**1. Volume nommé (pour bases de données)**
```bash
# Créer
docker volume create mon-volume

# Utiliser
docker run -v mon-volume:/var/lib/mysql mysql
```

**2. Bind mount (pour développement)**
```bash
# Utiliser
docker run -v /chemin/sur/pc:/chemin/dans/conteneur nginx
```

### Les chemins importants

| Base de données | Chemin du volume |
|----------------|------------------|
| MySQL | `/var/lib/mysql` |
| PostgreSQL | `/var/lib/postgresql/data` |
| MongoDB | `/data/db` |
| Redis | `/data` |

### Commandes essentielles

```bash
docker volume create nom          # Créer un volume
docker volume ls                  # Lister les volumes
docker volume inspect nom         # Détails d'un volume
docker volume rm nom              # Supprimer un volume
docker volume prune               # Nettoyer les volumes inutilisés
```

---

## 🚀 Et maintenant ?

**FÉLICITATIONS ! Vous maîtrisez les volumes Docker !** 🎉

### Ce que vous savez maintenant :

- ✅ Pourquoi les données disparaissent sans volume
- ✅ Créer et utiliser des volumes nommés
- ✅ Utiliser des bind mounts pour le développement
- ✅ Sauvegarder les données des bases de données
- ✅ Créer des applications avec données persistantes

### Dans le prochain cours (Cours 6) :

**Créer vos propres images avec DOCKERFILE !** 🏗️

Vous apprendrez :
1. C'est quoi un Dockerfile
2. Créer votre première image personnalisée
3. Les instructions essentielles (FROM, RUN, COPY, CMD...)
4. Construire et tester votre image
5. Cas pratique : Créer une API Node.js dans Docker

**C'est là que ça devient VRAIMENT puissant !** 💪

---

## 📝 Aide-mémoire volumes

**Pattern base de données avec volume :**
```bash
# 1. Créer le volume
docker volume create db-data

# 2. Lancer avec le volume
docker run -d \
  --name ma-base \
  -v db-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0
```

**Pattern site web en développement :**
```bash
# Lancer nginx avec votre code
docker run -d \
  --name dev-site \
  -p 8080:80 \
  -v ~/mon-projet:/usr/share/nginx/html \
  nginx
```

**Pattern application complète :**
```bash
# Volumes
docker volume create app-db
docker volume create app-files

# Base
docker run -d --name db -v app-db:/var/lib/mysql mysql

# App
docker run -d --name app -v app-files:/app/data mon-app
```

---

## 💡 Conseil de pro

**Créez un script pour vos projets !**

Fichier `start.sh` :
```bash
#!/bin/bash

# Créer les volumes si ils n'existent pas
docker volume create wordpress-db 2>/dev/null
docker volume create wordpress-files 2>/dev/null

# Créer le réseau
docker network create wordpress-network 2>/dev/null

# Lancer MySQL
docker run -d \
  --name wp-db \
  --network wordpress-network \
  -v wordpress-db:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8.0

# Attendre que MySQL soit prêt
sleep 10

# Lancer WordPress
docker run -d \
  --name wp \
  --network wordpress-network \
  -p 8080:80 \
  -v wordpress-files:/var/www/html \
  -e WORDPRESS_DB_HOST=wp-db \
  wordpress

echo "WordPress disponible sur http://localhost:8080"
```

**Rendez-le exécutable et lancez-le :**
```bash
chmod +x start.sh
./start.sh
```

---

**🎓 Vous êtes prêt pour le Cours 6 (DOCKERFILE) !**

Super travail ! Vous avez franchi une étape cruciale. Vos données ne disparaîtront plus jamais ! Le prochain cours va vous apprendre à créer vos propres images personnalisées. C'est passionnant ! 🚀💪
