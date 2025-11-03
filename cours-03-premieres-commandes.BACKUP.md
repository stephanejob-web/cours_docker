# Cours 3 : Vos Premières Commandes Docker 🚀

## 🎯 Ce que vous allez apprendre (en français simple !)

À la fin de ce cours, vous saurez :
- Vérifier que Docker marche sur votre PC
- Télécharger des "images" (comme télécharger une application)
- Lancer un "conteneur" (faire tourner cette application)
- Les commandes de base pour gérer tout ça

**Temps nécessaire : 30 minutes**

**Note importante :** Lisez TOUT le cours AVANT de taper les commandes. Ne tapez pas au hasard ! 😊

---

## 🎬 AVANT DE COMMENCER : Ouvrir un terminal

**Le terminal, c'est la fenêtre où on tape des commandes pour parler à Docker.**

---

### 🖥️ Ouvrir le Terminal

**La plupart d'entre vous êtes sur Ubuntu/Linux, voici comment ouvrir le terminal :**

**Méthode 1 : Avec le clavier (SUPER RAPIDE)** ⭐
```
Appuyez sur : Ctrl + Alt + T
```
→ Boom ! Le terminal s'ouvre ! 🚀

**Méthode 2 : Avec le menu Applications**
1. Cliquez sur "Activités" (en haut à gauche)
2. Tapez "Terminal"
3. Cliquez sur l'icône du Terminal

**Ce que vous devriez voir :**
```
votre-nom@votre-pc:~$
```

✅ **Vous voyez quelque chose comme ça ? Parfait ! Vous pouvez commencer !**

---

**💡 Conseil :** Gardez cette fenêtre de terminal ouverte pendant TOUT le cours ! Vous allez en avoir besoin ! 😊

---

✅ **Votre terminal est ouvert ? Parfait ! On peut commencer le cours !**

---

## ✅ Partie 1 : Est-ce que Docker est bien installé ?

**Sur Ubuntu/Linux, Docker tourne comme un service en arrière-plan. Pas besoin d'application graphique !**

---

### Commande 1 : Vérifier la version

**TAPEZ CETTE COMMANDE** (appuyez sur Entrée après) :
```bash
docker --version
```

