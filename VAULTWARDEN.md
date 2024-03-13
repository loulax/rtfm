# VAULTWARDEN



### Pré-requis:

Avoir un serveur linux avec les performances minimales suivantes:

- RAM min : 2Gb
- CPU : Dual core 1ghz
- 32Gb de stockage
- Connexion internet 100mb/s

## Installation docker

Docker est un système de containerisation qui permet d'avoir des machines isolés depuis son hôte, le princiipe est pratiquement similaire à la virtualisation seulement il consomme beaucoup moins de ressources matériel et se déploie bcp plus rapidement puisque c'est qu'une image d'un système. C'est d'ailleurs un fork de LXC qui est natif à linux

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Il faut ensuite mettre à jour la liste des paquets pour prendre en compte les modifications effectuées sur les repos locaux puis installer docker

```bash
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose docker-compose-plugin
```

Pour vérifier que docker est bien setup, faire la commande suivante:

```bash
docker run hello-world
```

## Installation de Vaultwarden

Il faut "pull" l'image depuis les dépots de docker hub

```bash
docker pull vaultwarden/server:latest
```

On va générer un token pour l'accès admin

```bash
openssl rand -base64 58

#Ca génère une suite comme ceci
zSm8Fqz0JuOAaUffIpyqpd8aG8wAg0mY+ktVN1j1i1NeOQwoUxVSqXQwqjZRuY0b/4Xv501t0fCLF5gwOgouRfU+kEg=
```

On va créer un point de montage local pour le montage du container

```bash
cd /root/
mkdir vaultwarden && chmod -R 777 vaultwarden
```

Maintenant on lancer le container comme ceci:

```bash
docker -d -e ADMIN_TOKEN=<generated token> --name <container name> -v <local path>:/data/ -p 8080:80 vaultwarden/server:latest
```



## Installation NGINX

Je vais installer maintenant nginx en tant que reverse-proxi pour permettre de pointer le sous-domaine de mon container et rediriger le traffic vers celui-ci.

```nginx
apt-get install -y nginx
cd /etc/nginx/sites-enabled
nano mydomain.conf


server {
    listen 443 ssl http2 ;
    server_name sub.mydomain.tld;

    http2_push_preload on;

    ssl_certificate <chemin vers le certification fullchain.cert>;
    ssl_certificate_key <chemin vers la clé privé du certificat>;

    add_header Strict-Transport-Security "max-age=31536000";

    location / {
        proxy_set_header X-Forwarded-Host $host:$server_port;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass_request_headers on;
        proxy_pass http://localhost:8080;
    }
}
```

Le coffre vaultwarden inclut une page d'administration accessible à l'adresse https://mydomain.tld/admin, cet espace permet de paramétrer pleins de choses, notamment gérer les utilisateurs etc ce qui est par conséquent une zone très sensible. Nous allons rajouter la section suivante dans vaultwarden avec de ne pas permettre à n'importe qui d'y accéder.

```nginx
   location /admin {

	allow <réseau autorisé>;
	deny all;

	proxy_set_header "Connection" "";

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass http://localhost:8080/admin;
}
```
Cette section contrôle toutes les ip qui accèdes à /admiin et si ça ne match pas avec la directive allow <réseau autorisé>; une erreur 403 sera retourné. 
Alors soit on autorise depuis sont réseau local ou depuis un vpn voir les 2 selon le cas de chacun.

## Iptables

