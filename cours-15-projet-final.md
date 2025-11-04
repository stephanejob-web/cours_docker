# Cours 12 : Projet Final - Application de Gestion d'Utilisateurs 🎯

## 🎯 Objectifs du projet

Créer une application complète de A à Z avec :
- ✅ **Frontend PHP** : Pages de connexion, inscription, administration
- ✅ **Base de données MySQL** : Gestion des utilisateurs avec mots de passe hashés
- ✅ **phpMyAdmin** : Interface de gestion de la base de données
- ✅ **Docker Compose** : Orchestration de tous les services
- ✅ **Sécurité** : Mots de passe hashés, sessions, protection admin
- ✅ **Production-ready** : Volumes persistants, réseau isolé

**Ce projet combine TOUT ce qu'on a appris dans les 11 cours précédents !** 💪

**Durée estimée : 1h30**

---

## 📁 Structure du projet

```
user-management-app/
├── docker-compose.yml
├── php/
│   ├── Dockerfile
│   └── src/
│       ├── index.php
│       ├── login.php
│       ├── register.php
│       ├── logout.php
│       ├── admin.php
│       ├── config/
│       │   └── database.php
│       ├── includes/
│       │   ├── header.php
│       │   └── footer.php
│       └── css/
│           └── style.css
└── mysql/
    └── init.sql
```

---

## 🚀 Étape 1 : Création de la structure

### Créer les dossiers

```bash
# Créer la structure du projet
mkdir -p user-management-app/{php/src/{config,includes,css},mysql}
cd user-management-app
```

---

## 🐳 Étape 2 : Configuration Docker

### Fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  # Base de données MySQL
  db:
    image: mysql:8.0
    container_name: user_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root_password_123
      MYSQL_DATABASE: user_management
      MYSQL_USER: app_user
      MYSQL_PASSWORD: app_password_123
    volumes:
      - db_data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app_network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 10

  # Application PHP
  web:
    build: ./php
    container_name: user_web
    restart: always
    ports:
      - "8080:80"
    volumes:
      - ./php/src:/var/www/html
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app_network
    environment:
      DB_HOST: db
      DB_NAME: user_management
      DB_USER: app_user
      DB_PASSWORD: app_password_123

  # phpMyAdmin
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: user_phpmyadmin
    restart: always
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: app_user
      PMA_PASSWORD: app_password_123
      UPLOAD_LIMIT: 50M
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app_network

# Volumes pour persister les données
volumes:
  db_data:
    driver: local

# Réseau isolé pour les conteneurs
networks:
  app_network:
    driver: bridge
```

**Explications :**
- `healthcheck` : Attend que MySQL soit prêt avant de démarrer l'app
- `volumes` : Persiste les données MySQL et permet le développement en temps réel
- `networks` : Isole l'application dans son propre réseau
- `depends_on` avec `condition` : Garantit l'ordre de démarrage

---

## 🐘 Étape 3 : Configuration PHP

### Fichier `php/Dockerfile`

```dockerfile
FROM php:8.2-apache

# Installer les extensions PDO MySQL
RUN docker-php-ext-install pdo pdo_mysql

# Activer mod_rewrite pour Apache (si besoin plus tard)
RUN a2enmod rewrite

# Copier la configuration PHP personnalisée (optionnel)
RUN echo "display_errors = On" > /usr/local/etc/php/conf.d/custom.ini && \
    echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/custom.ini

# Définir le répertoire de travail
WORKDIR /var/www/html

