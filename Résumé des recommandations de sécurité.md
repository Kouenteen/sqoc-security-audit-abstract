# 0. Résumé cité dans le cours sur l'audit

**PARTIE 1 : PROCESSUS DE DÉMARRAGE**

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| BOO.1   | Passer les droits sur l'arborescence **/etc/grub.d** à **700**  | ❌CRITICAL  | Moindre privilège  |
| BOO.2   | Créer un utilisateur et son **mot de passe chiffré** dans le fichier **01_users** afin de protéger l'accès au shell de Grub par une authentification  | ❌CRITICAL  | Moindre privilège  |
| BOO.3   | Passer l'option **iommu=force** au noyau lors du démarrage de Linux  | 🚸WARNING  | Minimisation  |
| BOO.4   | Bloquer le chargement de modules supplémentaires via la sysctl **kernel.modules_disabled=1**  | 🚸WARNING  | Minimisation  |
| BOO.5   | Vider le contenu du fichier **/etc/securetty** afin de bloquer toute connexion avec l'utilisateur root depuis une console virtuelle  | ❌CRITICAL  | Défense en profondeur  |
| BOO.6   | Augmenter l'intervalle minimal de temps entre chaque tentative de connexion sur le module **pam_faildelay.so** du fichier **/etc/pam.d/system-auth** à 5 ou 10 secondes afin de ralentir les attaques par dictionnaire  | 🚸WARNING  | Défense en profondeur  |
| BOO.7   | Désactiver la combinaison **Ctrl+Alt+Supp** sur le serveur pour prévenir tout redémarrage depuis un accès physique à la machine  | ❌CRITICAL  | Défense en profondeur  |
| BOO.8   | Désactiver les **Magic System Request Keys**  | 🚸WARNING  | Défense en profondeur  |
| BOO.9   | Supprimer les **services inutiles** démarrés automatiquement avec le serveur en passant par la **cible par défaut**  | ❌CRITICAL  | Minimisation  |
| BOO.10   | Supprimer les **services inutiles** démarrés automatiquement avec le serveur en passant par les **dépendances** de la cible par défaut  | 🚸WARNING  | Minimisation  |

**PARTIE 2 : SYSTÈME**

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| SYS.1   | Vérifier que le CPU dispose bien des flags **PAE** et **NX**  | ❌CRITICAL  | Défense en profondeur  |
| SYS.2   | Vérifier la présence d'un **minimum de mémoire SWAP** sur le système  | 🚸WARNING  | Défense en profondeur  |
| SYS.3   | Vérifier le **chiffrement des partitions** sensibles du système  | ❌CRITICAL  | Défense en profondeur  |
| SYS.4   | Vérifier que le **partitionnement** isole et protège les composants du système  | ❌CRITICAL  | Défense en profondeur  |
| SYS.5   | Vérifier les options des **points de montage** des systèmes de fichiers  | ❌CRITICAL  | Moindre privilège  |
| SYS.6   | Vérifier la **protection de la partition /boot**  | ❌CRITICAL  | Moindre privilège  |
| SYS.7   | 	
Vérifier que les **mots de passe** du module PAM sont **en mode shadow**  | ❌CRITICAL  | Défense en profondeur  |
| SYS.8   | Vérifier la robustesse des mots de passe avec le module **pam_pwquality**  | 🚸WARNING  | Défense en profondeur  |
| SYS.9   | Vérifier que les comptes utilisateurs qui peuvent se connecter ont pour obligation de **changer leur mot de passe régulièrement**  | ❌CRITICAL  | Défense en profondeur  |
| SYS.10   | Vérifier les valeurs par défaut des attributs des mots de passe pour chaque compte utilisateur dans **/etc/login.defs**  | 🚸WARNING  | Défense en profondeur  |
| SYS.11   | Examiner la liste des fichiers avec les droits spéciaux **setuid**, **setgid** et **sticky bit**  | ❌CRITICAL  | Moindre privilège  |
| SYS.12   | Vérifier la présence d'un **groupe d'utilisateurs** identifié comme **administrateur** de la machine et disposant des droits de changement de privilèges par l'intermédiaire d'un processus type sudo  | ❌CRITICAL  | Moindre privilège  |
| SYS.13   | Vérifier que la commande **sudo** peut être exécutée uniquement par le groupe administrateur (+ root)  | ❌CRITICAL  | Moindre privilège  |
| SYS.14   | Vérifier que le fichier de configuration **/etc/sudoers** contient la déclaration des commandes nécessaires au maintien du système pour les utilisateurs du groupe administrateur  | ❌CRITICAL  | Moindre privilège  |
| SYS.15   | 	
Vérifier que les **packages** installés sur le système sont **signés et sûrs**  | ❌CRITICAL  | Défense en profondeur  |
| SYS.16   | Vérifier que le **plugin gérant les mises à jour de sécurité des packages** de la distribution est bien installé  | ❌CRITICAL  | Défense en profondeur  |
| SYS.17   | Vérifier la planification régulière d'une tâche automatique pour **mettre à jour les packages corrigés pour raisons de sécurité**  | ❌CRITICAL  | Défense en profondeur  |

