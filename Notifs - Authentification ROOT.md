# Notifs - Authentification ROOT



## I) Préparer l'envoi de mail

Il faut installer les dépendances.

```bash
$ apt install mailutils msmtp msmtp-mta -y
[...]
```

Il faut configurer maintenant **msmtp** pour le compte root.

```bash
$ vim /root/.msmtprc
defaults
auth          on
tls           on
logfile       /var/log/msmtp

account       user
auth          plain
host          domain.tld
port          587
user          [MAIL]
password      [PASSWORD]

# Set a default account
account default : default
```

On attribut les bons droits au fichier.

```bash
$ chmod 600 /root/.msmtprc
```



## II) Préparer le pare-feu

Il faut mettre en place les règles iptables pour laisser passer l'envoi de mail.

```bash
$ iptables -I OUTPUT -s 192.168.1.35 -d 0.0.0.0 -p tcp --dport 587 -j ACCEPT
$ iptables -I INPUT -s 0.0.0.0 -d 192.168.1.35 -p tcp --sport 587 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

<u>À noter</u> : pour être plus précis, on peut remplacer "0.0.0.0" par l'adresse IP du serveur SMTP de **domain.tld** .



## III) Préparer le compte ROOT

Afin que l'envoie de mail se fasse automatiquement lorsqu'un utilisateur se connecte sur le compte root, nous allons mettre en place les bonnes commandes dans le fichier **.bashrc** de ce dernier.

```bash
$ vim /root/.bashrc
[...]
echo "<h2><b>Serveur `hostname` - Nouvelle connexion SSH</b></h2><br><b>- Hôte distant : </b>`hostname`<br><b>- Utilisateur : </b>`whoami`<br><b>- Date : </b>`date`" | /usr/bin/mail -r [EMAIL EXPEDITEUR] -s "Connexion SSH" "[EMAIL DU DESTINATAIRE]" -a "Content-Type: text/html"
```

Le mail reçu aura la forme suivante :

```
Serveur [HOSTNAME] - Nouvelle connexion SSH

- Hôte distant : [HOSTNAME]
- Utilisateur : [WHOAMI]
- Date : [DATE]
```



Tout est prêt.