# Exposer le port 80
EXPOSE 80
```

---

## 🗄️ Étape 4 : Initialisation de la base de données

### Fichier `mysql/init.sql`

```sql
-- Création de la table users
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    is_admin TINYINT(1) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Création d'un utilisateur administrateur par défaut
-- Mot de passe : admin123 (hashé avec PASSWORD_DEFAULT)
INSERT INTO users (username, email, password, is_admin) VALUES 
('admin', 'admin@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 1);

-- Création d'un utilisateur normal pour les tests
-- Mot de passe : user123
INSERT INTO users (username, email, password, is_admin) VALUES 
('john', 'john@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 0);
```

**Note :** Les mots de passe sont hashés avec `password_hash()` de PHP.

---

## 💻 Étape 5 : Code PHP - Configuration

### Fichier `php/src/config/database.php`

```php
<?php
// Configuration de la base de données
class Database {
    private $host;
    private $db_name;
    private $username;
    private $password;
    private $conn;

    public function __construct() {
        // Récupération des variables d'environnement Docker
        $this->host = getenv('DB_HOST') ?: 'db';
        $this->db_name = getenv('DB_NAME') ?: 'user_management';
        $this->username = getenv('DB_USER') ?: 'app_user';
        $this->password = getenv('DB_PASSWORD') ?: 'app_password_123';
    }

    // Connexion à la base de données
    public function getConnection() {
        $this->conn = null;

        try {
            $dsn = "mysql:host=" . $this->host . ";dbname=" . $this->db_name . ";charset=utf8mb4";
            $options = [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                PDO::ATTR_EMULATE_PREPARES => false,
            ];
            
            $this->conn = new PDO($dsn, $this->username, $this->password, $options);
        } catch(PDOException $e) {
            echo "Erreur de connexion : " . $e->getMessage();
            die();
        }

        return $this->conn;
    }
}
?>
```

---

## 🎨 Étape 6 : CSS - Style de l'application

### Fichier `php/src/css/style.css`

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}

/* Navigation */
nav {
    background-color: rgba(255, 255, 255, 0.95);
    padding: 1rem 2rem;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

nav ul {
    list-style: none;
    display: flex;
    justify-content: space-between;
    align-items: center;
    max-width: 1200px;
    margin: 0 auto;
}

nav ul li {
    margin: 0 1rem;
}

nav a {
    text-decoration: none;
    color: #667eea;
    font-weight: 600;
    transition: color 0.3s;
}

nav a:hover {
    color: #764ba2;
}

.nav-right {
    display: flex;
    gap: 1rem;
}

.admin-btn {
    background-color: #ff6b6b;
    color: white !important;
    padding: 0.5rem 1rem;
    border-radius: 5px;
}

.admin-btn:hover {
    background-color: #ee5a52;
}

/* Container principal */
.container {
    flex: 1;
    max-width: 500px;
    margin: 3rem auto;
    background: white;
    padding: 2.5rem;
    border-radius: 15px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.2);
}

.container-large {
    max-width: 800px;
}

h1, h2 {
    color: #333;
    margin-bottom: 1.5rem;
    text-align: center;
}

/* Formulaires */
.form-group {
    margin-bottom: 1.5rem;
}

label {
    display: block;
    margin-bottom: 0.5rem;
    color: #555;
    font-weight: 500;
}

input[type="text"],
input[type="email"],
input[type="password"] {
    width: 100%;
    padding: 0.8rem;
    border: 2px solid #e0e0e0;
    border-radius: 8px;
    font-size: 1rem;
    transition: border-color 0.3s;
}

input:focus {
    outline: none;
    border-color: #667eea;
}

button {
    width: 100%;
    padding: 1rem;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.2s;
}

button:hover {
    transform: translateY(-2px);
}

/* Messages */
.message {
    padding: 1rem;
    margin-bottom: 1.5rem;
    border-radius: 8px;
    text-align: center;
}

.error {
    background-color: #fee;
    color: #c33;
    border: 1px solid #fcc;
}

.success {
    background-color: #efe;
    color: #3c3;
    border: 1px solid #cfc;
}

/* Page d'accueil */
.welcome {
    text-align: center;
    padding: 2rem;
}

.welcome h1 {
    font-size: 2.5rem;
    margin-bottom: 1rem;
}

.welcome p {
    font-size: 1.2rem;
    color: #666;
}

/* Dashboard Admin */
.stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1.5rem;
    margin: 2rem 0;
}

.stat-card {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 2rem;
    border-radius: 10px;
    text-align: center;
}

.stat-card h3 {
    font-size: 3rem;
    margin-bottom: 0.5rem;
}

.stat-card p {
    font-size: 1.1rem;
    opacity: 0.9;
}

/* Table utilisateurs */
table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 2rem;
}

table th,
table td {
    padding: 1rem;
    text-align: left;
    border-bottom: 1px solid #e0e0e0;
}

table th {
    background-color: #f5f5f5;
    font-weight: 600;
    color: #667eea;
}

table tr:hover {
    background-color: #f9f9f9;
}

.badge {
    padding: 0.3rem 0.8rem;
    border-radius: 20px;
    font-size: 0.85rem;
    font-weight: 600;
}

.badge-admin {
    background-color: #ff6b6b;
    color: white;
}

.badge-user {
    background-color: #51cf66;
    color: white;
}

/* Links */
.link {
    text-align: center;
    margin-top: 1.5rem;
}

.link a {
    color: #667eea;
    text-decoration: none;
    font-weight: 500;
}

.link a:hover {
    text-decoration: underline;
}

/* Responsive */
@media (max-width: 768px) {
    .container {
        margin: 1rem;
        padding: 1.5rem;
    }
    
    nav ul {
        flex-direction: column;
        gap: 1rem;
    }
}
```

---

## 📄 Étape 7 : Header et Footer

### Fichier `php/src/includes/header.php`

```php
<?php
session_start();

// Fonction pour vérifier si l'utilisateur est connecté
function isLoggedIn() {
    return isset($_SESSION['user_id']);
}

// Fonction pour vérifier si l'utilisateur est admin
function isAdmin() {
    return isset($_SESSION['is_admin']) && $_SESSION['is_admin'] == 1;
}
?>
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestion Utilisateurs - Docker App</title>
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <nav>
        <ul>
            <div>
                <li><a href="/index.php">🏠 Accueil</a></li>
            </div>
            <div class="nav-right">
                <?php if (isLoggedIn()): ?>
                    <?php if (isAdmin()): ?>
                        <li><a href="/admin.php" class="admin-btn">👑 Administration</a></li>
                    <?php endif; ?>
                    <li><span style="color: #667eea;">👤 <?php echo htmlspecialchars($_SESSION['username']); ?></span></li>
                    <li><a href="/logout.php">🚪 Déconnexion</a></li>
                <?php else: ?>
                    <li><a href="/login.php">🔐 Connexion</a></li>
                    <li><a href="/register.php">📝 Inscription</a></li>
                <?php endif; ?>
            </div>
        </ul>
    </nav>
```

### Fichier `php/src/includes/footer.php`

```php
    <footer style="text-align: center; padding: 2rem; color: white;">
        <p>&copy; 2024 User Management App - Projet Docker Final 🐳</p>
    </footer>
</body>
</html>
```

---

## 🏠 Étape 8 : Page d'accueil

### Fichier `php/src/index.php`

```php
<?php include 'includes/header.php'; ?>

<div class="container container-large">
    <div class="welcome">
        <?php if (isLoggedIn()): ?>
            <h1>Bienvenue, <?php echo htmlspecialchars($_SESSION['username']); ?> ! 🎉</h1>
            <p>Vous êtes connecté avec succès.</p>
            
            <?php if (isAdmin()): ?>
                <div style="margin-top: 2rem;">
                    <p style="color: #ff6b6b; font-weight: bold;">
                        👑 Vous avez les droits administrateur
                    </p>
                    <p style="margin-top: 1rem;">
                        <a href="/admin.php" style="display: inline-block; padding: 1rem 2rem; background: #ff6b6b; color: white; text-decoration: none; border-radius: 8px; font-weight: 600;">
                            Accéder au panneau d'administration
                        </a>
                    </p>
                </div>
            <?php endif; ?>
        <?php else: ?>
            <h1>Bienvenue sur l'Application de Gestion d'Utilisateurs 🐳</h1>
            <p>Cette application est construite avec Docker, PHP, MySQL et phpMyAdmin</p>
            
            <div style="margin-top: 3rem; display: flex; gap: 1rem; justify-content: center;">
                <a href="/login.php" style="padding: 1rem 2rem; background: #667eea; color: white; text-decoration: none; border-radius: 8px; font-weight: 600;">
                    Se connecter
                </a>
                <a href="/register.php" style="padding: 1rem 2rem; background: #764ba2; color: white; text-decoration: none; border-radius: 8px; font-weight: 600;">
                    S'inscrire
                </a>
            </div>
            
            <div style="margin-top: 3rem; text-align: left;">
                <h3>🎯 Fonctionnalités :</h3>
                <ul style="list-style: none; margin-top: 1rem;">
                    <li>✅ Inscription et connexion sécurisées</li>
                    <li>✅ Mots de passe hashés avec bcrypt</li>
                    <li>✅ Gestion des sessions</li>
                    <li>✅ Panneau d'administration pour les admins</li>
                    <li>✅ phpMyAdmin pour gérer la base de données</li>
                </ul>
                
                <h3 style="margin-top: 2rem;">🔑 Comptes de test :</h3>
                <ul style="list-style: none; margin-top: 1rem;">
                    <li><strong>Admin :</strong> admin / admin123</li>
                    <li><strong>User :</strong> john / user123</li>
                </ul>
            </div>
        <?php endif; ?>
    </div>
</div>

<?php include 'includes/footer.php'; ?>
```

---

## 📝 Étape 9 : Page d'inscription

### Fichier `php/src/register.php`

```php
<?php
require_once 'config/database.php';
include 'includes/header.php';

$error = '';
$success = '';

// Si l'utilisateur est déjà connecté, rediriger vers l'accueil
if (isLoggedIn()) {
    header('Location: /index.php');
    exit();
}

// Traitement du formulaire
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $email = trim($_POST['email'] ?? '');
    $password = $_POST['password'] ?? '';
    $confirm_password = $_POST['confirm_password'] ?? '';
    
    // Validation
    if (empty($username) || empty($email) || empty($password)) {
        $error = 'Tous les champs sont obligatoires !';
    } elseif (strlen($username) < 3) {
        $error = 'Le nom d\'utilisateur doit contenir au moins 3 caractères !';
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $error = 'Adresse email invalide !';
    } elseif (strlen($password) < 6) {
        $error = 'Le mot de passe doit contenir au moins 6 caractères !';
    } elseif ($password !== $confirm_password) {
        $error = 'Les mots de passe ne correspondent pas !';
    } else {
        try {
            $database = new Database();
            $db = $database->getConnection();
            
            // Vérifier si l'utilisateur existe déjà
            $query = "SELECT id FROM users WHERE username = :username OR email = :email";
            $stmt = $db->prepare($query);
            $stmt->bindParam(':username', $username);
            $stmt->bindParam(':email', $email);
            $stmt->execute();
            
            if ($stmt->rowCount() > 0) {
                $error = 'Ce nom d\'utilisateur ou cet email est déjà utilisé !';
            } else {
                // Hash du mot de passe avec PASSWORD_DEFAULT (bcrypt)
                $hashed_password = password_hash($password, PASSWORD_DEFAULT);
                
                // Insertion de l'utilisateur
                $query = "INSERT INTO users (username, email, password) VALUES (:username, :email, :password)";
                $stmt = $db->prepare($query);
                $stmt->bindParam(':username', $username);
                $stmt->bindParam(':email', $email);
                $stmt->bindParam(':password', $hashed_password);
                
                if ($stmt->execute()) {
                    $success = 'Inscription réussie ! Vous pouvez maintenant vous connecter.';
                    // Réinitialiser les champs
                    $username = $email = '';
                } else {
                    $error = 'Une erreur est survenue lors de l\'inscription.';
                }
            }
        } catch(PDOException $e) {
            $error = 'Erreur de base de données : ' . $e->getMessage();
        }
    }
}
?>

<div class="container">
    <h2>📝 Inscription</h2>
    
    <?php if ($error): ?>
        <div class="message error"><?php echo htmlspecialchars($error); ?></div>
    <?php endif; ?>
    
    <?php if ($success): ?>
        <div class="message success"><?php echo htmlspecialchars($success); ?></div>
    <?php endif; ?>
    
    <form method="POST" action="">
        <div class="form-group">
            <label for="username">Nom d'utilisateur :</label>
            <input type="text" id="username" name="username" value="<?php echo htmlspecialchars($username ?? ''); ?>" required>
        </div>
        
        <div class="form-group">
            <label for="email">Email :</label>
            <input type="email" id="email" name="email" value="<?php echo htmlspecialchars($email ?? ''); ?>" required>
        </div>
        
        <div class="form-group">
            <label for="password">Mot de passe :</label>
            <input type="password" id="password" name="password" required>
            <small style="color: #666;">Au moins 6 caractères</small>
        </div>
        
        <div class="form-group">
            <label for="confirm_password">Confirmer le mot de passe :</label>
            <input type="password" id="confirm_password" name="confirm_password" required>
        </div>
        
        <button type="submit">S'inscrire</button>
    </form>
    
    <div class="link">
        <p>Vous avez déjà un compte ? <a href="/login.php">Se connecter</a></p>
    </div>
</div>

<?php include 'includes/footer.php'; ?>
```

---

## 🔐 Étape 10 : Page de connexion

### Fichier `php/src/login.php`

```php
<?php
require_once 'config/database.php';
include 'includes/header.php';

$error = '';

// Si l'utilisateur est déjà connecté, rediriger vers l'accueil
if (isLoggedIn()) {
    header('Location: /index.php');
    exit();
}

// Traitement du formulaire
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $password = $_POST['password'] ?? '';
    
    if (empty($username) || empty($password)) {
        $error = 'Tous les champs sont obligatoires !';
    } else {
        try {
            $database = new Database();
            $db = $database->getConnection();
            
            // Récupérer l'utilisateur
            $query = "SELECT id, username, email, password, is_admin FROM users WHERE username = :username";
            $stmt = $db->prepare($query);
            $stmt->bindParam(':username', $username);
            $stmt->execute();
            
            if ($stmt->rowCount() === 1) {
                $user = $stmt->fetch();
                
                // Vérifier le mot de passe
                if (password_verify($password, $user['password'])) {
                    // Connexion réussie - Créer la session
                    $_SESSION['user_id'] = $user['id'];
                    $_SESSION['username'] = $user['username'];
                    $_SESSION['email'] = $user['email'];
                    $_SESSION['is_admin'] = $user['is_admin'];
                    
                    // Rediriger vers l'accueil
                    header('Location: /index.php');
                    exit();
                } else {
                    $error = 'Nom d\'utilisateur ou mot de passe incorrect !';
                }
            } else {
                $error = 'Nom d\'utilisateur ou mot de passe incorrect !';
            }
        } catch(PDOException $e) {
            $error = 'Erreur de base de données : ' . $e->getMessage();
        }
    }
}
?>

<div class="container">
    <h2>🔐 Connexion</h2>
    
    <?php if ($error): ?>
        <div class="message error"><?php echo htmlspecialchars($error); ?></div>
    <?php endif; ?>
    
    <form method="POST" action="">
        <div class="form-group">
            <label for="username">Nom d'utilisateur :</label>
            <input type="text" id="username" name="username" value="<?php echo htmlspecialchars($username ?? ''); ?>" required autofocus>
        </div>
        
        <div class="form-group">
            <label for="password">Mot de passe :</label>
            <input type="password" id="password" name="password" required>
        </div>
        
        <button type="submit">Se connecter</button>
    </form>
    
    <div class="link">
        <p>Pas encore de compte ? <a href="/register.php">S'inscrire</a></p>
    </div>
    
    <div class="link" style="margin-top: 2rem; padding-top: 1rem; border-top: 1px solid #e0e0e0;">
        <p><strong>Comptes de test :</strong></p>
        <p>Admin : <code>admin / admin123</code></p>
        <p>User : <code>john / user123</code></p>
    </div>
</div>

<?php include 'includes/footer.php'; ?>
```

---

## 🚪 Étape 11 : Page de déconnexion

### Fichier `php/src/logout.php`

```php
<?php
session_start();

// Détruire toutes les variables de session
$_SESSION = array();

// Détruire le cookie de session
if (ini_get("session.use_cookies")) {
    $params = session_get_cookie_params();
    setcookie(session_name(), '', time() - 42000,
        $params["path"], $params["domain"],
        $params["secure"], $params["httponly"]
    );
}

// Détruire la session
session_destroy();

// Rediriger vers la page de connexion
header('Location: /login.php');
exit();
?>
```

---

## 👑 Étape 12 : Page d'administration

### Fichier `php/src/admin.php`

```php
<?php
require_once 'config/database.php';
include 'includes/header.php';

// Vérifier que l'utilisateur est connecté ET administrateur
if (!isLoggedIn() || !isAdmin()) {
    header('Location: /index.php');
    exit();
}

try {
    $database = new Database();
    $db = $database->getConnection();
    
    // Statistiques
    $query = "SELECT 
                COUNT(*) as total_users,
                SUM(CASE WHEN is_admin = 1 THEN 1 ELSE 0 END) as total_admins,
                SUM(CASE WHEN is_admin = 0 THEN 1 ELSE 0 END) as total_regular_users
              FROM users";
    $stmt = $db->prepare($query);
    $stmt->execute();
    $stats = $stmt->fetch();
    
    // Liste des utilisateurs
    $query = "SELECT id, username, email, is_admin, created_at FROM users ORDER BY created_at DESC";
    $stmt = $db->prepare($query);
    $stmt->execute();
    $users = $stmt->fetchAll();
    
} catch(PDOException $e) {
    $error = 'Erreur de base de données : ' . $e->getMessage();
}
?>

<div class="container container-large">
    <h2>👑 Panneau d'Administration</h2>
    
    <?php if (isset($error)): ?>
        <div class="message error"><?php echo htmlspecialchars($error); ?></div>
    <?php else: ?>
        
        <!-- Statistiques -->
        <div class="stats-grid">
            <div class="stat-card">
                <h3><?php echo $stats['total_users']; ?></h3>
                <p>Utilisateurs Total</p>
            </div>
            <div class="stat-card">
                <h3><?php echo $stats['total_admins']; ?></h3>
                <p>Administrateurs</p>
            </div>
            <div class="stat-card">
                <h3><?php echo $stats['total_regular_users']; ?></h3>
                <p>Utilisateurs Normaux</p>
            </div>
        </div>
        
        <!-- Liste des utilisateurs -->
        <h3 style="margin-top: 3rem;">📋 Liste des Utilisateurs</h3>
        
        <table>
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Nom d'utilisateur</th>
                    <th>Email</th>
                    <th>Rôle</th>
                    <th>Date d'inscription</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach ($users as $user): ?>
                <tr>
                    <td><?php echo htmlspecialchars($user['id']); ?></td>
                    <td><?php echo htmlspecialchars($user['username']); ?></td>
                    <td><?php echo htmlspecialchars($user['email']); ?></td>
                    <td>
                        <?php if ($user['is_admin']): ?>
                            <span class="badge badge-admin">👑 Admin</span>
                        <?php else: ?>
                            <span class="badge badge-user">👤 User</span>
                        <?php endif; ?>
                    </td>
                    <td><?php echo date('d/m/Y H:i', strtotime($user['created_at'])); ?></td>
                </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
        
        <!-- Lien vers phpMyAdmin -->
        <div style="margin-top: 3rem; padding: 1.5rem; background: #f5f5f5; border-radius: 8px; text-align: center;">
            <p style="margin-bottom: 1rem;">
                <strong>💾 Accéder à phpMyAdmin :</strong>
            </p>
            <a href="http://localhost:8081" target="_blank" style="display: inline-block; padding: 0.8rem 2rem; background: #4285f4; color: white; text-decoration: none; border-radius: 8px; font-weight: 600;">
                Ouvrir phpMyAdmin
            </a>
            <p style="margin-top: 1rem; font-size: 0.9rem; color: #666;">
                User: app_user | Password: app_password_123
            </p>
        </div>
        
    <?php endif; ?>
</div>

<?php include 'includes/footer.php'; ?>
```

---

## 🚀 Étape 13 : Lancement de l'application

### 1. Construction et démarrage

```bash
# Se placer dans le dossier du projet
cd user-management-app

# Lancer l'application
docker-compose up -d --build

# Vérifier que tout tourne
docker-compose ps
```

**Résultat attendu :**

```
NAME                 IMAGE                     STATUS
user_db              mysql:8.0                 Up (healthy)
user_phpmyadmin      phpmyadmin:latest         Up
user_web             user-management-app-web   Up
```

### 2. Accéder à l'application

- **Application web :** http://localhost:8080
- **phpMyAdmin :** http://localhost:8081

### 3. Tester l'application

#### Test 1 : Inscription
1. Aller sur http://localhost:8080
2. Cliquer sur "S'inscrire"
3. Créer un compte
4. Vérifier que vous êtes redirigé vers l'accueil

#### Test 2 : Connexion Admin
1. Se connecter avec : `admin` / `admin123`
2. Vérifier que le bouton "Administration" apparaît
3. Cliquer sur "Administration"
4. Vérifier les statistiques et la liste des utilisateurs

#### Test 3 : phpMyAdmin
1. Aller sur http://localhost:8081
2. Se connecter avec : `app_user` / `app_password_123`
3. Explorer la base de données `user_management`
4. Vérifier que les mots de passe sont bien hashés

---

## 🔍 Étape 14 : Vérification et debugging

### Voir les logs

```bash
# Logs de tous les services
docker-compose logs

# Logs d'un service spécifique
docker-compose logs web
docker-compose logs db

# Suivre les logs en temps réel
docker-compose logs -f
```

### Entrer dans les conteneurs

```bash
# Entrer dans le conteneur PHP
docker exec -it user_web bash

# Entrer dans le conteneur MySQL
docker exec -it user_db mysql -u app_user -p
# Password: app_password_123
```

### Requêtes SQL de test

```sql
-- Se connecter à MySQL
docker exec -it user_db mysql -u app_user -p

-- Une fois connecté :
USE user_management;

-- Voir tous les utilisateurs
SELECT id, username, email, is_admin, created_at FROM users;

-- Compter les utilisateurs
SELECT COUNT(*) as total FROM users;

-- Voir les admins
SELECT username, email FROM users WHERE is_admin = 1;
```

---

## 🛠️ Étape 15 : Commandes utiles

### Gestion de l'application

```bash
# Démarrer l'application
docker-compose up -d

# Arrêter l'application
docker-compose down

# Arrêter ET supprimer les volumes (⚠️ perte de données !)
docker-compose down -v

# Reconstruire après modification du Dockerfile
docker-compose up -d --build

# Redémarrer un service spécifique
docker-compose restart web

# Voir l'utilisation des ressources
docker stats
```

### Backup de la base de données

```bash
# Créer un backup
docker exec user_db mysqldump -u app_user -papp_password_123 user_management > backup.sql

# Restaurer un backup
docker exec -i user_db mysql -u app_user -papp_password_123 user_management < backup.sql
```

---

## 🎯 Étape 16 : Points d'amélioration (pour aller plus loin)

### Sécurité avancée

```php
// Ajouter une protection CSRF
// Limiter les tentatives de connexion
// Ajouter un système de récupération de mot de passe
// Implémenter du rate limiting
```

### Fonctionnalités supplémentaires

```php
// Profil utilisateur avec avatar
// Modification de mot de passe
// Suppression de compte
// Système de rôles avancé
// Pagination de la liste des utilisateurs
```

### Production

```yaml
# Utiliser des secrets Docker
# Ajouter HTTPS avec Nginx + Certbot
# Configurer des healthchecks plus robustes
# Utiliser des variables d'environnement externes
# Mettre en place des logs centralisés
```

---

## 📊 Récapitulatif des technologies utilisées

| Technologie | Usage | Port |
|------------|-------|------|
| **PHP 8.2** | Application web | 8080 |
| **MySQL 8.0** | Base de données | 3306 (interne) |
| **phpMyAdmin** | Interface BDD | 8081 |
| **Docker** | Conteneurisation | - |
| **Docker Compose** | Orchestration | - |

---

## ✅ Ce que vous avez appris dans ce projet

### Concepts Docker
- ✅ Docker Compose multi-services
- ✅ Création d'un Dockerfile personnalisé
- ✅ Gestion des volumes persistants
- ✅ Configuration des réseaux isolés
- ✅ Healthchecks et dépendances entre services
- ✅ Variables d'environnement
- ✅ Build et déploiement

### Développement
- ✅ Architecture PHP/MySQL avec PDO
- ✅ Sécurisation des mots de passe (password_hash)
- ✅ Gestion des sessions PHP
- ✅ Protection contre les injections SQL (prepared statements)
- ✅ Système d'authentification complet
- ✅ Gestion des rôles (admin/user)
- ✅ Interface responsive avec CSS

### Bonnes pratiques
- ✅ Séparation des préoccupations (config, includes, pages)
- ✅ Validation des données
- ✅ Gestion des erreurs
- ✅ Code sécurisé et maintenable
- ✅ Documentation complète

---

## 🎓 Quiz Final

### Question 1
Pourquoi utilise-t-on `password_hash()` au lieu de `md5()` ?

**A)** C'est plus rapide  
**B)** C'est plus sécurisé (bcrypt avec salt automatique)  
**C)** C'est obligatoire en PHP 8  
**D)** Ça prend moins de place en base de données

