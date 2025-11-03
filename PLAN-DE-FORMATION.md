# 🎓 PLAN DE FORMATION DOCKER
## Formation complète pour débutants en reconversion

**Objectif :** Former des étudiants novices à Docker, du concept de base jusqu'au déploiement en production.

**Public cible :** Étudiants en reconversion professionnelle, aucune connaissance Docker requise.

**Durée totale estimée :** 20-25 heures

---

## 📚 STRUCTURE DE LA FORMATION

### 🔵 BLOC 1 : DÉCOUVERTE & FONDATIONS (4-5h)

**Objectif :** Comprendre pourquoi Docker existe et maîtriser les commandes de base

#### 📘 Cours 01 : Pourquoi Docker ?
**Durée :** 30 min
**Type :** Théorique
**Contenu :**
- Le problème avant Docker ("Ça marche sur mon PC !")
- Comment Docker résout ce problème
- Analogie simple : le conteneur = la boîte de déménagement
- Différence avec les machines virtuelles

**Prérequis :** Aucun
**Acquis :** Compréhension du concept

---

#### 📘 Cours 02 : Les Concepts de Base
**Durée :** 30 min
**Type :** Théorique
**Contenu :**
- C'est quoi une IMAGE Docker
- C'est quoi un CONTENEUR Docker
- La différence entre les deux (recette vs pizza cuite)
- Le Docker Hub (le "magasin" d'images)

**Prérequis :** Cours 01
**Acquis :** Vocabulaire et concepts de base

---

#### 📗 Cours 03 : Premières Commandes
**Durée :** 2h
**Type :** Pratique
**Contenu :**
- Ouvrir le terminal
- Vérifier Docker (`docker --version`)
- Hello World
- Commandes essentielles :
  - `docker pull` (télécharger)
  - `docker images` (voir les images)
  - `docker run` (lancer)
  - `docker ps` (voir ce qui tourne)
  - `docker start/stop/restart`
  - `docker logs`
  - `docker rm/rmi` (supprimer)

**Exercices :**
1. Lancer un serveur Nginx
2. Créer un fichier dans Ubuntu
3. Gérer 3 sites web simultanément

**Prérequis :** Cours 02
**Acquis :** Maîtrise des commandes de base

---

### 🟢 BLOC 2 : CONCEPTS ESSENTIELS (6-7h)

**Objectif :** Maîtriser les 4 concepts clés pour faire tourner des applications réelles

#### 📕 Cours 04 : Les Ports - Accéder à vos Conteneurs
**Durée :** 1h
**Type :** Théorique + Pratique
**Contenu :**
- Le problème : mon conteneur est invisible !
- Comprendre l'isolation réseau
- Le mapping de port `-p` expliqué en détail
- Port de l'hôte vs port du conteneur
- Exemples : `-p 8080:80`, `-p 3000:3000`

**Exercices :**
1. Lancer Nginx accessible sur `localhost:8080`
2. Lancer plusieurs Nginx sur des ports différents (8080, 8081, 8082)
3. Comprendre les conflits de ports

**Prérequis :** Cours 03
**Acquis :** Savoir exposer des services web

---

#### 📕 Cours 05 : Les Volumes - Persister vos Données
**Durée :** 1h30
**Type :** Théorique + Pratique
**Contenu :**
- Le GROS problème : les données disparaissent !
- Démonstration de perte de données
- Les 3 types de volumes :
  1. **Volumes nommés** (Docker gère)
  2. **Bind mounts** (vous gérez - développement)
  3. **tmpfs** (en RAM - temporaire)
- Quand utiliser chaque type
- Le super pouvoir : modification en temps réel du code

**Exercices :**
1. MySQL avec persistance (volume nommé)
2. Site web avec code modifiable en direct (bind mount)
3. Comparaison avec/sans volume

**Prérequis :** Cours 04
**Acquis :** Savoir sauvegarder des données et développer avec Docker

---

#### 📕 Cours 06 : Les Réseaux - Connecter vos Conteneurs
**Durée :** 1h30
**Type :** Théorique + Pratique
**Contenu :**
- Le problème : mes conteneurs ne se parlent pas !
- Pourquoi l'isolation par défaut ?
- Créer un réseau Docker
- Communication par nom de conteneur
- Les différents types de réseaux (bridge, host, none)

**Exercices :**
1. Créer un réseau
2. Connecter 2 conteneurs (frontend + backend)
3. Test de communication (ping, curl)

**Prérequis :** Cours 05
**Acquis :** Savoir faire communiquer des conteneurs

---

#### 📕 Cours 07 : Variables d'Environnement - Configurer vos Conteneurs
**Durée :** 1h
**Type :** Théorique + Pratique
**Contenu :**
- C'est quoi une variable d'environnement ?
- Pourquoi `-e` au lieu de modifier le code ?
- Exemples concrets :
  - Mot de passe MySQL
  - Mode debug
  - URL d'API
- Fichiers `.env` pour gérer plusieurs variables

**Exercices :**
1. MySQL avec mot de passe personnalisé
2. Application avec différentes configurations (dev, prod)
3. Comprendre les variables par défaut

**Prérequis :** Cours 06
**Acquis :** Savoir configurer des conteneurs

---

### 🟡 BLOC 3 : PROJET D'INTÉGRATION (3h)

**Objectif :** Combiner TOUS les concepts dans un vrai projet

#### 📙 Cours 08 : PROJET COMPLET - Application Web PHP + MariaDB
**Durée :** 2h30
**Type :** Projet guidé
**Contenu :**
- Rappel des 4 concepts clés :
  1. **Ports** → Accéder au site
  2. **Volumes** → Code modifiable en direct
  3. **Réseaux** → PHP parle à MariaDB
  4. **Variables d'env** → Configuration de la base

**Le projet :**
- Créer un dossier de projet
- Écrire le code PHP (fourni)
- Lancer MariaDB avec configuration
- Lancer PHP+Apache avec montage de volume
- Installer les extensions PHP
- Tester l'application
- Modifier le code en temps réel
- Explorer la base de données

**Ce qu'on apprend :**
- Workflow de développement réel
- Debugging
- Compréhension complète du flow de données

**Prérequis :** Cours 04 + 05 + 06 + 07 (TOUS les concepts !)
**Acquis :** Capacité à créer une application multi-conteneurs

---

### 🟣 BLOC 4 : CRÉER SES IMAGES (3h)

**Objectif :** Ne plus dépendre des images existantes, créer les siennes

#### 📗 Cours 09 : Le Dockerfile - Créer vos Propres Images
**Durée :** 2h
**Type :** Théorique + Pratique
**Contenu :**
- Le problème jusqu'ici : on utilise des images toutes faites
- C'est quoi un Dockerfile ?
- Les 10 instructions essentielles :
  - `FROM` - Image de base
  - `WORKDIR` - Dossier de travail
  - `COPY` - Copier des fichiers
  - `RUN` - Exécuter une commande
  - `ENV` - Variables d'environnement
  - `EXPOSE` - Documenter un port
  - `CMD` - Commande par défaut
  - `ENTRYPOINT` - Point d'entrée
  - Et plus...
- Construire une image (`docker build`)
- Bonnes pratiques (layers, cache, .dockerignore)

**Exercices :**
1. Application Node.js simple
2. API Python Flask
3. Optimisation d'une image (taille)

**Prérequis :** Cours 08
**Acquis :** Savoir créer des images personnalisées

---

### 🔴 BLOC 5 : ORCHESTRATION (3h)

**Objectif :** Gérer facilement des applications multi-conteneurs

#### 📘 Cours 10 : Docker Compose - L'Orchestrateur Magique
**Durée :** 2h
**Type :** Théorique + Pratique
**Contenu :**
- Le problème : taper 10 commandes c'est pénible !
- Docker Compose = tout en UN fichier YAML
- Structure d'un `docker-compose.yml`
- Les commandes Compose :
  - `docker-compose up` - Tout lancer
  - `docker-compose down` - Tout arrêter
  - `docker-compose logs` - Voir tous les logs
  - `docker-compose ps` - Voir tous les services
- Gestion des dépendances entre services

**Exercices :**
1. Refaire le projet PHP+MariaDB en Compose (1 fichier !)
2. Application full-stack (React + Node + PostgreSQL)
3. Ajouter des services (Redis, monitoring)

**Prérequis :** Cours 09
**Acquis :** Savoir orchestrer des applications complexes

---

### ⚫ BLOC 6 : PRODUCTION & PROFESSIONNALISATION (6-8h)

**Objectif :** Passer du développement à la production

#### 📕 Cours 11 : Optimisation - Images Légères et Performantes
**Durée :** 1h30
**Type :** Théorique + Pratique
**Contenu :**
- Multi-stage builds
- Alpine Linux (images minimales)
- Réduire la taille des images
- Accélérer les builds (cache)
- Sécurité des images

**Prérequis :** Cours 09
**Acquis :** Créer des images optimisées

---

#### 📕 Cours 12 : Réseaux Avancés
**Durée :** 1h30
**Type :** Théorique + Pratique
**Contenu :**
- Types de réseaux (bridge, host, overlay, macvlan)
- DNS interne Docker
- Load balancing basique
- Reverse proxy (Nginx, Traefik)

**Prérequis :** Cours 06
**Acquis :** Maîtriser les réseaux Docker

---

#### 📕 Cours 13 : Production - Déployer pour de Vrai
**Durée :** 2h
**Type :** Théorique + Pratique
**Contenu :**
- Différences dev vs prod
- Health checks
- Restart policies
- Logs en production
- Secrets et configurations sensibles
- CI/CD basique (GitHub Actions)

**Prérequis :** Cours 10 + 11
**Acquis :** Savoir déployer en production

---

#### 📕 Cours 14 : Debug & Troubleshooting
**Durée :** 1h30
**Type :** Pratique
**Contenu :**
- Les erreurs les plus fréquentes
- Comment lire les logs
- Inspecter un conteneur
- Débugger un réseau
- Résoudre les problèmes de performance

**Prérequis :** Tous les cours
**Acquis :** Autonomie face aux problèmes

---

#### 📙 Cours 15 : PROJET FINAL - Application Complète
**Durée :** 3h
**Type :** Projet autonome
**Contenu :**
- Application e-commerce complète :
  - Frontend (React)
  - Backend API (Node.js)
  - Base de données (PostgreSQL)
  - Cache (Redis)
  - Reverse proxy (Nginx)
  - Monitoring (Prometheus + Grafana)

**Travail demandé :**
- Dockerfile pour chaque service
- docker-compose.yml complet
- Documentation
- Scripts de déploiement

**Prérequis :** TOUS les cours
**Acquis :** Portfolio professionnel + Certification de formation

---

## 🎯 PROGRESSION PÉDAGOGIQUE

```
SEMAINE 1 : Découverte
├── Jour 1 : Cours 01 + 02 (Théorie)
├── Jour 2 : Cours 03 (Pratique commandes)
└── Jour 3 : Révisions + Exercices

SEMAINE 2 : Concepts essentiels
├── Jour 1 : Cours 04 (Ports)
├── Jour 2 : Cours 05 (Volumes)
├── Jour 3 : Cours 06 (Réseaux)
├── Jour 4 : Cours 07 (Variables d'env)
└── Jour 5 : Révisions + Exercices

SEMAINE 3 : Projet & Images
├── Jour 1-2 : Cours 08 (Projet complet PHP)
└── Jour 3-4 : Cours 09 (Dockerfile)

SEMAINE 4 : Orchestration & Production
├── Jour 1-2 : Cours 10 (Docker Compose)
├── Jour 3 : Cours 11 + 12 (Optimisation + Réseaux avancés)
└── Jour 4-5 : Cours 13 + 14 (Production + Debug)

SEMAINE 5 : Projet final
├── Jour 1-4 : Cours 15 (Projet final)
└── Jour 5 : Présentation + Certification
```

---

## ✅ CRITÈRES DE RÉUSSITE

**À la fin de la formation, l'étudiant est capable de :**

✅ Expliquer pourquoi Docker est utilisé
✅ Différencier image et conteneur
✅ Utiliser toutes les commandes Docker de base
✅ Exposer des services web avec les ports
✅ Persister des données avec les volumes
✅ Faire communiquer des conteneurs avec les réseaux
✅ Configurer des conteneurs avec des variables d'environnement
✅ Créer des Dockerfiles optimisés
✅ Orchestrer avec Docker Compose
✅ Déployer une application complète en production
✅ Débugger et résoudre des problèmes

---

## 🎓 ÉVALUATION

**Contrôle continu (40%):**
- Quiz à la fin de chaque cours (10%)
- Exercices pratiques (30%)

**Projet PHP+MariaDB (20%):**
- Application fonctionnelle
- Code propre et commenté

**Projet final (40%):**
- Application complète multi-services (25%)
- Documentation technique (10%)
- Présentation orale (5%)

**Note minimale pour valider :** 12/20

---

## 📚 RESSOURCES COMPLÉMENTAIRES

**Documentation officielle :**
- [Docker Docs](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)

**Pratique :**
- [Play with Docker](https://labs.play-with-docker.com/) - Environnement gratuit en ligne
- [Docker Samples](https://github.com/dockersamples) - Exemples officiels

**Communauté :**
- [Docker Community Forums](https://forums.docker.com/)
- [Stack Overflow - Tag Docker](https://stackoverflow.com/questions/tagged/docker)

---

## 🚀 ET APRÈS ?

**Prochaines étapes :**
- Kubernetes (orchestration à grande échelle)
- CI/CD avancé (Jenkins, GitLab CI)
- Docker Swarm (alternative à Kubernetes)
- Sécurité Docker (scanning, hardening)
- Microservices architecture

---

**Version :** 2.0 - Restructurée pour débutants
**Date :** Novembre 2025
**Auteur :** Formation Docker pour reconversion professionnelle
**Contact formateur :** [À compléter]

---

## 📋 NOTES POUR LE FORMATEUR

**Points d'attention :**
- Toujours expliquer POURQUOI avant de montrer COMMENT
- Utiliser des analogies du quotidien (pizza, maison, déménagement)
- Faire des pauses régulières (toutes les 45min)
- Encourager les questions ("Il n'y a pas de question bête !")
- Faire pratiquer, pratiquer, pratiquer !

**Matériel nécessaire :**
- PC avec Docker installé pour chaque étudiant
- Accès Internet (pour télécharger les images)
- Projecteur/écran pour les démos
- Tableau blanc pour les schémas

**Difficultés fréquentes :**
- Confusion image vs conteneur → Répéter l'analogie recette/pizza
- Oubli d'être dans le bon dossier avec `-v $(pwd)` → Insister sur `pwd` avant !
- Ports déjà utilisés → Expliquer `docker ps` et `docker stop`
- Peur du terminal → Rassurer, montrer que c'est juste des phrases en anglais
