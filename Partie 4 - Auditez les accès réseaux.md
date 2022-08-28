# 0. Informations diverses

Voyons désormais l'audit des accès réseau ouverts sur le serveur, ainsi que les programmes et services qui ouvrent les ports associés.

# 1. Les ports ouverts du serveur et les processus associés

## 1.1 Introduction

> Les accès réseau sont la cible de la plupart des attaques à distance. Chaque service accessible à distance ouvre un ou plusieurs ports de connexion sur le serveur, et donc un accès via le réseau.

En fonction de ce qui est ouvert et des flux de requête qui leurs sont destinés, les accès réseaux doivent être particulièrement surveillés et maîtrisés.

Par exemple, ils sont critiques sur un serveur Web accessible au public. Ils sont probablement moins critiques, mais restent sensibles sur une base de données protégée dans le bastion de l'architecture réseau interne d'une entreprise.

Pour lister les ports ouverts sur le serveur, Linux propose plusieurs outils.

Parmi ces outils, on retrouve la traditionnelle commande `netstat`, qui n'est plus installée par défaut mais est accessible via le paquet `net-tools`.

> Cette commande est désormais remplacée par la commande `ss` qui reprend toutes les options de netstat.

Lançons la commande `ss -plunt` :

    [root@machine ~]# ss -lptun
    Netid State Recv-Q Send-Q Local Address:Port Peer Address:Port
    udp UNCONN 0 0 127.0.0.1:323 *:* users:(("chronyd",pid=573,fd=1))
    udp UNCONN 0 0 *:68 *:* users:(("dhclient",pid=2467,fd=6))
    udp UNCONN 0 0 ::1:323 :::* users:(("chronyd",pid=573,fd=2))
    tcp LISTEN 0 0 *:3306 *:* users:(("mysqld",pid=1209,fd=13))
    tcp LISTEN 0 0 *:22 *:* users:(("sshd",pid=818,fd=3))
    tcp LISTEN 0 0 127.0.0.1:25 *:* users:(("master",pid=1253,fd=13))
    tcp LISTEN 0 0 :::80 :::* users:(("httpd",pid=2004,fd=4),("httpd",pid=2003,fd=4),("httpd",pid=2002,fd=4),("httpd",pid=2001,fd=4),("httpd",pid=2000,fd=4),("httpd",pid=817,fd=4))
    tcp LISTEN 0 0 :::21 :::* users:(("proftpd",pid=860,fd=0))
    tcp LISTEN 0 0 :::22 :::* users:(("sshd",pid=818,fd=4))
    tcp LISTEN 0 0 ::1:25 :::* users:(("master",pid=1253,fd=14))

Les options passées à cette commande correspondent aux fonctionnalités suivantes :

| Option      | Description |
| ----------- | ----------- |
| p   | Affiche le processus utilisant la socket |
| l   | Affiche toutes les sockets en écoute |
| u   | Affiche les sockets de protocole UDP |
| n   | Affiche les ports ouverts en mode numérique |
| t   | Affiche les sockets de protocole TCP |

On peut remarquer que cette commande nous indique l'adresse IP associée au port, par exemple :
- **127.0.0.1** : l'adresse de loopback locale IPv4 (le port est accessible uniquement depuis ce serveur),
- **'*'**: toutes les adresses IPv4 du serveur,
- **::** : toutes les adresses IPv6 du serveur,
- **::1** : l'adresse de loopback locale IPv6 du serveur.

> ❌ **RECOMMANDATION-CRITICAL** (Minimisation) : Examinez la liste des _sockets_ et ports ouverts sur le réseau.

Par exemple, vous pourriez tout à fait nettoyer cette liste en fermant les ports suivants :

| Option      | Processus/Service |
| ----------- | ----------- |
| 127.0.0.1:323 et ::1:323   | Chronyd est un serveur de temps qui n'est pas indispensable et est seulement utilisé quand vous avez besoin d'une horloge commune pour vos applications  |
| *:68   | Pas besoin d'un client DHCP sur une adresse IP publique |
| 127.0.0.1:25 et ::1:25   | Serveur postfix interne est installé uniquement si vos applications envoient des mails  |

Les ports ouverts par les trois processus cités dans ce tableau dépendent de services réseau, lancés automatiquement lors du démarrage de la machine, ou lancés manuellement.

