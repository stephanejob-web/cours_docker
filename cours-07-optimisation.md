# Cours 7 : Optimiser et Sécuriser vos Images Docker 🚀

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Pourquoi l'optimisation est importante
- Réduire la taille de vos images (multi-stage builds)
- Accélérer les builds avec le cache Docker
- Sécuriser vos images
- Scanner les vulnérabilités
- Publier vos images sur Docker Hub

**Durée : 40 minutes (lecture + pratique)**

---

## 📊 Partie 1 : Pourquoi optimiser vos images ?

### Le problème des images lourdes

**Faisons un test simple :**

**Image non optimisée (mauvaise pratique) :**
```bash
mkdir ~/test-taille
cd ~/test-taille

cat > Dockerfile << 'EOF'
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "app.js"]
EOF

# Créez un package.json simple
echo '{"dependencies":{"express":"^4.18.2"}}' > package.json
echo 'console.log("Hello");' > app.js

docker build -t app-lourde .
```

**Vérifiez la taille :**
```bash
docker images app-lourde
```

**Résultat :** Environ **1.1 GB** ! 😱

Pour une application de 2 lignes !

### Les conséquences d'images lourdes

**1. Lenteur** 🐌
- Téléchargement long (plusieurs minutes)
- Déploiement lent
- Mises à jour pénibles

**2. Coûts** 💰
- Plus d'espace disque
- Plus de bande passante
- Plus de ressources serveur

**3. Sécurité** 🔓
- Plus de code = plus de vulnérabilités potentielles
- Plus de surface d'attaque

### La solution : L'optimisation !

**Avec optimisation :**
```dockerfile
FROM node:18-alpine    # alpine au lieu de node
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "app.js"]
```

**Nouvelle taille :** Environ **180 MB** ! 

**Gain :** De 1.1 GB à 180 MB = **Division par 6 !** 🎉

---

## 🏔️ Partie 2 : Utiliser les images Alpine

### C'est quoi Alpine ?

**Alpine Linux = Une distribution Linux ultra-légère**

**Comparaison :**
```
Ubuntu de base : ~70 MB
Debian : ~100 MB
Alpine : ~5 MB  ← 14 fois plus petit !
```

### Exemples de gains

| Image | Taille normale | Taille Alpine | Gain |
|-------|---------------|---------------|------|
| node | 990 MB | 180 MB | 81% |
| python | 920 MB | 50 MB | 95% |
| nginx | 142 MB | 23 MB | 84% |

### Comment utiliser Alpine ?

**Simple : Ajoutez `-alpine` au tag**

**Avant :**
```dockerfile
FROM node:18
FROM python:3.11
FROM nginx:latest
```

**Après :**
```dockerfile
FROM node:18-alpine
FROM python:3.11-alpine
FROM nginx:alpine
```

### Exercice pratique : Comparer les tailles

**Image 1 : Node.js normal**
```bash
mkdir ~/test-node
cd ~/test-node

cat > Dockerfile << 'EOF'
FROM node:18
WORKDIR /app
RUN echo "console.log('Hello');" > app.js
CMD ["node", "app.js"]
EOF

docker build -t node-normal .
```

**Image 2 : Node.js Alpine**
```bash
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
RUN echo "console.log('Hello');" > app.js
CMD ["node", "app.js"]
EOF

docker build -t node-alpine .
```

**Comparer :**
```bash
docker images | grep node-
```

**Résultat :**
```
node-normal    990 MB
node-alpine    180 MB
```

**Gain : 810 MB économisés !** 🎉

### Attention : Quelques différences

**Alpine utilise `apk` au lieu de `apt-get` :**

**Ubuntu/Debian :**
```dockerfile
RUN apt-get update && apt-get install -y curl
```

**Alpine :**
```dockerfile
RUN apk add --no-cache curl
```

**La plupart du temps, Alpine fonctionne sans modification !**

---

## 🏗️ Partie 3 : Multi-Stage Builds (Technique avancée)

