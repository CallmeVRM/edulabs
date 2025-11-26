---
title: Lab-awk
parent: Scénarios
grand_parent: Linux
nav_order: 2
---

####  Introduction

AWK est un outil puissant conçu pour la manipulation et l'analyse de données textuelles, particulièrement utile pour les administrateurs système Linux. Il excelle dans le traitement de fichiers structurés en lignes et colonnes, tels que les fichiers de configuration, les journaux système, et les sorties de commandes.

#### Objectif de ce document
Voici une liste de 30 questions/scénarios, classés par difficulté et logique métier, qui reflètent ce qu'un administrateur système pourrait devoir accomplir lors de l'examen ou dans la vie réelle.


###### Mise en place de l'environnement


```bash
#!/bin/bash

# Définition du dossier de travail
LAB_DIR=~/lab_awk
mkdir -p "$LAB_DIR"
cd "$LAB_DIR" || exit

echo "Préparation de l'environnement dans $LAB_DIR..."

# 1. Simulation d'un fichier /etc/passwd (délimiteur :)
cat << 'EOF' > passwd_lab
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
student:x:1000:1000:Student User:/home/student:/bin/bash
ansible:x:1001:1001:Ansible Automation:/home/ansible:/bin/zsh
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/false
developer:x:1002:1002:Dev Account:/home/developer:/bin/bash
EOF

# 2. Simulation d'un fichier de logs (délimiteur espace par défaut)
cat << 'EOF' > server.log
Nov 24 10:00:01 server1 systemd[1]: Started Session 1 of user root.
Nov 24 10:05:23 server1 sshd[1234]: Failed password for invalid user admin from 192.168.1.50
Nov 24 10:06:10 server1 sshd[1234]: Accepted password for student from 192.168.1.10 port 22 ssh2
Nov 24 10:15:00 server1 chronyd[567]: Selected source 1.rhel.pool.ntp.org
Nov 24 10:20:45 server1 app_custom[888]: Error: Database connection timeout
Nov 24 10:22:00 server1 app_custom[888]: Success: Connection restored
Nov 24 10:30:00 server1 systemd[1]: Stopping User Manager for UID 1000...
EOF

# 3. Simulation d'un inventaire CSV (délimiteur virgule)
cat << 'EOF' > inventory.csv
Hostname,IP,Role,RAM_GB,Disk_Usage_Percent
web01,192.168.1.10,Web,4,45
db01,192.168.1.20,Database,16,82
cache01,192.168.1.30,Cache,8,12
web02,192.168.1.11,Web,4,50
backup01,192.168.1.100,Backup,8,95
EOF

# 4. Simulation d'une sortie de commande 'ip addr' (complexe)
cat << 'EOF' > ip_output.txt
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:aa:bb:cc brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.50/24 brd 192.168.122.255 scope global dynamic noprefixroute eth0
       valid_lft 3500sec preferred_lft 3500sec
3: enp6s0f0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 00:19:99:db:58:87 brd ff:ff:ff:ff:ff:ff
4: enp6s0f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 00:19:99:db:58:86 brd ff:ff:ff:ff:ff:ff
5: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 92:8d:d1:3d:42:bf brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.10/24 metric 100 brd 192.168.1.255 scope global dynamic br0
       valid_lft 82919sec preferred_lft 82919sec
    inet6 2a01:cb04:db7:6900:908d:d1ff:fe3d:42bf/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 86389sec preferred_lft 589sec
    inet6 fe80::908d:d1ff:fe3d:42bf/64 scope link 
       valid_lft forever preferred_lft forever
6: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1e:c2:2d:c1:58:29 brd ff:ff:ff:ff:ff:ff
7: br-int: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 56:1c:f8:0b:8f:d5 brd ff:ff:ff:ff:ff:ff
8: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:dd:c7:ee brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
EOF

echo "Environnement prêt ! Fichiers créés :"
ls -1 "$LAB_DIR"
```


##### Les Exercices :

**I. Extraction de base (Fichier cible : server.log)**

**Objectif** : Manipuler les champs séparés par des espaces.

1. Affichez uniquement la 1ère colonne (le mois) du fichier server.log.
2. Affichez la date (colonne 2) et l'heure (colonne 3) séparées par une tabulation.
3. Affichez l'heure (colonne 3) avant la date (colonne 2 + colonne 1) séparées par des tirets (-).
4. Affichez l'intégralité du fichier (équivalent à cat).
5. Affichez la chaîne "Log de :" avant le nom du serveur (colonne 4).
6. Affichez le dernier mot de chaque ligne de log (la dernière colonne), peu importe la longueur de la ligne.
7. Affichez le nombre total de lignes ainsi que le nombre total de chaque colonne dans la ligne correspondante (field). 


