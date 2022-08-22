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

Analysons plus end étail le partitionnement et les systèmes de fichiers des périphériques de type bloc. On y verra les défauts de droits et mauvaises options de montage des disques sur le systèmes.

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

## 3.2 Le processus de mot de passe