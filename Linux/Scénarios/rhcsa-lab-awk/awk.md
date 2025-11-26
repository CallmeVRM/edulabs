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

cat << 'EOF' > tricky_one
root:x:0:0:root:/root:boobap:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:sbin/zbin:/sbin/nologin
#Commentaire1

adm:x:3:4:adm:/var/adm:whopab:veto:/sbin/nologin


student:x:1000:1000:Student User:/home/student:hacker:chilling:on:sofa:/bin/bash
#Commentaire2
ansible:x:1001:1001:Ansible Automation:/home/ansible:/bin/zsh

apache:x:48:48:Apache:/usr/share/httpd:rotab:rotaa:heaven:maina:/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/false


developer:x:1002:1002:Dev Account:/home/developer:/bin/bash
EOF

# 2. Simulation d'un fichier de logs (délimiteur espace par défaut)
cat << 'EOF' > server.log
Oct 23 09:02:13 server1 systemd[1]: Failed Session 1 of user root.
Nov 24 10:00:01 server1 systemd[1]: Started Session 1 of user root.
Nov 24 10:05:23 server1 sshd[7685]: Failed password for invalid user admin from 192.168.1.50
Nov 24 10:06:10 server1 sshd[13452]: Accepted password for student from 192.168.1.10 port 22 ssh2
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


#### Les questions :

###### 1. Affichez uniquement la 1ère colonne (le mois) du fichier server.log.
###### 2. Affichez la date (colonne 2) et l'heure (colonne 3) séparées par une tabulation.
###### 3. Affichez l'heure (colonne 3) avant la date (colonne 2 + colonne 1) séparées par des tirets (-).
###### 4. Affichez l'intégralité du fichier (équivalent à cat).
###### 5. Affichez la chaîne "Log de :" avant le nom du serveur (colonne 4).
###### 6. Affichez le dernier mot de chaque ligne de log (la dernière colonne), peu importe la longueur de la ligne.
###### 7. Affichez le nombre total de lignes ainsi que le nombre total de chaque colonne dans la ligne correspondante (field).
###### 8. Dans server.log, affichez la longueur (nombre de caractères) de chaque ligne.

