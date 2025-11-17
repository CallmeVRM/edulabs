---
title: brainstorming
grand_parent: packages & repository
parent: RHCSA
nav_order: 40
has_children: true
nav_exclude: true
---

# Brainstorming - Packages & Repository
yum vs dnf : 

```bash
# Yum n'est qu'un lien symbolique vers dnf
ls -la /usr/bin/yum
```



---

**Repositories par defaut sur RHEL 10 :**
- BaseOS : Tous les paquets sont au format RPM, c'est les paquets fondamentaux nécessaires au fonctionnement de l'OS.
- AppStream : Contient des paquets RPM, mais aussi des extensions appelé modules.


**Format RPM traditionnel :**
- Permet l'installation d'un paquet.
- Un seul numéro de version possible pour un paquet donné.
- Les dépendances sont séparées, et non regroupées ou versionnées conjointement.

**Format Module (AppStream) :**
- Etend le format RPM via les modules regroupant applications + dépendances.
- Permet d'accèder à plusieurs version du même logiciel (paquet) via les streams.
- Les profils permettent de choisir précisement ce qui est installé (ex. uniquement le client d'une base de données, sans le serveur).


### Les repositories :
```bash 
# afficher le status de la souscription
subscription-manager status

# lister les repositories disponibles
subscription-manager repos --list
dnf repolist



```




---

## Lab - Gestion des paquets avec dnf et les modules.

