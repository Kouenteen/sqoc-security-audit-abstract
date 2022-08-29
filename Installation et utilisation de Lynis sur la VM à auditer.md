# 0. Prérequis

Faire un snapshot de la VM avant de faire quoi que ce soit pour qu'elle garde son statut "de base".

# 1. Installer le paquet Lynis

Par exemple, avec la commande `apt install lynis` :

    root@vm-audit:/home/bigboss# apt install lynis -y

On aura donc la version du paquet présent sur le dépôt d'ubuntu 16.04 (Xenial).

Pour avoir la toute dernière version de Lynis (à ce jour, 3.0.8-100) :

    root@vm-audit:/home/bigboss# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 013baa07180c50a7101097ef9de922f1c2fde6c4
    root@vm-audit:/home/bigboss# sudo apt install apt-transport-https
    root@vm-audit:/home/bigboss# echo 'Acquire::Languages "none";' | sudo tee /etc/apt/apt.conf.d/99disable-translations
    root@vm-audit:/home/bigboss# echo "deb https://packages.cisofy.com/community/lynis/deb/ stable main" | sudo tee /etc/apt/sources.list.d/cisofy-lynis.list
    root@vm-audit:/home/bigboss# apt update
    root@vm-audit:/home/bigboss# apt install lynis
    root@vm-audit:/home/bigboss# lynis show version
    root@vm-audit:/home/bigboss# lynis show version
    3.0.8

Voilà, on a la toute dernière version de l'outil Lynis.

# 2. Lancer un audit

Son utilisation est assez simple. Il suffit de lancer le script principal **/usr/bin/lynis** avec l'une des deux commandes suivantes :
- **audit** : exécuter la batterie de tests et réaliser l'audit,
- **show** : afficher une information, telle que la configuration par exemple.

Ensuite, nous pouvons éventuellement ajouter les options suivantes dans le script :

| Option      | Abréviation | Description |
| ----------- | ----------- | ----------- |
| --auditor "Name"   |   | Définit le nom de l'auditeur dans le rapport  |
| --checkall   | -c   | Effectue tous les tests  |
| --check-update   |   | Vérifie les mises à jour  |
| --cronjob   |   | Exécute en tant que tâche planifiée (inclut les options -c -Q)  |
| --help   | -h  | Affiche l'aide  |
| --manpage   |   | Affiche la documentation  |
| --nocolors   |   | Annule l'utilisation de couleurs  |
| --pentest   |   | Effectue un test d'intrusion  |
| --quick   | -Q  | Mode non interactif, sauf pour les erreurs  |
| --quiet   |   | N'affiche que les avertissements  |
| --reverse-colors   |   | Utilise des couleurs inversées  |
| --version   | -V  | Vérifie la version du programme  |

# 3. Exécuter le premier rapport de Lynis

    root@vm-audit:/home/bigboss# lynis audit system

Une fois la commande lancée, pour la version 3.0.8 de Lynis, il ne nous demandera pas à chaque section de confirmer en faisant **ENTRÉE** pour passer à la suite, il fera l'audit du système jusqu'à la fin.