### Le problème

**Regardez ce Dockerfile pour compiler une application Go :**

```dockerfile
FROM golang:1.20
WORKDIR /app
COPY . .
RUN go build -o app main.go
CMD ["./app"]
```

**Problème :**
- L'image contient le compilateur Go (plusieurs centaines de MB)
- L'application finale n'a pas besoin du compilateur !
- C'est comme garder tout l'atelier de menuiserie dans votre maison alors que vous avez juste besoin de la chaise !

### La solution : Multi-Stage Build

**Concept :**
1. **Étape 1** : Utiliser une grosse image pour compiler/construire
2. **Étape 2** : Copier seulement le résultat dans une petite image finale

**Schéma :**
```
┌─────────────────┐
│  ÉTAPE 1        │
│  (Grosse image) │
│  - Compiler     │
│  - Construire   │
│  - Tests        │
└────────┬────────┘
         │ Copie seulement le résultat
         ⬇️
┌─────────────────┐
│  ÉTAPE 2        │
│ (Petite image)  │
│ - Juste l'app   │
│ - Rien d'autre  │
└─────────────────┘
```

### Exemple 1 : Application Go

**Sans multi-stage (lourd) :**
```dockerfile
FROM golang:1.20
WORKDIR /app
COPY . .
RUN go build -o app main.go
CMD ["./app"]
# Taille : ~800 MB
```

**Avec multi-stage (léger) :**
```dockerfile
# ÉTAPE 1 : Compilation
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o app main.go

# ÉTAPE 2 : Image finale
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]
# Taille : ~15 MB !
```

**Gain : De 800 MB à 15 MB = Division par 50 !** 🎉

**Explication :**
- `FROM golang:1.20 AS builder` : Première étape nommée "builder"
- `FROM alpine:latest` : Deuxième étape (image finale)
- `COPY --from=builder` : Copie depuis la première étape

### Exemple 2 : Application Node.js

**Projet pratique - Créons-le ensemble !**

**Étape 1 : Créer le projet**
```bash
mkdir ~/app-node-optimisee
cd ~/app-node-optimisee
```

**Étape 2 : Créer package.json**
```bash
cat > package.json << 'EOF'
{
  "name": "app-optimisee",
  "version": "1.0.0",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF
```

**Étape 3 : Créer app.js**
```bash
cat > app.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ message: "App optimisée !" });
});

app.listen(3000, '0.0.0.0', () => {
  console.log('App démarrée sur le port 3000');
});
EOF
```

**Étape 4 : Dockerfile SANS multi-stage**
```bash
cat > Dockerfile.normal << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
EOF
```

**Étape 5 : Dockerfile AVEC multi-stage**
```bash
cat > Dockerfile.optimized << 'EOF'
# ÉTAPE 1 : Installation des dépendances
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm install --production

# ÉTAPE 2 : Image finale (seulement l'essentiel)
FROM node:18-alpine
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY package*.json ./
COPY app.js ./
CMD ["npm", "start"]
EOF
```

**Étape 6 : Construire et comparer**
```bash
# Version normale
docker build -f Dockerfile.normal -t app-normal .

# Version optimisée
docker build -f Dockerfile.optimized -t app-optimized .

# Comparer
docker images | grep app-
```

**Résultats :**
```
app-normal     180 MB
app-optimized  125 MB
```

**Gain : 55 MB économisés (30% de réduction) !**

---

## ⚡ Partie 4 : Optimiser le Cache Docker

### Comment marche le cache Docker ?

**Docker met en cache chaque instruction du Dockerfile.**

Si une instruction n'a pas changé, Docker réutilise le cache au lieu de tout refaire.

### Exemple : Le cache en action

