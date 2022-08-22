# 0. Informations diverses
Les résultats des commandes citées dans les paragraphes qui suivent sont tirés directement du cours "Auditez la sécurité d'un système d'exploitation" donc ils peuvent varier en fonction de votre machine.

# 1. Le bootloader, les options du noyaux et ses modules

## 1.1 Introduction

Voyons la sécurité du bootloader, les options du noyau et le chargement dynamique des modules du noyau.

## 1.2 Le processus de démarrage et le bootloader
Sous Linux, le standard c'est Grub en version 2, il sert à lister les systèmes d'exploitations disponibles
sur l'ordinateur à choisir, ou encore de lancer par défaut un de ces systèmes.

    CentOS Linux (3.10)
    CentOS Linux (0-rescue)

Les deux entrées de boot concernent le système Linux CentOS mais le second est un mode dégradé (rescue).

> Grub2 dispose de son propre shell permettant d'intéragir pendant le processus de démarrage et de modifier, entre autres, les options de lancement du noyau Linux. 
> Options très utiles, mais quelqu'un avec un accès physique peut redémarrer le système et ainsi avoir accès au shell et lancer le noyau avec des options pour par exemple obtenir une invite de commande en compte privilégié.

Fichier de configuration principal : /boot/grub2/grub.cfg -> qui est un condensé mis à jour de manière dynamique, à partir de l'arborescence "/etc/grub.d".

Vérifications des droits des fichiers dans "/etc/grub.d" et propriétaires avec la commande `ls -lrtha /etc/grub.d/` :

    [root@machine ~]# ls -lrtha /etc/grub.d/
    total 84K
    -rw-r--r--. 1 root root 483 21 oct. 2017 README
    -rwxr-xr-x. 1 root root 216 21 oct. 2017 41_custom
    -rwxr-xr-x. 1 root root 214 21 oct. 2017 40_custom
    -rwxr-xr-x. 1 root root 11K 21 oct. 2017 30_os-prober
    -rwxr-xr-x. 1 root root 2,5K 21 oct. 2017 20_ppc_terminfo
    -rwxr-xr-x. 1 root root 11K 21 oct. 2017 20_linux_xen
    -rwxr-xr-x. 1 root root 11K 21 oct. 2017 10_linux
    -rwxr-xr-x. 1 root root 232 21 oct. 2017 01_users
    -rwxr-xr-x. 1 root root 8,5K 21 oct. 2017 00_header
    -rwxr-xr-x. 1 root root 1,1K 29 oct. 2017 00_tuned
    drwx------. 2 root root 182 27 mars 09:31 .
    drwxr-xr-x. 79 root root 8,0K 4 mai 13:56 ..

Normalement, c'est "root:root" pur tous les fichiers MAIS, droits 755 :

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Passez les droits sur tous les fichiers à 700.
>
> Par exemple, avec la commande suivante : `chmod -R 700 /etc/grub.d`

La valeur 700 permet la lecture, l'écriture et l'exécution uniquement par le propriétaire des fichiers, ici **root**.

De plus, dans l'arborescence, les fichiers numérotés de 10 à 40 servent principalement à configurer les propositions de démarrage présentées à l'utilisateur.

Le fichier **01_users** est codifié pour contenir les informations d'authentification qui protègent l'accès au shell de Grub.

    [root@machine ~]# cat /etc/grub.d/01_users
    #!/bin/sh -e
    cat << EOF
    if [ -f \${prefix}/user.cfg ]; then
    source \${prefix}/user.cfg
    if [ -n "\${GRUB2_PASSWORD}" ]; then
    set superusers="root"
    export superusers
    password_pbkdf2 root \${GRUB2_PASSWORD}
    fi
    fi
    EOF

