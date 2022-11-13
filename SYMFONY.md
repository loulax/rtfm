# SYMFONY



Symfony est un framework PHP utilisé pour développer des sites rapidement, il utilise composer pour être installer et fonctionner globalement.

## Installation

Il va falloir dans un 1er temps installer les dépendances nécessaires 

### Installation du serveur web

```bash
$ apt -y install apache2 libapache2-mod-php php php-mysql mariadb-server php-curl php-xml
```

### Installation de symfony

```bash
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php -r "if (hash_file('sha384', 'composer-setup.php') === '93b54496392c062774670ac18b134c3b3a95e5a5e5c8f1a9f115f203b75bf9a129d5daa8ba6a13e2cc8a1da0806388a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"
```

## Start-up

Afin de démarrer un projet symfony, il existe 2 squelettes de base 

- skeleton : le squelette minimaliste pour démarrer un projet PHP, il est recommandé pour des applications en ligne de commande, par exemple.
- website-skeleton : le squelette recommandé pour faire des projets web, que je vous recommande comme base de travail.

### Démarrer un projet

#### Symfony-cli

```bash
$ symfony new --full mon-super-projet
```

#### Composer

```bash
$ composer create-project symfony/website-skeleton mon-super-projet
```

### Démarrer le serveur

```bash
$ cd mon-super-projet
$ symfony server:start
```

Afin d'accéder à l'interface web, rendez-vous à l'adresse suivante : http://localhost:8000

## Architecture 

Lorsque l'on démarre le projet initial, voici son architecture.

```bash
$ tree -L 2
.
├── bin
│   ├── console
│   └── phpunit
├── composer.json
├── composer.lock
├── config
│   ├── bundles.php
│   ├── packages
│   ├── preload.php
│   ├── routes
│   ├── routes.yaml
│   └── services.yaml
├── docker-compose.override.yml
├── docker-compose.yml
├── migrations
├── phpunit.xml.dist
├── public
│   └── index.php
├── src
│   ├── Controller
│   ├── Entity
│   ├── Kernel.php
│   └── Repository
├── symfony.lock
├── templates
│   └── base.html.twig
├── tests
│   └── bootstrap.php
├── translations
├── var
│   ├── cache
│   └── log
└── vendor
    ├── autoload.php
    ├── autoload_runtime.php
    ├── bin
    ├── composer
    ├── doctrine
    ├── egulias
    ├── friendsofphp
    ├── laminas
    ├── masterminds
    ├── monolog
    ├── myclabs
    ├── nikic
    ├── phar-io
    ├── phpdocumentor
    ├── phpstan
    ├── phpunit
    ├── psr
    ├── sebastian
    ├── sensio
    ├── symfony
    ├── theseer
    ├── twig
    └── webmozart

38 directories, 18 files
```

### Le dossier "bin"

Ce dossier contient les exécutables disponibles dans le projet, que ce  soit ceux fournis avec le framework (la console Symfony) ou ceux des  dépendances (phpunit, simple-phpunit, php-cs-fixer, phpstan).

### Le dossier "config"

Ce dossier a fait l'objet d'un chapitre complet de ce cours. Il contient  toute la configuration de votre application, que ce soit le framework,  les dépendances (Doctrine, Twig, Monolog) ou encore les routes.

Ne pas oublier qu'il est possible d'adapter la configuration du framework  en fonction de l'environnement, et qu'une partie de la configuration se  trouve aussi dans le fichier .env du projet.

### Le dossier "public"

Par défaut, il ne contient que le contrôleur frontal de votre application,  le fichier dont la responsabilité est de recevoir toutes les requêtes  des utilisateurs.

Seul ce dossier doit être accessible de l'extérieur.

### Le dossier "migrations"

Dans ce dossier et si vous manipulez une base de données, alors vous  trouverez les migrations de votre projet générées à chaque changement  que vous effectuerez sur votre base de données à l'aide de l'ORM  Doctrine. Nous reviendrons sur ce dossier dans le chapitre "Gérez votre  base de données avec Doctrine ORM".

### Le dossier "src"

C'est ici que se trouve votre application ! Contrôleurs, formulaires,  écouteurs d'événements, modèles et tous vos services doivent se trouver  dans ce dossier. C'est également dans ce dossier que se trouve le  "moteur" de votre application, le kernel.

### Le dossier "tests"

Dans ce dossier se trouvent les tests unitaires, d'intégration et d'interfaces.

Par défaut, l'espace de nom du dossier **tests** est  `App\Tests` et celui du dossier **src** est  `App` .

### Le dossier "templates"

Ce dossier contient les gabarits qui sont utilisés dans votre projet. Par exemple, si dans un contrôleur on fait :

```php
<?php

$this->render('foo.html.twig');
```

### Le dossier "Translation"

Symfony fournit un composant appelé [Translation](https://symfony.com/doc/current/components/translation.htm) capable de gérer de nombreux formats de traductions, dont les formats yaml,  xliff, po, mo... Ces fichiers seront situés dans ce dossier.

### Le dossier "var"

Ce dossier contient trois choses principalement :

- les fichiers de cache dans le dossier **cache** ;
- les fichiers de log dans le dossier **log** ;
- et parfois, si le framework est configuré pour gérer les sessions PHP dans le système de fichiers, on trouve le dossier **sessions.**

### Le dossier "vendor"

Ce dossier contient votre chargeur de dépendances (ou "autoloader") et  l'ensemble des dépendances de votre projet PHP installées à l'aide de  Composer. Une autre façon de découvrir vos dépendances est d'utiliser la commande "composer show".

## Symfony Flex

Il est pour gérer la configuration de vos  applications. Très simplement, c'est une extension de Composer qui va  effectuer des actions supplémentaires à l'installation ou à la  désinstallation d'une dépendance. Flex sait quelles actions réaliser  grâce à la notion de "recettes". Dans celles-ci, les responsables de  projets définissent quelles actions complémentaires (ajouter de la  configuration, créer un ou des fichiers, etc...) devront être réalisées.

## Contrôleur front

PHP étant un langage serveur, il est systématiquement utilisé pour interagir entre l'utilisateur et le serveur. Le contrôleur front va permettre de faire le lien entre le client et le serveur.

Il utilise le composant HttpFoundation qu'il faudra appeler pour faire les requêtes/réponses