**PARTIE 3 : SERVICES**

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| SER.1   | Vérifier le **durcissement de la configuration** des services lancés  | ❌CRITICAL  | Défense en profondeur  |
| SER.2   | Vérifier les **comptes système** associés aux services  | ❌CRITICAL  | Moindre privilège  |
| SER.3   | Vérifier les **droits du système de fichiers** associé aux comptes système exécutant des services  | ❌CRITICAL  | Moindre privilège  |
| SER.4   | Vérifier qu'il est possible de **virtualiser l'architecture** applicative du serveur  | 🚸WARNING  | Défense en profondeur  |
| SER.5   | Vérifier que le **chrootage** est utilisé sur les services lancés dans la mesure du possible  | ❌CRITICAL  | Défense en profondeur  |
| SER.6   | Vérifier la présence des principaux **fichiers de trace** du système et leur droit d'accès  | ❌CRITICAL  | Moindre privilège  |
| SER.7   | Vérifier le **cloisonnement du service Syslog**  | ❌CRITICAL  | Défense en profondeur  |
| SER.8   | Vérifier le processus de **supervision des fichiers de trace**  | 🚸WARNING  | Défense en profondeur  |

**PARTIE 4 : RÉSEAU**

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| NET.1   | Examiner la liste des **sockets et ports ouverts** sur le réseau  | ❌CRITICAL  | Minimisation  |
| NET.2   | Vérifier que les services qui ouvrent des **ports inutiles** sont désactivés  | ❌CRITICAL  | Minimisation  |
| NET.3   | Vérifier que les services qui ouvrant des **ports** sur une interface **externe** sont pertinents  | ❌CRITICAL  | Défense en profondeur  |
| NET.4   | Vérifier que les **ports par défaut** des services ont été changés dans la mesure du possible  | 🚸WARNING  | Défense en profondeur  |
| NET.5   | Vérifier que les **TCP Wrappers** sont configurés pour les services réseau compatibles  | ❌CRITICAL  | Défense en profondeur  |
| NET.6   | Vérifier qu'il est possible de rendre les services réseau **compatibles** avec les **TCP Wrappers**  | 🚸WARNING  | Défense en profondeur  |
| NET.7   | 	
Vérifier qu'il est possible d'accéder au **firewall Netfilter** par le noyau sans la contrainte module  | 🚸WARNING  | Minimisation  |
| NET.8   | Vérifier que les **règles Netfilter** sont bien positionnées  | ❌CRITICAL  | Défense en profondeur  |
| NET.9   | Vérifier que les **règles Netfilter par défaut** sont définies dans le fichier de configuration  | ❌CRITICAL  | Défense en profondeur  |
| NET.10   | Vérifier le positionnement des sysctl réseau communes du noyau dans le fichier **/etc/sysctl.conf**  | ❌CRITICAL  | Défense en profondeur  |
| NET.11   | Vérifier la **version du protocole SSH** supporté par le service exploité sur le serveur  | ❌CRITICAL  | Défense en profondeur  |
| NET.12   | Examiner la directive **PermitRootLogin** du service SSH  | ❌CRITICAL  | Moindre privilège  |
| NET.13   | Vérifier le **port d'écoute du service SSH**  | ❌CRITICAL  | Défense en profondeur  |
| NET.14   | Vérifier le **durcissement avancé** du service SSH  | 🚸WARNING  | Défense en profondeur  |



# 0.5 Résumé indépendant de chaque partie du cours

# 1. Processus de démarrage

