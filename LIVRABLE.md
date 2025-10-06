# Livrable Docker - Partie 1

## 1. Fichiers Dockerfile
- [php/Dockerfile](php/Dockerfile) - Container Apache/PHP 8.0 avec extensions PDO
- [database/Dockerfile](database/Dockerfile) - Container MySQL 8.4 avec initialisation de la base

## 2. Commandes pour construire les images
```bash
# Build de l'image PHP
docker build -t alexandreduro/gestion-produits-php:v1.0 ./php

# Build de l'image MySQL
docker build -t alexandreduro/gestion-produits-mysql:v1.0 ./database
```

## 3. Commandes pour publier les images
```bash
# Connexion à Docker Hub
docker login

# Push des images
docker push alexandreduro/gestion-produits-php:v1.0
docker push alexandreduro/gestion-produits-mysql:v1.0
```

## 4. URL des images Docker publiées
- **Image PHP** : https://hub.docker.com/r/alexandreduro/gestion-produits-php
  - Tag: `alexandreduro/gestion-produits-php:v1.0`
- **Image MySQL** : https://hub.docker.com/r/alexandreduro/gestion-produits-mysql
  - Tag: `alexandreduro/gestion-produits-mysql:v1.0`

## 5. Fichier docker-compose.yml
✅ [docker-compose.yml](docker-compose.yml) inclus avec :
- Paramétrage des ports via variables d'environnement (`.env`)
- Gestion de la version de l'application via `APP_VERSION`
- Volume persistant pour les données MySQL (`mysql-gestion-data`)
- Network bridge pour la communication inter-containers

### Démarrage de l'application
```bash
docker compose up -d
```

### Configuration (fichier .env)
```env
APP_VERSION=v1.0      # Version des images à utiliser
PHP_PORT=8081         # Port externe pour l'application PHP
MYSQL_PORT=3306       # Port externe pour MySQL
MYSQL_PASSWORD=root   # Mot de passe root MySQL
```