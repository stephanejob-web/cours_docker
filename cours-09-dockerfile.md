# Cours 6 : Créer vos Propres Images avec Dockerfile 🏗️

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- C'est quoi un Dockerfile
- Écrire votre premier Dockerfile
- Les 10 instructions essentielles (FROM, RUN, COPY, CMD...)
- Construire une image à partir d'un Dockerfile
- Créer une vraie application Node.js dans Docker
- Les bonnes pratiques pour des images efficaces

**Durée : 45 minutes (lecture + pratique)**

---

## 📖 Partie 1 : C'est quoi un Dockerfile ?

### Le problème jusqu'à maintenant

Jusqu'ici, vous avez utilisé des images **déjà faites** :
- ubuntu
- nginx
- mysql
- wordpress

**Mais comment créer VOTRE propre image ?**

Par exemple :
- Votre application Node.js
- Votre site Python Django
- Votre API en Go
- N'importe quoi !

### La solution : Le Dockerfile

**Un Dockerfile = Une recette pour créer une image**

**Analogie :**
- **Sans Dockerfile** : Acheter un gâteau tout fait au supermarché
- **Avec Dockerfile** : Écrire la recette et faire votre propre gâteau

**Avantages :**
- ✅ Personnalisation totale
- ✅ Reproductible (même résultat à chaque fois)
- ✅ Automatique (pas besoin de tout refaire manuellement)
- ✅ Partageable (donnez la recette à vos collègues)

### C'est quoi concrètement ?

**Un Dockerfile = un simple fichier texte qui contient des instructions**

**Exemple super simple :**
```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y nginx
CMD ["nginx", "-g", "daemon off;"]
```

**Traduction en français :**
1. Prends l'image Ubuntu comme base
2. Mets à jour les paquets
3. Installe nginx
4. Démarre nginx quand le conteneur se lance

**4 lignes = une image nginx personnalisée !** 🎉

### Le processus complet

```
1. Vous écrivez un Dockerfile
         ⬇️
2. Vous "construisez" l'image avec docker build
         ⬇️
3. Docker crée l'image automatiquement
         ⬇️
4. Vous pouvez créer des conteneurs avec cette image
```

---

## 📝 Partie 2 : Votre Premier Dockerfile

### Projet simple : Une page web HTML

**Mission : Créer une image qui sert une page HTML personnalisée**

**Étape 1 : Créer un dossier pour le projet**
```bash
mkdir ~/mon-premier-dockerfile
cd ~/mon-premier-dockerfile
```

**Étape 2 : Créer la page HTML**
```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Ma Première Image Docker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 50px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        h1 { font-size: 48px; }
        p { font-size: 24px; }
    </style>
</head>
<body>
    <h1>🎉 Bravo !</h1>
    <p>Vous avez créé votre première image Docker !</p>
    <p>Cette page vient de VOTRE image personnalisée</p>
</body>
</html>
EOF
```

**Étape 3 : Créer le Dockerfile**

⚠️ **IMPORTANT : Le fichier DOIT s'appeler exactement `Dockerfile` (avec un D majuscule, pas d'extension)**

```bash
cat > Dockerfile << 'EOF'
# Partir de l'image nginx officielle
FROM nginx:alpine

# Copier notre page HTML dans le conteneur
COPY index.html /usr/share/nginx/html/index.html

# Nginx démarre automatiquement avec cette image
EOF
```

**Explication ligne par ligne :**

1. `FROM nginx:alpine` 
   - On part de l'image nginx (version alpine = légère)
   - C'est l'image de BASE