**Dockerfile :**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install        # ← Cette ligne prend 30 secondes
COPY . .
CMD ["npm", "start"]
```

**Premier build :**
```bash
docker build -t mon-app .
# Temps : 35 secondes
```

**Vous modifiez app.js et rebuilder :**
```bash
docker build -t mon-app .
# Temps : 2 secondes !
```

**Pourquoi si rapide ?**
- `package.json` n'a pas changé
- Docker réutilise le cache de `npm install`
- Il recopie juste le nouveau `app.js`

### Mauvais ordre (cache inefficace) ❌

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .                    # ← Tout est copié
RUN npm install            # ← Si vous changez app.js, ça réinstalle TOUT
CMD ["npm", "start"]
```

**Conséquence :**
Chaque modification de code → Réinstallation complète des dépendances (30 secondes perdues)

### Bon ordre (cache efficace) ✅

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./      # ← Seulement les dépendances
RUN npm install            # ← Cache réutilisé si package.json ne change pas
COPY . .                   # ← Code copié en dernier
CMD ["npm", "start"]
```

**Avantage :**
Modification du code → Pas de réinstallation → 2 secondes au lieu de 30 !

### Règle d'or du cache

**📌 Mettez les instructions qui changent rarement EN HAUT**
**📌 Mettez les instructions qui changent souvent EN BAS**

**Ordre du moins fréquent au plus fréquent :**
```dockerfile
FROM node:18-alpine        # Ne change JAMAIS
WORKDIR /app              # Ne change JAMAIS
COPY package.json ./      # Change RAREMENT (nouvelles dépendances)
RUN npm install           # Change RAREMENT
COPY . .                  # Change SOUVENT (votre code)
CMD ["npm", "start"]      # Ne change JAMAIS
```

---

## 🔐 Partie 5 : Sécuriser vos Images

### Risque 1 : Exécuter en root

**Problème :**
Par défaut, les conteneurs s'exécutent en **root** (super-utilisateur).

Si quelqu'un pirate votre conteneur, il a tous les droits ! 😱

**Solution : Créer un utilisateur dédié**

**Mauvais (root) ❌ :**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
# S'exécute en root !
```

**Bon (utilisateur dédié) ✅ :**
```dockerfile
FROM node:18-alpine

# Créer un utilisateur 'appuser'
RUN addgroup -g 1001 -S appuser && \
    adduser -S appuser -u 1001

WORKDIR /app

# Copier et installer (en root, c'est ok)
COPY package*.json ./
RUN npm install

# Copier le code
COPY . .

# Changer le propriétaire
RUN chown -R appuser:appuser /app

# Passer à l'utilisateur non-root
USER appuser

# Maintenant l'app s'exécute en tant que appuser
CMD ["node", "app.js"]
```

**Vérification :**
```bash
docker build -t app-secure .
docker run -d --name test-app app-secure
docker exec test-app whoami
# Résultat : appuser (pas root !)
```

---

### Risque 2 : Secrets dans l'image

**❌ NE JAMAIS FAIRE ÇA :**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
ENV DATABASE_PASSWORD=super_secret_123   # ← DANGEREUX !
CMD ["node", "app.js"]
```

**Pourquoi c'est dangereux ?**
- Le mot de passe est dans l'image
- Tout le monde peut le voir avec `docker inspect`
- Si vous partagez l'image, vous partagez le mot de passe !

**✅ Solution : Variables au runtime**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
# Pas de mot de passe ici !
CMD ["node", "app.js"]
```

**Passer les secrets au lancement :**
```bash
docker run -e DATABASE_PASSWORD=secret123 mon-app
```

Ou avec un fichier `.env` (on verra ça avec Docker Compose).

---

### Risque 3 : Images avec des vulnérabilités

**Le problème :**
Les images peuvent contenir des bugs de sécurité.

**Solution : Scanner vos images**

**Avec Docker Scout (intégré à Docker) :**
```bash
docker scout cves mon-app
```

**Avec Trivy (outil populaire) :**
```bash
# Installer Trivy
docker run aquasec/trivy image mon-app
```

**Résultat :**
```
Total: 15 vulnerabilities
- CRITICAL: 2
- HIGH: 5
- MEDIUM: 8
```

