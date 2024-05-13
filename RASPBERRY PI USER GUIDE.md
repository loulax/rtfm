# RASPBERRY PI USER GUIDE



## SSH

Secure Shell est un protocol réseau permettant d'accéder à un appareil à distance (si celui-ci a le protocol d'installé et le serveur en écoute). Il est principalement utilisé afin d'effectuer différentes tâches d'administration depuis une console. Il écoute sur le port 22 (Par défaut) mais ce port peut être changé dans son fichier de configuration dans "/etc/ssh/sshd_config" (Port 22).

### Sécurisation

Afin d'en empêcher les accès non autorisés, certains paramètres doivent être modifiés, voici une liste non exhaustive:

```
Port 22 -> Port XXXX # Le port sur lequel le service va écouter
PermitRootLogin no / without-password # Autoriser ou pas l'accès ssh via le compte super admin (à éviter)
PubkeyAuthentication yes # Autoriser l'accès en ssh via l'utilisation d'une paire de clé
PasswordAuthentication no # Ne pas autoriser l'accès en ssh via le mot de passe
AllowUsers <allowed user> # Autoriser l'accès à un certain utilisateur 
AllowGroups <allowed groups> # Autoriser l'accès à un certain groupe
MaxAuthTries 3 # Autoriser maximum 3 tentatives en échec, au delà, il sera bloqué
MaxSessions 2 # Autoriser maximum 2 connexions simultanées 
```

Pour permettre l'accès via une clé, il va falloir générer sur l'appareil utilisé pour se connecter, la paire de clé (chiffrement asymétrique utilisant généralement le chiffrement RSA) :

**Ce chiffrement va faire le lien entre une clé publique et privé pour vérifier l'hôte qui tente de se connecter, la clé publique se trouve sur le serveur et le client utilisera la clé privé pour se connecter.**

Dans un premier temps, il demande à quel endroit stocker le fichier et le nom de celui-ci

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/loulax/.ssh/id_rsa): raspberry-key
```

Ensuite, si l'on veut mettre une "passphrase" pour l'accès à cette clé, c'est fortement recommandé principalement dans le cas où c'est un ordinateur portable et que l'on est amené à se déplacer régulièrement avec:

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Si l'on veut pas de passphrase, faire ENTER 2x pour passer cette étape, puis la clé sera générée 

```
Your identification has been saved in raspberry-key
Your public key has been saved in raspberry-key.pub
The key fingerprint is:
SHA256:5zmA8VOTEbBSXS3mEeza9l0VtfQuk0h9WTCeOWIa+y0 loulax@undefined-pc
The key's randomart image is:
+---[RSA 4096]----+
|        ooo+oooo+|
|       . ..o=oo==|
|      o . =++o*o+|
|       = . Boo =.|
|      . S +o. + o|
|         =.oo. o.|
|          +.E....|
|           . .. .|
|                 |
+----[SHA256]-----+
```

On peux la retrouver dans le répertoire courant où nous nous trouvions lorsque la clé a était généré:

```
➜  ~ ls -l raspberry-key
-rw-------@ 1 loulax  staff  3454 May 13 12:21 raspberry-key
```

Une fois généré, il va falloir l'envoyer sur le serveur 

```
ssh-copy-id -i raspberry-key -p XXXX -l <user>@<ip du terminal distant> 

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "raspberry-key.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@172.20.10.2's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'root@172.20.10.2'"
and check to make sure that only the key(s) you wanted were added.
```

### Accès en ssh via la clé

```
ssh -i raspberry-key -p XXXX -l <user>@<ip du terminal distant>

Enter passphrase for key 'raspberry-key':


