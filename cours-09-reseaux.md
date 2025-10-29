# Cours 9 : Les Réseaux Docker en Détail 🕸️

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Les 3 types de réseaux Docker (bridge, host, none)
- Créer et gérer des réseaux personnalisés
- Isoler des services pour la sécurité
- Faire communiquer des conteneurs de manière avancée
- Diagnostiquer les problèmes réseau
- Cas pratiques d'architectures complexes

**Durée : 35 minutes (lecture + pratique)**

---

## 📖 Partie 1 : Comprendre les Réseaux Docker

### Le réseau par défaut

**Quand vous lancez un conteneur sans préciser de réseau :**
```bash
docker run -d nginx
```

**Docker le met automatiquement dans un réseau appelé `bridge`.**

**Voir les réseaux existants :**
```bash
docker network ls
```

**Résultat :**
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
789ghi012jkl   host      host      local
mno345pqr678   none      null      local
```

**3 réseaux par défaut :**
- **bridge** : Le réseau par défaut
- **host** : Partage le réseau de votre PC
- **none** : Pas de réseau du tout

### Pourquoi les réseaux sont importants ?

**Cas d'usage :**

**1. Communication entre conteneurs**
```
Frontend ← parle à → Backend ← parle à → Database
```

**2. Isolation (sécurité)**
```
Services publics (réseau public)
Services internes (réseau privé)
```

**3. Architectures complexes**
```
Réseau frontend : nginx, react
Réseau backend : api, workers
Réseau database : postgres, redis
```

---

## 🌉 Partie 2 : Le Réseau Bridge (Par défaut)

### C'est quoi ?

**Bridge = Pont**

Docker crée un pont réseau virtuel entre vos conteneurs.

**Schéma :**
```
Votre PC (192.168.1.100)
    ↓
Réseau bridge Docker (172.17.0.0/16)
    ↓
Conteneur 1 (172.17.0.2)
Conteneur 2 (172.17.0.3)
Conteneur 3 (172.17.0.4)
```

### Test pratique

**Lancer 2 conteneurs :**
```bash
docker run -d --name web1 nginx
docker run -d --name web2 nginx
```

**Inspecter le réseau bridge :**
```bash
docker network inspect bridge
```

**Vous verrez :**
```json
"Containers": {
    "abc123": {
        "Name": "web1",
        "IPv4Address": "172.17.0.2/16"
    },
    "def456": {
        "Name": "web2",
        "IPv4Address": "172.17.0.3/16"
    }
}
```

**Les conteneurs peuvent communiquer par IP :**
```bash
docker exec web1 ping 172.17.0.3
```

✅ **Ça marche !**

### Limites du bridge par défaut

❌ **Les conteneurs ne peuvent PAS communiquer par nom**

**Test :**
```bash
docker exec web1 ping web2
```

**Résultat :**
```
ping: bad address 'web2'
```

❌ **Ça ne marche pas !**

**Solution : Créer un réseau personnalisé !**

---

## 🔧 Partie 3 : Créer des Réseaux Personnalisés

### Créer un réseau

**Commande :**
```bash
docker network create mon-reseau
```

**Vérifier :**
```bash
docker network ls
```

Vous voyez `mon-reseau` dans la liste ! ✅

### Lancer des conteneurs dans ce réseau

**Conteneur 1 :**
```bash
docker run -d --name app1 --network mon-reseau nginx
```

**Conteneur 2 :**
```bash
docker run -d --name app2 --network mon-reseau nginx
```

**Maintenant tester la communication par NOM :**
```bash
docker exec app1 ping app2
```

**Résultat :**
```
PING app2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.123 ms
```

🎉 **Ça marche ! Les conteneurs communiquent par nom !**

**C'est la magie des réseaux personnalisés : DNS automatique !**

### Avantages des réseaux personnalisés

✅ **Communication par nom** (pas besoin de connaître les IP)  
✅ **Isolation** (séparé des autres conteneurs)  
✅ **Contrôle** (vous décidez qui communique avec qui)  
✅ **Meilleure organisation** (un réseau par projet)  

---

## 🏗️ Partie 4 : Projet Pratique - Architecture Multi-Réseaux

### Mission : Créer une application avec 2 réseaux

**Architecture :**
```
RÉSEAU FRONTEND (public)
  - nginx (reverse proxy)
  - frontend web

