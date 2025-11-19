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


#### Architecture & Dépôts (Repositories)
