# Cours 8 : Docker Compose - Orchestrer vos Conteneurs 🎼

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- C'est quoi Docker Compose et pourquoi c'est génial
- Écrire un fichier docker-compose.yml
- Lancer une application complète en UNE commande
- Gérer plusieurs services (frontend, backend, base de données)
- Les commandes essentielles de Docker Compose
- Projet complet : Application full-stack

**Durée : 45 minutes (lecture + pratique)**

---

## 😫 Partie 1 : Le Problème (sans Docker Compose)

### Situation : Lancer WordPress

**Rappelez-vous le Cours 4...**

Pour lancer WordPress, vous deviez taper :

```bash
# 1. Créer le réseau
docker network create wordpress-network

# 2. Lancer MySQL
docker run -d \
  --name wp-database \
  --network wordpress-network \
  -e MYSQL_ROOT_PASSWORD=root_password \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wp_user \
  -e MYSQL_PASSWORD=wp_password \
  -v wordpress-db:/var/lib/mysql \
  mysql:8.0

# 3. Attendre que MySQL démarre
sleep 10

# 4. Lancer WordPress
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

**C'est LONG, COMPLIQUÉ et PÉNIBLE !** 😤

**Et pour arrêter tout :**
```bash
docker stop wp-site wp-database
docker rm wp-site wp-database
docker network rm wordpress-network
```

**Encore plus de commandes !** 😫

### Les problèmes

❌ **Trop de commandes** : 10+ lignes à taper  
❌ **Erreurs faciles** : Oubli d'une option = ça ne marche pas  
❌ **Pas reproductible** : Difficile de partager avec l'équipe  
❌ **Pas de documentation** : Impossible de se souvenir de tout  

**Il doit y avoir une meilleure solution...**

---

## 🎉 Partie 2 : La Solution = Docker Compose !

### C'est quoi Docker Compose ?

**Docker Compose = Un outil pour définir et gérer des applications multi-conteneurs**

**En gros :**
- Vous écrivez UN fichier de configuration (`docker-compose.yml`)
- Ce fichier décrit TOUS vos services (bases de données, API, frontend...)
- Une seule commande lance TOUT : `docker-compose up`
- Une seule commande arrête TOUT : `docker-compose down`

**Analogie :**
- **Sans Compose** : Préparer un repas en suivant 50 instructions orales
- **Avec Compose** : Suivre une recette écrite, claire et complète

### Exemple : WordPress avec Compose

**Au lieu de 20 lignes de commandes, un seul fichier :**

```yaml
version: '3.8'

services:
  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wp_password
    volumes:
      - db-data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_password
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp-files:/var/www/html
    depends_on:
      - database

volumes:
  db-data:
  wp-files:
```

**Et pour lancer TOUT :**
```bash
docker-compose up -d
```

**C'est tout ! 3 mots !** 🎉

**Pour arrêter tout :**
```bash
docker-compose down
```

**Magique !** ✨

---

## 📝 Partie 3 : La Structure du fichier docker-compose.yml

### Format YAML

**YAML = Un format de fichier simple et lisible**

**Règles importantes :**
- ⚠️ **L'indentation compte** (2 espaces, PAS de tabulations)
- Les clés et valeurs sont séparées par `:`
- Les listes commencent par `-`

**Exemple simple :**
```yaml
nom: Alice
age: 25
hobbies:
  - lecture
  - musique
  - sport
```

### Structure de base

**Un fichier docker-compose.yml a cette structure :**

```yaml
version: '3.8'          # Version de Docker Compose

services:               # Liste de tous vos conteneurs
  service1:
    # Configuration du service 1
  service2:
    # Configuration du service 2

volumes:                # Volumes partagés (optionnel)
  volume1:
  volume2:

networks:               # Réseaux personnalisés (optionnel)
  network1:
```

### Les sections principales

**1. `version`** : Version du format (utilisez toujours '3.8')

**2. `services`** : Vos conteneurs
- Chaque service = un conteneur
- Vous définissez l'image, les ports, les volumes, etc.

**3. `volumes`** : Stockage permanent (comme `docker volume create`)

**4. `networks`** : Réseaux personnalisés (créés automatiquement si non spécifiés)

---

## 🛠️ Partie 4 : Votre Premier docker-compose.yml

### Projet Simple : Site web + Base de données

**Étape 1 : Créer le dossier**
```bash
mkdir ~/mon-premier-compose
cd ~/mon-premier-compose
```

**Étape 2 : Créer le fichier docker-compose.yml**

⚠️ **ATTENTION : Le nom doit être EXACTEMENT `docker-compose.yml`**

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  # Service 1 : Base de données PostgreSQL
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret123
      POSTGRES_USER: admin
      POSTGRES_DB: ma_base
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  # Service 2 : Adminer (interface web pour la DB)
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - database

volumes:
  db-data:
EOF
```

**Explication ligne par ligne :**

```yaml
version: '3.8'                    # Version du format
```

```yaml
services:                         # Début de la liste des services
```

```yaml
  database:                       # Nom du service (libre choix)
    image: postgres:15            # Image à utiliser
    environment:                  # Variables d'environnement
      POSTGRES_PASSWORD: secret123
      POSTGRES_USER: admin
      POSTGRES_DB: ma_base
    volumes:                      # Volumes pour persister
      - db-data:/var/lib/postgresql/data
    ports:                        # Exposition des ports
      - "5432:5432"               # Format: "PC:CONTENEUR"
```

```yaml
  adminer:                        # Deuxième service
    image: adminer
    ports:
      - "8080:8080"
    depends_on:                   # Démarre APRÈS database
      - database
```

```yaml
volumes:                          # Déclaration des volumes
  db-data:                        # Compose le créera automatiquement
```

**Étape 3 : Lancer avec Docker Compose**
```bash
docker-compose up -d
```

**Options :**
- `up` : Démarrer les services
- `-d` : Mode détaché (arrière-plan)

**Ce que vous verrez :**
```
Creating network "mon-premier-compose_default" with the default driver
Creating volume "mon-premier-compose_db-data" with default driver
Creating mon-premier-compose_database_1 ... done
Creating mon-premier-compose_adminer_1  ... done
```

✅ **Tout est créé automatiquement !**
- Réseau
- Volume
- 2 conteneurs

**Étape 4 : Vérifier que ça tourne**
```bash
docker-compose ps
```

**Résultat :**
```
Name                           State    Ports
----------------------------------------------------------------
mon-premier-compose_adminer_1    Up      0.0.0.0:8080->8080/tcp
mon-premier-compose_database_1   Up      0.0.0.0:5432->5432/tcp
```

**Étape 5 : Tester dans le navigateur**

Ouvrez : `http://localhost:8080`

**Connexion à Adminer :**
- Système : PostgreSQL
- Serveur : `database` ← Le nom du service !
- Utilisateur : `admin`
- Mot de passe : `secret123`
- Base de données : `ma_base`

🎉 **Ça marche !**

**Étape 6 : Voir les logs**
```bash
docker-compose logs
```

**Logs d'un service spécifique :**
```bash
docker-compose logs database
```

**Suivre les logs en temps réel :**
```bash
docker-compose logs -f
```

**Étape 7 : Arrêter tout**
```bash
docker-compose down
```

**Ce qui se passe :**
```
Stopping mon-premier-compose_adminer_1  ... done
Stopping mon-premier-compose_database_1 ... done
Removing mon-premier-compose_adminer_1  ... done
Removing mon-premier-compose_database_1 ... done
Removing network mon-premier-compose_default
```

✅ **Tout est arrêté et nettoyé !**

**Note :** Les volumes restent (vos données sont sauvées)

**Pour tout supprimer (volumes inclus) :**
```bash
docker-compose down -v
```

⚠️ **ATTENTION :** `-v` supprime les volumes = perte des données !

---

## 🎓 Partie 5 : Les Options Essentielles

### Option 1 : image

**Utiliser une image existante :**
```yaml
services:
  mon-service:
    image: nginx:alpine
```

### Option 2 : build

**Construire depuis un Dockerfile :**
```yaml
services:
  mon-app:
    build: .                    # Dockerfile dans le dossier actuel
    # ou
    build:
      context: ./app            # Dossier avec le Dockerfile
      dockerfile: Dockerfile    # Nom du Dockerfile
```

### Option 3 : ports

**Exposer des ports :**
```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"               # PC:CONTENEUR
      - "8443:443"              # Plusieurs ports
```

### Option 4 : environment