**Actions à prendre :**
1. Mettre à jour les images de base
2. Mettre à jour les dépendances
3. Utiliser des images spécifiques (pas `latest`)

---

### Risque 4 : Images trop grosses

**Plus une image est grosse, plus elle contient de code, plus il y a de risques de vulnérabilités.**

**Solution : On l'a déjà vue !**
- Utiliser Alpine
- Multi-stage builds
- Supprimer les fichiers temporaires

---

## 📦 Partie 6 : Publier sur Docker Hub

### C'est quoi Docker Hub ?

**Docker Hub = Le "GitHub" des images Docker**

Vous pouvez :
- Partager vos images publiquement
- Avoir des images privées
- Télécharger des millions d'images existantes

### Étape 1 : Créer un compte

1. Allez sur https://hub.docker.com
2. Cliquez sur "Sign Up"
3. Créez votre compte (gratuit)

Notez votre **nom d'utilisateur** (exemple: "johndev")

### Étape 2 : Se connecter depuis le terminal

```bash
docker login
```

**Entrez :**
- Username : votre nom d'utilisateur
- Password : votre mot de passe

**Résultat :**
```
Login Succeeded
```

✅ **Vous êtes connecté !**

### Étape 3 : Taguer votre image

**Format obligatoire :**
```
votre-username/nom-image:version
```

**Exemple :**
```bash
docker tag mon-app johndev/mon-app:1.0.0
```

**Ou directement lors du build :**
```bash
docker build -t johndev/mon-app:1.0.0 .
```

**Conventions des tags :**
- `1.0.0` : Version spécifique
- `latest` : Dernière version
- `dev` : Version de développement

**Plusieurs tags pour la même image :**
```bash
docker tag mon-app johndev/mon-app:1.0.0
docker tag mon-app johndev/mon-app:latest
```

### Étape 4 : Pousser l'image (upload)

```bash
docker push johndev/mon-app:1.0.0
```

**Ce qui se passe :**
```
The push refers to repository [docker.io/johndev/mon-app]
abc123: Pushed
def456: Pushed
1.0.0: digest: sha256:abc... size: 1234
```

⏱️ **Temps de push :** 10-60 secondes selon la taille

### Étape 5 : Vérifier sur Docker Hub

1. Allez sur https://hub.docker.com
2. Connectez-vous
3. Cliquez sur "Repositories"
4. Vous voyez `mon-app` ! ✅

### Étape 6 : N'importe qui peut maintenant utiliser votre image !

**Sur n'importe quel ordinateur avec Docker :**
```bash
docker pull johndev/mon-app:1.0.0
docker run -d -p 3000:3000 johndev/mon-app:1.0.0
```

🎉 **Votre image est publique et accessible partout !**

### Images privées

**Par défaut, les images sont publiques.**

**Pour rendre une image privée :**
1. Sur Docker Hub, allez dans votre repo
2. Settings → Make Private
3. Seul vous (et les personnes autorisées) peuvent la télécharger

**Note :** Le compte gratuit permet 1 repo privé.

---

## 💼 Partie 7 : Projet Complet - Image Optimisée

### Mission : Créer une image ultra-optimisée

**On va créer une API Node.js avec TOUTES les optimisations !**

**Étape 1 : Créer le projet**
```bash
mkdir ~/api-ultra-optimisee
cd ~/api-ultra-optimisee
```

**Étape 2 : Créer package.json**
```bash
cat > package.json << 'EOF'
{
  "name": "api-optimisee",
  "version": "2.0.0",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF
```

**Étape 3 : Créer app.js**
```bash
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({
    message: "API Ultra-Optimisée",
    version: "2.0.0",
    optimizations: [
      "Alpine Linux",
      "Multi-stage build",
      "Cache optimisé",
      "Utilisateur non-root",
      "Taille minimale"
    ]
  });
});

app.get('/health', (req, res) => {
  res.json({ status: "OK", uptime: process.uptime() });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`✅ API démarrée sur le port ${PORT}`);
});
EOF
```

