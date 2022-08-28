# 0. Informations diverses

Dans la partie d'audit du **processus de démarrage**, nous avons listés les services lancés automatiquement lors du démarrage de la machine :

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
    lrwxrwxrwx 1 root root 40 7 mai 18:27 yum-cron.service -> /usr/lib/systemd/system/yum-cron.service


De plus, nous avons listés les services ainsi que leur état courant, en filtrant la commande sur le mot-clé **enabled** pour repérer les services lancés :

    [root@machine ~]# systemctl list-unit-files --type service | grep enabled
    auditd.service enabled
    autovt@.service enabled
    chronyd.service enabled
    crond.service enabled
    dbus-org.freedesktop.NetworkManager.service enabled
    dbus-org.freedesktop.nm-dispatcher.service enabled
    getty@.service enabled
    httpd.service enabled
    irqbalance.service enabled
    kdump.service enabled
    lvm2-monitor.service enabled
    mariadb.service enabled
    microcode.service enabled
    NetworkManager-dispatcher.service enabled
    NetworkManager-wait-online.service enabled
    NetworkManager.service enabled
    postfix.service enabled
    proftpd.service enabled
    rhel-autorelabel.service enabled
    rhel-configure.service enabled
    rhel-dmesg.service enabled
    rhel-domainname.service enabled
    rhel-import-state.service enabled
    rhel-loadmodules.service enabled
    rhel-readonly.service enabled
    rsyslog.service enabled
    sshd.service enabled
    systemd-readahead-collect.service enabled
    systemd-readahead-drop.service enabled
    systemd-readahead-replay.service enabled
    tuned.service enabled
    yum-cron.service enabled

Cette commande affiche tous les services lancés à un instant t, **sans distinction** entre les services système et services réseau.

Concernant les services utiles, il est nécessaire d'auditer leur configuration afin d'effectuer une opération de **durcissement** ou **hardening** en anglais.

Cette opération consiste à configurer les services pour **retarder voire empêcher** toute tentative de compromission.


# 1. La configuration des services lancés

## 1.1 Introduction

Prenons ici l'exemple du service **httpd.service** ou **apache2**.

## 1.2 Statut global du service httpd

Pour afficher son statut global, on lance la commande `systemctl status httpd.service` :

    [root@machine ~]# systemctl status httpd.service
    ● httpd.service - The Apache HTTP Server
    Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
    Active: active (running) since lun. 2019-05-13 07:39:47 CEST; 1h 26min ago
    Docs: man:httpd(8)
    man:apachectl(8)
    Main PID: 817 (httpd)
    Status: "Total requests: 0; Current requests/sec: 0; Current traffic: 0 B/sec"
    CGroup: /system.slice/httpd.service
    ├─817 /usr/sbin/httpd -DFOREGROUND
    ├─973 /usr/sbin/httpd -DFOREGROUND
    ├─974 /usr/sbin/httpd -DFOREGROUND
    ├─975 /usr/sbin/httpd -DFOREGROUND
    ├─976 /usr/sbin/httpd -DFOREGROUND
    └─977 /usr/sbin/httpd -DFOREGROUND

    mai 13 07:39:46 machine systemd[1]: Starting The Apache HTTP Server...
    mai 13 07:39:47 machine systemd[1]: Started The Apache HTTP Server.

Cette commande :
- confirme que le service est bien lancé,
- indique le fichier _unit_ associé **httpd.service**,
- indique les sections de la documentation,
- indique le PID (process ID) du processus principal (ici 817),
- indique la commande exécutée pour lancer le service.

Cependant, aucune trace du fichier de configuration du service, pourtant bien connu : **httpd.conf**.

