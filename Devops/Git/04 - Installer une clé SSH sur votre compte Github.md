---
title: 04 - Installer une clé SSH sur votre compte Github
parent: Git
grand_parent: Devops
nav_order: 4
nav_exclude: false
---

# 04 - Sécuriser l’Accès à GitHub avec une Clé SSH

Dans cette section, nous allons apprendre à sécuriser l’accès à GitHub en utilisant une clé SSH.
L’objectif est simple : remplacer l’authentification par mot de passe par une authentification cryptographique, plus sûre et plus pratique.

Ce mécanisme est essentiel en environnement professionnel, il permet de travailler avec GitHub sans ressaisir de mot de passe, tout en renforçant la sécurité des dépôts.

### Générer une clé SSH :
la première étape consiste à générer une paire de clés SSH (une clé publique et une clé privée) sur votre machine locale :

ssh-keygen -t ed25519 -C "Clé SSH pour GitHub"

le -C permet d’ajouter un commentaire pour identifier la clé plus facilement.

Git vous demandera : 
- le chemin où sauvegarder la clé (choisissez le votre ou appuyez sur Entrée pour le chemin par défaut)
- une passphrase (optionnelle mais recommandée pour plus de sécurité)

Une fois la clé générée, une image randomart sera affichée pour représenter visuellement la clé.

Si vous avez choisis le chemin par défaut, la clé privée sera stockée dans `~/.ssh/id_ed25519` et la clé publique dans `~/.ssh/id_ed25519.pub`.

**Si vous avez un gestionnaire de mot de passe (Keepass...) et vous devez en avoir un, vous pouvez y enregistrer la passphrase et la clé privée**

Exécuté la commande suivante pour afficher le contenu de la clé publique :
```bash
cat ~/.ssh/id_ed25519.pub
```
Copiez l’intégralité de la ligne affichée.

Cette clé contient un préfixe `ssh-ed25519` suivi d’une longue suite de caractères : c’est normal.

Vous avez la possibilité d'utiliser votre clé SSH pour s'authentifier et pour signé vos commits, Choisir `Authentication key` suffit totalement pour travailler sur un dépôt GitHub, mais si vous souhaitez également signer vos commits avec cette clé, vous devez répéter les étapes suivante deux fois et choisir `Signing key` à la deuxième reprise.

### Ajouter la clé SSH à votre compte GitHub :
Rendez-vous sur votre compte `GitHub` :
1. Allez dans les paramètres `Settings`
2. Dans le menu de gauche, cliquez sur `SSH and GPG keys`
3. Cliquez sur le bouton `New SSH key`
4. Donnez un nom explicite à votre clé (par exemple `Laptop perso`, `Laptop pro`)
5. Laissez le type sur `Authentication key`
6. Collez votre clé publique que vous venez de copier dans le champ `Key`
7. Validez

![alt text](./images/ssh_key_github_add.png "Ajouter une clé SSH sur GitHub")


**Différence entre Clés SSH d’Authentification et Clés de Signature**

GitHub distingue deux types de clés SSH, chacune ayant un rôle bien spécifique :

- Clé d’authentification (Authentication Key) : C’est la clé classique utilisée pour accéder à vos dépôts GitHub.

Elle permet de :
    - cloner un dépôt,
    - pousser des modifications,
    - récupérer des mises à jour (git pull).

Si votre objectif est simplement de travailler avec vos dépôts, une clé d’authentification suffit.

- Clé de signature (Signing Key)

Une clé de signature sert à signer cryptographiquement vos commits.

Elle garantit :
    - que le commit provient réellement de vous,
    - qu’il n’a pas été modifié ou falsifié,
    - qu’un tiers n’a pas usurpé votre nom ou email Git.

Ce mécanisme renforce la confiance dans l’historique du projet, surtout dans les environnements professionnels ou open source.

### Conclusion

Vous avez maintenant :

- généré une clé SSH sur votre machine,
- ajouté cette clé à votre compte GitHub,
- testé l’accès sécurisé en clonant un dépôt via SSH.

Ce mode de connexion est indispensable pour travailler efficacement en environnement professionnel.