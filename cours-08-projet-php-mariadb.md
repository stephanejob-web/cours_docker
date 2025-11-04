# 📘 COURS 08 : PROJET COMPLET - Application PHP + MariaDB

**Durée :** 2h30
**Type :** Projet guidé (Théorique + Pratique)
**Niveau :** Débutant (mais APRÈS les Cours 04, 05, 06 et 07 !)

---

## 🎯 OBJECTIF DE CE COURS

### Ce que vous allez créer :

**Une application web complète** qui affiche et enregistre des visiteurs dans une base de données.

**MAIS SURTOUT :** Ce cours combine TOUS les concepts que vous avez appris !

---

## 🔑 LES 4 CONCEPTS CLÉS QUE VOUS ALLEZ UTILISER

Ce cours est CRUCIAL car il **combine les 4 concepts essentiels** :

```
┌────────────────────────────────────────────────────────┐
│                 VOTRE PROJET PHP+MariaDB               │
├────────────────────────────────────────────────────────┤
│                                                        │
│  📘 COURS 04 : LES PORTS                              │
│  └─ Pour accéder au site web depuis votre navigateur │
│     -p 8080:80                                        │
│                                                        │
│  📘 COURS 05 : LES VOLUMES                            │
│  └─ Pour modifier le code en temps réel              │
│     -v $(pwd):/var/www/html                           │
│                                                        │
│  📘 COURS 06 : LES RÉSEAUX                            │
│  └─ Pour que PHP parle à MariaDB                     │
│     --network mon-reseau                              │
│                                                        │
│  📘 COURS 07 : VARIABLES D'ENVIRONNEMENT              │
│  └─ Pour configurer la base de données               │
│     -e MYSQL_ROOT_PASSWORD=...                        │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Si vous n'avez PAS fait les cours 04, 05, 06 et 07 → Arrêtez-vous et faites-les d'abord !**

---

## ⚠️ PRÉREQUIS OBLIGATOIRES

**Vous DEVEZ avoir compris :**

✅ **Cours 04 - Les Ports** : Comment utiliser `-p` pour accéder à un conteneur
✅ **Cours 05 - Les Volumes** : Comment utiliser `-v` pour monter des fichiers
✅ **Cours 06 - Les Réseaux** : Comment utiliser `--network` pour connecter des conteneurs
✅ **Cours 07 - Variables d'env** : Comment utiliser `-e` pour configurer

**Si vous ne vous souvenez plus, relisez-les avant de continuer !**

---

## 📋 PLAN DU PROJET

**Voici ce qu'on va faire, étape par étape :**

```
1. Préparer le dossier du projet (sur votre PC)
2. Créer le fichier PHP (le code de l'application)
3. Créer un réseau Docker (Cours 06 !)
4. Lancer MariaDB (Cours 07 !)
5. Lancer PHP avec Apache (Cours 04, 05, 06 !)
6. Installer les extensions PHP
7. Tester l'application (Tout fonctionne ensemble !)
8. Modifier le code en temps réel (Cours 05 !)
```

---

## 🏗️ PARTIE 1 : PRÉPARER LE PROJET

### Étape 1 : Créer le dossier du projet

**POURQUOI UN DOSSIER ?**

Vous allez créer un fichier `index.php` qui contient le code de votre site.
Plus tard, grâce au **montage de volume (Cours 05)**, Docker pourra lire ce fichier.

**TAPEZ CES COMMANDES :**

```bash
# Aller dans votre dossier personnel
cd ~

# Créer un dossier pour le projet
mkdir mon-projet-docker

# Aller dedans
cd mon-projet-docker

# Vérifier que vous êtes bien dedans
pwd
```

**Vous DEVEZ voir :**
```
/home/votre-nom/mon-projet-docker
```

✅ **Parfait ! Vous êtes au bon endroit.**

---

### Étape 2 : Créer le fichier PHP

**CE QU'ON VA FAIRE :**

Créer un fichier `index.php` qui :
- Se connecte à MariaDB
- Crée une table `visiteurs`
- Ajoute un visiteur à chaque visite
- Affiche les derniers visiteurs

**TAPEZ CETTE COMMANDE :**

```bash
cat > index.php << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Mon App Docker</title>
    <style>
        body {
            font-family: Arial;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 50px;
            text-align: center;
        }
        .box {
            background: rgba(255,255,255,0.1);
            padding: 20px;
            border-radius: 10px;
            margin: 20px auto;
            max-width: 600px;
        }
    </style>
</head>
<body>
    <h1>🐳 Mon App Docker - PHP + MariaDB 🐳</h1>

    <?php
    $host = 'ma-base-de-donnees';  // Nom du conteneur MariaDB
    $user = 'stephane';
    $pass = 'motdepasse123';
    $db = 'ma_base';

    try {
        $connexion = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
        $connexion->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        echo '<div class="box">';
        echo '<h2>✅ Connexion réussie à MariaDB !</h2>';
        echo '<p>Les deux conteneurs communiquent parfaitement !</p>';
        echo '</div>';

        // Créer une table si elle n'existe pas
        $connexion->exec("CREATE TABLE IF NOT EXISTS visiteurs (
            id INT AUTO_INCREMENT PRIMARY KEY,
            nom VARCHAR(50),
            date_visite DATETIME
        )");

        // Ajouter un visiteur
        $nom = "Visiteur_" . rand(1, 1000);
        $stmt = $connexion->prepare("INSERT INTO visiteurs (nom, date_visite) VALUES (?, NOW())");
        $stmt->execute([$nom]);

        // Afficher les visiteurs
        $stmt = $connexion->query("SELECT * FROM visiteurs ORDER BY date_visite DESC LIMIT 5");
        $visiteurs = $stmt->fetchAll(PDO::FETCH_ASSOC);

        echo '<div class="box">';
        echo '<h2>📋 Derniers visiteurs :</h2>';
        foreach ($visiteurs as $v) {
            echo "<p>{$v['nom']} - {$v['date_visite']}</p>";
        }
        echo '</div>';

    } catch(PDOException $e) {
        echo '<div class="box">';
        echo '<h2>❌ Erreur de connexion</h2>';
        echo '<p>' . $e->getMessage() . '</p>';
        echo '</div>';
    }
    ?>

    <div class="box">
        <p>🔄 Actualisez la page pour ajouter un nouveau visiteur !</p>
    </div>
</body>
</html>
EOF
```

**Appuyez sur `Entrée`**

✅ **Le fichier `index.php` est créé !**

**Vérifiez :**
```bash
ls
```

**Vous devez voir :**
```
index.php
```

---

### 🧠 COMPRENDRE LE CODE PHP (ligne importante !)

**Regardez la ligne 24 du fichier :**

```php
$host = 'ma-base-de-donnees';  // ← Le nom du conteneur MariaDB !
```

**POURQUOI `'ma-base-de-donnees'` et pas une adresse IP ?**

→ **Grâce au réseau Docker (Cours 06) !**

Quand les deux conteneurs sont dans le même réseau :
- PHP peut appeler MariaDB par son **nom de conteneur**
- Docker fait automatiquement la traduction : `ma-base-de-donnees` → adresse IP réelle
- C'est comme un annuaire téléphonique automatique !

**C'est pour ça qu'on va créer un réseau !**

---

## 🌐 PARTIE 2 : CRÉER LE RÉSEAU (CONCEPT 1 : COURS 06)

### Pourquoi un réseau ?

**SANS RÉSEAU :** Les conteneurs sont isolés, ils ne peuvent pas se parler.

```
┌─────────────────┐         ┌──────────────────┐
│  Conteneur PHP  │    X    │ Conteneur MariaDB│
│                 │  Mur    │                  │
│  "Où es-tu ?"   │ ═══════ │  "Je suis là !"  │
└─────────────────┘         └──────────────────┘
```

**AVEC RÉSEAU :** Ils peuvent se parler par leur nom !

```
┌─────────────────┐         ┌──────────────────┐
│  Conteneur PHP  │ ←─────→ │ Conteneur MariaDB│
│                 │ Réseau  │                  │
│  "Hé MariaDB !" │ Docker  │  "Oui PHP ?"     │
└─────────────────┘         └──────────────────┘
```

**C'est exactement ce que vous avez appris au Cours 06 !**

---

### Créer le réseau

**TAPEZ :**

```bash
docker network create mon-reseau
```

**Ce que ça fait :**
- Crée un réseau privé appelé `mon-reseau`
- Comme créer un groupe WhatsApp pour vos conteneurs !
- Tous les conteneurs dans ce réseau pourront se parler par leur nom

**Vérifiez :**
```bash
docker network ls
```

**Vous devez voir :**
```
NETWORK ID     NAME         DRIVER    SCOPE
...
xxxxxxxxxxx    mon-reseau   bridge    local
```

✅ **Le réseau est créé !**

---

## 🗄️ PARTIE 3 : LANCER MARIADB (CONCEPT 2 : COURS 07)

### Lancer le conteneur MariaDB

**TAPEZ :**

```bash
docker run -d \
  --name ma-base-de-donnees \
  --network mon-reseau \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=ma_base \
  -e MYSQL_USER=stephane \
  -e MYSQL_PASSWORD=motdepasse123 \
  mariadb:latest
```

---

### 🎓 DÉCORTIQUONS LA COMMANDE

**Chaque ligne a un rôle précis :**

```bash
docker run -d \
```
→ Lance un conteneur en mode détaché (tourne en fond)

```bash
  --name ma-base-de-donnees \
```
→ Donne le nom `ma-base-de-donnees` au conteneur
→ **C'est ce nom que PHP va utiliser pour se connecter !**

```bash
  --network mon-reseau \
```
→ **CONCEPT COURS 06 : RÉSEAUX**
→ Met le conteneur dans le réseau `mon-reseau`
→ Grâce à ça, PHP pourra le trouver par son nom !

```bash
  -e MYSQL_ROOT_PASSWORD=rootpass \
```
→ **CONCEPT COURS 07 : VARIABLES D'ENVIRONNEMENT**
→ Définit le mot de passe de l'utilisateur `root` (admin de la base)

```bash
  -e MYSQL_DATABASE=ma_base \
```
→ **CONCEPT COURS 07 : VARIABLES D'ENVIRONNEMENT**
→ Crée automatiquement une base de données appelée `ma_base`

```bash
  -e MYSQL_USER=stephane \
```
→ **CONCEPT COURS 07 : VARIABLES D'ENVIRONNEMENT**
→ Crée un utilisateur appelé `stephane`

```bash
  -e MYSQL_PASSWORD=motdepasse123 \
```
→ **CONCEPT COURS 07 : VARIABLES D'ENVIRONNEMENT**
→ Définit le mot de passe de l'utilisateur `stephane`

```bash
  mariadb:latest
```
→ Utilise l'image `mariadb` version `latest` (la plus récente)

---

### ⏳ Attendre que MariaDB démarre

**MariaDB prend quelques secondes à démarrer complètement.**

**Attendez 10 secondes, puis vérifiez :**

```bash
docker ps
```

**Vous devez voir :**
```
CONTAINER ID   IMAGE            STATUS         NAMES
xxxxxxxxxxx    mariadb:latest   Up 5 seconds   ma-base-de-donnees
```

✅ **MariaDB tourne ! Passez à l'étape suivante.**

---

## 🌐 PARTIE 4 : LANCER PHP (CONCEPT 3 ET 4 : COURS 04 ET 05)

### La commande la plus importante du projet !

**ATTENTION : LISEZ BIEN LES EXPLICATIONS AVANT DE TAPER LA COMMANDE !**

---

### ⚠️ SUPER IMPORTANT : Vérifiez votre dossier !

**Avant de taper la commande, vérifiez que vous êtes dans `mon-projet-docker` :**

```bash
pwd
```

**Vous DEVEZ voir :**
```
/home/votre-nom/mon-projet-docker
```

**Pourquoi c'est important ?**

→ La commande utilise `-v $(pwd):/var/www/html` (Cours 05 : Volumes)
→ `$(pwd)` = le dossier actuel
→ Si vous n'êtes PAS dans `mon-projet-docker`, Docker montera le MAUVAIS dossier !
→ Résultat : Le site ne marchera pas !

**Si vous n'êtes pas dans le bon dossier :**
```bash
cd ~/mon-projet-docker
```

---

### Lancer PHP + Apache

**TAPEZ :**

```bash
docker run -d \
  --name mon-site-php \
  --network mon-reseau \
  -p 8080:80 \
  -v $(pwd):/var/www/html \
  php:8.2-apache
```

---

### 🎓 DÉCORTIQUONS LA COMMANDE (avec les concepts !)

```bash
docker run -d \
```
→ Lance un conteneur en mode détaché

```bash
  --name mon-site-php \
```
→ Donne le nom `mon-site-php` au conteneur

```bash
  --network mon-reseau \
```
→ **CONCEPT COURS 06 : RÉSEAUX**
→ Met le conteneur dans le réseau `mon-reseau`
→ **Grâce à ça, PHP peut parler à MariaDB !**

```bash
  -p 8080:80 \
```
→ **CONCEPT COURS 04 : PORTS**
→ Crée un "tunnel" :
→ `localhost:8080` (votre PC) → port `80` (dans le conteneur)
→ **Grâce à ça, vous pourrez accéder au site depuis votre navigateur !**

**Schéma du mapping de port :**
```
VOTRE NAVIGATEUR              DOCKER              CONTENEUR PHP
localhost:8080  ───────────────────────────►  Apache (port 80)
```

```bash
  -v $(pwd):/var/www/html \
```
→ **CONCEPT COURS 05 : VOLUMES**
→ Crée un "portail magique" :
→ `mon-projet-docker/` (votre PC) ⇔ `/var/www/html` (dans le conteneur)
→ **Grâce à ça :**
   - Apache peut lire votre fichier `index.php`
   - Vous pourrez modifier le code EN TEMPS RÉEL sans redémarrer !

**Schéma du montage de volume :**
```
┌─────────────────┐         ┌──────────────────┐
│   VOTRE PC      │         │  CONTENEUR PHP   │
│                 │         │                  │
│  mon-projet-    │ ═══════►│  /var/www/html   │
│  docker/        │ VOLUME  │                  │
│  └─ index.php   │         │  └─ index.php    │
└─────────────────┘         └──────────────────┘
```

```bash
  php:8.2-apache
```
→ Utilise l'image `php:8.2-apache` (PHP 8.2 + serveur web Apache)

---

### Vérifier que PHP tourne

```bash
docker ps
```

**Vous devez voir les 2 conteneurs :**
```
CONTAINER ID   IMAGE              STATUS         NAMES
xxxxxxxxxxx    php:8.2-apache     Up 5 seconds   mon-site-php
xxxxxxxxxxx    mariadb:latest     Up 1 minute    ma-base-de-donnees
```

✅ **Les deux conteneurs tournent !**

---

## 🔧 PARTIE 5 : INSTALLER LES EXTENSIONS PHP

### 🤔 Pourquoi cette étape ?

**LE PROBLÈME :**

L'image `php:8.2-apache` contient :
- ✅ PHP de base (le langage)
- ✅ Apache (le serveur web)
- ❌ **MAIS PAS** les extensions pour MySQL !

**C'est comme acheter une voiture :**
- ✅ Vous avez le moteur
- ✅ Vous avez les roues
- ❌ **MAIS PAS** la radio !

**Il faut installer la "radio" (les extensions MySQL) !**

---

### 💡 Pourquoi PHP n'inclut PAS MySQL par défaut ?

**3 raisons :**

1. **Taille** → MySQL est gros ! Si tout le monde ne l'utilise pas, pourquoi le mettre ?

2. **Flexibilité** → Certains utilisent MySQL, d'autres PostgreSQL, d'autres SQLite...
   Docker vous laisse choisir ce dont VOUS avez besoin !

3. **Sécurité** → Moins de code = moins de failles

---

### Installer les extensions

**TAPEZ :**

```bash
docker exec mon-site-php docker-php-ext-install pdo pdo_mysql
```

---

### 🎓 DÉCORTIQUONS LA COMMANDE

```bash
docker exec
```
→ "Docker, exécute une commande DANS un conteneur qui tourne"

```bash
mon-site-php
```
→ "Le conteneur s'appelle `mon-site-php`"

```bash
docker-php-ext-install
```
→ "Utilise l'outil d'installation d'extensions PHP"
→ (Cet outil est inclus dans l'image `php:8.2-apache` !)

```bash
pdo pdo_mysql
```
→ "Installe ces 2 extensions :"
- **pdo** = PHP Data Objects (pour se connecter à des bases de données)
- **pdo_mysql** = La couche spécifique pour MySQL/MariaDB

---

### Ce qui se passe pendant l'installation

**Vous verrez défiler plein de lignes :**

```
Configuring for:
PHP Api Version:         20220805
Zend Module Api No:      20220805
...
Installing '/usr/local/lib/php/extensions/...'
...
```

**C'EST NORMAL !** Ça compile les extensions.

⏳ **Ça prend environ 30 secondes à 1 minute.**

✅ **Quand ça dit "complete" ou que ça s'arrête → c'est bon !**

---

### Redémarrer PHP pour appliquer les changements

**Pourquoi redémarrer ?**

Imaginez que PHP est une voiture qui roule.
Vous venez d'installer la radio **pendant que la voiture roule** !

Pour que la radio fonctionne, il faut :
1. Arrêter la voiture
2. Redémarrer la voiture
3. Maintenant la radio marche !

**C'est pareil pour PHP !**

**TAPEZ :**

```bash
docker restart mon-site-php
```

**Ce qui se passe :**
1. Docker arrête le conteneur (2 secondes)
2. Docker redémarre le conteneur (2 secondes)
3. PHP charge maintenant les nouvelles extensions !

✅ **TERMINÉ ! Maintenant PHP peut parler à MariaDB !**

---

## 🌐 PARTIE 6 : TESTER L'APPLICATION

### Ouvrir le site dans le navigateur

**Ouvrez votre navigateur web et allez sur :**

```
http://localhost:8080
```

---

### ✅ Ce que vous devriez voir

**Si tout fonctionne, vous verrez :**

1. Un fond dégradé violet/bleu
2. Le titre : "🐳 Mon App Docker - PHP + MariaDB 🐳"
3. Un message : "✅ Connexion réussie à MariaDB !"
4. Une liste des derniers visiteurs avec leur nom et date

**Exemple :**
```
✅ Connexion réussie à MariaDB !
Les deux conteneurs communiquent parfaitement !

📋 Derniers visiteurs :
Visiteur_742 - 2025-11-03 14:32:10
```

---

### 🔄 Actualiser la page

**Appuyez sur `F5` ou `Ctrl + R` plusieurs fois.**

**Ce qui se passe :**
- À chaque actualisation, un nouveau visiteur est ajouté !
- La liste se met à jour !
- Les données sont sauvegardées dans MariaDB !

**C'EST MAGIQUE ! Les deux conteneurs communiquent !** 🎉

---

### 🎯 COMPRENDRE CE QUI SE PASSE

**Le voyage de la requête :**

```
1. Votre navigateur → http://localhost:8080
                      ▼
2. (COURS 04 : PORTS) Docker redirige → Conteneur PHP (port 80)
                      ▼
3. Apache cherche → /var/www/html/index.php
                      ▼
4. (COURS 05 : VOLUMES) Grâce au montage → Il trouve votre fichier sur votre PC !
                      ▼
5. PHP exécute le code
                      ▼
6. Le code se connecte à "ma-base-de-donnees"
                      ▼
7. (COURS 06 : RÉSEAUX) Docker trouve MariaDB grâce au réseau !
                      ▼
8. (COURS 07 : VARIABLES) PHP utilise les identifiants configurés
                      ▼
9. MariaDB renvoie les données → PHP génère le HTML → Votre navigateur
                      ▼
10. Vous voyez la page ! 🎉
```

**VOUS AVEZ UTILISÉ LES 4 CONCEPTS !**

---

## 🎨 PARTIE 7 : MODIFIER LE CODE EN TEMPS RÉEL (COURS 05)

### Le super pouvoir des volumes !

**Grâce au montage de volume (Cours 05), vous pouvez modifier le code SANS redémarrer !**

---

### Test 1 : Changer le titre

**1. Ouvrez `index.php` avec un éditeur**

```bash
nano index.php
```

**2. Changez le titre (ligne 6)**

**Avant :**
```php
<title>Mon App Docker</title>
```

**Après :**
```php
<title>Mon App Docker - MODIFIÉ EN DIRECT !</title>
```

**3. Enregistrez**
- Avec nano : `Ctrl + O` puis `Entrée`, puis `Ctrl + X`

**4. Actualisez le navigateur (`F5`)**

**5. 🎉 REGARDEZ L'ONGLET DU NAVIGATEUR !**
- Le titre a changé INSTANTANÉMENT !
- Vous n'avez PAS redémarré le conteneur !

---

### Test 2 : Changer le style

**1. Rouvrez `index.php`**

```bash
nano index.php
```

**2. Changez la couleur de fond (ligne 10)**

**Avant :**
```php
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
```

**Après :**
```php
background: linear-gradient(135deg, #FF6B6B 0%, #4ECDC4 100%);
```

**3. Enregistrez et actualisez**

**4. Le fond a changé de couleur !** 🎨

---

### 🧠 POURQUOI C'EST INSTANTANÉ ?

**Voici ce qui se passe en réalité :**

1. **Le fichier N'EST PAS copié dans le conteneur !**
2. **Le conteneur LIT DIRECTEMENT le fichier sur votre PC !**
3. **Quand vous modifiez le fichier → Apache lit la nouvelle version immédiatement !**

**C'est le principe du volume (Cours 05) !**

**Workflow de développement classique :**
```
1. Écrire du code dans votre éditeur
2. Enregistrer (Ctrl + S)
3. Actualiser le navigateur (F5)
4. Voir le résultat
5. Répéter → C'est ultra-rapide !
```

---

## 🔍 PARTIE 8 : EXPLORER LA BASE DE DONNÉES

### Entrer dans MariaDB

**Vous voulez voir ce qui est dans la base de données ?**

**TAPEZ :**

```bash
docker exec -it ma-base-de-donnees mysql -u stephane -pmotdepasse123 ma_base
```

**Décortiquons :**
- `docker exec -it` → Exécute une commande interactive dans le conteneur
- `ma-base-de-donnees` → Le nom du conteneur
- `mysql` → Lance le client MySQL
- `-u stephane` → Utilisateur
- `-pmotdepasse123` → Mot de passe (collé au `-p`, sans espace !)
- `ma_base` → La base de données à ouvrir

---

### Voir les visiteurs

**Une fois dedans, tapez :**

```sql
SELECT * FROM visiteurs;
```

**Vous verrez :**
```
+----+---------------+---------------------+
| id | nom           | date_visite         |
+----+---------------+---------------------+
|  1 | Visiteur_742  | 2025-11-03 14:32:10 |
|  2 | Visiteur_123  | 2025-11-03 14:33:05 |
|  3 | Visiteur_891  | 2025-11-03 14:34:12 |
+----+---------------+---------------------+
```

**C'est la table créée automatiquement par le code PHP !**

---

### Sortir de MySQL

```sql
exit
```

✅ **Vous êtes de retour dans votre terminal.**

---

## 🛑 PARTIE 9 : ARRÊTER ET NETTOYER

### Arrêter le projet

**Arrêter les 2 conteneurs :**

```bash
docker stop mon-site-php ma-base-de-donnees
```

**Vérifiez :**
```bash
docker ps
```

**Les conteneurs ne doivent plus apparaître (ils sont arrêtés).**

---

### Supprimer les conteneurs

```bash
docker rm mon-site-php ma-base-de-donnees
```

---

### Supprimer le réseau

```bash
docker network rm mon-reseau
```

---

### (Optionnel) Supprimer les images

**Si vous voulez libérer de l'espace :**

```bash
docker rmi php:8.2-apache mariadb:latest
```

⚠️ **Attention :** Si vous les supprimez, il faudra les retélécharger la prochaine fois !

---

## 🎓 RÉCAPITULATIF : Ce que vous avez appris

### Les 4 concepts utilisés dans ce projet

✅ **COURS 04 : PORTS (`-p 8080:80`)**
→ Accéder au site web depuis votre navigateur
→ `localhost:8080` → conteneur port `80`

✅ **COURS 05 : VOLUMES (`-v $(pwd):/var/www/html`)**
→ Monter votre code dans le conteneur
→ Modifier en temps réel sans redémarrer

✅ **COURS 06 : RÉSEAUX (`--network mon-reseau`)**
→ Faire communiquer PHP et MariaDB
→ Les conteneurs se trouvent par leur nom

✅ **COURS 07 : VARIABLES D'ENV (`-e MYSQL_...`)**
→ Configurer MariaDB (mot de passe, base de données, utilisateur)
→ Sans modifier le code !

---

### Le schéma complet du projet

```
┌────────────────────────────────────────────────────────────┐
│                      VOTRE PC                              │
│                                                            │
│  📁 mon-projet-docker/                                     │
│     └── index.php ─────────────┐                          │
│                                │ COURS 05 : Volume        │
│  🌐 Navigateur Web             │ (-v)                      │
│     http://localhost:8080 ──┐  │                          │
│                             │  │                          │
└─────────────────────────────│──│───────────────────────────┘
                              │  │
                 COURS 04 :   │  │
                 Ports (-p)   │  │
                              │  │
┌─────────────────────────────│──│───────────────────────────┐
│                    DOCKER   │  │                           │
│                             ▼  ▼                           │
│  ┌────────────────────────────────────────────────┐        │
│  │   COURS 06 : Réseau "mon-reseau"               │        │
│  │                                                 │        │
│  │   ┌──────────────────┐    ┌─────────────────┐ │        │
│  │   │ mon-site-php     │◄──►│ ma-base-de-     │ │        │
│  │   │                  │    │ donnees         │ │        │
│  │   │ PHP + Apache     │    │                 │ │        │
│  │   │                  │    │ MariaDB         │ │        │
│  │   │ Port 80          │    │                 │ │        │
│  │   │                  │    │ COURS 07 :      │ │        │
│  │   │ /var/www/html/   │    │ Variables -e    │ │        │
│  │   │ └─ index.php     │────┤ • MYSQL_USER    │ │        │
│  │   │                  │    │ • MYSQL_PASS    │ │        │
│  │   └──────────────────┘    └─────────────────┘ │        │
│  │                                                 │        │
│  └─────────────────────────────────────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ❓ QUESTIONS FRÉQUENTES

**Q1 : Pourquoi mon fichier index.php apparaît dans le conteneur ?**
→ Grâce au montage de volume `-v $(pwd):/var/www/html` (Cours 05)
→ Docker crée un "lien magique" entre votre dossier et le conteneur

**Q2 : Pourquoi PHP peut se connecter à MariaDB avec juste le nom ?**
→ Grâce au réseau Docker (Cours 06) !
→ Dans le réseau, chaque conteneur a un "nom de domaine" = son nom
→ PHP dit "ma-base-de-donnees" → Docker trouve automatiquement l'adresse IP !

**Q3 : Pourquoi il faut installer pdo/pdo_mysql ?**
→ L'image PHP de base est "légère" → pas d'extensions par défaut
→ Ça permet de garder l'image petite et de choisir ce dont vous avez besoin

**Q4 : Si je modifie index.php, est-ce que je dois redémarrer le conteneur ?**
→ **NON !** C'est ça la magie du volume (Cours 05) !
→ Modification → Enregistrement → Actualisation du navigateur → Changement visible !

**Q5 : Pourquoi localhost:8080 et pas localhost:80 ?**
→ Le port 80 est souvent déjà utilisé sur votre PC
→ 8080 est un port "libre" et une convention pour le développement (Cours 04)

---

## ❌ TROUBLESHOOTING : Erreurs fréquentes

### Erreur 1 : "Connection refused" dans le navigateur

**Message dans le navigateur :** "Impossible de se connecter"

**Cause possible :** Le conteneur PHP ne tourne pas

**Solution :**
```bash
docker ps
```
→ Vérifiez que `mon-site-php` apparaît
→ Si non, relancez-le avec la commande de l'Étape 4

---

### Erreur 2 : "SQLSTATE[HY000] [2002] Connection refused"

**Message dans le navigateur :** Erreur de connexion à MariaDB

**Causes possibles :**

1. MariaDB ne tourne pas
   ```bash
   docker ps
   ```
   → Vérifiez que `ma-base-de-donnees` apparaît

2. Les conteneurs ne sont pas dans le même réseau
   ```bash
   docker inspect mon-site-php | grep NetworkMode
   docker inspect ma-base-de-donnees | grep NetworkMode
   ```
   → Les deux doivent montrer `mon-reseau`

3. MariaDB n'a pas fini de démarrer
   → Attendez 10 secondes et réessayez

---

### Erreur 3 : "SQLSTATE[HY000] [2002] php_network_getaddresses: getaddrinfo failed: Name or service not known"

**Message :** PHP ne trouve pas MariaDB

**Cause :** Le nom du conteneur MariaDB ne correspond pas

**Solution :**
- Dans `index.php`, vérifiez la ligne 24 : `$host = 'ma-base-de-donnees';`
- Ce nom DOIT correspondre au `--name` utilisé pour MariaDB
- Vérifiez le vrai nom :
  ```bash
  docker ps
  ```
- Si le nom est différent, modifiez `index.php` ou relancez MariaDB avec le bon nom

---

### Erreur 4 : La page est blanche

**Cause possible :** Le fichier `index.php` n'est pas monté correctement

**Solution :**

1. Vérifiez que vous étiez dans `mon-projet-docker` quand vous avez lancé le conteneur
   ```bash
   docker inspect mon-site-php | grep -A 10 Mounts
   ```
   → Vous devez voir votre dossier monté sur `/var/www/html`

2. Vérifiez que `index.php` existe
   ```bash
   ls ~/mon-projet-docker/
   ```
   → Vous devez voir `index.php`

3. Si besoin, supprimez et relancez le conteneur PHP
   ```bash
   docker rm -f mon-site-php
   cd ~/mon-projet-docker
   pwd  # Vérifiez que vous êtes au bon endroit !
   # Relancez la commande de l'Étape 4
   ```

---

### Erreur 5 : "Call to undefined function PDO::__construct()"

**Cause :** Les extensions PHP (pdo, pdo_mysql) ne sont pas installées

**Solution :**
```bash
docker exec mon-site-php docker-php-ext-install pdo pdo_mysql
docker restart mon-site-php
```

---

## ✅ QUIZ FINAL

**Question 1 : Quel concept permet d'accéder au site sur localhost:8080 ?**
- A) Les volumes
- B) Les réseaux
- C) Les ports
- D) Les variables d'environnement

