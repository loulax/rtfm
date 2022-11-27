# KIVY

## Introduction

Kivy est un module python qui permet de créer des interfaces graphiques facilement sous windows, mac et linux. Il est possible d'utiliser l'application finale pour ordinateur, tablette ou smartphone. Pour l'installer, il suffit d'ouvrir un terminal et saisir la commande suivante:

<u>Sous windows</u>

```
pip install kivy
```

<u>Sous Linux et mac</u>

```
pip3 install kivy
```

Sa documentation officielle est disponible à cette adresse: https://kivy.org/doc/stable/.

## Configuration

Afin de configurer son interface, kivy utilise un fichier avec l'extension .kv. Celui-ci doit se trouver absolument dans le même répertoire que l'application principale et nommé exactement comme le nom de la classe dans le fichier python qui possède les fonctions de l'application.

Exemple :

<u>main.py</u>

```python
from kivy.app import App

class MyApp():
    
    pass
```

<u>MyApp.kv</u>

### Les layouts

Cette partie permet d'agencer les éléments graphique de manière à être adaptés à la taille d'écran etc.  Il en existe 9 mais les priincipaux sont les suivants:

1 - Box Layout : Il propose une disposition verticale ou horizontale des éléments.

2 - AnchorLayout: Il permet de disposer les éléments de manière à les rangers sois dans les coins de l'écran, soit au milieu.

3 - GridLayout : Il permet d'organiser les éléments soit par ligne, soit par colonne.

4 - StackLayout : Il permet de disposer les éléments sur plusieurs ligne et de tailles différentes que le BoxLayout.

5 - ScrollView : Utilisé lorsqu'un élément est trop grand pour l'écran. Cela permet à l'utilisateur de scroller de manière verticale ou horizontale l'élément.

6 - PageLayout : Utile pour donner l'impression d'avoir un livre et tourner les pages.

7 - FloatLayout : Très similaire au BoxLayout

8 - RelativeLayout : Utilisé avec les canvas

9 - ScatterLayout : Modèle très spécifique et très peu utilisé



Pour positionner des éléments type box il existe les propriétés suivantes:

pos_hint :  (Verticalement) => y, top, center_y.

Exemple: 

```kotlin
pos_hint: { "top": 1, y: 0.5, center_y: 1.5}
```

Pour se référencer la position initiale 0 se trouve en bas de l'écran.

pos_hint: (horizontalement) => x, right, center_x

Pour se référencer la position initiale 0 se trouve à gauche de l'écran. 

Pour les boutons:

pos: "200dp", "100dp"

