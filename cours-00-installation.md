# Cours 0 : Installation de Docker sur Ubuntu 🐳

## 🎯 Ce que vous allez apprendre

À la fin de ce cours, vous aurez :
- ✅ Docker Engine installé et fonctionnel
- ✅ Docker Compose installé
- ✅ Les permissions correctement configurées
- ✅ Vérifié que tout fonctionne
- ✅ Résolu les problèmes courants

**Durée : 30 minutes**

---

## 📋 Prérequis

### Système requis

- **OS** : Ubuntu 20.04 LTS ou supérieur (64 bits)
- **RAM** : 4 Go minimum (8 Go recommandé)
- **Espace disque** : 10 Go libres minimum
- **Droits** : Accès sudo (administrateur)
- **Internet** : Connexion active

### Vérifier votre version Ubuntu

```bash
# Afficher la version d'Ubuntu
lsb_release -a
```

**Résultat attendu :**
```
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.x LTS
Release:        22.04
Codename:       jammy
```

---

## 🗑️ Étape 1 : Désinstaller les anciennes versions (si existantes)

**Pourquoi ?** Pour éviter les conflits avec d'anciennes installations.

```bash
# Supprimer les anciennes versions de Docker
sudo apt-get remove docker docker-engine docker.io containerd runc

# Note : c'est normal si cette commande dit qu'aucun paquet n'est installé
```

**Résultat attendu :**
```
Lecture des listes de paquets... Fait
...
0 mis à jour, 0 nouvellement installés, 0 à enlever...
```

---

## 📦 Étape 2 : Mettre à jour le système

```bash
# Mettre à jour la liste des paquets
sudo apt-get update

# Mettre à jour les paquets installés (optionnel mais recommandé)
sudo apt-get upgrade -y
```

**Temps estimé :** 2-5 minutes selon votre connexion

---

## 🔑 Étape 3 : Installer les prérequis

```bash
# Installer les paquets nécessaires
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

**Ce que font ces paquets :**
- `ca-certificates` : Certificats SSL pour télécharger en sécurité
- `curl` : Outil pour télécharger des fichiers
- `gnupg` : Gestion des clés de sécurité
- `lsb-release` : Informations sur votre système

---

## 🔐 Étape 4 : Ajouter la clé GPG officielle de Docker

```bash
# Créer le dossier pour les clés
sudo install -m 0755 -d /etc/apt/keyrings

# Télécharger la clé GPG de Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Définir les bonnes permissions
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

**Pourquoi ?** Pour vérifier que les paquets Docker viennent bien du site officiel.

---

## 📚 Étape 5 : Ajouter le dépôt Docker

```bash
# Ajouter le dépôt officiel Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Cette commande :**
- Détecte automatiquement votre architecture (amd64, arm64...)
- Configure le dépôt Docker pour votre version Ubuntu

---

## 🐳 Étape 6 : Installer Docker Engine

```bash
# Mettre à jour la liste des paquets (avec le nouveau dépôt)
sudo apt-get update

# Installer Docker Engine, CLI et containerd
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Ce qui est installé :**
- `docker-ce` : Docker Engine (moteur principal)
- `docker-ce-cli` : Interface en ligne de commande
- `containerd.io` : Runtime des conteneurs
- `docker-buildx-plugin` : Builder avancé
- `docker-compose-plugin` : Docker Compose v2

**Temps estimé :** 2-3 minutes

---

## ✅ Étape 7 : Vérifier l'installation

```bash
# Vérifier la version de Docker
sudo docker --version
```

**Résultat attendu :**
```
Docker version 24.0.x, build xxxxx
```

```bash
# Vérifier que le service Docker fonctionne
sudo systemctl status docker
```

**Résultat attendu :**
```
● docker.service - Docker Application Container Engine
   Loaded: loaded
   Active: active (running) since ...
```

Appuyez sur `q` pour quitter.

---

## 🎉 Étape 8 : Test avec Hello World

```bash
# Lancer le conteneur de test
sudo docker run hello-world
```

**Résultat attendu :**
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

**Si vous voyez ce message, BRAVO ! Docker fonctionne ! 🎉**

---

## 👤 Étape 9 : Configurer les permissions (IMPORTANT)

**Le problème :** Pour l'instant, vous devez taper `sudo` avant chaque commande Docker.

**La solution :** Ajouter votre utilisateur au groupe `docker`.

