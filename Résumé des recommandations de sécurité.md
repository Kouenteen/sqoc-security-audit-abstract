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




# 5. Rapport d'audit ?