**Exemple de sortie attendue pour la question 7 :**
```bash
1 11
2 13
...
```
###### 9. Au niveau du fichier `passwd_lab`, affichez le tout premier champ (nom d'utilisateur).
###### 10. Dans `passwd_lab`, affichez le nom d'utilisateur (colonne 1) et son Shell (colonne 7), séparé par une double tabulation .
###### 11. Dans `inventory.csv`, affichez uniquement les adresses IP (colonne 2).
###### 12. Dans `inventory.csv`, affichez le nom d'hôte (col 1) et le rôle (col 3) dans le format suivant : `Le host web01 à pour rôle Web`
###### 13. Dans `inventory.csv`, modifiez le séparateur de sortie (OFS) pour que les colonnes extraites soient séparées par des tabulations au lieu d'espaces.
###### 14. Dans `inventory.csv`, étant donné que tout les ip's font partie du même sous-réseau, affichez uniquement le dernier octet de chaque adresse IP.
###### 15. Affichez uniquement les lignes de `server.log` qui contiennent le mot "Error".
###### 16. Affichez toutes les lignes qui ne contiennent pas "Error" dans `server.log`.
###### 17. Affichez les lignes dans `server.log` qui commencent par "Nov".
###### 18. Affichez les lignes dans `server.log` qui ne commencent pas par "Nov".
##### 19. Dans `server.log`, affichez les lignes qui se terminent par “ssh2”.
###### 20. Affichez uniquement la ligne numéro 5 du fichier `server.log`.
###### 21. Affichez les lignes allant de la ligne 2 à la ligne 4 incluse avec leur numéro de ligne du fichier `server.log`.
###### 22. Affichez uniquement l'heure (colonne 3) pour les lignes contenant "sshd" du fichier `server.log`.
###### 23. Dans `passwd_lab`, affiche les lignes complètes où l'UID (colonne 3) est supérieur ou égal à 1000.
###### 24. Dans `passwd_lab`, affiche uniquement les noms d'utilisateurs (col 1) qui ont un UID (col 3) < 10.
###### 25. Dans `inventory.csv`, affiche les lignes où le Rôle (col 3) est exactement égal à "Web".
###### 26. Dans `inventory.csv`, affiche les serveurs qui ont plus de 80% d'utilisation disque (col 5 > 80).
###### 27. (Avancé) Dans `inventory.csv`, affiche les serveurs qui ont comme rôle "Web" et qui ont une utilisation disque < 60%.
###### 28. Isolez et affichez toutes les addresses IP (sans le masque /24) sauf la 127.0.0.1 dans le fichier `ip_output.txt`.
###### 29. Dans passwd_lab, affichez les lignes dont le GID (col 4) n’est pas égal à l’UID (col 3).
###### 30. Dans `passwd_lab`, affiche uniquement le nom des utilisateurs dont le shell est /bin/bash.
###### 31. Dans `passwd_lab`, compte combien il y a de lignes au total (en utilisant AWK, pas wc).
###### 32. Dans `passwd_lab`, affichez la dernière ligne du fichier.
###### 33. Dans `server.log`, affiche le numéro de ligne devant chaque message d'erreur contenant "Failed".
###### 34. Dans `inventory.csv`, n'affiche que les noms d'hôtes (col 1) en sautant la première ligne (l'en-tête).
###### 35. Dans server.log, extrais uniquement les processus qui contiennent sshd (col5, ex: `sshd[1234]:`) sans le reste de la ligne.
###### 36. (Variante) Dans server.log, extrais uniquement les processus (ex: sshd[1234]:) sans le reste de la ligne (colonne 5) Uniquement le PID.
###### 37. Modifie l'affichage de passwd_lab pour qu'il dise : "L'utilisateur [NOM] utilise le shell [SHELL]" pour chaque ligne. L'utilisateur avec `'` obligatoirement.
###### 38. Affiche le contenu de inventory.csv, mais remplace toutes les virgules par des tirets - à l'affichage.
##### 39. Afficher les utilisateurs avec leur shell spécifique dans le fichier `tricky_one`, en ignorant les lignes commentées (commençant par #) et les lignes vides en affichant le résultat dans le style `L'utilisateur root utilise le shell /bin/bash`.


##### Bonus : 
###### B1. Dans inventory.csv, calculez et affichez le total de RAM (colonne 4).
###### B2. Dans tricky_one, affichez uniquement les lignes qui ont exactement 7 champs.
###### B3. Dans passwd_lab, comptez combien d’utilisateurs utilisent chaque shell (table associative).
###### B4. Dans ip_output.txt, affichez les interfaces et le total de lignes par interface.
###### B5. Dans server.log, affichez un résumé du nombre d’erreurs par type (Error, Failed, Timeout).
###### B6. Dans passwd_lab, faites un tableau aligné : utilisateur | UID | Shell.



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
awk '{print $0}' server.log
```

###### 5. Affichez la chaîne "Log de :" avant le nom du serveur (colonne 4).
```bash
awk '{print "Log de : " $4}' server.log
```

###### 6. Affichez le dernier mot de chaque ligne de log (la dernière colonne), peu importe la longueur de la ligne.
```bash
awk '{print $NF}' server.log
```

###### 7. Affichez le nombre total de lignes ainsi que le nombre total de chaque colonne dans la ligne correspondante (field). 
```bash
awk '{print NR, NF}' server.log
```
###### 8. Dans server.log, affichez la longueur (nombre de caractères) de chaque ligne.
```bash
awk '{print length($0)}' server.log
```

###### 9. Au niveau du fichier `passwd_lab`, affichez le tout premier champ (nom d'utilisateur).
```bash
awk -F: '{print $1}' passwd_lab
```

###### 10. Dans `passwd_lab`, affichez le nom d'utilisateur (colonne 1) et son Shell (colonne 7), séparé par une double tabulation .
```bash
awk -F: '{print $1 "\t\t" $7}' passwd_lab
```

###### 11. Dans `inventory.csv`, affichez uniquement les adresses IP (colonne 2).
```bash
awk -F, '{print $2}' inventory.csv
```

###### 12. Dans `inventory.csv`, affichez le nom d'hôte (col 1) et le rôle (col 3) dans le format suivant : `Le host web01 à pour rôle Web`
```bash
awk -F, '{print "Le host " $1 " à pour rôle " $3}' inventory.csv
```

###### 13. Dans `inventory.csv`, modifiez le séparateur de sortie (OFS) pour que les colonnes extraites soient séparées par des tabulations au lieu d'espaces.
```bash
awk -F, -v OFS="\t" '{print $1, $3}' inventory.csv
```

###### 14. Dans `inventory.csv`, étant donné que tout les ip's font partie du même sous-réseau, affichez uniquement le dernier octet de chaque adresse IP.
```bash
awk -F, '{print $2}'  inventory.csv | awk -F. '{print $4}'
```

Biensur il existe d'autres méthodes pour faire cette question 13, par exemple en une seule commande awk, mais gardons la pédagogie :
```bash
awk -F, '{split($2, a, "."); print a[4]}' inventory.csv
```

###### 15. Affichez uniquement les lignes de `server.log` qui contiennent le mot "Error".
```bash
awk '/Error/' server.log
```

###### 16. Affichez toutes les lignes qui ne contiennent pas "Error" dans `server.log`.
```bash
awk '!/Error/' server.log

