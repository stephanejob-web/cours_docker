# Cours 7 : Variables d'Environnement - Configurer vos Conteneurs ⚙️

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- C'est quoi une variable d'environnement (expliqué simplement)
- Pourquoi on NE DOIT PAS mettre les mots de passe en dur dans le code
- Comment passer des configurations avec `-e`
- Configurer MySQL, PostgreSQL et d'autres services
- Utiliser des fichiers `.env` pour gérer plein de variables
- Les bonnes pratiques de sécurité

**Durée : 1 heure (lecture + pratique)**

**Prérequis : Cours 04 (Les Ports) + Cours 05 (Les Volumes) + Cours 06 (Les Réseaux)**

---

## 😱 Partie 1 : LE PROBLÈME - Les mots de passe en dur

### 🎬 Scène 1 : Le code qui fuite

**Vous (l'étudiant) :** "Chef ! J'ai fini mon application !"

**Le formateur :** "Super ! Montre-moi le code !"

**Vous montrez votre fichier `config.php` :**

```php
<?php
$host = 'localhost';
$user = 'admin';
$password = 'MotDePasseSuper123!';  // ← OH NON ! 😱
$database = 'ma_base';
?>
```

**Le formateur :** "STOP ! Tu as mis le mot de passe EN DUR dans le code ?!"

**Vous :** "Euh... oui, pourquoi ?"

**Le formateur :** "3 GROS problèmes !"

---

### ❌ Les 3 problèmes du mot de passe en dur

**Problème 1 : Sécurité**

```
Vous : "Je vais mettre mon code sur GitHub !"
       ▼
Le monde entier peut voir votre mot de passe ! 😱
       ▼
Des pirates trouvent votre code
       ▼
Ils ont le mot de passe de votre base de données !
       ▼
💥 Vous êtes hacké !
```

**Problème 2 : Flexibilité**

```
En développement :
├── Mot de passe : "dev123"
├── Base de données : "localhost"

En production :
├── Mot de passe : "Pr0d_S3cur3_P@ss!"
├── Base de données : "db.production.com"

❌ Il faut MODIFIER le code à chaque fois !
❌ Risque d'erreur (oublier de changer)
❌ Compliqué à gérer
```

**Problème 3 : Travail en équipe**

```
Développeur 1 : Mot de passe MySQL = "pass1"
Développeur 2 : Mot de passe MySQL = "pass2"
Développeur 3 : Mot de passe MySQL = "pass3"

❌ Le code de Dev1 ne marche pas chez Dev2 !
❌ Conflits Git constants
❌ Perte de temps
```

**Il faut une MEILLEURE solution !** 💡

---

## 🤔 Partie 2 : COMPRENDRE LA SOLUTION

### C'est quoi une variable d'environnement ?

**Analogie simple : Les réglages de votre four**

Imaginez que vous cuisinez une pizza :

```
┌─────────────────────────────────┐
│         VOTRE FOUR              │
│                                 │
│  Réglages (variables) :         │
│  ├── Température : 220°C        │
│  ├── Mode : Chaleur tournante   │
│  └── Durée : 15 minutes         │
│                                 │
│  Vous ne CHANGEZ PAS le four !  │
│  Vous changez les RÉGLAGES !    │
└─────────────────────────────────┘
```

**Avec Docker, c'est pareil :**

```
┌─────────────────────────────────┐
│      VOTRE CONTENEUR            │
│                                 │
│  Variables d'environnement :    │
│  ├── DB_PASSWORD=secret123      │
│  ├── DB_HOST=ma-db              │
│  └── DEBUG_MODE=true            │
│                                 │
│  Vous ne CHANGEZ PAS le code !  │
│  Vous changez les VARIABLES !   │
└─────────────────────────────────┘
```

**Avantages :**
- ✅ Le code reste le même
- ✅ Chaque personne/environnement a ses propres réglages
- ✅ Pas de mot de passe dans le code
- ✅ Facile à changer

---

### Comment ça marche ?

**Dans votre code (PHP, Node, Python, etc.) :**

**❌ AVANT (mot de passe en dur) :**
```php
$password = 'MotDePasseSuper123!';  // Dangereux !
```

**✅ APRÈS (variable d'environnement) :**
```php
$password = getenv('DB_PASSWORD');  // Lit la variable d'environnement !
```

**Au lancement du conteneur :**
```bash
docker run -e DB_PASSWORD=MotDePasseSuper123! mon-app
```

**Le code lit la variable `DB_PASSWORD` fournie par Docker !**

---

## 🚀 Partie 3 : Utiliser `-e` avec Docker

### La syntaxe de base

**Format général :**
```bash
docker run -e NOM_VARIABLE=valeur mon-image
```

**Exemple concret :**
```bash
docker run -e DB_PASSWORD=secret123 nginx
```

**Ce qui se passe :**
1. Docker crée la variable `DB_PASSWORD`
2. Il lui donne la valeur `secret123`
3. Le conteneur peut lire cette variable avec son code

---

### Exemple 1 : MySQL avec mot de passe

**Lancer MySQL SANS mot de passe (ne marche pas) :**

```bash
docker run -d --name ma-db mysql:8.0
```

**💥 ERREUR !**

```bash
docker logs ma-db
```

```
ERROR: Database is uninitialized and password option is not specified
You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD...
```

**MySQL dit :** "Je veux un mot de passe !"

---

**✅ LA BONNE MÉTHODE : Avec `-e`**

```bash
docker run -d \
  --name ma-db \
  -e MYSQL_ROOT_PASSWORD=monmotdepasse \
  mysql:8.0
```

**Maintenant ça marche ! MySQL a son mot de passe ! 🎉**

---

### Passer plusieurs variables

**Vous pouvez passer autant de variables que vous voulez :**

```bash
docker run -d \
  --name ma-db \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=ma_base \
  -e MYSQL_USER=stephane \
  -e MYSQL_PASSWORD=userpass \
  mysql:8.0
```

**Ce qui se passe :**
- Variable `MYSQL_ROOT_PASSWORD` = `rootpass`
- Variable `MYSQL_DATABASE` = `ma_base` (MySQL crée cette base automatiquement !)
- Variable `MYSQL_USER` = `stephane` (MySQL crée cet utilisateur !)
- Variable `MYSQL_PASSWORD` = `userpass`

**MySQL lit ces variables au démarrage et se configure automatiquement !**

---

## 💪 Partie 4 : EXERCICES PRATIQUES

### ✏️ Exercice 1 : MySQL avec configuration complète

**Mission : Lancer MySQL avec utilisateur et base de données**

**Étape 1 : Lancez MySQL avec les variables**

```bash
docker run -d \
  --name test-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass123 \
  -e MYSQL_DATABASE=blog \
  -e MYSQL_USER=john \
  -e MYSQL_PASSWORD=johnpass456 \
  mysql:8.0
```

**Attendez 10 secondes** que MySQL démarre.

**Étape 2 : Vérifiez les logs**

```bash
docker logs test-mysql
```

Cherchez ces lignes :
```
[Server] /usr/sbin/mysqld: ready for connections.
```

✅ **MySQL est prêt !**

**Étape 3 : Connectez-vous avec l'utilisateur créé**

```bash
docker exec -it test-mysql mysql -u john -pjohnpass456 blog
```

**Vous êtes connecté ! Tapez :**

```sql
SHOW DATABASES;
```

**Résultat :**
```
+--------------------+
| Database           |
+--------------------+
| blog               | ← La base créée automatiquement !
| information_schema |
+--------------------+
```

**Tapez :**
```sql
SELECT USER();
```

**Résultat :**
```
+----------------+
| USER()         |
+----------------+
| john@localhost | ← L'utilisateur créé automatiquement !
+----------------+
```

**🎉 Tout a été configuré grâce aux variables d'environnement !**

**Sortez :**
```sql
exit
```

**Nettoyez :**
```bash
docker rm -f test-mysql
```

---

### ✏️ Exercice 2 : PostgreSQL avec variables

**Mission : Lancer PostgreSQL (autre base de données)**

**PostgreSQL utilise des noms de variables différents :**

```bash
docker run -d \
  --name test-postgres \
  -e POSTGRES_PASSWORD=postgrespass \
  -e POSTGRES_USER=alice \
  -e POSTGRES_DB=shop \
  postgres:16
```

**Attendez 5 secondes.**

**Connectez-vous :**

```bash
docker exec -it test-postgres psql -U alice -d shop
```

**Vous êtes dans PostgreSQL ! Tapez :**

```sql
\l
```

**Vous verrez la base `shop` !**

**Tapez :**
```sql
\q
```

**Nettoyez :**
```bash
docker rm -f test-postgres
```

**✅ Vous savez maintenant configurer MySQL ET PostgreSQL !**

---

### ✏️ Exercice 3 : Application Node.js avec variables

**Mission : Créer une application qui lit des variables d'environnement**

**Étape 1 : Créez un dossier**

```bash
mkdir test-env-app
cd test-env-app
```

**Étape 2 : Créez un fichier `app.js`**

```bash
cat > app.js << 'EOF'
console.log("=== Variables d'environnement ===");
console.log("APP_NAME:", process.env.APP_NAME || "Non définie");
console.log("APP_VERSION:", process.env.APP_VERSION || "Non définie");
console.log("DEBUG_MODE:", process.env.DEBUG_MODE || "Non définie");
console.log("================================");
EOF
```

**Étape 3 : Lancez avec Node.js**

```bash
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  -e APP_NAME="Mon Super App" \
  -e APP_VERSION="1.0.0" \
  -e DEBUG_MODE="true" \
  node:20 node app.js
```

**Résultat :**
```
=== Variables d'environnement ===
APP_NAME: Mon Super App
APP_VERSION: 1.0.0
DEBUG_MODE: true
================================
```

**🎉 L'application lit les variables passées avec `-e` !**

**Étape 4 : Testez SANS variables**

```bash
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:20 node app.js
```

**Résultat :**
```
=== Variables d'environnement ===
APP_NAME: Non définie
APP_VERSION: Non définie
DEBUG_MODE: Non définie
================================
```

**Sans `-e`, les variables n'existent pas !**

**Nettoyez :**
```bash
cd ..
rm -rf test-env-app
```

---

## 📁 Partie 5 : Fichiers `.env` (niveau avancé)

### Le problème avec beaucoup de variables

**Imaginez que vous avez 15 variables :**

```bash
docker run -d \
  -e VAR1=value1 \
  -e VAR2=value2 \
  -e VAR3=value3 \
  -e VAR4=value4 \
  -e VAR5=value5 \
  -e VAR6=value6 \
  -e VAR7=value7 \
  -e VAR8=value8 \
  -e VAR9=value9 \
  -e VAR10=value10 \
  mon-image
```

**❌ C'est HORRIBLE ! Trop long ! Illisible !**

---

### La solution : Fichier `.env`

**Un fichier `.env` = Un fichier avec toutes vos variables**

**Créez un fichier `.env` :**

```bash
cat > .env << 'EOF'
MYSQL_ROOT_PASSWORD=rootpass123
MYSQL_DATABASE=blog
MYSQL_USER=john
MYSQL_PASSWORD=johnpass456
EOF
```

**Puis lancez avec `--env-file` :**

```bash
docker run -d \
  --name ma-db \
  --env-file .env \
  mysql:8.0
```

**Docker lit TOUTES les variables du fichier `.env` automatiquement !**

**Beaucoup plus propre ! ✨**

---

### Exemple complet avec fichier `.env`

**Étape 1 : Créez `.env`**

```bash
cat > .env << 'EOF'
# Configuration de la base de données
DB_ROOT_PASSWORD=supersecret123
DB_NAME=mon_blog
DB_USER=blogger
DB_PASSWORD=blogpass456

# Configuration de l'application
APP_ENV=development
APP_DEBUG=true
APP_PORT=3000
EOF
```

**Étape 2 : Utilisez-le**

```bash
docker run -d \
  --name ma-db \
  --env-file .env \
  mysql:8.0
```

**Tous les paramètres sont chargés depuis le fichier !**

---

### ⚠️ SÉCURITÉ : NE JAMAIS committer `.env` sur Git !

**Le fichier `.env` contient des SECRETS !**

**Créez un `.gitignore` :**

```bash
cat > .gitignore << 'EOF'
.env
*.env
.env.local
EOF
```

**Maintenant Git ignore le fichier `.env` !**

**À la place, créez un `.env.example` (SANS les vrais mots de passe) :**

```bash
cat > .env.example << 'EOF'
# Configuration de la base de données
DB_ROOT_PASSWORD=changeme
DB_NAME=mon_blog
DB_USER=blogger
DB_PASSWORD=changeme

# Configuration de l'application
APP_ENV=development
APP_DEBUG=true
APP_PORT=3000
EOF
```

**Les autres développeurs copient `.env.example` en `.env` et mettent leurs propres valeurs !**

---

## 🎓 Partie 6 : Variables d'environnement courantes

### MySQL / MariaDB

| Variable | Description | Exemple |
|----------|-------------|---------|
| `MYSQL_ROOT_PASSWORD` | Mot de passe root (OBLIGATOIRE) | `rootpass123` |
| `MYSQL_DATABASE` | Nom de la base à créer | `blog` |
| `MYSQL_USER` | Utilisateur à créer | `john` |
| `MYSQL_PASSWORD` | Mot de passe de l'utilisateur | `userpass456` |
| `MYSQL_ALLOW_EMPTY_PASSWORD` | Autoriser mot de passe vide (dangereux !) | `yes` |

---

### PostgreSQL

| Variable | Description | Exemple |
|----------|-------------|---------|
| `POSTGRES_PASSWORD` | Mot de passe (OBLIGATOIRE) | `postgrespass` |
| `POSTGRES_USER` | Utilisateur | `alice` |
| `POSTGRES_DB` | Nom de la base | `shop` |

---

### MongoDB

| Variable | Description | Exemple |
|----------|-------------|---------|
| `MONGO_INITDB_ROOT_USERNAME` | Utilisateur admin | `admin` |
| `MONGO_INITDB_ROOT_PASSWORD` | Mot de passe admin | `adminpass` |
| `MONGO_INITDB_DATABASE` | Base de données initiale | `mydb` |

---

### Redis

| Variable | Description | Exemple |
|----------|-------------|---------|
| `REDIS_PASSWORD` | Mot de passe | `redispass` |

---

### WordPress

| Variable | Description | Exemple |
|----------|-------------|---------|
| `WORDPRESS_DB_HOST` | Hôte de la base de données | `ma-db:3306` |
| `WORDPRESS_DB_USER` | Utilisateur MySQL | `wpuser` |
| `WORDPRESS_DB_PASSWORD` | Mot de passe MySQL | `wppass` |
| `WORDPRESS_DB_NAME` | Nom de la base | `wordpress` |

---

## 📋 Partie 7 : AIDE-MÉMOIRE

### Les commandes essentielles

| Commande | Ce que ça fait |
|----------|----------------|
| `docker run -e VAR=value image` | Passe une variable d'environnement |
| `docker run --env-file .env image` | Charge les variables depuis un fichier |
| `docker exec conteneur env` | Voir toutes les variables d'un conteneur |
| `docker inspect conteneur` | Voir la config complète (y compris les variables) |

---

### Workflow typique

**1. Créez un fichier `.env`**
```bash
cat > .env << 'EOF'
DB_PASSWORD=secret123
APP_ENV=development
EOF
```

**2. Ajoutez-le au `.gitignore`**
```bash
echo ".env" >> .gitignore
```

**3. Créez un `.env.example` pour l'équipe**
```bash
cat > .env.example << 'EOF'
DB_PASSWORD=changeme
APP_ENV=development
EOF
```

**4. Utilisez-le avec Docker**
```bash
docker run --env-file .env mon-image
```

---

## ❌ Partie 8 : Les erreurs fréquentes

### Erreur 1 : Oublier `-e` ou `--env-file`

**Votre application dit :**
```
Error: DB_PASSWORD is not defined
```

**Problème :** Vous avez oublié de passer la variable !

**Solution :**
```bash
docker run -e DB_PASSWORD=secret123 mon-app
```

---

### Erreur 2 : Espaces dans les valeurs

**❌ MAUVAIS :**
```bash
docker run -e APP_NAME=Mon Super App nginx
```

**Docker pense que `Super` et `App` sont des arguments séparés !**

**✅ BON (avec guillemets) :**
```bash
docker run -e APP_NAME="Mon Super App" nginx
```

---

### Erreur 3 : Committer `.env` sur Git

**💥 DANGER ! Vos secrets sont publics !**

**Solution préventive :**

```bash
# Toujours créer .gitignore AVANT de committer
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add .gitignore"

# Puis créer .env
cat > .env << 'EOF'
DB_PASSWORD=secret123
EOF
```

**Si déjà commis par erreur :**

```bash
# Supprimer du Git (mais garder localement)
git rm --cached .env
git commit -m "Remove .env from git"

# Ajouter au .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Ignore .env"

# CHANGER TOUS LES MOTS DE PASSE ! Ils sont dans l'historique Git !
```

---

### Erreur 4 : Noms de variables incorrects

**Chaque image a SES PROPRES noms de variables !**

**❌ MAUVAIS (MySQL) :**
```bash
docker run -e PASSWORD=secret123 mysql:8.0
```

**✅ BON (MySQL) :**
```bash
docker run -e MYSQL_ROOT_PASSWORD=secret123 mysql:8.0
```

**Comment savoir les bons noms ?**

Allez sur Docker Hub et cherchez l'image :
- [hub.docker.com/\_/mysql](https://hub.docker.com/_/mysql)
- [hub.docker.com/\_/postgres](https://hub.docker.com/_/postgres)
- etc.

La documentation liste toutes les variables !

---

## ✅ Quiz final

**Question 1 : Pourquoi on ne met PAS les mots de passe en dur dans le code ?**

<details>
<summary>Voir la réponse</summary>

**3 raisons :**
1. **Sécurité** - Si le code fuite (GitHub), les secrets sont exposés
2. **Flexibilité** - Changer de mot de passe = modifier le code (pénible)
3. **Équipe** - Chaque dev a ses propres mots de passe locaux
</details>

---

**Question 2 : Comment passer une variable d'environnement à un conteneur ?**
- A) Dans le code
- B) Avec `-e NOM=valeur`
- C) Dans un fichier de config

<details>
<summary>Voir la réponse</summary>
✅ **B) Avec `-e NOM=valeur`**

