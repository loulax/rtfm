# SECURISATION RÉSEAU (VPN - FIREWALL)



Un pare-feu est un moyen très largement répendu principalement dans le monde de l'enterprise pour permettre de filtrer tout le traffic qui transite entre le réseau WAN extérieur à l'entreprise et le réseau interne ainsi que entre les différents réseaux de l'entreprise (pour les grosses sociétés).

Il permet d'autoriser, bloquer les flux selon différents critères (ip, port, protocole, type de connexion, pays, application). Il permet également de rediriger un flux d'une destination vers une autre

## Définition d'une politique

Avant de commencer à configurer un pare-feu, car par défaut il ne possède pas de règles préalablement configurés, il faut définir la politique qui lui sera appliqué. Celle-ci dépendra du besoin de l'entreprise donc c'est une étape qui n'est pas à négliger.

> [!NOTE]
>
> Généralement par défaut, il est fortement recommander de bloquer tout le traffic entrant / sortant puis d'autoriser les flux nécessaires. Cela évite des ouvertures non désirées et potentiellement des vulnérabilités exploitables.