Linux vm1 6.1.0-21-arm64 #1 SMP Debian 6.1.90-1 (2024-05-03) aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon May 13 06:10:23 2024 from 172.20.10.3
root@vm1:~#
```

Me voilà connecter à mon équipement via la clé privé.

> [!NOTE]
>
> Dans cet exemple je n'ai pas changé le port ssh en écoute donc pas besoin de spécifier quand c'est celui par défaut



## Debug service

Un service sous linux peut-être amené à "crash" pour différentes raisons. Voici les premiers diagnostiques à faire pour vérifier l'origine de la panne:

- Utiliser systemd status pour vérifier l'état sur service et voir différentes erreurs. systemd est le démon qui gère les différents services sous linux (pour la majorité des distributions).

  - Pour vérifier l'état d'un service, utiliser la commande systemctl status <service>

  - pour démarrer un service systemctl start <service>
  - pour stopper un service systemctl stop <service>
  - pour activer un service au démarrage systemctl enable <service>

  ```
  root@vm1:~# systemctl status sshd
  ● ssh.service - OpenBSD Secure Shell server
       Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
       Active: active (running) since Mon 2024-05-13 06:09:34 EDT; 33min ago
         Docs: man:sshd(8)
               man:sshd_config(5)
      Process: 1699 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
     Main PID: 1700 (sshd)
        Tasks: 1 (limit: 2250)
       Memory: 6.8M
          CPU: 462ms
       CGroup: /system.slice/ssh.service
               └─1700 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
  
  May 13 06:35:26 vm1 sshd[1743]: Connection closed by authenticating user root 172.20.10.3 port 53865 [preauth]
  May 13 06:35:31 vm1 sshd[1745]: Accepted password for root from 172.20.10.3 port 53866 ssh2
  May 13 06:35:31 vm1 sshd[1745]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
  May 13 06:35:31 vm1 sshd[1745]: pam_env(sshd:session): deprecated reading of user environment enabled
  May 13 06:35:31 vm1 sshd[1745]: Received disconnect from 172.20.10.3 port 53866:11: disconnected by user
  May 13 06:35:31 vm1 sshd[1745]: Disconnected from user root 172.20.10.3 port 53866
  May 13 06:35:31 vm1 sshd[1745]: pam_unix(sshd:session): session closed for user root
  May 13 06:36:20 vm1 sshd[1758]: Accepted publickey for root from 172.20.10.3 port 53968 ssh2: RSA SHA256:5zmA8VOTEbBSXS3mEeza9l0VtfQuk0h9WTCeOWIa+y0
  May 13 06:36:20 vm1 sshd[1758]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
  May 13 06:36:20 vm1 sshd[1758]: pam_env(sshd:session): deprecated reading of user environment enabled
  ```

  Dans cet exemple avec le service ssh, on peux voir est chargé et actif (qu'il démarre automatiquement au démarrage du serveur).

  Aucune erreur apparente dans l'exemple ci-dessus, par contre dans le cas suivant, j'ai créé un service firewall qui gère mes règles de pare-feu dans un fichier bash (shell script):

  ```
  root@debian:~# systemctl status firewall
  × firewall.service - IPtables firewall
       Loaded: loaded (/lib/systemd/system/firewall.service; disabled; preset: enabled)
       Active: failed (Result: exit-code) since Mon 2024-05-13 07:47:50 EDT; 18s ago
     Duration: 50.450s
      Process: 14727 ExecStart=/etc/init.d/fw_on.sh (code=exited, status=203/EXEC)
     Main PID: 14727 (code=exited, status=203/EXEC)
          CPU: 1ms
  
  May 13 07:47:50 debian systemd[1]: Starting firewall.service - IPtables firewall...
  May 13 07:47:50 debian (fw_on.sh)[14727]: firewall.service: Failed to execute /etc/init.d/fw_on.sh: Exec format error
  May 13 07:47:50 debian (fw_on.sh)[14727]: firewall.service: Failed at step EXEC spawning /etc/init.d/fw_on.sh: Exec format error
  May 13 07:47:50 debian systemd[1]: firewall.service: Main process exited, code=exited, status=203/EXEC
  May 13 07:47:50 debian systemd[1]: firewall.service: Failed with result 'exit-code'.
  May 13 07:47:50 debian systemd[1]: Failed to start firewall.service - IPtables firewall.
  ```

  On peux constater que le service est loaded mais active: failed. Et en regardant en dessous, on voit l'erreur qui empêche au service de se charger d'où l'intérêt de regarder en 1er lieu avec systemd. On voit exec format error, autrement dit une erreur dans le format du fichier /etc/init.d/fw_on.sh, si j'inspecte avec la commande cat ou head, je peux voir l'erreur (bien sûr faite exprès pour l'exemple) :

  ```
  z#!/bin/bash # Le shebang
  
  iptables="/usr/sbin/iptables"
  
  #CLEAR IPTABLES RULES
  $iptables -F
  $iptables -X
  $iptables -t nat -F
  $iptables -t nat -X
  ```

  Le shebang est utilisé pour les fichiers de code sous linux lorsque l'extension d'un fichier n'est pas spécifié, il faut lui indiquer le langage utilisé dans le script pour pouvoir ensuite l'interpréter correctement. Dans ce cas, c'est du bash d'où /bin/bash (le fichier binaire du programme/langage)

  

- journalctl permet d'avoir accès à plus de message pour débogue
  journalctl permet de visualiser les journaux systèmes plus facilement sans devoir aller fouiller manuellement dans le répertoire /var/log/. Pour voir le journal d'un service:

  ```
  journalctl -xeu <service>
  ```

  Dans l'exemple avec le pare-feu:

  ```
  journalctl -xeu firewall.service
  ```

  Ce qui donne beaucoup d'informations mais la plus utilie est celle-ci :

  ```
  May 13 07:47:50 debian (fw_on.sh)[14727]: firewall.service: Failed to execute /etc/init.d/fw_on.sh: Exec format error
  May 13 07:47:50 debian (fw_on.sh)[14727]: firewall.service: Failed at step EXEC spawning /etc/init.d/fw_on.sh: Exec format error
  ```

  Dans ce contexte les journaux système ne donne pas plus d'infos que systemd mais il peut parfois être plus généreux.

- (Si c'est un service réseau), vérifier les ports en écoute sur la machine
  Il existe plusieurs commandes pour ça mais les principales sont <u>**netstat**</u> et <u>**ss**</u>.
  Par exemple pour voir s le ssh est bien en écoute sur la machine...
  Avec netstat :

  ```
  root@debian:~# netstat -laputen
  Active Internet connections (servers and established)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name
  tcp        0      0 0.0.0.0:2043            0.0.0.0:*               LISTEN      0          46003      4660/sshd: /usr/sbi
  tcp        0      0 192.168.1.64:59684      199.232.170.132:80      TIME_WAIT   0          0          -
  tcp        0      0 192.168.1.64:2043       192.168.1.118:56051     ESTABLISHED 0          82705      15395/sshd: root@pt
  tcp        0      0 192.168.36.215:2043     192.168.40.25:50275     ESTABLISHED 0          79837      14729/sshd: root@pt
  tcp6       0      0 :::2043                 :::*                    LISTEN      0          46014      4660/sshd: /usr/sbi
  udp        0      0 0.0.0.0:68              0.0.0.0:*                           0          82675      15321/dhclient
  udp        0      0 0.0.0.0:58537           0.0.0.0:*                           0          14812      -
  udp6       0      0 :::58537                :::*                                0          14813      -
  ```

  On peux voir sur la 1ère ligne déjà que le port 2043 écoute sur toutes les ip "0.0.0.0" et au bout de la ligne on voit le programme /usr/bin/sshd

  Avec ss :

  ```
  root@debian:~# ss -sltnpu
  Total: 102
  TCP:   4 (estab 2, closed 0, orphaned 0, timewait 0)
  
  Transport Total     IP        IPv6
  RAW	  0         0         0
  UDP	  3         2         1
  TCP	  4         3         1
  INET	  7         5         2
  FRAG	  0         0         0
  
  Netid            State              Recv-Q             Send-Q                         Local Address:Port                          Peer Address:Port            Process
  udp              UNCONN             0                  0                                    0.0.0.0:68                                 0.0.0.0:*                users:(("dhclient",pid=15321,fd=7))
  udp              UNCONN             0                  0                                    0.0.0.0:58537                              0.0.0.0:*
  udp              UNCONN             0                  0                                       [::]:58537                                 [::]:*
  tcp              LISTEN             0                  128                                  0.0.0.0:2043                               0.0.0.0:*                users:(("sshd",pid=4660,fd=3))
  tcp              LISTEN             0                  128                                     [::]:2043                                  [::]:*                users:(("sshd",pid=4660,fd=4))
  ```

  Dans ce cas on peux voir également le port 2043 en écoute sur toutes les ip via l'utilisateur sshd. Au dessus on peux voir l'utilisateur dhclient qui écoute sur le port 68 pour recevoir des trames DHCP.

  

