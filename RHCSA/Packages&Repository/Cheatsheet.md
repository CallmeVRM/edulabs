---
title: brainstorming
parent: Packages & Repositories
grand_parent: RHCSA
nav_order: 42
has_children: false
nav_exclude: false
---

# RHCSA 10 CHEATSHEET: SOFTWARE MANAGEMENT & REPOSITORIES

***Moteur : DNF5 (C++) | Base : RPM | Contexte : RHEL 10***


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


#### Commandes basique DNF

Note : yum est un alias vers dnf, qui est un alias vers dnf5.

| Commande | Description |
| --- | --- |
| `dnf install <paquet>` | Pour installer un paquet. Ajouter `-y` pour éviter la confirmation. |
| `dnf reinstall <paquet>` | Réinstaller un paquet — restaure les binaires/configs modifiés. |
| `dnf update <paquet>` | Mettre à jour un paquet. Ne touche pas au reste du système. |
| `dnf upgrade` | Mettre à jour tout l'OS. Alias de `update` ; met l'OS à jour. |
| `dnf remove <paquet>` | Supprimer un paquet. Supprime aussi les dépendances inutilisées. |
| `dnf downgrade <paquet>` | Downgrader un paquet — revient à la version précédente. |


**Fonctions Avancées (Le "Plus Complet")**

| Commande | Description |
| --- | --- |
| `dnf install -C <paquet>` | Installer depuis le cache local, pas de téléchargement. |
| `dnf download <paquet>` | Télécharger un paquet sans l'installer. |
| `dnf install ./mon-logiciel.rpm` | Installer un paquet localement depuis un fichier RPM. |
| `dnf update --exclude=kernel*` | Mettre à jour tout sauf les paquets correspondant au motif. |


#### Investigation & Recherche

| Commande | Description |
| --- | --- |
| `dnf search all "web server"` | Rechercher tous les paquets correspondant à "web server". |
| `dnf list available` | Lister tous les paquets disponibles. |
| `dnf list installed` | Lister tous les paquets installés. |

**LA COMMANDE : provides**

Cas : "Installez l'outil qui génère le fichier /etc/mime.types"

    - Commande : dnf provides /etc/mime.types

    - Réponse : mailcap-2.1.48...

    - Action : dnf install mailcap

Astuce RHEL 10 : Tu peux utiliser des jokers. dnf provides */semanage (Trouve le binaire peu importe où il est rangé).

#### Groupes & Environnements

| Commande | Description |
| --- | --- |
| `dnf group list -v` | Lister les groupes (IDs inclus), (Le `-v` est crucial pour voir les IDs techniques comme `development`). |
| `dnf group install "Server with GUI"` | Installer un groupe de paquets, ici "Server with GUI". |
| `dnf group install --with-optional "Development Tools"` | Installer les paquets obligatoires ET optionnels d'un groupe. |
| `dnf group info "Development Tools"` | Voir le contenu d'un groupe. |


#### Historique & Rollback
| Commande | Description |
| --- | --- |
| `dnf history` | Voir l'historique des transactions DNF. |
| `dnf history info <ID>` | Voir les détails d'une transaction spécifique. |
| `dnf history undo <ID>` | Annuler une transaction spécifique. |
| `dnf history rollback <ID>` | (Annule TOUT ce qui s'est passé après la transaction 5) |

#### RPM (Bas Niveau & Intégrité)
| Commande | Description |
| --- | --- |
| `rpm -ivh <paquet.rpm>` | Installer un paquet RPM (bas niveau). |
| `rpm -Uvh <paquet.rpm>` | Mettre à jour ou installer un paquet RPM (bas niveau). |
| `rpm -e <paquet>` | Supprimer un paquet RPM (bas niveau). |
| `rpm -q <paquet>` | Vérifier si un paquet est installé. |
| `rpm -qa | grep <motif>` | Lister tous les paquets installés correspondant au motif. |
| `rpm -V <paquet>` | Vérifier l'intégrité d'un paquet installé |
| `rpm -qi <paquet>` | Afficher la description et la version |
| `rpm -ql <paquet>` | Où sont les fichiers installés par ce paquet ? |
| `rpm -qc <paquet>` | Où sont les fichiers de configuration de ce paquet? |
| `rpm -qd <paquet>` | Où sont les fichiers de documentation de ce paquet ? |
| `rpm -qf <fichier>` | À quel paquet appartient ce fichier (ex. rpm -qf /bin/ls) ? |
| `rpm -q --scripts <pkg>` | Afficher les scripts d'installation/désinstallation d'un paquet |

**Requêtes sur fichiers .rpm NON INSTALLÉS**

Ajoute le flag -p (package file).

    rpm -qip fichier.rpm (Infos)

    rpm -qlp fichier.rpm (Quels fichiers ça va créer ?)

    rpm -qcp fichier.rpm (Quels configs ça contient ?)

Vérification d'intégrité :

Un fichier de config a été modifié ou un binaire corrompu ?

- Vérifier un paquet : `rpm -V httpd`
    - Rien ne s'affiche ? Tout est parfait.
    - Sortie S.5....T. ? Le fichier a changé (S=Taille, 5=MD5, T=Date).
- Importer une clé GPG manuellement : `rpm --import /etc/pki/rpm-gpg/...`
- Vérifier la signature d'un RPM : `rpm -K paquet.rpm`

#### Verrouillage de Version (Plugin Versionlock)

Verrouillage de Version (Plugin Versionlock)

Empêcher la mise à jour d'un paquet critique.

Sous RHEL 10, c'est souvent un plugin dnf5.

- Installer le plugin (si besoin) : `dnf install dnf-command(versionlock)`
- Verrouiller : `dnf versionlock add httpd` (Httpd ne se mettra plus jamais à jour).
- Lister les verrous : `dnf versionlock list`
- Supprimer le verrou : `dnf versionlock delete httpd`
- Tout nettoyer : `dnf versionlock clear`

#### Checklist Dépannage (Troubleshooting)

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


