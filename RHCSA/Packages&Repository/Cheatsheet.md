---
title: Cheatsheet
parent: Packages & Repositories
grand_parent: RHCSA
nav_order: 42
has_children: false
nav_exclude: false
---

# RHCSA 10 CHEATSHEET: SOFTWARE MANAGEMENT & REPOSITORIES

***Moteur : DNF5 (C++)  Base : RPM  Contexte : RHEL 10***


#### Anatomie d'un fichier .repo

Fichier : /etc/yum.repos.d/<nom>.repo

```ini
[baseos-rhel10]                 # ID Unique (Pas d'espaces !)
name=RHEL 10 BaseOS             # Nom descriptif
baseurl=https://content.example.com/rhel10/BaseOS  # HTTP, HTTPS, FTP ou FILE
enabled=1                       # 1=ON, 0=OFF
gpgcheck=1                      # Vérifier la signature GPG
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
skip_if_unavailable=1           # (Avancé) Ne pas bloquer dnf si ce repo est down
priority=1                      # (Avancé) 1 est la plus haute priorité
```

**Gestion des Dépôts (CLI)**

Sous RHEL 10, config-manager est intégré, donc il n y a pas besoin d'installer un package supplémentaire.

- Ajouter un repo (URL) `dnf config-manager --add-repo <url>`
- Activer un repo `dnf config-manager --enable <id>`
- Désactiver un repo `dnf config-manager --disable <id>`
- Lister les actifs `dnf repolist`
- Lister TOUT `dnf repolist all`
- Voir infos étendues `dnf repolist -v`


#### Commandes basiques DNF

Note : yum est un alias vers dnf, qui est un alias vers dnf5.

DNF (Dandified YUM) est le gestionnaire de paquets moderne pour RHEL 8/9/10. Utilisez ces commandes pour les opérations quotidiennes: installer, mettre à jour, réinstaller, supprimer et downgrader des paquets.


- `dnf install <paquet>` Pour installer un paquet. Ajouter `-y` pour éviter la confirmation. 
- `dnf reinstall <paquet>` Réinstaller un paquet - restaure les binaires/configs modifiés.
- `dnf update <paquet>` Mettre à jour un paquet. Ne touche pas au reste du système.
- `dnf upgrade` Mettre à jour tout l'OS. Alias de `update` ; met l'OS à jour.
- `dnf remove <paquet>` Supprimer un paquet. Supprime aussi les dépendances inutilisées.
- `dnf downgrade <paquet>`  Downgrader un paquet - revient à la version précédente.
- `dnf update --bugfix` - Mettre à jour uniquement les paquets avec des corrections de bugs (pas de nouvelles fonctionnalités).
- `dnf update --security` - Seulement les paquets avec des mises à jour de sécurité.
- `dnf check-update`   Vérifier les mises à jour disponibles sans les installer.



**Fonctions Avancées (Le "Plus Complet")**

Ces options avancées sont utiles pour des scénarios spécifiques (installation hors-ligne, mise à jour partielle, récupération et téléchargement de paquets).

- `dnf install -C <paquet>`   Installer depuis le cache local, pas de téléchargement.  
- `dnf download <paquet>`   Télécharger un paquet sans l'installer.  
- `dnf install ./mon-logiciel.rpm`   Installer un paquet localement depuis un fichier RPM.  
- `dnf update --exclude=kernel*`   Mettre à jour tout sauf les paquets correspondant au motif.  


#### Investigation & Recherche

Utilisez ces commandes pour rechercher des paquets, lister les paquets disponibles/ installés, et trouver quel paquet contient un fichier particulier.
 
- `dnf search all "web server"`   Rechercher tous les paquets correspondant à "web server".  
- `dnf list available`   Lister tous les paquets disponibles.  
- `dnf list installed`   Lister tous les paquets installés.  
- `dnf info <paquet>`   Voir les détails d'un paquet spécifique.
- `dnf list <paquet>` Lister un paquet spécifique.


**LA COMMANDE : provides**

Cas : "Installez l'outil qui génère le fichier /etc/mime.types"

- Commande : dnf provides /etc/mime.types
    - Réponse : mailcap-2.1.48...
        - Action : dnf install mailcap

Astuce RHEL 10 : Tu peux utiliser des jokers. dnf provides */semanage (Trouve le binaire peu importe où il est rangé).

- `dnf provides /chemin/fichier` : Quel paquet fournit ce fichier ?
- `dnf provides '*/commande'` (trouve le paquet qui fournit cette commande, peu importe où elle est rangée).

#### Groupes & Environnements

Les groupes d'installation permettent de déployer un ensemble de paquets pour un environnement spécifique (ex. server avec GUI, outils dev). On peut inclure les paquets optionnels si besoin.

- `dnf group list -v`   Lister les groupes (IDs inclus), (Le `-v` est crucial pour voir les IDs techniques comme `development`).  
- `dnf group install|remove "Server with GUI"`   Installer un groupe de paquets, ici "Server with GUI".  
- `dnf group install --with-optional "Development Tools"`   Installer les paquets obligatoires ET optionnels d'un groupe.  
- `dnf group info "Development Tools"`   Voir le contenu d'un groupe.  
- `dnf mark install package` # Marquer un paquet comme installé manuellement (empêche la suppression automatique).

#### Historique & Rollback

DNF conserve un historique des transactions qui permet d'annuler ou de revenir en arrière en cas de problème (utile pour les environnements de production et les tests de mise à jour).