Lorsque les informations du service ne sont pas affichées sur la commande `ss`, il est possible de les retrouver avec la commande `lsof`: 

    [root@machine ~]# lsof -i :323
    COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
    chronyd 573 chrony 1u IPv4 14211 0t0 UDP localhost:323
    chronyd 573 chrony 2u IPv6 14212 0t0 UDP localhost:323

> ❌ **RECOMMANDATION-CRITICAL** (Minimisation) : Vérifiez que les services qui ouvrent des ports inutiles sont désactivés.

Dans notre cas, pour couper ces services, il faudrait lancer les commandes suivantes :

    systemctl stop chronyd postfix
    systemctl disable chronyd postfix

Le service **dhclient** est directement géré par le processus de gestion de la carte réseau. Il ne démarre pas lorsque l'adresse est fixée en statique.

Autre remarque importante : concernant le port d'écoute du service de base de données, vous pouvez voir la ligne suivante :

    tcp LISTEN 0 0 *:3306 *:* users:(("mysqld",pid=1209,fd=13))

Elle indique que le service est en écoute sur l'adresse IPv4 du serveur et sur le port 3306. Cette configuration ne convient pas car il est très peu probable qu'il soit possible d'accéder au service de base de données directement depuis l'environnement externe du serveur.

Nous pouvons même dire qu'une connexion interne en provenance des codes sources de l'application sera largement suffisante.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que les services qui ouvrent des ports sur une interface externe sont pertinents.

Pour vérifier cela et ouvrir le port en écoute uniquement sur la loopback IPv4, modifiez le fichier de configuration du service **/etc/my.cnf** et ajoutez-y la ligne suivante :

    bind-address = 127.0.0.1

Par ailleurs, et dans la mesure du possible, si les services associés ne sont pas publics, il est fortement recommandé de changer les ports d'accès par défaut à ces services, car ce sont des cibles pour les attaquants.

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez que les ports par défaut des services ont été changés dans la mesure du possible.

Ici, l'application Web n'est pas publique et nous pourrions tout à fait envisager de changer le port par défaut dans le fichier **/etc/httpd/conf/httpd.conf** en modifiant la ligne concernée :

    # Listen: Allows you to bind Apache to specific IP addresses and/or
    # ports, instead of the default. See also the <VirtualHost>
    # directive.
    #
    # Change this to Listen on specific IP addresses as shown below to
    # prevent Apache from glomming onto all bound IP addresses.
    #
    #Listen 12.34.56.78:80
    #Listen 80
    Listen 8282


# 2. Les processus de filtrage réseau

> Le filtrage réseau consiste à analyser le trafic entrant et sortant du serveur, et à établir des règles définissant les actions à effectuer sur ce trafic. Cette fonctionnalité est exécutée directement par le noyau Linux.

## 2.1 Les compatibilités avec les TCP Wrappers

Le premier processus de filtrage réseau sous Linux est le **TCP Wrapper**. Cette technique s'appuie sur la librairie partagée **libwrap** et permet de définir des règles d'accès sur deux fichiers de configuration :
- le fichier **/etc/hosts.allow** qui contient la liste des règles autorisant la connexion à un service,
- le fichier **/etc/hosts.deny** qui contient la liste des règles refusant la connexion à un service.

Dans le cas où les deux fichiers seraient renseignés et présenteraient des incohérences (par exemple, un même service autorisé dans l'un et refusé dans l'autre), ce serait le fichier **host.allow** qui prévaudrait. Les règles qu'il contient sont prioritaires.

Le grand avantage de cette technique, c'est qu'il est possible de modifier ces règles sans toucher à la configuration spécifique des services concernés.

Par contre, le service doit être compilé avec la librairie **libwrap**.

Pour vérifier si un service peut être compilé avec **libwrap**, lancez les commandes suivantes :

    [root@machine ~]# which sshd
    /usr/sbin/sshd
    [root@machine ~]# ldd /usr/sbin/sshd | grep libwrap
    libwrap.so.0 => /lib64/libwrap.so.0 (0x00007fc69c57f000)

    [root@machine ~]# find / -name mysqld
    /usr/libexec/mysqld
    [root@machine ~]# ldd /usr/libexec/mysqld | grep libwrap
    [root@machine ~]#

    [root@machine ~]# which httpd
    /usr/sbin/httpd
    [root@machine ~]# ldd /usr/sbin/httpd | grep libwrap
    [root@machine ~]#

    [root@machine ~]# which proftpd
    /usr/sbin/proftpd
    [root@machine ~]# ldd /usr/sbin/proftpd | grep libwrap
    [root@machine ~]#