**Étape 4 : Créer .dockerignore**
```bash
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
Dockerfile*
.dockerignore
EOF
```

**Étape 5 : Dockerfile OPTIMAL**
```bash
cat > Dockerfile << 'EOF'
# =================================
# ÉTAPE 1 : Installation dépendances
# =================================
FROM node:18-alpine AS dependencies

WORKDIR /app

# Copier seulement package.json (cache optimal)
COPY package*.json ./

# Installer seulement les dépendances de production
RUN npm ci --only=production && \
    npm cache clean --force

# =================================
# ÉTAPE 2 : Image finale
# =================================
FROM node:18-alpine

# Créer un utilisateur non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copier les dépendances depuis l'étape 1
COPY --from=dependencies /app/node_modules ./node_modules

# Copier les fichiers de l'application
COPY --chown=nodejs:nodejs package*.json ./
COPY --chown=nodejs:nodejs app.js ./

# Passer à l'utilisateur non-root
USER nodejs

# Exposer le port
EXPOSE 3000

# Santé du conteneur
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Démarrer l'application
CMD ["node", "app.js"]
EOF
```

**Étape 6 : Construire**
```bash
docker build -t api-optimized:2.0.0 .
```

**Étape 7 : Vérifier la taille**
```bash
docker images api-optimized
```

**Résultat : Environ 120-130 MB !** 🎉

Pour une API complète avec Express, c'est excellent !

**Étape 8 : Tester**
```bash
docker run -d -p 3000:3000 --name api-test api-optimized:2.0.0
```

**Tester les routes :**
```
http://localhost:3000
http://localhost:3000/health
```

**Étape 9 : Vérifier la sécurité**
```bash
docker exec api-test whoami
# Résultat : nodejs (pas root !)
```

✅ **Image optimale créée !**

**Récap des optimisations :**
- ✅ Alpine Linux (léger)
- ✅ Multi-stage build (seulement le nécessaire)
- ✅ Cache optimisé (builds rapides)
- ✅ Utilisateur non-root (sécurité)
- ✅ .dockerignore (pas de fichiers inutiles)
- ✅ npm ci au lieu de npm install (plus rapide et déterministe)
- ✅ HEALTHCHECK (monitoring)

---

## 📝 Partie 8 : Checklist d'optimisation

### ✅ Avant de partager votre image

**1. Taille**
- [ ] Utiliser Alpine si possible
- [ ] Multi-stage build pour les apps compilées
- [ ] Nettoyer les fichiers temporaires
- [ ] .dockerignore configuré

**2. Sécurité**
- [ ] Utilisateur non-root défini
- [ ] Pas de secrets dans l'image
- [ ] Image scannée (Trivy, Docker Scout)
- [ ] Version spécifique (pas `latest`)

**3. Performance**
- [ ] Cache optimisé (ordre des instructions)
- [ ] Une seule instruction RUN pour les installations
- [ ] Dépendances installées avant le code

**4. Documentation**
- [ ] README.md avec instructions
- [ ] LABELS dans le Dockerfile
- [ ] EXPOSE pour documenter les ports
- [ ] HEALTHCHECK si applicable

---

## ✅ Quiz de validation

**Question 1 : Pourquoi utiliser Alpine ?**

- A) C'est plus joli
- B) C'est obligatoire
- C) C'est beaucoup plus léger (5 MB vs 100 MB)
- D) C'est plus rapide

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : C'est beaucoup plus léger**

Alpine Linux fait seulement ~5 MB contre ~100 MB pour Debian/Ubuntu. Ça réduit drastiquement la taille des images finales.

</details>

---

**Question 2 : C'est quoi un multi-stage build ?**

- A) Construire plusieurs images en même temps
- B) Utiliser une grosse image pour construire, puis copier le résultat dans une petite image
- C) Construire l'image en plusieurs étapes séparées
- D) Une technique pour accélérer Docker

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B**

