# Packet Tracer

## Console

Accéder à la liste d'aide, saisir la commande **help** . Celle-ci est accessible dans tous les modes d'exécution.

Elle permet d'affiche la liste des commandes accessibles dans le mode actuel avec une petite description de ce qu'elle fait.

### Les modes d'exécution

- **Exécution utilisateur** : Ce mode offre des fonctionnalités limitées mais utile pour les opérations de base. Il autorise seulement un nombre de commandes limitées pour la surveillance de base mais aucune susceptible de modifier la configuration du périphérique. Ce mode est reconnu par le symbole **>** qui termine le prompt.
- **Exécution privilégié / actif** : Ce mode est utile pour les administrateurs réseau. Il permet d'accéder à des configurations avancées pour modifier certains paramètres. Il est reconnu par le symbole **#** qui termine le prompt. Pour accéder à ce mode, saisir la commande suivante.

```
switch> enable
switch#
```

- **Configuration de ligne** : Utilisé pour l'accès par la console, ssh, telnet ou AUX. L'invite par défaut pour le mode de configuration de ligne est **Switch(config-line)#**.

```
switch(config)# line console <numero>
```

- **Configuration d'interface** : Utilisé pour configurer l'interface réseau d'un port (ou plusieurs ports) d'un switch ou d'un routeur. Le prompt par défaut est **Switch(config-if)#**. 

```
switch(config)# interface <type><number>
- Ex : int Fa0/1
- int g0/1
- int range fa0/1 - 5
switch(config-if)#
switch(config-if-range)#
```

### Accès mode privilégié

Afin de pouvoir accéder au mode privilégié de la console, il faut saisir la commande suivante. Dans certains cas, un mot de passe est nécessaire.

```
enable / en
```

Pour repasser au mode utilisateur

```
disable
```

Pour passer en mode configuration globale .

```
configure terminal / conf t
```

## Sauvegarde

Lorsque le switch est configuré correctement, afin de ne pas perdre ses paramètres au prochain démarrage, il est possible de sauvegarder la config pour qu'elle soit prise en compte à chaque démarrage.

```
Switch# copy running-config startup-config
```

## Accès sécurisé

Afin de limiter de limiter les accès au différent modes et à distance, il est fortement recommandé de configurer un mot de passe.

### Mode privilégié

```
Switch(Config)# enable secret <password>
```

### SSH

Dans la plupart des cas, il sera constaté que pour l'accès ssh toutes les lignes virtuelles de 0 à 15 seront dédié au protocole ssh pour des mesures de sécurité afin d'empêcher le telnet

```
Switch(config)# ip domain-name <domain-name>
Switch(config)# ip ssh version 2
Switch(config)# crypto key generate rsa
How many bits in the modulus [512]: <key bit size>
Switch(config)# username admin secret <password>
Switch(config)# line vty 0 15
Switch(config-line)# transport input ssh
Switch(config-line)# login local
Switch(config-line)# exit
```

## Vlan

Par défaut les switchs cisco ont un vlan 1 déjà configuré sur lequel tous les appareils connecté au switch est relié. Celui-ci n'est ni modifiable, ni supprimable.

Pour créer un vlan et le configurer voici les commandes:

```
Switch(config)# vlan <number>
Switch(config-vlan)# name <name>
Switch(config-vlan)# exit
Switch(config)# int <int number>
Switch(config)# switchport mode <trunk><access>
Switch(config)# switchport access vlan <number>
```

