# NFTABLES



nftables est un pare-feu sous linux, la dernière génération de netfilter et successeur d'iptables. Il a était développé dans le but d'améliorer les performances d'iptables .

La syntaxe d'nftables change un peu d'iptables. Par défaut iptables intègre des tables qui ne sont pas possible de supprimer (filter, nat, mangle) tandis qu'nftables au contraire n'ajoute rien par défaut, c'est à soi-même de les créer avec le nom que l'on souhaite. En fait pas tout à fait car un ficheir de configuration existe dans /etc/nftables.conf mais il est possible de modifier le nom des chaînes / tables à sa convenance. Et lorsque ce fichier est modifié, il faut alors redémarrer le service nftables pour prendre en compte les modifications.

Pour ce faire, il faut d'abord créer une table puis ajouter une chaine et les règles dans la chaine. 

Si l'on veut que les règles persistes, il est possible de les mettre dans un fichier qui sera exécuté au boot via différentes techniques (service, cron...)

```
#!/usr/sbin/nft -f

table inet filter {
	chain INPUT {
		type filter hook input priority 0; policy accept;
	}
}
```

La syntaxe d'nft est la suivante :

```
nft add table inet <table name>
```

inet : correspond au type de famille.

Il existe différent type de famille  :

- ip : IPv4 address family
- ip6 : IPv6 address family
- inet : Internet (IPv4/IPv6) address family
- arp : ARP Address family, handling IPv4 ARP Packets
- bridge : Bridge family address, handling packets which traverse a bridge device
- netdev : Netdev address family, handling packets from ingress

La syntaxe pour ajouter une chaîne est la suivante :

```
nft add chain inet <table_name> <chain_name> {
	type filter hook input priority 0; policy accept;
}
```

Pour les chaînes, il existe différents types :

| TYPE   | FAMILIES      | HOOKS                                  |
| ------ | ------------- | -------------------------------------- |
| filter | all           | all                                    |
| nat    | ip, ip6, inet | prerouting, postrouting, input, output |
| route  | ip, ip6       | output                                 |

Au niveau des priorités, il en existe plusieurs également :

| Name     | value | Families                   | Hooks       |
| -------- | ----- | -------------------------- | ----------- |
| raw      | -300  | ip, ip6, inet              | all         |
| mangle   | -150  | ip, ip6, inet              | all         |
| dstnat   | -100  | ip, ip6, inet              | prerouting  |
| filter   | 0     | ip, ip6, inet, arp, netdev | all         |
| security | 50    | ip, ip6, inet              | all         |
| srcnat   | 100   | ip, ip6, inet              | postrouting |

## Lister les composants 

pour lister l'ensemble des tables, chaînes et règles :

```
nft list ruleset
```

Pour lister uniquement les tables :

```
nft list tables
```

