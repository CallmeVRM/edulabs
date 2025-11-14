---
title: 03 - Cloner un dépôt Github distant
parent: Git
grand_parent: Devops
nav_order: 3
---

# Cloner un Dépôt GitHub Distant et Créer un Clone Local

Dans cette section, nous allons apprendre à cloner un dépôt GitHub distant sur notre machine locale en utilisant Git. Le clonage d'un dépôt permet de créer une copie locale complète du dépôt distant, y compris tout son historique de versions.

### Préparer l'emplacement local du dépôt

*Dans un environnement partagé, assurez-vous de mettre en place les permissions nécessaires pour que les autres collaborateurs puissent accéder au dépôt cloné, tout en ayant une stratégie du moindre privilège, qui consite à ne donner que les droits nécessaires aux personnes concernés.*

Commençons par créer une répertoire destiné à accueillir le clonage du dépôt GitHub.

```bash
mkdir ~/projetA
cd ~/projetA
```

### Cloner le dépôt distant
Le dépôt distant que nous allons cloner est hébergé sur GitHub. Pour cloner un dépôt, nous avons besoin de son URL. Vous pouvez trouver cette URL sur la page du dépôt GitHub, généralement sous un bouton "Code" ou "Clone".

![alt text](./images/url_github.png)

Utilisez la commande `git clone` suivie de l'URL du dépôt pour cloner le dépôt sur votre machine locale.

```bash
git clone https://github.com/nom-utilisateur/nom-depot.git
```
Remplacez `nom-utilisateur` et `nom-depot` par les informations appropriées du dépôt que vous souhaitez cloner.

L'utilisation du `.` à la fin de la commande est essentiel dans notre cas, car il indique à Git de cloner le dépôt dans le répertoire courant (`~/projets`), plutôt que de créer un nouveau sous-répertoire pour le dépôt.

### Vérifier le clonage
Une fois le clonage terminé, vous pouvez vérifier que le dépôt a été cloné correctement en listant le contenu du répertoire :

```bash
ls
```