**II. Gestion des Délimiteurs (Fichier cible : `passwd_lab` et `inventory.csv`)**

**Objectif** : Manipuler l'option -F.

8. Au niveau du fichier `passwd_lab`, affichez le tout premier champ (nom d'utilisateur).
9. Dans `passwd_lab`, affichez le nom d'utilisateur (colonne 1) et son Shell (colonne 7), séparé par une double tabulation.
10. Dans `inventory.csv`, affichez uniquement les adresses IP (colonne 2).
11. Dans `inventory.csv`, affichez le nom d'hôte (col 1) et le rôle (col 3) dans le format suivant : `Le host web01 à pour rôle Web`
12. Dans `inventory.csv`, modifiez le séparateur de sortie (OFS) pour que les colonnes extraites soient séparées par des tabulations au lieu d'espaces.
13. Dans `inventory.csv`, étant donné que tout les ip's font partie du même sous-réseau, affichez uniquement le dernier octet de chaque adresse IP.

**III. Filtrage par Motifs (Fichier cible : `server.log`)**

**Objectif** : Sélectionner les lignes pertinentes.

14. Affichez uniquement les lignes de server.log qui contiennent le mot "Error".
15. Affichez toutes les lignes qui ne contiennent pas "Error".
16. Affichez les lignes qui commencent par "Nov".
17. Affichez uniquement la ligne numéro 5 du fichier server.log.
18. Affichez les lignes allant de la ligne 2 à la ligne 4 incluse avec leur numéro de ligne.
19. Affichez uniquement l'heure (colonne 3) pour les lignes contenant "sshd".

**IV. Conditions sur les Valeurs (Fichier cible : `passwd_lab` et `inventory.csv`)**

**Objectif** : Comparer des nombres et des chaînes exactes.

20. Dans passwd_lab, affiche les lignes complètes où l'UID (colonne 3) est supérieur ou égal à 1000.
21. Dans passwd_lab, affiche uniquement les noms d'utilisateurs (col 1) qui ont un UID (col 3) < 10.
22. Dans inventory.csv, affiche les lignes où le Rôle (col 3) est exactement égal à "Web".
23. Dans inventory.csv, affiche les serveurs qui ont plus de 80% d'utilisation disque (col 5 > 80).
24. (Avancé) Dans inventory.csv, affiche les serveurs qui sont de type "Web" ET qui ont une utilisation disque < 60%.

**V. Scénarios "Sysadmin" (Fichiers mixtes)**

**Objectif** : Parsing complexe et nettoyage.

25. Utilise le fichier ip_output.txt. Isole et affiche uniquement l'adresse IP 192.168.122.50 (sans le masque /24).
# (25 Bis) Rajouter plus de lignes (6 interfaces) et filtre pour n'afficher que les IP de celles qui sont UP.
26. Dans passwd_lab, affiche uniquement le nom des utilisateurs dont le shell est /bin/bash.
27. Dans passwd_lab, compte combien il y a de lignes au total (en utilisant AWK, pas wc).
28. Dans server.log, affiche le numéro de ligne devant chaque message d'erreur contenant "Failed".
29. Dans inventory.csv, n'affiche que les noms d'hôtes (col 1) en sautant la première ligne (l'en-tête).
30. Dans server.log, extrais uniquement les processus (ex: sshd[1234]:) sans le reste de la ligne (colonne 5).
30. (Variante) Dans server.log, extrais uniquement les processus (ex: sshd[1234]:) sans le reste de la ligne (colonne 5) Uniquement le PID.
31. Modifie l'affichage de passwd_lab pour qu'il dise : "L'utilisateur [NOM] utilise le shell [SHELL]" pour chaque ligne. L'utilisateur avec `'` obligatoirement.
32. (Bonus nettoyage) Affiche le contenu de inventory.csv, mais remplace toutes les virgules par des tirets - à l'affichage.




### Solutions 


###### 1. Affichez uniquement la 1ère colonne (le mois) du fichier server.log.

```bash
awk '{print $1}' server.log
```

###### 2. Affichez la date (colonne 2) et l'heure (colonne 3) séparées par une tabulation.

```bash
awk '{print $2, $3}' server.log
```


###### 3. Affichez l'heure (colonne 3) avant la date (colonne 2 + colonne 1) séparées par des tirets (-).

```bash
awk '{print $3 "-" $2 "-" $1}' server.log
```

###### 4. Affichez l'intégralité du fichier (équivalent à cat).

```bash

```
###### 5. Affichez la chaîne "Log de :" avant le nom du serveur (colonne 4).

```bash


```

###### 6. Affichez le dernier mot de chaque ligne de log (la dernière colonne), peu importe la longueur de la ligne.

```bash
awk '{print NR, NF}' server.log
```

###### 7. Affichez le nombre total de lignes ainsi que le nombre total de chaque colonne dans la ligne correspondante (field). 

```bash


```


###### 8. Au niveau du fichier `passwd_lab`, affichez le tout premier champ (nom d'utilisateur).

```bash


```

###### 9. Dans `passwd_lab`, affichez le nom d'utilisateur (colonne 1) et son Shell (colonne 7), 
séparé par une double tabulation.

```bash


```

###### 10. Dans `inventory.csv`, affichez uniquement les adresses IP (colonne 2).

```bash


```

###### 11. Dans `inventory.csv`, affichez le nom d'hôte (col 1) et le rôle (col 3) dans le format suivant : `Le host web01 à pour rôle Web`

```bash


```

###### 12. Dans `inventory.csv`, modifiez le séparateur de sortie (OFS) pour que les colonnes extraites soient séparées par des tabulations au lieu d'espaces.

```bash


```

###### 13. Dans `inventory.csv`, étant donné que tout les ip's font partie du même sous-réseau, affichez uniquement le dernier octet de chaque adresse IP.

```bash
awk -F, '{print $2}'  inventory.csv | awk -F. '{print $4}'
```

###### 14. Affichez uniquement les lignes de server.log qui contiennent le mot "Error".

```bash


```

###### 15. Affichez toutes les lignes qui ne contiennent pas "Error".

```bash


```

###### 16. Affichez les lignes qui commencent par "Nov".

```bash


```

###### 17. Affichez uniquement la ligne numéro 5 du fichier server.log.

```bash


```

###### 18. Affichez les lignes allant de la ligne 2 à la ligne 4 incluse avec leur numéro de ligne.

```bash
awk 'NR>=4 && NR<=6 {print NR, $0}' server.log
```

###### 19. Affichez uniquement l'heure (colonne 3) pour les lignes contenant "sshd".

```bash


```

###### 20. Dans passwd_lab, affiche les lignes complètes où l'UID (colonne 3) est supérieur ou égal à 1000.

```bash


```

###### 21. Dans passwd_lab, affiche uniquement les noms d'utilisateurs (col 1) qui ont un UID (col 3) < 10.

```bash


```

###### 22. Dans inventory.csv, affiche les lignes où le Rôle (col 3) est exactement égal à "Web".

```bash


```

###### 23. Dans inventory.csv, affiche les serveurs qui ont plus de 80% d'utilisation disque (col 5 > 80).

```bash


```

###### 24. (Avancé) Dans inventory.csv, affiche les serveurs qui sont de type "Web" ET qui ont une utilisation disque < 60%.

```bash


```


###### 25. Utilise le fichier ip_output.txt. Isole et affiche uniquement l'adresse IP 192.168.122.50 (sans le masque /24).

```bash


```

# (25 Bis) Rajouter plus de lignes (6 interfaces) et filtre pour n'afficher que les IP de celles qui sont UP.

```bash


```

###### 26. Dans passwd_lab, affiche uniquement le nom des utilisateurs dont le shell est /bin/bash.

```bash
awk -F: '$NF == "/bin/bash" {print $1}' passwd_lab
ou
awk -F: '$7 == "/bin/bash" {print $1}' passwd_lab
```

###### 27. Dans passwd_lab, compte combien il y a de lignes au total (en utilisant AWK, pas wc).

```bash


```

###### 28. Dans server.log, affiche le numéro de ligne devant chaque message d'erreur contenant "Failed".

```bash


```

###### 29. Dans inventory.csv, n'affiche que les noms d'hôtes (col 1) en sautant la première ligne (l'en-tête).

```bash


```

###### 30. Dans server.log, extrais uniquement les processus (ex: sshd[1234]:) sans le reste de la ligne (colonne 5).

```bash


```
###### 30. (Variante) Dans server.log, extrais uniquement les processus (ex: sshd[1234]:) sans le reste de la ligne (colonne 5) Uniquement le PID.
```bash
awk '{gsub(/[^0-9]/,"",$5); print $5}' server.log
```

###### 31. Modifie l'affichage de passwd_lab pour qu'il dise : "L'utilisateur [NOM] utilise le shell [SHELL]" pour chaque ligne. L'utilisateur avec `'` obligatoirement.

```bash
awk -F: '{print "L'\''utilisateur " $1 " utilise le shell " $7}' passwd_lab
```

###### 32. (Bonus nettoyage) Affiche le contenu de inventory.csv, mais remplace toutes les virgules par des tirets - à l'affichage.

```bash
awk -F: '{print "L'\''utilisateur " $1 " utilise le shell " $7}' passwd_lab
```
- `-v OFS="-"` : On définit le séparateur de sortie (Output Field Separator) comme étant un tiret.

- `$1=$1` : C'est une astuce nécessaire dans AWK. Elle force AWK à "reconstruire" la ligne en utilisant le nouveau OFS. jf








31. Modifie l'affichage de passwd_lab pour qu'il dise : "L'utilisateur [NOM] utilise le shell [SHELL]" pour chaque ligne. L'utilisateur avec `'` obligatoirement.



32. (Bonus nettoyage) Affiche le contenu de inventory.csv, mais remplace toutes les virgules par des tirets - à l'affichage.













créer une variante de la question 6 Error avec case insensitive donc rajouter une ligne dans le script ...
Ajouter deux ligne pour d'autres mois pour la question 16

dans un fichier créer une question piège elle des lignes avec certaines 5 d'autres 10 d'autres 7 ... colonnes, le dernier fields représente quelquechose (par exemple le shell /bin/bash), et on doit filtrer en se basant sur le dernier champ, en posant la question




















##### I. Extraction de base (Champs et Colonnes)

Ces questions testent vos capacités à isoler une information précise.

    Comment afficher uniquement la première colonne d'un fichier texte ?

    Comment afficher simultanément la première et la troisième colonne d'un fichier, séparées par un espace ?

    Comment inverser l'ordre d'affichage de deux colonnes (afficher la colonne 2 avant la colonne 1) ?

    Quelle est la méthode pour afficher tout le contenu d'une ligne (l'enregistrement complet) sans filtrage ?

    Comment insérer une chaîne de caractères personnalisée (ex: "Le résultat est :") avant l'affichage d'une colonne ?

    Comment extraire uniquement le dernier champ d'une ligne, quel que soit le nombre de colonnes ?

II. Gestion des Délimiteurs (L'option -F)

Indispensable pour traiter des fichiers comme /etc/passwd ou des CSV.

    Comment traiter un fichier dont le séparateur n'est pas l'espace, mais le caractère "deux-points" (:) ?

    Comment extraire le nom d'utilisateur (1er champ) du fichier /etc/passwd ?

    Comment extraire le shell par défaut (dernier champ) pour chaque utilisateur dans /etc/passwd ?

    Comment définir plusieurs séparateurs possibles (par exemple, espace OU virgule) pour une même ligne ?

    Comment modifier le séparateur de sortie (OFS) pour que les colonnes extraites soient séparées par des tabulations au lieu d'espaces ?

III. Filtrage par Motifs et Lignes (Pattern Matching)

AWK excelle pour n'afficher que les lignes qui nous intéressent.

    Comment afficher uniquement les lignes d'un fichier qui contiennent le mot "Error" ?

    Comment afficher uniquement les lignes qui ne contiennent pas le mot "Success" ?

    Comment afficher uniquement les lignes qui commencent par le mot "root" ?

    Comment afficher uniquement la 5ème ligne d'un fichier ?

    Comment afficher une plage de lignes (par exemple, de la ligne 10 à la ligne 20) ?

    Comment n'afficher la première colonne que si la ligne contient le motif "Failed" ?

IV. Conditions sur les Valeurs (Comparaisons)

Logique très utile pour les scripts de monitoring.

    Comment afficher les lignes dont la valeur de la troisième colonne est supérieure à 500 ?

    Comment lister les utilisateurs du fichier /etc/passwd qui ont un UID (3ème champ) supérieur ou égal à 1000 ?

    Comment afficher les lignes où la deuxième colonne est strictement égale à la chaîne "active" ?

    Comment combiner deux conditions : la colonne 3 doit être > 100 et la colonne 4 doit contenir "OK" ?

V. Scénarios "Sysadmin RHEL" (Mise en situation réelle)

C'est ici que se joue l'examen : combiner des commandes système et AWK via des pipes (|).

    Depuis la commande ip addr, comment isoler uniquement l'adresse IP (sans le masque de sous-réseau) de l'interface eth0 ?

    Depuis la commande df -h, comment afficher uniquement le pourcentage d'utilisation de la partition montée sur / ?

    Depuis la sortie de ps aux, comment extraire uniquement les PID (Process ID) des processus appartenant à l'utilisateur apache ?

    Comment lister uniquement les noms des connexions actives (NAME) depuis la commande nmcli connection show ?

    Comment récupérer l'UUID d'une partition spécifique depuis le fichier /etc/fstab (en ignorant les lignes commentées) ?

    Comment compter le nombre total de lignes dans un fichier sans utiliser la commande wc -l, mais uniquement AWK ?

    Comment afficher le numéro de ligne devant chaque ligne extraite d'un fichier de configuration ?

    Depuis le fichier /var/log/messages ou journalctl, comment extraire uniquement l'horodatage et le nom du processus émetteur ?

    Comment supprimer les lignes vides d'un fichier texte lors de l'affichage ?