2. `COPY index.html /usr/share/nginx/html/index.html`
   - On copie notre fichier `index.html` (sur notre PC)
   - Vers le chemin `/usr/share/nginx/html/index.html` (dans l'image)

3. Le `# Nginx démarre...` est un commentaire (explications)

**Étape 4 : Vérifier que vous avez bien 2 fichiers**
```bash
ls
```

Vous devez voir :
```
Dockerfile
index.html
```

✅ **Si vous avez ces 2 fichiers, continuez !**

**Étape 5 : Construire l'image**

**Commande magique :**
```bash
docker build -t mon-site-web .
```

**Explication :**
- `docker build` : Commande pour construire une image
- `-t mon-site-web` : Donne le nom "mon-site-web" à l'image
- `.` : Le point = cherche le Dockerfile dans le dossier actuel

**Ce que vous verrez :**
```
[+] Building 2.5s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:alpine
 => [1/2] FROM docker.io/library/nginx:alpine
 => [internal] load build context
 => [2/2] COPY index.html /usr/share/nginx/html/index.html
 => exporting to image
 => => naming to docker.io/library/mon-site-web
```

🎉 **Votre image est créée !**

**Étape 6 : Vérifier que l'image existe**
```bash
docker images
```

Vous devez voir `mon-site-web` dans la liste ! ✅

**Étape 7 : Lancer un conteneur avec VOTRE image**
```bash
docker run -d -p 8080:80 --name mon-super-site mon-site-web
```

**Étape 8 : Tester dans le navigateur**

Ouvrez : `http://localhost:8080`

🎊 **Vous voyez votre page personnalisée !**

**BRAVO ! Vous venez de créer votre première image Docker !** 🎉

---

## 🔧 Partie 3 : Les Instructions Essentielles du Dockerfile

### Instruction 1 : FROM (obligatoire)

**Définir l'image de base**

**Syntaxe :**
```dockerfile
FROM image:tag
```

**Exemples :**
```dockerfile
FROM ubuntu:22.04        # Ubuntu version 22.04
FROM node:18             # Node.js version 18
FROM python:3.11         # Python 3.11
FROM nginx:alpine        # Nginx version alpine (légère)
FROM scratch             # Image vide (rare)
```

⚠️ **FROM doit TOUJOURS être la première instruction !**

---

### Instruction 2 : COPY

**Copier des fichiers de votre PC vers l'image**

**Syntaxe :**
```dockerfile
COPY source destination
```

**Exemples :**
```dockerfile
# Copier un fichier
COPY app.js /app/app.js

# Copier un dossier complet
COPY ./src /app/src

# Copier plusieurs fichiers
COPY package.json package-lock.json /app/

# Copier tout le dossier actuel
COPY . /app
```

**Différence avec ADD :**
- `COPY` : Simple copie de fichiers (recommandé)
- `ADD` : Comme COPY mais peut télécharger des URLs et extraire des archives (évitez-le)

**Règle : Utilisez toujours COPY, pas ADD**

---

### Instruction 3 : RUN

**Exécuter des commandes pendant la CONSTRUCTION de l'image**

**Syntaxe :**
```dockerfile
RUN commande
```

**Exemples :**
```dockerfile
# Installer des paquets sur Ubuntu
RUN apt-get update && apt-get install -y curl

# Installer des dépendances Node.js
RUN npm install

# Installer des dépendances Python
RUN pip install -r requirements.txt

# Créer un dossier
RUN mkdir -p /app/data

# Plusieurs commandes en une ligne
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean
```

**Important :** 
- `RUN` s'exécute PENDANT la construction
- Pas quand le conteneur démarre !

---

### Instruction 4 : CMD

**Commande à exécuter quand le conteneur DÉMARRE**

**Syntaxe (recommandée) :**
```dockerfile
CMD ["executable", "param1", "param2"]
```

**Exemples :**
```dockerfile
# Démarrer nginx
CMD ["nginx", "-g", "daemon off;"]

# Démarrer une app Node.js
CMD ["node", "app.js"]

# Démarrer Python
CMD ["python", "app.py"]

# Démarrer un script bash
CMD ["./start.sh"]
```

**Important :**
- Il ne peut y avoir qu'UN SEUL `CMD` dans un Dockerfile
- S'il y en a plusieurs, seul le dernier compte
- `CMD` peut être écrasé quand vous faites `docker run`

---

### Instruction 5 : WORKDIR

**Définir le dossier de travail dans le conteneur**

**Syntaxe :**
```dockerfile
WORKDIR /chemin
```

**Exemples :**
```dockerfile
WORKDIR /app

# Maintenant toutes les commandes se passent dans /app
COPY . .              # Copie dans /app
RUN npm install       # Exécute dans /app
CMD ["node", "app.js"] # Démarre depuis /app
```

**Pourquoi c'est important ?**
- Sans `WORKDIR`, vous êtes à la racine `/`
- Avec `WORKDIR`, vous définissez où vous travaillez
- Plus propre et organisé

---

### Instruction 6 : ENV

**Définir des variables d'environnement**

**Syntaxe :**
```dockerfile
ENV NOM_VARIABLE=valeur
```

**Exemples :**
```dockerfile
# Définir le port
ENV PORT=3000

# Définir l'environnement
ENV NODE_ENV=production

# Plusieurs variables
ENV DATABASE_HOST=localhost \
    DATABASE_PORT=5432 \
    DATABASE_NAME=myapp
```

**Utilisation dans le code :**
```javascript
// Dans votre app Node.js
const port = process.env.PORT || 3000;
```

---

### Instruction 7 : EXPOSE

**Indiquer quel port l'application utilise**

**Syntaxe :**
```dockerfile
EXPOSE port
```

**Exemples :**
```dockerfile
EXPOSE 80        # HTTP
EXPOSE 443       # HTTPS
EXPOSE 3000      # Node.js app
EXPOSE 5432      # PostgreSQL
```

⚠️ **Important :** `EXPOSE` est juste une **documentation** !
- Ça n'ouvre pas vraiment le port
- Vous devez quand même utiliser `-p` avec `docker run`
- C'est pour indiquer aux autres développeurs quel port utiliser

---

### Instruction 8 : USER

**Définir l'utilisateur qui exécute les commandes**

**Syntaxe :**
```dockerfile
USER nom_utilisateur
```

**Exemple :**
```dockerfile
# Créer un utilisateur non-root
RUN useradd -m -s /bin/bash appuser

# Passer à cet utilisateur
USER appuser

# Maintenant toutes les commandes s'exécutent en tant que appuser
CMD ["node", "app.js"]
```

**Pourquoi ?**
- Sécurité : Ne jamais exécuter en tant que root dans un conteneur
- Bonne pratique en production

---

### Instruction 9 : VOLUME

**Définir un point de montage pour un volume**

**Syntaxe :**
```dockerfile
VOLUME /chemin
```

**Exemple :**
```dockerfile
VOLUME /app/data
```

**Note :** On utilise rarement `VOLUME` dans un Dockerfile. Il vaut mieux définir les volumes avec `docker run -v`.

---

### Instruction 10 : ENTRYPOINT

**Définir la commande principale (non modifiable)**

**Différence avec CMD :**
- `CMD` : Peut être écrasée avec `docker run`
- `ENTRYPOINT` : Commande fixe, on peut seulement ajouter des arguments

**Exemple :**
```dockerfile
ENTRYPOINT ["python", "app.py"]

# Maintenant on peut faire :
# docker run mon-image --debug
# Ça exécutera : python app.py --debug
```

**On utilise rarement ENTRYPOINT au début. Concentrez-vous sur CMD.**

---

## 💻 Partie 4 : Projet Complet - API Node.js

### Mission : Créer une API REST avec Docker

**On va créer :**
1. Une API Node.js simple
2. Un Dockerfile pour l'empaqueter
3. Construire l'image
4. Lancer et tester

### Étape par étape

**Étape 1 : Créer le dossier du projet**
```bash
mkdir ~/api-nodejs-docker
cd ~/api-nodejs-docker
```

---

**Étape 2 : Créer package.json**
```bash
cat > package.json << 'EOF'
{
  "name": "api-docker",
  "version": "1.0.0",
  "description": "API simple avec Docker",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF
```

---

**Étape 3 : Créer l'application app.js**
```bash
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Route principale
app.get('/', (req, res) => {
  res.json({
    message: "Bienvenue sur mon API Docker !",
    version: "1.0.0",
    status: "running"
  });
});

// Route /health pour vérifier que l'API fonctionne
app.get('/health', (req, res) => {
  res.json({ status: "OK", timestamp: new Date() });
});

// Route /users (exemple)
app.get('/users', (req, res) => {
  res.json([
    { id: 1, name: "Alice", role: "Admin" },
    { id: 2, name: "Bob", role: "User" },
    { id: 3, name: "Charlie", role: "User" }
  ]);
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`🚀 API démarrée sur le port ${PORT}`);
  console.log(`📡 http://localhost:${PORT}`);
});
EOF
```

---

**Étape 4 : Créer le Dockerfile**
```bash
cat > Dockerfile << 'EOF'
# Étape 1 : Partir d'une image Node.js
FROM node:18-alpine

