# CHROOT



chroot est un environnement sur linux qui permet d'isoler un système ou des services. Il suffit de créer un dossier dans lequel se trouvera le système puis mettre en place les outils pour l'isoler.

Afin que chaque services fonctionnent, il faut bien évidemment avoir le chemin de chaque binaires pour les services "chrootés".

## Installation

Pour la partie installation plusieurs méthodes sont possibles :

#### Installation manuelle:

Il faudra choisir un dossier dans lequel installer le système isolé avec l'arborescence nécessaire pour les commandes/services souhaités. Pour l'exemple, je vais créer un dossier dans /chroot/srv

```bash
mkdir -p /chroot/srv
```

Maintenant je veux que dans mon système chrooté, je puisse utiliser certaines commandes telles que ls, cat, cp.... Et pour ça, il me faut les bibliothèques de chaque binaire. On peut voir les bibliothèques nécessaires via la commande ldd.

Déjà je vais vérifier le chemin relatif à la commande ls :

```bash
root@debian-gnu-linux-11:~# whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

Ensuite je récupère les bibliothèques via le chemin absolu

```bash
root@debian-gnu-linux-11:~# ldd /usr/bin/ls
	linux-vdso.so.1 (0x0000ffff93a6e000)
	libselinux.so.1 => /lib/aarch64-linux-gnu/libselinux.so.1 (0x0000ffff939ba000)
	libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000ffff93846000)
	/lib/ld-linux-aarch64.so.1 (0x0000ffff93a3e000)
	libpcre2-8.so.0 => /lib/aarch64-linux-gnu/libpcre2-8.so.0 (0x0000ffff937b4000)
	libdl.so.2 => /lib/aarch64-linux-gnu/libdl.so.2 (0x0000ffff937a0000)
	libpthread.so.0 => /lib/aarch64-linux-gnu/libpthread.so.0 (0x0000ffff9376f000)
```

Ensuite je prépare l'arborescence nécessaire dans mon futur système chrooté:

```
mkdir -p /chroot/srv/usr/bin
mkdir -p /chroot/srv/lib/aarch64-linux-gnu/
```

puis je copie les différentes bibliothèques dans leur dossier:

```
cp /usr/bin/ls /chroot/srv/usr/bin
cp /lib/aarch64-linux-gnu/libselinux.so.1 /lib/aarch64-linux-gnu/libc.so.6 /lib/aarch64-linux-gnu/libpcre2-8.so.0 /lib/aarch64-linux-gnu/libdl.so.2 /lib/aarch64-linux-gnu/libpthread.so.0 /chroot/srv/lib/aarch64-linux-gnu/ 
cp /lib/ld-linux-aarch64.so.1 /chroot/srv/lib
```

On va faire aussi pour bash afin d'avoir le shell:

```
cp /usr/bin/bash /chroot/srv/usr/bin
cp /lib/aarch64-linux-gnu/libtinfo.so.6 /lib/aarch64-linux-gnu/libdl.so.2 /lib/aarch64-linux-gnu/libc.so.6 /chroot/srv/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 /chroot/srv/lib/
```



#### Accès à notre système chroot

Pour accéder à notre système isolé, il faut utiliser la commande chroot de la manière suivante : chroot <path directory install>

```bash
root@debian-gnu-linux-11:~# chroot /chroot/srv/
bash-5.1# 
bash-5.1# ls
bin  lib  usr
bash-5.1# ls -alh
total 20K
drwxr-xr-x 5 0 0 4.0K Jul  7 07:57 .
drwxr-xr-x 5 0 0 4.0K Jul  7 07:57 ..
drwxr-xr-x 2 0 0 4.0K Jul  7 07:57 bin
drwxr-xr-x 3 0 0 4.0K Jul  7 07:44 lib
drwxr-xr-x 3 0 0 4.0K Jul  7 07:37 usr
bash-5.1#
```

Pour revenir sur le système précédent, il suffit de faire exit.

**Il existe sur internet une multitude de script bash permettant d'automatiser l'ajout des commandes rapidement et monter un système chroot sans tout faire manuellement.**

#### <u>Méthode debootstrap</u>

Debootstrap est un paquet qui s'installe sur les systèmes basé debian. Il installe un système complet et après à nous de supprimer les binaire etc dont on a pas besoin pour alléger le système isolé et améliorer la sécurité de ce dernier.

La commande debootstrap s'utilise de la manière suivante : debootstrap <version debian> <install path>

```
apt -y install debootstrap
mkdir -p /chroot/srv
debootstrap bullseye /chroot/srv/
chroot /chroot/srv
```