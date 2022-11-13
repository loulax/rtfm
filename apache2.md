# 					APACHE 2



## LAMP

### INSTALLATION

### CONFIGURATION

CONFIG HTACCESS



Afin de permettre l'authentification à une page web ou un serveur web apache2 il est possible de configurer certains paramètres pour exiger l'authentification d'un utilisateur à celui-ci. 

Pour ce faire il faudra taper la commande suivante : 

```bash
# Rentrer le chemin d'accès au fichier .htpasswd 
$ htpasswd -c /var/www/html/private/.htpasswd <user>
# Editer la configuration apache2 pour l'accès restreint
$ vim /etc/apache2/sites-available/000-default.conf
# Ajouter les lignes suivantes à l'intérieur de virtualhost
# Au niveau du Directory c'est là qu'il faut spécifier quel chemin nous souhaitons restreindre
<Directory "/var/www/html">
      AuthType Basic
      AuthName "Restricted Content"
      AuthUserFile /etc/apache2/.htpasswd
      Require valid-user
  </Directory
# Redémarrer le service apache2 pour prendre effet
$ systemctl restart apache2
```

INSTALLATION LAMP

Apache 2 est un service web très répandu qui permet de fournir des pages web html/php etc.

Ce dernier est développé notamment en Perl et C. Il implémente beaucoup de modules afin de le rendre très flexible pour s'adapter aux besoin de chacun. Il tourne sur tous les systèmes d'exploitations (les plus populaires du moins). Voici la procédure pour installer LAMP (Linux Apache, MariaDB, PHP).

Linux : Le système d'exploitation sur lequel apache est le plus répandue

MariaDB: SGBD (Système de Gestion de Base de Donnée), autrement dit le moteur qui gère les bases de données MySQL, PostgreSQL...

PHP : Langage de développement web (Back-End)

Voici comment installer tout-ceci:

```bash
#Installation apache
$ sudo apt -y install apache2
# Installation php
$ sudo apt -y install php php-mysql libapache2-mod-php 
# Installation mariadb
$ sudo apt -y install mariadb-server mariadb-client
```

