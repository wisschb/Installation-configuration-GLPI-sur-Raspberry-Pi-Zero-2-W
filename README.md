📌 Présentation

Ce projet consiste à installer et configurer GLPI 11 sur un Raspberry Pi Zero 2 W afin de mettre en place une solution de gestion de parc informatique et d'inventaire.

Le projet comprend :

Installation de GLPI 11
Configuration du serveur web Apache
Installation et configuration de PHP
Installation de MariaDB
Création de la base de données GLPI
Configuration de l'accès à l'interface web
Préparation de l'inventaire avec GLPI Inventory et GLPI Agent
1. Préparation du Raspberry Pi

Connexion au Raspberry Pi en SSH :

ssh pi@192.168.1.50

Mise à jour du système :

sudo apt update
sudo apt full-upgrade -y

Redémarrage :

sudo reboot
2. Installation d'Apache

Installation du serveur web Apache :

sudo apt install -y apache2

Activation du service :

sudo systemctl enable apache2

Démarrage :

sudo systemctl start apache2

Vérification :

sudo systemctl status apache2
3. Installation de PHP

Installation de PHP et des extensions nécessaires à GLPI :

sudo apt install -y \
php \
libapache2-mod-php \
php-cli \
php-mysql \
php-curl \
php-gd \
php-intl \
php-mbstring \
php-xml \
php-zip \
php-bcmath \
php-apcu \
php-bz2 \
php-ldap \
php-soap

Un fichier test.php a ensuite été créé afin de vérifier que PHP fonctionnait correctement avec Apache.

Exemple :

<?php
phpinfo();
?>

Le fichier a permis de vérifier que PHP était correctement exécuté par Apache.

4. Installation de MariaDB

Installation de MariaDB :

sudo apt install -y mariadb-server

Sécurisation de MariaDB :

sudo mariadb-secure-installation
5. Création de la base de données GLPI

Connexion à MariaDB :

sudo mariadb

Création de la base de données :

CREATE DATABASE glpi CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

Création de l'utilisateur :

CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'VOTRE_MOT_DE_PASSE';

Attribution des droits :

GRANT ALL PRIVILEGES ON glpi.* TO 'glpiuser'@'localhost';

Application des modifications :

FLUSH PRIVILEGES;

Puis sortie de MariaDB :

EXIT;

Le mot de passe utilisé pour la base de données ne doit pas être publié sur GitHub.

6. Téléchargement de GLPI

La version utilisée pour ce projet est GLPI 11.0.11.

Téléchargement :

cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/11.0.11/glpi-11.0.11.tgz

Vérification de l'archive :

tar -xOzf glpi-11.0.11.tgz glpi/public/index.php | wc -c

Le fichier index.php a bien été trouvé dans l'archive.

7. Extraction de GLPI

Lors de la première extraction, une erreur est apparue :

No space left on device

Le problème ne venait pas de l'espace disponible sur le Raspberry Pi mais de l'espace limité du dossier /tmp.

Vérification :

df -h /tmp

Le dossier /tmp utilisait un espace limité à environ 208 Mo.

L'extraction a donc été réalisée directement dans /var/www :

sudo rm -rf /tmp/glpi
sudo tar -xzf /tmp/glpi-11.0.11.tgz -C /var/www

Vérification :

ls -lh /var/www/glpi/public/index.php

Puis :

wc -c /var/www/glpi/public/index.php
8. Configuration des droits

Les fichiers GLPI ont été attribués à Apache :

sudo chown -R www-data:www-data /var/www/glpi
9. Configuration d'Apache pour GLPI

Création du fichier de configuration :

sudo nano /etc/apache2/sites-available/glpi.conf

Configuration utilisée :

<VirtualHost *:80>
    ServerName 192.168.1.50

    DocumentRoot /var/www/glpi/public

    <Directory /var/www/glpi/public>
        Require all granted
        AllowOverride All

        RewriteEngine On

        RewriteCond %{HTTP:Authorization} ^(.+)$
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%1]

        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
</VirtualHost>
10. Activation de la configuration Apache

Activation du module Rewrite :

sudo a2enmod rewrite

Désactivation du site Apache par défaut :

sudo a2dissite 000-default.conf

Activation du site GLPI :

sudo a2ensite glpi.conf

Vérification de la configuration :

sudo apache2ctl configtest

Résultat attendu :

Syntax OK

Redémarrage d'Apache :

sudo systemctl restart apache2

Vérification :

sudo systemctl status apache2 --no-pager
11. Installation de GLPI depuis l'interface web

L'interface GLPI est accessible depuis un ordinateur du réseau local :

http://192.168.1.50

Pendant l'installation, les informations de la base de données sont renseignées :

Serveur SQL : localhost
Utilisateur SQL : glpiuser
Mot de passe SQL : VOTRE_MOT_DE_PASSE
Base de données : glpi

GLPI peut ensuite créer les tables nécessaires dans la base de données.

12. Connexion à GLPI

Une fois l'installation terminée, l'interface de connexion permet d'accéder à GLPI.

Le compte administrateur créé par défaut lors de l'installation peut être utilisé pour accéder à l'administration.

Après la première connexion, il est recommandé de modifier le mot de passe administrateur.

13. Inventaire informatique

Le projet prévoit l'utilisation de :

GLPI Inventory

GLPI Inventory permet de gérer l'inventaire des équipements informatiques depuis GLPI.

GLPI Agent

Le GLPI Agent permet de récupérer automatiquement des informations sur les ordinateurs et de les transmettre au serveur GLPI.

L'objectif est notamment de récupérer des informations comme :

Nom du poste
Système d'exploitation
Processeur
Mémoire RAM
Stockage
Logiciels installés
Informations matérielles
14. Architecture du projet
                  ┌─────────────────────┐
                  │     Ordinateur      │
                  │       client        │
                  │    GLPI Agent       │
                  └──────────┬──────────┘
                             │
                             │ Inventaire
                             ▼
┌─────────────────────────────────────────────┐
│          Raspberry Pi Zero 2 W              │
│                                             │
│                 GLPI 11                     │
│                                             │
│  ┌────────────┐  ┌──────────┐  ┌────────┐ │
│  │   Apache   │  │   PHP    │  │MariaDB │ │
│  └────────────┘  └──────────┘  └────────┘ │
│                                             │
└─────────────────────────────────────────────┘
