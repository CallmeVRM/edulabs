---
title: Lab-01-Solution
parent: Scénarios
grand_parent: Linux
nav_order: 2
---

# 💡 Solutions du Lab Linux — Sprint 1

Cette page contient les corrections détaillées des tickets et incidents du Sprint 1.

---

## Tickets

<div style="border-left: 4px solid #6c757d; background-color: #f9f9f9; padding: 12px 20px; margin: 20px 0; font-family: 'Segoe UI', sans-serif; color: #333;">
  <strong style="color: #6c757d;">🎫 Ticket 1 :</strong>  
   Onboarding d’Alice Dupont
</div>

**Objectif**  
- Créer `alice.dupont`
- lui attribuer le shell `/bin/bash`
- L'ajouter au **groupe primaire** `marketing`
- La forcer à changer son mot de passe au 1er login.

```bash
#Créer l'utilisateur alice.dupont, lui attribuer le shell bash, et l'ajouter au groupe marketing comme groupe primare
useradd -m -c "Alice Dupont" -s /bin/bash -g marketing alice.dupont

#Définir un nouveau mot de passe temporaire
echo "alice.dupont:MotDePasse123!" | chpasswd

#Forcer Alice a changer son mot de passe au prochain login (équivalent: chage -d 0 alice.dupont)
passwd -e alice.dupont          
```

**Vérification :**
```bash
# Le groupe primaire = marketing
id alice.dupont   

#Vérifier que Alice à bien /bin/bash comme shell
getent passwd alice.dupont | grep ':/bin/bash'

# Vérifier que Alice possède bien son /home
ls -l /home/ 

# Doit indiquer "Password must be changed"
chage -l alice.dupont           
```

---

<div style="border-left: 4px solid #6c757d; background-color: #f9f9f9; padding: 12px 20px; margin: 20px 0; font-family: 'Segoe UI', sans-serif; color: #333;">
  <strong style="color: #6c757d;">🎫 Ticket 2 :</strong>  
   Groupe transverse com 
</div>

**Objectif**  

- Vérifier si le groupe `com` est présent.
- Créer le groupe `com`
- Ajouter `alice.dupont` au groupe `com`, ce groupe doit être son groupe *secondaire*

```bash
#Vérifier si le groupe com est existant
getent group com
#Créer le groupe com
groupadd -f com
#Ajouter alice.dupont au groupe com (en tant que groupe secondaire)
usermod -aG com alice.dupont
```

**Vérification :**
```bash
#Vérifier que alice.dupont est dans le groupe com, et que ce groupe est son groupe secondaire
id -nG alice.dupont | grep com
```

---

<div style="border-left: 4px solid #6c757d; background-color: #f9f9f9; padding: 12px 20px; margin: 20px 0; font-family: 'Segoe UI', sans-serif; color: #333;">
  <strong style="color: #6c757d;">🎫 Ticket 3 :</strong>  
   Partage Marketing (setgid) (Dev)
</div>

**Objectif**  

- Attribuer au groupe Marketing les permissions de lecture, écriture et exécution sur le répertoire `/lab/depts/marketing/share`, avec validation de la procédure (test)

En d'autres termes : Activer le setgid et droits d’équipe sur /lab/depts/marketing/share, avec héritage du groupe.

```bash
#Vérifier les permissions et droits actuelles, le groupe share est en lecture uniquement
ls -ld /lab/depts/marketing/share

#Tester l'écriture avec un utilisateur faisant partie du groupe marketing !
sudo -u alice.dupont touch /lab/depts/marketing/share/testfile

# Changer les propriétaire du répertoire share 
chown root:marketing /lab/depts/marketing/share

# Activer le setgid et modifier les droits d'équipe marketing
chmod 2770 /lab/depts/marketing/share
```

**Vérification :**

```bash
# Test d'héritage de groupe
sudo -u alice.dupont touch /lab/depts/marketing/share/testfile

#Doit retourner : 2770 root:marketing
stat -c '%a %U:%G' /lab/depts/marketing/share   
#Doit retourner  ...alice.dupont:marketing
stat -c '%n %U:%G' /lab/depts/marketing/share/testfile  

#Résultat attendu : drwxrws--- 2 root marketing
ls -ld /lab/depts/marketing/share
```

---

<div style="border-left: 4px solid #6c757d; background-color: #f9f9f9; padding: 12px 20px; margin: 20px 0; font-family: 'Segoe UI', sans-serif; color: #333;">
  <strong style="color: #6c757d;">🎫 Ticket 4 :</strong>  
   Squelette &amp; Bob Martin (Dev)
</div>

**Objectif**  

1. Mise à jour du squelette `/etc/skel` :
    - ajout d'un dossier `Documents`
    - ajout d'un alias `ll='ls -alF'`
2. Création d'un utilisateur martin.bob avec comme groupe primaire `dev`
3. Validation du bon fonctionnement de la procédure

