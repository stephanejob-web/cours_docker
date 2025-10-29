# Cours 2 : Les Concepts de Base de Docker 🎓

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Comment fonctionne Docker (simplement)
- C'est quoi une IMAGE Docker
- C'est quoi un CONTENEUR Docker
- La différence entre les deux
- C'est quoi le Docker Hub

**Durée : 20 minutes de lecture**

---

## 📖 Partie 1 : Comment marche Docker ? (Version simple)

### L'architecture Docker expliquée simplement

Imaginez Docker comme une **usine à conteneurs** :

```
┌─────────────────────────────────────────┐
│         VOTRE ORDINATEUR                │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │     VOS CONTENEURS              │   │
│  │  📦 Site web  📦 Base données   │   │
│  │  📦 Redis     📦 Application    │   │
│  └─────────────────────────────────┘   │
│               ⬆️                         │
│  ┌─────────────────────────────────┐   │
│  │      DOCKER ENGINE              │   │
│  │   (Le moteur qui fait marcher)  │   │
│  └─────────────────────────────────┘   │
│               ⬆️                         │
│  ┌─────────────────────────────────┐   │
│  │   VOTRE SYSTÈME (Ubuntu)        │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**En gros :**
1. **En bas** : Votre Ubuntu (le système de base)
2. **Au milieu** : Docker Engine (le "chef d'orchestre")
3. **En haut** : Vos conteneurs (vos programmes)

### Les 3 composants de Docker

**1. Docker Engine** = Le moteur 🚂
- C'est le programme principal de Docker
- Il gère tout : créer, démarrer, arrêter les conteneurs
- Vous ne le voyez pas, mais c'est lui qui travaille

**2. Docker Client** = Vous ! 👤
- C'est vous qui donnez les ordres dans le terminal
- Vous tapez des commandes comme `docker run`
- Le client envoie vos ordres au Docker Engine

**3. Docker Hub** = Le magasin 🏪
- C'est là où il y a des milliers d'images toutes prêtes
- MySQL, PostgreSQL, Redis, Node.js... tout est là !
- Gratuit et accessible à tous

### Comment ça marche en pratique ?

**Exemple :** Vous voulez lancer MySQL

```
1. Vous tapez : docker run mysql
                    ⬇️
2. Docker Client reçoit votre demande
                    ⬇️
3. Docker Engine vérifie : "Est-ce que j'ai déjà MySQL ?"
   - Non → Je vais le télécharger sur Docker Hub
   - Oui → Je l'utilise directement
                    ⬇️
4. Docker Engine crée le conteneur MySQL
                    ⬇️
5. MySQL démarre et fonctionne ! ✅
```

**Tout ça en 30 secondes !** 🚀

---

## 🖼️ Partie 2 : C'est quoi une IMAGE Docker ?

### Définition simple

**Une IMAGE = Une RECETTE de cuisine** 📝

Comme une recette qui explique comment faire un gâteau, une image Docker explique comment créer un conteneur.

### Exemple concret : L'image "Ubuntu"

L'image Ubuntu contient :
- ✅ Le système Ubuntu de base
- ✅ Tous les fichiers système nécessaires
- ✅ Les paramètres de configuration
- ✅ Les instructions pour démarrer

**Mais attention !** Une image, c'est juste la recette. Ce n'est **PAS** le gâteau fini !

### Caractéristiques d'une image

**1. Elle est en LECTURE SEULE** 🔒
- Vous ne pouvez pas la modifier directement
- Elle reste toujours identique
- C'est un modèle fixe

**2. Elle est RÉUTILISABLE** ♻️
```
1 image Ubuntu → Peut créer 10 conteneurs
1 image MySQL  → Peut créer 5 conteneurs
1 image Redis  → Peut créer 3 conteneurs
```

**3. Elle est PARTAGEABLE** 🌍
- Vous créez une image sur votre PC
- Vous la partagez sur Docker Hub
- N'importe qui peut l'utiliser !

### D'où viennent les images ?

**3 sources possibles :**

**1. Docker Hub (le magasin officiel)** 🏪
- Des milliers d'images gratuites
- ubuntu, mysql, postgres, redis, nginx...
- Testées et sûres

**2. Vous les créez vous-même** 👨‍💻
- Avec un fichier appelé "Dockerfile"
- On verra ça dans un prochain cours
- Vous pouvez personnaliser tout ce que vous voulez

**3. D'autres registres** 📦
- Des entreprises ont leurs propres magasins
- Google, Amazon, Microsoft...
- Mais Docker Hub est le plus populaire

### Voir les images sur votre ordinateur

**Commande pour lister vos images :**
```bash
docker images
```

**Ce que vous verrez :**
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
ubuntu        latest    3b418d7b466a   2 weeks ago    77MB
mysql         8.0       abc123def456   1 month ago    516MB
redis         alpine    789ghi012jkl   3 days ago     28MB
```