```bash
# Ajouter votre utilisateur au groupe docker
sudo usermod -aG docker $USER
```

**Important :** Pour que ce changement prenne effet, vous devez :

**Option 1 : Se déconnecter/reconnecter**
```bash
# Fermer la session et se reconnecter
# Ou redémarrer l'ordinateur
```

**Option 2 : Activer dans le terminal actuel (temporaire)**
```bash
# Activer le nouveau groupe dans le terminal actuel
newgrp docker
```

### Vérifier que ça marche sans sudo

```bash
# Tester sans sudo (après déconnexion/reconnexion)
docker run hello-world
```

**Si ça marche sans `sudo`, c'est parfait ! ✅**

---

## 🔧 Étape 10 : Installer Docker Compose (standalone - optionnel)

**Note :** Docker Compose v2 est déjà installé comme plugin (`docker compose`).

Si vous voulez aussi la commande `docker-compose` (v1 style) :

```bash
# Télécharger Docker Compose standalone
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Rendre exécutable
sudo chmod +x /usr/local/bin/docker-compose

# Vérifier la version
docker-compose --version
```

**Quelle version utiliser ?**
- `docker compose` (plugin v2) ✅ Recommandé, moderne
- `docker-compose` (standalone v1) ⚠️ Ancienne version

**Dans ce cours, nous utiliserons `docker compose` (v2).**

---

## 🎯 Étape 11 : Tests de validation complète

### Test 1 : Vérifier les versions

```bash
# Version Docker
docker --version

# Version Docker Compose
docker compose version
```

### Test 2 : Lancer Nginx

```bash
# Lancer un serveur web Nginx
docker run -d -p 8080:80 --name test-nginx nginx

# Vérifier qu'il tourne
docker ps
```

**Ouvrir dans le navigateur :** http://localhost:8080

**Vous devez voir :** La page "Welcome to nginx!"

### Test 3 : Nettoyer

```bash
# Arrêter et supprimer le conteneur de test
docker stop test-nginx
docker rm test-nginx

# Supprimer l'image hello-world
docker rmi hello-world nginx
```

---

## ⚙️ Étape 12 : Configuration optionnelle (recommandé)

### Démarrage automatique de Docker

```bash
# Activer le démarrage automatique de Docker au boot
sudo systemctl enable docker

# Vérifier
sudo systemctl is-enabled docker
```

**Résultat attendu :** `enabled`

### Limiter l'utilisation des ressources (optionnel)

Créer le fichier `/etc/docker/daemon.json` :

```bash
sudo nano /etc/docker/daemon.json
```

Ajouter :

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Sauvegarder : `Ctrl+O`, `Entrée`, `Ctrl+X`

Redémarrer Docker :

```bash
sudo systemctl restart docker
```

---

## ⚠️ Troubleshooting : Problèmes fréquents

### Problème 1 : "permission denied while trying to connect"

**Erreur :**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution :**
```bash
# Vérifier que vous êtes dans le groupe docker
groups

# Si "docker" n'apparaît pas :
sudo usermod -aG docker $USER

# Puis SE DÉCONNECTER et SE RECONNECTER
```

---

### Problème 2 : "Cannot connect to the Docker daemon"

**Erreur :**
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```

**Solution :**
```bash
# Vérifier si Docker tourne
sudo systemctl status docker

# Si "inactive (dead)" :
sudo systemctl start docker

# Activer au démarrage
sudo systemctl enable docker
```

---

### Problème 3 : "docker: command not found"

**Erreur :**
```
bash: docker: command not found
```

**Solution :**
```bash
# Vérifier l'installation
which docker

# Si rien ne s'affiche, Docker n'est pas installé
# Recommencer depuis l'étape 6
```

---

### Problème 4 : Port 8080 déjà utilisé

**Erreur :**
```
Error starting userland proxy: listen tcp 0.0.0.0:8080: bind: address already in use
```

**Solution :**
```bash
# Utiliser un autre port
docker run -d -p 8081:80 --name test-nginx nginx

# Ou trouver quel processus utilise le port 8080
sudo lsof -i :8080
```

---

### Problème 5 : Pas assez d'espace disque

**Erreur :**
```
no space left on device
```

**Solution :**
```bash
# Nettoyer les images, conteneurs et volumes non utilisés
docker system prune -a --volumes

