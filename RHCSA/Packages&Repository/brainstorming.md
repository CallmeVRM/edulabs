---
title: brainstorming
grand_parent: RHCSA
parent: Packages & Repositories
nav_order: 10
has_children: true
nav_exclude: false
---



# Brainstorming - Packages & Repository
## H2 - Hello World
### H3 - Hello World
#### H4 - Hello World
##### H5 - Hello World
###### H6 - Hello World

**Repositories par defaut sur RHEL 10 :**
- BaseOS : Tous les paquets sont au format RPM, c'est les paquets fondamentaux nécessaires au fonctionnement de l'OS.
- AppStream : Contient des paquets RPM, mais aussi des extensions appelé modules.

Les fichiers des répertoires sont situés dans le répertoire `/etc/yum.repos.d/` et ont l'extension `.repo`. Le fichier standard `redhat.repo` est géré automatiquement par le système via le Subscription Manager.
Un fichier repo unique peut contenir plusieurs repositories comme source.

**Format RPM traditionnel :**
- Permet l'installation d'un paquet.
- Un seul numéro de version possible pour un paquet donné.
- Les dépendances sont séparées, et non regroupées ou versionnées conjointement.

**Format Module (AppStream) :**
- Etend le format RPM via les modules regroupant applications + dépendances.
- Permet d'accèder à plusieurs version du même logiciel (paquet) via les streams.
- Les profils permettent de choisir précisement ce qui est installé (ex. uniquement le client d'une base de données, sans le serveur).


### Les repositories :

#### Comprendre où sont stockés les repositories

Les fichiers des répertoires sont situés dans le répertoire `/etc/yum.repos.d/` et ont l'extension `.repo`. Le fichier standard `redhat.repo` est géré automatiquement par le système via le Subscription Manager.
Un fichier repo unique peut contenir plusieurs repositories comme source.

Il contient :

- les dépôts AppStream et BaseOS,
- et une grande quantité de dépôts désactivés par défaut.

RHEL 8 utilise dnf-3 (appelé via yum ou dnf) pour accéder aux dépôts logiciels.

Pour lister les dépôts activés :
```bash
dnf repolist
```
Pour lister tous les dépôts, y compris ceux désactivés :
```bash
dnf repolist --all
```

### Créer un repository local depuis le DVD RHEL

#### Monter l'ISO RHEL
l'ISO fait 10 Go, il est préférable de le monter sur une machine virtuelle avec suffisamment d'espace disque.

```bash
# monter l'ISO RHEL
mount /dev/cdrom /mnt
# vérifiér le contenu
ls /mnt
```
on doit voir plusieurs fichiers et dossiers dont : 
```plaintext
AppStream BaseOS boot EFI  ...
```
C'est AppStream et BaseOS qui nous intéressent pour créer des dépôts locaux.

#### Installer le plugin nécessaire
```bash
dnf install dnf-plugins-core
```
Le paquet enrichit DNF avec des fonctionnalités avancées qui ne sont pas incluses par défaut. Il fournit des outils supplémentaires pour la gestion des dépôts, la manipulation des paquets, et d'autres opérations liées à la gestion des logiciels sur les systèmes basés sur RPM.

#### Ajouter les repositories locaux
```bash
dnf config-manager --add-repo file:///mnt/BaseOS
dnf config-manager --add-repo file:///mnt/AppStream
```
⚠️ Les chemins sont sensible à la casse :
BaseOS ≠ baseos.

Ces nouvelles entrées repo apparaîtront dans /etc/yum.repos.d/ sous un nom du style :
`mnt-BaseOS.repo` et `mnt-AppStream.repo`.

Désactiver le check de la signature GPG (le DVD n'inclut pas les clés GPG) :
```bash
dnf config-manager --setopt=mnt-BaseOS.gpgcheck=0 --save
dnf config-manager --setopt=mnt-AppStream.gpgcheck=0 --save
```

Attention si vous aviez déjà les dépôts officiels RHEL activés, dnf utilisera ceux-ci en priorité. Et vous ne pouvez pas les désactiver avec `dnf config-manager --disable ...` car ils sont gérés par le Subscription Manager. 

Désactiver les dépôts officiels RHEL :
```bash
subscription-manager repos --list-enabled 
subscription-manager repos --disable=rhel-10-for-x86_64-appstream-rpms
subscription-manager repos --disable=rhel-10-for-x86_64-baseos-rpms
```

#### Tester le repository local
1. Désactiver le BaseOS officiel :
```bash
dnf config-manager --disable rhel-10-for-x86_64-baseos-rpms
``` 
2. Essayez d'installer le paquet `zsh` :
```bash
dnf install zsh
```
Si cela fonctionne → zsh vient bien du repo local dans /mnt.

Réactiver le BaseOS officiel :
```bash
dnf config-manager --enable rhel-10-for-x86_64-baseos-rpms
```
On peut vérifier la provenance du paquet installé (From repo):
```bash
dnf info zsh
```

Réactover les dépôts distants officiels RHEL :
```bash
subscription-manager repos --list-disabled
subscription-manager repos --enable=rhel-10-for-x86_64-appstream-rpms
subscription-manager repos --enable=rhel-10-for-x86_64-baseos-rpms
```

### Limites du repository local DVD
- Pas de mises à jour.
- Seulement les versions présentes sur le DVD.
- Dépendent du montage de /mnt.

### Créer un repository servi via Apache (HTTP) :
Objectif : rendre AppStream et BaseOS disponibles via HTTP, localement ou sur le réseau.

#### 1. Supprimer les repos précédents (ceux créés avec mnt_) :
```bash
rm -f /etc/yum.repos.d/mnt_*.repo
```

#### 2. Installer Apache
```bash
dnf install httpd
systemctl enable --now httpd
```

#### 3. Monter le DVD dans la racine web

Apache sert son contenu depuis /var/www/html.

Monter l’ISO dedans :
```bash
mount /dev/cdrom /var/www/html
```

On obtient donc plusieurs fichiers et dossiers dont :
```plaintext
/var/www/html/AppStream/
/var/www/html/BaseOS/
```
#### 4. Ajouter les repositories HTTP

BaseOS :
```bash
dnf config-manager --add-repo http://localhost/BaseOS
```
AppStream :
```bash
dnf config-manager --add-repo http://localhost/AppStream
```

On pourrait remplacer localhost par :

- une IP locale : http://192.168.1.50/BaseOS
- une IP publique : http://x.x.x.x/AppStream

Utile pour éviter que 20 machines téléchargent via Internet.

#### 5. Vérifier le bon fonctionnement 
```bash
dnf makecache
dnf repolist
dnf install zsh
```
Si les repos ne sont pas lisibles, makecache renverra une erreur.
S'il fonctionne, les dépôts HTTP sont opérationnels.

### Intérêt global

Créer un repository depuis le DVD permet :

- d’utiliser RHEL 8 sans souscription (mais sans mises à jour),
- d’avoir un dépôt local pour économiser la bande passante,
- de servir un dépôt sur plusieurs machines via HTTP,
- de travailler hors ligne dans un environnement de labo.

#### Commandes utiles pour la gestion des repositories
```bash
ls /etc/yum.repos.d/
cat /etc/yum.repos.d/redhat.repo
dnf repolist
dnf repolist --all
subscription-manager status
# lister les repositories disponibles
subscription-manager repos --list
```

### Gestion avancée des mises à jour & métadonnées :
RHEL 10 permet une gestion avancée des mises à jour grâce aux métadonnées Red Hat.
Ces métadonnées incluent :

- des tags de sécurité ;
- les niveaux de sévérité (Critical, Important, Moderate, etc.) ;
- les RHSA (Red Hat Security Advisories).

#### Lire les informations d’updates (métadonnées)

Pour afficher toutes les catégories d'updates disponibles :
```bash
dnf updateinfo list
```
On obtient par exemple :

- 16 security updates
- 33 bug fixes
- 3 enhancements

#### Filtrer uniquement les mises à jour de sécurité
```bash
dnf updateinfo list security
```
Cela affiche uniquement les security advisories (ex : les 16 notifications de sécurité).

#### Filtrer selon un niveau de sévérité
```bash
dnf updateinfo list --sec-severity=Critical
dnf updateinfo list --sec-severity=Important,Moderate,Low
```

#### Obtenir le détail d’une notification de mise à jour :
On peut afficher le contenu exact d’une mise à jour, donc pour avoir plus d'infos sur une notification spécifique :

```bash
dnf updateinfo info RHSA-2025:1234
```
Le détail inclut :

- les paquets concernés
- les CVE
- le numéro RHSA, par ex. : `RHSA-2025:1234`

Une mise à jour accompagnée de son ID.


#### Mettre à jour uniquement un advisory spécifique
Pour appliquer uniquement une mise à jour de sécurité spécifique :
```bash
dnf update --advisory=RHSA-2025:1234
```
Cela met uniquement à jour les paquets liés à cet avis, rien d’autre.

#### Mettre à jour uniquement les mises à jour de sécurité Important, et Critical
Exemple : appliquer automatiquement toutes les mises à jour critiques/urgentes :
```bash
dnf update --sec-severity=Critical,Important
```

#### Info supplémentaire et importante
Les mises à jour de sécurité sont cruciales, mais comme dans la nuit des temps, certaines mises à jour peuvent introduire des bugs ou des incompatibilités, il est important de savoir machine arrière.

dnf garde un historique des transactions, permettant de revenir en arrière si nécessaire.


Voici quelques commandes pour du `en cas où` :
```bash
dnf history list
dnf history info <ID_transaction>
dnf history undo <ID_transaction>
```

Le rollback (undo) permet de :
- downgrade les paquets mis à jour 
- supprime les paquets installés pendant la mise à jour 
C’est pratique si une mise à jour casse une dépendance ou provoque un comportement inattendu.






### Gestion des paquets avec dnf

***Attention a la date et l'heure de votre système, si vous avez une heure décalée, vous risquez d'avoir des erreurs lors de l'installation des paquets via dnf. (problèmes de certificats)***

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
#check si un paquet est installé (si installé il sera listé comme installed, sinon il sera lister comme available)
dnf list <nom_du_paquet>
# afficher quel paquet RPM fournit un fichier binaire
dnf provides <*/bin/tree>
```




---

## Labs - 
### Lab 001- Gestion des paquets avec dnf et les modules.

- Trouvez un paquet contenant une ` Virtualization API`.
```bash
dnf search Virtualization API
```
- Trouvez un paquet fournissant un "`web server` et `reverse proxy`".
```bash
dnf search web server reverse proxy
```
- Trouvez un outil combinant "`traceroute` et `ping`".
```bash
dnf search traceroute ping
```

- Trouvez le paquet fournissant `nmap` :
    - déterminer s’il est installé ou non
    - afficher des infos supplémentaires
    - lister les fichiers fournis
```bash
dnf list nmap
dnf info nmap
dnf repoquery --available -l nmap
```


- Avec la commande `dnf provides` trouvez le paquet fournissant l'outil `netstat` :
    - Affichez les informations du paquet, et lister les fichiers fournit par le paquet.

```bash
sudo dnf list net-tools
sudo dnf info net-tools
sudo dnf repoquery --installed -l net-tools
```

- Utilisez `dnf provides` pour trouver le paquet qui fournit le fichier `smb.conf`, (uniquement les paquets installés).

```bash
sudo dnf provides "*/smb.conf"
sudo dnf list samba-common
```

- Utilisez `dnf repoquery` pour afficher les informations supplémentaire sur le paquet trouvé précédement.
```bash
sudo dnf repoquery --installed -i samba-common
```


- Utilisez `dnf repoquery` pour Identifier le paquet propriétaire de `/etc/chrony.conf` (uniquement les paquets installés).
```bash
sudo dnf repoquery --installed -f "/etc/chrony.conf"
sudo dnf repoquery --installed -f "/etc/chrony.conf" -i
```

### Lab 002 - Gestion des paquets avec dnf et les modules.
