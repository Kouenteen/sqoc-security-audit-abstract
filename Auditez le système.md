# 0. Informations diverses
Les résultats des commandes citées dans les paragraphes qui suivent sont tirés directement du cours "Auditez la sécurité d'un système d'exploitation" donc ils peuvent varier en fonction de votre machine.

# 1. Les composants matériels

## Introduction

Nous allons prendre connaissance de la machine en auditant ses composants matériels, à savoir son CPU, Disque et Mémoire.

## Les composants CPU de la machine

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

## Les composants mémoire de la machine

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

## Les composants DISK de la machine

Inventaire des traces laissées par le noyau lors du démarrage :

    [root@machine ~]# dmesg | grep disk
    [ 0.495770] systemd[1]: Running in initial RAM disk.
    [ 1.379411] sd 0:0:0:0: [sda] Attached SCSI disk

Le noyau détecte un disque de type SCSI, comme souvent en virtualisation donc RAS.

Pas de recommandation car la présence d'un périphérique de type bloc est commune sur linux. En revanche, la sécurisation de son accès pourrait nécessiter des vérifications.


# 2. Le partitionnement des disques

## Introduction

Analysons plus end étail le partitionnement et les systèmes de fichiers des périphériques de type bloc. On y verra les défauts de droits et mauvaises options de montage des disques sur le systèmes.

## Le partitionnement du disque dur