**Variables d'environnement :**
```yaml
services:
  api:
    image: node:18
    environment:
      NODE_ENV: production
      PORT: 3000
      DATABASE_URL: postgresql://user:pass@db:5432
```

**OU avec un fichier .env :**
```yaml
services:
  api:
    image: node:18
    env_file:
      - .env                    # Charge les variables depuis .env
```

### Option 5 : volumes

**Monter des volumes :**
```yaml
services:
  database:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data    # Volume nommé
      - ./config:/etc/config                # Bind mount
      - /var/run/docker.sock:/var/run/docker.sock  # Socket

volumes:
  db-data:
```

### Option 6 : depends_on

**Ordre de démarrage :**
```yaml
services:
  database:
    image: postgres

  backend:
    image: mon-api
    depends_on:
      - database              # Backend démarre APRÈS database

  frontend:
    image: mon-site
    depends_on:
      - backend               # Frontend démarre APRÈS backend
```

**Note :** `depends_on` attend juste que le conteneur démarre, pas qu'il soit "prêt".

### Option 7 : command

**Surcharger la commande par défaut :**
```yaml
services:
  app:
    image: node:18
    command: npm run dev        # Au lieu de la commande par défaut
```

### Option 8 : restart

**Politique de redémarrage :**
```yaml
services:
  app:
    image: nginx
    restart: always             # Toujours redémarrer en cas d'arrêt
    # Autres options :
    # restart: "no"             # Ne jamais redémarrer
    # restart: on-failure       # Seulement si erreur
    # restart: unless-stopped   # Toujours sauf si arrêté manuellement
```

### Option 9 : networks

**Réseaux personnalisés :**
```yaml
services:
  app:
    image: nginx
    networks:
      - frontend
      - backend

networks:
  frontend:
  backend:
```

### Option 10 : container_name

**Nom personnalisé du conteneur :**
```yaml
services:
  web:
    image: nginx
    container_name: mon-nginx   # Au lieu de "projet_web_1"
```

---

## 💻 Partie 6 : Projet Complet - Application Full-Stack

### Mission : Créer une app avec Frontend + Backend + Database

**Architecture :**
```
Frontend (React) ← Port 3000
    ↓
Backend (Node.js API) ← Port 4000
    ↓
Database (PostgreSQL) ← Port 5432
```

**Étape 1 : Créer la structure**
```bash
mkdir ~/app-fullstack
cd ~/app-fullstack
mkdir backend frontend
```

---

### BACKEND (API Node.js)

**Créer backend/package.json :**
```bash
cat > backend/package.json << 'EOF'
{
  "name": "backend",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.11.0",
    "cors": "^2.8.5"
  }
}
EOF
```

**Créer backend/server.js :**
```bash
cat > backend/server.js << 'EOF'
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');

const app = express();
app.use(cors());
app.use(express.json());

// Connexion PostgreSQL
const pool = new Pool({
  host: process.env.DB_HOST || 'database',
  port: 5432,
  user: process.env.DB_USER || 'admin',
  password: process.env.DB_PASSWORD || 'secret',
  database: process.env.DB_NAME || 'appdb'
});

// Route de test
app.get('/', (req, res) => {
  res.json({ message: "API Backend fonctionne !" });
});

// Route pour récupérer des utilisateurs
app.get('/api/users', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM users ORDER BY id');
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Route pour ajouter un utilisateur
app.post('/api/users', async (req, res) => {
  const { name, email } = req.body;
  try {
    const result = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      [name, email]
    );
    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, '0.0.0.0', () => {
  console.log(`🚀 Backend API sur le port ${PORT}`);
});
EOF
```

**Créer backend/Dockerfile :**
```bash
cat > backend/Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 4000
CMD ["npm", "start"]
EOF
```

---

### FRONTEND (HTML simple)