<details>
<summary>Voir la réponse</summary>
✅ **C) Les ports** (Cours 04)
Explication : `-p 8080:80` crée un tunnel entre votre PC et le conteneur
</details>

---

**Question 2 : Quel concept permet de modifier index.php en temps réel ?**
- A) Les ports
- B) Les volumes
- C) Les réseaux
- D) Les variables d'environnement

<details>
<summary>Voir la réponse</summary>
✅ **B) Les volumes** (Cours 05)
Explication : `-v $(pwd):/var/www/html` monte votre dossier dans le conteneur
</details>

---

**Question 3 : Quel concept permet à PHP de parler à MariaDB ?**
- A) Les ports
- B) Les volumes
- C) Les réseaux
- D) Les variables d'environnement

<details>
<summary>Voir la réponse</summary>
✅ **C) Les réseaux** (Cours 06)
Explication : `--network mon-reseau` met les deux conteneurs dans le même réseau
</details>

---

**Question 4 : Quel concept permet de configurer le mot de passe MySQL ?**
- A) Les ports
- B) Les volumes
- C) Les réseaux
- D) Les variables d'environnement

<details>
<summary>Voir la réponse</summary>
✅ **D) Les variables d'environnement** (Cours 07)
Explication : `-e MYSQL_PASSWORD=...` configure MariaDB sans modifier le code
</details>

