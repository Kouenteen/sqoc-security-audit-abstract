# 0. Informations diverses
Les résultats des commandes citées dans les paragraphes qui suivent sont tirés directement du cours "Auditez la sécurité d'un système d'exploitation" donc ils peuvent varier en fonction de votre machine.

# 1. Les composants matériels

## 1.1 Introduction

Nous allons prendre connaissance de la machine en auditant ses composants matériels, à savoir son CPU, Disque et Mémoire.

## 1.2 Les composants CPU de la machine

Sur une nouvelle machine qu'on ne connaît pas, on fait généralement un petit inventaire :

Pour le CPU, Linux nous fournit quelques commandes pour relever l'architecture et le composant matériel : `dmesg` ou `more /proc/cpuinfo`.

    [root@machine ~]# dmesg | grep CPU
    [ 0.000000] CPU MTRRs all blank - virtualized system.
    [ 0.000000] ACPI: SSDT 000000003fff02a0 001CC (v01 VBOX VBOXCPUT 00000002 INTL 20100528)
    [ 0.000000] smpboot: Allowing 1 CPUs, 0 hotplug CPUs
    [ 0.000000] setup_percpu: NR_CPUS:5120 nr_cpumask_bits:1 nr_cpu_ids:1 nr_node_ids:1
    [ 0.000000] PERCPU: Embedded 35 pages/cpu @ffff9326ffc00000 s104856 r8192 d30312 u2097152
    [ 0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
    [ 0.000000] RCU restricting CPUs from NR_CPUS=5120 to nr_cpu_ids=1.
    [ 0.029375] CPU: Physical Processor ID: 0
    [ 0.030343] mce: CPU supports 0 MCE banks
    [ 0.086315] smpboot: CPU0: Intel(R) Core(TM) i9-8950HK CPU @ 2.90GHz (fam: 06, model: 9e, stepping: 0a)
    [ 0.182324] Performance Events: unsupported p6 CPU model 158 no PMU driver, software events only.
    [ 0.184102] Brought up 1 CPUs
    [ 0.485378] microcode: CPU0 sig=0x906ea, pf=0x2, revision=0x0

Ici, on peut voir que la commande nous indique que le CPU est virtualisé et repose sur un Intel i9 cadencé à 2.9 Ghz.

Le noyau maintient également quelques variables relatives au CPU sur son système de fichier virtuel **/proc**.

    [root@machine ~]# more /proc/cpuinfo
    processor : 0
    vendor_id : GenuineIntel
    cpu family : 6
    model : 158
    model name : Intel(R) Core(TM) i9-8950HK CPU @ 2.90GHz
    stepping : 10
    cpu MHz : 2904.000
    cache size : 12288 KB
    physical id : 0
    siblings : 1
    core id : 0
    cpu cores : 1
    apicid : 0
    initial apicid : 0
    fpu : yes
    fpu_exception : yes
    cpuid level : 22
    wp : yes
    flags : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc pni pclmulqdq monitor ssse
    3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch fsgsbase avx2 invpcid rdseed clflushopt
    bogomips : 5808.00
    clflush size : 64
    cache_alignment : 64
    address sizes : 39 bits physical, 48 bits virtual
    power management:

Parmi ces variables, on peut relever :
- processor : Le numéro du processeur
- cpu family : La famille du processeur (ici 6, héritage des Pentium Pro)
- model name : Le nom du modèle
- cpu MHz : La fréquence précise
- flags : Une liste d'attributs associés au CPU

Ce qui est important est de remarquer la présence des **flags** **PAE** et **NX**.

Ils indiquent que le processeur protège l'exécution d'instructions stockées dans les régions mémoire non-autorisées.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que le CPU dispose bien des flags PAE et NX.
>
> Par exemple, avec la commande suivante : `grep ^flags /proc/cpuinfo | head -n1 | egrep --color=auto ' (pae|nx) '`

    [root@machine ~]# grep ^flags /proc/cpuinfo | head -n1 | egrep --color=auto ' (pae|nx) '
    flags : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc pni pclmulqdq monitor ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch fsgsbase avx2 invpcid rdseed clflushopt

Cette commande filtre le contenu du fichier **/proc/cpuinfo** sur les lignes contenant le mot-clé **flags**.

## 1.3 Les composants mémoire de la machine

Le noyau laisse aussi une trace des composants mémoire qu'il sollicite au démarrage :

    [root@machine ~]# dmesg | grep Memory
    [ 0.000000] Memory: 991412k/1048512k available (7324k kernel code, 392k absent, 56708k reserved, 6305k data, 1832k init)

Elle permet de connaître la quantité en Ko de mémoire disponible et mémoire totale présente sur le système.

Le démarrage mobilise 7Ko de mémoire comme on peut le remarquer.

Le noyau affiche ensuite les statistiques d'occupation des ressources mémoire en temps réell, dans le fichier **/proc/meminfo** :

    [root@machine ~]# cat /proc/meminfo
    MemTotal: 1015508 kB
    MemFree: 672608 kB
    MemAvailable: 678432 kB
    Buffers: 2108 kB
    Cached: 118560 kB
    SwapCached: 0 kB
    Active: 152884 kB
    ...
    DirectMap4k: 24512 kB
    DirectMap2M: 1024000 kB

D'autres commandes comme **free**, **vmstat**, **top** permettent de trier les informations du fichier **/proc/meminfo**.

    [root@machine ~]# vmstat -s | grep memory
    1015508 K total memory
    170684 K used memory
    154328 K active memory
    94916 K inactive memory
    670736 K free memory
    2108 K buffer memory

La mémoire SWAP permet d'ajouter au noyau, un périphérique de type bloc (qui est stocké sur le disque dur) pour écrire et lire les pages mémoire au cas où la RAM serait saturée.

Avant, c'était indispensable car le prix de la RAM était élevée et les ressources physiques étaient limitées.

Quoi qu'il en soit, même aujourd'hui, il est toujours important de conserver un minimum de SWAP pour compenser un bug qui viendrait manger toute la RAM du système et ainsi éviter un crash.

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez la présence d'un minimum de mémoire SWAP.
>
> Par exemple, avec la commande : `vmstat -s | grep swap`.

    [root@machine ~]# vmstat -s | grep swap
    171980 K swap cache
    839676 K total swap
    0 K used swap
    839676 K free swap
    0 pages swapped in
    0 pages swapped out

## 1.4 Les composants DISK de la machine

Inventaire des traces laissées par le noyau lors du démarrage :

    [root@machine ~]# dmesg | grep disk
    [ 0.495770] systemd[1]: Running in initial RAM disk.
    [ 1.379411] sd 0:0:0:0: [sda] Attached SCSI disk

Le noyau détecte un disque de type SCSI, comme souvent en virtualisation donc RAS.

Pas de recommandation car la présence d'un périphérique de type bloc est commune sur linux. En revanche, la sécurisation de son accès pourrait nécessiter des vérifications.


# 2. Le partitionnement des disques

## 2.1 Introduction

Analysons plus en détail le partitionnement et les systèmes de fichiers des périphériques de type bloc. On y verra les défauts de droits et mauvaises options de montage des disques sur le systèmes.

## 2.2 Le partitionnement du disque dur

Le partitionnement des périphériques de type bloc s'effectue généralement lors de l'installation du système. 

Toutes les distributions proposent un mode de partitionnement dit **automatique** qui sert à facilitier le processus d'installation.

Le niveau de sécurité de ce mode est bien évidemment faible.

Reprenons le disque SCSI détecté juste avant, et analysons son partitionnement avec la commande `fdisk -l` :

    [root@machine ~]# fdisk -l

    Disque /dev/sda : 8589 Mo, 8589934592 octets, 16777216 secteurs
    Unités = secteur de 1 × 512 = 512 octets
    Taille de secteur (logique / physique) : 512 octets / 512 octets
    taille d'E/S (minimale / optimale) : 512 octets / 512 octets
    Type d'étiquette de disque : dos
    Identifiant de disque : 0x000cfef5

    Périphérique Amorçage Début Fin Blocs Id. Système
    /dev/sda1 * 2048 2099199 1048576 83 Linux
    /dev/sda2 2099200 16777215 7339008 8e Linux LVM

    Disque /dev/mapper/centos_machine-root : 6652 Mo, 6652166144 octets, 12992512 secteurs
    Unités = secteur de 1 × 512 = 512 octets
    Taille de secteur (logique / physique) : 512 octets / 512 octets
    taille d'E/S (minimale / optimale) : 512 octets / 512 octets


    Disque /dev/mapper/centos_machine-swap : 859 Mo, 859832320 octets, 1679360 secteurs
    Unités = secteur de 1 × 512 = 512 octets
    Taille de secteur (logique / physique) : 512 octets / 512 octets
    taille d'E/S (minimale / optimale) : 512 octets / 512 octets

On peut constater que le disque a deux partitions (**sda1** et **sda2**).

La première dispose du **flag** d'amorçage et d'un système de fichier de type Linux (83) ; la seconde d'un fichier de type Linux LVM (8e).

Il existe donc du LVM sur ce disque **sda**, vérifions le mapping des potentiels volumes avec la commande `lsblk` :

    [root@machine ~]# lsblk
    NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    sda 8:0 0 8G 0 disk
    ├─sda1 8:1 0 1G 0 part /boot
    └─sda2 8:2 0 7G 0 part
    ├─centos_machine-root 253:0 0 6,2G 0 lvm /
    └─centos_machine-swap 253:1 0 820M 0 lvm [SWAP]

Les deux volumes logiques sont bien accrochés à la partition LVM.

Nous pouvons relever un défaut de sécurité important avec la commande `blkid chemindudisque` :

    [root@machine ~]# blkid /dev/sda2
    /dev/sda2: UUID="mKkT60-aw2n-pcr0-032A-j0ru-gfRf-XBHbni" TYPE="LVM2_member"

La commande **blkid** nous confirme que la partition **sda2** est bien membre d'un groupe LVM mais qu'elle n'est pas chiffrée sinon son type serait **crypto_LUKS**.

Sur une machine sensible, il faut chiffrer les partitions pour rendre illisible leur contenu, très utile en cas de vol.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez le chiffrement des partitions sensibles du système.

De plus, on peut constater que tout le système (sauf _à priori_ le noyau) tient dans **une seule partition**.

Or, le partitionnement doit permettre **d'isoler les composants du système de fichiers**.

De cette façon, il n'est **pas possible** pour une application quelconque de **saturer de fichiers la partition système** et provoquer une interruption de service.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que le partitionnement isole et protège les composants du système.

Il est recommandé d'isoler :
- **/boot** : qui contient le noyau,
- **/tmp** : qui dispose de droits spéciaux et concerne les fichiers temporaires,
- **/home** : qui contient les répertoires personnels des comptes utilisateurs,
- **/var** : qui contient les données dites _variables_, notamment les applications web dans notre cas,
- **/var/log** : qui contient les fichiers de trace du système.

## 2.3 Les options de montage des partitions

Cette partie va être rapide car il n'y a que deux partitions et que Linux propose des options pour les points de montage des partitions. Parmi les plus importantes, nous pouvons citer :

| Option      | Description |
| ----------- | ----------- |
| async/sync      | Les entrées/sorties sont asynchrones/synchrones |
| noauto/auto      | Peut être montée automatiquement ou non |
| dev/nodev      | Supporte ou non les fichiers de type périphérique |
| exec/noexec      | Permet l'exécution ou non de fichier |
| suid/nosuid      | Compatible avec le bit S-[U/G]ID |
| ro/rw      | Lecture seule ou lecture/écriture |
| user/nouser      | Peut être montée par un utilisateur privilégié ou non |

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez les options des points de montage des systèmes de fichiers.
>
> Par exemple, via la commande suivante `mount | grep /dev/mapper` :

    [root@machine ~]# mount | grep /dev/mapper
    /dev/mapper/centos_fichesproduits-root on / type xfs (rw,relatime,attr2,inode64,noquota)

Ces options sont regroupées dans un libellé nommé **defaults**, qui est indiqué dans le fichier **/etc/fstab**.

Celui-ci permet de monter automatiquement des systèmes de fichiers au démarrage.

    [root@machine ~]# more /etc/fstab

    #
    # /etc/fstab
    # Created by anaconda on Wed Mar 27 09:29:55 2019
    #
    # Accessible filesystems, by reference, are maintained under '/dev/disk'
    # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
    #
    /dev/mapper/centos_machine-root / xfs defaults 0 0
    UUID=02dc2283-eb61-43d6-9aa2-92be25654a26 /boot xfs defaults 0 0
    /dev/mapper/centos_machine-swap swap swap defaults 0 0

Les options du libellé **defaults** sont :
- **rw**
- **suid**
- **dev**
- **exec**
- **auto**
- **nouser**
- **async**

En reprenant le partitionnement _idéal_ évoqué ci-dessus, les options de montage seraient :

| Montage      | Option |
| ----------- | ----------- |
| /tmp      | rw,nosuid,nodev,noexec |
| /home      | rw,nosuid,nodev,noexec |
| /var      | rw,nosuid,nodev,noexec,sync |
| /var/log      | rw,nosuid,nodev,noexec,sync |

Il n'y a pas d'intérêt de donner aux points de montage (ou du moins à la plupart d'entre eux), les droits de fichiers spéciaux, montages de périphériques ou d'exécution.

En effet, il existe des partitions spécifiques et normalisées pour ça (**/usr/lib**, **/mnt** par exemple).

La partition **/boot** n'est pas ci-dessus car elle est particulière...

## 2.4 La partition de boot

La partition **/boot** est très sensible car elle contient le noyau de Linux et la configuration dynamique de Grub.

    [root@machine ~]# ls -lrtha /boot/
    total 89M
    -rw-------. 1 root root 3,3M 20 avril 2018 System.map-3.10.0-862.el7.x86_64
    -rw-r--r--. 1 root root 145K 20 avril 2018 config-3.10.0-862.el7.x86_64
    -rw-r--r--. 1 root root 166 20 avril 2018 .vmlinuz-3.10.0-862.el7.x86_64.hmac
    -rwxr-xr-x. 1 root root 6,0M 20 avril 2018 vmlinuz-3.10.0-862.el7.x86_64
    -rw-r--r--. 1 root root 298K 20 avril 2018 symvers-3.10.0-862.el7.x86_64.gz
    drwxr-xr-x. 3 root root 17 27 mars 09:30 efi
    drwxr-xr-x. 2 root root 27 27 mars 09:30 grub
    -rw-------. 1 root root 53M 27 mars 09:33 initramfs-0-rescue-743eb5ea9918447198668f35891e3d86.img
    -rwxr-xr-x. 1 root root 6,0M 27 mars 09:33 vmlinuz-0-rescue-743eb5ea9918447198668f35891e3d86
    dr-xr-xr-x. 5 root root 4,0K 27 mars 09:34 .
    -rw-------. 1 root root 21M 27 mars 09:34 initramfs-3.10.0-862.el7.x86_64.img
    drwx------. 5 root root 97 27 mars 09:34 grub2
    dr-xr-xr-x. 17 root root 233 27 mars 09:48 ..

Le fichier **/boot/System.map** contient la table des symboles utilisés par le noyau.

C'est à dire, une liste des variables et fonctions associées à leurs adresses mémoire respectives. En quelque sorte, les **pages jaunes du noyau Linux**.

Ce fichier est très souvent la cible d'attaques visant à exploiter les failles dans le code du noyau.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez la protection de la partition **/boot**.

Pour toutes ces raisons, elle doit bénéficier d'une attention particulière. Dans le meilleur des cas, il faut attribuer l'option **noauto** afin qu'elle ne soit pas montée automatiquement.

Cependant, dans le cas où nous devrions la monter pour par exemple changer de noyau, il faudrait alors la monter avec les options **ro**, **nosuid**, **nodev**, **noexec** et la passer à *rw* pour l'utilisateur **root** uniquement lorsque nécessaire.

    [root@machine ~]# ls -lrth / | grep boot
    dr-xr-xr-x. 5 root root 4,0K 27 mars 09:34 boot

Ici, la partition **/boot** ne devrait être accessible qu'à l'utilisateur **/root**.

# 3. Les mots de passes, comptes utilisateurs, droits spéciaux et mises à jour de sécurité

## 3.1 Introduction

Sur Linux, le mot de passe est la donnée la plus sensible. Il permet d'authentifier ou de vérifier l'identité d'un compte utilisateur, et donc de bénéficier de ses droits sur le système.

Les mots de passe sont gérés par la commande **passwd** et doivent s'appuyer sur le module **PAM** de Linux.

## 3.2 Le processus de mot de passe

    [root@machine ~]# ldd /bin/passwd | grep pam
    libpam.so.0 => /lib64/libpam.so.0 (0x00007f84fd844000)
    libpam_misc.so.0 => /lib64/libpam_misc.so.0 (0x00007f84fd640000)

Le module **PAM** est responsable du stockage chiffré des mots de passes sur le système de fichiers. Historiquement, le fichier **/etc/passwd** contenait la liste des utilisateurs ET leurs mots de passe.

Or, ce fichier est accessible à tout le monde en lecture seule, ce qui permet à un attaquant d'en faire une copie et de déchiffrer les mots de passes.

Désormais, Linux utilise le principe de mots de passes dits **shadow**. Les mots de passes sont en fait stockés dans un fichier **/etc/shadow** qui est lisible uniquement par **root**.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que les mots de passe du module PAM sont en mode shadow.
>
> Par exemple, avec la commande `grep shadow /etc/pam.d/*` :

    [root@machine ~]# grep shadow /etc/pam.d/*
    /etc/pam.d/password-auth:password sufficient pam_unix.so sha512 shadow nullok try_first_pass use_authtok
    /etc/pam.d/password-auth-ac:password sufficient pam_unix.so sha512 shadow nullok try_first_pass use_authtok
    /etc/pam.d/system-auth:password sufficient pam_unix.so sha512 shadow nullok try_first_pass use_authtok
    /etc/pam.d/system-auth-ac:password sufficient pam_unix.so sha512 shadow nullok try_first_pass use_authtok

Le module **PAM** garantit également un minimum de **_robustesse_** des mots de passe. La librairie qui vérifie la robustesse est **pam_pwquality.so**.

    [root@machine ~]# grep pam_pwquality /etc/pam.d/*
    /etc/pam.d/password-auth:password requisite pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
    /etc/pam.d/password-auth-ac:password requisite pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
    /etc/pam.d/system-auth:password requisite pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
    /etc/pam.d/system-auth-ac:password requisite pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=

Pour assurer la robustesse des mots de passe, il est possible de changer les valeurs par défaut du module **pam_pwquality** :

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez la robustesse des mots de passe avec le module **pam_pwquality**.
>
> Par exemple, en consultant son fichier `/etc/security/pwquality.conf`.

| Configuration      | Description | Valeur par défaut |
| ----------- | ----------- | ----------- |
| difok      | Nombre de caractères dans le nouveau mot de passe, qui ne doivent pas être présents dans l'ancien | 5 |
| minlen      | Longueur minimale | 9 (>6 obligatoire) |
| ucredit      | Nombre de majuscules | 1 |
| lcredit      | Nombre de minuscules | 1 |
| dcredit      | Nombre de chiffres | 1 |
| ocredit      | Nombre de caractère non alphanumériques | 1 |
| maxrepeat      | Nombre maximal de caractères se répétant | 0 (désactivé) |


## 3.3 Les comptes utilisateurs et leur mot de passe

Le fichier contenant les utilisateurs est donc **/etc/passwd**.

    [root@machine ~]# cat /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    ...
    marketing:x:1000:1000:marketing:/home/marketing:/bin/bash
    patrick:x:1001:1001::/home/patrick:/bin/bash
    stephanie:x:1002:1002::/home/stephanie:/bin/bash
    christophe:x:1003:1003::/home/christophe:/bin/bash
    apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
    mysql:x:27:27:MariaDB Server:/var/lib/mysql:/sbin/nologin

Le dernier champ (après le séparateur "**:**") indique le **_shell_** à exécuter lorsque l'utilisateur se connecte sur une console ou un terminal distant.

Si la valeur du champ indique **/sbin/nologin**, le _shell_ exécuté renverra un message d'erreur et l'utilisateur ne pourra évidemment pas se connecter.

Il est possible de filter les comptes qui peuvent se connecter :

    [root@machine ~]# cat /etc/passwd | grep -v nologin
    root:x:0:0:root:/root:/bin/bash
    sync:x:5:0:sync:/sbin:/bin/sync
    shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
    halt:x:7:0:halt:/sbin:/sbin/halt
    marketing:x:1000:1000:marketing:/home/marketing:/bin/bash
    patrick:x:1001:1001::/home/patrick:/bin/bash
    stephanie:x:1002:1002::/home/stephanie:/bin/bash
    christophe:x:1003:1003::/home/christophe:/bin/bash

On retrouve ici les comptes systèmes **sync**, **shutdown**, **halt** que l'on peut ignorer, et quatre comptes utilisateurs **marketing**, **patrick**, **stephanie** et **christophe** puis le compte super-utilisateur **root**.

Considérons ici que les comptes utilisateurs sont tous configurés de la même façon. Pour obtenir des informations sur le mot de passe du compte **stephanie**, il faut utiliser la commande **`chage`** :

    [root@machine ~]# chage -l stephanie
    Dernier changement de mot de passe : mars 27, 2019
    Fin de validité du mot de passe : jamais
    Mot de passe désactivé : jamais
    Fin de validité du compte : jamais
    Nombre minimum de jours entre les changements de mot de passe : 0
    Nombre maximum de jours entre les changements de mot de passe : 99999
    Nombre de jours d'avertissement avant la fin de validité du mot de passe : 7

Exemple typique de configuration par défaut, l'utilisateur n'est pas forcé de changer son mot de passe.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que les comptes utilisateurs pouvant se connecter, ont pour obligation de changer leur mot de passe régulièrement.
>
> Par exemple, avec la commande `chage -m 7 -M 90 -W 10 utilisateur`:

    [root@machine ~]# chage -m 7 -M 90 -W 10 stephanie
    [root@machine ~]# chage -l stephanie
    Dernier changement de mot de passe : mars 27, 2019
    Fin de validité du mot de passe : juin 25, 2019
    Mot de passe désactivé : jamais
    Fin de validité du compte : jamais
    Nombre minimum de jours entre les changements de mot de passe : 7
    Nombre maximum de jours entre les changements de mot de passe : 90
    Nombre de jours d'avertissement avant la fin de validité du mot de passe : 10

Il est possible de modifier les valeurs par défaut pour la gestion des mots de passe dans le fichier **/etc/login.defs**, pratique si l'admin système doit créer des utilisateurs supplémentaires.

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez les valeurs par défaut des attributs des mots de passe pour chaque compte utilisateur dans **/etc/login.defs** :

    [root@machine ~]# cat /etc/login.defs | grep PASS
    # PASS_MAX_DAYS Maximum number of days a password may be used.
    # PASS_MIN_DAYS Minimum number of days allowed between password changes.
    # PASS_MIN_LEN Minimum acceptable password length.
    # PASS_WARN_AGE Number of days warning given before a password expires.
    PASS_MAX_DAYS 99999
    PASS_MIN_DAYS 0
    PASS_MIN_LEN 5
    PASS_WARN_AGE 7

Ici, la période de validité du mot de passe est infinie par défaut. Il serait nécessaire au moins de modifier la directive **PASS_MAX_DAYS** à 90.

## 3.4 Droits spéciaux - Lister les fichiers avec les attributs setuid, setgid et stickybit

### 3.4.1 Introduction

Un fichier **_setuidé_** permet à un utilisateur simple d'utiliser une commande/écrire dans un fichier qui nécessiterait des privilèges élevés (par exemple des droits **root**).

Un fichier **_setgidé_** a le même principe que le **setuid** mais sera effectif au niveau des droits du groupe.

Un répertoire **_setgidé_** est la même chose que le **setuid** mais pour un groupe d'utilisateurs. Ainsi, si une équipe de personnes travaillent sur le même projet donc le même répertoire, lui accorder un droit **setgid** peut être une bonne idée.

De cette façon, les fichiers créés appartiendront tous au même groupe et non aux groupes de chaque utilisateur individuel.

Le **_sticky bit_** peut être placé sur un exécutable par exemple, et cet exécutable restera en mémoire même lorsqu'il aura terminé son exécution. Ainsi, il se lancera plus rapidement au prochain lancement.

Cette pratique tend à être obsolète et seul l'administrateur système peut positionner le **sticky bit**.

Pour les répertoires, il sert à _sécuriser_ un fichier qui se trouve dans un répertoire où tout le monde a les droits en écriture. Par exemple le répertoire **/tmp**, il serait gênant que quelqu'un d'autre que nous, puisse supprimer notre fichier.

Pour y remédier, on met le sticky bit sur ce répertoire, autorisant la suppression d'un fichier uniquement à son propriétaire (nous).

### 3.4.2 Cas concrets

Prenons un exemple avec la commande **`passwd`** :

    [root@machine ~]# ls -lrtha /bin/passwd
    -rwsr-xr-x. 1 root root 28K 10 juin 2014 /bin/passwd

La commande utilise **setuid** pour permettre à un utilisateur simple de la lancer avec le compte propriétaire de la commande (généralement **root**).

C'est indispensable pour obtenir les privilèges nécessaires pour modifier le fichier **/etc/shadow**, car seul le compte **root** peut le faire.

    [root@machine ~]# ls -lrth /etc/shadow
    ---------- 1 root root 1,2K 5 mai 19:13 /etc/shadow

Les fichiers disposant du droit spécial **setuid** sont donc très sensibles et doivent être vérifiés par l'administrateur car potentiellement modifiable par un utilisateur simple.

En effet, les commandes associées sont spécialement conçues pour utiliser ce droit et, par conséquent, toute commande non vérifiée peut provoquer des attributions de privilèges interdites.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Examinez la liste des fichiers avec les droits spéciaux setuid, setgid et sticky bit.

Listons les **_fichiers_** **`setuid`** :

    [root@machine ~]# find / -type f -perm /4000 -ls 2>/dev/null
    12849314 24 -rws--x--x 1 root root 24048 avril 11 2018 /usr/bin/chfn
    12849317 24 -rws--x--x 1 root root 23960 avril 11 2018 /usr/bin/chsh
    ...
    13008451 140 ---s--x--x 1 root root 143184 avril 11 2018 /usr/bin/sudo
    ...
    12897991 16 -rwsr-xr-x 1 root root 15432 avril 11 2018 /usr/lib/polkit-1/polkit-agent-helper-1
    12879166 60 -rwsr-x--- 1 root dbus 58016 avril 11 2018 /usr/libexec/dbus-1/dbus-daemon-launch-helper

Listons les **_fichiers_** **`setgid`** :

    [root@machine ~]# find / -type f -perm /6000 -ls 2>/dev/null
    12638690 16 -r-xr-sr-x 1 root tty 15344 juin 10 2014 /usr/bin/wall
    12849314 24 -rws--x--x 1 root root 24048 avril 11 2018 /usr/bin/chfn
    12849317 24 -rws--x--x 1 root root 23960 avril 11 2018 /usr/bin/chsh
    ...
    271691 460 ---x--s--x 1 root ssh_keys 469880 avril 11 2018 /usr/libexec/openssh/ssh-keysign

Listons les **_répertoires_** **`stickybit`** :

    [root@machine ~]# find / -type d -perm -1000 -exec ls -ld {} \;
    drwxrwxrwt 2 root root 40 5 mai 12:23 /dev/mqueue
    drwxrwxrwt 2 root root 40 5 mai 12:23 /dev/shm
    ...
    drwxrwxrwt 2 root root 6 5 mai 12:23 /tmp/systemd-private-857d97fdb38348769fb88204ce6007f3-mariadb.service-GrLeTT/tmp


## 3.5 Le processus sudo

Le seul compte utilisateur qui semble disposer des privilèges administrateur est le compte **root**.

Cependant cette configuration par défaut n'est pas acceptable, on devrait pouvoir identifier chaque administrateur de la machine et ils doivent avoir les permissions associés à leur rôle sur le machine.

L'utilisateur **root** doit vraiment être utilisé qu'en dernier recours.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez la présence d'un groupe d'utilisateurs identifié comme administrateur de la machine et disposant des droits de changement de privilèges, par l'intermédiaire d'un processus type sudo.
>
> Par exemple, avec la commande `groupadd admin`.

Parmi les fichiers disposant du droit **setuid**, nous avons pu observer la présence de **/usr/bin/sudo**.

Cet outil permet à un utilisateur d'obtenir des droits prédéfinis dans le fichier de configuration **/etc/sudoers**.

    [root@machine ~]# ls -lrtha /usr/bin/sudo
    ---s--x--x. 1 root root 140K 11 avril 2018 /usr/bin/sudo

Elle dipose bien du bit **setuid** mais est configurée par défaut avec les droits d'exécution pour tout utilisateur. Cette configuration n'est pas sécurisée et il est nécessaire de restreindre le droit d'exécution de cette commande uniquement au groupe administrateur créé précédemment.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez que la commande **sudo** peut être exécutée uniquement par le groupe **admin** et l'utilisateur **root**.
>
> Par exemple, avec la commande `chmod 4750 /usr/bin/sudo` et `chown root:admin /usr/bin/sudo` :

    [root@machine ~]# chmod 4750 /usr/bin/sudo
    [root@machine ~]# chown root:admin /usr/bin/sudo
    [root@machine ~]# ls -lrtha /usr/bin/sudo
    -rwsr-x---. 1 root admin 140K 11 avril 2018 /usr/bin/sudo

Ensuite, il est nécessaire de vérifier que le groupe admin dispose bien au minimum des droits pour exécuter les commandes nécessaires au maintien du système dans le fichier **/etc/sudoers**.

Au mieux, toutes les commandes possibles.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez que le fichier **/etc/sudoers** contient la déclaration des commandes nécessaires pour les utilisateurs du groupe **admin**.
>
> Par exemple :

    %admin ALL=(ALL) ALL

## 3.6 Les mises à jour de sécurité

_La partie que nous verrons **concernera CentOS** (yum/dnf) mais le principe est presque le même pour Debian/Ubuntu (apt)_

Les distributions fournissent le système sous la forme de packages de sources précompilées.

Cela nous évite de devoir compiler manuellement les sources des composants du système. Cependant, il faudrait toujours vérifier que les éditeurs de distributions n'ont pas mis à jour un package suite à la publication d'une CVE (_**Common Vulnerabilities Exposure**_).

Pour faire cette veille de façon correcte, il faut regarder du côté des dépôts de paquets, et s'assurer qu'ils soient qualifiés et sûrs.

Il faut vérifier les fichiers de configuration du **gestionnaire de paquets** ou _**package manager**_. Sur CentOS, c'est **yum** ou bien **dnf**,  sur Debian/Ubuntu, c'est **apt**.

Les vérifications sont un peu différentes entre les deux package manager (yum et apt) mais l'opération consiste à vérifier que les adresses des dépôts pointent vers des serveurs maintenus directement par l'éditeur et que les packages téléchargés comportent une signature numérique.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que les packages installés sur le système sont signés et sûrs.
>
> Par exemple, en consultant le fichier **/etc/yum.repos.d/CentOS-Base.repo** :

    [root@machine ~]# more /etc/yum.repos.d/CentOS-Base.repo
    # CentOS-Base.repo
    ...
    [base]
    name=CentOS-$releasever - Base
    mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
    #baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

Ici, les attributs **gpgcheck** et **gpgkey** garantissent que toutes les signatures numériques des packages téléchargés seront vérifiées.

Il faudrait également constamment vérifier qu'il n'y a pas besoin de mettre à jour des paquets qui auraient été corrigés par l'éditeur.

Pour cela, sous CentOS, il faut installer un plugin qui permet de mettre à jour uniquement les paquets corrigés pour des raisons de sécurité : **yum-plugin-security**.

On vérifie d'abord si le plugin est installé :

    [root@machine ~]# yum list installed | grep security
    [root@machine ~]#

Si la commande ne renvoie pas de résultat, c'est qu'il n'est pas installé.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que le plugin gérant les mises à jour de sécurité des paquets est bien installé.
>
> Par exemple, avec la commande `yum install yum-plugin-security` :

Depuis la version 7 de CentOS, le plugin est directement intégré au gestionnaire de paquets **yum**.

La commande **yum --security check-update** vérifie s'il y a besoin de mettre à jour des paquets :

    [root@machine ~]# yum --security check-update
    Modules complémentaires chargés : fastestmirror
    base/7/x86_64 | 3.6 kB 00:00:00
    epel/x86_64/metalink | 24 kB 00:00:00
    epel/x86_64
    ...
    --> 1:openssl-1.0.2k-16.el7_6.1.x86_64 from updates excluded (updateinfo)
    --> 1:openssl-devel-1.0.2k-16.el7_6.1.i686 from updates excluded (updateinfo)
    No packages needed for security; 165 packages available

Le retour de la commande indique qu'il n'y a pas de mises à jour de paquets **for security** mais en revanche, 165 paquets peuvent être mis à jour.

Cette vérification doit être effectuée au moins 1 fois par jour par exemple, surtout sur les serveurs accessibles publiquement.

Il serait donc utile de mettre une tâche planifiée via le paquet **yum-cron**, qui s'appuie sur le gestionnaire de tâches planifiées **cron**.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez la planification régulière d'une tâche automatique pour mettre à jour les paquets corrigés pour raisons de sécurité.
>
> Par exemple, avec la commande `yum install yum-cron`.

La commande va programmer une tâche quotidienne avec la configuration suivante :

    [root@machine ~]# grep ^[^#] /etc/yum/yum-cron.conf
    [commands]
    update_cmd = default
    update_messages = yes
    download_updates = yes
    apply_updates = no
    random_sleep = 360
    [emitters]
    system_name = None
    emit_via = stdio
    output_width = 80
    [email]
    email_from = root@localhost
    email_to = root
    email_host = localhost
    [groups]
    group_list = None
    group_package_types = mandatory, default
    [base]
    debuglevel = -2
    mdpolicy = group:main

La directive **update_cmd** peut être positionné aux valeurs suivantes :

| Valeur      | Commande correspondante |
| ----------- | ----------- |
| update_cmd = default      | yum upgrade |
| update_cmd = security      | yum --security upgrade |
| update_cmd = security-severity:Critical      | yum --sec-severity=Critical upgrade |
| update_cmd = minimal      | yum --bugfix update-minimal |
| update_cmd = minimal-security      | yum --security update-minimal |
| update_cmd = minimal-security-severity:Critical      | yum --sec-severity=Critical update-minimal |

# 4. Résumé des recommandations de toute cette partie

| Recommandation      | Type | Principe |
| ----------- | ----------- | ----------- |
| Vérifier que le CPU dispose bien des flags **PAE** et **NX**      | ❌CRITICAL | Défense en profondeur |
| Vérifier la présence d'un **minimum de mémoire SWAP** sur le système      | 🚸WARNING |Défense en profondeur |
| Vérifier le **chiffrement des partitions** sensibles du système      | ❌CRITICAL | Défense en profondeur |
| Vérifier que le **partitionnement isole et protège** les composants du système      | ❌CRITICAL | Défense en profondeur |
| Vérifier les **options des points de montage** des systèmes de fichiers      | ❌CRITICAL | Moindre privilège |
| Vérifier la **protection de la partition /boot**      | ❌CRITICAL | Moindre privilège |
| Vérifier que les **mots de passe du module PAM sont en mode shadow**      | ❌CRITICAL | Défense en profondeur |
| Vérifier la robustesse des mots de passe avec le **module pam_pwquality**      | 🚸WARNING | Défense en profondeur |
| Vérifier que les comptes utilisateurs qui peuvent se connecter ont pour obligation de **changer leur mot de passe régulièrement**      | ❌CRITICAL | Défense en profondeur |
| Vérifier les valeurs par défaut des attributs des mots de passe pour chaque compte utilisateur dans **/etc/login.defs**      | 🚸WARNING | Défense en profondeur |
| Examiner la liste des fichiers avec les droits spéciaux **setuid**, **setgid** et **sticky bit**      | ❌CRITICAL | Moindre privilège |
| Vérifier la présence d'un **groupe d'utilisateurs** identifié comme **administrateur** de la machine et disposant des droits de changement de privilèges par l'intermédiaire d'un processus type sudo      | ❌CRITICAL | Moindre privilège |
| Vérifier que la commande **sudo** peut être exécutée **uniquement par le groupe administrateur** (+ root)      | ❌CRITICAL | Moindre privilège |
| Vérifier que le fichier de configuration **/etc/sudoers** contient la déclaration des commandes nécessaires au maintien du système pour **les utilisateurs du groupe administrateur**      | ❌CRITICAL | Moindre privilège |
| Vérifier que les packages installés sur le système sont **signés et sûrs**      | ❌CRITICAL | Défense en profondeur |
| Vérifier que le **plugin gérant les mises à jour de sécurité** des packages de la distribution est bien installé      | ❌CRITICAL | Défense en profondeur |
| Vérifier la planification régulière d'une tâche automatique pour **mettre à jour les packages corrigés pour raisons de sécurité**      | ❌CRITICAL | Défense en profondeur |