Arrivé à la fin, nous avons un petit récapitulatif contenant un **score de durcissement** (l'index hardening).

Cet index, basé sur **100** est spécifique à Lynis. Chaque résultat de tests effectué pendant l'audit, ajoute ou non des points à cet indicateur. L'objectif est de suivre les recommandations de Lynis afin d'augmenter cette valeur et de se rapprocher de 100.

Concernant des serveurs sensibles, il faut à peu près considérer qu'ils sont sécurisés à partir du score 85 (**mais cela dépend évidemment du contexte**).

La liste des tests effectués par Lynis et les traces détaillées des tests se trouvent dans les deux fichiers suivants :
- **/var/log/lynis.log**,
- **/var/log/lynis-report.dat**.

**Récapitulatif de fin d'audit de Lynis** :

    ================================================================================

    Lynis security scan details:

    Hardening index : 58 [###########         ]
    Tests performed : 270
    Plugins enabled : 0

    Components:
    - Firewall               [V]
    - Malware scanner        [X]

    Scan mode:
    Normal [V]  Forensics [ ]  Integration [ ]  Pentest [ ]

    Lynis modules:
    - Compliance status      [?]
    - Security audit         [V]
    - Vulnerability scan     [V]

    Files:
    - Test and debug information      : /var/log/lynis.log
    - Report data                     : /var/log/lynis-report.dat

    ================================================================================

# 4. Comparez les résultats de Lynis avec ceux de notre audit

On va prendre l'exemple des recommandations de Lynis concernant le système de fichiers :

    [+] Systèmes de fichier
    ------------------------------------
    - Checking mount points
        - Checking /home mount point                              [ SUGGESTION ]
        - Checking /tmp mount point                               [ SUGGESTION ]
        - Checking /var mount point                               [ SUGGESTION ]
    - Checking LVM volume groups                                [ TROUVÉ ]
        - Checking LVM volumes                                    [ TROUVÉ ]
    - Query swap partitions (fstab)                             [ OK ]
    - Testing swap partitions                                   [ OK ]
    - Testing /proc mount (hidepid)                             [ SUGGESTION ]
    - Checking for old files in /tmp                            [ OK ]
    - Checking /tmp sticky bit                                  [ OK ]
    - Checking /var/tmp sticky bit                              [ OK ]
    - ACL support root file system                              [ ACTIVÉ ]
    - Mount options of /                                        [ PAS PAR DÉFAUT ]
    - Mount options of /boot                                    [ PAR DÉFAUT ]
    - Mount options of /dev                                     [ PARTIELLEMENT RENFORCÉ ]
    - Mount options of /dev/shm                                 [ PARTIELLEMENT RENFORCÉ ]
    - Mount options of /run                                     [ PARTIELLEMENT RENFORCÉ ]
    - Total without nodev:10 noexec:11 nosuid:7 ro or noexec (W^X): 11 of total 30
    - Checking Locate database                                  [ TROUVÉ ]
    - Disable kernel support of some filesystems

Pour les points de montages, Lynis propose trois recommanadtions sur **/home**, **/tmp** et **/var**.

Ces recommandations se situent en fin de rapport :

    * To decrease the impact of a full /home file system, place /home on a separate partition [FILE-6310]
      https://cisofy.com/lynis/controls/FILE-6310/

    * To decrease the impact of a full /tmp file system, place /tmp on a separate partition [FILE-6310]
        https://cisofy.com/lynis/controls/FILE-6310/

    * To decrease the impact of a full /var file system, place /var on a separate partition [FILE-6310]
        https://cisofy.com/lynis/controls/FILE-6310/

Lynis conseille donc des partitions distinctes pour ces trois points de montage. Ce qui était exactement notre conclusion :

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| SYS.4   | Vérifier que le **partitionnement** isole et protège les composants du système  | ❌CRITICAL  | Défense en profondeur  |

Mais à priori, Lynis ne teste pas les options de montage de ces partitions. On est donc allés un peu plus loin avec la recommandation suivante :

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| SYS.5   | Vérifier les options des **points de montage** des systèmes de fichiers  | ❌CRITICAL  | Moindre privilège  |

Prenons maintenant un second exemple avec le chapitre **SSH Support** (Prise en charge SSH). Le résultat de Lynis montre :

    [+] Prise en charge SSH
    ------------------------------------
    - Checking running SSH daemon                               [ TROUVÉ ]
        - Searching SSH configuration                             [ TROUVÉ ]
        - OpenSSH option: AllowTcpForwarding                      [ SUGGESTION ]
        - OpenSSH option: ClientAliveCountMax                     [ SUGGESTION ]
        - OpenSSH option: ClientAliveInterval                     [ OK ]
        - OpenSSH option: Compression                             [ SUGGESTION ]
        - OpenSSH option: FingerprintHash                         [ OK ]
        - OpenSSH option: GatewayPorts                            [ OK ]
        - OpenSSH option: IgnoreRhosts                            [ OK ]
        - OpenSSH option: LoginGraceTime                          [ OK ]
        - OpenSSH option: LogLevel                                [ SUGGESTION ]
        - OpenSSH option: MaxAuthTries                            [ SUGGESTION ]
        - OpenSSH option: MaxSessions                             [ SUGGESTION ]
        - OpenSSH option: PermitRootLogin                         [ SUGGESTION ]
        - OpenSSH option: PermitUserEnvironment                   [ OK ]
        - OpenSSH option: PermitTunnel                            [ OK ]
        - OpenSSH option: Port                                    [ SUGGESTION ]
        - OpenSSH option: PrintLastLog                            [ OK ]
        - OpenSSH option: StrictModes                             [ OK ]
        - OpenSSH option: TCPKeepAlive                            [ SUGGESTION ]
        - OpenSSH option: UseDNS                                  [ OK ]
        - OpenSSH option: X11Forwarding                           [ SUGGESTION ]
        - OpenSSH option: AllowAgentForwarding                    [ SUGGESTION ]
        - OpenSSH option: Protocol                                [ OK ]
        - OpenSSH option: UsePrivilegeSeparation                  [ SUGGESTION ]
        - OpenSSH option: AllowUsers                              [ NON TROUVÉ ]
        - OpenSSH option: AllowGroups                             [ NON TROUVÉ ]
    
Il spécifie les recommandations suivantes :

    * Consider hardening SSH configuration [SSH-7408]
    - Details  : AllowTcpForwarding (set YES to NO)
      https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : ClientAliveCountMax (set 3 to 2)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : Compression (set DELAYED to NO)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : LogLevel (set INFO to VERBOSE)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : MaxAuthTries (set 6 to 3)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : MaxSessions (set 10 to 2)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : PermitRootLogin (set YES to (FORCED-COMMANDS-ONLY|NO|PROHIBIT-PASSWORD|WITHOUT-PASSWORD))
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : Port (set 22 to )
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : TCPKeepAlive (set YES to NO)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : X11Forwarding (set YES to NO)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : AllowAgentForwarding (set YES to NO)
        https://cisofy.com/lynis/controls/SSH-7408/

    * Consider hardening SSH configuration [SSH-7408]
        - Details  : UsePrivilegeSeparation (set YES to SANDBOX)
        https://cisofy.com/lynis/controls/SSH-7408/

On retrouve ici certaines de nos recommandations :

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| NET.12   | Examiner la directive **PermitRootLogin** du service SSH  | ❌CRITICAL  | Moindre privilège  |
| NET.13   | Vérifier le **port d'écoute du service SSH**  | ❌CRITICAL  | Défense en profondeur  |
| NET.14   | Vérifier le **durcissement avancé** du service SSH  | 🚸WARNING  | Défense en profondeur  |

La recommandation **NET.14** couvre de nombreuses suggestions de Lynis car le **durcissement** de SSH consiste à activer/désactiver ou même rajouter des options dans le fichier de configuration.

On peut cependant noter que Lynis propose le durcissement de certaines directives supplémentaires parmi :
- **AllowTcpForwarding**;
- **MaxAuthTries**;
- **TCPKeepAlive**;
- **AllowAgentForwarding**.

