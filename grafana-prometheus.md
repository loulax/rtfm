# PROMETHEUS / GRAFANA

## Introduction

Ce sont deux outils opensource et permettent de faire du monitoring et de l'alerte sur une infrastructure. Ils sont très utilisés dans le monde de l'entreprise pour consulter en temps réel les logs, les status des services etc

## Installation

Comme dit au dessus, ils sont opensource et disponibles sur pratiquement toutes les distributions Linux. 
Je vais monbtrer ici comment les installer sur Debian/Ubuntu:

Ouvrir une console av ec les privilèges administrateur (root) puis saisir les commandes suiavntes:

```
sudo apt -y install prometheus
```

Grafana:

```
apt -y install -y apt-transport-https software-propperties-common wget
mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Une fois que les dépots sont ajoutés, il va falloir les mettre à jour puis enfin installer le paquet Grafana:

```
apt update && apt -y install grafana
```

Pour la version entreprise :

```
apt -y install grafana-enterprise
```

Les services se gèrent directement par systemd pour prometheus:

```
systemctl status | start | stop prometheus
```

Pour Grafana :

```
systemctl status | start | stop grafana-server
```

