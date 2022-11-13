# Mail

## LXD 

Les services sont dans des LXD. Pour se faire, on crée le conteneur LXD avec la configuration suivante :

    $ lxc launch images:debian/bullseye mail
    $ lxc stop mail
    $ lxc network attach lxdbr0 mail eth0
    $ lxc config device set mail eth0 ipv4.address 10.0.5.7
    $ lxc start mail
    $ lxc exec mail -- /bin/bash

## Installation

Fix le DNS :

    $ systemctl disable --now systemd-resolved
    $ rm /etc/resolv.conf
    $ echo "nameserver 1.1.1.1" > /etc/resolv.conf

Pendant l'installation des paquets, dans le popup de postfix, on va selectionner "Local".

On va Ègalement dÈfinir le hostname pour le Serveur :

    $ echo "127.0.0.1       mail.mydomain.com" >> /etc/hosts
    $ hostnamectl set-hostname mail.mydomain.com

On installe gnupg2 et wget :

    $ apt update
    $ apt install -y gnupg2 wget

On ajoute la clef de Sogo :

    $ apt-key adv --recv-keys F8A27B36A6E2EAE9

On installe l'ensemble des outils via :

    $ wget https://github.com/iredmail/iRedMail/archive/refs/tags/1.6.0.tar.gz
    $ tar xvzf 1.6.0.tar.gz
    $ cd iRedMail-1.6.0/
    $ chmod +x iRedMail.sh
    $ ./iRedMail.sh

Pendant l'installation du script, on sera prompt de quelques options :

- Serveur Web : Nginx
- Database : MariaDB
- Mot de passe : lalaland
- Your first mail domain name : mydomain.com
- Mail domain admin password : lalaland
- Optionnal components : Decocher RoundCube et Fail2Ban, Cocher SoGo

    < Question > File: /etc/nftables.conf, with SSHD ports: 22. [Y|n]n

```
* URLs of installed web applications:
*
* - SOGo groupware: https://mail.mydomain.com/SOGo/
* - netdata (monitor): https://mail.mydomain.com/netdata/
*
* - Web admin panel (iRedAdmin): https://mail.mydomain.com/iredadmin/
*
* You can login to above links with below credential:
*
* - Username: postmaster@mydomain.com
* - Password: lalaland
```

On reboot le conteneur avant de continuer.

Ensuite il faudra mettre fullchain.pem dans /etc/ssl/certs/iRedMail.crt et privkey.pem dans /etc/ssl/private/iRedMail.key.

Le login par defaut est postmaster@mydomain.com / lalaland.

## Post-Install (Web)

Pour accéder au conteneur via l'IP de la VM qui a crée le conteneur, on va faire une régle de PREROUTING :

    $ iptables -t nat -D PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.5.7
    $ iptables -t nat -I POSTROUTING -j MASQUERADE

On peut maintenant acceder ‡ l'interface de iRedMail via l'IP de la VM qui a crÈe le Conteneur (‡ /iredadmin ou /sogo).

## Backup

    $ cd
    $ lxc stop mail
    $ lxc export mail mail.tar.gz
    $ lxc delete mail

## Clean des Règles Iptables

    $ iptables -t nat -D PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.5.7