maintenant il faut mettre les règles iptables adéquates pour faire fonctionner le container, sachant que docker en créé initialement (se référer à ma doc iptables pour la gestion d'un service et gérer les règles iptables) :

```bash
#Création des règles de pare-feu
vim /etc/init.d/fw_up.sh

#CREATEION DES CHAINES DOCKER
/usr/sbin/iptables -N DOCKER
/usr/sbin/iptables -N DOCKER-ISOLATION-STAGE-1
/usr/sbin/iptables -N DOCKER-ISOLATION-STAGE-2
/usr/sbin/iptables -N DOCKER-USER
/usr/sbin/iptables -t nat -N DOCKER

#LOCAL TRAFFIC
/usr/sbin/iptables -A INPUT -i lo -j ACCEPT
/usr/sbin/iptables -A OUTPUT -o lo -j ACCEPT

#Autoriser le traffic web
/usr/sbin/iptables -A INPUT -p tcp -m multiport --dports 80,443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
/usr/sbin/iptables -A OUTPUT -p tcp -m multiport --sports 80,443 -m conntrack --ctstate ESTABLISHED -j ACCEPT

#DOCKER
/usr/sbin/iptables -A FORWARD -j DOCKER-USER
/usr/sbin/iptables -A FORWARD -j DOCKER-ISOLATION-STAGE-1
/usr/sbin/iptables -A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
/usr/sbin/iptables -A FORWARD -o docker0 -j DOCKER
/usr/sbin/iptables -A FORWARD -i docker0 ! -o docker0 -j ACCEPT
/usr/sbin/iptables -A FORWARD -i docker0 -o docker0 -j ACCEPT
/usr/sbin/iptables -A DOCKER -d 172.17.0.2/32 ! -i docker0 -o docker0 -p tcp -m tcp --dport 80 -j ACCEPT
/usr/sbin/iptables -A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
/usr/sbin/iptables -A DOCKER-ISOLATION-STAGE-1 -j RETURN
/usr/sbin/iptables -A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
/usr/sbin/iptables -A DOCKER-ISOLATION-STAGE-2 -j RETURN
/usr/sbin/iptables -A DOCKER-USER -j RETURN
/usr/sbin/iptables -t nat -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
/usr/sbin/iptables -t nat -A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
/usr/sbin/iptables -t nat -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
/usr/sbin/iptables -t nat -A POSTROUTING -s 172.17.0.2/32 -d 172.17.0.2/32 -p tcp -m tcp --dport 80 -j MASQUERADE
/usr/sbin/iptables -t nat -A DOCKER -i docker0 -j RETURN
/usr/sbin/iptables -t nat -A DOCKER ! -i docker0 -p tcp -m tcp --dport 9090 -j DNAT --to-destination 172.17.0.2:80

```

Maintenant rendez-vous à l'adresse web configuré dans nginx pour accéder au container vaultwarden https://sub.mydomain.tld/admin

![](img/admin_bw.png)

Puis dans la section SMTP, renseigner les paramètres suivants selon votre fournisseur

![](img/smtp_vaultwarden.jpg)

Et enfin validez en bas à gauche.

Il sera probablement nécessaire de redémarrer le container, pour ce faire:

```bash
docker stop <container name>
docker start <container name>
#pour y accéder
docker exec -it <container name> /bin/bash
```

## Backup ses données

Toutes les données tel qu'identifiants et comptes sur le serveur bitwarden se trouvent sur le container dans /data/db.sqlite3 à l'exception des fichiers et send uploadé dessus. Ces derniers se trouvent dans un dossier propre à eux.

Une tache cron est programmé sur le container docker qui sauvegardera le fichier db.sqlite3 depuis le dossier "Data" dans le container vers le dossier local (celui sur l'hote)

Pour transférer manuellement le fichier db.sqlite3 depuis le container vers l'hote utiliser la commande suivante :

```
docker cp <container name>:/data/db.sqlite3 /chemin-destination
```

Puis si nécessaire utiliser SSH ou FTP afin de mettre le fichier en lieu sûr en cas de panne du serveur hote.

## Restoration de ses données

Si dans un cas on a besoin de migrer son container sur un autre serveur, revenir à l'étape de déploiement du container puis restaurer le fichier sql comme ceci:

```
docker cp db.sqlite3 <container name>:/data/
```

Attention, il va falloir stop le container et le relancer pour pouvoir prendre en compte le dernier fichier et ainsi récupérer les données importer.
