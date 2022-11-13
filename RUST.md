# RUST



## Installation

### Windows

Installation de l'executable rust : https://www.rust-lang.org/tools/install

Installation de Visual Studio : https://visualstudio.microsoft.com/fr/downloads/

Prendre la derniière version et durant le setup, ajouter les dépendances pour C++

### Linux & MacOS

Pour installer rustup

```bash
<<<<<<< HEAD
$ curl --proto "=https" --tlsv1.2 https://sh.rustup.rs -sSf | sh
=======
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
>>>>>>> 3b94ad5e0a11761c19a4b6fbf7ddd5d97e2ff84a
```

### Linker

**Il faudra également un linker**

C'est un programme que RUST utilise pour regruoper ses multiples résultats de compliation dans un même fichier

#### Linux

Installer GCC ou CLANG selon la distribution

#### MacOS

```bash
$ xcode-select --install
```

## Compiler

### Rustrc

```bash
$ cd myproject
$ rustrc main.rs
$ ./main
```

### Cargo

```bash
$ cd myproject
$ cargo build --release
Compiling hello_cargo v0.1.0 (C:\Users\loulax\Documents\rust\hello_cargo)
Finished release [optimized] target(s) in 0.23s
```

## Commandes initiales

### Version de cargo

```bash
$ cargo --version
```

### Créer projet

```bash
$ cargo new project_name
Created binary (application) `hello_cargo` package
```

Lorsqu'un nouveau projet est créé, l'architecture suivante sera crée :

- **Un fichier Cargo.toml** : Dedans se trouvera plusieurs informations ou ressources utiles au projet 
  - Le nom du projet
  - La version du projet
  - Un lien vers une doc
  - Et surtout les dépendances du projet, les librairies à inclure si nécessaire pour son bon fonctionnement
- **Un fichier .gitignore** qui va créer un nouveau dépot git
- **Un dossier src** : C'est dans celui-ci que se trouve tous les fichiers de développement du projet

### Vérifier un programme

Lorsque l'on veut vérifier que le code du programme ne contient pas d'erreur avant de l'exécuter, il faudra le faire avec la commande :

```bash
$ cargo check
Finished dev [unoptimized + debuginfo] target(s) in 0.00s
```

### Exécuter son programme

Maintenant que tout est vérifier et ok, saisir la commande suivante pour exécuter le programme:

```bash
$ cargo run
Compiling hello_cargo v0.1.0 (C:\Users\loulax\Documents\rust\hello_cargo)
Finished dev [unoptimized + debuginfo] target(s) in 0.21s
Running `C:\Users\loulax\Documents\rust\hello_cargo\target\debug\hello_cargo.exe`
Hello, world!
```