```bash
# Squelette standard
install -d -m 0755 /etc/skel/Documents
echo "alias ll='ls -alF'" >> /etc/skel/.bashrc
# Créer Bob (après maj du skel)
useradd -m -d /home/bob.martin -s /bin/bash -g dev bob.martin
echo "bob.martin:Password123!" | chpasswd
```

**Vérification :**
```bash
#Groupe primaire doit être dev
id -nG bob.martin 
#Vérifier que le répertoire Documents est bien présent dans le home de bob.martin
ls -l /home/bob.martin | grep Documents
#Résultat attendu : alias ll='ls -alF'
cat /home/bob.martin/.bashrc | grep "alias ll='ls -alF'"
```


---

## 🚨 Incidents - Troubleshooting 🚨


<div style="border-left: 4px solid #774ae2; background-color: #f4f0fa; padding: 10px 15px; margin: 20px 0; border-radius: 4px; font-family: 'Segoe UI', sans-serif; color: #333;">
  <p style="margin: 0; font-size: 15px;">
    <strong style="color: #774ae2;">Incident INC-01 :</strong> « Je suis dans le groupe mais je ne peux pas écrire. Alice.dupont »
  </p>
</div>


**Ticket de alice.dupont :** « Je suis dans le groupe `marketing` mais je ne peux pas écrire »

**Symptômes :** Un membre de marketing ne peut pas créer dans `…/marketing/share`.

**Diagnostic :**

On fait un test pour reproduire l'erreur :
```bash
sudo -u alice.dupont touch /lab/depts/marketing/share/test
```

On à bien un problème de permission : `touch: cannot touch '/lab/depts/marketing/share/test': Permission denied`

On vérifie les droits :
```bash
stat -c '%a %U:%G' /lab/depts/marketing/share
```

**Correctif :**

```bash
#Changement de propriétaire
chown root:marketing /lab/depts/marketing/share
#Changement de permissions
chmod 2770 /lab/depts/marketing/share
```

**Vérification :**

```bash
#Création d'un fichier test dans le partage share avec l'utilisateur alice.dupont
sudo -u alice.dupont touch /lab/depts/marketing/share/test
#Doit retourner marketing
stat -c '%G' /lab/depts/marketing/share/test 
#Suppression du fichier test
rm -f /lab/depts/marketing/share/test
```

---

<div style="border-left: 4px solid #774ae2; background-color: #f4f0fa; padding: 10px 15px; margin: 20px 0; border-radius: 4px; font-family: 'Segoe UI', sans-serif; color: #333;">
  <p style="margin: 0; font-size: 15px;">
    <strong style="color: #774ae2;">Incident INC-02 :</strong> « Oups, j'ai supprimé par erreur le fichier d'un collègue. Alice.dupont »
  </p>
</div>

Dans le répertoire `/lab/depts/marketing/share`, Alice a supprimer le fichier de bob sans faire attention. Heuresement Bob possédait le fichier dans son drive, cependant, ce genre
d'incident ne doit plus se produire, trouvez une solution pour permettre aux utilsiateurs dugroupe marketing qui travaillent sur le dossier share de supprimer leurs propres fichiers
mais pas ceux des autres. Tout en gardant les possibilité de lectures/ecritures/exécutions. 

**Symptômes :**

Afficher l'état du répertoire pour avoir une idée sur le problème.

```bash
ls -ld /lab/depts/marketing/share
```
Le dossier est bien group-writable (rwxrwx---) mais le sticky-bit est absent.

Tout membre du groupe marketing peut supprimer n’importe quel fichier, même s’il n’en est pas propriétaire.

**Diagnostic :**

Reproduire le problème :

```bash
# Création d’un fichier de test dans le dossier "share"
# → exécuté en tant qu’utilisateur dhomas.tru
# → le fichier appartiendra à dhomas.tru et au groupe marketing
sudo -u dhomas.tru touch /lab/depts/marketing/share/coucouAlice

# Tentative de suppression du fichier par un autre utilisateur (alice.dupont)
# → si le sticky bit n’est PAS activé, la suppression sera possible
# → si le sticky bit est activé, la suppression échouera (seul le propriétaire peut supprimer)
sudo -u alice.dupont rm -f /lab/depts/marketing/share/coucouAlice
```


**Correctif :**

- Réactiver le sticky-bit tout en conservant l’écriture groupe :
```bash
chmod +t /lab/depts/marketing/share  
```
Aucun changement supplémentaire n’est nécessaire ; les droits rwx du groupe restent intacts.

**Vérification :**