# Étape 2 : Définir le dossier de travail
WORKDIR /app

# Étape 3 : Copier les fichiers package
COPY package*.json ./

# Étape 4 : Installer les dépendances
RUN npm install

# Étape 5 : Copier le code source
COPY . .

# Étape 6 : Exposer le port (documentation)
EXPOSE 3000

# Étape 7 : Définir la commande de démarrage
CMD ["npm", "start"]
EOF
```

**Explication détaillée :**

1. `FROM node:18-alpine`
   - Image de base : Node.js version 18
   - Alpine = version ultra-légère de Linux (5 Mo au lieu de 50 Mo !)

2. `WORKDIR /app`
   - On travaille dans le dossier `/app`
   - Tout ce qui suit se passe là

3. `COPY package*.json ./`
   - On copie `package.json` et `package-lock.json`
   - Pourquoi en premier ? Pour optimiser le cache (on verra ça plus tard)

4. `RUN npm install`
   - Installe toutes les dépendances (express, etc.)

5. `COPY . .`
   - Copie tout le reste du code

6. `EXPOSE 3000`
   - Documentation : l'API écoute sur le port 3000

7. `CMD ["npm", "start"]`
   - Démarre l'API avec `npm start`

---

**Étape 5 : Vérifier les fichiers**
```bash
ls
```

Vous devez avoir :
```
Dockerfile
package.json
app.js
```

✅ **Si c'est bon, continuez !**

---

**Étape 6 : Construire l'image**
```bash
docker build -t mon-api .
```

**Ce que vous verrez :**
```
[+] Building 15.3s (10/10) FINISHED
 => [1/5] FROM docker.io/library/node:18-alpine
 => [2/5] WORKDIR /app
 => [3/5] COPY package*.json ./
 => [4/5] RUN npm install
 => [5/5] COPY . .
 => exporting to image
