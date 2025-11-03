# Cours 4 : Les Ports - Accéder à vos Conteneurs 🚪

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Pourquoi vos conteneurs sont "invisibles" par défaut
- C'est quoi un port (expliqué simplement)
- Comment exposer un conteneur avec `-p`
- La différence entre port de l'hôte et port du conteneur
- Gérer plusieurs services sur des ports différents
- Résoudre les conflits de ports

**Durée : 1 heure (lecture + pratique)**

**Prérequis : Cours 03 (Premières Commandes)**

---

## 😱 Partie 1 : LE PROBLÈME - Mon conteneur est invisible !

### 🎬 Scène 1 : L'étudiant confus

**Vous (l'étudiant) :** "Chef ! J'ai lancé Nginx dans un conteneur !"

**Le formateur :** "Super ! Ouvre ton navigateur sur `http://localhost` pour voir le site !"

**Vous :** *[Ouvre le navigateur]* "Euh... il n'y a rien... 😰"

**Le formateur :** "Vérifie que ton conteneur tourne !"

**Vous :**
```bash
docker ps
```
```
CONTAINER ID   IMAGE   STATUS      PORTS     NAMES
abc123def456   nginx   Up 2 min    80/tcp    mon-nginx
```

**Vous :** "Il tourne ! Mais le site ne s'affiche pas dans le navigateur..."

**Le formateur :** "Ah ! Tu as oublié d'**EXPOSER LE PORT** ! Le conteneur tourne, mais tu ne peux pas y accéder depuis ton PC !"

---

## 🤔 Partie 2 : COMPRENDRE LE PROBLÈME

### L'analogie de la maison

**Imaginez cette situation :**

Vous construisez une maison (= votre conteneur).
Dans cette maison, il y a un serveur web Nginx qui tourne.

**Le problème :**

```
┌──────────────────────────────────────────┐
│           VOTRE PC (l'hôte)              │
│                                          │
│   Vous dans le navigateur                │
│   "Je veux voir le site !"               │
│                ❌                         │
│                │                          │
│        Pas de porte !                    │
│                │                          │
│   ┌────────────▼──────────────┐          │
│   │   CONTENEUR (la maison)   │          │
│   │                           │          │
│   │   🏠 Nginx tourne sur     │          │
│   │      le port 80           │          │
│   │                           │          │
│   │   MAIS personne ne peut   │          │
│   │   entrer de l'extérieur ! │          │
│   └───────────────────────────┘          │
│                                          │
└──────────────────────────────────────────┘
```

**Le conteneur tourne bien**, mais il est **isolé** !
C'est comme une maison **sans porte** : impossible d'entrer !

---

### Pourquoi Docker fait ça ?

**C'est voulu !** Docker isole les conteneurs **par sécurité**.

**Imaginez :**
- Vous lancez 10 conteneurs
- Chacun a un serveur web sur le port 80
- Si tous étaient accessibles automatiquement → CHAOS ! Conflits de ports partout !

**Docker dit :** "Si vous voulez accéder à un conteneur, DEMANDEZ-LE explicitement !"

---

## 🚪 Partie 3 : LA SOLUTION - Le Mapping de Ports

### C'est quoi un port ?

**Analogie simple :**

Votre PC = Un immeuble
Chaque port = Un numéro d'appartement

```
┌─────────────────────────────┐
│    VOTRE PC (l'immeuble)    │
│                             │
│  Port 80   → Appartement 80 │
│  Port 8080 → Appartement 8080│
│  Port 3000 → Appartement 3000│
│  Port 5432 → Appartement 5432│
│                             │
└─────────────────────────────┘
```

**Quand vous tapez `http://localhost:8080` :**
→ Vous frappez à la porte de l'**appartement 8080**

**Quelques ports célèbres :**
- **Port 80** : HTTP (sites web)
- **Port 443** : HTTPS (sites web sécurisés)
- **Port 3306** : MySQL
- **Port 5432** : PostgreSQL
- **Port 6379** : Redis
- **Port 8080** : Souvent utilisé en développement

---

### Le mapping de ports avec `-p`

**La commande magique :**

```bash
docker run -d -p 8080:80 --name mon-nginx nginx
```

**Décomposons :**

| Partie | Signification |
|--------|---------------|
| `docker run` | Lance un conteneur |
| `-d` | Mode détaché (en fond) |
| `-p 8080:80` | **MAPPE le port 8080 de votre PC vers le port 80 du conteneur** |
| `--name mon-nginx` | Donne un nom au conteneur |
| `nginx` | Image à utiliser |

---

### 🎯 DÉCORTIQUONS `-p 8080:80`

**Syntaxe générale :**
```
-p  PORT_DE_VOTRE_PC : PORT_DU_CONTENEUR
```

**Dans notre cas :**
```
-p  8080 : 80
    ▲▲▲▲   ▲▲
    │      │
    │      └─ Port 80 DANS le conteneur (là où Nginx écoute)
    │
    └─ Port 8080 sur VOTRE PC (là où vous allez taper dans le navigateur)
```

**Traduction en français :**
"Docker, quand quelqu'un frappe à la porte 8080 de mon PC, redirige-le vers la porte 80 du conteneur !"

---

### Le tunnel magique

```
┌──────────────────────────────────────────────────────┐
│               VOTRE PC                               │
│                                                      │
│   Navigateur Web                                     │
│   http://localhost:8080  ──┐                         │
│                            │                         │
│                            │ -p 8080:80              │
│                            │ (le tunnel)             │
│                            ▼                         │
│   ┌────────────────────────────────────┐            │
│   │   CONTENEUR                        │            │
│   │                                    │            │
│   │   🚪 Port 80                       │            │
│   │   Nginx écoute ici !               │            │
│   │                                    │            │
│   └────────────────────────────────────┘            │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Grâce à `-p 8080:80`, vous créez un "tunnel" entre votre PC et le conteneur !**

---

## 🎬 Partie 4 : TESTEZ VOUS-MÊME !

### Exercice 1 : Lancer Nginx avec exposition de port

**Étape 1 : Lancez Nginx AVEC `-p`**

```bash
docker run -d -p 8080:80 --name mon-nginx nginx
```

**Étape 2 : Vérifiez qu'il tourne**

```bash
docker ps
```

**Vous devriez voir :**
```
CONTAINER ID   IMAGE   STATUS      PORTS                  NAMES
abc123def456   nginx   Up 5 sec    0.0.0.0:8080->80/tcp   mon-nginx
```

**Regardez la colonne `PORTS` :**
- `0.0.0.0:8080->80/tcp`
- Ça veut dire : "Le port 8080 de ton PC (0.0.0.0) redirige vers le port 80 du conteneur"

✅ **Parfait !**

**Étape 3 : Ouvrez votre navigateur**

Allez sur : `http://localhost:8080`

**Vous devriez voir :**
```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed...
```

**🎉 FÉLICITATIONS ! Vous accédez à votre conteneur pour la première fois !**

---

### 🧠 Comprendre ce qui s'est passé

**Le voyage de votre requête HTTP :**

```
1. Vous tapez http://localhost:8080 dans le navigateur
                    ▼
2. Votre navigateur envoie une requête au port 8080 de votre PC
                    ▼
3. Docker intercepte cette requête (grâce à -p 8080:80)
                    ▼
4. Docker la redirige vers le port 80 du conteneur
                    ▼
5. Nginx (qui écoute sur le port 80 du conteneur) reçoit la requête
                    ▼
6. Nginx renvoie la page HTML
                    ▼
7. Docker renvoie la réponse à votre navigateur
                    ▼
8. La page s'affiche ! ✨
```

**Tout ça en quelques millisecondes !**

---

### ❓ Questions fréquentes

**Q1 : Pourquoi 8080 et pas 80 directement ?**

Si vous faites `-p 80:80`, vous mappez le port 80 de votre PC.
**MAIS** le port 80 est souvent **déjà utilisé** sur votre PC !

Par convention, on utilise :
- `8080` pour le premier service
- `8081` pour le deuxième
- `8082` pour le troisième
- etc.

**Q2 : Je DOIS utiliser 8080 ?**

**Non !** Vous pouvez utiliser n'importe quel port libre :
- `-p 3000:80`
- `-p 9000:80`
- `-p 1234:80`

Le seul truc important : le port de gauche (votre PC) doit être **libre** !

**Q3 : Et si je veux utiliser le port 80 de mon PC ?**

C'est possible, mais il faut que le port 80 soit libre :
```bash
docker run -d -p 80:80 --name mon-nginx nginx
```

Ensuite vous accédez directement avec `http://localhost` (sans `:8080`)

---

## 💪 Partie 5 : EXERCICES PRATIQUES

### ✏️ Exercice 2 : Lancer plusieurs Nginx sur des ports différents

**Mission : Créer 3 serveurs Nginx accessibles sur 3 ports différents**

**Étape 1 : Créez 3 conteneurs**

```bash
docker run -d -p 8080:80 --name nginx1 nginx
docker run -d -p 8081:80 --name nginx2 nginx
docker run -d -p 8082:80 --name nginx3 nginx
```

**Étape 2 : Vérifiez qu'ils tournent tous**

```bash
docker ps
```

Vous devriez voir les 3 conteneurs avec des ports différents :
```
PORTS
0.0.0.0:8080->80/tcp
0.0.0.0:8081->80/tcp
0.0.0.0:8082->80/tcp
```

**Étape 3 : Testez dans le navigateur**

Ouvrez **3 onglets** :
- `http://localhost:8080`
- `http://localhost:8081`
- `http://localhost:8082`

**Tous les 3 affichent la même page Nginx !** 🎉

**Ce que vous venez de faire :**
- 3 serveurs web qui tournent **en même temps**
- Chacun est **accessible** sur un port différent
- Chacun est **isolé** des autres

---

### ✏️ Exercice 3 : Comprendre les conflits de ports

**Mission : Provoquer une erreur de conflit de ports (pour comprendre !)**

**Étape 1 : Essayez de lancer un 4ème Nginx sur le port 8080**

```bash
docker run -d -p 8080:80 --name nginx4 nginx
```

**💥 ERREUR !**

```
Error response from daemon: driver failed programming external connectivity:
Bind for 0.0.0.0:8080 failed: port is already allocated
```

**Pourquoi ?**
- Le port 8080 est **déjà utilisé** par `nginx1` !
- Docker refuse de mapper deux fois le même port

**Solution :**
Utilisez un autre port !

```bash
docker run -d -p 8083:80 --name nginx4 nginx
```

✅ **Ça marche !**

---

### ✏️ Exercice 4 : Nettoyer tout

**Arrêtez et supprimez tous les conteneurs :**

```bash
docker stop nginx1 nginx2 nginx3 nginx4
docker rm nginx1 nginx2 nginx3 nginx4
```

**Ou en une seule ligne (plus rapide) :**

```bash
docker rm -f nginx1 nginx2 nginx3 nginx4
```

Le `-f` = force (arrête ET supprime en même temps)

---

## 🎓 Partie 6 : CAS PRATIQUES

### Cas 1 : Plusieurs applications différentes

**Vous pouvez faire tourner différents services sur différents ports !**

```bash
# Un site web sur le port 8080
docker run -d -p 8080:80 --name mon-site nginx

# Une API sur le port 3000
docker run -d -p 3000:3000 --name mon-api node:20

# Une base de données sur le port 5432
docker run -d -p 5432:5432 --name ma-db postgres
```

**Maintenant vous avez :**
- Site web → `http://localhost:8080`
- API → `http://localhost:3000`
- Base de données → accessible sur `localhost:5432`

**Chacun est accessible depuis votre PC !**

---

### Cas 2 : Mapper plusieurs ports pour un même conteneur

**Certaines applications utilisent plusieurs ports.**

Exemple : Un serveur web + un port d'administration

```bash
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  --name mon-serveur nginx
```

**Maintenant :**
- HTTP → `http://localhost:8080`
- HTTPS → `https://localhost:8443`

**Vous pouvez mapper autant de ports que vous voulez !**

---

### Cas 3 : Laisser Docker choisir le port

**Vous pouvez laisser Docker choisir un port aléatoire :**

```bash
docker run -d -p 80 --name nginx-auto nginx
```

**Puis vérifier quel port a été choisi :**

```bash
docker ps
```

```
PORTS
0.0.0.0:32768->80/tcp
```

Docker a choisi le port `32768` automatiquement !

**Utile quand :**
- Vous voulez lancer plein de conteneurs rapidement
- Le port exact ne vous importe pas

---

## 📋 Partie 7 : AIDE-MÉMOIRE

### Les commandes essentielles

| Commande | Ce que ça fait |
|----------|----------------|
| `docker run -d -p 8080:80 nginx` | Lance Nginx, port 8080 de votre PC → port 80 du conteneur |
| `docker run -d -p 80:80 nginx` | Lance Nginx, port 80 de votre PC → port 80 du conteneur |
| `docker run -d -p 3000:8080 nginx` | Port 3000 de votre PC → port 8080 du conteneur |
| `docker ps` | Voir les ports mappés dans la colonne PORTS |

### Syntaxe `-p` récapitulative

```
-p  PORT_HÔTE : PORT_CONTENEUR

PORT_HÔTE      = Le port sur VOTRE PC (localhost)
PORT_CONTENEUR = Le port DANS le conteneur
```

**Exemples :**
- `-p 8080:80` → localhost:8080 → conteneur:80
- `-p 3000:3000` → localhost:3000 → conteneur:3000
- `-p 80:8080` → localhost:80 → conteneur:8080

---

## ❌ Partie 8 : Les erreurs fréquentes

### Erreur 1 : "port is already allocated"

**Message :**
```
Bind for 0.0.0.0:8080 failed: port is already allocated
```

**Problème :** Le port 8080 est déjà utilisé

**Solutions :**

**Solution 1 : Utiliser un autre port**
```bash
docker run -d -p 8081:80 nginx
```

**Solution 2 : Trouver qui utilise le port 8080**
```bash
docker ps
```
→ Cherchez quel conteneur utilise `8080` dans la colonne `PORTS`

**Solution 3 : Arrêter le conteneur qui utilise le port**
```bash
docker stop nom-du-conteneur
```

---

### Erreur 2 : "Cannot connect" dans le navigateur

**Message dans le navigateur :**
```
This site can't be reached
localhost refused to connect
```

**Problèmes possibles :**

**1. Vous avez oublié le `-p` !**
```bash
# ❌ MAUVAIS
docker run -d nginx

# ✅ BON
docker run -d -p 8080:80 nginx
```

**2. Le conteneur n'est pas démarré**
```bash
docker ps  # Vérifiez qu'il est dans la liste
```

**3. Vous utilisez le mauvais port**
```bash
docker ps  # Vérifiez la colonne PORTS
```

Si vous voyez `0.0.0.0:8080->80/tcp`, utilisez `localhost:8080` (pas `localhost:80`)

---

### Erreur 3 : Confusion port hôte vs port conteneur

**❌ ERREUR CLASSIQUE :**

Vous lancez :
```bash
docker run -d -p 3000:80 nginx
```

Puis vous essayez : `http://localhost:80` ❌

**✅ BON :**
Vous devez utiliser le **port de gauche** (port de l'hôte) : `http://localhost:3000`

**Règle simple :**
Dans le navigateur, utilisez toujours le **premier nombre** de `-p` !

---

## ✅ Quiz final

**Question 1 : Que fait `-p 8080:80` ?**
- A) Ouvre le port 8080 du conteneur
- B) Mappe le port 8080 de votre PC vers le port 80 du conteneur
- C) Ouvre le port 80 de votre PC

