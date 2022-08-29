# 0. Informations diverses

Il est temps de commencer à rédiger le rapport.

Nous verrons en premier la première page du rapport qui s'adresse plus spécifiquement à notre commanditaire, généralement le DSI.

Puis nous verrons comment remplir les pages suivantes, qui s'adressent aux techniciens qui vont appliquer les recommandations.

# 1. Les informations indispensables à faire figurer sur la première page

Un rapport d'audit est un document qui sera soigneusement conservé. Il peut servir à justifier une certification par exemple.

Il ser ainsi probablement relu plusieurs fois longtemps après la réalisation de l'audit.

> La première page d'un rapport d'audit est toujours particulière. Elle doit présenter de manière synthétique les informations principales attendues par le commanditaire, qu'elles soient négatives ou positives. Il arrive d'ailleurs souvent que cette page soit suffisante. Le détail des informations est rédigé dans les pages suivantes, voire dans les annexes.

La première page doit donc contenir toutes les informations permettant de re-situer le contexte de l'audit.

On y trouvera notamment les informations générales suivantes :
- les **dates de réalisation** de l'audit et de **rédaction** du rapport,
- le **nom de l'auditeur** et son **organisme juridique d'affiliation**,
- le **titre** du rapport, par exemple "Audit de sécurité du serveur hébergeant l'application XYZ",
- la liste des **destinataires** du rapport, avec leur nom et leur titre.

Il est également nécessaire de rappeler :
- **l'identité du commanditaire**,
- les **objectifs** de l'audit (sécurisation en deux étapes, dans un contexte existant puis dans un autre, par exemple).

Nous pouvons aussi résumer très succinctement le **mode opératoire** de l'auditeur :
- la façon dont il a procédé et depuis quel lieu,
- les outils et logiciels utilisés,
- les données initiales possédées par l'auditeur (le compte **root** par exemple),
- les personnes rencontrées ou avec lesquelles l'auditeur a communiqué lors de l'audit.

Enfin, sûrement le plus important, il faut indiquer les conclusions de l'audit de manière claire et succincte, par exemple :
- le nombre de **recommandations totales**,
- le nombre de **recommandations critiques**,
- le nombre de **recommandations moins critiques**,
- une **conclusion** globale sous la forme d'une recommandation unique, par exemple :

"Le serveur présente des défauts de sécurité critiques. Les recommandations critiques n'induisant pas de changement opérationnel pour le serveur doivent être exécutées dans les plus brefs délais, elles représentent une estimation de XX jours hommes de charge."

> 🚸 ATTENTION : Sur la première page, les informations sont **nombreuses** et doivent être très **qualitatives**. Il ne faut pas négliger leur présentation, au risque de rendre la première poage illisible. Nous pouvons utiliser le recto et le verso de la page, les commanditaires n'hésiteront pas à lire les deux. Dans ce cas, indiquez clairement qu'il faut tourner la page (avec un signe par exemple).

# 2. Page d'exemple citée sur le cour d'audit