Successfully built abc123def456
Successfully tagged mon-api:latest
```

⏱️ **Ça prend 15-30 secondes la première fois** (téléchargement de Node.js + installation d'express)

---

**Étape 7 : Vérifier l'image**
```bash
docker images mon-api
```

Vous voyez `mon-api` avec une taille d'environ 180 Mo.

---

**Étape 8 : Lancer le conteneur**
```bash
docker run -d -p 3000:3000 --name api-container mon-api
```

---

**Étape 9 : Vérifier les logs**
```bash
docker logs api-container
```

Vous devez voir :
```
🚀 API démarrée sur le port 3000
📡 http://localhost:3000
```

✅ **L'API fonctionne !**

---

**Étape 10 : Tester l'API**

**Dans le navigateur :**
```
http://localhost:3000
```

Vous voyez :
```json
{
  "message": "Bienvenue sur mon API Docker !",
  "version": "1.0.0",
  "status": "running"
}
```

**Tester les autres routes :**
```
http://localhost:3000/health
http://localhost:3000/users
```

🎉 **Tout fonctionne !**

---

**Étape 11 : Tester avec curl (dans le terminal)**
```bash
curl http://localhost:3000
curl http://localhost:3000/health
curl http://localhost:3000/users
```

---

**Étape 12 : Modifier le code et reconstruire**

Modifiez `app.js` pour ajouter une nouvelle route :
```bash
cat >> app.js << 'EOF'

// Nouvelle route
app.get('/info', (req, res) => {
  res.json({
    hostname: require('os').hostname(),
    platform: process.platform,
    nodeVersion: process.version,
    uptime: process.uptime()
  });
});
EOF
```

**Reconstruire l'image :**
```bash
docker build -t mon-api .
```

**Arrêter l'ancien conteneur et lancer le nouveau :**
```bash
docker stop api-container
docker rm api-container
docker run -d -p 3000:3000 --name api-container mon-api
```

**Tester la nouvelle route :**
```
http://localhost:3000/info
```

✅ **Vous voyez les nouvelles infos !**

---

## 📊 Partie 5 : Les Bonnes Pratiques

### ✅ Bonne pratique 1 : Utiliser des images Alpine

**Pourquoi ?**
- Plus petites (5-50 Mo au lieu de 100-500 Mo)
- Plus rapides à télécharger
- Plus sécurisées (moins de code = moins de failles)

**Exemples :**
```dockerfile
FROM node:18-alpine      # Au lieu de node:18
FROM python:3.11-alpine  # Au lieu de python:3.11
FROM nginx:alpine        # Au lieu de nginx
```

---

### ✅ Bonne pratique 2 : Ordre des instructions

**Mauvais ordre (lent) ❌ :**
```dockerfile
FROM node:18-alpine
COPY . .                    # Copie TOUT
RUN npm install            # Si vous changez le code, ça réinstalle TOUT
CMD ["npm", "start"]
```

**Bon ordre (rapide) ✅ :**
```dockerfile
FROM node:18-alpine
COPY package*.json ./      # Copie juste les dépendances
RUN npm install            # Cache cette étape si package.json ne change pas
COPY . .                   # Copie le code (change souvent)
CMD ["npm", "start"]
```

**Pourquoi ?**
Docker utilise un **cache**. Si une instruction ne change pas, il réutilise le cache au lieu de tout refaire.

---

### ✅ Bonne pratique 3 : Un seul RUN pour plusieurs commandes

**Mauvais (crée plusieurs couches) ❌ :**
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean
```

