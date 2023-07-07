# Github

## Créer un repository

Nous allons voir dans cette section comment initialiser un répertoire puis envoyer les modifications à l'aide d'une authentification ssh.

Il faudra au préalable créer sa clé ssh et la mettre dans son compte git.

### Créer sa clé ssh

Afin de pouvoir travailler de manière sécurisé sur son compte github sans devoir saisir ses information d'identification systématiquement, il va falloir générer une pair de clé qui chiffrera les communications entre votre ordinateur local et votre compte github. Le protocole ssh est conçu pour ça et nous allons voir comment configurer tout ça.

```bash
# Dans une console saisir les commandes suivantes
$ ssh-keygen -t rsa -b 4096 (taille de la clé à choisir)
➜  ~ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/<username>/.ssh/id_rsa):  # Laisser le nom par défaut autrement git risque de ne pas pouvoir lire la clé et vous retourner une erreur lorsque vous effecturez des actions sur votre compte
Enter passphrase (empty for no passphrase): # Vous pouvez créer une passphrase pour accéder à cette clé (optionnel)!
Enter same passphrase again: 
Your identification has been saved in id_rsa
Your public key has been saved in id_rsa.pub
The key fingerprint is:
SHA256:alYwZMaLY1PQRKR+LZIJ6EmKnPgk9Dr9Pi4mWWyTaYE <myhostname>.local
The key's randomart image is:
+---[RSA 4096]----+
|    .BB          |
| .   =+          |
|.oo .oo.         |
|OE++=o.+         |
|==ooOoo S        |
| +oO o +         |
| o*.. +          |
| o.o.+           |
|  o ++.          |
+----[SHA256]-----+
➜  ~ 
$ git config --add --global core.sshCommand "ssh -i <ssk_key_path>"
$ git config --global user.email "<your email>"
$ git config --global user.name "<your name>"
```

Voilà, nous avons notre clé. Maintenant il va falloir récupérer le contenu de la clé publique pour pouvoir la mettre sur notre compte.

### Enregistrer sa clé dans le compte

```bash
# Récupérer le contenu de la clé publique
$ cat ~/.ssh/id_rsa.pub
```

Pour pouvoir faire le lien maintenant entre ses repos local (sur votre ordinateur) et votre compte GitHub, il va falloir renseigner à github la clé qui permettra de vérifier l'authenticité des communications.

Pour ce faire rendez-vous dans votre compte, aller en haut à droite sur votre profil puis paramètre > SSH et clés GPG.

Cliquer sur le bouton vert "Ajouter une clé SSH" puis renseigner le nom à donner pour la clé (car on peut rajouter autant de clé que l'on souhaite) puis coller en dessous le contenu de la clé publique.

**La clé ssh commence à partir de ssh-rsa jusqu'au nom de votre ordinateur**

### Initialiser son repo

```bash
# Dans une console saisir les commandes suivantes
$ mkdir myrepo && cd myrepo
$ git init
$ echo "Description du repo" >> README.md
$ git add .
$ git commit -m "first commit"
$ git branch -M origin main
$ git remote add origin https://github.com/<my username>/<repo>.git
$ git remote set-url origin git@github.com:<my username>/<repo>.git
$ git push -u origin main
# Si tout s'est bien passé vous devriez voir ceci
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 1.01 KiB | 1.01 MiB/s, done.
Total 5 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:<username>/myrepo>.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

### Mettre à jour un repo

Ok maintenant nous avons notre répertoire d'initialiser. Maintenant vous souhaitez apporter des modifications à celui-ci.

```bash
# Dans une console se rendre dans le répertoire du repository
$ cd myrepo
# Ajouter les modifications
$ git add .
# Laisser un message pour notifier des modifications
$ git commit -m "modif..."
# Envoyer les modifications sur le dépot distant
$ git push -u origin main
```

### Cloner un repo

#### Méthode SSH

```bash
$ git clone git@github.com:<username>/<repo>.git
Cloning into 'iptables'...
Enter passphrase for key '/c/Users/<username>/.ssh/id_rsa':
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (5/5), done.
Receiving objects: 100% (5/5), done.
remote: Total 5 (delta 0), reused 5 (delta 0), pack-reused 0
```

#### Méthode HTTP

```bash
$ git clone https://github.com/<username>/<repo>.git
```