RÉSEAU BACKEND (privé)
  - API Node.js
  - Database PostgreSQL
```

**Règles de sécurité :**
- Frontend peut parler au Backend ✅
- Backend peut parler à la Database ✅
- Frontend ne peut PAS parler directement à la Database ❌

### Étape par étape

**Étape 1 : Créer les 2 réseaux**
```bash
docker network create reseau-frontend
docker network create reseau-backend
```

**Étape 2 : Lancer la base de données (réseau backend uniquement)**
```bash
docker run -d \
  --name database \
  --network reseau-backend \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=appdb \
  postgres:15
```

**Étape 3 : Créer l'API (dans les DEUX réseaux)**

**Créer le dossier :**
```bash
mkdir ~/app-multi-reseau
cd ~/app-multi-reseau
```

**Créer package.json :**
```bash
cat > package.json << 'EOF'
{
  "name": "api",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.11.0"
  }
}
EOF
```

**Créer server.js :**
```bash
cat > server.js << 'EOF'
const express = require('express');
const { Pool } = require('pg');

const app = express();

const pool = new Pool({
  host: 'database',
  user: 'admin',
  password: 'secret',
  database: 'appdb',
  port: 5432
});

app.get('/api/status', (req, res) => {
  res.json({ status: 'API fonctionne !', network: 'backend' });
});