**Créer frontend/index.html :**
```bash
cat > frontend/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>App Full-Stack</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
            background: #f5f5f5;
        }
        h1 { color: #333; }
        .user-form {
            background: white;
            padding: 20px;
            border-radius: 8px;
            margin-bottom: 20px;
        }
        input {
            margin: 10px 0;
            padding: 10px;
            width: 200px;
        }
        button {
            padding: 10px 20px;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover { background: #45a049; }
        .users-list {
            background: white;
            padding: 20px;
            border-radius: 8px;
        }
        .user-item {
            padding: 10px;
            border-bottom: 1px solid #eee;
        }
    </style>
</head>
<body>
    <h1>🚀 Application Full-Stack avec Docker Compose</h1>
    
    <div class="user-form">
        <h2>Ajouter un utilisateur</h2>
        <input type="text" id="name" placeholder="Nom">
        <input type="email" id="email" placeholder="Email">
        <button onclick="addUser()">Ajouter</button>
    </div>

    <div class="users-list">
        <h2>Liste des utilisateurs</h2>
        <div id="users"></div>
    </div>

    <script>
        const API_URL = 'http://localhost:4000';

        async function loadUsers() {
            try {
                const response = await fetch(`${API_URL}/api/users`);
                const users = await response.json();
                const usersDiv = document.getElementById('users');
                usersDiv.innerHTML = users.map(user => 
                    `<div class="user-item">
                        <strong>${user.name}</strong> - ${user.email}
                    </div>`
                ).join('');
            } catch (err) {
                console.error('Erreur:', err);
                document.getElementById('users').innerHTML = 
                    '<p style="color: red;">Erreur de chargement des utilisateurs</p>';
            }
        }

        async function addUser() {
            const name = document.getElementById('name').value;
            const email = document.getElementById('email').value;
            
            if (!name || !email) {
                alert('Veuillez remplir tous les champs');
                return;
            }

            try {
                await fetch(`${API_URL}/api/users`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ name, email })
                });
                
                document.getElementById('name').value = '';
                document.getElementById('email').value = '';
                loadUsers();
            } catch (err) {
                alert('Erreur lors de l\'ajout');
            }
        }

        // Charger les utilisateurs au démarrage
        loadUsers();
        // Recharger toutes les 3 secondes
        setInterval(loadUsers, 3000);
    </script>
</body>
</html>
EOF
```

**Créer frontend/Dockerfile :**
```bash
cat > frontend/Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
EOF
```

---

### DOCKER COMPOSE

**Créer docker-compose.yml (racine du projet) :**
```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  # Base de données PostgreSQL
  database:
    image: postgres:15
    container_name: fullstack-db
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API Node.js
  backend:
    build: ./backend
    container_name: fullstack-backend
    environment:
      DB_HOST: database
      DB_USER: admin
      DB_PASSWORD: secret
      DB_NAME: appdb
      PORT: 4000
    ports:
      - "4000:4000"
    depends_on:
      database:
        condition: service_healthy
    restart: unless-stopped

  # Frontend
  frontend:
    build: ./frontend
    container_name: fullstack-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    restart: unless-stopped

volumes:
  db-data:
EOF
```

---

### SCRIPT D'INITIALISATION DE LA BASE

**Créer init-db.sql :**
```bash
cat > init-db.sql << 'EOF'
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (name, email) VALUES 
    ('Alice Dupont', 'alice@example.com'),
    ('Bob Martin', 'bob@example.com'),
    ('Charlie Durand', 'charlie@example.com');
EOF
```

**Modifier docker-compose.yml pour initialiser la DB :**
```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  database:
    image: postgres:15
    container_name: fullstack-db
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    container_name: fullstack-backend
    environment:
      DB_HOST: database
      DB_USER: admin
      DB_PASSWORD: secret
      DB_NAME: appdb
      PORT: 4000
    ports:
      - "4000:4000"
    depends_on:
      database:
        condition: service_healthy
    restart: unless-stopped

  frontend:
    build: ./frontend
    container_name: fullstack-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    restart: unless-stopped

volumes:
  db-data:
EOF
```

---

### LANCEMENT

**Vérifier la structure :**
```bash
tree
```

**Vous devez avoir :**
```
app-fullstack/
├── docker-compose.yml
├── init-db.sql
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── frontend/
    ├── Dockerfile
    └── index.html
```

**Lancer l'application :**
```bash
docker-compose up -d --build
```

**Options :**
- `up` : Démarrer
- `-d` : Arrière-plan
- `--build` : Reconstruire les images

**Attendre 30 secondes que tout démarre.**

**Vérifier :**
```bash
docker-compose ps
```

Vous devez voir 3 services "Up" ! ✅

---

### TESTER L'APPLICATION

**Ouvrir le navigateur :**
```
http://localhost:3000
```