---

**Question 5 : Dans quel ordre faut-il lancer les conteneurs ?**
- A) PHP d'abord, puis MariaDB
- B) MariaDB d'abord, puis PHP
- C) Peu importe, ils sont indépendants

<details>
<summary>Voir la réponse</summary>
✅ **B) MariaDB d'abord, puis PHP**
Explication : PHP va essayer de se connecter à MariaDB au démarrage. Si MariaDB n'est pas là, ça va échouer (mais dans notre cas, PHP retente automatiquement)
</details>

---

## 🎯 EXERCICES SUPPLÉMENTAIRES

### Exercice 1 : Changer le port d'accès

**Objectif :** Accéder au site sur `localhost:9000` au lieu de `localhost:8080`

**Instructions :**
1. Arrêtez et supprimez le conteneur PHP
2. Relancez-le en modifiant la commande pour utiliser le port 9000
3. Testez dans le navigateur

<details>
<summary>Voir la solution</summary>

```bash
docker rm -f mon-site-php
docker run -d \
  --name mon-site-php \
  --network mon-reseau \
  -p 9000:80 \
  -v $(pwd):/var/www/html \
  php:8.2-apache
```

**Accédez à :** `http://localhost:9000`
</details>

---

### Exercice 2 : Changer les identifiants MariaDB

