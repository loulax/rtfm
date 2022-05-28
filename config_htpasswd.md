CONFIG HTACCESS



Afin de permettre l'authentification à une page web ou un serveur web apache2 il est possible de configurer certains paramètres pour exiger l'authentification d'un utilisateur à celui-ci. 

Pour ce faire il faudra taper la commande suivante : 

```bash
# Rentrer le chemin d'accès au fichier .htpasswd 
htpasswd -c /var/www/html/private/.htpasswd <user>
# Editer la configuration apache2 pour l'accès restreint
vim /etc/apache2/sites-available/000-default.conf
# Ajouter les lignes suivantes à l'intérieur de virtualhost
# Au niveau du Directory c'est là qu'il faut spécifier quel chemin nous souhaitons restreindre
<Directory "/var/www/html">
      AuthType Basic
      AuthName "Restricted Content"
      AuthUserFile /etc/apache2/.htpasswd
      Require valid-user
  </Directory
  
```

