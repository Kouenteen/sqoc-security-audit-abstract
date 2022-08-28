# 0. Informations diverses

# 1. Premier bilan de l'audit

## 1.1 Résumé des recommandations d'audit

Disponible sur le fichier "Résumé des recommandations..."

## 1.2 Triez les actions prioritaires en fonction du besoin

> Les objectifs de l'audit. Ils visaient à répondre à deux interrogations principales :
- quelles sont les actions de sécurisation à réaliser immédiatement sur la machine hébergée à l'extérieur ?
- quelles sont les actions de sécurisation à prévoir lorsque la plateforme applicative sera hébergée en interne ?

Prenons l'exemple des recommandations de la catégorie **BOO** qui concernent le processus de démarrage.

Le serveur étant hébergé à distance, il peut être difficile de réaliser toutes ces recommandations.

C'est d'autant plus difficile que, pour suivre certaines d'entre elles, nous devrions forcément être physiquement devant la machine (pour bloquer les options de démarrage Grub).

Enfin, il nous faut aussi différencier les actions selon leur _**type**_ **(priorité)** : "❌CRITICAL", à faire quel que soit le contexte et "🚸WARNING" moins sensibles voire optionnelles en fonction du contexte.

C'est pourquoi, il faudrait trier les recommandations pour cette situation, en 4 axes :
- dans le **contexte actuel** (donc sur le serveur hébergé à distance) :
    - **❌CRITICAL**,
    - **🚸WARNING**.
- lorsque le serveur **sera sur le réseau d'entreprise** :
    - **❌CRITICAL**,
    - **🚸WARNING**.

# 2. Croiser notre analyse avec un outil d'audit automatique

