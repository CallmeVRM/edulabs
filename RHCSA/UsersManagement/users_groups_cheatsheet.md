---
title: Cheatsheet - Users & Groups
parent: UsersManagement
grand_parent: RHCSA
nav_order: 2
parent: RHCSA
has_children: true
nav_exclude: false
---

# 🔐 CHEATSHEET - Gestion des Utilisateurs et Groupes RHEL/CentOS

## 📋 Table des Matières
- [Fichiers de Configuration](#fichiers-de-configuration)
- [Gestion des Utilisateurs](#gestion-des-utilisateurs)
- [Gestion des Groupes](#gestion-des-groupes)
- [Gestion des Mots de Passe](#gestion-des-mots-de-passe)
- [Permissions et Propriétés](#permissions-et-propriétés)
- [Sudo et Privilèges](#sudo-et-privilèges)
- [Commandes de Consultation](#commandes-de-consultation)
- [User ID (UID) et Group ID (GID)](#user-id-uid-et-group-id-gid)
- [Comptes Système vs Utilisateurs](#comptes-système-vs-utilisateurs)
- [Scripts et Automatisation](#scripts-et-automatisation)

---

## 📁 Fichiers de Configuration

### Fichiers Principaux
```bash
/etc/passwd          # Informations des utilisateurs
/etc/shadow          # Mots de passe chiffrés
/etc/group           # Informations des groupes
/etc/gshadow         # Mots de passe des groupes (rarement utilisé)
/etc/login.defs      # Configuration par défaut (création user)
/etc/default/useradd # Valeurs par défaut useradd
/etc/skel/           # Fichiers template pour nouveaux users
/etc/sudoers         # Configuration sudo
/etc/sudoers.d/      # Fichiers sudo supplémentaires
```

### Structure de /etc/passwd
```
username:x:UID:GID:GECOS:home_directory:shell
john:x:1001:1001:John Doe,Room 101,555-1234:/home/john:/bin/bash
```
- **username**: nom d'utilisateur
- **x**: mot de passe (dans /etc/shadow)
- **UID**: User ID
- **GID**: Group ID principal
- **GECOS**: informations supplémentaires
- **home_directory**: répertoire personnel
- **shell**: shell par défaut

### Structure de /etc/shadow
```
username:$6$salt$hash:last_change:min:max:warn:inactive:expire:reserved
john:$6$xyz...:19000:0:99999:7:::
```
- **$6$**: SHA-512 (algorithme de hachage)
- **last_change**: jours depuis 01/01/1970 du dernier changement
- **min**: jours minimum entre changements
- **max**: jours maximum avant changement obligatoire
- **warn**: jours d'avertissement avant expiration
- **inactive**: jours d'inactivité avant désactivation
- **expire**: date d'expiration du compte

### Structure de /etc/group
```
groupname:x:GID:members
developers:x:1500:john,jane,bob
```

---

## 👤 Gestion des Utilisateurs

### Création d'Utilisateurs

```bash
# Création basique
useradd username

# Création avec options complètes
useradd -m -d /home/username -s /bin/bash -c "Nom Complet" -G group1,group2 username

# Options importantes
useradd -m username              # Créer le home directory
useradd -d /custom/path username # Home directory personnalisé
useradd -s /bin/zsh username     # Shell personnalisé
useradd -c "John Doe" username   # Commentaire/nom complet
useradd -G group1,group2 username # Groupes secondaires
useradd -g groupname username    # Groupe principal
useradd -u 1500 username         # UID spécifique
useradd -e 2024-12-31 username   # Date d'expiration
useradd -f 30 username           # Jours d'inactivité avant désactivation
useradd -r username              # Créer un compte système (UID < 1000)
useradd -M username              # Ne pas créer de home directory
useradd -k /etc/skel username    # Répertoire skeleton personnalisé

# Création interactive
adduser username                 # Version interactive (si disponible)

# Créer avec mot de passe en une ligne
useradd -m username && echo "username:password" | chpasswd
```

### Modification d'Utilisateurs

```bash
# Modifier les informations utilisateur
usermod -l new_username old_username  # Changer le nom
usermod -d /new/home -m username      # Changer et déplacer home
usermod -s /bin/zsh username          # Changer le shell
usermod -c "Nouveau Nom" username     # Changer le commentaire
usermod -u 2000 username              # Changer l'UID
usermod -g newgroup username          # Changer groupe principal
usermod -G group1,group2 username     # Remplacer groupes secondaires
usermod -aG group3 username           # Ajouter à un groupe (append)
usermod -L username                   # Verrouiller le compte
usermod -U username                   # Déverrouiller le compte
usermod -e 2024-12-31 username        # Définir date d'expiration
usermod -e "" username                # Supprimer date d'expiration
usermod -f 30 username                # Inactivité avant désactivation

# Exemples pratiques
usermod -aG wheel username            # Ajouter aux sudoers (RHEL)
usermod -aG docker username           # Ajouter au groupe docker
usermod -s /sbin/nologin username     # Désactiver login
```

### Suppression d'Utilisateurs

```bash
# Supprimer utilisateur (garde home et mail)
userdel username

# Supprimer avec home et mail
userdel -r username

# Supprimer et forcer (même si connecté)
userdel -f username

# Supprimer avec backup
userdel -r -Z username  # Supprime aussi contexte SELinux

# Vérifier avant suppression
find / -user username 2>/dev/null  # Trouver tous les fichiers
ps -u username                     # Vérifier processus actifs
```

---

## 👥 Gestion des Groupes

### Création de Groupes

```bash
# Création basique
groupadd groupname

# Avec options
groupadd -g 1500 groupname        # GID spécifique
groupadd -r groupname             # Groupe système
groupadd -f groupname             # Force (ne plante pas si existe)
groupadd -o -g 1000 groupname     # Permet GID dupliqué (non-unique)

# Exemples
groupadd developers
groupadd -g 2000 admins
groupadd -r systemgroup
```

### Modification de Groupes

```bash
# Renommer un groupe
groupmod -n newname oldname

# Changer le GID
groupmod -g 2500 groupname

# Changer GID même si groupe utilisé
groupmod -o -g 1000 groupname
```

### Suppression de Groupes

```bash
# Supprimer un groupe
groupdel groupname

# Vérifier avant suppression
grep groupname /etc/group
find / -group groupname 2>/dev/null
```

### Gestion des Membres

```bash
# Ajouter utilisateur au groupe
usermod -aG groupname username
gpasswd -a username groupname

# Retirer utilisateur du groupe
gpasswd -d username groupname

# Définir les membres du groupe (écrase existants)
gpasswd -M user1,user2,user3 groupname

# Définir un administrateur de groupe
gpasswd -A admin_user groupname

# Retirer tous les administrateurs
gpasswd -A "" groupname

# Définir mot de passe de groupe (rare)
gpasswd groupname
```

---

## 🔑 Gestion des Mots de Passe

### Définir/Changer les Mots de Passe

```bash
# Changer son propre mot de passe
passwd

# Changer le mot de passe d'un utilisateur (root)
passwd username

# Définir mot de passe en mode non-interactif
echo "username:newpassword" | chpasswd
echo 'newpassword' | passwd --stdin username  # RHEL uniquement

# Générer mot de passe sécurisé
openssl rand -base64 12
pwgen 16 1  # Si installé
```

### Politiques de Mots de Passe

```bash
# Forcer changement au prochain login
passwd -e username
chage -d 0 username

# Verrouiller/Déverrouiller compte
passwd -l username    # Lock
passwd -u username    # Unlock
usermod -L username   # Lock (alternative)
usermod -U username   # Unlock (alternative)

# Définir expiration
passwd -n 7 username   # Min 7 jours entre changements
passwd -x 90 username  # Max 90 jours avant changement
passwd -w 14 username  # Avertir 14 jours avant expiration
passwd -i 30 username  # 30 jours inactivité avant désactivation

# Supprimer le mot de passe (dangereux!)
passwd -d username

# Voir le statut
passwd -S username
```

### Commande chage (Change Age)

```bash
# Voir les informations d'expiration
chage -l username

# Modifier les paramètres d'expiration
chage -m 7 username       # Min days entre changements
chage -M 90 username      # Max days avant changement obligatoire
chage -W 14 username      # Warning days
chage -I 30 username      # Inactive days
chage -E 2024-12-31 username  # Date d'expiration
chage -E -1 username      # Pas d'expiration

# Mode interactif
chage username

# Exemples pratiques
chage -d 0 username                    # Forcer changement
chage -M 60 -W 7 -I 14 username       # Politique stricte
chage -E $(date -d "+90 days" +%Y-%m-%d) username  # Expire dans 90 jours
```

---

## 🔐 Permissions et Propriétés

### Changer Propriétaire et Groupe

```bash
# Changer propriétaire
chown username file
chown username:groupname file
chown -R username directory/     # Récursif

# Changer seulement le groupe
chown :groupname file
chgrp groupname file

# Exemples
chown john:developers /home/john/project
chown -R www-data:www-data /var/www/html
chown --reference=file1 file2    # Copier propriété de file1
```

### Permissions et Masques

```bash
# Voir umask actuel
umask

# Définir umask
umask 0022    # Fichiers: 644, Dossiers: 755
umask 0027    # Fichiers: 640, Dossiers: 750
umask 0077    # Fichiers: 600, Dossiers: 700

# Umask permanent
echo "umask 0027" >> ~/.bashrc
echo "umask 0027" >> /etc/bashrc  # Global
```

### ACL (Access Control Lists)

```bash
# Installer si nécessaire
yum install acl

# Voir les ACL
getfacl file

# Définir ACL
setfacl -m u:username:rwx file        # User
setfacl -m g:groupname:rx file        # Group
setfacl -m o::r file                  # Others
setfacl -R -m u:username:rwx directory/  # Récursif

# ACL par défaut (nouveaux fichiers)
setfacl -d -m u:username:rwx directory/

# Retirer ACL
setfacl -x u:username file
setfacl -b file                       # Toutes les ACL

# Copier ACL
getfacl file1 | setfacl --set-file=- file2
```

---

## 🛡️ Sudo et Privilèges

### Configuration Sudo

```bash
# Éditer sudoers (toujours avec visudo!)
visudo
visudo -f /etc/sudoers.d/custom

# Vérifier syntaxe
visudo -c
```

### Syntaxe /etc/sudoers

```bash
# Format: user HOST=(RUN_AS) COMMANDS

# Donner tous les droits
username ALL=(ALL) ALL
%groupname ALL=(ALL) ALL

# Sans mot de passe
username ALL=(ALL) NOPASSWD: ALL
%wheel ALL=(ALL) NOPASSWD: ALL

# Commandes spécifiques
username ALL=(ALL) /usr/bin/systemctl, /usr/bin/journalctl
username ALL=(ALL) /usr/bin/systemctl restart httpd

# Exécuter en tant qu'autre utilisateur
username ALL=(otheruser) /path/to/command

# Avec alias
Cmnd_Alias SERVICES = /usr/bin/systemctl restart *, /usr/bin/systemctl status *
User_Alias ADMINS = john, jane, bob
ADMINS ALL=(ALL) SERVICES

# Variables d'environnement
Defaults env_keep += "COLORS DISPLAY"
Defaults secure_path = /usr/local/bin:/usr/bin:/bin

# Exemples pratiques
%developers ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp
john ALL=(ALL) NOPASSWD: /usr/bin/docker, /usr/bin/docker-compose
```

### Utilisation de Sudo

```bash
# Exécuter une commande
sudo command

# Devenir root
sudo -i          # Login shell
sudo -s          # Shell non-login
sudo su -        # Alternative

# Exécuter en tant qu'autre utilisateur
sudo -u username command

# Voir privilèges sudo
sudo -l
sudo -ll         # Format long

# Éditer fichier avec sudo
sudo -e /etc/hosts
sudoedit /etc/hosts

# Exécuter avec variables d'environnement
sudo -E command

# Historique sudo
sudo journalctl -u sudo
```

---

## 🔍 Commandes de Consultation

### Informations Utilisateurs

```bash
# Qui est connecté
who
w
users
last            # Historique connexions
lastb           # Tentatives échouées
lastlog         # Dernière connexion par user

# Informations sur un utilisateur
id username
finger username  # Si installé
getent passwd username

# Tous les utilisateurs
cat /etc/passwd
getent passwd
cut -d: -f1 /etc/passwd  # Juste les noms

# Utilisateurs connectés
w -h | cut -d' ' -f1 | sort -u
who | awk '{print $1}' | sort -u
```

### Informations Groupes

```bash
# Groupes d'un utilisateur
groups username
id -Gn username

# Tous les groupes
cat /etc/group
getent group

# Membres d'un groupe
getent group groupname
grep groupname /etc/group
lid -g groupname  # Si installé
members groupname # Si installé
```

### Recherches et Filtres

```bash
# Utilisateurs avec UID spécifique
awk -F: '$3 >= 1000 {print $1}' /etc/passwd  # Users normaux
awk -F: '$3 < 1000 {print $1}' /etc/passwd   # System users

# Utilisateurs sans mot de passe
awk -F: '($2 == "") {print $1}' /etc/shadow

# Utilisateurs avec shell spécifique
grep "/bin/bash" /etc/passwd
grep -v "/sbin/nologin" /etc/passwd  # Avec shell actif

# Comptes expirés
chage -l username | grep "Account expires"

# Trouver fichiers d'un utilisateur
find / -user username 2>/dev/null
find /home -group groupname 2>/dev/null
```

---

## 🆔 User ID (UID) et Group ID (GID)

### Plages Standards RHEL

```bash
# UID 0: root
# UID 1-200: Statiquement alloués (système)
# UID 201-999: Services/daemons système
# UID 1000+: Utilisateurs normaux

# Voir les plages dans la config
grep "^UID_MIN\|^UID_MAX\|^SYS_UID" /etc/login.defs
grep "^GID_MIN\|^GID_MAX\|^SYS_GID" /etc/login.defs
```

### Gestion UID/GID

```bash
# Trouver prochain UID/GID disponible
awk -F: '{print $3}' /etc/passwd | sort -n | tail -1
awk -F: '{print $3}' /etc/group | sort -n | tail -1

# Changer UID
usermod -u 2000 username
find / -user OLD_UID -exec chown -h NEW_UID {} \;

# Changer GID
groupmod -g 2000 groupname
find / -group OLD_GID -exec chgrp -h NEW_GID {} \;

# Vérifier conflits UID/GID
awk -F: '{print $3}' /etc/passwd | sort | uniq -d
awk -F: '{print $3}' /etc/group | sort | uniq -d
```

---

## ⚙️ Comptes Système vs Utilisateurs

### Différences

```bash
# Compte système
useradd -r -s /sbin/nologin -d /var/lib/service service_user
# - UID < 1000
# - Pas de home dans /home
# - Shell /sbin/nologin ou /bin/false
# - Pas d'expiration

# Utilisateur normal
useradd -m -s /bin/bash regular_user
# - UID >= 1000
# - Home dans /home
# - Shell interactif
# - Peut expirer
```

### Comptes de Service

```bash
# Créer compte pour application
useradd -r -s /sbin/nologin -d /opt/myapp -c "MyApp Service" myapp
chown -R myapp:myapp /opt/myapp

# Exemples de comptes système
apache / www-data   # Web server
mysql              # Database
nginx              # Web server
redis              # Cache
postfix            # Mail server
```

---

## 🔧 Scripts et Automatisation

### Création en Masse

```bash
# Script de création d'utilisateurs
#!/bin/bash
# users.txt: username:password:fullname:groups

while IFS=: read -r username password fullname groups; do
    useradd -m -c "$fullname" -G "$groups" "$username"
    echo "$username:$password" | chpasswd
    chage -d 0 "$username"  # Forcer changement mot de passe
done < users.txt

# Avec newusers (plus efficace)
# Format: username:password:uid:gid:gecos:home:shell
newusers users.txt
```

### Script de Nettoyage

```bash
#!/bin/bash
# Trouver utilisateurs inactifs (>90 jours)

for user in $(awk -F: '$3 >= 1000 {print $1}' /etc/passwd); do
    last_login=$(lastlog -u "$user" | tail -1 | awk '{print $4,$5,$6}')
    if [ "$last_login" == "Never logged in" ]; then
        echo "Jamais connecté: $user"
    else
        # Logique pour vérifier date
        echo "$user: $last_login"
    fi
done
```

### Audit et Rapport

```bash
#!/bin/bash
# Rapport utilisateurs

echo "=== RAPPORT UTILISATEURS ==="
echo "Total utilisateurs: $(wc -l < /etc/passwd)"
echo "Utilisateurs normaux: $(awk -F: '$3 >= 1000 {print $1}' /etc/passwd | wc -l)"
echo "Comptes système: $(awk -F: '$3 < 1000 {print $1}' /etc/passwd | wc -l)"
echo ""
echo "Utilisateurs avec sudo:"
getent group wheel sudo 2>/dev/null | cut -d: -f4
echo ""
echo "Comptes verrouillés:"
awk -F: '($2 ~ /^!/) {print $1}' /etc/shadow
echo ""
echo "Comptes sans mot de passe:"
awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow
```

### Backup et Restauration

```bash
# Sauvegarder configuration utilisateurs
tar -czf users_backup_$(date +%Y%m%d).tar.gz \
    /etc/passwd /etc/shadow /etc/group /etc/gshadow \
    /home

# Backup sélectif
getent passwd | grep -E ":(100[0-9]|10[1-9][0-9]):" > passwd.backup
getent shadow | grep -E "^(user1|user2|user3):" > shadow.backup
getent group | grep -E ":(100[0-9]|10[1-9][0-9]):" > group.backup
```

---

## 📊 Commandes Avancées et Tips

### Vérifications de Sécurité

```bash
# Comptes avec UID 0 (danger!)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Comptes avec shell et pas de password
awk -F: '($2 == "" && $7 !~ /nologin|false/) {print $1}' /etc/passwd

# Fichiers SUID/SGID
find / -perm /6000 -type f 2>/dev/null

# Vérifier intégrité
pwck      # Vérifier /etc/passwd et /etc/shadow
grpck     # Vérifier /etc/group et /etc/gshadow
```

### Performance et Optimisation

```bash
# Utiliser getent au lieu de cat (LDAP/NIS friendly)
getent passwd username
getent group groupname

# Cache nscd (si utilisé)
systemctl restart nscd
nscd -i passwd
nscd -i group
```

### Troubleshooting

```bash
# Utilisateur ne peut pas se connecter
passwd -S username           # Vérifier status
chage -l username           # Vérifier expiration
grep username /var/log/secure  # Logs auth
journalctl -u sshd | grep username

# Réinitialiser compte bloqué
passwd -u username
pam_tally2 --user=username --reset  # Si pam_tally2
faillock --user username --reset    # Si faillock

# Problèmes de permissions home
ls -la /home/username
chmod 700 /home/username
chown -R username:username /home/username
```

---

## 📝 Bonnes Pratiques

1. **Toujours utiliser `visudo`** pour éditer /etc/sudoers
2. **Ne jamais éditer directement** /etc/passwd, /etc/shadow (utiliser les commandes)
3. **Utiliser `-aG`** avec usermod pour ajouter aux groupes (pas `-G` seul)
4. **Forcer le changement de mot de passe** pour nouveaux utilisateurs
5. **Politique de mots de passe** stricte (complexité, expiration)
6. **Audit régulier** des comptes et permissions
7. **Supprimer les comptes inutilisés** régulièrement
8. **Utiliser ACL** pour permissions granulaires
9. **Groupes pour gestion** plutôt que permissions individuelles
10. **Backup régulier** de /etc/passwd, /etc/shadow, /etc/group

---

## 🎯 Cas d'Usage Pratiques

### Créer un Développeur Web

```bash
useradd -m -s /bin/bash -c "John Developer" -G wheel,docker jdoe
passwd jdoe
usermod -aG apache jdoe
mkdir -p /var/www/jdoe
chown jdoe:apache /var/www/jdoe
chmod 775 /var/www/jdoe
setfacl -R -m d:g:apache:rwx /var/www/jdoe
```

### Créer un Compte de Service

```bash
useradd -r -s /sbin/nologin -d /opt/myapp myapp
mkdir -p /opt/myapp/{bin,config,logs}
chown -R myapp:myapp /opt/myapp
chmod 750 /opt/myapp
# Ajouter au sudoers pour restart service
echo "myapp ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp" > /etc/sudoers.d/myapp
```

### Migration d'Utilisateur

```bash
# Ancien serveur
tar -czf user_migration.tar.gz /home/username
grep "^username:" /etc/passwd /etc/shadow /etc/group > user_info.txt

# Nouveau serveur
# 1. Créer avec même UID/GID
useradd -u 1500 -m -s /bin/bash username
# 2. Restaurer home
tar -xzf user_migration.tar.gz -C /
# 3. Restaurer password hash depuis /etc/shadow
usermod -p 'HASH_FROM_OLD_SERVER' username
```

---

**Version**: 2.0  
**Dernière mise à jour**: Octobre 2025  
**Compatible**: RHEL 7/8/9, CentOS, Rocky Linux, AlmaLinux
```


Structure de /etc/passwd
```bash
username:x:UID:GID:GECOS:home_directory:shell
john:x:1001:1001:John Doe,Room 101,555-1234:/home/john:/bin/bash
```

    username: nom d'utilisateur
    x: mot de passe (dans /etc/shadow)
    UID: User ID
    GID: Group ID principal
    GECOS: informations supplémentaires
    home_directory: répertoire personnel
    shell: shell par défaut


Structure de /etc/shadow
```bash
username:$6$salt$hash:last_change:min:max:warn:inactive:expire:reserved
john:$6$xyz...:19000:0:99999:7:::
