# Cours 10 : Docker en Production 🚀

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous saurez :
- Préparer vos applications pour la production
- Configurer CI/CD avec Docker (GitHub Actions)
- Déployer sur un serveur (VPS, cloud)
- Monitoring et logs en production
- Mettre à jour sans interruption (zero-downtime)
- Les bonnes pratiques de production

**Durée : 40 minutes (lecture + pratique)**

---

## 📖 Partie 1 : Production vs Développement

### Les différences importantes

| Aspect | Développement | Production |
|--------|---------------|------------|
| **Volumes** | Bind mounts (code en direct) | Volumes nommés |
| **Logs** | Console | Fichiers + monitoring |
| **Secrets** | Fichier .env local | Variables sécurisées |
| **Ressources** | Illimitées | Limites CPU/RAM |
| **Redémarrage** | Manuel | Automatique (restart: always) |
| **Sécurité** | Utilisateur root ok | Utilisateur non-root |
| **Images** | :latest ok | Tags de version |
| **Build** | Sur le PC | CI/CD automatique |

### Exemple concret

**docker-compose.dev.yml (développement) :**
```yaml
version: '3.8'

services:
  api:
    build: ./api
    volumes:
      - ./api:/app          # Code en direct
    environment:
      NODE_ENV: development
      DEBUG: true
    ports:
      - "3000:3000"
```

**docker-compose.prod.yml (production) :**
```yaml
version: '3.8'

services:
  api:
    image: monregistry.com/mon-api:1.2.3  # Version fixe
    restart: always
    environment:
      NODE_ENV: production
    env_file:
      - .env.production     # Secrets externes
    deploy:
      resources:
        limits:
          cpus: '0.5'       # Limite CPU
          memory: 512M      # Limite RAM
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## 🔒 Partie 2 : Sécuriser pour la Production

### 1. Ne jamais utiliser root

**Mauvais (dangereux) ❌ :**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
# S'exécute en root !
```

**Bon (sécurisé) ✅ :**
```dockerfile
FROM node:18-alpine

# Créer un utilisateur
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# Passer à cet utilisateur
USER nodejs

EXPOSE 3000
CMD ["node", "app.js"]
```

---

### 2. Scanner les vulnérabilités

**Avec Docker Scout :**
```bash
docker scout cves mon-image:latest
```

**Avec Trivy :**
```bash
docker run aquasec/trivy image mon-image:latest
```

**Résultat :**
```
Total: 23 vulnerabilities
- CRITICAL: 3
- HIGH: 8
- MEDIUM: 12
```

**Actions :**
1. Mettre à jour l'image de base
2. Mettre à jour les dépendances
3. Re-scanner jusqu'à 0 CRITICAL

---

### 3. Secrets et variables d'environnement

**❌ JAMAIS faire ça :**
```dockerfile
ENV DATABASE_PASSWORD=mon_super_secret_123
```

**✅ Solutions sécurisées :**

**Option 1 : Variables au runtime**
```bash
docker run -e DATABASE_PASSWORD=$DB_PASS mon-app
```

**Option 2 : Docker Secrets (Swarm)**
```bash
echo "mon_secret" | docker secret create db_password -
```

**Option 3 : Fichier .env (non commité)**
```bash
# .env (dans .gitignore)
DATABASE_PASSWORD=secret

# Charger avec
docker run --env-file .env mon-app
```

---

### 4. Limiter les ressources

**Sans limites (danger) :**
Un conteneur peut consommer 100% CPU et toute la RAM !

**Avec limites :**
```yaml
services:
  api:
    image: mon-api
    deploy:
      resources:
        limits:
          cpus: '0.5'      # 50% d'un CPU
          memory: 512M     # Max 512 Mo RAM
        reservations:
          cpus: '0.25'     # Minimum garanti
          memory: 256M
```

**Ou avec docker run :**
```bash
docker run -d \
  --cpus="0.5" \
  --memory="512m" \
  mon-api
```

---

### 5. Healthchecks

**Pourquoi ?**
Savoir si le conteneur fonctionne vraiment, pas juste s'il est démarré.

**Dans le Dockerfile :**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

**Dans docker-compose.yml :**
```yaml
services:
  api:
    image: mon-api
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

**Créer une route /health dans votre app :**
```javascript
// Node.js
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});
```

---

## 🔄 Partie 3 : CI/CD avec GitHub Actions

### C'est quoi CI/CD ?

**CI (Continuous Integration) :**
- À chaque push sur Git
- Les tests s'exécutent automatiquement
- L'image Docker se construit automatiquement

**CD (Continuous Deployment) :**
- Si les tests passent
- L'image est poussée sur Docker Hub
- L'application se déploie automatiquement

**Schéma :**
```
Code Push → GitHub
     ↓