**Explications :**
- **REPOSITORY** : Le nom de l'image (ubuntu, mysql...)
- **TAG** : La version (latest = la dernière)
- **SIZE** : La taille de l'image
- **IMAGE ID** : Un code unique pour identifier l'image

---

## 📦 Partie 3 : C'est quoi un CONTENEUR Docker ?

### Définition simple

**Un CONTENEUR = Le GÂTEAU cuit** 🍰

Si l'image est la recette, le conteneur c'est le résultat : le gâteau que vous pouvez manger !

**En d'autres mots :**
- L'image = Le plan de la maison 📄
- Le conteneur = La maison construite 🏠

### Caractéristiques d'un conteneur

**1. Il est VIVANT** 💓
```
Un conteneur peut être :
- ⚡ En cours d'exécution (running)
- ⏸️  En pause (paused)
- 🛑 Arrêté (stopped)
- ❌ Supprimé (deleted)
```

**2. Il est MODIFIABLE** ✏️
- Vous pouvez y entrer
- Créer des fichiers
- Installer des choses
- Modifier des paramètres

**3. Il est ISOLÉ** 🔒
- Chaque conteneur est séparé des autres
- Un bug dans le conteneur A ne casse pas le conteneur B
- Comme des appartements dans un immeuble

**4. Il est TEMPORAIRE** ⏱️
- Vous le créez quand vous en avez besoin
- Vous le supprimez quand vous n'en voulez plus
- Les données disparaissent quand vous le supprimez (on verra comment les sauver plus tard)

### Relation IMAGE ↔ CONTENEUR

**La relation en schéma :**

```
       IMAGE                    CONTENEURS
     (La recette)              (Les gâteaux)

   ┌──────────┐              
   │  MySQL   │  ──────────→  📦 Conteneur 1
   │  Image   │  ──────────→  📦 Conteneur 2
   └──────────┘  ──────────→  📦 Conteneur 3

      1 IMAGE peut créer PLUSIEURS CONTENEURS
```

**Exemple concret :**

Vous avez une **image "nginx"** (serveur web).

Avec cette image, vous pouvez créer :
- Un conteneur pour votre site perso
- Un conteneur pour le site de votre client
- Un conteneur pour tester un nouveau design

**3 conteneurs différents, mais tous créés à partir de la même image !**

### Voir vos conteneurs

**Pour voir les conteneurs EN COURS d'exécution :**
```bash
docker ps
```

**Pour voir TOUS les conteneurs (même arrêtés) :**
```bash
docker ps -a
```

**Ce que vous verrez :**
```
CONTAINER ID   IMAGE     COMMAND      CREATED        STATUS       NAMES
a1b2c3d4e5f6   nginx     "nginx"      2 hours ago    Up 2 hours   mon-site-web
g7h8i9j0k1l2   mysql     "mysqld"     1 day ago      Up 1 day     ma-base
```

**Explications :**
- **CONTAINER ID** : Un code unique pour le conteneur
- **IMAGE** : De quelle image il vient
- **STATUS** : Est-il actif (Up) ou arrêté (Exited)
- **NAMES** : Le nom du conteneur (vous pouvez le choisir)

---

## 🔄 Partie 4 : Cycle de vie d'un conteneur

### Les 5 états d'un conteneur

**1. CRÉATION** 🐣
```bash
docker create nginx
```
Le conteneur existe mais ne fonctionne pas encore.

**2. DÉMARRAGE** ▶️
```bash
docker start mon-conteneur
```
Le conteneur commence à fonctionner.

**3. EN COURS D'EXÉCUTION** 🏃
```bash
docker ps
```
Le conteneur fonctionne normalement.

**4. ARRÊT** ⏹️
```bash
docker stop mon-conteneur
```
Le conteneur s'arrête mais existe toujours.

**5. SUPPRESSION** 🗑️
```bash
docker rm mon-conteneur
```
Le conteneur est définitivement supprimé.