# Attention : cela supprime TOUT ce qui n'est pas utilisé !
```

---

### Problème 6 : Docker trop lent

**Symptômes :** Téléchargement très lent, conteneurs qui mettent du temps à démarrer

**Solutions :**
```bash
# 1. Vérifier l'espace disque
df -h

# 2. Vérifier la RAM
free -h

# 3. Redémarrer Docker
sudo systemctl restart docker

# 4. Nettoyer le cache
docker system prune
```

---

## 📊 Commandes de vérification finale

Copiez-collez ce bloc complet pour tout vérifier d'un coup :

```bash
echo "=== Vérification de l'installation Docker ==="
echo ""
echo "1. Version Docker:"
docker --version
echo ""
echo "2. Version Docker Compose:"
docker compose version
echo ""
echo "3. Informations Docker:"
docker info | grep -E "Server Version|Operating System|Total Memory"
echo ""
echo "4. Services Docker:"
sudo systemctl is-active docker
echo ""
echo "5. Permissions (vous devez voir 'docker' dans la liste):"
groups | grep docker && echo "✅ Groupe docker OK" || echo "❌ Groupe docker manquant"
echo ""
echo "=== Fin de la vérification ==="
```

---

## ✅ Checklist de validation

Cochez chaque point avant de passer au cours suivant :

- [ ] `docker --version` affiche une version
- [ ] `docker compose version` affiche une version
- [ ] `docker run hello-world` fonctionne SANS `sudo`
- [ ] `docker ps` fonctionne SANS `sudo`
- [ ] Vous avez lancé et accédé à Nginx sur http://localhost:8080
- [ ] Docker démarre automatiquement au boot du système
- [ ] Vous avez nettoyé les conteneurs de test

**Si tous les points sont cochés : BRAVO ! Vous êtes prêt ! 🎉**

---

## 🎓 Récapitulatif

### Ce que vous avez installé

- ✅ **Docker Engine** : Le moteur principal qui fait tourner les conteneurs
- ✅ **Docker CLI** : L'interface en ligne de commande
- ✅ **Docker Compose** : Pour gérer des applications multi-conteneurs
- ✅ **containerd** : Le runtime qui gère les conteneurs
- ✅ **BuildKit** : Pour construire des images optimisées

### Les commandes à retenir

```bash
# Vérifier l'installation
docker --version
docker compose version

# Lancer un conteneur
docker run [image]

# Voir les conteneurs en cours
docker ps

# Arrêter un conteneur
docker stop [nom]

# Nettoyer
docker system prune
```

---

## 🚀 Et maintenant ?

**Félicitations ! Docker est installé et configuré ! 🎉**

Vous êtes maintenant prêt pour le **Cours 1 : Pourquoi Docker ?**

### Avant de continuer

Prenez 5 minutes pour :
1. Redémarrer votre ordinateur (pour finaliser les permissions)
2. Ouvrir un nouveau terminal
3. Taper `docker run hello-world` pour confirmer que tout marche

---

## 📚 Ressources supplémentaires

### Documentation officielle

- [Installation Ubuntu - Docker Docs](https://docs.docker.com/engine/install/ubuntu/)
- [Post-installation - Docker Docs](https://docs.docker.com/engine/install/linux-postinstall/)

### Commandes utiles

```bash
# Désinstaller Docker complètement
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

# Réinstaller depuis zéro
# Recommencer depuis l'étape 1
```

---

## 💡 Conseils pour la suite

1. **Ne jamais utiliser sudo** : Si vous devez utiliser `sudo`, c'est que les permissions ne sont pas bien configurées
2. **Nettoyer régulièrement** : `docker system prune` pour libérer de l'espace
3. **Vérifier l'espace disque** : Docker peut vite prendre beaucoup de place
4. **Lire les messages d'erreur** : Ils sont souvent très explicites

---

## ❓ Besoin d'aide ?

Si vous rencontrez un problème non couvert ici :

1. Vérifiez la section Troubleshooting ci-dessus
2. Consultez le **cours-14-debug-troubleshooting.md**
3. Relisez les messages d'erreur (ils contiennent souvent la solution)
4. Demandez de l'aide au formateur

---

**🎯 Prochaine étape : Cours 1 - Pourquoi Docker ?**

**Rappel :** N'oubliez pas de redémarrer votre session ou ordinateur pour que les permissions prennent effet !

Bon courage pour la suite de votre apprentissage ! 💪
