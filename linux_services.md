# Les services sous linux

## SystemV

C'est le système historique d'initialisation des processus sous linux. Tous les fichiers de configuration des processus se trouvent dans /etc/init.d.

A l'heure actuelle la plupart des systèmes sous linux utilisent systemd. Il permet de gérer les processus via des deamons.
Pour gérer les états des services avec systemd, il faut utiliser la commande systemctl, par exemple :
Pour démarrer un service 

```
systemctl start <service>
```

Pour stopper un service :

```
systemctl stop <service>
```

Pour redémarrer un service:

```
systemctl restart <service>
```

Pour consulter l'état d'un service :

```
systemctl status <service>
```

Une commande qui peux être utile aussi pour lister les dépendences d'une target c'est 

```
systemctl list-dependencies <name.target>
```



## NTP

Le protocole NTP (Network Time Protocol) est un protcole réseau qui écoute en UDP(123) pour permettre de donner l'heure à un système informatique.

### Gérer l'heure manuellement

Pour modifier l'heure manuellement, il faut utiliser la commande suivante:

```
timedatectl (Pour afficher les informations)
timedatectl set-time "yyyy-mm-dd hh:mm:ss"
```

Pour afficher les timezones disponibles :

```
timedatectl list-timezones
timedatctl list-timezones |grep Paris
```

**Le service qui gère l'heure s'appel systemd-timesyncd.**

### Gérer l'heure via chrony

Chrony est un outil sous Debian, Ubuntu et dérivés qui permet de gérer tout ça facilement et monter un serveur ntp afin de synchroniser tout son réseau dessus. Cela évite de surcharger les serveurs NTP publiques.

ON peux installer chrony via :

```
apt -y install chrony
```

Son fichier de configuraiton se trouve dans /etc/chrony/chrony.conf

Pour synchroniser le serveur local avec les serveurs de temps sur internet, il faut spécifier les pools dans le fichier de conf.

Exemple:

```
pool ntp.ubuntu.com        iburst maxsources 4
pool 0.ubuntu.pool.ntp.org iburst maxsources 1
pool 1.ubuntu.pool.ntp.org iburst maxsources 1
pool 2.ubuntu.pool.ntp.org iburst maxsources 2
```

maxsources indique le nombre de ressources à utiliser pour chacun des serveurs ntp listés.

A noter que par défaut le serveur ntp bloque les communications entre les différents hotes du réseau. Il faut ajouter dans la configuration, la directive **allow** pour permettre d'autoriser un réseau à faire des requêtes.

Il existe aussi une directive qui permet d'utiliser un fichier pour accélérer la synchronisation :

```
driftfile /var/lib/chrony/chrony.drift
```

Une fois terminé, il faut redémarrer le service :

```
systemctl restart chrony # redémarre le service pour prendre en compte les modifications
systemctl enable chrony # active chrony au démarrage
```

Puisque chrony se charge de l'heure, il faut désactiver le service ntp par défaut via la commande :

```
timedatectl set-ntp false
```

Nous pouvons voir maintenant les source de chrony via :

```
chronyc sources
```

Le serveur qui a ce symbole ^* indique que c'est celui utilisé par chrony.

Ceux qui ont ^+ sont utilisés pour calculer une moyenne de temps et les autres ^- ne sont pas utilisés.

### Configuration client

Du côté client, il suffit de modifier le fichier de configuration /etc/systemd/timesyncd.conf :

```
[Time]
NTP=vm-server
```



## LDAP

Lightweight Directory Access Protocol est un standard permettant de gérer l'accès des utilisateurs et groupes au sein d'environnement pro aux ressources de ce dernier.

Il existe 2 servvices majeurs:

- Microsoft ADDS (sous windows, propriétaire Microsoft)
- OpenLDAP (sous linux, opensource)

Voici comment est représenté un annuaire LDAP, on appel généralement ça une forêt ou DIT (Directory Information Tree) dont la racine en haut :

![](images/ldap_tree.png)

On peux distinguer des attributs et des valeurs dans une classe d'objets.
Comme attributs on peux retrouver :

- dc (domain component) : Une partie d'un nom DNS.  Pour une entreprise cela vaut souvent dc=mon-entreprise,dc=com

- cn (common name) : le nom commun. Pour une upersonne, c'est le prénom + nom

- gn (given name) : le prénom

- sn (surname) : le nom

- o (organization name) : pour une entreprise, c'est son nom ou celui de sa filiale

- ou (organisational unit) : l'unité d'organisation. Dans une entreprise c'est souvent le département (commercial, compta, IT ...)

  

## OpenLDAP

Comme dit plus haut OpenLDAP est un annuaire sous Linux qui est équivalent à Active Directory sous licence Microsoft, seulement il est open source.

> [!IMPORTANT]
>
> Dans mon exemple je me baserais sur le domaine intrarmour.local.

### Installation

```
apt-get -y install slapd ldap-utils
```

slapd est le démon qui gère l'annuaire tandis qu'ldap-utils ajoute des utilitaires pour les clients afin de permettre d'interroger / modifier l'annuaire.

Durant l'installation il sera demandé le mot de passe de l'administrateur de l'annuaire, inutile de le saisir car il va falloir reconfigurer ensuite le programme.

Il faut désormais utiliser l'outil de configuration debconf de Debian pour définir la configuratio nde base de l'annuaire:

```
dpkg-reconfigure slapd
```

indiquez :

- No pour la première question afin de pouvoir utiliser l’outil de configuration ;
- pour nom DNS : mon-entreprise.com ;
- pour nom d’organisation : mon-entreprise ;
- le mot de passe administrateur x2 ;
- choisissez le format de base par défaut : mdb ;
- No pour savoir si la base doit être supprimée quand slapd est purgé ;
- Yes pour déplacer l’ancienne base de données.