**Bon (une seule couche) ✅ :**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Avantage :** Image plus petite et plus rapide à construire.

---

### ✅ Bonne pratique 4 : Utiliser .dockerignore

**C'est quoi ?**
Comme `.gitignore`, mais pour Docker. Ça évite de copier des fichiers inutiles dans l'image.

**Créer un .dockerignore :**
```bash
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
Dockerfile
.dockerignore
EOF
```

**Pourquoi ?**
- `node_modules` : On va les réinstaller avec `npm install`, pas besoin de les copier
- `.git` : Inutile dans l'image
- `.env` : Fichiers secrets ne doivent jamais être dans l'image

---

### ✅ Bonne pratique 5 : Ne pas exécuter en root

**Mauvais (root = dangereux) ❌ :**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
# Par défaut, ça s'exécute en root !
```

**Bon (utilisateur dédié) ✅ :**
```dockerfile
FROM node:18-alpine

# Créer un utilisateur
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copier et installer en root
COPY package*.json ./
RUN npm install

# Copier le code
COPY . .

# Changer les permissions
RUN chown -R nodejs:nodejs /app

# Passer à l'utilisateur non-root
USER nodejs

CMD ["node", "app.js"]
```

---

### ✅ Bonne pratique 6 : Versions spécifiques

**Mauvais (version change) ❌ :**
```dockerfile
FROM node:latest     # Quelle version ? On ne sait pas !
```

**Bon (version fixée) ✅ :**
```dockerfile
FROM node:18-alpine  # Toujours la version 18
```

**Pourquoi ?**
- `latest` peut changer et casser votre application
- Versions spécifiques = reproductible

---

### ✅ Bonne pratique 7 : Labels pour la documentation

```dockerfile
LABEL maintainer="votre@email.com"
LABEL version="1.0.0"
LABEL description="API REST avec Node.js"
```

---

## 📝 Partie 6 : Exercice Complet

### Exercice : Créer une application Python Flask

**Mission : Créer une API Python avec Docker**

**Étape 1 : Créer le dossier**
```bash
mkdir ~/api-python-docker
cd ~/api-python-docker
```

**Étape 2 : Créer requirements.txt**
```bash
cat > requirements.txt << 'EOF'
Flask==2.3.0
EOF
```

**Étape 3 : Créer app.py**
```bash
cat > app.py << 'EOF'
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        "message": "API Python avec Docker",
        "status": "running"
    })

@app.route('/hello/<name>')
def hello(name):
    return jsonify({
        "message": f"Bonjour {name} !",
        "from": "Docker"
    })

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port)
EOF
```

**Étape 4 : Créer le Dockerfile**
```bash
cat > Dockerfile << 'EOF'
# Image de base
FROM python:3.11-alpine

# Dossier de travail
WORKDIR /app

# Copier les dépendances
COPY requirements.txt .

# Installer les dépendances
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code
COPY . .

# Exposer le port
EXPOSE 5000