### Le cycle en schéma

```
    [CRÉATION]
        ⬇️
    docker start
        ⬇️
   [EN MARCHE] ←──────┐
        ⬇️             │
    docker stop    docker restart
        ⬇️             │
    [ARRÊTÉ] ──────────┘
        ⬇️
    docker rm
        ⬇️
   [SUPPRIMÉ]
```

### Exemple complet en pratique

**Scénario : Lancer un serveur web nginx**

**Étape 1 : Créer et démarrer en une seule commande**
```bash
docker run -d --name mon-serveur nginx
```
- `run` = créer + démarrer en même temps
- `-d` = en arrière-plan (détaché)
- `--name` = donner un nom au conteneur
- `nginx` = l'image à utiliser

**Étape 2 : Vérifier qu'il tourne**
```bash
docker ps
```
Vous voyez : `mon-serveur` dans la liste ✅

**Étape 3 : Arrêter le conteneur**
```bash
docker stop mon-serveur
```

**Étape 4 : Redémarrer le conteneur**
```bash
docker start mon-serveur
```

**Étape 5 : Supprimer le conteneur**
```bash
docker stop mon-serveur    # D'abord l'arrêter
docker rm mon-serveur      # Puis le supprimer
```

**OU en une commande (forcer la suppression) :**
```bash
docker rm -f mon-serveur
```

---

## 🏪 Partie 5 : Le Docker Hub (le magasin d'images)

### C'est quoi Docker Hub ?

**Docker Hub = Le Play Store / App Store de Docker** 📱

C'est un site web où vous trouvez des milliers d'images Docker gratuites et prêtes à utiliser.

**Adresse :** https://hub.docker.com

### Ce qu'on trouve sur Docker Hub

**Des images officielles :**
- ✅ **ubuntu** - Le système Ubuntu
- ✅ **mysql** - Base de données MySQL
- ✅ **postgres** - Base de données PostgreSQL
- ✅ **redis** - Système de cache Redis
- ✅ **nginx** - Serveur web Nginx
- ✅ **node** - Node.js pour JavaScript
- ✅ **python** - Python
- Et des milliers d'autres...

### Comment chercher une image ?

**Sur le site Docker Hub :**
1. Allez sur https://hub.docker.com
2. Tapez le nom dans la barre de recherche (exemple : "postgres")
3. Cliquez sur le résultat
4. Vous voyez la commande pour l'installer !

**Directement dans le terminal :**
```bash
docker search postgres
```

Vous verrez une liste des images disponibles.

### Comment télécharger une image ?

**Commande :**
```bash
docker pull nom-de-limage
```

**Exemples :**
```bash
docker pull ubuntu        # Télécharge Ubuntu
docker pull mysql         # Télécharge MySQL
docker pull redis:alpine  # Télécharge Redis (version Alpine)
```

**Ce qui se passe :**
1. Docker cherche l'image sur Docker Hub
2. Il la télécharge sur votre ordinateur
3. L'image est maintenant disponible localement
4. Vous pouvez créer des conteneurs avec !

### Les TAGS (versions)

**Chaque image peut avoir plusieurs versions (tags)**

**Exemple avec Ubuntu :**
- `ubuntu:22.04` → Ubuntu version 22.04
- `ubuntu:20.04` → Ubuntu version 20.04
- `ubuntu:latest` → La dernière version

**Comment utiliser un tag spécifique :**
```bash
docker pull ubuntu:22.04
docker run ubuntu:22.04
```

**Si vous ne précisez pas de tag, Docker prend "latest" par défaut :**
```bash
docker pull ubuntu       # = ubuntu:latest
```

### Images officielles vs non-officielles

**Images OFFICIELLES** ⭐
- Vérifiées par Docker
- Sûres et maintenues
- Badge "Official Image"
- Exemples : ubuntu, mysql, nginx

**Images COMMUNAUTAIRES** 👥
- Créées par des développeurs
- Peuvent être bonnes mais vérifiez avant !
- Format : `utilisateur/nom-image`
- Exemple : `john/mon-application`

**Conseil :** Utilisez toujours les images officielles quand c'est possible !

---

## 🧩 Partie 6 : Résumé avec analogie complète

Pour bien tout comprendre, voici **l'analogie de la construction**

### 🏗️ Construire des maisons