🎉 **Vous voyez votre application avec 3 utilisateurs déjà présents !**

**Testez :**
1. Ajoutez un nouvel utilisateur (nom + email)
2. Cliquez sur "Ajouter"
3. L'utilisateur apparaît dans la liste !

**Ce qui se passe :**
```
Frontend (port 3000)
    ↓ Appel API
Backend (port 4000)
    ↓ Requête SQL
Database (PostgreSQL)
```

**Tout communique parfaitement !** ✨

---

### VOIR LES LOGS

**Tous les services :**
```bash
docker-compose logs -f
```

**Un service spécifique :**
```bash
docker-compose logs -f backend
```

**Arrêter tout :**
```bash
docker-compose down
```

**Les données restent dans le volume `db-data` !**

**Redémarrer :**
```bash
docker-compose up -d
```

**Les utilisateurs sont toujours là !** 🎉

---

## 🎮 Partie 7 : Commandes Docker Compose Essentielles

### Gestion des services

**Démarrer tous les services :**
```bash
docker-compose up
docker-compose up -d              # En arrière-plan
docker-compose up -d --build      # Reconstruire avant de démarrer
```

**Démarrer un service spécifique :**
```bash
docker-compose up database
```

**Arrêter tous les services :**
```bash
docker-compose stop
```

**Arrêter et supprimer (mais garde les volumes) :**
```bash
docker-compose down
```

**Tout supprimer (volumes inclus) :**
```bash
docker-compose down -v
```

---

### Voir l'état

**Lister les conteneurs :**
```bash
docker-compose ps
```

**Voir les logs :**
```bash
docker-compose logs
docker-compose logs -f            # Suivre en temps réel
docker-compose logs backend       # Logs d'un service
docker-compose logs -f --tail 50  # 50 dernières lignes
```

---

### Exécuter des commandes

**Exécuter une commande dans un service :**
```bash
docker-compose exec backend sh
docker-compose exec database psql -U admin -d appdb
```

**Exemple : Se connecter à PostgreSQL :**
```bash
docker-compose exec database psql -U admin -d appdb
```

Vous êtes dans PostgreSQL ! Tapez :
```sql
\dt                    -- Voir les tables
SELECT * FROM users;   -- Voir les données
\q                     -- Quitter
```

---

### Reconstruction

**Reconstruire les images :**
```bash
docker-compose build
docker-compose build backend      # Rebuild un service
docker-compose build --no-cache   # Sans cache
```

**Rebuild et restart :**
```bash
docker-compose up -d --build
```

---

### Mise à l'échelle

**Lancer plusieurs instances d'un service :**
```bash
docker-compose up -d --scale backend=3
```

**3 instances du backend !** (Nécessite un load balancer en production)

---

## 📝 Partie 8 : Fichier .env pour les secrets

### Le problème

**Mettre les mots de passe dans docker-compose.yml :**
```yaml
environment:
  POSTGRES_PASSWORD: mon_super_secret_123  # ← Visible par tous !
```

❌ **Mauvaise idée si vous partagez le code (Git) !**

### La solution : Fichier .env

**Créer .env :**
```bash
cat > .env << 'EOF'
# Base de données
DB_USER=admin
DB_PASSWORD=super_secret_password
DB_NAME=appdb

# Backend
API_PORT=4000
NODE_ENV=production
EOF
```

**Modifier docker-compose.yml :**
```yaml
services:
  database:
    image: postgres:15
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
  
  backend:
    build: ./backend
    environment:
      DB_PASSWORD: ${DB_PASSWORD}
      PORT: ${API_PORT}
    ports:
      - "${API_PORT}:4000"
```

**Ajouter .env au .gitignore :**
```bash
echo ".env" >> .gitignore
```

**Créer .env.example (à partager) :**
```bash
cat > .env.example << 'EOF'
DB_USER=admin
DB_PASSWORD=changez_moi
DB_NAME=appdb
API_PORT=4000
NODE_ENV=production
EOF
```

✅ **Maintenant les secrets ne sont pas dans Git !**

---

## ✅ Quiz de validation

**Question 1 : Quelle commande pour démarrer tous les services ?**

- A) `docker-compose start`
- B) `docker-compose run`
- C) `docker-compose up`
- D) `docker-compose launch`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : `docker-compose up`**

