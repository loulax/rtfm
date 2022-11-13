## SPIP



## Installation serveur

Avoir accès à une console ssh sur le serveur avec les accès root

```bash
# Mettre à jour le serveur
apt update && apt -y full-upgrade
# Installation de lamp et des dépendances
apt -y install wget apache2 libapache2-mod-php php php-mysql mariadb-server mariadb-client libsodium23 php-common php-cli php-common php-json php-opcache php-readline php-pdo php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath
# Création du répertoire du site spip et paramétrage des permissions pour www-data
mkdir spip
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html
cd /var/www/html/spip
# Récupération du fichier d'installation de spip
wget https://get.spip.net/spip_loader.php
# Création de la base de donnée et de l'administrateur de la base
mysql
> CREATE DATABASE spipdb;
> GRANT ALL PRIVILEGES ON spipdb.* TO 'spipusr'@'localhost' IDENTIFIED BY 'spipasswd';
> FLUSH PRIVILEGES;
> quit;

```

Maintenant que cette partie est configuré, il va falloir passer à la configuration web.

## Configuration web

Rendez-vous sur le navigateur à l'adresse : http://<ip>/spip/spip_loader.php

 Nous arrivons sur la page d'installation de SPIP.  Il est possible de choisir la version à installer, par défaut, je vais prendre la dernière version disponible.

![](img\conf-web-1.png)

Son téléchargement va se poursuivre.

![conf-web-2](img\conf-web-2.png)

Une fois le téléchargement terminé, l'étape installation démarre. Choisir la langue d'affichage puis faire suivant.

![conf-web-3](img\conf-web-3.png)

Paramétrage de l'accès à la base de donnée mysql. Il faudra renseigner ici l'utilisateur qui est administrateur de la base, je vais utiliser root par défaut. Et l'adresse correspond à l'ip du serveur sur lequel est installé la base. Localhost, si c'est une installation locale.

![conf-web-4](img\conf-web-4.png)

Il faudra ici renseigner la base de donnée et l'utilisateur de la base spip créé précédemment.

![conf-web-5](img\conf-web-5.png)

Ici se trouve les informations qui permettront de se connecter à l'interface web admin de spip. Renseignez au choix puis faire suivant.

![conf-web-6](img\conf-web-6.png)

L'installation est terminé et la configuration semble correcte. Nous pouvons maintenant cliquer sur espace privé pour accéder à l'interface principale pour se connecter.

![conf-web-7](img\conf-web-7.png)

Ce message indique simplement qu'aucun fichier .htaccess n'est configuré sur le site ce qui peut le rendre vulnérable.

![](img\conf-web-8.png)

Nous arrivons désormais sur l'interface de connexion à l'interface web admin. Pour y accéder par la suite se rendre à l'adresse http://<ip>/spip/spip.php?page=login&url=%2Fspip%2Fecrire%2F

![conf-web-9](img\conf-web-9.png)