**1. LE PLAN (= Image)**
```
Vous avez le plan d'une maison type "Maison Moderne"
- Le plan montre comment construire
- Il est en lecture seule (vous ne le modifiez pas)
- Vous pouvez l'utiliser plusieurs fois
```

**2. LES MAISONS (= Conteneurs)**
```
Avec ce plan, vous construisez 3 maisons :
- Maison A : pour vous
- Maison B : pour votre ami
- Maison C : pour la vendre

Chaque maison est différente maintenant :
- Vous peignez la maison A en bleu
- Votre ami met des meubles dans la maison B
- La maison C est encore vide
```

**3. LE MAGASIN DE PLANS (= Docker Hub)**
```
Vous allez au magasin de plans et vous trouvez :
- Plan "Maison Moderne"
- Plan "Maison Traditionnelle"  
- Plan "Appartement"
- Des milliers d'autres plans !

Vous choisissez celui que vous voulez et vous le téléchargez.
```

**4. L'ENTREPRISE DE CONSTRUCTION (= Docker Engine)**
```
C'est l'entreprise qui construit réellement les maisons.
Vous leur donnez le plan, ils construisent la maison.
```

### Traduction en Docker

| Analogie | Docker | Exemple |
|----------|--------|---------|
| Plan de maison | Image | `ubuntu`, `mysql` |
| Maison construite | Conteneur | Votre base de données MySQL qui tourne |
| Magasin de plans | Docker Hub | https://hub.docker.com |
| Entreprise de construction | Docker Engine | Le programme qui fait marcher Docker |
| Vous (le client) | Docker Client | Vous qui tapez les commandes |

---

## 📝 Partie 7 : Exercice pratique simple

### Exercice : Comprendre la différence Image/Conteneur

**Lisez cette situation :**

Thomas télécharge l'image `nginx`. Avec cette image, il crée 2 conteneurs :
- **Conteneur A** : Pour son site personnel (il le configure avec son contenu)
- **Conteneur B** : Pour tester un nouveau design

Ensuite, il supprime le conteneur B car il n'en a plus besoin.

**Questions :**

**Question 1 :** Est-ce que l'image nginx est supprimée quand Thomas supprime le conteneur B ?

**Question 2 :** Combien de conteneurs Thomas peut-il créer avec l'image nginx ?

**Question 3 :** Si Thomas modifie des fichiers dans le conteneur A, est-ce que ça change l'image nginx ?

<details>
<summary>👉 Cliquez pour voir les réponses</summary>

**Réponse 1 :**
Non ! L'image nginx reste sur l'ordinateur. Supprimer un conteneur ne supprime PAS l'image. L'image c'est la recette, elle reste même si vous jetez le gâteau !

**Réponse 2 :**
Autant qu'il veut ! Une image peut créer 1, 10, 100 conteneurs. Il n'y a pas de limite. C'est comme une recette : vous pouvez faire autant de gâteaux que vous voulez avec la même recette.

**Réponse 3 :**
Non ! Les modifications dans un conteneur ne changent PAS l'image. L'image est en lecture seule. C'est comme si vous cuisinez un gâteau et que vous le décorez : ça ne change pas la recette originale !

</details>

---

## ✅ Quiz de validation

**Question 1 : Une image Docker, c'est quoi ?**

- A) Un conteneur qui fonctionne
- B) Un modèle (comme une recette) pour créer des conteneurs
- C) Un fichier image (jpg, png)
- D) Un programme qui s'exécute

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B**

Une image c'est comme une recette ou un plan. C'est le modèle qui sert à créer des conteneurs. Elle ne "fonctionne" pas toute seule.

</details>

---

**Question 2 : Un conteneur Docker, c'est quoi ?**

- A) Une image téléchargée
- B) Docker Hub
- C) Une instance en cours d'exécution créée à partir d'une image
- D) Un fichier ZIP

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C**

Un conteneur c'est l'image "en action". C'est ce qui tourne réellement sur votre ordinateur. Une image qui prend vie !

</details>

---

**Question 3 : Combien de conteneurs peut-on créer avec UNE image ?**

- A) Un seul
- B) Maximum 10
- C) Autant qu'on veut
- D) Ça dépend de l'image

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C**

Avec une seule image, vous pouvez créer autant de conteneurs que vous voulez ! C'est tout l'intérêt : réutiliser le même modèle plusieurs fois.

</details>

---