**Si ça marche, vous verrez :**
```
Docker version 24.0.7, build afdd53b
```
(Le numéro peut être différent, c'est pas grave)

✅ **Vous voyez "Docker version..." ?** → Super, Docker est installé !  
❌ **Vous voyez "command not found" ?** → Docker n'est pas installé. Appelez votre formateur !

---

### Commande 2 : Le test "Hello World"

**C'est comme le "Allô" quand on teste un micro !** 🎤

**TAPEZ ÇA :**
```bash
docker run hello-world
```

**Ce qui va se passer (étape par étape) :**

**1. Docker cherche "hello-world" sur votre PC**
```
Unable to find image 'hello-world:latest' locally
```
→ Traduction : "Je cherche sur ton PC... je trouve pas !"

**2. Docker le télécharge depuis Internet**
```
latest: Pulling from library/hello-world
...
Status: Downloaded newer image for hello-world:latest
```
→ Traduction : "Pas grave, je vais le chercher sur Internet... Voilà c'est téléchargé !"

**3. Docker affiche un message de bienvenue**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```
→ Traduction : "Tout marche bien ! Bravo !"

🎉 **Si vous voyez ce message → FÉLICITATIONS ! Docker fonctionne parfaitement !**

---

## 📚 PAUSE EXPLICATION : C'est quoi une "image" ?

**Imaginez que vous voulez cuisiner une pizza.** 🍕

- **L'IMAGE** = C'est la RECETTE de la pizza (liste des ingrédients + instructions)
- **LE CONTENEUR** = C'est la PIZZA CUITE que vous mangez

**Avec Docker c'est pareil :**
- **L'IMAGE** = La recette du logiciel (exemple : Ubuntu, Nginx, MySQL)
- **LE CONTENEUR** = Le logiciel qui tourne vraiment

**Donc quand vous faites `docker run`, vous :**
1. Prenez la recette (l'image)
2. Cuisinez la pizza (créez le conteneur)
3. Mangez la pizza (utilisez le conteneur)

**Simple non ?** 😊

---

## 📥 Partie 2 : Télécharger des images

### La commande magique : docker pull

**"Pull" en anglais = "Tirer" → On "tire" l'image depuis Internet**

**Syntaxe :**
```bash
docker pull nom-de-limage
```

---

### EXEMPLE 1 : Télécharger Ubuntu

**TAPEZ ÇA :**
```bash
docker pull ubuntu
```

**Ce que vous allez voir :**
```
Using default tag: latest
latest: Pulling from library/ubuntu
d25f557d7f31: Pull complete
Digest: sha256:abc123def456...
Status: Downloaded newer image for ubuntu:latest
```

**Explications ligne par ligne :**

| Ce que vous voyez | Ce que ça veut dire |
|-------------------|---------------------|
| `Using default tag: latest` | On prend la version la plus récente |
| `Pulling from library/ubuntu` | On télécharge depuis le magasin officiel |
| `Pull complete` | Téléchargement terminé ! |
| `Status: Downloaded` | C'est dans ton PC maintenant ! |

⏱️ **Ça prend 30 secondes environ** (selon votre connexion Internet)

---

### EXEMPLE 2 : Télécharger Nginx (un serveur web)

**TAPEZ ÇA :**
```bash
docker pull nginx
```

**Pareil, ça télécharge !** Attendez que ça dise "Downloaded".

---

### Voir toutes vos images téléchargées

**TAPEZ ÇA :**
```bash
docker images
```

**Vous allez voir un tableau comme ça :**
```
REPOSITORY       TAG       IMAGE ID       CREATED        SIZE
nginx            latest    b619c34a163a   15 hours ago   225MB
ubuntu           latest    66460d557b25   4 weeks ago    117MB
hello-world      latest    d2c94e258dcb   8 months ago   13.3kB
```

**Lecture du tableau :**

| Colonne | Ça veut dire quoi ? | Exemple |
|---------|---------------------|---------|
| **REPOSITORY** | Le nom de l'image | nginx, ubuntu |
| **TAG** | La version | latest = la plus récente |
| **SIZE** | Combien ça pèse | Ubuntu = 117 Mo |
| **CREATED** | Quand elle a été créée | Il y a 4 semaines |

**C'est comme voir les applications installées sur votre téléphone !** 📱

---

## 🎮 Partie 3 : Lancer un conteneur

### La commande SUPER IMPORTANTE : docker run

**"Run" en anglais = "Exécuter" → On fait tourner le logiciel**

---

### ❌ ERREUR CLASSIQUE : Lancer Ubuntu (et se demander pourquoi ça marche pas)

**Si vous tapez ça :**
```bash
docker run ubuntu
```

**Ce qui se passe :**
```
1. Docker crée un conteneur Ubuntu
2. Il démarre
3. Il s'arrête IMMÉDIATEMENT
```

**Vous vous dites : "Hein ?? Pourquoi il s'arrête direct ??"** 🤔

**Explication simple :**
Un conteneur, c'est comme une personne. Si on lui donne rien à faire, elle s'ennuie et elle part ! 😅

Ubuntu démarre, se dit "Y'a rien à faire ici", et s'arrête.

---

### ✅ LA BONNE MÉTHODE : Lancer Ubuntu en mode interactif

**TAPEZ ÇA :**
```bash
docker run -it ubuntu bash
```

**Décomposons cette commande :**

| Partie | Ça veut dire quoi ? |
|--------|---------------------|
| `docker run` | Lance un conteneur |
| `-it` | Mode interactif (je veux taper des commandes dedans) |
| `ubuntu` | L'image à utiliser |
| `bash` | Lance le terminal |

**Ce qui va se passer :**

**AVANT** (votre terminal normal) :
```
votre-nom@votre-pc:~$
```

**APRÈS** (vous êtes DANS le conteneur Ubuntu !) :
```
root@a1b2c3d4e5f6:/#
```

**Regardez bien la différence :**
- `votre-nom@votre-pc` → Vous êtes sur VOTRE PC
- `root@a1b2c3d4e5f6` → Vous êtes DANS le conteneur !

**C'est magique non ?** ✨ Vous avez un Ubuntu dans votre Ubuntu !

---

### 🎪 TESTEZ DES COMMANDES DANS LE CONTENEUR !

**Maintenant que vous êtes dedans, testez ça :**

```bash
pwd
```
→ Affiche où vous êtes (vous êtes dans `/`)

```bash
ls
```
→ Affiche les fichiers (vous voyez les dossiers d'Ubuntu)

```bash
whoami
```
→ Affiche qui vous êtes (vous êtes `root` = le super admin)

```bash
cat /etc/os-release
```
→ Affiche la version d'Ubuntu

**Vous pouvez faire ce que vous voulez ! Créez des fichiers, installez des trucs, cassez tout... c'est PAS votre vrai PC ! C'est un bac à sable !** 🏖️

---

### 🚪 SORTIR du conteneur

**Pour revenir à votre PC normal, tapez :**
```bash
exit
```

**Boom ! Vous êtes de retour sur votre vrai PC !**

Le conteneur s'est arrêté automatiquement quand vous êtes sorti.

---

### 🎬 Lancer un serveur web Nginx EN ARRIÈRE-PLAN

**Maintenant on fait quelque chose de différent : lancer Nginx SANS entrer dedans.**

**TAPEZ ÇA :**
```bash
docker run -d --name mon-serveur-web nginx
```

**Décomposons :**

| Partie | Ça veut dire quoi ? |
|--------|---------------------|
| `docker run` | Lance un conteneur |
| `-d` | Mode "detached" = tourne en fond (comme une musique en arrière-plan) |
| `--name mon-serveur-web` | Donne un nom clair au conteneur |
| `nginx` | L'image à utiliser |

**Ce que vous verrez :**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```

C'est l'ID du conteneur (comme un numéro de série). **Vous n'aurez presque JAMAIS besoin de ce numéro !**

**Ce qu'il faut retenir :** Le conteneur "mon-serveur-web" tourne maintenant en fond ! 🎉

---

## 👀 Partie 4 : Voir ce qui tourne

### La commande : docker ps

**"ps" = "Process Status" = "Qu'est-ce qui tourne en ce moment ?"**

**TAPEZ ÇA :**
```bash
docker ps
```

**Vous allez voir un tableau :**
```
CONTAINER ID   IMAGE   COMMAND                  CREATED          STATUS          PORTS     NAMES
a1b2c3d4e5f6   nginx   "/docker-entrypoint.…"   30 seconds ago   Up 29 seconds   80/tcp    mon-serveur-web
```

**Lecture ligne par ligne :**

| Colonne | Ça veut dire quoi ? | Dans notre exemple |
|---------|---------------------|-------------------|
| **CONTAINER ID** | Le numéro de série | a1b2c3d4e5f6 |
| **IMAGE** | Quelle image est utilisée | nginx |
| **STATUS** | Il tourne depuis combien de temps | Up 29 seconds (il tourne depuis 29 secondes) |
| **NAMES** | Le nom qu'on lui a donné | mon-serveur-web |

**C'est comme un gestionnaire de tâches, mais pour Docker !**

---

### Voir TOUS les conteneurs (même ceux qui sont arrêtés)

**TAPEZ ÇA :**
```bash
docker ps -a
```

**Le `-a` veut dire "all" = "tous"**

**Vous verrez :**
```
CONTAINER ID   IMAGE         STATUS                       NAMES
a1b2c3d4e5f6   nginx         Up 2 minutes                 mon-serveur-web
g7h8i9j0k1l2   ubuntu        Exited (0) 5 minutes ago     competent_gates
m3n4o5p6q7r8   hello-world   Exited (0) 10 minutes ago    hello-test
```

**Les différents STATUS :**

| Status | Ça veut dire quoi ? | Emoji |
|--------|---------------------|-------|
| `Up X minutes` | Il tourne depuis X minutes | ✅ |
| `Exited (0)` | Il s'est arrêté normalement | ⏹️ |
| `Exited (1)` | Il s'est arrêté avec une erreur | ❌ |

---

## 🎮 Partie 5 : Les commandes pour gérer les conteneurs

### 1️⃣ DÉMARRER un conteneur arrêté

**Si un conteneur est arrêté et que vous voulez le relancer :**

```bash
docker start mon-serveur-web
```

**C'est comme appuyer sur "Play" ▶️**

---

### 2️⃣ ARRÊTER un conteneur qui tourne

```bash
docker stop mon-serveur-web
```

**C'est comme appuyer sur "Stop" ⏹️**

⏱️ Ça prend ~10 secondes (Docker demande gentiment au conteneur de s'arrêter)

---

### 3️⃣ REDÉMARRER un conteneur

```bash
docker restart mon-serveur-web
```

**C'est comme faire Stop puis Play, mais en une seule commande !**

---

### 4️⃣ VOIR LES LOGS (ce que le conteneur dit)

**Les logs = Les messages du conteneur**

```bash
docker logs mon-serveur-web
```

**Vous verrez tous les messages que Nginx a affiché.**

**Exemple de logs Nginx :**
```
/docker-entrypoint.sh: Configuration complete
2024/01/15 10:30:25 [notice] 1#1: start worker processes
```

**Suivre les logs EN DIRECT :**
```bash
docker logs -f mon-serveur-web
```

Le `-f` = "follow" = suivre en direct (comme un live)

**Pour arrêter de suivre :** Appuyez sur `Ctrl + C`

---

### 5️⃣ SUPPRIMER un conteneur OU une image (LA DIFFÉRENCE IMPORTANTE !)

**🤔 Quelle est la différence entre un conteneur et une image ?**

Reprenons l'analogie de la pizza 🍕 :

| Chose | C'est quoi ? | Commande pour supprimer |
|-------|--------------|------------------------|
| **IMAGE** | La RECETTE (le plan) | `docker rmi` (remove **image**) |
| **CONTENEUR** | La PIZZA CUITE (ce qui tourne) | `docker rm` (remove) |

**Règle d'or :** Il faut d'abord manger/jeter la pizza (supprimer le conteneur), PUIS on peut jeter la recette (supprimer l'image) !

---

### 5️⃣-A) SUPPRIMER UN CONTENEUR

**⚠️ ATTENTION : Pour supprimer un conteneur, il FAUT d'abord l'arrêter !**

**Méthode 1 : En deux étapes (la méthode propre)**

**Étape 1 : Arrêter le conteneur**
```bash
docker stop mon-serveur-web
```

**Étape 2 : Supprimer le conteneur**
```bash
docker rm mon-serveur-web
```

**Méthode 2 : En une seule fois (mode bourrin)**
```bash
docker rm -f mon-serveur-web
```

Le `-f` = "force" = force la suppression même s'il tourne

**Vérifier que c'est bien supprimé :**
```bash
docker ps -a
```
→ Le conteneur ne devrait plus apparaître !

---

### 5️⃣-B) SUPPRIMER UNE IMAGE

**Pour supprimer une image (comme désinstaller une app) :**

```bash
docker rmi ubuntu
```

**Le "i" dans `rmi` = "image"** (pour pas confondre avec `rm` qui supprime un conteneur)

**⚠️ ERREUR FRÉQUENTE : "image is being used"**

**Si vous voyez ce message :**
```
Error: conflict: unable to remove repository reference "ubuntu"
(must force) - container abc123 is using its referenced image def456
```

**Ça veut dire :** Un conteneur utilise encore cette image !

**Solution : Supprimer d'abord le conteneur, puis l'image**

**1. Trouvez quel conteneur utilise l'image**
```bash
docker ps -a
```

**2. Supprimez le(s) conteneur(s)**
```bash
docker rm nom-du-conteneur
```

**3. Maintenant vous pouvez supprimer l'image**
```bash
docker rmi ubuntu
```

---

### 5️⃣-C) FAIRE LE GRAND MÉNAGE (supprimer plein de trucs d'un coup)

**Supprimer TOUS les conteneurs arrêtés :**
```bash
docker container prune
```
→ Docker va vous demander confirmation (tapez `y` pour Yes)

**Supprimer TOUTES les images non utilisées :**
```bash
docker image prune -a
```

**SUPER MÉNAGE (tout supprimer d'un coup - DANGER !) :**
```bash
docker system prune -a
```
→ Supprime TOUT ce qui ne tourne pas !

**💡 Conseil :** Faites ce ménage régulièrement (une fois par semaine) pour libérer de l'espace disque !

---

### 📋 RÉCAP VISUEL : rm vs rmi

```
┌─────────────────────────────────────────────────┐
│  SUPPRIMER UN CONTENEUR (la pizza cuite 🍕)     │
│  ✅ Commande : docker rm                        │
│  📝 Exemple : docker rm mon-serveur-web         │
└─────────────────────────────────────────────────┘
                    ⬇️  D'abord supprimer ça
┌─────────────────────────────────────────────────┐
│  SUPPRIMER UNE IMAGE (la recette 📄)            │
│  ✅ Commande : docker rmi                       │
│  📝 Exemple : docker rmi ubuntu                 │
└─────────────────────────────────────────────────┘
```

**Moyen mnémotechnique :**
- **rm** = Remove = enlève ce qui **M**arche (le conteneur)
- **rmi** = Remove **I**mage = enlève l'**I**mage (la recette)

---

### 7️⃣ ENTRER dans un conteneur qui tourne

**Pour entrer dans un conteneur déjà lancé :**

```bash
docker exec -it mon-serveur-web bash
```

**Décomposition :**
- `exec` = exécute une commande dans le conteneur
- `-it` = mode interactif
- `mon-serveur-web` = le nom du conteneur
- `bash` = lance le terminal

**Vous êtes maintenant DANS le conteneur ! Faites ce que vous voulez, puis tapez `exit` pour sortir.**

---

### 8️⃣ COPIER des fichiers

**Du conteneur vers votre PC :**
```bash
docker cp mon-serveur-web:/chemin/fichier.txt ./fichier.txt
```

**De votre PC vers le conteneur :**
```bash
docker cp ./fichier.txt mon-serveur-web:/chemin/fichier.txt
```

**Exemple concret :**
```bash
# Récupérer la page d'accueil de Nginx
docker cp mon-serveur-web:/usr/share/nginx/html/index.html ./page.html
```

**📁 NOTE SUR LES CHEMINS DE FICHIERS :**

```bash
# Le point = dossier actuel
./page.html

# Votre dossier personnel
~/Documents/page.html

# Chemin absolu
/home/votre-nom/Documents/page.html
```

---

### 9️⃣ VOIR les statistiques (CPU, RAM...)

```bash
docker stats
```

**Vous verrez :**
```
CONTAINER ID   NAME              CPU %   MEM USAGE / LIMIT
a1b2c3d4e5f6   mon-serveur-web   0.05%   2.5MB / 7.8GB
```

**Traduction :**
- CPU : 0.05% (il utilise presque rien)
- RAM : 2.5 Mo utilisés sur 7.8 Go disponibles

**Pour arrêter l'affichage :** `Ctrl + C`

---

### 🔟 FAIRE LE MÉNAGE

**Supprimer tous les conteneurs arrêtés :**
```bash
docker container prune
```

**Supprimer toutes les images non utilisées :**
```bash
docker image prune
```

**GRAND MÉNAGE (tout supprimer d'un coup) :**
```bash
docker system prune -a
```

**⚠️ DANGER :** Cette commande supprime TOUT ce qui n'est pas utilisé !

---

## 📋 Partie 6 : AIDE-MÉMOIRE (à imprimer et garder !)

### Les commandes essentielles

| Je veux... | Je tape... | Exemple |
|------------|------------|---------|
| **Télécharger une image** | `docker pull` | `docker pull nginx` |
| **Voir mes images** | `docker images` | `docker images` |
| **Lancer un conteneur** | `docker run -d --name` | `docker run -d --name web nginx` |
| **Voir ce qui tourne** | `docker ps` | `docker ps` |
| **Voir tout (même arrêté)** | `docker ps -a` | `docker ps -a` |
| **Démarrer** | `docker start` | `docker start web` |
| **Arrêter** | `docker stop` | `docker stop web` |
| **Redémarrer** | `docker restart` | `docker restart web` |
| **Voir les logs** | `docker logs` | `docker logs web` |
| **Entrer dedans** | `docker exec -it ... bash` | `docker exec -it web bash` |
| **Supprimer** | `docker rm` | `docker rm web` |
| **Faire le ménage** | `docker system prune` | `docker system prune -a` |

**💡 Conseil : Imprimez cette page et collez-la près de votre écran !**

---

## 💪 Partie 7 : EXERCICES PRATIQUES

### ✏️ Exercice 1 : Lancer votre premier serveur web

**Mission : Créer un serveur web Nginx**

**Étapes :**

**1. Téléchargez Nginx**
```bash
docker pull nginx
```

**2. Vérifiez qu'il est bien téléchargé**
```bash
docker images
```
✅ Vous devez voir `nginx` dans la liste

**3. Lancez-le avec un nom clair**
```bash
docker run -d --name mon-site nginx
```

**4. Vérifiez qu'il tourne**
```bash
docker ps
```
✅ Vous devez voir `mon-site` avec Status = "Up"

**5. Regardez ses logs**
```bash
docker logs mon-site
```

**6. Arrêtez-le**
```bash
docker stop mon-site
```

**7. Vérifiez qu'il est arrêté**
```bash
docker ps -a
```
✅ Status doit être "Exited"

**8. Supprimez-le**
```bash
docker rm mon-site
```

**✅ SI VOUS AVEZ RÉUSSI → BRAVO ! Vous maîtrisez les bases !** 🎉

---

### ✏️ Exercice 2 : Créer un fichier dans Ubuntu

**Mission : Créer un fichier texte dans un conteneur Ubuntu**

**Étapes :**

**1. Lancez Ubuntu en mode interactif**
```bash
docker run -it --name test-ubuntu ubuntu bash
```

**2. Vous êtes maintenant DANS Ubuntu. Créez un fichier**
```bash
echo "Bonjour Docker !" > /tmp/message.txt
```

**3. Vérifiez que le fichier existe**
```bash
cat /tmp/message.txt
```
✅ Vous devez voir : "Bonjour Docker !"

**4. Sortez du conteneur**
```bash
exit
```

**5. Le conteneur est arrêté. Redémarrez-le**
```bash
docker start test-ubuntu
```

**6. Entrez à nouveau dedans**
```bash
docker exec -it test-ubuntu bash
```

**7. Vérifiez que le fichier existe toujours**
```bash
cat /tmp/message.txt
```
✅ Le fichier est toujours là ! Les données persistent !

**8. Sortez et faites le ménage**
```bash
exit
docker stop test-ubuntu
docker rm test-ubuntu
```

**✅ MISSION ACCOMPLIE !** 🎉

---

### ✏️ Exercice 3 : Gérer 3 sites web en même temps

**Mission : Créer 3 serveurs web différents**

**1. Créez 3 conteneurs avec des noms différents**
```bash
docker run -d --name site1 nginx
docker run -d --name site2 nginx
docker run -d --name site3 nginx
```

**2. Vérifiez qu'ils tournent tous**
```bash
docker ps
```
✅ Vous devez voir site1, site2, site3 avec "Up"

**3. Arrêtez seulement site2**
```bash
docker stop site2
```

**4. Vérifiez**
```bash
docker ps -a
```
✅ site1 et site3 = "Up" | site2 = "Exited"

**5. Redémarrez site2**
```bash
docker start site2
```

**6. Arrêtez tout d'un coup**
```bash
docker stop site1 site2 site3
```

**7. Supprimez tout d'un coup**
```bash
docker rm site1 site2 site3
```

**✅ VOUS ÊTES UN PRO !** 🚀

---

### ✏️ Exercice 4 : PROJET - Faire communiquer PHP et MariaDB

**Mission : Créer une application web PHP qui se connecte à une base de données MariaDB**

**C'est le projet le plus réaliste ! Comme un vrai site web !** 🌐

---

## 🧠 AVANT TOUT : Comprendre ce qu'on va faire

### Le plan de l'exercice (en 3 étapes simples)

**Imaginez que vous construisez une maison avec 2 pièces :**

```
┌──────────────────────────────────────────┐
│         VOTRE MAISON (le réseau)         │
│                                          │
│  ┌─────────────┐      ┌──────────────┐  │
│  │   Pièce 1   │◄────►│   Pièce 2    │  │
│  │             │      │              │  │
│  │   PHP       │      │   MariaDB    │  │
│  │ (site web)  │      │ (stockage)   │  │
│  └─────────────┘      └──────────────┘  │
│         ▲                                │
│         │                                │
│    Porte d'entrée                        │
│    (port 8080)                           │
└──────────────────────────────────────────┘
         ▲
         │
    Vous dans votre
    navigateur web
```

**Les 3 étapes :**

**1️⃣ CONSTRUIRE LA MAISON** (créer un réseau Docker)
- Sans maison, les 2 pièces ne peuvent pas exister ensemble
- Le réseau = le terrain sur lequel on construit

**2️⃣ CONSTRUIRE LES 2 PIÈCES** (lancer 2 conteneurs)
- Pièce 1 = PHP (le site web)
- Pièce 2 = MariaDB (la base de données)
- Les 2 pièces sont dans la même maison, donc elles peuvent se parler !

**3️⃣ INSTALLER UNE PORTE D'ENTRÉE** (ouvrir le port 8080)
- Pour que VOUS puissiez entrer dans la maison depuis votre navigateur
- Sans porte, vous ne pouvez pas accéder au site !

**Simple non ? Maintenant on va le faire pour de vrai !** 💪

---

**📁 PRÉPARATION : Créer les fichiers**

**1. Créez un dossier pour le projet**

**💡 POURQUOI un dossier ?**
- Votre fichier PHP doit être quelque part sur votre PC
- Docker va "regarder" dans ce dossier pour trouver votre fichier
- C'est comme créer un dossier de projet pour votre code

**TAPEZ CES COMMANDES :**
```bash
mkdir mon-projet-docker
cd mon-projet-docker
```

**Vérifiez que vous êtes dans le bon dossier :**
```bash
pwd
```
→ Vous devriez voir quelque chose comme `/home/votre-nom/mon-projet-docker`

✅ **Vous êtes dans le dossier ? Parfait !**

---

**2. Créez un fichier PHP**

**💡 CE QU'ON VA FAIRE :**
- On va créer un fichier `index.php` DANS ce dossier
- Ce fichier contient le code de notre site web
- Plus tard, Docker va "lire" ce fichier pour afficher le site

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

---

**🧠 MAIS AVANT... COMPRENDRE LES RÉSEAUX DOCKER !**

### Pourquoi on ne peut pas juste lancer les 2 conteneurs ?

**Imaginez cette situation :**

Vous avez 2 personnes dans 2 pièces différentes qui doivent se parler :
- **Pièce 1** = Conteneur PHP (votre site web)
- **Pièce 2** = Conteneur MariaDB (votre base de données)

**❌ SANS RÉSEAU = Impossible de communiquer**
```
┌─────────────────┐         ┌──────────────────┐
│  Conteneur PHP  │    X    │ Conteneur MariaDB│
│                 │  Mur    │                  │
│  "Où es-tu ?"   │ ═══════ │  "Je suis là !"  │
└─────────────────┘         └──────────────────┘
```

**✅ AVEC RÉSEAU = Ils peuvent se parler par leur nom !**
```
┌─────────────────┐         ┌──────────────────┐
│  Conteneur PHP  │ ←─────→ │ Conteneur MariaDB│
│                 │ Réseau  │                  │
│  "Hé MariaDB !" │ Docker  │  "Oui PHP ?"     │
└─────────────────┘         └──────────────────┘
```

---

### Comprendre avec une analogie simple

**SANS RÉSEAU DOCKER :**

C'est comme si vous vouliez appeler votre ami, mais :
- Vous ne connaissez pas son numéro de téléphone
- Son numéro change tout le temps
- Impossible de le joindre !

**AVEC RÉSEAU DOCKER :**

Maintenant, vous avez un annuaire :
- Votre ami s'appelle "MariaDB"
- Vous l'appelez par son nom
- Docker trouve automatiquement son "numéro" (adresse IP)

**Dans le code PHP, regardez ligne 1040 du fichier :**
```php
$host = 'ma-base-de-donnees';  // ← On utilise le NOM, pas une adresse IP !
```

**Grâce au réseau Docker, PHP sait où trouver MariaDB juste avec son nom !** 🎯

---

### Les 3 choses à retenir

**1. Un réseau Docker = Un annuaire de noms**
```
Réseau "mon-reseau" :
├── ma-base-de-donnees  → 172.18.0.2
├── mon-site-php        → 172.18.0.3
```

**2. Sans réseau, les conteneurs sont isolés**
```
Conteneur 1 : "Je ne connais personne"
Conteneur 2 : "Moi non plus"
```

**3. Avec un réseau, ils se trouvent par leur nom**
```
PHP : "Hé ma-base-de-donnees, tu es où ?"
Docker : "Elle est à 172.18.0.2, je te connecte !"
MariaDB : "Salut PHP !"
```

---

**🚀 LANCEMENT DU PROJET**

**Étape 1 : Créer un réseau pour que les conteneurs se parlent**
```bash
docker network create mon-reseau
```

**💡 Ce que ça fait :**
- Crée un réseau privé appelé "mon-reseau"
- Comme créer un groupe WhatsApp pour vos conteneurs !
- Tous les conteneurs dans ce réseau pourront se parler par leur nom

**Étape 2 : Lancer MariaDB**
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

**📝 Explications :**
- `--name ma-base-de-donnees` → C'est comme ça que PHP va l'appeler !
- `--network mon-reseau` → Le met dans le réseau
- `-e` → Variables d'environnement (configuration)

**⏳ Attendez 10 secondes** que MariaDB démarre complètement

**Étape 3 : Vérifier que MariaDB tourne**
```bash
docker ps
```
✅ Vous devez voir `ma-base-de-donnees` avec "Up"

**Étape 4 : Lancer PHP (LISEZ BIEN LES EXPLICATIONS !)**

**🧠 AVANT DE TAPER LA COMMANDE, COMPRENONS :**

Cette commande est la plus importante de tout l'exercice ! On va la décomposer ligne par ligne.

```bash
docker run -d \
  --name mon-site-php \
  --network mon-reseau \
  -p 8080:80 \
  -v $(pwd):/var/www/html \
  php:8.2-apache
```

---

### 🎓 DÉCORTIQUONS CETTE COMMANDE (super important !)

**Ligne 1 : `docker run -d`**
- Lance un conteneur
- `-d` = mode détaché (tourne en fond)
✅ Simple !

**Ligne 2 : `--name mon-site-php`**
- Donne le nom "mon-site-php" au conteneur
✅ Simple aussi !

**Ligne 3 : `--network mon-reseau`**
- Met le conteneur dans le réseau "mon-reseau"
- Grâce à ça, il pourra parler à MariaDB !
✅ OK !

**Ligne 4 : `-p 8080:80` (la PORTE d'ENTRÉE)**

**💡 EXPLICATION SIMPLE :**

Imaginez que le conteneur PHP est une maison avec une porte :
- La porte de la maison est le port **80** (c'est le port par défaut pour les sites web)
- MAIS vous ne pouvez pas rentrer directement par cette porte depuis votre PC !
- Il faut créer un "tunnel" entre votre PC et la maison

**Le tunnel fonctionne comme ça :**
```
VOTRE PC              DOCKER              CONTENEUR PHP
(port 8080) ───────────────────────► (port 80)

Vous tapez            Docker fait         Apache (serveur web)
localhost:8080        le lien            écoute sur le port 80
dans le navigateur
```

**En français simple :**
- Vous tapez `http://localhost:8080` dans votre navigateur
- Docker dit "Ah ! Port 8080 ? Je redirige vers le conteneur PHP port 80 !"
- Le site s'affiche ! ✨

**Pourquoi 8080 et pas 80 directement ?**
- Le port 80 sur votre PC est souvent déjà utilisé
- 8080 est un port "libre" qu'on peut utiliser
- C'est juste une convention pratique

---

**Ligne 5 : `-v $(pwd):/var/www/html` (LE POINT LE PLUS IMPORTANT !)**

### 🔑 CETTE LIGNE EST LA CLÉ DE TOUT ! LISEZ BIEN !

**💡 LE PROBLÈME :**

Vous avez créé `index.php` sur VOTRE PC, dans le dossier `mon-projet-docker`.

**MAIS** le conteneur PHP est une machine séparée, isolée !

```
┌─────────────────┐         ┌──────────────────┐
│   VOTRE PC      │         │  CONTENEUR PHP   │
│                 │         │                  │
│  mon-projet-    │    ?    │  /var/www/html   │
│  docker/        │         │                  │
│  └─ index.php   │         │  (vide !)        │
└─────────────────┘         └──────────────────┘
```

**Le conteneur PHP cherche les fichiers dans `/var/www/html`**

**MAIS votre fichier est sur VOTRE PC !**

**Comment faire pour que le conteneur voie votre fichier ?** 🤔

---

### 💡 LA SOLUTION : LE MONTAGE DE VOLUME (-v)

**Le montage de volume = créer un "portail magique" entre votre PC et le conteneur**

```
┌─────────────────┐         ┌──────────────────┐
│   VOTRE PC      │         │  CONTENEUR PHP   │
│                 │         │                  │
│  mon-projet-    │ ═══════►│  /var/www/html   │
│  docker/        │ PORTAIL │                  │
│  └─ index.php   │ MAGIQUE │  └─ index.php    │
└─────────────────┘         └──────────────────┘
```

**Grâce à `-v`, c'est comme si les 2 dossiers étaient FUSIONNÉS !**

---

### 🎯 DÉCORTIQUONS `-v $(pwd):/var/www/html`

**Syntaxe générale :**
```
-v  DOSSIER_SUR_VOTRE_PC : DOSSIER_DANS_LE_CONTENEUR
```

**Dans notre cas :**
```
-v  $(pwd) : /var/www/html
    ▲▲▲▲▲    ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲
    │         │
    │         └─ Dossier DANS le conteneur où Apache cherche les fichiers
    │
    └─ $(pwd) = "Print Working Directory" = le dossier actuel (mon-projet-docker)
```

**Traduction en français :**
"Docker, prends le dossier où je suis actuellement (`mon-projet-docker`) et fais-le apparaître dans le conteneur à l'emplacement `/var/www/html`"

---

### ⚠️ ERREUR FRÉQUENTE : Être dans le mauvais dossier !

**CE QUI SE PASSE SI VOUS N'ÊTES PAS DANS `mon-projet-docker` :**

**❌ MAUVAIS - Vous êtes dans `/home/votre-nom` :**
```
/home/votre-nom$ docker run -d -v $(pwd):/var/www/html php:8.2-apache
                  ▲
                  │
              $(pwd) = /home/votre-nom

Résultat : Docker monte /home/votre-nom dans le conteneur
           → index.php n'est PAS là !
           → Le site ne marche pas ! ❌
```

**✅ BON - Vous êtes dans `/home/votre-nom/mon-projet-docker` :**
```
/home/votre-nom/mon-projet-docker$ docker run -d -v $(pwd):/var/www/html php:8.2-apache
                                   ▲
                                   │
                               $(pwd) = /home/votre-nom/mon-projet-docker

Résultat : Docker monte /home/votre-nom/mon-projet-docker dans le conteneur
           → index.php est là !
           → Le site marche ! ✅
```

**RÈGLE D'OR :**
**Avant TOUTE commande avec `-v $(pwd)`, faites `pwd` pour vérifier où vous êtes !**

---

### 🎬 CE QUI SE PASSE CONCRÈTEMENT :

**1. Vous créez `index.php` dans `mon-projet-docker` sur votre PC**

**2. Vous lancez la commande avec `-v $(pwd):/var/www/html`**

**3. Docker crée le "portail magique" :**
   ```
   mon-projet-docker/index.php  ═══► /var/www/html/index.php (dans le conteneur)
   ```

**4. Apache (le serveur web dans le conteneur) lit `/var/www/html/index.php`**

**5. MAIS en réalité il lit votre fichier sur votre PC !**

---

### ✨ LE SUPER POUVOIR DU MONTAGE DE VOLUME

**Le gros avantage ? VOUS POUVEZ MODIFIER LE FICHIER EN DIRECT !**

```
Vous modifiez index.php      Apache voit le changement
sur votre PC avec un         IMMÉDIATEMENT !
éditeur de code

      ▼                            ▼
[Enregistrer]  ═══════════► [Actualiser le navigateur]
                            └─ Le site est déjà mis à jour !
```

**Pas besoin de :**
- ❌ Copier le fichier dans le conteneur
- ❌ Redémarrer le conteneur
- ❌ Reconstruire quoi que ce soit

**C'EST MAGIQUE !** ✨

---

### 🎨 TESTEZ VOUS-MÊME : Modification en temps réel

**Une fois que votre site tourne sur `http://localhost:8080`, faites ce test :**

**1. Ouvrez `index.php` avec un éditeur de texte (nano, gedit, VSCode, etc.)**
```bash
nano index.php
```

**2. Changez le titre de la page (ligne 6)**

**Avant :**
```php
<title>Mon App Docker</title>
```

**Après :**
```php
<title>Mon App Docker - MODIFIÉ EN DIRECT !</title>
```

**3. Enregistrez le fichier**
- Avec nano : `Ctrl + O` puis `Entrée`, puis `Ctrl + X`
- Avec gedit ou VSCode : `Ctrl + S`

**4. Allez dans votre navigateur et actualisez la page**
- Appuyez sur `F5` ou `Ctrl + R`

**5. 🎉 REGARDEZ L'ONGLET DU NAVIGATEUR !**
- Le titre a changé INSTANTANÉMENT !
- Vous n'avez PAS redémarré le conteneur !
- Vous n'avez RIEN copié !

---

### 🧠 COMPRENDRE POURQUOI C'EST INSTANTANÉ

**Voici ce qui se passe en réalité :**

```
┌─────────────────────────────────────────────────────┐
│                   VOTRE PC                          │
│                                                     │
│  📝 Vous modifiez index.php avec votre éditeur     │
│     └─ Le fichier change sur votre disque dur      │
│                                                     │
│        ▼ SYNCHRONISATION AUTOMATIQUE ▼             │
│                                                     │
│  ┌──────────────────────────────────────┐          │
│  │  CONTENEUR DOCKER                    │          │
│  │                                      │          │
│  │  Apache lit /var/www/html/index.php │          │
│  │           ▲                          │          │
│  │           │ Pointe vers              │          │
│  │           │ votre fichier PC         │          │
│  │           │ grâce au volume !        │          │
│  └──────────────────────────────────────┘          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Explication simple :**

1. **Le fichier N'EST PAS copié dans le conteneur !**
2. **Le conteneur LIT DIRECTEMENT le fichier sur votre PC !**
3. **Quand vous modifiez le fichier → Apache lit la nouvelle version immédiatement !**

**C'est comme si Apache avait un "raccourci" vers votre fichier PC !**

---

### 💡 TESTEZ AVEC DU CSS AUSSI !

**Changez le style de la page :**

**Trouvez cette partie dans index.php (lignes 7-14) :**
```php
<style>
    body {
        font-family: Arial;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 50px;
        text-align: center;
    }
```

**Changez la couleur de fond :**
```php
<style>
    body {
        font-family: Arial;
        background: linear-gradient(135deg, #FF6B6B 0%, #4ECDC4 100%);  ← NOUVELLE COULEUR !
        color: white;
        padding: 50px;
        text-align: center;
    }
```

**Enregistrez → Actualisez le navigateur → Le fond a changé de couleur ! 🎨**

---

### ✅ CE QUE VOUS DEVEZ RETENIR

**Grâce à `-v $(pwd):/var/www/html` :**

✅ Vous modifiez le code sur votre PC
✅ Les changements sont visibles IMMÉDIATEMENT dans le conteneur
✅ Pas besoin de redémarrer quoi que ce soit
✅ C'est exactement comme si vous développiez sur votre PC normal !

**C'est pour ça qu'on utilise Docker pour développer !** 🚀

**Workflow de développement classique :**
```
1. Écrire du code dans votre éditeur
2. Enregistrer (Ctrl + S)
3. Actualiser le navigateur (F5)
4. Voir le résultat
5. Répéter → C'est ultra-rapide !
```

---

### 📝 RÉCAP VISUEL

```
VOTRE COMMANDE :
docker run -d --name mon-site-php --network mon-reseau -p 8080:80 -v $(pwd):/var/www/html php:8.2-apache

CE QUI SE PASSE :
1. Crée un conteneur nommé "mon-site-php"
2. Le met dans le réseau "mon-reseau" (pour parler à MariaDB)
3. Ouvre la porte : localhost:8080 → conteneur:80
4. Crée le portail : mon-projet-docker → /var/www/html
5. Installe PHP 8.2 avec Apache
```

---

**MAINTENANT TAPEZ LA COMMANDE !**

**⚠️ ATTENTION - SUPER IMPORTANT ⚠️**

**AVANT de taper la commande, vérifiez que vous êtes DANS le dossier `mon-projet-docker` !**

**Comment vérifier ? Tapez :**
```bash
pwd
```

**Vous DEVEZ voir :**
```
/home/votre-nom/mon-projet-docker
```

**Pourquoi c'est important ?**
- `$(pwd)` = le dossier actuel
- Si vous êtes dans `/home/votre-nom`, Docker va monter le MAUVAIS dossier !
- Il faut être dans `mon-projet-docker` où se trouve `index.php` !

**Si vous n'êtes PAS dans le bon dossier, allez-y :**
```bash
cd mon-projet-docker
```

✅ **Vous êtes dans le bon dossier ? Maintenant tapez la commande :**

```bash
docker run -d \
  --name mon-site-php \
  --network mon-reseau \
  -p 8080:80 \
  -v $(pwd):/var/www/html \
  php:8.2-apache
```

**⏳ Attendez 5 secondes** que le conteneur démarre

**Vérifiez qu'il tourne :**
```bash
docker ps
```
✅ Vous devez voir `mon-site-php` avec "Up"

---

**Étape 5 : Installer l'extension MySQL dans PHP (TRÈS IMPORTANT !)**

### 🤔 POURQUOI CETTE ÉTAPE ? (la réponse que PERSONNE ne vous donne !)

**🧠 CE QU'IL FAUT COMPRENDRE :**

Quand vous installez PHP sur votre PC normal, vous installez souvent "PHP + toutes les extensions".

**MAIS** avec Docker, c'est différent !

**L'image `php:8.2-apache` contient :**
- ✅ PHP de base (le langage)
- ✅ Apache (le serveur web)
- ❌ MAIS PAS les extensions pour MySQL !

**C'est comme acheter une voiture :**
- ✅ Vous avez le moteur
- ✅ Vous avez les roues
- ❌ MAIS pas la radio !

**Il faut installer la radio (l'extension MySQL) APRÈS !**

---

### 💡 POURQUOI PHP n'inclut PAS MySQL par défaut ?

**3 raisons :**

**1. Taille** → MySQL/MariaDB est gros ! Si tout le monde ne l'utilise pas, pourquoi le mettre ?

**2. Flexibilité** → Certains utilisent MySQL, d'autres PostgreSQL, d'autres SQLite...
   Docker vous laisse choisir ce dont VOUS avez besoin !

**3. Sécurité** → Moins de code = moins de failles de sécurité

**C'est pour ça qu'on doit l'installer manuellement !**

---

### 🔧 INSTALLER LES EXTENSIONS MYSQL

**TAPEZ CETTE COMMANDE :**
```bash
docker exec mon-site-php docker-php-ext-install pdo pdo_mysql
```

### 🎓 DÉCOMPOSONS :

**`docker exec`**
→ "Docker, je veux exécuter une commande DANS un conteneur qui tourne"

**`mon-site-php`**
→ "Le conteneur s'appelle mon-site-php"

**`docker-php-ext-install`**
→ "Utilise l'outil d'installation d'extensions PHP"
   (Cet outil est inclus dans l'image `php:8.2-apache` !)

**`pdo pdo_mysql`**
→ "Installe ces 2 extensions :"
   - **pdo** = PHP Data Objects (pour se connecter à des bases de données)
   - **pdo_mysql** = La couche spécifique pour MySQL/MariaDB

---

### 🎬 CE QUI SE PASSE QUAND VOUS TAPEZ LA COMMANDE :

```
1. Docker entre dans le conteneur "mon-site-php"
2. Il compile l'extension PDO
3. Il compile l'extension PDO_MySQL
4. Il les active dans la configuration PHP
```

**Vous verrez défiler plein de lignes :**
```
Configuring for:
PHP Api Version:         20220805
Zend Module Api No:      20220805
...
Installing '/usr/local/lib/php/extensions/...'
...
```

**C'EST NORMAL ! Ça compile les extensions.**

**⏳ Ça prend environ 30 secondes à 1 minute.**

**✅ Quand ça dit "complete" ou que ça s'arrête → c'est bon !**

---

### 🔄 POURQUOI IL FAUT REDÉMARRER ?

**Imaginez que PHP est une voiture qui roule.**

Vous venez d'installer la radio **pendant que la voiture roule** !

**Pour que la radio fonctionne, il faut :**
1. Arrêter la voiture
2. Redémarrer la voiture
3. Maintenant la radio marche !

**C'est pareil pour PHP !**

---

**Étape 6 : Redémarrer PHP pour appliquer les changements**

**TAPEZ ÇA :**
```bash
docker restart mon-site-php
```

**Ce qui se passe :**
1. Docker arrête le conteneur (2 secondes)
2. Docker redémarre le conteneur (2 secondes)
3. PHP charge maintenant les nouvelles extensions !

**✅ TERMINÉ ! Maintenant PHP peut parler à MariaDB !**

---

**🌐 TESTER L'APPLICATION**

**Ouvrez votre navigateur et allez sur :**
```
http://localhost:8080
```

**✅ Vous devriez voir :**
- "Connexion réussie à MariaDB !"
- Une liste de visiteurs

**🔄 Actualisez la page plusieurs fois** → De nouveaux visiteurs apparaissent !

**C'EST MAGIQUE ! Les deux conteneurs communiquent !** 🎉

---

**🔍 EXPLORER LA BASE DE DONNÉES**

**Voir ce qui est dans la base :**
```bash
docker exec -it ma-base-de-donnees mysql -u stephane -pmotdepasse123 ma_base
```

**Une fois dedans, tapez :**
```sql
SELECT * FROM visiteurs;
```

**Sortir :**
```sql
exit
```

---

**🛑 ARRÊTER LE PROJET**

```bash
docker stop mon-site-php ma-base-de-donnees
```

---

**🗑️ NETTOYER TOUT**

```bash
# Supprimer les conteneurs
docker rm mon-site-php ma-base-de-donnees

# Supprimer le réseau
docker network rm mon-reseau

# (Optionnel) Supprimer les images
docker rmi php:8.2-apache mariadb:latest
```

---

**🎓 CE QUE VOUS AVEZ APPRIS**

✅ Créer un réseau Docker pour connecter des conteneurs
✅ Faire communiquer 2 conteneurs (PHP et MariaDB)
✅ Monter un fichier local dans un conteneur avec `-v`
✅ Exposer un port avec `-p`
✅ Configurer une base de données avec des variables d'environnement

**C'EST EXACTEMENT COMME ÇA QUE FONCTIONNENT LES VRAIS SITES WEB !** 🚀

---

## 📚 RÉCAPITULATIF COMPLET : Qu'est-ce qu'on a fait exactement ?

### 🎬 Le film complet de l'exercice

**Imaginez que vous racontez à un ami ce que vous avez fait. Voici l'histoire :**

---

**CHAPITRE 1 : La préparation (sur votre PC)**

1. Vous avez créé un dossier `mon-projet-docker`
2. Vous y avez créé un fichier `index.php` qui contient le code du site web

**À ce stade :**
- ✅ Le fichier existe sur VOTRE PC
- ❌ Docker ne sait pas encore qu'il existe

---

**CHAPITRE 2 : Construire le terrain (le réseau)**

3. Vous avez créé un réseau Docker appelé `mon-reseau`

**Pourquoi ?**
- Pour que les 2 conteneurs (PHP et MariaDB) puissent se parler
- Sans réseau = 2 personnes dans 2 pièces séparées sans téléphone
- Avec réseau = 2 personnes avec un téléphone direct entre elles

---

**CHAPITRE 3 : Installer la base de données**

4. Vous avez lancé MariaDB dans un conteneur
5. Vous l'avez mis dans le réseau `mon-reseau`
6. Vous lui avez donné un nom : `ma-base-de-donnees`

**Le résultat :**
```
Réseau "mon-reseau"
└── ma-base-de-donnees (MariaDB qui tourne)
```

---

**CHAPITRE 4 : Installer le serveur web (LA PARTIE COMPLEXE !)**

7. Vous avez lancé PHP/Apache dans un conteneur

**MAIS** cette commande fait 4 choses magiques :

**a) `-p 8080:80` → Ouvrir une porte d'entrée**
```
Votre navigateur → localhost:8080 → Conteneur PHP:80
```

**b) `-v $(pwd):/var/www/html` → Créer le portail magique**
```
Votre dossier mon-projet-docker ═══► /var/www/html dans le conteneur
```
→ Grâce à ça, Apache peut lire votre fichier index.php !

**c) `--network mon-reseau` → Rejoindre le réseau**
```
Réseau "mon-reseau"
├── ma-base-de-donnees (MariaDB)
└── mon-site-php (PHP/Apache)  ← Peut maintenant parler à MariaDB !
```

**d) `php:8.2-apache` → Installer PHP + Apache**

---

**CHAPITRE 5 : Installer la "radio" (les extensions MySQL)**

8. Vous avez installé `pdo` et `pdo_mysql`

**Pourquoi ?**
- L'image PHP de base n'a PAS les extensions MySQL
- C'est comme une voiture sans radio
- Il faut l'installer après !

9. Vous avez redémarré le conteneur pour activer les extensions

---

**CHAPITRE 6 : Tester !**

10. Vous ouvrez `http://localhost:8080` dans le navigateur

**Le voyage de la requête :**
```
1. Votre navigateur → http://localhost:8080
                      ▼
2. Docker redirige → Conteneur PHP (port 80)
                      ▼
3. Apache cherche → /var/www/html/index.php
                      ▼
4. Grâce au montage de volume → Il trouve votre fichier sur votre PC !
                      ▼
5. PHP exécute le code
                      ▼
6. Le code se connecte à "ma-base-de-donnees"
                      ▼
7. Grâce au réseau → Docker trouve MariaDB !
                      ▼
8. PHP récupère les données de MariaDB
                      ▼
9. PHP génère le HTML
                      ▼
10. Apache renvoie le HTML → Docker → Votre navigateur
                      ▼
11. Vous voyez la page ! 🎉
```

---

### 🔑 LES 3 CONCEPTS CLÉS À RETENIR

**1. LE RÉSEAU (`--network`)**
```
Permet aux conteneurs de se parler par leur nom
PHP peut dire "Hé ma-base-de-donnees, donne-moi les données !"
```

**2. LE MONTAGE DE VOLUME (`-v`)**
```
Fait apparaître vos fichiers PC dans le conteneur
Vous modifiez index.php → Le changement est INSTANTANÉ dans le conteneur !
```

**3. LE MAPPING DE PORT (`-p`)**
```
Ouvre une porte pour accéder au conteneur depuis votre PC
localhost:8080 → conteneur:80
```

---

### 🎯 SCHÉMA FINAL : Tout comprendre d'un coup d'œil

```
┌────────────────────────────────────────────────────────────┐
│                      VOTRE PC                              │
│                                                            │
│  📁 mon-projet-docker/                                     │
│     └── index.php ─────────────┐                          │
│                                │ Montage de volume        │
│  🌐 Navigateur Web             │ (-v)                      │
│     http://localhost:8080 ──┐  │                          │
│                             │  │                          │
└─────────────────────────────│──│───────────────────────────┘
                              │  │
                              │  │ Mapping de port (-p)
                              │  │
┌─────────────────────────────│──│───────────────────────────┐
│                    DOCKER   │  │                           │
│                             ▼  ▼                           │
│  ┌────────────────────────────────────────────────┐        │
│  │       Réseau "mon-reseau"                      │        │
│  │                                                 │        │
│  │   ┌──────────────────┐    ┌─────────────────┐ │        │
│  │   │ mon-site-php     │◄──►│ ma-base-de-     │ │        │
│  │   │                  │    │ donnees         │ │        │
│  │   │ PHP + Apache     │    │                 │ │        │
│  │   │                  │    │ MariaDB         │ │        │
│  │   │ Port 80          │    │ Port 3306       │ │        │
│  │   │                  │    │                 │ │        │
│  │   │ /var/www/html/   │    │ Base de données │ │        │
│  │   │ └─ index.php ────┼────┼─► visiteurs     │ │        │
│  │   │    (pointeur)    │    │                 │ │        │
│  │   └──────────────────┘    └─────────────────┘ │        │
│  │           ▲                                    │        │
│  │           │                                    │        │
│  └───────────│────────────────────────────────────┘        │
│              │                                             │
│              └─── Pointe vers votre fichier PC             │
│                   grâce au volume !                        │
└────────────────────────────────────────────────────────────┘
```

---

### ❓ QUESTIONS FRÉQUENTES (avec réponses claires !)

**Q1 : Pourquoi mon fichier index.php apparaît dans le conteneur ?**
→ Grâce au montage de volume `-v $(pwd):/var/www/html`
→ Docker crée un "lien magique" entre votre dossier et le conteneur

**Q2 : Pourquoi PHP peut se connecter à MariaDB avec juste le nom ?**
→ Grâce au réseau Docker !
→ Dans le réseau, chaque conteneur a un "nom de domaine" = son nom de conteneur
→ PHP dit "ma-base-de-donnees" → Docker trouve automatiquement l'adresse IP !

**Q3 : Pourquoi il faut installer pdo/pdo_mysql ?**
→ L'image PHP de base est "légère" → pas d'extensions par défaut
→ Ça permet de garder l'image petite et de choisir ce dont vous avez besoin
→ C'est comme un téléphone sans applications → vous installez ce que vous voulez !

**Q4 : Si je modifie index.php, est-ce que je dois redémarrer le conteneur ?**
→ **NON !** C'est ça la magie du volume !
→ Modification → Enregistrement → Actualisation du navigateur → Changement visible !

**Q5 : Pourquoi localhost:8080 et pas localhost:80 ?**
→ Le port 80 est souvent déjà utilisé sur votre PC
→ 8080 est un port "libre" et une convention pour le développement
→ Vous pouvez utiliser n'importe quel port (8000, 3000, etc.)

---

### 🎓 TESTEZ VOTRE COMPRÉHENSION

**Sans regarder les réponses, pouvez-vous expliquer à voix haute :**

1. À quoi sert le réseau Docker ?
2. À quoi sert `-v $(pwd):/var/www/html` ?
3. À quoi sert `-p 8080:80` ?
4. Pourquoi on installe pdo et pdo_mysql ?
5. Que se passe-t-il quand vous tapez localhost:8080 dans le navigateur ?

**Si vous pouvez répondre à ces 5 questions → VOUS AVEZ TOUT COMPRIS !** 🎉

---

**✅ MISSION ULTRA-ACCOMPLIE ! VOUS ÊTES UN VRAI PRO DOCKER !** 🏆

**Maintenant vous comprenez VRAIMENT ce que vous faites, pas juste copier-coller des commandes !**

---

## ❌ Partie 8 : Les erreurs fréquentes

### Erreur 1 : "command not found"

**Message :**
```
bash: docker: command not found
```

**Problème :** Docker n'est pas installé

**Solution :** Appelez votre formateur pour l'installer

---

### Erreur 2 : "Cannot connect to the Docker daemon"

**Message :**
```
Cannot connect to the Docker daemon. Is the docker daemon running?
```

**Problème :** Le service Docker ne tourne pas

**Solution :**
```bash
sudo systemctl start docker
```

---

### Erreur 3 : "permission denied"

**Message :**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Problème :** Vous n'avez pas les droits

**Solution :**
```bash
sudo usermod -aG docker $USER
```
Puis déconnectez-vous et reconnectez-vous

---

### Erreur 4 : "No such container"

**Message :**
```
Error: No such container: mon-site
```

**Problème :** Le conteneur n'existe pas

**Solution :**
```bash
docker ps -a  # Vérifiez le vrai nom
```

---

### Erreur 5 : "The container name is already in use"

**Message :**
```
Error: The container name "/mon-site" is already in use
```

**Problème :** Un conteneur avec ce nom existe déjà

**Solution :** Supprimez l'ancien ou utilisez un autre nom
```bash
docker rm mon-site
```

---

### Erreur 6 : "error during connect"

**Message complet :**
```
error during connect: This error may indicate that the docker daemon is not running.
```

**Problème :** Le service Docker ne tourne pas

**Solution :**
```bash
sudo systemctl start docker
```

Si le problème persiste, appelez votre formateur.

---

## ✅ Quiz final

**Question 1 : Quelle commande pour voir les conteneurs qui tournent ?**
- A) `docker list`
- B) `docker ps`
- C) `docker show`

<details>
<summary>Voir la réponse</summary>
✅ **B) docker ps**
</details>

---

**Question 2 : Que fait `docker pull` ?**
- A) Supprime une image
- B) Télécharge une image
- C) Lance un conteneur

<details>
<summary>Voir la réponse</summary>
✅ **B) Télécharge une image**
</details>