Comme le DNS est mon-entreprise.com, la racine du DIT est "dc=mon-entreprise,dc=com". On peux faire une recherche via la commande :

```
ldap-search -Q -L -Y EXTERNAL -H ldapi:/// -b dc=mon-enreprise,dc=com
```

-Q : Active le mode silencieux pour L'authentification SASL
-Y Indique le mode SASL choisi pour l'authentification. Normalement, EXTERNAL implique une authentification par certificat client mais dans ce cas ça signifie qu'elle se fera par l'UID et le GID du compte système.
-H  : Indique l'url à requêter ici **ldapi:///** dit de se connecter à la socket unix locale.
-b indique le noeud à partir duquel faire la recherche.

### LDIF

Cela signifie LDAP Directory Interchange Format. C'est le format utilisé par ldap pour apporter des modifications dans l'annuaire.
Le format sera toujours de la forme suivante :

```
dn: <Le dn que vous voulez changer>
changetype: <add, replace ou delete. Cette ligne est optionnelle>
<attribut ou objectclass>: valeur
```

Dans les anciennes version de slapd, il fallait modifier la configuration de l'annuaire dans le répertoire /etc/ldap/ puis redémarrer le serveur pour prendre en compte la modification.

Bien que cette méthode soit toujours possible, il est aujourd'hui recommandé de faire la configuration à chaud, ce que l'on appel aussi OLC (On-Line Configuration). Cette dernière permet de ne pas avoir à redémarrer le serveur pour que la configuration soit prise en compte.

On peux utiliser la commande ldapsearch pour effectuer tout type de requête sur le serveur.
Par ex pour afficher le nivau de log : 

```
ldapsearch -Q -LLLL -Y EXTERNAL -H ldapi:/// -b cn=config -s base olcLogLevel
```

Et on peux constater qu'il est à None par défaut

```
dn: cn=config
olcLogLevel: none
```

Il est possible de changer le niveau de log en créant un fichier ldiff comme ceci :

Il doit contenir le dn de l'entrée que l'on veut changer, si c'est la racine comme ici, il faut indiquer le **cn config**

```
# vi log_level.ldif

dn: cn=config
changeType:modify
replace:olcLogLevel
olcLogLevel:stats
```

Enregistrer le fichier puis appliquer les modifications comme ceci:

```
ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f log_level.ldif
```

Si l'on rappel la commande pour vérifier le niveau de log, on pourra constater que la modification a bien était prise en compte et que le logLevel est passé à stats.

**Par défaut les logs se trouvent dans /var/log/syslog**, il est recommandé de n'avoir qu'un fichier pour permettre de ne voir que les logs liés à l'annuaire.
Pour ce faire, il faut <u>ajouter le fichier /etc/rsyslog.d/10-sldap.conf</u>

> [!NOTE]
>
> Il est possible que rsyslog.d ne soit pas présent, il faut alors installer le paquet rsyslog

```
# vi /etc/rsyslog.d/10-slapd.conf
Ajouter la ligne suivante :
local4.* /var/log/slapd.conf
```

Puis redémarrer le démon rsyslog pour prendre en compte les modifications.

### Ajouter une structure à son annuaire

Créer un fichier structure.ldif:

```
dn: ou=Person,dc=mon-entreprise,dc=com
objectclass: organizationalUnit
ou: Person
description: Employee de l entreprise

dn: ou=Machines,dc=mon-entreprise,dc=com
objectclass: organizationalUnit
ou: Machines
description: Ordinateurs de l entreprise

dn: cn=Marie Dupont, ou=Person, dc=mon-entreprise,dc=com
objectClass: inetOrgPerson
givenName: Marie
sn: Dupont
cn: Marie Dupont
uid: mdupont
userPassword: mDupont3*
```

Pour appliquer ces modifications, il faut s'authentifier avec l'administrateur de l'annuaire car le compte root n'a pas les droits pour modifier dans les autres arbres:

```f
ldapadd -x -W  -D "cn=admin,dc=mon-entreprise,dc=com" -H ldap://localhost -f structure.ldif
```

Ce qui donne le résultat suivant :

```
root@intrarmour:~# ldapadd -x -W -D "cn=admin,dc=intrarmour,dc=local" -H ldap://localhost -f structure.ldif
Enter LDAP Password:
adding new entry "ou=Personnes,dc=intrarmour,dc=local"

adding new entry "ou=Machines,dc=intrarmour,dc=local"

adding new entry "cn=Marie Dupond,ou=Personnes,dc=intrarmour,dc=local"

root@intrarmour:~#
```



### Sauvegarder l'annuaire

Etape 1 : Sauvegarder l'arbre

```
slapcat -b dc=intrarmour,dc=com -l mon_backup.ldif
```

Etape 2 : Sauvegarder la configuration de l'annuaire :

```
tar -cvf ma_conf_g_ldap.tar /etc/ldap
```



### Restaurer l'annuaire

Etape 1 : Eteindre slapd

```
systemctl stop slapd
```

Etape 2 : Restaurer la base à partir du fichier de sauvegarde

```
slapadd -c -b dc=intrarmour,dc=local -F /etc/ldap/slapd.d -l mon_backup.ldif
```

**-c** indique de continuer en casd 'erreur (vous voulez ajouter une entrée déjà présente, par exemple)
**-f** indique le répertoire de configuration
**-l** spécifie le ficheir de configuration LDIF à utiliser

Etape 3 : Restaurer la configuration

```
tar -xvf ma_conf_g_ldap.tar /etc/ldap
```