```

###### 17. Affichez les lignes dans `server.log` qui commencent par "Nov".
```bash
awk '/^Nov/' server.log
```

###### 18. Affichez les lignes dans `server.log` qui ne commencent pas par "Nov".
```bash
awk '!/^Nov/' server.log
```

##### 19. Dans `server.log`, affichez les lignes qui se terminent par “ssh2”.
```bash
awk '/ssh2$/' server.log
```

###### 20. Affichez uniquement la ligne numéro 5 du fichier `server.log`.
```bash
awk 'NR==5' server.log
```

###### 21. Affichez les lignes allant de la ligne 2 à la ligne 4 incluse avec leur numéro de ligne du fichier `server.log`.

```bash
awk 'NR>=4 && NR<=6 {print NR, $0}' server.log
```

###### 22. Affichez uniquement l'heure (colonne 3) pour les lignes contenant "sshd" du fichier `server.log`.
```bash
awk '/sshd/ {print $3}' server.log
```

###### 23. Dans `passwd_lab`, affiche les lignes complètes où l'UID (colonne 3) est supérieur ou égal à 1000.
```bash
awk -F: '$3 >= 1000' passwd_lab
```

###### 24. Dans `passwd_lab`, affiche uniquement les noms d'utilisateurs (col 1) qui ont un UID (col 3) < 10.
```bash
awk -F: '$3 < 10 {print $1}' passwd_lab
```

###### 25. Dans `inventory.csv`, affiche les lignes où le Rôle (col 3) est exactement égal à "Web".
```bash
awk -F, '$3 == "Web"' inventory.csv
```

###### 26. Dans `inventory.csv`, affiche les serveurs qui ont plus de 80% d'utilisation disque (col 5 > 80).
```bash
awk -F, '$5 > 80' inventory.csv
```

###### 27. (Avancé) Dans `inventory.csv`, affiche les serveurs qui ont comme rôle "Web" et qui ont une utilisation disque < 60%.
```bash
awk -F, '$3 == "Web" && $5 < 60' inventory.csv
```

###### 28. Isolez et affichez toutes les addresses IP (sans le masque /24) sauf la 127.0.0.1 dans le fichier `ip_output.txt`.
```bash
awk '/inet/ {print $2}' ip_output.txt | awk '!/127.0.0.1/' | awk -F/ '{print $1}'
```

Note : Il existe d'autres méthode plus élagantes, à vous de jouer.

###### 29. Dans passwd_lab, affichez les lignes dont le GID (col 4) n’est pas égal à l’UID (col 3).
```bash
awk -F: '$3 != $4' passwd_lab
```
###### 30. Dans `passwd_lab`, affiche uniquement le nom des utilisateurs dont le shell est /bin/bash.

```bash
awk -F: '$NF == "/bin/bash" {print $1}' passwd_lab
ou
awk -F: '$7 == "/bin/bash" {print $1}' passwd_lab
```

###### 31. Dans `passwd_lab`, compte combien il y a de lignes au total (en utilisant AWK, pas wc).
```bash
awk 'END {print NR}' passwd_lab
```

###### 32. Dans `passwd_lab`, affichez la dernière ligne du fichier.
```bash
awk 'END {print $0}' passwd_lab
```

###### 33. Dans `server.log`, affiche le numéro de ligne devant chaque message d'erreur contenant "Failed".
```bash
awk '/Failed/ {print NR, $0}' server.log
```

###### 34. Dans `inventory.csv`, n'affiche que les noms d'hôtes (col 1) en sautant la première ligne (l'en-tête).
```bash
awk -F, 'NR>1 {print $1}' inventory.csv