Multi-stage = utiliser une image avec tous les outils pour compiler/construire, puis copier seulement le résultat final dans une petite image. Ça réduit énormément la taille !

</details>

---

**Question 3 : Quel ordre pour optimiser le cache ?**

- A) COPY tout d'abord, puis RUN npm install
- B) RUN npm install d'abord, puis COPY package.json
- C) COPY package.json, RUN npm install, puis COPY le reste
- D) L'ordre n'a pas d'importance

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C**

L'ordre optimal : copier package.json → installer → copier le code. Comme ça, si le code change mais pas package.json, npm install est mis en cache !

</details>

---

**Question 4 : Pourquoi ne pas exécuter en root ?**

- A) C'est interdit par Docker
- B) C'est plus lent
- C) Sécurité : si le conteneur est piraté, l'attaquant n'a pas tous les droits
- D) Ça prend plus de place

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : Sécurité**

Si quelqu'un pirate un conteneur qui tourne en root, il a tous les droits. Avec un utilisateur dédié, les dégâts sont limités.

</details>

---

**Question 5 : Comment pousser une image sur Docker Hub ?**

- A) `docker upload mon-image`
- B) `docker send mon-image`
- C) `docker push username/mon-image`
- D) `docker publish mon-image`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : `docker push username/mon-image`**

Il faut d'abord taguer l'image avec votre username Docker Hub, puis faire `docker push`.

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Les 5 piliers de l'optimisation

**1. Taille réduite**
```dockerfile
FROM node:18-alpine    # Alpine au lieu de node:18
```

**2. Multi-stage builds**
```dockerfile
FROM node:18 AS builder
# ... compilation
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
```

**3. Cache optimisé**
```dockerfile
COPY package.json ./   # En premier
RUN npm install        # Mis en cache
COPY . .               # En dernier
```

**4. Sécurité**
```dockerfile
RUN adduser -S appuser
USER appuser           # Pas root
```

**5. Nettoyage**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Commandes essentielles

```bash
# Construire
docker build -t mon-app .

# Voir la taille
docker images mon-app

# Scanner les vulnérabilités
docker scout cves mon-app

# Publier sur Docker Hub
docker tag mon-app username/mon-app:1.0.0
docker push username/mon-app:1.0.0
```

---

## 🚀 Et maintenant ?

**EXCELLENT TRAVAIL ! Vos images sont maintenant optimales !** 🎉

### Ce que vous maîtrisez maintenant :

- ✅ Réduire la taille des images drastiquement
- ✅ Multi-stage builds pour images ultra-légères
- ✅ Optimiser le cache pour des builds rapides
- ✅ Sécuriser vos images (non-root, scan vulnérabilités)
- ✅ Publier sur Docker Hub

### Dans le prochain cours (Cours 8) :

**DOCKER COMPOSE - Orchestrer plusieurs conteneurs !** 🎼

Vous apprendrez :
1. C'est quoi Docker Compose
2. Écrire un fichier docker-compose.yml
3. Lancer une app complète (frontend + backend + base de données) en une commande
4. Gérer plusieurs services facilement
5. Projet complet : Application web full-stack

**C'est LE cours qui va tout simplifier !** 💪

---

## 📚 Template Dockerfile Optimal (à réutiliser)

**Pour Node.js :**
```dockerfile
# Build stage
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Final stage
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .
USER nodejs
EXPOSE 3000
CMD ["node", "app.js"]
```

**Pour Python :**
```dockerfile
FROM python:3.11-alpine
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -s /bin/sh -D appuser
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=appuser:appgroup . .
USER appuser
EXPOSE 5000
CMD ["python", "app.py"]
```

---

**🎓 Vous êtes prêt pour le Cours 8 (DOCKER COMPOSE) !**

Bravo ! Vos images sont maintenant professionnelles, optimisées et sécurisées. Le prochain cours va vous montrer comment orchestrer facilement plusieurs conteneurs ensemble. C'est génial ! 🚀💪