<details>
<summary>Voir la réponse</summary>
✅ **B) Mappe le port 8080 de votre PC vers le port 80 du conteneur**
</details>

---

**Question 2 : Vous lancez `docker run -d -p 3000:80 nginx`. Quelle URL utiliser ?**
- A) http://localhost:80
- B) http://localhost:3000
- C) http://localhost:8080

<details>
<summary>Voir la réponse</summary>
✅ **B) http://localhost:3000**

C'est toujours le **premier port** (port de l'hôte) dans `-p 3000:80`
</details>

---

**Question 3 : Pourquoi mon `docker run -d -p 8080:80 nginx` ne marche pas ?**

<details>
<summary>Voir la réponse</summary>
**Réponse :** Le port 8080 est probablement déjà utilisé !

**Solution :** Utilisez un autre port : `-p 8081:80`
</details>

---

**Question 4 : Je veux 2 Nginx qui tournent en même temps. Que faire ?**
- A) C'est impossible
- B) Utiliser 2 ports différents : `-p 8080:80` et `-p 8081:80`
- C) Utiliser le même port

<details>
<summary>Voir la réponse</summary>
✅ **B) Utiliser 2 ports différents**

Exemple :
```bash
docker run -d -p 8080:80 --name nginx1 nginx
docker run -d -p 8081:80 --name nginx2 nginx
```
</details>