```bash
# Création d’un fichier de test dans le dossier "share" par dhomas.tru
sudo -u dhomas.tru touch /lab/depts/marketing/share/coucouAlice

#Suppression du fichier par alice.dupont
sudo -u alice.dupont rm -f /lab/depts/marketing/share/coucouAlice

# Vérification des permissions du dossier "share"
# → permet de voir si le sticky bit est présent (drwxrwsT ou drwxrws+t)
# → sans sticky bit : les membres du groupe peuvent supprimer les fichiers des autres
# → avec sticky bit : seuls les propriétaires peuvent supprimer leurs fichiers
ls -ld /lab/depts/marketing/share
```

---

<div style="border-left: 4px solid #774ae2; background-color: #f4f0fa; padding: 10px 15px; margin: 20px 0; border-radius: 4px; font-family: 'Segoe UI', sans-serif; color: #333;">
  <p style="margin: 0; font-size: 15px;">
    <strong style="color: #774ae2;">Incident INC-03 : </strong> « Je n'arrive plus à changer mon mot de passe : Authentication token manipulation error. Camel.chalal »
  </p>
</div>

**Symptômes :**
- Suite à une manipulation hasardeuse de ma part `camel.chalal` je n'arrive plus à changer mon mot de passe, j'ai une erreur `passwd: Authentication token manipulation error`.

Mot de passe actuel : `Motdepasse123!`

```bash
#On simule le changement de mot de passe depuis le compte root sur le compte camel.chalal
sudo -u camel.chalal passwd
```
On à une erreur de type `passwd: Authentication token manipulation error`

**Diagnostic :**

```bash
#On force le changement de mot de passe en passant par le compte root directement
passwd camel.chalal
```

Echec, même en essayant de forcer le changement de mot de passe en utilisant le compte root, on n'arrive pas à changer le mot de passe de camel.chalal.

- Vérifier permissions/attribus de `/etc/shadow` :

```bash
#Vérification des permissions
ls -l /etc/shadow
```

#Résultats par défaut : -rw-r----- 1 root shadow ... /etc/shadow

```bash
#Vérification des attribus
lsattr /etc/shadow
```
Résultat : `----i---------e------- /etc/shadow`

Notre console retourne deux attribus `i` et `e` :

`i` → immutable : le fichier ne peut pas être modifié, supprimé, renommé ou écrasé, même par root.

`e` → extent format (lié au système de fichiers ext4, sans impact ici).

**Correctif :**

Retirer l'attribut d'immutabilité

```bash
chattr -i /etc/shadow || true
```

**Vérification :**
```bash
#Vérification des attribus à nouveau
lsattr /etc/shadow

#Tenter le changement de passe en simulant l'utilisateur camel.chalal
sudo -u camel.chalal passwd
```
---

<div style="border-left: 4px solid #774ae2; background-color: #f4f0fa; padding: 10px 15px; margin: 20px 0; border-radius: 4px; font-family: 'Segoe UI', sans-serif; color: #333;">
  <p style="margin: 0; font-size: 15px;">
    <strong style="color: #774ae2;">Incident INC-04 :</strong> « Je n'arrive pas à me connecter en SSH avec la nouvelle clé. Camel.chalal »
  </p>
</div>

**Symptômes :**
Votre collaborateur `camel.chalal` n'arrive pas à se connecter avec sa clé privé, le serveur semble ignorer l'authentification par clé et bascule en authentification par mot de passe.


**Diagnostic :**
1. Vérifier les permissions sur le répertoire /home/camel.chalal/.ssh et le fichier  /home/camel.chalal/.ssh/authorized_keys

```bash
# Le répertoire .ssh doit avoir les permissions : drwx------ (0700)
# Propriétaire et groupe : camel.chalal
ls -ld /home/camel.chalal/.ssh

# Le fichier authorized_keys doit avoir les permissions : -rw------- (0600)
# Propriétaire et groupe : camel.chalal
ls -ld /home/camel.chalal/.ssh/authorized_keys
```

Si les permissions sont trop permissives (ex : 0644 ou 0755), le serveur SSH ignore la clé pour des raisons de sécurité, ce qui explique le basculement vers l’authentification par mot de passe.

Dans ce cas précis, bien que les fichiers soient situés dans le répertoire de camel.chalal, ils appartiennent à sylvain.morel (la personne qui les a créés). Ce mauvais paramétrage des droits d’accès est conforme au comportement attendu du serveur SSH, qui refuse toute clé dont les permissions sont incorrectes ou trop ouvertes.

**Correctif :**
```bash
# Répertoire .ssh
chown camel.chalal:camel.chalal /home/camel.chalal/.ssh
chmod 0700 /home/camel.chalal/.ssh

# Fichier authorized_keys
chown camel.chalal /home/camel.chalal/.ssh/authorized_keys
chmod 0600 /home/camel.chalal/.ssh/authorized_keys
```

**Vérification :**
```bash
#Attendu : drwx------ (0700) camel.chalal camel.chalal
ls -ld /home/camel.chalal/.ssh

#Attendu : -rw------- (0600) camel.chalal camel.chalal
ls -ld /home/camel.chalal/.ssh/authorized_keys
```