On peut afficher le contenu du fichier avec la commande `cat` et en filtrant les lignes commentées afin qu'elles ne polluent pas l'affichage :

    [root@machine ~]# cat /etc/httpd/conf/httpd.conf | grep [a-z] | grep -v "#"
    ServerRoot "/etc/httpd"
    Listen 80
    Include conf.modules.d/*.conf
    User apache
    Group apache
    ServerAdmin root@localhost
    <Directory />
    AllowOverride none
    Require all denied
    </Directory>
    DocumentRoot "/var/www/html"
    <Directory "/var/www">
    AllowOverride None
    Require all granted
    </Directory>
    <Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
    </Directory>
    <IfModule dir_module>
    DirectoryIndex index.html
    </IfModule>
    <Files ".ht*">
    Require all denied
    </Files>
    ErrorLog "logs/error_log"
    LogLevel warn
    <IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    <IfModule logio_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog "logs/access_log" combined
    </IfModule>
    <IfModule alias_module>
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
    </IfModule>
    <Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
    </Directory>
    <IfModule mime_module>
    TypesConfig /etc/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
    </IfModule>
    AddDefaultCharset UTF-8
    <IfModule mime_magic_module>
    MIMEMagicFile conf/magic
    </IfModule>
    EnableSendfile on
    IncludeOptional conf.d/*.conf

Le fichier de configuration de HTTPD est sûrement celui par défaut, installé via le paquet de la distribution.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez le durcissement de la configuration des services lancés.

Concernant le service **HTTPD**, on peut déjà relever plusieurs actions prioritaires :
- Ajouter les directives **ServerTokens Prod** et **ServerSignature Off** pour ne pas afficher la bannière du service, qui renvoie par défaut les informations sur la version du service et système d'exploitation. _Une simple commande `telnet` ou `netcat` permet cependant de récupérer ces informations très utiles pour un attaquant._

Par exemple :

    [root@machine ~]# nc localhost 80
    HEAD / HTTP/1.0
    HTTP/1.1 400 Bad Request
    Date: Mon, 13 May 2019 07:26:27 GMT
    Server: Apache/2.4.6 (CentOS) PHP/5.4.16
    Content-Length: 226
    Connection: close
    Content-Type: text/html; charset=iso-8859-1

- Pour chaque répertoire, vérifier la présence de la directive **Options      -Indexes**, qui permet d'empêcher de lister le contenu du répertoire. Ce n'est pas le cas ici pour les répertoires références, pire, pour certains elle est même explicitement autorisée.

Par exemple :

    <Directory "/var/www/html">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

- Pour terminer, il faut limiter les méthodes HTTP autorisées selon notre besoin. La plupart du temps, les méthodes **GET**, **HEAD**, **POST** suffisent largement. Les autres comme **OPTIONS**, **PUT**, **DELETE**, **TRACE** et **CONNECT**, induisent des risques inutiles. Il faut vérifier la présence de la directive **LimitExcept** telle que :

Par exemple :

    <LimitExcept GET POST HEAD>
        deny from all
    </LimitExcept>


# 2. Le cloisonnement des services lancés

## 2.1 Introduction

> Le cloisonnement consiste à mettre en place des techniques pour exécuter des services dans un environnement d'accès aux ressources limitées et contrôlées.

En d'autres termes, le cloisonnement permet d'isoler les différents services installés sur une seule machine. Nous allons nous pencher sur les comptes système utilisés pour chaque service, et sur le processus d'isolation ou **chroot**.

## 2.2 Les comptes Linux associés aux services

Dans un premier temps, il est nécessaire de vérifier que les services sont lancés avec un compte (système ou utilisateur) qui dispose des privilèges nécessaires à l'exécution de ce service.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez les comptes systèmes associés aux services.

Dans la plupart des cas, le compte associé au service est indiqué dans les fichiers de configuration.

En reprenant l'exemple du service **HTTPD** :

    [root@machine ~]# ps -edf | grep httpd
    root 817 1 0 07:39 ? 00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache 973 817 0 07:39 ? 00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache 974 817 0 07:39 ? 00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache 975 817 0 07:39 ? 00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache 976 817 0 07:39 ? 00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache 977 817 0 07:39 ? 00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache 1634 817 0 09:26 ? 00:00:00 /usr/sbin/httpd -DFOREGROUND

Le processus **père** est lancé avec l'utilisateur **root** (première ligne), ce qui est indispensable pour que le service ouvre un port inférieur à 1000 sur le serveur.

Par contre, on peut constater que les **threads** (processus _**fils**_) sont attribués avec un utilisateur non privilégié, dans notre cas, **apache**.

On retrouve cette configuration dans le fichier **httpd.conf** :

    [root@machine ~]# grep apache /etc/httpd/conf/httpd.conf
    # See <URL:http://httpd.apache.org/docs/2.4/> for detailed information.
    # <URL:http://httpd.apache.org/docs/2.4/mod/directives.html>
    User apache
    Group apache
    # http://httpd.apache.org/docs/2.4/mod/core.html#options

Pour obtenir des informations sur le compte **apache**, lançons les commandes `id apache`, `grep apache /etc/passwd` et `grep apache /etc/group` :

    [root@machine ~]# id apache
    uid=48(apache) gid=48(apache) groupes=48(apache)
    [root@machine ~]# grep apache /etc/passwd
    apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
    [root@machine ~]# grep apache /etc/group
    apache:x:48:

L'UID du compte **apache** est inférieur à **100**. Généralement sur Linux, cette plage est réservée aux comptes système. De plus, on peut constater que le _shell_ du compte pointe vers **/sbin/nologin**, ce qui est une bonne configuration.