```bash
docker run -e DB_PASSWORD=secret123 mon-app
```
</details>

---

**Question 3 : Comment passer 10 variables facilement ?**

<details>
<summary>Voir la réponse</summary>

**Avec un fichier `.env` :**

```bash
# Créer .env
cat > .env << 'EOF'
VAR1=value1
VAR2=value2
...
EOF

# Utiliser
docker run --env-file .env mon-app
```
</details>

---

**Question 4 : Doit-on committer `.env` sur Git ?**

<details>
<summary>Voir la réponse</summary>

**❌ NON ! JAMAIS !**

Le fichier `.env` contient des secrets !

**À faire :**
1. Ajouter `.env` au `.gitignore`
2. Créer `.env.example` (sans les vrais mots de passe)
3. Committer `.env.example` et `.gitignore`
</details>

---

## 🎯 Récapitulatif final

### Ce que vous avez appris aujourd'hui :

✅ Pourquoi on ne met PAS les mots de passe en dur
✅ C'est quoi une variable d'environnement
✅ Passer des variables avec `-e`
✅ Configurer MySQL, PostgreSQL avec des variables
✅ Utiliser des fichiers `.env` pour gérer plein de variables
✅ Les bonnes pratiques de sécurité (`.gitignore`)

### La règle d'or :

