# Cours 0 : Installation de Docker - Tous Systèmes 🐳

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous aurez :
- ✅ Docker Desktop ou Docker Engine installé sur votre système
- ✅ Docker Compose installé et fonctionnel
- ✅ Vérifié que tout fonctionne
- ✅ Résolu les problèmes courants spécifiques à votre OS

**Durée : 30-45 minutes selon votre système**

---

## 🖥️ Choisissez votre système d'exploitation

Cliquez sur votre système pour accéder aux instructions :

- **[🐧 Linux (Ubuntu/Debian)](#linux-ubuntu--debian)**
- **[🍎 macOS](#macos)**
- **[🪟 Windows](#windows)**

---

# 🐧 Linux (Ubuntu / Debian)

## 📋 Prérequis

- **OS** : Ubuntu 20.04 LTS ou supérieur (64 bits)
- **RAM** : 4 Go minimum (8 Go recommandé)
- **Espace disque** : 10 Go libres minimum
- **Droits** : Accès sudo (administrateur)
- **Internet** : Connexion active

### Vérifier votre version Ubuntu

```bash
lsb_release -a
```

---

## 🗑️ Étape 1 : Désinstaller les anciennes versions

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

---

## 📦 Étape 2 : Mettre à jour le système

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

---

## 🔑 Étape 3 : Installer les prérequis

```bash
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

---

## 🔐 Étape 4 : Ajouter la clé GPG officielle

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

---

## 📚 Étape 5 : Ajouter le dépôt Docker

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 🐳 Étape 6 : Installer Docker Engine

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 👤 Étape 7 : Configurer les permissions

```bash
# Ajouter votre utilisateur au groupe docker
sudo usermod -aG docker $USER

# Se déconnecter et se reconnecter OU utiliser :
newgrp docker
```

---

## ✅ Étape 8 : Vérifier l'installation

```bash
docker --version
docker compose version
docker run hello-world
```

**Si "Hello from Docker!" s'affiche, BRAVO ! ✅**

---

# 🍎 macOS

## 📋 Prérequis

- **macOS** : 11 (Big Sur) ou supérieur
- **Processeur** : Intel ou Apple Silicon (M1/M2/M3)
- **RAM** : 4 Go minimum (8 Go recommandé)
- **Espace disque** : 10 Go libres minimum

---

## 📥 Étape 1 : Télécharger Docker Desktop

### Option 1 : Via le site officiel (recommandé)

1. Aller sur : https://www.docker.com/products/docker-desktop
2. Cliquer sur **"Download for Mac"**
3. Choisir :
   - **Mac with Intel chip** si vous avez un Mac Intel
   - **Mac with Apple chip** si vous avez un Mac M1/M2/M3

### Option 2 : Via Homebrew

```bash
# Installer Homebrew si pas déjà fait
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Installer Docker Desktop
brew install --cask docker
```

---

## 🔧 Étape 2 : Installer Docker Desktop

1. **Ouvrir le fichier téléchargé** : `Docker.dmg`
2. **Glisser Docker** dans le dossier Applications
3. **Ouvrir Docker** depuis Applications
4. **Autoriser l'accès** :
   - macOS va demander votre mot de passe administrateur
   - Cliquer sur **"OK"** pour autoriser

---

## 🚀 Étape 3 : Première configuration

1. **Docker démarre** - Vous verrez l'icône de baleine dans la barre de menu
2. **Accepter les conditions d'utilisation**
3. **Configurer les ressources** (optionnel) :
   - Cliquer sur l'icône Docker → **Settings**
   - **Resources** → Ajuster RAM et CPU si nécessaire
   - Recommandé : 4 Go RAM, 2 CPUs minimum

---

## ✅ Étape 4 : Vérifier l'installation

Ouvrir le **Terminal** et taper :

```bash
docker --version
docker compose version
docker run hello-world
```

**Résultat attendu :**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

**Si ça marche, BRAVO ! ✅**

---

## 🔧 Paramètres recommandés pour macOS

### Augmenter les ressources (si vous avez 16 Go+ RAM)

1. Cliquer sur l'icône Docker (baleine) → **Settings**
2. **Resources** :
   - **CPUs** : 4
   - **Memory** : 8 GB
   - **Swap** : 2 GB
   - **Disk image size** : 60 GB

3. Cliquer sur **Apply & Restart**

---

## ⚠️ Problèmes courants macOS

### Problème 1 : "Docker Desktop requires a newer version of macOS"

**Solution :**
- Mettre à jour macOS vers la version 11 minimum
- Ou installer Docker Toolbox (ancienne version)

---

### Problème 2 : Docker démarre lentement sur Mac M1/M2/M3

**Solution :**
```bash
# Vérifier que vous avez bien la version Apple Silicon
docker version | grep -i arch

# Devrait afficher : arm64
```

Si ça affiche "amd64", vous avez téléchargé la mauvaise version !

---

### Problème 3 : "Cannot connect to Docker daemon"

**Solution :**
1. Vérifier que Docker Desktop est bien lancé (icône baleine en haut)
2. Redémarrer Docker Desktop
3. Si ça ne marche pas : Désinstaller et réinstaller

---

# 🪟 Windows

## 📋 Prérequis

- **Windows** : Windows 10 version 2004+ ou Windows 11
- **RAM** : 4 Go minimum (8 Go recommandé)
- **Virtualisation** : Activée dans le BIOS
- **WSL 2** : Windows Subsystem for Linux v2 (sera installé automatiquement)

---

## 🔍 Étape 1 : Vérifier la version de Windows

1. Appuyer sur **Windows + R**
2. Taper : `winver`
3. Vérifier :
   - **Windows 10** : Version 2004 (Build 19041) ou supérieur
   - **Windows 11** : Toutes les versions

---

## ⚙️ Étape 2 : Activer la virtualisation

### Vérifier si c'est déjà activé

1. Ouvrir le **Gestionnaire des tâches** (Ctrl+Shift+Esc)
2. Onglet **Performance** → **CPU**
3. Vérifier "Virtualisation" : doit afficher **Activé**

### Si "Désactivé" : Activer dans le BIOS

1. **Redémarrer le PC**
2. Appuyer sur **F2** ou **Del** ou **F10** (selon votre PC) au démarrage
3. Chercher :
   - Intel : **Intel VT-x** ou **Intel Virtualization Technology**
   - AMD : **AMD-V** ou **SVM Mode**
4. **Activer** l'option
5. **Sauvegarder et quitter** (F10)

---

## 📥 Étape 3 : Télécharger Docker Desktop

1. Aller sur : https://www.docker.com/products/docker-desktop
2. Cliquer sur **"Download for Windows"**
3. Télécharger **Docker Desktop Installer.exe**

---

## 🔧 Étape 4 : Installer Docker Desktop

1. **Double-cliquer** sur `Docker Desktop Installer.exe`
2. **Cocher** : "Use WSL 2 instead of Hyper-V" (recommandé)
3. **Cocher** : "Add shortcut to desktop"
4. Cliquer sur **"OK"**
5. **Attendre** l'installation (5-10 minutes)
6. Cliquer sur **"Close and restart"**

**⚠️ IMPORTANT : Le PC va redémarrer !**

---

## 🚀 Étape 5 : Configuration initiale

### Après le redémarrage

1. **Docker Desktop se lance automatiquement**
2. Un message peut apparaître : "WSL 2 installation is incomplete"
   - Si oui, cliquer sur le lien et suivre les instructions
   - Télécharger et installer : **WSL2 Linux kernel update package**
   - Redémarrer Docker Desktop

3. **Accepter les conditions d'utilisation**

4. **Créer un compte Docker Hub** (optionnel)
   - Vous pouvez cliquer sur "Skip" si vous voulez

---

## ✅ Étape 6 : Vérifier l'installation

### Ouvrir PowerShell ou CMD

1. Appuyer sur **Windows + R**
2. Taper : `powershell` ou `cmd`
3. Appuyer sur **Entrée**

### Tester Docker

```powershell
docker --version
docker compose version
docker run hello-world
```

**Résultat attendu :**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

**Si ça marche, BRAVO ! ✅**

---

## 🔧 Configuration recommandée Windows

### Paramètres Docker Desktop

1. **Cliquer sur l'icône Docker** (dans la barre des tâches)
2. **Settings** → **Resources**
3. **WSL Integration** :
   - Activer **"Enable integration with my default WSL distro"**
   - Si vous avez Ubuntu dans WSL : Activer l'intégration

4. **Resources** → **Advanced** :
   - **CPUs** : 2-4 (selon votre PC)
   - **Memory** : 4-8 GB
   - **Swap** : 1 GB

5. Cliquer sur **Apply & Restart**

---

## ⚠️ Problèmes courants Windows

### Problème 1 : "WSL 2 installation is incomplete"

**Solution :**
```powershell
# Ouvrir PowerShell en administrateur (clic droit → "Exécuter en tant qu'administrateur")

# Activer WSL
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# Activer la plateforme de machine virtuelle
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Redémarrer le PC
Restart-Computer
```

Puis télécharger et installer : https://aka.ms/wsl2kernel

---

### Problème 2 : "Hardware assisted virtualization is not enabled"

**Solution :**
- La virtualisation n'est pas activée dans le BIOS
- Suivre les instructions de l'Étape 2 ci-dessus

---

### Problème 3 : Docker démarre lentement ou freeze

**Solution :**
1. **Réduire les ressources** dans Settings → Resources
2. **Désactiver les antivirus** temporairement
3. **Nettoyer Docker** :
   ```powershell
   docker system prune -a
   ```

---

### Problème 4 : "This version of Docker Desktop requires Windows 10 version 2004"

**Solution :**
- Mettre à jour Windows 10 vers la version 2004 minimum
- Windows Update → Rechercher les mises à jour

---

### Problème 5 : Docker fonctionne dans WSL mais pas dans PowerShell

**Solution :**
1. Docker Desktop → Settings
2. **General** → Cocher "Use the WSL 2 based engine"
3. **Resources** → **WSL Integration** → Activer pour votre distribution
4. Redémarrer Docker Desktop

---

# 🧪 Tests de Validation (Tous Systèmes)

## Test 1 : Vérifier les versions

```bash
# Version Docker
docker --version

# Version Docker Compose
docker compose version

# Informations système
docker info
```

---

## Test 2 : Lancer un serveur web Nginx

```bash
# Lancer Nginx
docker run -d -p 8080:80 --name test-nginx nginx

# Vérifier qu'il tourne
docker ps
```

**Ouvrir dans le navigateur :** http://localhost:8080

**Vous devez voir :** "Welcome to nginx!"

---

## Test 3 : Tester Docker Compose

Créer un fichier `test-compose.yml` :

```yaml
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

Lancer :

```bash
docker compose -f test-compose.yml up -d
```

Ouvrir : http://localhost:8080

---

## Test 4 : Nettoyer

```bash
# Arrêter et supprimer
docker stop test-nginx
docker rm test-nginx

# Nettoyer tout
docker system prune -a
```

---

# ✅ Checklist de Validation Finale

Cochez chaque point avant de passer au cours suivant :

- [ ] `docker --version` affiche une version (20.x ou 24.x)
- [ ] `docker compose version` affiche une version (2.x)
- [ ] `docker run hello-world` fonctionne
- [ ] Vous avez lancé Nginx et accédé à http://localhost:8080
- [ ] **Linux uniquement** : Docker fonctionne SANS `sudo`
- [ ] **Windows uniquement** : WSL 2 est installé et activé
- [ ] **macOS uniquement** : Docker Desktop démarre automatiquement
- [ ] Vous avez nettoyé les conteneurs de test

**Si tous les points sont cochés : BRAVO ! Vous êtes prêt ! 🎉**

---

# 🔧 Commandes Utiles (Tous Systèmes)

## Vérification de l'installation

```bash
# Afficher toutes les infos Docker
docker info

# Vérifier l'espace disque utilisé
docker system df

# Voir les conteneurs en cours
docker ps

# Voir TOUS les conteneurs (même arrêtés)
docker ps -a
```

---

## Gestion de Docker Desktop (macOS & Windows)

### macOS
- **Démarrer** : Ouvrir "Docker" depuis Applications
- **Arrêter** : Clic sur icône Docker → Quit Docker Desktop
- **Redémarrer** : Clic sur icône Docker → Restart

### Windows
- **Démarrer** : Chercher "Docker Desktop" dans le menu Démarrer
- **Arrêter** : Clic droit sur icône Docker → Quit Docker Desktop
- **Redémarrer** : Clic droit sur icône Docker → Restart

---

## Gestion de Docker Engine (Linux)

```bash
# Démarrer Docker
sudo systemctl start docker

# Arrêter Docker
sudo systemctl stop docker

# Redémarrer Docker
sudo systemctl restart docker

# Statut de Docker
sudo systemctl status docker

# Activer au démarrage
sudo systemctl enable docker
```

---

## Nettoyage

```bash
# Supprimer les conteneurs arrêtés
docker container prune

# Supprimer les images non utilisées
docker image prune -a

# Supprimer les volumes non utilisés
docker volume prune

# TOUT nettoyer (⚠️ Attention !)
docker system prune -a --volumes
```

---

# 📊 Tableau Récapitulatif

| Critère | Linux | macOS | Windows |
|---------|-------|-------|---------|
| **Installation** | Docker Engine | Docker Desktop | Docker Desktop |
| **Commande** | `docker` | `docker` | `docker` |
| **Compose v2** | Plugin | Intégré | Intégré |
| **Interface graphique** | Non | Oui | Oui |
| **Démarrage auto** | systemctl | Oui | Oui |
| **WSL requis** | Non | Non | Oui (WSL 2) |
| **Performances** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

# 🆘 Besoin d'Aide ?

## Documentation officielle

- **Linux** : https://docs.docker.com/engine/install/ubuntu/
- **macOS** : https://docs.docker.com/desktop/install/mac-install/
- **Windows** : https://docs.docker.com/desktop/install/windows-install/

## Communauté

- **Forums Docker** : https://forums.docker.com/
- **Stack Overflow** : https://stackoverflow.com/questions/tagged/docker
- **Reddit** : r/docker

---

# 🚀 Et Maintenant ?

**Félicitations ! Docker est installé ! 🎉**

### Avant de continuer

1. **Redémarrer votre ordinateur** (pour finaliser l'installation)
2. **Ouvrir un nouveau terminal**
3. **Taper** `docker run hello-world` pour confirmer

### Prochaine étape

**➡️ Cours 1 : Pourquoi Docker ?**

Vous allez découvrir :
- Pourquoi Docker a été créé
- Quel problème il résout
- Les cas d'usage concrets

---

## 💡 Conseils avant de commencer

1. **Pas de sudo sur Linux** : Si vous devez utiliser `sudo`, c'est que les permissions ne sont pas bien configurées
2. **Vérifier l'espace disque** : Docker peut prendre beaucoup de place (nettoyer avec `docker system prune`)
3. **Internet requis** : Pour télécharger les images Docker
4. **Patience** : Les premiers téléchargements peuvent être longs

---

## 🎯 Objectif du prochain cours

Dans le **Cours 1**, vous comprendrez :
- Le problème "Ça marche sur mon PC !"
- Comment Docker résout ce problème
- La différence entre VM et conteneur
- Pourquoi 70% des développeurs utilisent Docker

**Bon courage pour la suite ! 💪**

---

**Version :** 1.0 - Installation Multi-OS
**Systèmes couverts :** Linux (Ubuntu/Debian), macOS (Intel & Apple Silicon), Windows 10/11
**Dernière mise à jour :** Novembre 2025