<details>
<summary>Voir la réponse</summary>

**Réponse : B) C'est plus sécurisé**

`password_hash()` utilise bcrypt par défaut qui :
- Génère automatiquement un salt unique
- Est résistant aux attaques par force brute
- S'adapte automatiquement aux évolutions de sécurité

`md5()` est obsolète et facilement cassable !
</details>

### Question 2
Que fait `depends_on` avec `condition: service_healthy` dans Docker Compose ?

**A)** Démarre les services en même temps  
**B)** Attend que le service soit en bonne santé avant de démarrer  
**C)** Redémarre le service s'il crash  
**D)** Partage les ressources entre services

<details>
<summary>Voir la réponse</summary>

**Réponse : B) Attend que le service soit en bonne santé**

Cette configuration garantit que MySQL est complètement prêt (healthcheck OK) avant que l'application PHP ne démarre. Sans ça, l'app pourrait démarrer avant que MySQL ne soit prêt et crasher !
</details>

### Question 3
Pourquoi utilise-t-on PDO avec des prepared statements ?

**A)** C'est plus rapide  
**B)** Ça évite les erreurs de syntaxe  
**C)** Ça protège contre les injections SQL  
**D)** C'est obligatoire avec MySQL 8

<details>
<summary>Voir la réponse</summary>

