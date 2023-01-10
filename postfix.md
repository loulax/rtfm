# SMTP Relay Fedora

## Introduction

<u>Prérequis: Posséder un domaine avec un serveur configuré pour l'envoi de mail et/ou la réception (optionnel)</u>

Dans le but de pouvoir utiliser son serveur web pour envoyer des mails, cas d'usage envoi de mail en php. Il va falloir faire certaines étapes pour faire office de relay smtp. Dans ce cas, je me trouve sur un système Fedora mais les manips sont identiques sur d'autres distributions Linux, seulement les paquets peuvent changer pour certains et quelques petites différences au niveau des fichiers de configurations (proablement). Je vais utiliser le service postfix très populaire qui permet de deployer rapidement un serveur mail et mailutils qui permet d'envoyer des mails via la console.

### Installation

```
# Se connecter en super user
> sudo su -
> dnf -y install postfix mailx
```

### Configuration général

Editer le fichier /etc/postfix/main.cf avec votre éditeur préféré

```
> vim /etc/postfix/main.cf
```

Editer les paramètres suivants:

```
myhostname : virtual.domain.tld (Pour mon cas j'ai un serveur mail sous **mail.loulax.fr**)
mydomain : domain.tld (Pour mon cas **loulax.fr**)
relayhost : [virtual.domain.tld]:<port_smtp>(optionnel) Si le port smtp par defaut est changé ou vous préférez utiliser le smtps 
```



#### configuration ssl/tls (optionnel)

```
smtp_tls_security_level = encrypt
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_use_tls = yes
smtp_tls_CAfile = /etc/pki/tls/certs/mycert.pem
smtp_tls_key_file = /etc/pki/tls/private/mycert.key
smtp_tls_wrappermode = yes
meta_directory = /etc/postfix
shlib_directory = /usr/lib64/postfix
```

Il va falloir maintenant créer un fichier dans lequel se trouvera les identifiants du compte smtp

```
> echo "[mail.domain.tld]:<port> user@domain.tld:password" > /etc/postfix/sasl_passwd
```

Ensuite créer un fichier .db pour avoir un hash des identifiants à des fins de mesures de sécurité

```
> postmap /etc/postfix/sasl_passwd
```

Un fichier sasl_passwd.db sera créé suite à cette commande.

Maintenant plus qu'à redémarrer le service postfix

```
> systemctl restart postfix

# Nous pouvons vérifier avant de redémarrer, la configuration avec la commande suivante
> postconf 
> postconf -t
```

Nous allons tester le bon fonctionnement en envoyant un mail 

```
echo "Sending mail from my terminal" |mail -s "Subject" -r user@domain.tld user@domain.tld
```

echo : Ecrire le corps du mail

| pipe le corps dans une autre commande, en loccurence mail

mail -s : pour Renseigner un sujet

mail -r : pour renseigner l'adresse de l'expéditeur

puis enfin renseigner l'adresse du destinataire à la fin



## Apache

Maintenant que le système est prêt pour l'envoi de mail, nous pouvons faire de l'envoi de mail depuis notre serveur web

#### Sous Debian/Ubuntu

Editer le fichier /etc/php/<version>/apache2/php.ini

```
[mail function]
sendmail_path = "/usr/bin/mail -r user@domain.tld"
> systemctl restart apache2
```

#### Sous Redhat/Fedora

Editer le fichier /etc/php.ini

```
 [mail function]
 sendmail_path = "/usr/bin/mail -r user@domain.tld"
 
 > systemctl restart httpd
```