**Question 4 : C'est quoi Docker Hub ?**

- A) Un logiciel à installer
- B) Un site web où on trouve des images Docker gratuites
- C) Un type de conteneur
- D) Une commande Docker

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B**

Docker Hub c'est comme un magasin en ligne où vous trouvez des milliers d'images Docker prêtes à utiliser. C'est gratuit et accessible à tous !

</details>

---

**Question 5 : Quand vous supprimez un conteneur, que se passe-t-il ?**

- A) L'image est aussi supprimée
- B) Tous les conteneurs créés avec cette image sont supprimés
- C) Seulement le conteneur est supprimé, l'image reste
- D) Docker se désinstalle

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C**

Supprimer un conteneur ne supprime PAS l'image. L'image reste sur votre ordinateur et vous pouvez créer d'autres conteneurs avec. C'est indépendant !

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Les 6 points essentiels de ce cours :

**1️⃣ L'architecture Docker**
- Docker Engine = Le moteur qui fait marcher tout
- Docker Client = Vous qui donnez les ordres
- Docker Hub = Le magasin d'images

**2️⃣ Une IMAGE**
- = Un modèle, une recette, un plan
- En lecture seule (ne change pas)
- Peut créer plusieurs conteneurs
- Se télécharge depuis Docker Hub

**3️⃣ Un CONTENEUR**
- = Une image qui tourne, qui est "vivante"
- On peut le modifier
- Il est isolé des autres conteneurs
- On peut le créer, démarrer, arrêter, supprimer

**4️⃣ La relation**
```
1 IMAGE ──────→ Plusieurs CONTENEURS
(La recette)     (Les gâteaux cuisinés)
```

**5️⃣ Docker Hub**
- Site web avec des milliers d'images gratuites
- Images officielles = sûres et vérifiées
- Commande : `docker pull nom-image`

**6️⃣ Le cycle de vie d'un conteneur**
```
Création → Démarrage → En marche → Arrêt → Suppression
```

### Ce que vous devez pouvoir expliquer :

**"C'est quoi la différence entre une image et un conteneur ?"**
→ *"L'image c'est la recette, le conteneur c'est le gâteau cuit. L'image ne bouge pas, le conteneur fonctionne vraiment."*

**"Docker Hub c'est quoi ?"**
→ *"C'est un site web où il y a plein d'images Docker gratuites. MySQL, Ubuntu, Redis... tout est là, prêt à télécharger."*

**"On peut faire combien de conteneurs avec une image ?"**
→ *"Autant qu'on veut ! C'est comme une recette : on peut cuisiner 10 gâteaux avec la même recette."*

---

## 🚀 Et maintenant ?

**Bravo ! Vous avez terminé le cours 2 !** 🎉

### Ce que vous savez maintenant :

- ✅ Comment Docker fonctionne (architecture)
- ✅ C'est quoi une image (le modèle)
- ✅ C'est quoi un conteneur (l'instance qui tourne)
- ✅ La différence entre les deux
- ✅ Docker Hub et comment trouver des images

### Dans le prochain cours (Cours 3) :

**On passe ENFIN à la PRATIQUE !** 💪

Vous allez apprendre :
1. Installer votre première image
2. Créer votre premier conteneur
3. Les commandes essentielles
4. Manipuler les conteneurs (start, stop, logs...)
5. Faire des exercices pratiques

**Préparez votre terminal, on va coder !** 🖥️

---

## 📚 Mini-glossaire (les mots à connaître)

| Mot | Définition simple |
|-----|------------------|
| **Image** | Le modèle pour créer un conteneur (comme une recette) |
| **Conteneur** | Un programme qui tourne, créé à partir d'une image |
| **Docker Hub** | Le site web avec plein d'images gratuites |
| **Docker Engine** | Le moteur principal de Docker |
| **Tag** | La version d'une image (ex: ubuntu:22.04) |
| **Pull** | Télécharger une image |
| **Run** | Créer et démarrer un conteneur |
| **PS** | Voir les conteneurs qui tournent |

---

**🎓 Vous êtes prêt pour le Cours 3 (LA PRATIQUE) !**

Bon courage ! N'hésitez pas à relire ce cours si besoin. Docker c'est simple, vous allez voir ! 💪

**Conseil :** Avant de passer au cours 3, assurez-vous de bien comprendre la différence entre IMAGE et CONTENEUR. C'est LE concept le plus important !
