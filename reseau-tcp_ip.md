# Mettez en place et documentez le réseau d'une startup



## Le fonctionnement de TCP et ses "Headers"

Une entête TCP fait 20 octets initialement comme pour IP. Voici le tableau d'une entête TCP. Chaque ligne est codé sur 32bits (4 octets) puisqu'un octet vault 8 bits

![](images/header_tcp.jpg)

**<u>Version (4bits)</u>** : Indique la version d'IP 4 ou 6
<u>**HLEN (Header Length)**</u> : 20 Octets. A savoir que cette valeur est dynamique selon les options ajoutées dans l'entête.
**<u>TOS (Type of Service) 8 bits</u>**: Ce champs était prévu initialement pour donner une priorité aux paquets mais n'est plus très utilisé aujourd'hui.
 **<u>Total length (16bits)</u>**: Il représente la longueur total d'octets présents dans le datagramme IP.
<u>**IPID : **</u> C'est un identifiant qui permet d'identifier de quel datagramme d'origine proviennent les fragments. Chaque fragment aura un ID et au moment du réassemblage c'est ce dernier qui aidera le destinataire à retrouver l'ensemble des fragments.
<u>**Flags:**</u>
<u>**Fragments offset:**</u> 
<u>**TTL (Time to Live, 8bits) :**</u> Ce champs est très très utilisé pour permettre de supprimer un paquet lorsque celui-ci est perdu dans le réseau. Chaque fois qu'il passe un noeud, il décrémenté de 1. 
**Protocol 4:** Ce champs représente l'indication du protocole de couche 4 utilisé lors de l'envoi. Généralement TCP/UDP.
<u>**Checksum:**</u> La somme de contrôle sert à s'assurer que les informations contenues dans le paquet n'ont pas étaient altérées durant son cheminement. Il suffit qu'il y ait un seul bit de changé pour qu'une erreur dans le paquet soit reconnue.

On peux constater que les paquets IP sont fragmentés car la norme ethernet limite la taille d'un paquet à 1518 octets alors qu'une entête peux avoir 65535 octets.

La fragmentation consiste donc à découper un datagramme en plusieurs morceaux qui seront réassemblés sur la machine de destination. La taille des fragments sera toujours de la plus grande taille possible afin de limiter au maximum son nombre. Une valeur est définie par la <u>**MTU (maximum transmission unit).**</u> Il est possible que chaque paquets ne prennent pas le même chemin et dans le bon ordre.

Exemple de fragmentation :

Un datagramme de 1600 octets est envoyé. Puisque chaque fragments doivent faire une taille la plus élevé possible.

Pour illustrer l'exemple on a un datagramme de 5580 octets de données entre 2 réseaux et passent par un routeur.
Le MTU de chaque réseau est de 1500 octets. Il faut prendre en compte la taille de l'entete de 20 octets;

Chaque fragments pourra donc contenir 1480 octets de données. Le calcul sera le suivant:

5580 / 1480 = 3.7

Il va donc falloir 3 fragments de 1480 octets et pour connaître la taille du 4ème fragment:

5580 - (1480 * 3) = 5580 - 4440 = 1140 octets. 



## Scan de port

Un port est un numéro assigné à une application sur le réseau qui permet de faire communiquer chaque hôtes.