| Recommandation      | Type | Principe   |
| ----------- | ----------- | -----------|
| Passer les droits sur l'arborescence **/etc/grub.d/** à **700**      | ❌CRITICAL       | Moindre privilège |
| Créer un user et son **mot de passe chiffré** dans le fichier **01_users** afin de protéger l'accès au `shell` de GRUB2 par une authentification  | ❌CRITICAL        | Minimisation |
| Passer l'option **iommu=force** au noyau lors du démarrage de Linux      | 🚸WARNING       | Minimisation |
| Bloquer le chargement de modules supplémentaires via la commande **sysctl kernel.modules_disabled=1**   | 🚸WARNING        | Minimisation |
| Vider le contenu du fichier **/etc/securetty** afin de bloquer toute connexion avec l'utilisateur root depuis une console virtuelle      | ❌CRITICAL       | Défense en profondeur |
| Augmenter l'intervalle minimal de temps entre chaque tentative de connexion sur le module **pam_faildelay.so** du fichier **/etc/pam.d/system-auth** à 5 ou 10 secondes afin de ralentir les attaques par dictionnaire   | 🚸WARNING        | Défense en profondeur |
| Désactiver la combinaison **Ctrl+Alt+Supp** sur le serveur pour prévenir tout redémarrage depuis un accès physique à la machine      | ❌CRITICAL       | Défense en profondeur |
| Désactiver les **Magic System Request Keys**   | 🚸WARNING        | Défense en profondeur |
| Supprimer les **services inutiles** démarrés automatiquement avec le serveur en passant par **la cible par défaut**      | ❌CRITICAL       | Minimisation |
| Supprimer les **services inutiles** démarrés automatiquement avec le serveur en passant par **les dépendances de la cible par défaut**   | 🚸 WARNING        | Minimisation |


# 2. Le système

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


# 3. Les services

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

## 3.1 Activité audit configuration de base serveur MariaDB

**Recommandations de sécurité exécutées par le script** `mysql_secure_installation`

| Recommandation      | Type | Principe | Commande associée |
| ----------- | ----------- | ----------- | ----------- |
| Vérifier la force du mot de passe de l'utilisateur SQL **root** disposant de tous les droits sur toutes les bases de données     | ❌CRITICAL | Défense en profondeur | mysqladmin password UnPasswordTresLongEtTresComplique |
| Vérifier que les comptes anonymes ne peuvent pas se connecter sur la base      | ❌CRITICAL | Moindre privilège | use mysql; drop user ""@"localhost"; flush privileges; |
| Désactiver les connexions distantes de l'utilisateur **root**      | ❌CRITICAL | Défense en profondeur | DELETE FROM mysql.user WHERE user='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1'); FLUSH PRIVILEGES; |
| Supprimer la base **test** installée par défaut      | ❌CRITICAL | Minimisation | DELETE FROM mysql.db WHERE db LIKE 'test%'; FLUSH PRIVILEGES |


**Recommandations supplémentaires avec 2 qui concernent phpMyAdmin**

| Recommandation | Type | Principe | Commande associée |
| ----------- | ----------- | ----------- | ----------- |
| Vérifier qu'il existe bien un compte SQL applicatif dédié au schéma de base de données de l'application fichesproduits | ❌CRITICAL | Défense en profondeur | use mysql; select * from user; |
| Vérifier que le compte SQL applicatif dédié au schéma de base de données de l'application fichesproduits dispose des droits minimums et nécessaires sur le schéma | 🚸WARNING | Moindre privilège | use mysql; grant select, insert, update, delete privileges on mydb.* to utilisateur@"localhost" identified by 'motDePasseDuCompteutilisateur'; |
| Vérifier que le service est disponible uniquement pour les connexions locales | ❌CRITICAL | Défense en profondeur | echo "bind-address = 127.0.0.1" >> /etc/my.cnf |
| Changer le port d'écoute par défaut du service | 🚸WARNING | Défense en profondeur | echo "Port = 6033" >> /etc/my.cnf |
| Vérifier la gestion des fichiers de traces du service | 🚸WARNING | Minimisation | echo "log=/var/log/mysql.log" >> /etc/my.cnf |
| Change l'url d'accès par défaut à phpMyAdmin | ❌CRITICAL | Défense en profondeur | Modifier le fichier **/etc/httpd/conf.d/phpMyAdmin.conf** et notamment les lignes suivantes : console Alias /phpMyAdmin /usr/share/phpMyAdmin Alias /phpMyAdmin /usr/share/phpMyAdmin |
| Désactiver la connexion du compte SQL **root** sur phpMyAdmin | ❌CRITICAL | Moindre privilège | Modifier le fichier **/etc/phpMyAdmin/config.inc.php** et notamment la ligne suivante : $cfg['Servers'][$i]['AllowRoot'] = TRUE; // whether to allow root login |


# 4. Les accès réseaux

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