Comme nous pouvons le voir, seul le service **sshd** est compatible avec la fonctionnalité **TCP Wrapper**.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que les TCP Wrappers sont configurés pour les services réseau compatibles.

Ici, pour établir les règles de connexion au service **sshd**, il faudrait renseigner les deux lignes suivantes dans les fichiers **/etc/hosts.allow** et **/etc/hosts.deny** :

    # /etc/hosts.allow
    sshd: 1.2.3.0/255.255.255.0 # Masque réseau complet
    sshd: 192.168.0.0/255.255.255.0 # Adresse IP spécifique

    # /etc/hosts.deny
    sshd: ALL # Rejet de tout le reste

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez qu'il est possible de rendre les services réseau compatibles avec les TCP Wrappers.

C'est notamment le cas du service **proftpd**, qui devient compatible une fois compilé avec le module **contrib/mod_wrap.c**.

## 2.2 Les règles du firewall Netfilter

Le second processus de filtrage réseau est le firewall **Netfilter**, codé directement dans le noyau Linux.

Ce pare-feu fonctionne avec un système composé de trois principaux éléments :
- les **chaînes**, qui déterminent la caractéristique principale du paquet réseau. La chaîne **INPUT** règle le trafic entrant, **OUTPUT** règle le trafic sortant et **FORWARD** le trafic qui transite par le serveur,
- les **règles** qui servent à reconnaître le type de paquet. Cela peut se faire, entre autres, en fonction de son **adresse source**, de son **port de destination** ou encore de son protocole,
- les **actions**, qui sont associées aux **chaînes** et aux **règles**. Elles déterminent ce qu'il se passe : par exemple, **INPUT and DPORT 80 ACCEPT** va laisser entrer tous les paquets à destination du port 80.

La description est succincte (très simplifiée) car il faudrait faire un cours entier avec beaucoup de pratique sur Netfilter pour y détailler son fonctionnement.

Avant toute chose, on vérifie que le code de Netfilter est bien intégré au noyau :

    [root@machine ~]# grep IPTABLES /boot/config-`uname -r`
    CONFIG_IP_NF_IPTABLES=m
    CONFIG_IP6_NF_IPTABLES=m

La commande ci-dessus indique que le code Netfilter est compilé comme un module, d'où le **m** et qu'il est donc nécessaire de le charger pour accéder aux fonctionnalités du pare-feu.

Cela ne convient pas au vu de la recommandation que nous avions émise sur le noyau (Bloquer le chargement de modules supplémentaires).

> 🚸 **RECOMMANDATION-WARNING** (Minimisation) : Vérifiez qu'il est possible d'accéder au firewall Netfilter par le noyau sans la contraite module.

Linux fournit avec son pare-feu, un jeu de binaires qui permet de manipuler sa configuration, en l'occurence avec la commande **iptables**.

Pour voir la configuration actuelle du pare-feu Netfilter, il faut lancer la commande :

    [root@machine ~]# iptables -L -n
    Chain INPUT (policy ACCEPT)
    target prot opt source destination

    Chain FORWARD (policy ACCEPT)
    target prot opt source destination

    Chain OUTPUT (policy ACCEPT)
    target prot opt source destination

Le résultat de cette commande affiche une configuration par défaut typique : une vraie passoire !

Le firewall ne dispose d'aucune règle et est configuré par défaut sur l'action **ACCEPT**. Évidemment, cette situation ne convient pas.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que les règles Netfilter sont bien positionnées.

Il faudrait déjà passer la configuration par défaut en **DROP**.

    :INPUT DROP [0:0]
    :FORWARD DROP [0:0]
    :OUTPUT DROP [0:0]

Ensuite, il faut laisser entrer les paquets à destination des services. Ci-dessous, exemple avec le port par défaut du service Web :

    -A INPUT -i eth0 -p tcp -m tcp --dport 80 -j ACCEPT
    -A OUTPUT -o eth0 -p tcp -m tcp --sport 80 -j ACCEPT