# Commande de démarrage
CMD ["python", "app.py"]
EOF
```

**Étape 5 : Construire**
```bash
docker build -t api-python .
```

**Étape 6 : Lancer**
```bash
docker run -d -p 5000:5000 --name python-api api-python
```

**Étape 7 : Tester**
```
http://localhost:5000
http://localhost:5000/hello/Alice
```

✅ **Si ça marche, BRAVO !**

**Nettoyage :**
```bash
docker stop python-api
docker rm python-api
```

---

## ✅ Quiz de validation

**Question 1 : Quelle instruction DOIT être en premier dans un Dockerfile ?**

- A) RUN
- B) COPY
- C) FROM
- D) CMD

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : FROM**

`FROM` définit l'image de base et doit TOUJOURS être la première instruction.

</details>

---

**Question 2 : Quelle différence entre RUN et CMD ?**

- A) Aucune différence
- B) RUN s'exécute pendant la construction, CMD au démarrage du conteneur
- C) CMD s'exécute pendant la construction, RUN au démarrage
- D) RUN est plus rapide

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B**

- `RUN` : Exécuté PENDANT la construction de l'image (installer des paquets, etc.)
- `CMD` : Exécuté quand le conteneur DÉMARRE

</details>

---

**Question 3 : Commande pour construire une image nommée "mon-app" ?**

- A) `docker create -t mon-app .`
- B) `docker build -t mon-app .`
- C) `docker make mon-app`
- D) `docker image mon-app .`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : `docker build -t mon-app .`**

- `docker build` : Construire une image
- `-t mon-app` : Tag (nom) de l'image
- `.` : Chercher le Dockerfile dans le dossier actuel

</details>

---

**Question 4 : À quoi sert WORKDIR ?**

- A) Créer un nouveau dossier
- B) Définir où on travaille dans le conteneur
- C) Copier des fichiers
- D) Installer des programmes

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Définir où on travaille dans le conteneur**

`WORKDIR /app` fait que toutes les commandes suivantes s'exécutent dans `/app`.

</details>

---

**Question 5 : Pourquoi utiliser des images Alpine ?**

- A) Elles sont plus belles
- B) Elles sont plus petites et rapides
- C) Elles sont obligatoires
- D) Elles contiennent plus de logiciels

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Elles sont plus petites et rapides**

Alpine = version ultra-légère de Linux. node:18-alpine fait ~50 Mo au lieu de ~300 Mo !

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Le Dockerfile en 3 étapes

**1. Image de base**
```dockerfile
FROM node:18-alpine
```

**2. Préparer l'environnement**
```dockerfile
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
```

**3. Définir le démarrage**
```dockerfile
EXPOSE 3000
CMD ["npm", "start"]
```

### Les instructions essentielles

| Instruction | Usage | Exemple |
|------------|-------|---------|
| `FROM` | Image de base | `FROM node:18-alpine` |
| `WORKDIR` | Dossier de travail | `WORKDIR /app` |
| `COPY` | Copier des fichiers | `COPY . .` |
| `RUN` | Exécuter pendant construction | `RUN npm install` |
| `CMD` | Commande au démarrage | `CMD ["node", "app.js"]` |
| `EXPOSE` | Documenter le port | `EXPOSE 3000` |
| `ENV` | Variables d'environnement | `ENV PORT=3000` |

### Construire et lancer

```bash
# Construire
docker build -t nom-image .

# Lancer
docker run -d -p 3000:3000 nom-image
```

---

## 🚀 Et maintenant ?

**ÉNORME BRAVO ! Vous savez créer vos propres images Docker !** 🎉

### Ce que vous maîtrisez maintenant :

- ✅ Écrire un Dockerfile
- ✅ Les 10 instructions essentielles
- ✅ Construire des images
- ✅ Créer des API Node.js et Python dans Docker
- ✅ Les bonnes pratiques pour des images optimisées

### Dans le prochain cours (Cours 7) :

**Optimiser et sécuriser vos images** 🔒

Vous apprendrez :
1. Réduire la taille des images (multi-stage builds)
2. Accélérer les builds avec le cache
3. Sécuriser vos images
4. Scanner les vulnérabilités
5. Publier sur Docker Hub

---

## 📚 Template de Dockerfile (à réutiliser)

**Pour Node.js :**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

**Pour Python :**
```dockerfile
FROM python:3.11-alpine
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

**Pour un site statique :**
```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

---

**🎓 Vous êtes prêt pour le Cours 7 (Optimisation) !**

Excellent travail ! Vous venez de franchir l'étape la plus importante de Docker : créer vos propres images ! La suite va vous montrer comment les rendre encore meilleures. 💪🚀