Tests automatiques
     ↓
Build Docker
     ↓
Push sur Docker Hub
     ↓
Déploiement auto sur serveur
```

---

### Projet Pratique : GitHub Actions

**Structure du projet :**
```
mon-projet/
├── .github/
│   └── workflows/
│       └── docker.yml    # ← Pipeline CI/CD
├── Dockerfile
├── app.js
└── package.json
```

**Créer .github/workflows/docker.yml :**
```yaml
name: Docker CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    
    steps:
    # 1. Récupérer le code
    - name: Checkout code
      uses: actions/checkout@v3
    
    # 2. Configuration Docker Buildx
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    # 3. Login Docker Hub
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    # 4. Build et Push
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          ${{ secrets.DOCKER_USERNAME }}/mon-app:latest
          ${{ secrets.DOCKER_USERNAME }}/mon-app:${{ github.sha }}
        cache-from: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/mon-app:buildcache
        cache-to: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/mon-app:buildcache,mode=max
```

**Configuration sur GitHub :**

1. Aller dans Settings → Secrets and variables → Actions
2. Ajouter les secrets :
   - `DOCKER_USERNAME` : Votre nom d'utilisateur Docker Hub
   - `DOCKER_PASSWORD` : Votre mot de passe Docker Hub

**Résultat :**
À chaque push sur main → Build automatique + Push sur Docker Hub ! 🎉

---

### Pipeline avec Tests

**Amélioration du workflow :**
```yaml
name: Docker CI/CD avec Tests

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Run tests
      run: |
        docker build -t test-image .
        docker run test-image npm test

  build-and-push:
    needs: test              # Attend que les tests passent
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/mon-app:${{ github.sha }}
```

**Si les tests échouent → Pas de build ! ✅**

---

## 🖥️ Partie 4 : Déployer sur un Serveur

### Option 1 : Déploiement manuel sur VPS

**Étape 1 : Se connecter au serveur**
```bash
ssh root@votre-serveur.com
```

**Étape 2 : Installer Docker**
```bash
# Sur Ubuntu
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

**Étape 3 : Créer docker-compose.yml sur le serveur**
```bash
mkdir /app
cd /app
nano docker-compose.yml
```

**Contenu :**
```yaml
version: '3.8'

services:
  app:
    image: votre-username/mon-app:latest
    restart: always
    ports:
      - "80:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://...
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G

  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

**Étape 4 : Créer .env**
```bash
nano .env
```

```
DB_PASSWORD=votre_super_secret
```

**Étape 5 : Lancer**
```bash
docker-compose up -d
```

**Étape 6 : Vérifier**
```bash
docker-compose ps
docker-compose logs -f
```

**Votre app est en ligne !** 🎉

---

### Option 2 : Déploiement automatique avec Webhook

**Script sur le serveur (/app/deploy.sh) :**
```bash
#!/bin/bash
cd /app
docker-compose pull
docker-compose up -d
docker image prune -f
```

**Le rendre exécutable :**
```bash
chmod +x /app/deploy.sh
```

**Ajouter au workflow GitHub (.github/workflows/docker.yml) :**
```yaml
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    
    steps:
    - name: Deploy to server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /app
          ./deploy.sh
```

**Ajouter les secrets GitHub :**
- `SERVER_HOST` : IP du serveur
- `SERVER_USER` : root
- `SSH_PRIVATE_KEY` : Votre clé SSH privée

**Résultat :**
Push sur GitHub → Tests → Build → Push Docker Hub → Déploiement automatique ! 🚀

---

## 📊 Partie 5 : Monitoring et Logs

### 1. Centraliser les logs

**Problème :**
Logs dispersés dans chaque conteneur.

**Solution : Utiliser un système de logs centralisé**

**docker-compose.yml avec logging :**
```yaml
services:
  app:
    image: mon-app
    logging:
      driver: "json-file"
      options:
        max-size: "10m"     # Taille max par fichier
        max-file: "3"       # Nombre de fichiers
```

**Voir les logs :**
```bash
docker-compose logs -f app
docker-compose logs --tail 100 app
```

---

### 2. Monitoring avec Portainer (Interface Web)

**Installer Portainer :**
```bash
docker volume create portainer_data

docker run -d \
  -p 9000:9000 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

**Accéder à l'interface :**
```
http://votre-serveur:9000
```

**Créez un compte admin.**

**Portainer vous permet de :**
- ✅ Voir tous les conteneurs
- ✅ Consulter les logs
- ✅ Voir l'utilisation CPU/RAM
- ✅ Redémarrer/arrêter des conteneurs
- ✅ Gérer les volumes et réseaux

**Tout depuis une interface web !** 🎨

---

### 3. Métriques avec cAdvisor + Prometheus + Grafana