![Première page exemple d'audit de sécurité](/première_page_audit.png "Première page exemple d'audit de sécurité")

# 3. Écrire le reste du rapport

La plupart du temps, le reste du rapport n'est pas destiné au même lecteur. Souvent, la première page est à destination des **managers** tandis que le reste concerne plutôt les **équipes techniques**.

Il s'agit donc dans le reste du rapport, de détailler chaque recommandation de façon à ce que le lecteur (avec un profil technique) puisse se représenter notre mode opératoire et s'imaginer ce qui vous a permis de rédiger cette recommandation.

Il est souvent efficace de présenter cette partie sous forme de fiche.

En prenant la recommandation suivante comme exemple :

| N°      | Recommandation | Type | Principe |
| ----------- | ----------- | ----------- | ----------- |
| SYS.11   | Examiner la liste des fichiers avec les droits spéciaux **setuid**, **setgid** et **sticky bit**  | ❌CRITICAL  | Moindre privilège  |

A partir de cette recommandation, nous pourrions imaginer la fiche suivante :

| SYS.11      | Fichiers à droits spéciaux |
| ----------- | ----------- |
| Objectif   | Examiner la liste des fichiers avec les droits spéciaux **setuid**, **setgid** et **sticky bit**  |
| Principe associé   | Moindre privilège  |
| Type   | ❌CRITICAL  |
| Explications   | Les fichiers disposant de ces droits spéciaux s'exécutent avec les privilèges de l'utilisateur ou du groupe, et non avec ceux de l'utilisateur qui les exécute. C'est donc un processus d'**élévation de privilèges** qui peut représenter une faille de sécurité si les fichiers concernés ne prennent pas en compte cette élévation  |
| Exemple   | La commande **/usr/bin/passwd** s'exécute avec les privilèges de **root** et permet à tout utilisateur de changer son mot de passe de connexion  |
| Mode opératoire   | Une fois connecté en tant que **root**, lister les fichiers disposant de ces droits spéciaux avec la commande suivante : **find / -type f -perm /6000 -ls 2>/dev/null**  |
| Résultat    | Voir le code ci-dessous  |
| Recommandation   | À comparer avec un référentiel connu et fiable, notamment celui proposé en page 33 du guide de recommandation de sécurité de l'ANSSI : https://www.ssi.gouv.fr/administration/guide/recommandations-de-securite-relatives-a-un-systeme-gnulinux/ -> Retirer les bits de droits spéciaux lorsque non nécessaires en fonction du contexte du serveur. Par exemple ici : **newgrp**, **postdrop**, **postqueue**, **wall**, **chsh**, **chfn**, …|

Résultat :

    [root@machine ~]# find / -type f -perm /6000 -ls 2>/dev/null
    12638690 16 -r-xr-sr-x 1 root tty 15344 juin 10 2014 /usr/bin/wall
    12849314 24 -rws--x--x 1 root root 24048 avril 11 2018 /usr/bin/chfn
    12849317 24 -rws--x--x 1 root root 23960 avril 11 2018 /usr/bin/chsh
    12849350 44 -rwsr-xr-x 1 root root 44320 avril 11 2018 /usr/bin/mount
    12849198 64 -rwsr-xr-x 1 root root 64240 nov. 5 2016 /usr/bin/chage
    12849199 80 -rwsr-xr-x 1 root root 78216 nov. 5 2016 /usr/bin/gpasswd
    12849201 44 -rwsr-xr-x 1 root root 41776 nov. 5 2016 /usr/bin/newgrp
    12849365 32 -rwsr-xr-x 1 root root 32184 avril 11 2018 /usr/bin/su
    12849369 32 -rwsr-xr-x 1 root root 32048 avril 11 2018 /usr/bin/umount
    12849375 20 -rwxr-sr-x 1 root tty 19624 avril 11 2018 /usr/bin/write
    13008451 140 -rwsr-x--- 1 root root 143184 avril 11 2018 /usr/bin/sudo
    12897988 28 -rwsr-xr-x 1 root root 27680 avril 11 2018 /usr/bin/pkexec
    12898157 60 -rwsr-xr-x 1 root root 57576 avril 11 2018 /usr/bin/crontab
    12969583 376 ---x--s--x 1 root nobody 382240 avril 11 2018 /usr/bin/ssh-agent
    13008558 28 -rwsr-xr-x 1 root root 27832 juin 10 2014 /usr/bin/passwd
    70193 12 -rwsr-xr-x 1 root root 11216 avril 11 2018 /usr/sbin/pam_timestamp_check
    70195 36 -rwsr-xr-x 1 root root 36280 avril 11 2018 /usr/sbin/unix_chkpwd
    269843 12 -rwxr-sr-x 1 root root 11224 avril 11 2018 /usr/sbin/netreport
    269848 12 -rwsr-xr-x 1 root root 11288 avril 11 2018 /usr/sbin/usernetctl
    344902 216 -rwxr-sr-x 1 root postdrop 218552 juin 10 2014 /usr/sbin/postdrop
    344909 256 -rwxr-sr-x 1 root postdrop 259992 juin 10 2014 /usr/sbin/postqueue
    12897991 16 -rwsr-xr-x 1 root root 15432 avril 11 2018 /usr/lib/polkit-1/polkit-agent-helper-1
    12849263 12 -rwx--s--x 1 root utmp 11192 juin 10 2014 /usr/libexec/utempter/utempter
    12879166 60 -rwsr-x--- 1 root dbus 58016 avril 11 2018 /usr/libexec/dbus-1/dbus-daemon-launch-helper
    271691 460 ---x--s--x 1 root ssh_keys 469880 avril 11 2018 /usr/libexec/openssh/ssh-keysign

Nous pouvons donc procéder de la même manière avec toutes nos recommandations effectuées précédemment sur notre machine VirtualBox.

Une fois toutes nos fiches constituées, nous aurons une version finalisée du rapport d'audit que nous pourrons adresser aux différents destinataires.

# 4. S'entraîner à rédiger des fiches formalisées de recommandations

Pour rappel, une fiche de recommandation comprend les éléments suivants :
- **Nom** de la recommandation,
- **Objectif** de la recommandation,
- **Principe de sécurisation associé** : minimisation, moindre privilège, défense en profondeur,
- **Type** : ❌CRITICAL ou 🚸WARNING,
- **Explications** : description du problème,
- **Exemple** : de ce problème,
- **Mode opératoire** : comment opérer cette recommandation techniquement,
- **Résultat** : copie du résultat (code) obtenu,
- **Recommandation** : la ou les action(s) à prendre par l'équipe technique suite au résultat obtenu.