Attention : ne pas oublier que la configuration de Netfilter toujours à chaud et que le noyau ne garde pas cette configuration d'un _reboot_ à l'autre. Il est donc nécessaire de renseigner des fichiers de configuration comme **/etc/sysconfig/iptables** afin de charger des règles par défaut à la fin du processus de démarrage.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que les règles Netfilter par défaut sont définies dans le fichier de configuration **/etc/sysconfig/iptables**.

## 2.3 Les sysctl du noyau

> Les sysctl du noyau sont des variables accessibles par l'administrateur et permettent d'influer sur le comportement du noyau pour certains aspects de son périmètre fonctionnel, comme les pages mémoires, les systèmes de fichiers ou encore le réseau.

Il existe de nombreuses **sysctl** pour le réseau. Chacune permet de renforcer un peu plus la sécurité réseau de la machine. Recensons les plus communes et leur implication :

| SYSCTL      | Description |
| ----------- | ----------- |
| net.ipv4.ip_forward = 0   | Absence de routage sur les interfaces  |
| net.ipv4.conf.all.accept_source_route = 0 net.ipv4.conf.default.accept_source_route = 0   | Refus des paquets de source _routing_  |
| net.ipv4.icmp_echo_ignore_broadcasts = 1   | Configuration contre les tempêtes ICMP (_Smurf attacks_)  |
| net.ipv4.accept_source_route = 0   | Configuration contre les paquets contenant leur propre routage (utile seulement aux attaquants)  |
| net.ipv4.rp_filter = 1   | Configuration contre les paquets avec une IP incohérente avec l'interface réseau de destination (_Spoofing_)  |

Il faut ajouter ces configurations dans le fichier **/etc/sysctl.conf** pour qu'elles soient appliquées à chaque redémarrage du serveur.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez le positionnement des sysctl réseau communes au noyau dans le fichier **/etc/sysctl.conf**.

# 3. Le service SSH

## 3.1 Introduction

Le service SSH est sûrement le **service le plus critique sur le serveur**.

C'est le moyen le plus connu et commun pour obtenir un _shell_ distant sur le système. Cela est possible en se connectant via un terminal sur le service et en chiffrant les communications avec le protocole SSH.

Il est donc très important de veiller à ce que ce service soit configuré de manière sécurisée.

## 3.2 Les fondamentaux du service SSH

> Le fichier de configuration du service SSH est /etc/ssh/sshd_config. Ce fichier est unique et n'a pas de ramification dans d'autres fichiers.

Vérifions dans un premier temps la version du protocole SSH supporté avec la commande `ssh -v localhost` :

    [root@machine ssh]# ssh -v localhost
    OpenSSH_7.4p1, OpenSSL 1.0.2k-fips 26 Jan 2017
    debug1: Reading configuration data /etc/ssh/ssh_config
    ...
    debug1: Remote protocol version 2.0, remote software version OpenSSH_7.4
    ..
    ECDSA key fingerprint is SHA256:7H8DIiivqyhmlDIcOFc7jq3C4zna6CdPxrVH+ng5VSI.
    ECDSA key fingerprint is MD5:c5:86:1e:cd:ab:69:b9:a0:6a:bf:d2:89:6e:dd:31:9f.
    Are you sure you want to continue connecting (yes/no)?

Parmi les lignes affichées, on doit retrouver **Remote protocol version 2.0**. La version 2 amène plus de fonctionnalités et la version 1 du protocole est aujourd'hui obsolète.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez la version du protocole SSH supporté par le service exploité sur le serveur.

S'il y aune chose qu'il faut absolument bannir en ce qui concerne SSH, c'est la possibilité de se connecter directement avec le compte **root**. Ce réglage est indiqué dans le fichier de configuration avec la directive **PermitRootLogin** :

    [root@machine ~]# grep PermitRootLogin /etc/ssh/sshd_config
    #PermitRootLogin yes

Sur CentOS, par défaut, la directive possède la valeur **yes** ce qui n'est pas le cas sur Debian.

Ici, il faudrait décommenter la ligne et indiquer **no** :

    PermitRootLogin no