```

###### 35. Dans server.log, extrais uniquement les processus qui contiennent sshd (col5, ex: `sshd[1234]:`) sans le reste de la ligne.
```bash
awk '/sshd/ {print $5}' server.log
```
###### 36. (Variante) Dans server.log, extrais uniquement les processus (ex: sshd[1234]:) sans le reste de la ligne (colonne 5) Uniquement le PID.
```bash
awk '/sshd/ {match($5, /\[([0-9]+)\]/, arr); print arr[1]}' server.log
```

###### 37. Modifie l'affichage de passwd_lab pour qu'il dise : "L'utilisateur [NOM] utilise le shell [SHELL]" pour chaque ligne. L'utilisateur avec `'` obligatoirement.

```bash
awk -F: '{print "L'\''utilisateur " $1 " utilise le shell " $7}' passwd_lab
```

###### 38. (Bonus nettoyage) Affiche le contenu de inventory.csv, mais remplace toutes les virgules par des tirets - à l'affichage.

```bash
awk -F: '{print "L'\''utilisateur " $1 " utilise le shell " $7}' passwd_lab
```
- `-v OFS="-"` : On définit le séparateur de sortie (Output Field Separator) comme étant un tiret.

- `$1=$1` : C'est une astuce nécessaire dans AWK. Elle force AWK à "reconstruire" la ligne en utilisant le nouveau OFS. jf

##### 39. Afficher les utilisateurs avec leur shell spécifique dans le fichier `tricky_one`, en ignorant les lignes commentées (commençant par #) et les lignes vides en affichant le résultat dans le style `L'utilisateur root utilise le shell /bin/bash`.

```bash
awk -F: '!/^#/ && NF {print "L'\''utilisateur " $1 " utilise le shell " $NF}' tricky_one
```


##### Bonus : 
###### B1. Dans inventory.csv, calculez et affichez le total de RAM (colonne 4).
```bash
awk -F, 'NR>1 {total += $4} END {print total}' inventory.csv
```
###### B2. Dans tricky_one, affichez uniquement les lignes qui ont exactement 7 champs.
```bash
awk -F: 'NF==7' tricky_one
```


###### B3. Dans passwd_lab, comptez combien d’utilisateurs utilisent chaque shell (table associative).
```bash
awk -F: '{shell[$7]++} END {for (s in shell) print s, shell[s]}' passwd_lab
```
###### B4. Dans ip_output.txt, affichez les interfaces et le total de lignes par interface.
```bash
awk '/^[0-9]+: / {if (iface) print iface, count; iface=$2; count=0} /inet / {count++} END {if (iface) print iface, count}' ip_output.txt
```

###### B5. Dans server.log, affichez un résumé du nombre d’erreurs par type (Error, Failed, Timeout).
```bash
awk '/Error/ {error++} /Failed/ {failed++} /Timeout/ {timeout++} END {print "Error:", error; print "Failed:", failed; print "Timeout:", timeout}' server.log
``` 

###### B6. Dans passwd_lab, faites un tableau aligné : utilisateur | UID | Shell.
```bash
awk -F: '{printf "%-15s %-8s %-20s\n", $1, $3, $7}' passwd_lab
```