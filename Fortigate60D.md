# Fortigate 60D

## Introduction

Fortigate est un routeur/firewall de chez Fortinet qui propose un large panel de  fonctionnalités pour toutes tailles d'entreprises.

Beaucoup de modèles sont disponibles selon les besoins et les prix varient en conséquences.

## Reset to factory default

Lorsque le mot de passe admin est perdu ou une mauvaise configuration a était faite, il est parfois nécessaire de réinitialiser le boîtier à sa configuration d'usine.

**Prérequis** : Posséder un câble console, une aiguille puis l'alimentation et un logiciel tel que putty pour se connecter en console dessus.

<u>Pour ce faire voici la procédure à suivre rigoureusement:</u>

1. Brancher le câble série (partie usb vers pc) et rj45 sur le port console à l'avant.

2.  Lancer putty et sélectionner le mode de connexion "Serial". Il faudra se rendre une fois le boîtier allumé dans le gestionnaire de périphérique (sous windows) pour voir sur quel port com le boîtier est joignable.

3. Brancher l'alimentation

4. Saisir dans putty le numéro du port indiqué

5. Attendre que le système boot jusqu'à voir le prompt.

6. Avec l'aiguille ou n'importe quel objet très fin, appuyer sur le bouton "reset" qui se trouve à l'arrière pendant 20s jusqu'à voir resetting system. Attendre un petit peu le temps que le boitier redémarre. Une fois que le prompt est de nouveau affiché, l'os est reconfiguré à ses paramètres d'usine. 

   Les identifiants par défauts sont user => admin / password => vide

   **Si au moment d'appuyer sur le bouton reset à l'arrière , un message indique que celui-ci est désactivé, il va falloir débrancher le boîter de son alimentation puis le rebrancher et appuyer sur le bouton reset durant le boot pendant 60s. Ensuite procéder à l'étape 6.**

   



