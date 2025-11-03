# Cours 6 : Les Réseaux - Connecter vos Conteneurs 🔗

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Pourquoi vos conteneurs ne peuvent pas se parler par défaut
- C'est quoi un réseau Docker (expliqué simplement)
- Comment créer un réseau
- Comment connecter des conteneurs ensemble
- La "magie" de la résolution DNS (trouver un conteneur par son nom)
- Les différents types de réseaux et quand les utiliser

**Durée : 1h30 (lecture + pratique)**

**Prérequis : Cours 04 (Les Ports) + Cours 05 (Les Volumes)**

---

## 😱 Partie 1 : LE PROBLÈME - Mes conteneurs ne se parlent pas !

### 🎬 Scène 1 : L'application qui ne marche pas

**Vous (l'étudiant) :** "Chef ! J'ai créé mon application web !"

**Le formateur :** "Super ! C'est quoi ton architecture ?"

**Vous :** "Un conteneur pour le site web (PHP), un conteneur pour la base de données (MySQL) !"

**Le formateur :** "Parfait ! Lance tout et montre-moi !"

**Vous lancez les 2 conteneurs :**
```bash
docker run -d --name ma-db mysql:8.0
docker run -d --name mon-site php:8.2-apache
```

**Vous :** "C'est lancé ! Mais... le site ne peut pas se connecter à la base de données... 😰"

**Le formateur :** "Ah ! Tes 2 conteneurs ne sont pas dans le **MÊME RÉSEAU** ! Ils ne peuvent pas se parler !"

**Vous :** "Hein ? Mais ils sont sur le même PC !"

**Le formateur :** "Oui, mais Docker les **ISOLE** ! C'est comme s'ils étaient dans 2 pièces séparées sans téléphone !"

---

## 🤔 Partie 2 : COMPRENDRE LE PROBLÈME

### L'analogie des pièces isolées

**Imaginez cette situation :**

Vous avez 2 personnes dans 2 pièces différentes :
- **Pièce 1** = Conteneur PHP (votre site web)
- **Pièce 2** = Conteneur MySQL (votre base de données)

**Le problème :**

```
┌─────────────────────────────────────────────┐
│              VOTRE PC                       │
│                                             │
│   ┌─────────────────┐   ┌──────────────┐   │
│   │  Pièce 1        │   │  Pièce 2     │   │
│   │                 │   │              │   │
│   │  PHP            │ X │  MySQL       │   │
│   │  "MySQL, tu     │   │              │   │
│   │   es où ?"      │   │  "Je suis    │   │
│   │                 │   │   là !"      │   │
│   └─────────────────┘   └──────────────┘   │
│           ▲                     ▲           │
│           │                     │           │
│        Conteneur             Conteneur      │
│         isolé                 isolé         │
│                                             │
└─────────────────────────────────────────────┘
```

**PHP essaie d'appeler MySQL, mais il n'y a PAS de téléphone entre les 2 pièces !**

Les conteneurs sont **isolés** les uns des autres.

---

### Pourquoi Docker isole les conteneurs ?

**3 raisons :**

**1. Sécurité**
- Si un conteneur est piraté, les autres restent protégés
- Isolation = protection

**2. Éviter les conflits**
- Imaginez 10 applications qui essaient toutes de se connecter à "localhost"
- Chaos total ! Qui parle à qui ?

**3. Flexibilité**
- Vous décidez explicitement qui parle à qui
- Vous contrôlez votre architecture

**Docker dit :** "Si vous voulez que 2 conteneurs communiquent, créez un réseau et mettez-les dedans !"

---

## 🔗 Partie 3 : LA SOLUTION - Le Réseau Docker

### C'est quoi un réseau Docker ?

**Analogie simple :**

Un réseau Docker = **Un groupe WhatsApp pour vos conteneurs**

```
┌─────────────────────────────────────────────┐
│       Groupe WhatsApp "mon-reseau"          │
│                                             │
│   👤 PHP         👤 MySQL      👤 Redis     │
│                                             │
│   "Hé MySQL,     "Oui PHP,     "Salut      │
│    donne-moi      voilà les     les amis   │
│    les données"   données !"    !"         │
│                                             │
│   Ils peuvent TOUS se parler !              │
└─────────────────────────────────────────────┘
```

**Grâce au réseau :**
- Les conteneurs peuvent se trouver par leur **nom**
- Ils peuvent se parler directement
- Ils restent isolés des conteneurs hors du réseau

---

### Créer un réseau

**La commande magique :**

```bash
docker network create mon-reseau
```

**C'est tout !** Vous venez de créer un réseau ! 🎉

**Vérifier que ça a marché :**

```bash
docker network ls
```

**Vous verrez :**
```
NETWORK ID     NAME          DRIVER    SCOPE
abc123def456   mon-reseau    bridge    local
xyz789ghi012   bridge        bridge    local
...
```

✅ `mon-reseau` apparaît dans la liste !

---

### Connecter des conteneurs au réseau

**Méthode 1 : Au moment du lancement (recommandé)**

```bash
docker run -d --name ma-db --network mon-reseau mysql:8.0
docker run -d --name mon-site --network mon-reseau php:8.2-apache
```

**Regardez le `--network mon-reseau` !**
- C'est comme dire : "Mets ce conteneur dans le groupe WhatsApp 'mon-reseau' !"

**Méthode 2 : Après coup (si vous avez oublié)**

```bash
# Lancer sans réseau
docker run -d --name ma-db mysql:8.0

# Oups ! J'ai oublié le réseau. Pas grave, je le connecte maintenant :
docker network connect mon-reseau ma-db
```

---

## ✨ Partie 4 : LA MAGIE - Trouver un conteneur par son nom

### Le super pouvoir du réseau Docker

**Une fois que vos conteneurs sont dans le même réseau, ils peuvent se trouver par leur NOM !**

**Exemple :**

Vous avez lancé :
```bash
docker run -d --name ma-base --network mon-reseau mysql:8.0
docker run -d --name mon-app --network mon-reseau php:8.2-apache
```

**Dans le conteneur `mon-app`, vous pouvez faire :**

```bash
docker exec -it mon-app bash
```

**Puis à l'intérieur :**

```bash
ping ma-base
```

**Résultat :**
```
PING ma-base (172.18.0.2) 56(84) bytes of data.
64 bytes from ma-base.mon-reseau (172.18.0.2): icmp_seq=1 ttl=64 time=0.073 ms
```

**🎉 Ça marche ! Le conteneur trouve `ma-base` par son nom !**

---

### Comment ça marche ? Le DNS interne

**Docker a un "annuaire magique" (DNS) :**

```
┌──────────────────────────────────────┐
│      Réseau "mon-reseau"             │
│                                      │
│   Annuaire DNS :                     │
│   ├── ma-base    → 172.18.0.2        │
│   ├── mon-app    → 172.18.0.3        │
│   └── mon-cache  → 172.18.0.4        │
│                                      │
│   Quand "mon-app" demande            │
│   "Où est ma-base ?", Docker         │
│   répond : "172.18.0.2" !            │
└──────────────────────────────────────┘
```

**Vous n'avez PAS besoin de connaître les adresses IP !**
**Utilisez juste les NOMS !** 🎯

---

### Exemple concret : PHP se connecte à MySQL

**Dans votre code PHP :**

**❌ MAUVAIS (adresse IP - ça peut changer) :**
```php
$host = '172.18.0.2';  // Fragile !
$connexion = new PDO("mysql:host=$host;dbname=ma_base", $user, $pass);
```

**✅ BON (nom du conteneur - stable) :**
```php
$host = 'ma-base';  // Le nom du conteneur MySQL !
$connexion = new PDO("mysql:host=$host;dbname=ma_base", $user, $pass);
```

**Docker traduit automatiquement `ma-base` en adresse IP !**

---

## 💪 Partie 5 : EXERCICES PRATIQUES

### ✏️ Exercice 1 : Créer un réseau et connecter 2 conteneurs

**Mission : Faire communiquer 2 conteneurs Nginx**

**Étape 1 : Créez un réseau**

```bash
docker network create test-reseau
```

**Étape 2 : Lancez 2 conteneurs dans ce réseau**

```bash
docker run -d --name nginx1 --network test-reseau nginx
docker run -d --name nginx2 --network test-reseau nginx
```

**Étape 3 : Testez la communication**

Entrez dans le premier conteneur :
```bash
docker exec -it nginx1 bash
```

**À l'intérieur, installez `ping` (pas installé par défaut dans Nginx) :**
```bash
apt-get update && apt-get install -y iputils-ping
```

**Puis testez de pinguer le 2ème conteneur :**
```bash
ping nginx2
```

**Résultat :**
```
PING nginx2 (172.18.0.3) 56(84) bytes of data.
64 bytes from nginx2.test-reseau (172.18.0.3): icmp_seq=1 ttl=64
```

**🎉 Ça marche ! `nginx1` trouve `nginx2` par son nom !**

**Sortez du conteneur :**
```bash
exit
```

---

### ✏️ Exercice 2 : Tester SANS réseau (pour comprendre la différence)

**Mission : Voir ce qui se passe sans réseau**

**Étape 1 : Lancez 2 conteneurs SANS réseau**

```bash
docker run -d --name test1 nginx
docker run -d --name test2 nginx
```

**Étape 2 : Essayez de pinguer**

```bash
docker exec -it test1 bash
apt-get update && apt-get install -y iputils-ping
ping test2
```

**Résultat :**
```
ping: test2: Name or service not known
```

**❌ Ça ne marche PAS ! Docker ne peut pas trouver `test2` !**

**Pourquoi ?**
- Les 2 conteneurs ne sont **pas dans le même réseau**
- Ils sont **isolés**
- Docker ne peut pas résoudre le nom `test2`

**Sortez et nettoyez :**
```bash
exit
docker rm -f test1 test2
```

---

### ✏️ Exercice 3 : Application web + Base de données

**Mission : Créer une vraie architecture (Alpine Linux + MySQL)**

**Étape 1 : Créez un réseau**

```bash
docker network create app-network
```

**Étape 2 : Lancez MySQL**

```bash
docker run -d \
  --name ma-db \
  --network app-network \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=test_db \
  mysql:8.0
```

**Attendez 10 secondes** que MySQL démarre.

**Étape 3 : Lancez un conteneur Alpine (léger) pour tester**

```bash
docker run -it --network app-network alpine sh
```

**Vous êtes maintenant dans Alpine !**

**Étape 4 : Installez le client MySQL**

```sh
apk add mysql-client
```

**Étape 5 : Connectez-vous à MySQL par son NOM**

```sh
mysql -h ma-db -u root -ppassword
```

**Résultat :**
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
mysql>
```

**🎉 VOUS ÊTES CONNECTÉ ! Alpine trouve MySQL par son nom `ma-db` !**

**Tapez :**
```sql
SHOW DATABASES;
```

Vous verrez la base `test_db` !

**Sortez :**
```sql
exit
```

Puis quittez Alpine :
```sh
exit
```

**Nettoyez :**
```bash
docker rm -f ma-db
docker network rm app-network
```

---

## 📚 Partie 6 : Les différents types de réseaux

### Les 3 types principaux

Docker a plusieurs types de réseaux. Voici les 3 principaux :

| Type | Quand l'utiliser | Isolation |
|------|------------------|-----------|
| **bridge** | Applications multi-conteneurs (défaut) | Conteneurs peuvent se parler dans le réseau |
| **host** | Performance maximale (pas d'isolation réseau) | Conteneur utilise le réseau de l'hôte directement |
| **none** | Conteneur complètement isolé | Aucune connexion réseau |

---

### Type 1 : Bridge (par défaut)

**C'est ce qu'on a utilisé jusqu'ici !**

```bash
docker network create mon-reseau  # Type bridge par défaut
```

**Caractéristiques :**
- Les conteneurs dans le réseau peuvent se parler
- Les conteneurs hors du réseau sont isolés
- C'est le type le plus utilisé

**Cas d'usage :**
- Application web + Base de données
- Microservices
- 99% des cas !

---

### Type 2 : Host

**Le conteneur utilise DIRECTEMENT le réseau de votre PC.**

```bash
docker run -d --network host nginx
```

**Caractéristiques :**
- Pas d'isolation réseau
- Le conteneur voit tous les ports de votre PC
- Performance maximale (pas de traduction de ports)

**❌ Inconvénient :**
- Moins de sécurité
- Conflits de ports possibles

**Cas d'usage :**
- Applications qui ont besoin de performances réseau max
- Monitoring réseau
- Rarement utilisé en pratique

---

### Type 3 : None

**Le conteneur n'a AUCUN accès réseau.**

```bash
docker run -d --network none alpine
```

**Caractéristiques :**
- Isolation totale
- Pas d'accès Internet
- Pas de communication avec d'autres conteneurs

**Cas d'usage :**
- Traitement de données sensibles
- Jobs batch qui n'ont pas besoin de réseau
- Tests d'isolation

---

## 🎓 Partie 7 : Commandes utiles

### Voir tous les réseaux

```bash
docker network ls
```

**Résultat :**
```
NETWORK ID     NAME          DRIVER    SCOPE
abc123def456   bridge        bridge    local
xyz789ghi012   host          host      local
def456ghi789   none          null      local
jkl012mno345   mon-reseau    bridge    local
```

---

### Inspecter un réseau

```bash
docker network inspect mon-reseau
```

**Vous verrez :**
- Les conteneurs connectés
- Les adresses IP
- La configuration

**Exemple :**
```json
"Containers": {
    "abc123...": {
        "Name": "ma-db",
        "IPv4Address": "172.18.0.2/16"
    },
    "def456...": {
        "Name": "mon-app",
        "IPv4Address": "172.18.0.3/16"
    }
}
```

---

### Déconnecter un conteneur d'un réseau

```bash
docker network disconnect mon-reseau mon-app
```

**Maintenant `mon-app` n'est plus dans le réseau !**

---

### Supprimer un réseau

```bash
docker network rm mon-reseau
```

**⚠️ ATTENTION :** Vous devez d'abord arrêter/supprimer tous les conteneurs qui utilisent ce réseau !

**Si vous voyez cette erreur :**
```
Error: network mon-reseau has active endpoints
```

**Solution :**
```bash
# 1. Arrêtez les conteneurs
docker stop ma-db mon-app

# 2. Supprimez les conteneurs
docker rm ma-db mon-app

# 3. Maintenant vous pouvez supprimer le réseau
docker network rm mon-reseau
```

---

### Nettoyer les réseaux non utilisés

```bash
docker network prune
```

**Supprime tous les réseaux qui ne sont utilisés par aucun conteneur.**

---

## 📋 Partie 8 : AIDE-MÉMOIRE

### Les commandes essentielles

| Commande | Ce que ça fait |
|----------|----------------|
| `docker network create mon-reseau` | Crée un réseau |
| `docker network ls` | Liste tous les réseaux |
| `docker network inspect mon-reseau` | Détails d'un réseau |
| `docker run --network mon-reseau nginx` | Lance un conteneur dans un réseau |
| `docker network connect mon-reseau mon-conteneur` | Connecte un conteneur existant |
| `docker network disconnect mon-reseau mon-conteneur` | Déconnecte un conteneur |
| `docker network rm mon-reseau` | Supprime un réseau |
| `docker network prune` | Nettoie les réseaux non utilisés |

---

### Workflow typique

```bash
# 1. Créer un réseau
docker network create app-network

# 2. Lancer les conteneurs dans le réseau
docker run -d --name db --network app-network mysql:8.0
docker run -d --name web --network app-network nginx

# 3. Les conteneurs peuvent maintenant se parler par leur nom !
# Dans "web", vous pouvez faire : curl http://db:3306
```

---

## ❌ Partie 9 : Les erreurs fréquentes

### Erreur 1 : "Name or service not known"

**Dans votre conteneur :**
```bash
ping ma-db
# ping: ma-db: Name or service not known
```

**Problème :** Les conteneurs ne sont **pas dans le même réseau** !

**Solution :**

**1. Vérifiez les réseaux de vos conteneurs**
```bash
docker inspect mon-app | grep NetworkMode
docker inspect ma-db | grep NetworkMode
```

**2. Si différents, connectez-les au même réseau**
```bash
docker network connect mon-reseau mon-app
docker network connect mon-reseau ma-db
```

---

### Erreur 2 : "network has active endpoints"

**Message :**
```
Error response from daemon: network mon-reseau has active endpoints
```

**Problème :** Vous essayez de supprimer un réseau qui a encore des conteneurs connectés.

**Solution :**

**1. Trouvez les conteneurs connectés**
```bash
docker network inspect mon-reseau | grep Name
```

**2. Arrêtez et supprimez les conteneurs**
```bash
docker rm -f conteneur1 conteneur2
```

**3. Maintenant supprimez le réseau**
```bash
docker network rm mon-reseau
```

---

### Erreur 3 : "Erreur de connexion à la base de données"

**Votre application dit :**
```
SQLSTATE[HY000] [2002] Connection refused
```

**Problèmes possibles :**

**1. Les conteneurs ne sont pas dans le même réseau**
```bash
docker network inspect mon-reseau
```
→ Vérifiez que les 2 conteneurs apparaissent

**2. Le nom du conteneur est faux**

Dans votre code :
```php
$host = 'ma-db';  // ← Le nom DOIT correspondre au --name du conteneur !
```

Vérifiez le nom :
```bash
docker ps
```

**3. La base de données n'est pas encore démarrée**

MySQL/PostgreSQL mettent ~10 secondes à démarrer.

**Vérifiez les logs :**
```bash
docker logs ma-db
```

Attendez de voir : `ready for connections`

---

## ✅ Quiz final

**Question 1 : Pourquoi 2 conteneurs ne peuvent pas se parler par défaut ?**

<details>
<summary>Voir la réponse</summary>

**Réponse :** Docker isole les conteneurs par sécurité.

Il faut **explicitement** les mettre dans le même réseau pour qu'ils puissent communiquer.
</details>

---

**Question 2 : Comment faire communiquer 2 conteneurs ?**
- A) Les lancer sur la même machine
- B) Les mettre dans le même réseau avec `--network`
- C) Utiliser leurs adresses IP

<details>
<summary>Voir la réponse</summary>
✅ **B) Les mettre dans le même réseau avec `--network`**

```bash
docker network create mon-reseau
docker run -d --name c1 --network mon-reseau nginx
docker run -d --name c2 --network mon-reseau nginx
```
</details>

---

**Question 3 : Comment un conteneur PHP trouve-t-il le conteneur MySQL ?**
- A) Par son adresse IP
- B) Par son nom de conteneur (grâce au DNS Docker)
- C) Avec un fichier de configuration

<details>
<summary>Voir la réponse</summary>
✅ **B) Par son nom de conteneur**

```php
$host = 'ma-db';  // Le nom du conteneur !
```

Docker traduit automatiquement `ma-db` en adresse IP.
</details>

---

**Question 4 : Quelle commande pour voir les conteneurs dans un réseau ?**

<details>
<summary>Voir la réponse</summary>

```bash
docker network inspect mon-reseau
```

Cherchez la section `"Containers"` dans le JSON.
</details>

---

## 🎯 Récapitulatif final

### Ce que vous avez appris aujourd'hui :

✅ Pourquoi les conteneurs sont isolés par défaut
✅ C'est quoi un réseau Docker (un "groupe WhatsApp")
✅ Créer un réseau avec `docker network create`
✅ Connecter des conteneurs avec `--network`
✅ La magie du DNS Docker (trouver par nom, pas par IP)
✅ Les 3 types de réseaux (bridge, host, none)
✅ Les commandes pour gérer les réseaux
✅ Débugger les problèmes de communication

### La règle d'or :

**Pour que 2 conteneurs communiquent, mettez-les dans le MÊME réseau !**

```bash
docker network create mon-reseau
docker run -d --name c1 --network mon-reseau image1
docker run -d --name c2 --network mon-reseau image2
```

**Puis utilisez les NOMS dans votre code !**

---

## 🚀 Et maintenant ?

**Dans le prochain cours (Cours 7), vous apprendrez :**

📘 **Les Variables d'Environnement** → Configurer vos conteneurs !

**Vous saurez :**
- C'est quoi une variable d'environnement
- Comment passer des configurations à vos conteneurs avec `-e`
- Gérer des secrets (mots de passe, clés API)
- Utiliser des fichiers `.env`

**Puis dans le Cours 8, vous ferez LE PROJET COMPLET qui combine tout :**
- Ports ✅ (Cours 04)
- Volumes ✅ (Cours 05)
- Réseaux ✅ (Cours 06)
- Variables d'environnement → (Cours 07)

**Application PHP + MariaDB complète !** 🚀

---

**Aide-mémoire ultra-court :**
```bash
docker network create mon-reseau              # Créer un réseau
docker run --network mon-reseau --name c1 ... # Lancer dans le réseau
docker network inspect mon-reseau             # Voir qui est dedans
docker network rm mon-reseau                  # Supprimer
```

**Entraînez-vous bien avec les exercices ! 🐳**