Il n'y a aucune raison qu'un compte système chargé de l'exécution de services puisse se connecter et obtienne un _shell_ sur le serveur.

Enfin, le compte **apache** possède son propre groupe primaire **apache**.

Lançons la commande `apachectl` évoquée plus tôt dans une autre partie, avec le filtre suivant :

    [root@machine ~]# apachectl -S | grep DocumentRoot
    Main DocumentRoot: "/var/www/html"

La **racine** (DocumentRoot) du serveur Apache est le répertoire par défaut des sources applicatives. Il peut en exister un différent pour chaque site hébergé par le service. On peut consulter les attributs du répertoire ave cla commande `ls -lrtha` :

    [root@machine ~]# ls -lrtha /var/www/
    total 4,0K
    drwxr-xr-x. 2 root root 6 5 nov. 2018 html
    drwxr-xr-x. 2 root root 6 5 nov. 2018 cgi-bin
    drwxr-xr-x. 4 root root 33 27 mars 09:43 .
    drwxr-xr-x. 21 root root 4,0K 27 mars 09:50 ..

Ici, le propriétaire du répertoire est l'utilisateur **root** et tout le monde peut y accéder. Cette configuration par défaut est inacceptable.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez les droits du système de fichiers associé aux comptes système exécutant des services.
>
> Par exemple, en modifiant les droits et propriétaires du répertoire racine d'apache via les commandes `chown` et `chmod` :

    chown -R apache:apache /var/www/html/
    chmod -R 750 /var/www/html/

## 2.3 Isolation des services lancés

Dans l'idéal, il faudrait attribuer une machine et un système d'exploitation à chaque service. Cette solution drastique est coûteuse en matériel et en maintenance.

Aujourd'hui, il existe plusieurs solutions d'isolation moins coûteuses :
- les conteneurs, où un même noyau gère plusieurs instances systèmes,
- la virtualisation où un hyperviseur gère plusieurs machines virtuelles émulées sur la même machine physique,
- un hyperviseur noyau, comme Linux KVM.

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez qu'il est possible de virtualiser l'architecture applicative du serveur.

Ici, nous pourrions envisager d'émuler deux machines virtuelles par exemple avec VirtualBox pour y installer sur l'une, le serveur Web Apache HTTPD et l'autre la base de données MySQL.

Bien entendu, ces deux machines devraient pouvoir communiquer via un réseau. Celui-ci pourrait être spécifique (nous y reviendrons plus tard).

Sous Linux, il existe également un processus de cloisonnement historique, nommé **chroot**.

Cette technique utilise l'argument obligatoire qui est passé à tous les processus lors de leur exécution, désignant leur racine dans l'arborescence et dont la valeur par défaut est **/**.

Le **chrootage** consiste à donner à cet argument une valeur différente de **/** pour ce répertoire (/directory par exemple).

Le résultat de cette opération limite le processus affecté dans cette sous-arborescence.

Par exemple, en exécutant la commande :

    chroot /directory /directory/command