app.get('/api/db-test', async (req, res) => {
  try {
    const result = await pool.query('SELECT NOW()');
    res.json({ 
      database: 'Connexion OK', 
      time: result.rows[0].now 
    });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(3000, '0.0.0.0', () => {
  console.log('API démarrée sur le port 3000');
});
EOF
```

**Créer Dockerfile :**
```bash
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
EOF
```

**Construire l'image :**
```bash
docker build -t mon-api .
```

**Lancer l'API dans les DEUX réseaux :**
```bash
docker run -d \
  --name api \
  --network reseau-backend \
  mon-api

# Connecter aussi au réseau frontend
docker network connect reseau-frontend api
```

**Étape 4 : Lancer le frontend (réseau frontend uniquement)**
```bash
docker run -d \
  --name frontend \
  --network reseau-frontend \
  -p 8080:80 \
  nginx:alpine
```

**Étape 5 : Vérifier l'isolation**

**Frontend peut parler à l'API :**
```bash
docker exec frontend ping api
```
✅ **Ça marche !**

**Frontend NE PEUT PAS parler à la database :**
```bash
docker exec frontend ping database
```
❌ **Ça ne marche pas ! (erreur : bad address 'database')**

**API peut parler à la database :**
```bash
docker exec api ping database
```
✅ **Ça marche !**

🎉 **L'isolation fonctionne parfaitement !**

**Schéma de ce qu'on a créé :**
```
┌──────────────────────────────────────┐
│      RÉSEAU FRONTEND                 │
│                                      │
│  [frontend:nginx]  ←→  [api:node]   │
│                                      │
└──────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────┐
│      RÉSEAU BACKEND                  │
│                                      │
│  [api:node]  ←→  [database:postgres] │
│                                      │
└──────────────────────────────────────┘
```

**L'API est dans les 2 réseaux (passerelle) !**

**Nettoyage :**
```bash
docker stop frontend api database
docker rm frontend api database
docker network rm reseau-frontend reseau-backend
```

---

## 🔌 Partie 5 : Le Réseau Host

### C'est quoi ?

**Mode host = Le conteneur utilise directement le réseau de votre PC**

Pas d'isolation réseau, le conteneur voit tout comme votre ordinateur.

### Quand l'utiliser ?

**Cas d'usage :**
- Performance maximale (pas de couche réseau Docker)
- Applications qui ont besoin d'accéder à tout le réseau
- Tests et développement

### Exemple

**Lancer nginx en mode host :**
```bash
docker run -d --name web-host --network host nginx
```

**Note :** Pas besoin de `-p 8080:80` car nginx écoute directement sur le port 80 de votre PC !

**Tester :**
```
http://localhost
```

Vous voyez nginx ! (Si le port 80 est libre sur votre PC)

**Voir les processus réseau :**
```bash
docker exec web-host netstat -tulpn
```

**Nettoyage :**
```bash
docker stop web-host
docker rm web-host
```

### Avantages et inconvénients

**Avantages :**
✅ Performance maximale (pas de traduction réseau)  
✅ Accès direct à tout le réseau de l'hôte  

**Inconvénients :**
❌ Pas d'isolation (moins sécurisé)  
❌ Conflits de ports possibles  
❌ Moins portable (dépend de la config réseau de l'hôte)  

**Conclusion : Utilisez rarement, seulement si besoin spécifique.**

---

## 🚫 Partie 6 : Le Réseau None

### C'est quoi ?

**Mode none = Aucun réseau du tout**

Le conteneur est complètement isolé, même pas de loopback.

### Quand l'utiliser ?

**Cas d'usage :**
- Traitement de fichiers sans connexion réseau
- Sécurité maximale
- Tests d'isolation

### Exemple

```bash
docker run -d --name isolated --network none alpine sleep 3600
```

**Tester :**
```bash
docker exec isolated ping google.com
```

**Résultat :**
```
ping: bad address 'google.com'
```

❌ **Pas de réseau du tout !**

Même pas Internet, même pas les autres conteneurs.

**Nettoyage :**
```bash
docker stop isolated
docker rm isolated
```

---

## 🎓 Partie 7 : Commandes de Gestion des Réseaux

### Créer un réseau

**Basique :**
```bash
docker network create mon-reseau
```

**Avec options :**
```bash
docker network create \
  --driver bridge \
  --subnet 192.168.100.0/24 \
  --gateway 192.168.100.1 \
  mon-reseau-custom
```

### Lister les réseaux

```bash
docker network ls
```

### Inspecter un réseau

```bash
docker network inspect mon-reseau
```

**Informations affichées :**
- Subnet et Gateway
- Liste des conteneurs connectés
- Configuration DNS
- Options du driver

### Connecter un conteneur à un réseau

**Pendant la création :**
```bash
docker run -d --name app --network mon-reseau nginx
```

**Après la création :**
```bash
docker network connect mon-reseau app
```

**Un conteneur peut être dans PLUSIEURS réseaux !**

### Déconnecter un conteneur

```bash
docker network disconnect mon-reseau app
```

### Supprimer un réseau

```bash
docker network rm mon-reseau
```

⚠️ **Pas possible si des conteneurs sont connectés !**

**Supprimer tous les réseaux non utilisés :**
```bash
docker network prune
```

---

## 🐛 Partie 8 : Diagnostiquer les Problèmes Réseau

### Problème 1 : Les conteneurs ne communiquent pas

**Symptôme :**
```bash
docker exec app1 ping app2
# ping: bad address 'app2'
```

**Solutions à vérifier :**

**1. Sont-ils dans le même réseau ?**
```bash
docker network inspect mon-reseau
```

Vérifiez que les 2 conteneurs apparaissent dans la liste.

**2. Utilisent-ils le réseau bridge par défaut ?**

Le réseau bridge par défaut ne supporte PAS la résolution DNS par nom.

**Solution :** Créez un réseau personnalisé.

**3. Le conteneur destination est-il démarré ?**
```bash
docker ps
```

---

### Problème 2 : Impossible d'accéder depuis l'extérieur

**Symptôme :**
```
http://localhost:8080 ne répond pas
```

**Solutions :**

**1. Port mappé correctement ?**
```bash
docker ps
```

Vérifiez la colonne PORTS : `0.0.0.0:8080->80/tcp`

**2. Application écoute sur 0.0.0.0 ?**

Dans votre code, assurez-vous :
```javascript
app.listen(3000, '0.0.0.0'); // Pas 'localhost' !
```

**3. Firewall ?**
```bash
# Sur Ubuntu
sudo ufw status
sudo ufw allow 8080
```

---

### Problème 3 : Erreur "network not found"

**Symptôme :**
```
Error: network mon-reseau not found
```

**Solution :**

Le réseau n'existe pas ou a été supprimé.

**Créez-le :**
```bash
docker network create mon-reseau
```

---

### Outils de diagnostic

**1. Inspecter un conteneur :**
```bash
docker inspect app
```

Cherchez la section "Networks" pour voir les réseaux connectés.

**2. Tester la connectivité :**
```bash
# Ping
docker exec app1 ping app2

# Test HTTP
docker exec app1 wget -O- http://app2:80

# DNS lookup
docker exec app1 nslookup app2
```

**3. Voir les ports ouverts :**
```bash
docker exec app netstat -tulpn
```

---

## 💼 Partie 9 : Cas Pratiques Avancés

### Cas 1 : Application microservices

**Architecture :**
```
RÉSEAU PUBLIC
  - API Gateway (nginx)

RÉSEAU SERVICES
  - API Gateway (aussi dans ce réseau)
  - Service Users
  - Service Products
  - Service Orders

RÉSEAU DATABASE
  - Service Users (aussi dans ce réseau)
  - Service Products (aussi dans ce réseau)
  - Service Orders (aussi dans ce réseau)
  - PostgreSQL
  - Redis
```

**Avec Docker Compose :**
```yaml
version: '3.8'

services:
  gateway:
    image: nginx
    networks:
      - public
      - services
    ports:
      - "80:80"

  users:
    image: users-service
    networks:
      - services
      - database

  products:
    image: products-service
    networks:
      - services
      - database

  db:
    image: postgres:15
    networks:
      - database

networks:
  public:
  services:
  database:
```

---

### Cas 2 : Environnements Dev / Staging / Prod

**Créer des réseaux par environnement :**
```bash
docker network create dev-network
docker network create staging-network
docker network create prod-network
```

**Lancer les services dans l'environnement approprié :**
```bash
# Développement
docker-compose -f docker-compose.dev.yml up -d

# Staging
docker-compose -f docker-compose.staging.yml up -d

# Production
docker-compose -f docker-compose.prod.yml up -d
```

**Isolation totale entre environnements !** ✅

---

### Cas 3 : Services partagés

**Certains services doivent être accessibles depuis plusieurs réseaux :**

**Exemple : Service de logs**
```bash
# Créer le réseau de logs
docker network create logs-network

# Lancer le service de logs
docker run -d --name logger --network logs-network mon-logger

# Connecter chaque application au réseau de logs
docker network connect logs-network app1
docker network connect logs-network app2
docker network connect logs-network app3
```

**Maintenant tous les apps peuvent envoyer leurs logs au logger !**

---

## 📝 Partie 10 : Bonnes Pratiques

### ✅ À FAIRE

**1. Créer des réseaux personnalisés**
```bash
docker network create mon-projet
```

Au lieu d'utiliser le bridge par défaut.

**2. Un réseau par projet**
```bash
docker network create projet-blog
docker network create projet-ecommerce
```

Isolation et organisation.

**3. Nommer clairement les réseaux**
```bash
# BON ✅
docker network create mon-app-frontend
docker network create mon-app-backend

# MOINS BON ⚠️
docker network create network1
docker network create network2
```

**4. Utiliser Docker Compose pour les réseaux**
```yaml
networks:
  frontend:
    name: mon-app-frontend
  backend:
    name: mon-app-backend
```

**5. Documenter l'architecture réseau**

Créez un schéma ou un README expliquant les réseaux.

---

### ❌ À ÉVITER

**1. Ne pas tout mettre dans le même réseau**

Séparez les couches pour la sécurité :
- Frontend
- Backend
- Database

**2. Ne pas utiliser le mode host en production**

Sauf besoin très spécifique.

**3. Ne pas oublier de nettoyer**
```bash
docker network prune
```

Régulièrement.

**4. Ne pas hardcoder les IP**

Utilisez les noms de services :
```javascript
// BON ✅
const dbHost = 'database';

// MAUVAIS ❌
const dbHost = '172.18.0.3'; // L'IP peut changer !
```

---

## ✅ Quiz de validation

**Question 1 : Quelle différence entre bridge par défaut et réseau personnalisé ?**

- A) Aucune différence
- B) Le réseau personnalisé permet la communication par nom (DNS)
- C) Le bridge est plus rapide
- D) Le bridge est plus sécurisé

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Le réseau personnalisé permet la communication par nom**

Dans un réseau personnalisé, Docker fournit un DNS automatique : les conteneurs peuvent se parler par leur nom au lieu des IP.

</details>

---

**Question 2 : Comment connecter un conteneur à plusieurs réseaux ?**

- A) Ce n'est pas possible
- B) Avec `docker network connect`
- C) En créant plusieurs conteneurs
- D) Avec l'option `-n`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : `docker network connect`**

Exemple : `docker network connect mon-reseau mon-conteneur` ajoute le conteneur au réseau sans le déconnecter des autres.

</details>

---

**Question 3 : Quel mode réseau pour une isolation totale ?**

- A) bridge
- B) host
- C) none
- D) isolated

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : none**

Le mode `none` coupe tout accès réseau au conteneur. Même pas de loopback !

</details>

---

**Question 4 : Pourquoi isoler la base de données dans un réseau privé ?**

- A) Pour qu'elle aille plus vite
- B) Pour la sécurité : seuls les services autorisés peuvent y accéder
- C) C'est obligatoire
- D) Pour économiser de la RAM

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Pour la sécurité**

En mettant la DB dans un réseau séparé, seuls les services explicitement connectés à ce réseau peuvent y accéder. Le frontend ne peut pas accéder directement à la DB.

</details>

---

**Question 5 : Comment voir les conteneurs dans un réseau ?**

- A) `docker network ps`
- B) `docker network ls`
- C) `docker network inspect nom-reseau`
- D) `docker network show nom-reseau`

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C : `docker network inspect nom-reseau`**

Cette commande affiche tous les détails du réseau, y compris la liste des conteneurs connectés.

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Les 3 types de réseaux

**1. Bridge (par défaut)**
- Réseau isolé virtuel
- Recommandé pour la plupart des cas

**2. Host**
- Partage le réseau de l'hôte
- Meilleure performance mais moins sécurisé

**3. None**
- Aucun réseau
- Isolation maximale

### Commandes essentielles

```bash
# Créer un réseau
docker network create mon-reseau

# Lister
docker network ls

# Inspecter
docker network inspect mon-reseau

# Connecter un conteneur
docker network connect mon-reseau conteneur

# Déconnecter
docker network disconnect mon-reseau conteneur

# Supprimer
docker network rm mon-reseau

# Nettoyer
docker network prune
```

### Architecture type

```yaml
networks:
  frontend:    # Public
  backend:     # Services internes
  database:    # Base de données

services:
  nginx:
    networks: [frontend, backend]
  
  api:
    networks: [backend, database]
  
  db:
    networks: [database]
```

### Règle d'or

**Un conteneur peut être dans plusieurs réseaux, mais choisissez-les stratégiquement pour l'isolation et la sécurité !**

---

## 🚀 Et maintenant ?

**BRAVO ! Vous maîtrisez les réseaux Docker !** 🎉

### Ce que vous savez maintenant :

- ✅ Les 3 types de réseaux Docker
- ✅ Créer des réseaux personnalisés
- ✅ Isoler les services (frontend/backend/database)
- ✅ Diagnostiquer les problèmes réseau
- ✅ Architectures multi-réseaux complexes

### Dans le prochain cours (Cours 10) :

**Docker en Production - CI/CD et Déploiement** 🚀

Vous apprendrez :
1. Préparer Docker pour la production
2. CI/CD avec Docker (GitHub Actions, GitLab CI)
3. Déployer sur des serveurs (VPS, cloud)
4. Monitoring et logs en production
5. Mises à jour sans interruption (zero-downtime)

**On passe au niveau professionnel !** 💼

---

## 📚 Aide-mémoire Réseaux

**Créer et utiliser un réseau personnalisé :**
```bash
# Créer
docker network create mon-projet

# Lancer des conteneurs dedans
docker run -d --name app1 --network mon-projet nginx
docker run -d --name app2 --network mon-projet redis

# app1 peut parler à app2 par nom
docker exec app1 ping app2  # ✅ Marche
```

**Multi-réseaux pour isolation :**
```bash
# 2 réseaux
docker network create frontend
docker network create backend

# API dans les 2 (passerelle)
docker run -d --name api --network backend mon-api
docker network connect frontend api

# Frontend seulement dans frontend
docker run -d --name web --network frontend nginx

# DB seulement dans backend
docker run -d --name db --network backend postgres
```

---

**🎓 Vous êtes prêt pour le Cours 10 (Production) !**

Super travail ! Vous comprenez maintenant comment orchestrer des architectures réseau complexes. Le prochain cours va vous montrer comment mettre tout ça en production. C'est parti ! 💪🚀
