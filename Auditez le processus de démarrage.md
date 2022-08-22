# 1. Le bootloader, les options du noyaux et ses modules

### Introduction

## Le processus de démarrage et le bootloader

## Options par défaut du noyau Linux

## Blocage du chargement des modules Linux supplémentaires


# 2. Les consoles virtuelles

### Introduction

Elles sont les **premiers éléments d'interaction** avec l'utilisateur après le processus de démarrage.

Nous verrons comment vérifier que la **connexion root est empêchée par défaut** depuis ces consoles ou bien comment vérifier que le **processus de connexion résiste à une attaque par dictionnaire**.

Enfin, nous verrons **comment redémarrer le serveur à partir du raccourci clavier CTRL+ALT+FN** et ses risques.

## Les accès aux consoles virtuelles

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

## Le raccourci CTRL+ALT+SUPPR dangereux

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


# Le mode de démarrage du serveur

### Introduction

Nous allons voir les services qui sont lancés avec la machine, pour en faire une liste et suggérer d'éventuelles suppressions.

## La cible de démarrage configurée par défaut