**Objectif :** Utiliser un utilisateur `admin` avec le mot de passe `secret456`

**Instructions :**
1. Arrêtez et supprimez les deux conteneurs
2. Relancez MariaDB avec les nouvelles variables
3. Modifiez `index.php` pour utiliser les nouveaux identifiants
4. Relancez PHP
5. Testez

<details>
<summary>Voir la solution</summary>

**1. Arrêter et supprimer :**
```bash
docker rm -f mon-site-php ma-base-de-donnees
```

**2. Relancer MariaDB :**
```bash
docker run -d \
  --name ma-base-de-donnees \
  --network mon-reseau \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=ma_base \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=secret456 \
  mariadb:latest
```

**3. Modifier `index.php` (lignes 25-26) :**
```php
$user = 'admin';
$pass = 'secret456';
```

**4. Relancer PHP :**
```bash
docker run -d \
  --name mon-site-php \
  --network mon-reseau \
  -p 8080:80 \
  -v $(pwd):/var/www/html \
  php:8.2-apache

docker exec mon-site-php docker-php-ext-install pdo pdo_mysql
docker restart mon-site-php
```

**5. Tester :** `http://localhost:8080`
</details>

---

### Exercice 3 : Ajouter un compteur de visites total

**Objectif :** Afficher le nombre total de visiteurs dans la base