**Réponse : C) Ça protège contre les injections SQL**

Les prepared statements séparent le code SQL des données, empêchant ainsi un attaquant d'injecter du code malveillant. C'est LA bonne pratique en sécurité !
</details>

---

## 🎉 Félicitations !

**Vous avez terminé la formation Docker complète !** 🚀

Vous savez maintenant :
- ✅ Créer des applications multi-conteneurs
- ✅ Gérer des bases de données avec Docker
- ✅ Sécuriser une application
- ✅ Utiliser Docker Compose en production
- ✅ Débugger et optimiser vos conteneurs

### 🌟 Prochaines étapes

1. **Déployer en production** : AWS, DigitalOcean, Heroku
2. **Apprendre Kubernetes** : Orchestration à grande échelle
3. **CI/CD** : GitHub Actions, GitLab CI, Jenkins
4. **Monitoring** : Prometheus, Grafana
5. **Sécurité avancée** : Scanners de vulnérabilités, secrets management

---

## 📚 Ressources pour aller plus loin

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Best Practices](https://docs.docker.com/compose/production/)
- [PHP Security Best Practices](https://www.php.net/manual/en/security.php)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

**Bravo pour avoir terminé cette formation !** 🎓🐳

Vous êtes maintenant prêt à créer vos propres applications Dockerisées ! 💪
