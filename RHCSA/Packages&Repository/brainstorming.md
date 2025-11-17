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

```bash
dnf --version
```

```bash

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

### Gestion des paquets avec dnf

```bash
# rechercher un paquet
dnf search <nom_du_paquet>
# obtenir des informations sur un paquet
dnf info <nom_du_paquet>
# installer un paquet
dnf install <nom_du_paquet>
# désinstaller un paquet
dnf remove <nom_du_paquet>
# mettre à jour les paquets installés
dnf update
# nettoyer le cache
dnf clean all
# lister les paquets installés
dnf list installed
# lister les paquets disponibles
dnf list available
# vérifier les dépendances
dnf check
# afficher les détails d'un paquet installé
dnf repoquery --installed <nom_du_paquet>
# afficher les fichiers d'un paquet installé
dnf repoquery --list <nom_du_paquet>
# afficher les dépendances d'un paquet
dnf repoquery --requires <nom_du_paquet>
# afficher les paquets qui dépendent d'un paquet
dnf repoquery --whatrequires <nom_du_paquet>
# afficher les paquets fournis par un paquet
dnf repoquery --provides <nom_du_paquet>
# afficher les paquets obsolètes
dnf repoquery --obsoletes <nom_du_paquet>
# afficher les paquets orphelins
dnf repoquery --orphans
# afficher les paquets de sécurité
dnf repoquery --security
# afficher les paquets mis à jour récemment
dnf repoquery --last
# afficher les paquets installés par date
dnf repoquery --installed --last
# afficher les paquets installés par taille
dnf repoquery --installed --size
# afficher les paquets installés par groupe
dnf group list
# afficher les paquets installés par groupe
dnf group info "<nom_du_groupe>"
# installer un groupe de paquets
dnf group install "<nom_du_groupe>"
# supprimer un groupe de paquets
dnf group remove "<nom_du_groupe>"
# mettre à jour un groupe de paquets
dnf group update "<nom_du_groupe>"
# nettoyer les paquets orphelins
dnf autoremove
# afficher l'historique des transactions
dnf history
# annuler une transaction
dnf history undo <ID_transaction>
# rétablir une transaction
dnf history redo <ID_transaction>
# afficher les détails d'une transaction
dnf history info <ID_transaction>
# répertorier les transactions
dnf history list
# restaurer le système à un état antérieur
dnf history rollback <ID_transaction>
# afficher les plugins disponibles
dnf plugin list
# activer un plugin
dnf plugin enable <nom_du_plugin>
# désactiver un plugin
dnf plugin disable <nom_du_plugin>
# afficher les paramètres de configuration
dnf config-manager --dump
# check si un paquet est signé
dnf check-signature <nom_du_paquet>
# check si un paquet est corrompu
dnf verify <nom_du_paquet>
#check si un paquet est installé
dnf list <nom_du_paquet>
```


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