---

## 🎯 Récapitulatif final

### Ce que vous avez appris aujourd'hui :

✅ Pourquoi les conteneurs sont isolés par défaut
✅ C'est quoi un port (un "numéro d'appartement")
✅ Utiliser `-p` pour exposer des conteneurs
✅ La syntaxe `-p PORT_HÔTE:PORT_CONTENEUR`
✅ Lancer plusieurs services sur des ports différents
✅ Résoudre les conflits de ports
✅ Débugger les problèmes de connexion

### La règle d'or :

**Pour accéder à un conteneur depuis votre PC, vous DEVEZ utiliser `-p` !**

```bash
docker run -d -p 8080:80 --name mon-app nginx
```

**Puis ouvrez :** `http://localhost:8080`

---

## 🚀 Et maintenant ?

**Dans le prochain cours (Cours 5), vous apprendrez :**

📘 **Les Volumes** → Persister vos données et modifier du code en temps réel !

**Vous saurez :**
- Pourquoi vos données disparaissent quand vous supprimez un conteneur
- Comment sauvegarder vos données
- Comment développer en modifiant du code directement (sans reconstruire !)

**Entraînez-vous bien avec les exercices sur les ports avant de passer au cours 5 !**

---

**Aide-mémoire ultra-court :**
```bash
docker run -d -p 8080:80 nginx  # Lance Nginx accessible sur localhost:8080
docker ps                       # Voir les ports mappés
docker stop nom                 # Arrêter
docker rm nom                   # Supprimer
```

**Bon courage ! 🐳**