La commande va alors considérer **/directory** comme sa **/** (racine).

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez que le chrootage est utilisé sur les services lancés dans la mesure du possible.

Par exemple, pour le service **HTTPD**. Un attaquant qui profite d'un défaut de configuration pourrait obtenir des accès sur l'arborescence du serveur.

Le processus de **chrootage** d'Apache limiterait ces accès.

La directive **ChrootDir** du module **unixd_module** permet de réaliser cette opération.

Il n'y a aucun sens à **chrooter certains services** comme **sshd** et ce pour plusieurs raisons :
- l'objectif de ssh étant de fournir un _shell_ distant aux utilisateurs connectés, il est nécessaire d'inclure toutes les commandes possibles dans l'environnement isolé.
- un admin se connectant via ssh aura bien du mal à effectuer une opération de maintenance ou de restauration de services s'il est restreint dans un environnement spécifique.

Ceci dit, l'opération reste faisable et il est possible d'isoler directement le processus ssh ainsi que les utilisateurs s'y connectant.

Pour cela, nous devrons utiliser respectivement la commande **chroot** ou le module **pam_chroot.so**.

Concernant le processus d'isolation **chroot**, il présente quelques faiblesses :
- le cloisonnement est **limité** en ce qui concerne les accès aux systèmes de fichiers, mais pas en ce qui concerne les accès aux processus ou aux réseaux,
- les utilisateurs peuvent **sortir** de leur environnement spécifique en utilisant des processus externes.

# 3. Les fichiers de trace des services

> Les fichiers de trace sont l'expression principale du système d'exploitation. Chaque évènement, normal ou anormal, y est consigné. Leur présence et la sécurisation du processus qui les maintient et les alimente est indispensable sur un serveur.

## 3.1 Introduction

Vérifions la présence des principaux fichiers de logs du système et que le processus qui les gère est bien cloisonné et qu'il existe bien un processus supervisant le contenu de ces fichiers.


## 3.2 La présence des principaux fichiers de trace

La commande `ls -lrtha /var/log` permet de nous en assurer :

    [root@machine ~]# ls -lrtha /var/log/
    ...
    -rw-r--r-- 1 root root 28K 13 mai 07:39 dmesg
    -rw-------. 1 root root 8,5K 13 mai 07:39 boot.log
    -rw------- 1 root root 416 13 mai 07:39 maillog
    -rw------- 1 root root 7,1K 13 mai 07:42 secure
    -rw-rw-r--. 1 root utmp 23K 13 mai 07:42 wtmp
    -rw-r--r--. 1 root root 287K 13 mai 07:42 lastlog
    -rw-------. 1 root root 4,9K 13 mai 09:26 yum.log
    -rw------- 1 root root 139K 13 mai 10:01 messages
    -rw------- 1 root root 6,9K 13 mai 10:01 cron

Les derniers fichiers de cette liste sont les derniers fichiers mis à jour. On y retrouve notamment :
- **messages** : ce fichier contient toutes les infos génériques sur l'activité du système. C'est souvent le premier fichier consulté lorsqu'un problème survient,
- **secure** : ce fichier contient toutes les informations sur le processus d'authentification. Il est très important de consulter le contenu de ce fichier régulièrement pour se tenir au courant des élévations de privilèges ou des tentatives de connexion,
- **boot.log** : contient toutes les informations loggées lors du processus de démarrage. Dans ce fichier, les infos concernent souvent les procédures de redémarrage afin de détecter d'éventuels échecs lors d'opérations associées,
- **dmesg** : c'est le fichier de trace du noyau. Toutes les intéractions entre le système d'exploitation et le matériel sont stockées dans ce fichier.

Chacun de ces fichiers joue un rôle très important dans la supervision du système d'exploitation. Ils doivent donc être présents et modifiables uniquement par **root**.

> ❌ **RECOMMANDATION-CRITICAL** (Moindre privilège) : Vérifiez la présence des principaux fichiers de logs du système et leurs droits d'accès.


## 3.3 Cloisonnement du service Syslog

Le service **Syslog** est utilisé par chaque processus pour écrire des informations dans les fichiers de trace évoqués ci-dessus.

**Syslog** est indispensable pour vérifier le fonctionnement du serveur. Les attaquants cherchant à effacer les traces de leur passage vont essayer de compromettre ce service ou de modifier les fichiers associés !

Le fichier de configuration du service **Syslog** est **/etc/rsyslog.conf**.

    [root@machine ~]# grep ^[^#] /etc/rsyslog.conf
    $ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
    $ModLoad imjournal # provides access to the systemd journal
    $WorkDirectory /var/lib/rsyslog
    $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
    $IncludeConfig /etc/rsyslog.d/*.conf
    $OmitLocalLogging on
    $IMJournalStateFile imjournal.state
    *.info;mail.none;authpriv.none;cron.none /var/log/messages
    authpriv.* /var/log/secure
    mail.* -/var/log/maillog
    cron.* /var/log/cron
    *.emerg :omusrmsg:*
    uucp,news.crit /var/log/spooler
    local7.* /var/log/boot.log

Chaque canal de trace est défini en indiquant le fichier destinataire des traces.

Par exemple, le canal **mail.*** est défini dans le fichier destinataire **-/var/log/maillog**.

> ❌ **RECOMMANDATION-CRITICAL** (Défense en profondeur) : Vérifiez le cloisonnement du service Syslog.

Ce cloisonnement passe par plusieurs opérations distinctes :
- **privilégier une partition spécifique pour les fichiers de trace**, afin de nous assurer qu'un processus tiers ne viendra pas saturer la partition des fichiers de trace,
- **instaurer un processus de rotation des fichiers de trace en les compressant**. Les fichiers de logs sont souvent volumineux et la création de nouveaux fichiers de manière régulière permet une maintenance plus simple des archives. Cette configuration est stockée dans le fichier **/etc/logrotate.conf**.

