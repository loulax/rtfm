# Element

----------------------------------------------------------------------------------------------------------------------------------------------------------

Element est une application de messagerie instantanée qui se base sur le protocole Matrix. Une des dernières pointes dans les systèmes de chiffrement. Cette application est libre et open source. Elle est compatible sur tout type d'appareil. Initialement, lorsque l'on s'inscrit sur le site officiel, il faut se connecter aux serveurs de matrix mais il est possible d'héberger son propre serveur afin de garder tout chez soi ou au bureau de son entreprise. 

Nous allons voir maintenant comment le mettre en place sur un serveur.

#### Prérequis

Un serveur Linux ayant 60go de stockage avec 4go de mémoire et un cpu 2 coeurs minimum.

### Installation serveur synapse

Se connecter au serveur en ssh avec un accès root (requis).

Il faut maintenant installer le serveur synapse pour l'usage de l'application bureau.

```bash
# Mettre à jour le serveur
sudo apt update && sudo apt -y full-upgrade
# Installation des prérequis
sudo apt install lsb-release wget openssl apt-transport-https curl postgresql libpq5 wget
# Installation de la clé gpg 
sudo wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
# Ajout du dépot de matrix
echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/matrix-org.list
# Mise à jour des dépots et installation de matrix-synapse
sudo apt update && sudo apt -y install matrix-synapse-py3
# Activation du service postgresql
sudo systemctl enable --now postgresql
# Connexion à postgresql
sudo -u postgres psql postgres
# Création de l'utilisateur et de la base pour matrix
create user matrix_synapse_rw with password 'm@trix!';
create database matrix_synapse with encoding='UTF8' lc_collate='C' lc_ctype='C' template='template0' owner='matrix_synapse_rw';
exit
```

### Editer les paramètres de Matrix-Synapse

Toute la configuration relative du serveur se trouve dans /etc/matrix-synapse/homeserver.yaml

```yaml
# Adapter les paramètres pour se connecter à la base de donnée
database:
  name: psycopg2
  txn_limit: 10000
  args:
    user: matrix_synapse_rw
    password: m@trix!
    database: matrix_synapse
    host: localhost
    port: 5432
    cp_min: 5
    cp_max: 10
```

Ajouter à la fin du fichier la ligne suivante :

```yaml
suppress_key_server_warning: true
```

### Génération d'une clé

Dans une console saisir les commandes suivantes:

```bash
RANDOMSTRING=$(openssl rand -base64 30)
echo "registration_shared_secret: $RANDOMSTRING" | sudo tee -a /etc/matrix-synapse/homeserver.yaml > /dev/null

# Redémarrer le service pour prendre en compte les modifications
sudo systemctl restart matrix-synapse
```

### Création d'un utilisateur

```bash
register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml http://DNSorIP:8008
```

On peut vérifier maintenant que le serveur tourne correctement en accédant à l'adresse : http://DNSorIP:8008

**Maintenant que tout est bien configuré, il est possible de se connecter depuis l'application bureau.** 

## 

## Element Web

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Dans cette section on va passer à l'étape d'installation de la partie web d'element pour pouvoir y accéder depuis une interface web

### Apache

Mise en place du serveur web apache2 qui hébergera les fichiers d'element.

```bash
sudo apt -y install apache2
# Vérifier la dernière version d'element
regex='<link rel="alternate" type="text\/html" href="https:\/\/github\.com\/vector-im\/element-web\/releases\/tag\/([^/]*)"' && response=$(curl -s https://github.com/vector-im/element-web/releases.atom) && [[ $response =~ $regex ]] && latestTag="${BASH_REMATCH[1]}"
# Récupérer depuis le dépot git la dernière version disponible
wget -O element.tar.gz https://github.com/vector-im/element-web/releases/download/$latestTag/element-$latestTag.tar.gz
# Extraction de l'archive dans le répertoire de site
sudo tar xzvf element.tar.gz -C /var/www/html
# Renommer le dossier 
sudo mv /var/www/html/element* /var/www/html/element
# Configuration des accès sur le répertoire html et des sous dossiers
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html
# Editer la configuration d'element dans /var/www/html/element/config.json
# Editer la ligne base_url en remplacant la valeur par celle-ci
https://mondomaine:8008
# Editer la ligne server_name 
mondomain.com
```

### LetsEncrypt + Certbot

Pour chiffrer les communications sur le serveur web, il va falloir créer un certificat qui sera lié au domaine. Avec letsencrypt et Certbot, il est possible d'avoir un certificat autosigné gratuitement.

Dans une console avec les accès administrateurs saisir les commandes suivantes :

```bash
# Installation de snap
sudo apt -y install snapd
# Installation de core
snap install core; snap refresh core;
# Installation de certbot
snap install certbot --classic
# Création du lien symbolic de certbot dans les commandes système 
ln -s /snap/bin/certbot /usr/bin/certbot
# Création du certificat
certbot --apache
# create ssl-certs group
sudo groupadd ssl-certs
# add matrix-synapse and root users to group
sudo usermod -aG ssl-certs matrix-synapse
sudo usermod -aG ssl-certs root
# verify the members of ssl-cert
getent group ssl-certs
# set owner group of /etc/letsencrypt
sudo chgrp -R ssl-certs /etc/letsencrypt
# set permissions on /etc/letsencrypt
sudo chmod -R g=rX /etc/letsencrypt
```

### Synapse

Il va falloir apporter une retourche à la configuration précédente pour spécifier les certificats et les liens vers la page web

Editer le fichier dans /etc/matrix-synapse/homeserver.yaml

```bash
# Ajouter les lignes suivantes à la fin du fichier 
tls_certificate_path: /etc/letsencrypt/live/<%DNS NAME%>/fullchain.pem
tls_private_key_path : /etc/letsencrypt/live/<%DNS NAME%>/privkey.pem
# Chercher la ligne tls et remplacer ainsi
tls: true
# Redémarrer le service
sudo systemctl restart matrix-synapse
```



## Nginx

Pour certaines configurations il est nécessaire de configurer un reverse_proxy afin de rediriger les requêtes qui sont sur l'hôte vers le bon serveur.

```
server {

	listen 443 ssl;
	server_name message.mydomain.com;
	ssl_certificate /etc/ssl/domain_name/fullchain.pem;
	ssl_certificate_key /etc/ssl/domaine_name/privkey.pem;

 	location / {
		proxy_pass http://10.0.5.2:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        client_max_body_size 50M;
    }

	location /element {

	proxy_pass http://10.0.5.2;

	}

}
```



### Spécificité pour synapse

Dans le cas du reverse_proxy certaines choses changent au niveau de la configuration de synapse.

#### Création de l'utilisateur

La commande plus au dessus doit être adapté en spécifiant **localhost** dans l'adresse. Très important sinon la requête va échouer:

```
register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml http://DNSorIP:8008
```

### Désactivation du ssl

Puisque nginx s'occuper de rediriger tous le traffic vers le https, il n'est pas nécessaire de l'activer pour le service synapse.

Editer la configuration dans /etc/matrix-synapse/homeserver.yaml

- Commenter les lignes qui renseigne les fichiers tls en mettant un # au début des 2 
- Remplacer tls: true par tls: false

Puis redémarrer le service matrix-synapse pour prendre en compte les dernière modification.