**Instructions :**
1. Modifiez `index.php` pour ajouter une requête SQL qui compte le nombre total de visiteurs
2. Affichez ce nombre sur la page

<details>
<summary>Voir la solution</summary>

**Ajoutez ce code dans `index.php` après la ligne 61 :**

```php
// Compter le nombre total de visiteurs
$stmt = $connexion->query("SELECT COUNT(*) as total FROM visiteurs");
$total = $stmt->fetch(PDO::FETCH_ASSOC);

echo '<div class="box">';
echo '<h2>📊 Statistiques</h2>';
echo "<p><strong>Nombre total de visiteurs :</strong> {$total['total']}</p>";
echo '</div>';
```

**Enregistrez et actualisez le navigateur.**
</details>

---

## 🚀 CONCLUSION

### Ce que vous avez accompli

**BRAVO !** Vous venez de créer une application web complète avec Docker !

**Vous avez utilisé :**
- ✅ Les ports (Cours 04)
- ✅ Les volumes (Cours 05)
- ✅ Les réseaux (Cours 06)
- ✅ Les variables d'environnement (Cours 07)

**Vous êtes maintenant capable de :**
- Faire communiquer plusieurs conteneurs
- Développer en temps réel avec Docker
- Configurer des bases de données
- Créer des applications web complètes

