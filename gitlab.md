# GitLab

Installation d'un GitLab pour Egide.

## Setup de la Machine Virtuelle

ATTENTION : Les paramËtres minimums recommandés par GitLab sont 4GO de RAM et 4 coeur de processeur.

La Machine Virtuelle est crée from Scratch, ‡ partir d'une Image Debian 10.9 AMD64 NetInst, sur Proxmox.

Un mot de passe root a été attribué, le nom de l'utilisateur par défaut est "adam". La VM est chiffré avec LUKS.

Une adresse IP fixe (4.15) a été attribué à la machine virtuelle.

## Setup du DNS

A ce moment, il faut setup le DNS avant de continuer. L'adresse du Serveur sera https://gitlab.mydomain.com. Pour se faire j'ai donc ajouté cette ligne au fichier de configuration de DNS de MyDomain :

    # vi /etc/bind/db.xxxxx
    gitlab  IN      A       192.168.4.15

Ne pas oublier également d'ajouter cette ligne ‡ la db du VPN. Sans oublier d'incrémenter de 1 la valeur du sérial, puis de relancer bind avec : 

    # systemctl restart bind9

On peut ensuite constater via un dig depuis le laptop que la bonne adresse est retournée :

    $ dig +short gitlab.mydomain.com
    192.168.4.15

## Installation de GitLab

(L'installation de postfix a été skip).

    # apt-get update
    # apt-get install -y curl openssh-server ca-certificates perl
    
    # curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | bash

On peut reprendre notre installation de Gitlab (pour l'instant, en HTTP).

    # EXTERNAL_URL="http://gitlab.mydomain.com" apt-get install gitlab-ee

L'installation de Gitlab va s'effectuer et on aura un joli ascii art ‡ la fin de l'installation en confirmation :

On va modifier le fichier de configuration de Gitlab afin d'activer le HTTPS (il faut mettre l'URL du Serveur, ainsi que DESACTIVER LetsEncrypt) :

    # nano /etc/gitlab/gitlab.rb
    
    external_url "https://gitlab.mydomain.com"
    letsencrypt['enable'] = false

Ensuite on va créer l'architecture de dossier attendue par Gitlab :

    # mkdir -p /etc/gitlab/ssl
    # chmod 755 /etc/gitlab/ssl
    # cp gitlab.mydomain.com.key gitlab.mydomain.com.crt /etc/gitlab/ssl/

Pour rappel, le fichier *.key correspond a privkey.pem, et le fichier *.crt correspond ‡ fullchain.pem.

Attention ‡ bien nommer les certificats comme précisé ci dessus, le chemin final des certificats doit Ítre /etc/gitlab/ssl/gitlab.mydomain.com.key et /etc/gitlab/ssl/gitlab.mydomain.com.crt afin que GitLab puisse trouver les certificats.

    # gitlab-ctl reconfigure

AprËs le reconfigure, le Serveur Gitlab est maintenant accessible ‡ https://gitlab.mydomain.com. Le mot de passe initial de l'utilisateur root est dans /etc/gitlab/initial_root_password, accessible pendant 24h.

## Hardening Gitlab

### Désactiver l'Inscription Publique

On va cliquer sur Menu -> Admin.

Puis Settings -> Général.

Puis dans Sign-up restrictions, décocher la case "Sign-up enabled", et sauvegarder.

### Changer le mot de passe, activer la 2FA, changer l'user root par defaut

On va cliquer sur Edit Profile -> Account. Ici on peut activer la 2FA et changer l'username root pour un autre nom.

On va cliquer sur Edit Profile -> Password. Ici on va changer le mot de passe par defaut. Il faut ensuite se reconnecter.

### Restreindre la visibilité du Gitlab (forcer ‡ Ítre login pour naviguer, mÍme sur les projets public)

On va cliquer sur Menu -> Admin.

Puis Settings -> Général.

Puis dans  Visibility and access controls, dans Restricted visibility levels, tout cocher.

## Sources

- https://about.gitlab.com/install/#debian
- https://docs.gitlab.com/omnibus/settings/nginx.html#manually-configuring-https