- `dnf history`   Voir l'historique des transactions DNF.  
- `dnf history info <ID>`   Voir les détails d'une transaction spécifique.  
- `dnf history undo <ID>`   Annuler une transaction spécifique.  
- `dnf history redo <ID>`   Annuler une transaction spécifique.  
- `dnf history rollback <ID>`   (Annule TOUT ce qui s'est passé après la transaction ex. ID="5").
- `dnf history rollback 10..15`   Annuler transactions 10 à 15. 


#### RPM (Bas Niveau & Intégrité) à corriger

RPM est l'outil bas-niveau pour manipuler les fichiers `.rpm` : inspection, installation à bas niveau et vérification de l'intégrité.
- `rpm -ivh <paquet.rpm>`           Installer un paquet RPM en mode bas niveau.
- `rpm -Uvh <paquet.rpm>`           Mettre à jour un paquet existant ou l’installer s’il n’est pas présent.
- `rpm -e <paquet>`                 Désinstaller un paquet installé. 
- `rpm -q <paquet>`                 Vérifier si un paquet est installé et afficher sa version. 
- `rpm -qa   grep <motif>`          Lister tous les paquets installés, filtrés par motif.
- `rpm -V <paquet>`                 Vérifier l’intégrité d’un paquet installé (fichiers modifiés, permissions, taille…).  
- `rpm -qi <paquet>`                Afficher les informations détaillées du paquet (description, version, vendor…).  
- `rpm -ql <paquet>`                Lister tous les fichiers installés par ce paquet.  
- `rpm -qc <paquet>`                Lister uniquement les fichiers de configuration installés par ce paquet.  
- `rpm -qd <paquet>`                Lister les fichiers de documentation associés au paquet.  
- `rpm -qf <fichier>`               Identifier à quel paquet appartient un fichier donné. (ex. rpm -qf /bin/ls) ?  
- `rpm -q --scripts <pkg>`          Afficher les scripts associés au cycle de vie du paquet (pre-install, post-install, pre-uninstall, post-uninstall).  

**Requêtes sur fichiers .rpm NON INSTALLÉS**

Ajoutez le flag `-p` (package file) pour interroger un fichier RPM sans l'installer — utile pour inspecter son contenu.

- `rpm -qip fichier.rpm` Info sur le paquet
- `rpm -qlp fichier.rpm` Quels fichiers ça va créer ?
- `rpm -qcp fichier.rpm` Quels configs ça contient ?

**Vérification d'intégrité :**

Un fichier de config a été modifié ou un binaire corrompu ?

- Vérifier un paquet : `rpm -V httpd`
    - Rien ne s'affiche ? Tout est parfait.
    - Sortie S.5....T. ? Le fichier a changé (S=Taille, 5=MD5, T=Date).
- Importer une clé GPG manuellement : `rpm --import /etc/pki/rpm-gpg/...`
- Vérifier la signature d'un RPM : `rpm -K paquet.rpm`

#### Verrouillage de Version (Plugin Versionlock)

Le plugin `versionlock` permet d'empêcher la mise à jour d'un paquet critique en verrouillant sa version, utile pour garantir une compatibilité applicative.

Empêcher la mise à jour d'un paquet critique. Sous RHEL 10, c'est souvent un plugin dnf.

- `dnf install dnf-plugin-versionlock -y` : Installer le plugin versionlock.
- `dnf versionlock add httpd` : Geler la version actuellement installée du paquet httpd. (DNF empêchera désormais toute mise à jour de ce paquet.)
- `dnf versionlock list` : Afficher la liste des paquets dont la version est verrouillée.
- `dnf versionlock delete httpd` : Retirer le gel appliqué au paquet httpd.
- `dnf versionlock clear` : Supprimer tous les verrous de version existants.

*Ou via configuration*
```bash
echo "exclude=httpd*" >> /etc/dnf/dnf.conf
```

#### Checklist Dépannage (Troubleshooting)

En cas d'erreur liée à la gestion des paquets ou aux dépôts, commencez par ces vérifications et résolutions rapides.

**"GPG Check FAILED"**

- Fix rapide Exam : Édite le `.repo` et mets `gpgcheck=0`.
- Fix Propre : Trouve l'URL de la clé et fais `rpm --import <url>`.

**"404 Not Found" sur un repo**

- Vérifie si tu as mis http au lieu de https (ou l'inverse).
- Vérifie s'il y a une majuscule dans l'URL (Linux est sensible à la casse : BaseOS != baseos).

**La base RPM est corrompue**

- Commande de la dernière chance : `rpm --rebuilddb`
- DNF est lent ou buggé
- Le réflexe absolu : `dnf clean all` suivi de `dnf makecache`.

**Nettoyage et cache**
- `dnf clean all` : Tout nettoyer (cache, métadonnées)
- `dnf makecache` : Recréer le cache


| Fichier                  | Usage                                     |
| ------------------------ | ----------------------------------------- |
| `/var/log/dnf.log`       | Historique des opérations DNF             |
| `/var/log/dnf.rpm.log`   | Opérations RPM via DNF                    |
| `/var/cache/dnf/`        | Cache des paquets                         |
| `/usr/lib/sysimage/rpm/` | Base de données RPM                       |
| `/etc/dnf/protected.d/`  | Paquets protégés (impossible à supprimer) |



Pro Tips :

    En examen, toujours -y pour éviter les prompts
    Jamais rpm -i pour installer depuis un repo (perd les dépendances)
    Toujours reset un module avant de changer de stream
    Clean all si comportement étrange (cache corrompu)
    Simulation avec --assumeno avant action critique
    Versionlock pour préserver configuration de l'examen
    Variables $releasever et $basearch pour portabilité
    GPG : toujours vérifier les paquets signés (sauf repo local temporaire)
    Protection : vérifier /etc/dnf/protected.d/ avant suppression
    Autocomplétion : dnf install <TAB><TAB> pour explorer