Le code contenu dans ce fichier propose de créer un utilisateur root avec les droits superusers ainsi qu'un mot de passe crypté en fonction d'un fichier ${prefix}/user.cfg, qui n'existe pas par défaut.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Créez un utilisateur et son mot de passe chiffré dans le fichier **01_users** afin de protéger l'accès au shell de Grub par une authentification.
>
> Par exemple, avec les commandes suivantes : `grub2-mkpasswd-pbkdf2`, `nano /etc/grub.d/01_users` et `grub2-mkconfig -o /boot/grub2/grub.cfg`.

    [root@machine grub.d]# grub2-mkpasswd-pbkdf2
    Entrez le mot de passe :
    Entrez de nouveau le mot de passe :
    Le hachage PBKDF2 du mot de passe est grub.pbkdf2.sha512.10000.5DE21XXXXXXXXXXX

    [root@machine grub.d]# nano /etc/grub.d/01_users
    set superusers="admin"
    password_pbkdf2 admin grub.pbkdf2.sha512.10000.5DE21XXXXXXXXXXX

    [root@machine grub.d]# grub2-mkconfig -o /boot/grub2/grub.cfg

## 1.3 Options par défaut du noyau Linux

Le fichier **/boot/grub2/grub.cfg** rassemble les fichiers contenus dans **/etc/grub.d**. Consultons le fichier et affichons les options utilisées pour le lancement du noyau Linux :

    [root@machine ~]# grep linuz /boot/grub2/grub.cfg | head -1
    linux16 /vmlinuz-3.10.0-862.el7.x86_64 root=/dev/mapper/centos_fichesproduits-root ro crashkernel=auto rd.lvm.lv=centos_fichesproduits/root rd.lvm.lv=centos_fichesproduits swap rhgb quiet LANG=fr_FR.UTF-8

En plus des options liées au partitionnement LVM et configuration des variables d'environnement (langue, encodage), deux **options** sont passées au noyau : **rhgb** et **quiet**.

> Elles servent au confort de l'utilisateur car elles offrent un écran graphique pendant le démarrage, cachant ainsi la plupart des messages du noyau pendant le processus.

Ces options sont configurées dans **/etc/default/grub** :

    [root@machine grub.d]# cat /etc/default/grub
    GRUB_TIMEOUT=5
    GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
    GRUB_DEFAULT=saved
    GRUB_DISABLE_SUBMENU=true
    GRUB_TERMINAL_OUTPUT="console"
    GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos_fichesproduits/root rd.lvm.lv=centos_fichesproduits/swap rhgb quiet"
    GRUB_DISABLE_RECOVERY="true"

Dans le cadre de l'exploitation de systèmes virtualisés, il existe un service géré directement par le noyau Linux : **iommu**. Il permet de protéger la mémoire contre des accès non-contrôlés, issus des périphériques du système.

Par défaut, Linux gère iommu et selon le contexte va décider de l'activer ou non. Il est recommandé de **forcer l'activation de ce service** en passant une option supplémentaire lors du démarrage du noyau.

> 🚸 **RECOMMANDATION-WARNING** (Minimisation) : Passez l'option `iommu=force` au noyau lors du démarrage de Linux.
>
> Par exemple, en modifiant le fichier `/etc/default/grub` dans sa ligne **GRUB_CMDLINE_LINUX**, en rajoutant l'option après **quiet** :

    [root@machine ~]# nano /etc/default/grub
    GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos_fichesproduits/root rd.lvm lv=centos_fichesproduits/swap rhgb quiet iommu=force"


## 1.4 Blocage du chargement des modules Linux supplémentaires

Linux est un noyau de type monolithique modulaire, c'est-à-dire que tous les services (systèmes et utilisateurs) sont gérés dans le même espace d'adressage.

Cette fonctionnalité est très intéressante lorsqu'il s'agit de modéliser un système d'exploitation spécifique, **en ajoutant et en retirant à chaud** des modules pour notre besoin.

Mais sur une machine d'exploitation -> faille potentielle car elle permet de modifier un noyau. On vérifie qu'il est possible de charger ces modules supplémentaires avec la commande `sysctl kernel.modules_disabled` :

    [root@machine ~]# sysctl kernel.modules_disabled
    kernel.modules_disabled = 0

Par défaut, la valeur est **souvent à 0**. Donc le chargement de modules supplémentaires est **autorisé**.

> 🚸 **RECOMMANDATION-WARNING** (Minimisation) : Bloquer le chargement de modules via la commande `sysctl kernel.modules_disabled=1`.

Voici la commande pour l'exécution en cours :

    [root@machine ~]# sysctl -w kernel.modules_disabled=1
    kernel.modules_disabled = 1

Ou bien, une prise en compte pour le prochain démarrage :

    [root@machine ~]# echo "kernel.modules_disabled = 1" >> /etc/sysctl.conf