**docker-compose.monitoring.yml :**
```yaml
version: '3.8'

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana-data:/var/lib/grafana

volumes:
  prometheus-data:
  grafana-data:
```

**Lancer :**
```bash
docker-compose -f docker-compose.monitoring.yml up -d
```

**Accéder à Grafana :**
```
http://votre-serveur:3000
Login : admin
Password : admin
```

**Vous aurez des graphiques magnifiques !** 📊

---

## 🔄 Partie 6 : Mise à Jour Zero-Downtime

### Le problème

**Mise à jour classique :**
```bash
docker-compose down    # ← Le site est COUPÉ
docker-compose pull
docker-compose up -d   # ← Le site revient
```

**Temps d'arrêt : 10-30 secondes** 😱

### Solution : Rolling Update avec Docker Swarm

**Initialiser Swarm :**
```bash
docker swarm init
```

**Créer un service :**
```bash
docker service create \
  --name mon-app \
  --replicas 3 \
  --update-delay 10s \
  --update-parallelism 1 \
  -p 80:3000 \
  mon-app:1.0.0
```

**Mettre à jour SANS interruption :**
```bash
docker service update --image mon-app:1.1.0 mon-app
```

**Ce qui se passe :**
```
1. Conteneur 1 → Arrêt → Nouveau conteneur 1 démarre
2. Attendre 10s
3. Conteneur 2 → Arrêt → Nouveau conteneur 2 démarre
4. Attendre 10s
5. Conteneur 3 → Arrêt → Nouveau conteneur 3 démarre
```

**Pendant tout ce temps, au moins 2 conteneurs fonctionnent !**

**Zéro interruption de service !** ✨

---

### Alternative simple : Blue-Green Deployment

**Principe :**
- Version actuelle (blue) sur le port 80
- Nouvelle version (green) sur le port 8080
- Tester la green
- Basculer le trafic de blue vers green

**docker-compose.yml :**
```yaml
services:
  app-blue:
    image: mon-app:1.0.0
    ports:
      - "80:3000"

  app-green:
    image: mon-app:1.1.0
    ports:
      - "8080:3000"
```

**Étapes :**
1. Lancer green
2. Tester sur le port 8080
3. Si ok, changer nginx pour router vers green
4. Arrêter blue

---

## ⚙️ Partie 7 : Configuration pour Production

### docker-compose.prod.yml complet

```yaml
version: '3.8'

services:
  # Application
  app:
    image: ${DOCKER_REGISTRY}/mon-app:${VERSION}
    restart: always
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      REDIS_URL: redis://redis:6379
    env_file:
      - .env.production
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # Base de données
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 2G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis pour le cache
  redis:
    image: redis:7-alpine
    restart: always
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

volumes:
  postgres-data:
  redis-data:

networks:
  default:
    driver: bridge
```

**Fichier .env.production :**
```bash
VERSION=1.2.3
DOCKER_REGISTRY=registry.example.com
DB_USER=produser
DB_PASSWORD=super_secure_password_123
DB_NAME=production_db
```

---

## 📝 Partie 8 : Checklist de Production

### Avant le déploiement

**Sécurité :**
- [ ] Images scannées (0 vulnérabilités CRITICAL)
- [ ] Pas d'exécution en root
- [ ] Secrets dans variables d'environnement (pas dans le code)
- [ ] Ports minimaux exposés
- [ ] HTTPS configuré (certificat SSL)

**Performance :**
- [ ] Images optimisées (Alpine, multi-stage)
- [ ] Limites CPU/RAM définies
- [ ] Healthchecks configurés
- [ ] Cache configuré (Redis)

**Monitoring :**
- [ ] Logs centralisés
- [ ] Système de monitoring (Portainer/Grafana)
- [ ] Alertes configurées
- [ ] Backups automatiques de la DB

**Configuration :**
- [ ] `restart: always` sur tous les services
- [ ] Tags de version spécifiques (pas :latest)
- [ ] Variables d'environnement validées
- [ ] docker-compose.prod.yml séparé du dev

**Tests :**
- [ ] Tests automatisés dans CI
- [ ] Test de charge effectué
- [ ] Plan de rollback préparé
- [ ] Documentation à jour

---

## 🚨 Partie 9 : Gestion des Incidents

### Que faire si l'app crash ?

**1. Voir les logs immédiatement :**
```bash
docker-compose logs --tail 100 app
```

**2. Vérifier l'état des conteneurs :**
```bash
docker-compose ps
```

**3. Redémarrer si nécessaire :**
```bash
docker-compose restart app
```

**4. Si le problème persiste, rollback :**
```bash
# Revenir à la version précédente
docker-compose down
# Modifier docker-compose.yml avec l'ancienne version
docker-compose up -d
```

---

### Sauvegardes automatiques

