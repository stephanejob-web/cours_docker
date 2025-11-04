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

### 🎉 Félicitations ! Vous maîtrisez les commandes de base !

**Ce que vous savez faire maintenant :**

✅ Utiliser le terminal
✅ Télécharger des images (`docker pull`)
✅ Lancer des conteneurs (`docker run`)
✅ Voir ce qui tourne (`docker ps`)
✅ Gérer les conteneurs (start, stop, restart, logs)
✅ Faire le ménage (rm, rmi, prune)
✅ Créer et gérer plusieurs conteneurs

**📘 Dans les prochains cours, vous apprendrez :**

- **Cours 04 : Les Ports** → Accéder à vos conteneurs dans le navigateur
- **Cours 05 : Les Volumes** → Sauvegarder vos données et modifier du code en temps réel
- **Cours 06 : Les Réseaux** → Faire communiquer vos conteneurs entre eux
- **Cours 07 : Les Variables d'Environnement** → Configurer vos conteneurs
- **Cours 08 : PROJET COMPLET** → Application web PHP + MariaDB (on combine tout !)

**💪 Vous êtes prêt pour la suite !**
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