---

### Et maintenant ?

**Dans le prochain cours (Cours 09 : Dockerfile), vous apprendrez :**
- À créer vos PROPRES images Docker
- À automatiser l'installation des extensions PHP
- À personnaliser vos conteneurs

**Mais avant, entraînez-vous bien avec ce projet !**

**Testez les exercices, modifiez le code, expérimentez !**

---

## 📚 AIDE-MÉMOIRE DU PROJET

**Commandes essentielles :**

```bash
# Créer un réseau
docker network create mon-reseau

# Lancer MariaDB
docker run -d --name ma-base-de-donnees --network mon-reseau \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=ma_base \
  -e MYSQL_USER=stephane \
  -e MYSQL_PASSWORD=motdepasse123 \
  mariadb:latest

# Lancer PHP
docker run -d --name mon-site-php --network mon-reseau \
  -p 8080:80 -v $(pwd):/var/www/html php:8.2-apache

# Installer les extensions PHP
docker exec mon-site-php docker-php-ext-install pdo pdo_mysql
docker restart mon-site-php

# Accéder au site
http://localhost:8080

# Arrêter
docker stop mon-site-php ma-base-de-donnees

# Nettoyer
docker rm mon-site-php ma-base-de-donnees
docker network rm mon-reseau
```

---

**FÉLICITATIONS ! Vous êtes maintenant un vrai développeur Docker !** 🏆🐳

**N'oubliez pas : Ce cours combine TOUS les concepts. Si vous l'avez compris, vous maîtrisez les bases de Docker !**