# 2. Les consoles virtuelles

## 2.1 Introduction

Elles sont les **premiers éléments d'interaction** avec l'utilisateur après le processus de démarrage.

Nous verrons comment vérifier que la **connexion root est empêchée par défaut** depuis ces consoles ou bien comment vérifier que le **processus de connexion résiste à une attaque par dictionnaire**.

Enfin, nous verrons **comment redémarrer le serveur à partir du raccourci clavier CTRL+ALT+FN** et ses risques.

## 2.2 Les accès aux consoles virtuelles

> Elles sont un système interne au noyau. Elles permettent de recevoir ou d'émettre un message sous l'apparence d'un périphérique. 
>
> Elles émulent aujourd'hui les anciennes consoles physiques pour que les opérateurs/administrateurs puissent intéragir avec les anciens systèmes (tel que le VT100).

Ces consoles sont accessibles via le raccourci **CTRL+ALT+FN** où le **n** de **FN** correspond au numéro de la console. Par défaut, Linux en propose 7 donc de F1 jusqu'à F7.

> Traditionnellement, les six premières proposent un terminal en mode texte, alors que la septième (déplacée il y a quelques années à la première place) exécute le système de fenêtre graphique (X Windows System).

Les consoles qui proposent un terminal en mode texte exécutent une tâche **getty**, qui lance en boucle le processus /bin/login permettant d'afficher le **prompt** de connexion à l'utilisateur.

Ces consoles sont accessibles à toute personne disposant d'un accès physique à la machine donc -> potentielle faille de sécurité.

Sous Linux, la gestion de la connexion console est gérée par le module **PAM** (Pluggable Authentication Module). Sa configuration est située dans **`/etc/pam.d`**.

Pour afficher la configuration du module :

    [root@machine ~]# grep "^[^#;]" /etc/pam.d/login
    auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
    auth substack system-auth
    auth include postlogin
    account required pam_nologin.so
    account include system-auth
    password include system-auth
    session required pam_selinux.so close
    session required pam_loginuid.so
    session optional pam_console.so
    session required pam_selinux.so open
    session required pam_namespace.so
    session optional pam_keyinit.so force revoke
    session include system-auth
    session include postlogin
    -session optional pam_ck_connector.so

Le processus exploite le module **pam_securetty.so**. Il s'appuie sur le fichier de configuration **`/etc/securetty`**.

    [root@machine ~]# cat /etc/securetty
    console
    vc/1
    vc/2
    ...
    ...
    hvsi1
    hvsi2
    xvc0