`docker-compose up` crée et démarre tous les services définis dans docker-compose.yml. Ajoutez `-d` pour l'arrière-plan.

</details>

---

**Question 2 : À quoi sert `depends_on` ?**

- A) Définir les dépendances npm
- B) Définir l'ordre de démarrage des services
- C) Installer des paquets
- D) Créer des volumes

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Définir l'ordre de démarrage**

`depends_on` indique qu'un service doit démarrer après un autre. Exemple : le backend démarre après la base de données.

</details>

---

**Question 3 : Comment voir les logs en temps réel ?**

- A) `docker-compose logs`
- B) `docker-compose logs -f`
- C) `docker-compose watch`
- D) `docker-compose tail`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : `docker-compose logs -f`**

L'option `-f` (follow) permet de suivre les logs en temps réel, comme `tail -f`.

</details>

---

**Question 4 : Comment supprimer tout (services + volumes) ?**

- A) `docker-compose down`
- B) `docker-compose down -v`
- C) `docker-compose remove`
- D) `docker-compose delete -all`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : `docker-compose down -v`**

`down` arrête et supprime les conteneurs. L'option `-v` supprime aussi les volumes (donc les données !).

</details>

---

**Question 5 : Où mettre les mots de passe sensibles ?**

- A) Dans docker-compose.yml
- B) Dans un fichier .env
- C) Dans le code source
- D) Dans un commentaire

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Dans un fichier .env**

Le fichier .env contient les secrets et ne doit PAS être commité dans Git. Ajoutez-le au .gitignore !

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Le fichier docker-compose.yml

```yaml
version: '3.8'

services:
  nom-service:
    image: nom-image        # OU build: ./chemin
    ports:
      - "8080:80"
    environment:
      VAR: valeur
    volumes:
      - volume:/chemin
    depends_on:
      - autre-service

volumes:
  volume:
```

### Commandes essentielles

```bash
# Démarrer
docker-compose up -d

# Arrêter
docker-compose down

# Voir l'état
docker-compose ps

# Logs
docker-compose logs -f

# Rebuild
docker-compose up -d --build

# Entrer dans un conteneur
docker-compose exec service sh

# Tout supprimer (avec volumes)
docker-compose down -v
```

### Avantages

✅ **Un seul fichier** : Toute la config au même endroit  
✅ **Une commande** : `up` lance tout  
✅ **Reproductible** : Même résultat partout  
✅ **Partageable** : Donnez le fichier à l'équipe  
✅ **Simple** : Plus besoin de se souvenir de 20 commandes  

---

## 🚀 Et maintenant ?

**BRAVO ! Vous maîtrisez Docker Compose !** 🎉

### Ce que vous savez maintenant :

- ✅ Écrire un fichier docker-compose.yml
- ✅ Orchestrer plusieurs services ensemble
- ✅ Créer une application full-stack en 1 commande
- ✅ Gérer les volumes, réseaux, et variables
- ✅ Utiliser .env pour les secrets

### Dans le prochain cours (Cours 9) :

**Les Réseaux Docker en Détail** 🕸️

Vous apprendrez :
1. Les 3 types de réseaux Docker
2. Créer et gérer des réseaux personnalisés
3. Isolation et sécurité réseau
4. Communication inter-conteneurs avancée
5. Cas pratiques de réseaux complexes

---

## 📚 Template docker-compose.yml (à réutiliser)

**Application Web complète :**
```yaml
version: '3.8'

services:
  database:
    image: postgres:15
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@database:5432/${DB_NAME}
    ports:
      - "${API_PORT}:4000"
    depends_on:
      database:
        condition: service_healthy
    restart: unless-stopped

  frontend:
    build: ./frontend
    ports:
      - "${WEB_PORT}:80"
    depends_on:
      - backend
    restart: unless-stopped

volumes:
  db-data:
```

**Fichier .env correspondant :**
```
DB_USER=admin
DB_PASSWORD=changez_moi
DB_NAME=myapp
API_PORT=4000
WEB_PORT=3000
```

---

**🎓 Vous êtes prêt pour le Cours 9 (Réseaux Docker) !**

Excellent travail ! Docker Compose va vous faire gagner un temps fou. Le prochain cours approfondit les réseaux Docker pour des architectures plus complexes. Continue comme ça ! 💪🚀