---

**Question 3 : Comment lancer un conteneur en arrière-plan ?**
- A) `docker run -b`
- B) `docker run -d`
- C) `docker run -background`

<details>
<summary>Voir la réponse</summary>
✅ **B) docker run -d**
</details>

---

**Question 4 : Comment arrêter un conteneur nommé "web" ?**
- A) `docker end web`
- B) `docker stop web`
- C) `docker kill web`

<details>
<summary>Voir la réponse</summary>
✅ **B) docker stop web**
</details>

---

**Question 5 : Que fait `docker rm` ?**
- A) Supprime une image
- B) Supprime un conteneur
- C) Redémarre un conteneur

<details>
<summary>Voir la réponse</summary>
✅ **B) Supprime un conteneur**
</details>

---

## 🎯 Récapitulatif final

### Ce que vous avez appris aujourd'hui :

✅ Vérifier que Docker fonctionne  
✅ Télécharger des images avec `docker pull`  
✅ Lancer des conteneurs avec `docker run`  
✅ Voir ce qui tourne avec `docker ps`  
✅ Démarrer/Arrêter avec `docker start/stop`  
✅ Entrer dans un conteneur avec `docker exec`  
✅ Faire le ménage avec `docker system prune`

### Les 3 erreurs à NE PAS faire :

❌ **Oublier de donner un nom avec `--name`**  
→ Utilisez TOUJOURS `--name quelque-chose`

❌ **Oublier de nettoyer régulièrement**  
→ Faites `docker ps -a` et supprimez ce qui ne sert plus

❌ **Essayer de supprimer un conteneur qui tourne**  
→ Arrêtez-le AVANT avec `docker stop`

---

## 🚀 Et maintenant ?

**BRAVO ! Vous savez utiliser Docker !** 🎉

**Dans le prochain cours (Cours 4), vous apprendrez :**
- Comment accéder à vos conteneurs depuis le navigateur web
- Comment connecter plusieurs conteneurs ensemble
- Comment faire tourner une vraie application web

**Entraînez-vous bien avec les exercices avant de passer au cours 4 !**

---

**Aide-mémoire ultra-court :**
```bash
docker pull IMAGE          # Télécharger
docker run -d --name NOM IMAGE   # Lancer
docker ps                  # Voir
docker stop NOM            # Arrêter
docker rm NOM              # Supprimer
```

**Bon courage et amusez-vous bien avec Docker !** 🐳