Ce fichier contient la liste des terminaux sous la forme de périphériques (sans le **/dev/**), donc autorisant la connexion de l'utilisateur **root**. De nombreux terminaux autorisent par défaut cette connexion. C'est donc une faille de sécurité.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Videz le contenu du fichier **/etc/securetty** afin de bloquer toute connexion avec l'utilisateur **root** depuis une console virtuelle.
>
> Par exemple, via la commande suivante :
> `echo > /etc/securetty`

Cette action empêche la connexion directe de l'utilisateur **root** depuis une console virtuelle. Il est cependant possible de se connecter avec un utilisateur moins privilégié pour ensuite passer sous l'utilisateur **root** avec la commande **`su`**.

La seconde ligne du fichier **`/etc/pam.d/login`** indique qu'un autre fichier est utilisé pour gérer le processus de connexion, notamment **`/etc/pam.d/system-auth`**.

    [root@machine ~]# cat /etc/pam.d/system-auth
    #%PAM-1.0
    # This file is auto-generated.
    # User changes will be destroyed the next time authconfig is run.
    auth required pam_env.so
    auth required pam_faildelay.so delay=2000000
    auth sufficient pam_unix.so nullok try_first_pass
    auth requisite pam_succeed_if.so uid >= 1000 quiet_success
    auth required pam_deny.so

    account required pam_unix.so
    account sufficient pam_localuser.so
    account sufficient pam_succeed_if.so uid < 1000 quiet
    account required pam_permit.so

    password requisite pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
    password sufficient pam_unix.so sha512 shadow nullok try_first_pass use_authtok
    password required pam_deny.so

    session optional pam_keyinit.so revoke
    session required pam_limits.so
    -session optional pam_systemd.so
    session [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
    session required pam_unix.so

Comme nous pouvons le voir, de nombreux modules entrent en jeu lors de l'authentification (xyz.so).

Le module **pam_faildelay.so** (deuxième ligne du fichier ci-dessus) permet de définir en millisecondes, l'intervalle minimal de temps entre chaque tentative de connexion. Sa valeur par défaut est de 2 secondes.

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Augmentez l'intervalle minimal de temps entre chaque tentative de connexion sur le module **pam_faildelay.so** du fichier **`/etc/pam.d/system-auth`** à 5-10 secondes pour ralentir les attaques par diction.

## 2.3 Le raccourci CTRL+ALT+SUPPR dangereux

Linux est le descendant d'une longue lignée d'OS. Sur Unix, l'administrateur pouvait lancer certaines commandes avec une combinaison de touches sur le clavier de la console. Elles étaient nommées **Secure Attention Key** (SAK), ou **Magic System Request Key** (MSRK).

Le raccourci CTRL+ALT+SUPPR est un héritage des anciennes commandes SAK. Elle permettait à l'époque de garantir un prompt initial sans usurpation d'identité.

Aujourd'hui, elle se traduit le plus souvent par un ordre de redémarrage de la machine -> ce qui n'est pas acceptable.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Désactivez la combinaison CTRL+ALT+SUPPR sur le serveur pour éviter un redémarrage depuis un accès physique à la machine.
>
> Par exemple, en passant la commande suivante : `ln -sf /dev/null /etc/systemd/system/ctrl-alt-del.target`

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Désactivez les Magic System Request Keys.
>
> Par exemple, avec la commande suivante : (pour l'exécution courante) `sysctl -w kernel.sysrq = 0`
>
> Pour la prise en compte au prochain démarrage : `echo "kernel.sysrq=0" >> /etc/sysctl.conf``
>
> Nous pouvons aussi recompiler le noyau sans les combinaisons associées...


# 3. Le mode de démarrage du serveur

## 3.1 Introduction

Nous allons voir les services qui sont lancés avec la machine, pour en faire une liste et suggérer d'éventuelles suppressions.

## 3.2 La cible de démarrage configurée par défaut

Le traditionnel gestionnaire de démarrage Linux **System V** se voit progressivement remplacé par le nouveau gestionnaire **systemd**, ce qui a susciter de nombreux débats.

Quoi qu'il en soit, les principales distributions (RedHat/CentOS, Debian/Ubuntu) proposent depuis leur dernière version, un mode transitoire System V basé sur systemd.

Le gestionnaire **systemd** va gérer beaucoup d'aspects de Linux. On va se concentrer uniquement sur la manière dont il gère le lancement automatique des services de la machine lors de son démarrage.

> Les **_runlevels_** de System V sont désormais remplacés par les **cibles** de systemd.

![Équivalent runlevels System V et cibles systemd](/systemv_target.png "Cibles systemd par rapport aux runlevels System V")

Sur un serveur avec System V, pour connaître le **_runlevel_** configuré par défaut, il suffit de consulter **/etc/inittab**. Sur un système dit **transitoire** le fichier existe toujours et, nous pouvons le consulter :

    [root@machine ~]# more /etc/inittab
    # inittab is no longer used when using systemd.
    #
    # ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
    #
    # Ctrl-Alt-Delete is handled by /usr/lib/systemd/system/ctrl-alt-del.target
    #
    # systemd uses 'targets' instead of runlevels. By default, there are two main targets:
    #
    # multi-user.target: analogous to runlevel 3
    # graphical.target: analogous to runlevel 5
    #
    # To view current default target, run:
    # systemctl get-default
    #
    # To set a default target, run:
    # systemctl set-default TARGET.target
    #

En effet, il faut passer à systemd. Le fichier indique également la commande à exécuter pour obtenir la **cible** par défaut de la machine :

    [root@machine ~]# systemctl get-default
    multi-user.target

La cible par défaut est **multi-user.target** qui correspond anciennement au **runlevel3**.

Obtenir la liste de toutes les cibles chargées sur le système :

    [root@machine ~]# systemctl list-units --type target
    UNIT LOAD ACTIVE SUB DESCRIPTION
    basic.target loaded active active Basic System
    cryptsetup.target loaded active active Local Encrypted Volumes
    getty.target loaded active active Login Prompts
    local-fs-pre.target loaded active active Local File Systems (Pre)
    local-fs.target loaded active active Local File Systems
    multi-user.target loaded active active Multi-User System
    network-online.target loaded active active Network is Online
    network.target loaded active active Network
    paths.target loaded active active Paths
    remote-fs.target loaded active active Remote File Systems
    slices.target loaded active active Slices
    sockets.target loaded active active Sockets
    sound.target loaded active active Sound Card
    swap.target loaded active active Swap
    sysinit.target loaded active active System Initialization
    timers.target loaded active active Timers

    LOAD = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB = The low-level unit activation state, values depend on unit type.

    16 loaded units listed. Pass --all to see loaded but inactive units, too.
    To show all installed unit files use 'systemctl list-unit-files'.

La plupart des **cibles listées ici** sont indispensables (sauf sound.target).

Il n'y a pas de recommandations à faire ici. Il pourrait y avoir la cible graphical.target, qui propose de démarrer le serveur en mode graphique.

Les cibles qui ouvrent des services pas indispensables au serveur doivent être supprimées.

Chaque **cible** définit la liste des services à lancer automatiquement lors du démarrage de la machine.

## 3.3 La cible par défaut du système

    [root@machine ~]# ls -lrtha /etc/systemd/system/multi-user.target.wants/
    total 8,0K
    lrwxrwxrwx. 1 root root 40 27 mars 09:30 remote-fs.target -> /usr/lib/systemd/system/remote-fs.target
    lrwxrwxrwx. 1 root root 46 27 mars 09:30 rhel-configure.service -> /usr/lib/systemd/system/rhel-configure.service
    lrwxrwxrwx. 1 root root 37 27 mars 09:30 crond.service -> /usr/lib/systemd/system/crond.service
    lrwxrwxrwx. 1 root root 46 27 mars 09:30 NetworkManager.service -> /usr/lib/systemd/system/NetworkManager.service
    lrwxrwxrwx. 1 root root 37 27 mars 09:30 tuned.service -> /usr/lib/systemd/system/tuned.service
    lrwxrwxrwx. 1 root root 42 27 mars 09:30 irqbalance.service -> /usr/lib/systemd/system/irqbalance.service
    lrwxrwxrwx. 1 root root 39 27 mars 09:30 rsyslog.service -> /usr/lib/systemd/system/rsyslog.service
    lrwxrwxrwx. 1 root root 37 27 mars 09:30 kdump.service -> /usr/lib/systemd/system/kdump.service
    lrwxrwxrwx. 1 root root 36 27 mars 09:30 sshd.service -> /usr/lib/systemd/system/sshd.service
    lrwxrwxrwx. 1 root root 38 27 mars 09:30 auditd.service -> /usr/lib/systemd/system/auditd.service
    lrwxrwxrwx. 1 root root 39 27 mars 09:30 postfix.service -> /usr/lib/systemd/system/postfix.service
    lrwxrwxrwx. 1 root root 39 27 mars 09:30 chronyd.service -> /usr/lib/systemd/system/chronyd.service
    lrwxrwxrwx. 1 root root 39 27 mars 09:46 mariadb.service -> /usr/lib/systemd/system/mariadb.service
    lrwxrwxrwx. 1 root root 37 27 mars 09:46 httpd.service -> /usr/lib/systemd/system/httpd.service
    lrwxrwxrwx 1 root root 39 27 mars 09:50 proftpd.service -> /usr/lib/systemd/system/proftpd.service
    drwxr-xr-x. 11 root root 4,0K 27 mars 09:51 ..
    drwxr-xr-x. 2 root root 4,0K 27 mars 09:51 .

On peut constater la présence de services qui ne sont pas indispensables au serveur, comme **postfix.service**, **chronyd.service** ou encore **tuned.service**.

Il faut stopper ces services inutiles aux **contextes fonctionnel et opérationnel du serveur**. Rien de plus frustrant qu'un serveur corrompu par un/des services inutiles.

> ❌ **RECOMMANDATION-CRITICAL** (Minimisation) : Supprimez les services inutiles démarrés automatiquement avec le serveur en passant par la cible par défaut.
>
> Par exemple, avec la commande suivante : **`systemctl stop postfix.service`** (pour l'exécution courante)
>
> Pour le lancement automatique de la cible multi-user.target : **`systemctl disable postfix.service`**

La cible **multi-user.target** configure elle-même des dépendances dans son fichier de config :

    [root@machine ~]# cat /lib/systemd/system/multi-user.target
    # This file is part of systemd.
    #
    # systemd is free software; you can redistribute it and/or modify it
    # under the terms of the GNU Lesser General Public License as published by
    # the Free Software Foundation; either version 2.1 of the License, or
    # (at your option) any later version.

    [Unit]
    Description=Multi-User System
    Documentation=man:systemd.special(7)
    Requires=basic.target
    Conflicts=rescue.service rescue.target
    After=basic.target rescue.service rescue.target
    AllowIsolate=yes

Le mot-clé **Requires** indique ici que la cible **basic.target** sera exécutée avant **multi-user.target**. Elle même a des dépendances :

    [root@machine ~]# cat /lib/systemd/system/basic.target
    # This file is part of systemd.
    #
    # systemd is free software; you can redistribute it and/or modify it
    # under the terms of the GNU Lesser General Public License as published by
    # the Free Software Foundation; either version 2.1 of the License, or
    # (at your option) any later version.

    [Unit]
    Description=Basic System
    Documentation=man:systemd.special(7)

    Requires=sysinit.target
    After=sysinit.target
    Wants=sockets.target timers.target paths.target slices.target
    After=sockets.target paths.target slices.target

Les mots-clés **After** et **Wants** indiquent les dépendances. Se référer à la documentation de systemd pour voir les différents types de dépendance. (https://www.freedesktop.org/software/systemd/man/systemd.unit.html)

Quoi qu'il en soit, on va lister l'ensemble des services et leur status actuel :

    [root@machine ~]# systemctl list-unit-files --type service
    UNIT FILE STATE
    auditd.service enabled
    autovt@.service enabled
    blk-availability.service disabled
    brandbot.service static
    chrony-dnssrv@.service static
    chrony-wait.service disabled
    chronyd.service enabled
    console-getty.service disabled
    console-shell.service disabled
    container-getty@.service static
    cpupower.service disabled
    crond.service enabled
    dbus-org.freedesktop.hostname1.service static
    ...

Tous les services avec le statut **enabled** seront lancés automatiquement sur le serveur.

> 🚸 **RECOMMANDATION-WARNING** (Minimisation) : Supprimez les services inutiles démarrés automatiquement en passant par les dépendances de la cible par défaut.


# 4. Résumé des recommandations de toute cette partie

| Recommandation      | Type | Principe   |
| ----------- | ----------- | -----------|
| Passer les droits sur l'arborescence **/etc/grub.d/** à **700**      | ❌ CRITICAL       | Moindre privilège |
| Créer un user et son **mot de passe chiffré** dans le fichier **01_users** afin de protéger l'accès au `shell` de GRUB2 par une authentification  | ❌ CRITICAL        | Minimisation |
| Passer l'option **iommu=force** au noyau lors du démarrage de Linux      | 🚸 WARNING       | Minimisation |
| Bloquer le chargement de modules supplémentaires via la commande **sysctl kernel.modules_disabled=1**   | 🚸 WARNING        | Minimisation |
| Vider le contenu du fichier **/etc/securetty** afin de bloquer toute connexion avec l'utilisateur root depuis une console virtuelle      | ❌ CRITICAL       | Défense en profondeur |
| Augmenter l'intervalle minimal de temps entre chaque tentative de connexion sur le module **pam_faildelay.so** du fichier **/etc/pam.d/system-auth** à 5 ou 10 secondes afin de ralentir les attaques par dictionnaire   | 🚸 WARNING        | Défense en profondeur |
| Désactiver la combinaison **Ctrl+Alt+Supp** sur le serveur pour prévenir tout redémarrage depuis un accès physique à la machine      | ❌ CRITICAL       | Défense en profondeur |
| Désactiver les **Magic System Request Keys**   | 🚸 WARNING        | Défense en profondeur |
| Supprimer les **services inutiles** démarrés automatiquement avec le serveur en passant par **la cible par défaut**      | ❌ CRITICAL       | Minimisation |
| Supprimer les **services inutiles** démarrés automatiquement avec le serveur en passant par **les dépendances de la cible par défaut**   | 🚸 WARNING        | Minimisation |