**Les secrets ne vont JAMAIS dans le code, toujours dans les variables d'environnement !**

```bash
# ❌ MAUVAIS
$password = 'secret123';

# ✅ BON
$password = getenv('DB_PASSWORD');
```

**Et au lancement :**
```bash
docker run -e DB_PASSWORD=secret123 mon-app
```

---

## 🚀 Et maintenant ?

**FÉLICITATIONS ! Vous avez terminé les 4 concepts essentiels !** 🎉

✅ **Cours 04 : Les Ports** → Accéder aux conteneurs
✅ **Cours 05 : Les Volumes** → Persister les données
✅ **Cours 06 : Les Réseaux** → Connecter les conteneurs
✅ **Cours 07 : Variables d'Environnement** → Configurer les conteneurs

**Dans le prochain cours (Cours 8), vous allez TOUT COMBINER !**

📘 **PROJET COMPLET : Application PHP + MariaDB**

**Vous utiliserez :**
- `-p` pour accéder au site
- `-v` pour modifier le code en temps réel
- `--network` pour que PHP parle à MariaDB
- `-e` pour configurer les mots de passe

**C'est LE projet qui met tout ensemble !** 🚀

---

**Aide-mémoire ultra-court :**
```bash
docker run -e VAR=value image              # Passer une variable
docker run --env-file .env image           # Charger depuis un fichier
docker exec conteneur env                  # Voir les variables
echo ".env" >> .gitignore                  # Sécurité !
```

**À bientôt pour le grand projet ! 🐳**