**Script de backup (backup.sh) :**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# Backup de la base de données
docker exec postgres pg_dump -U admin appdb > $BACKUP_DIR/db_$DATE.sql

# Backup des volumes
docker run --rm \
  -v app_postgres-data:/data \
  -v $BACKUP_DIR:/backup \
  alpine tar czf /backup/volumes_$DATE.tar.gz /data

# Garder seulement les 7 derniers backups
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

**Cron job (tous les jours à 2h du matin) :**
```bash
crontab -e
```

Ajouter :
```
0 2 * * * /app/backup.sh >> /var/log/backup.log 2>&1
```

---

## ✅ Quiz de validation

**Question 1 : Quelle différence principale entre dev et prod ?**

- A) Aucune différence
- B) En prod on utilise des tags de version, limites ressources, restart always
- C) En dev c'est plus rapide
- D) En prod on n'utilise pas Docker

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B**

Production nécessite : versions spécifiques (pas :latest), limites CPU/RAM, restart automatique, healthchecks, logs, monitoring, sécurité renforcée.

</details>

---

**Question 2 : Pourquoi ne jamais utiliser root en production ?**

- A) C'est interdit
- B) Sécurité : si le conteneur est compromis, l'attaquant a moins de droits
- C) Ça va plus vite
- D) C'est obligatoire pour Docker

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Sécurité**

Un utilisateur non-root limite les dégâts en cas de compromission du conteneur.

</details>

---

**Question 3 : C'est quoi CI/CD ?**

- A) Un nouveau langage de programmation
- B) Continuous Integration / Continuous Deployment (automatisation)
- C) Une base de données
- D) Un type de conteneur

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B**

CI/CD automatise les tests, builds et déploiements à chaque changement de code.

</details>

---

**Question 4 : Comment faire une mise à jour sans interruption ?**

- A) Avec docker-compose down puis up
- B) Avec Docker Swarm et rolling updates
- C) Ce n'est pas possible
- D) En arrêtant tous les conteneurs

<details>
<summary>Voir la réponse</summary>

✅ **Réponse B : Docker Swarm et rolling updates**

Swarm met à jour les conteneurs un par un, garantissant qu'au moins un conteneur fonctionne toujours.

</details>

---

**Question 5 : Où mettre les secrets en production ?**

- A) Dans le code source
- B) Dans le Dockerfile
- C) Dans des variables d'environnement sécurisées
- D) Sur GitHub

<details>
<summary>Voir la réponse</summary>

✅ **Réponse C**

Utilisez des variables d'environnement, Docker Secrets, ou des gestionnaires de secrets (Vault, AWS Secrets Manager).

</details>

---

## 🎯 Récapitulatif (À RETENIR)

### Production vs Dev

**Dev :** Code en direct, logs console, :latest ok  
**Prod :** Versions fixes, limites ressources, monitoring, sécurité

### Checklist Production

✅ Utilisateur non-root  
✅ Images scannées  
✅ Secrets sécurisés  
✅ Limites CPU/RAM  
✅ Healthchecks  
✅ restart: always  
✅ Logs centralisés  
✅ Monitoring (Portainer/Grafana)  
✅ Backups automatiques  
✅ CI/CD configuré  

### Pipeline CI/CD

```
Code Push → Tests → Build Docker → Push Registry → Deploy Serveur
```

### Mise à jour sans interruption

**Docker Swarm :**
```bash
docker service update --image mon-app:new mon-app
```

Rolling update automatique ! ✨

---

## 🚀 Et maintenant ?

**BRAVO ! Vous savez déployer Docker en production !** 🎉

### Ce que vous maîtrisez maintenant :

- ✅ Différences dev/production
- ✅ Sécuriser les applications
- ✅ CI/CD avec GitHub Actions
- ✅ Déploiement sur serveur
- ✅ Monitoring et logs
- ✅ Mises à jour zero-downtime

### Dans le prochain cours (Cours 11) :

**Debug et Troubleshooting** 🔧

Vous apprendrez :
1. Diagnostiquer tous les problèmes Docker
2. Commandes de debug avancées
3. Résoudre les erreurs courantes
4. Optimiser les performances
5. Guide de dépannage complet

**Le guide ultime quand ça ne marche pas !** 💪

---

## 📚 Templates Prêts à l'Emploi

**Workflow GitHub Actions :**
```yaml
name: Production Deploy

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Login Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/app:${{ github.ref_name }}
    
    - name: Deploy
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USER }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd /app
          docker-compose pull
          docker-compose up -d
```

---

**🎓 Vous êtes prêt pour le Cours 11 (Debug) !**

Excellent ! Vos applications sont maintenant production-ready. Le prochain cours va vous apprendre à résoudre TOUS les problèmes que vous pourriez rencontrer. Presque fini ! 💪🚀