Il faut noter qu'il est possible de positionner cette valeur sur **forced-commands-only**, ce qui laisse la possibilité au compte **root** de lancer une commande via le service en utilisant les clés d'authentification. On revient dessus plus tard.

Enfin, dernier point fondamental : le port d'écoute. Par défaut, SSH écoute sur le port 22, il faudrait changer le port afin de compliquer le travail des attaquants.

    [root@machine ~]# grep Port /etc/ssh/sshd_config
    #Port 22

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez le port d'écoute du service SSH.

Par exemple, on pourrait indiquer ici :

    Port 6060

## 3.3 Le durcissement avancé du service SSH

Une fois les directives basiques passées, il est temps de se pencher sur l'aspect un peu plus sécurisé de la configuration du service :
- pour empêcher des sessions connectées sans activité (idle), il faut utiliser les directives suivantes dans le fichier **/etc/ssh/sshd_config**.

Directives à ajouter :

    ClientAliveInterval 300 # Timout idle max en seconde
    ClientAliveCountMax 0 # Nombre de messages de continuité autorisés sans réponse du client

- pour empêcher toute connexion d'un compte utilisateur sans mot de passe, il faut positionner la directive suivante :

Directives à modifier :

    PermitEmptyPasswords no

- pour autoriser une seule liste d'utilisateurs ou de groupes utilisateur à se connecter, il faut positionner les directives suivantes :

Directives à modifier :

    AllowGroups groupe1,groupe2,...
    AllowUsers user1,user2,...

- pour tracer les tentatives de connexion échouées en détail, il faut positionner la directive suivante :

Directives à modifier :

    MaxAuthTries 4 # Après 2 tentatives, les traces seront détaillées dans les fichiers de log

- dans le meilleur des cas, il est possible d'indiquer au service SSH de procéder à l'authentification des comptes uniquement par échange de clés de cryptographie asymétrique (**duo clé privée/publique**). Ces clés doivent être générées de préférence avant le déploiement du service. Elles permettent de s'authentifier sans avoir à saisir de mot de passe, ce qui complique toute tentative d'espionnage de ces mots de passe. Les directives concernées sont :

Directives à modifier :

    PasswordAuthentication no
    PubkeyAuthentication yes

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez le durcissement avancé du service SSH.

# 4. Résumé des recommandations de toute cette partie

| Recommandation | Type | Principe |
| ----------- | ----------- | ----------- |
| Examiner la liste des _**sockets**_ **et ports ouverts** sur le réseau | ❌CRITICAL | Minimisation |
| Vérifier que les services qui ouvrent des **ports inutiles** sont désactivés | ❌CRITICAL | Minimisation |
| Vérifier que les services qui ouvrent des **ports** sur une interface **externe** sont pertinents | ❌CRITICAL | Défense en profondeur |
| Vérifier que les **ports par défaut** des services ont été changés dans la mesure du possible | 🚸WARNING | Défense en profondeur |
| Vérifier que les **TCP Wrappers** sont configurés pour les services réseau compatibles | ❌CRITICAL | Défense en profondeur |
| Vérifier qu'il est possible de rendre les services réseau **compatibles** avec les **TCP Wrappers** | 🚸WARNING | Défense en profondeur |
| Vérifier qu'il est possible d'accéder **au firewall Netfilter** par le noyau sans la contrainte module | 🚸WARNING  | Minimisation |
| Vérifier que les **règles Netfilter** sont bien positionnées | ❌CRITICAL | Défense en profondeur |
| Vérifier que les **règles Netfilter par défaut** sont définies dans le fichier de configuration | ❌CRITICAL | Défense en profondeur |
| Vérifier le positionnement des sysctl réseau communes du noyau dans le fichier **/etc/sysctl.conf** | ❌CRITICAL | Défense en profondeur |
| Vérifier la **version du protocole SSH** supporté par le service exploité sur le serveur | ❌CRITICAL | Défense en profondeur |
| Examiner la directive **PermitRootLogin** du service SSH | ❌CRITICAL | Moindre privilège |
| Vérifier le **port d'écoute du service SSH** | ❌CRITICAL | Défense en profondeur |
| Vérifier le **durcissement avancé** du service SSH | 🚸WARNING | Défense en profondeur |