Nous pouvons voir son contenu avec la commande :

    [root@machine ~]# grep ^[^#] /etc/logrotate.conf
    weekly
    rotate 4
    create
    dateext
    include /etc/logrotate.d
    /var/log/wtmp {
    monthly
    create 0664 root utmp
    minsize 1M
    rotate 1
    }
    /var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
    }

- **externaliser et centraliser les fichiers de trace**. Dans la mesure du possible, écrivez les traces dans les fichiers de l'arborescence locale, mais aussi dans des fichiers distants, en passant par le service réseau d'un serveur de centralisation des logs. Pour cela, il suffit d'indiquer dans le fichier de configuration **/etc/rsyslog.conf**, la destination réseau du fichier distant (UDP ou TCP) -> _déjà fait dans un projet OC et en sécurisé._

Par exemple :

    # Log anything (except mail) of level info or higher.
    # Don't log private authentication messages!
    *.info;mail.none;authpriv.none;cron.none /var/log/messages
    *.info;mail.none;authpriv.none;cron.none @@remote-server:port


## 3.4 Supervision des fichiers de trace

Il est important de mettre en place un service cloisonné et sécurisé de gestion des traces.

Cependant, les évènements critiques, les tentatives de connexions échouées, les requêtes vers des ressources Web inexistantes figureront toujours dans les fichiers de trace.

Il est donc primordial de superviser ces fichiers car si on ne les lit pas, on ne sait pas ce qu'il se passe sur le serveur.

> 🚸 **RECOMMANDATION-WARNING** (Défense en profondeur) : Vérifiez qu'il y a un processus de supervision des fichiers de trace.

Il existe plusieurs façons de connaître le contenu de ces fichiers :
- dégager du temps _humain_ pour **consulter manuellement** le contenu des fichiers. C'est la technique la plus simple, mais la plus coûteuse en termes de temps,
- installer un **processus de supervision automatique** du contenu des fichiers. Par exemple, des sondes **Nagios** qui viendraient consulter, à intervalle de temps régulier, les lignes des fichiers, afin de lever des alertes lorsque nécessaire,
- installer un **parseur de fichiers de trace**. Celui-ci permet de recevoir par mail, un résumé des lignes jugées critiques dans tous les fichiers parsés. **Logwatch** est très efficace en la matière.

# 4. Résumé des recommandations de toute cette partie

| Recommandation      | Type | Principe |
| ----------- | ----------- | ----------- |
| Vérifier le **durcissement de la configuration** des services lancés     | ❌CRITICAL | Défense en profondeur |
| Vérifier les **comptes système** associés aux services      | ❌CRITICAL | Moindre privilège |
| Vérifier les **droits du système de fichiers** associé aux comptes système exécutant des services     | ❌CRITICAL | Moindre privilège |
| Vérifier qu'il est possible de **virtualiser l'architecture applicative** du serveur      | 🚸WARNING | Défense en profondeur |
| Vérifier que le **chrootage** est utilisé sur les services lancés dans **la mesure du possible**     | ❌CRITICAL | Défense en profondeur |
| Vérifier la présence des principaux **fichiers de trace** du système et leurs droits d'accès      | ❌CRITICAL | Moindre privilège |
| Vérifier le **cloisonnement du service Syslog**     | ❌CRITICAL | Défense en profondeur |
| Vérifier le processus de **supervision des fichiers de trace**      | 🚸WARNING | Défense en profondeur |