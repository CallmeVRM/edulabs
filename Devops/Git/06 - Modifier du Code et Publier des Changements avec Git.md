---
title: 06 - Modifier du Code et Publier des Changements avec Git
parent: Git
grand_parent: Devops
nav_order: 6
nav_exclude: false
---

# 06 - Modifier du Code et Publier des Changements avec Git

- Installer et configurer Git (si ce n’est pas déjà fait) : [Voir Section 01 - Installer Git](./01%20-%20Installer%20Git.md)
- Mettez en place une clé SSH pour GitHub (si ce n’est pas déjà fait) : [Voir Section 04 - Installer une clé SSH sur votre compte Github](./04%20-%20Installer%20une%20cl%C3%A9%20SSH%20sur%20votre%20compte%20Github.md)


### Créer un fork d'un projet GitHub

Pour ma part je vais utiliser cette repository d'exemple : `https://github.com/vinta/awesome-python`

Comme on est dans un contexte collaboratif, la bonne pratique est de créer un fork du dépôt original dans son propre compte GitHub.

**La procédure** :

Après vous avoir connecté sur votre compte GitHub, allez y sur la page du dépôt et cliquez sur le bouton `Fork` en haut à droite.

![alt text](./images/fork.png "Fork d'un dépôt GitHub")

1. Donnez lui un nom explicite si vous le souhaitez, et une description.
2. Laissez cocher l'option `Copy the master branch only`
3. Cliquez sur le bouton `Create fork`

Vous avez maintenant une copie complète du dépôt dans votre compte GitHub.

### Cloner le dépôt forké en local
Revenons sur votre machine, et clonez le fork :

```bash
git clone git@github.com:username/awesome-python.git
```

Remplacez `username` par votre nom d'utilisateur GitHub.
Si c'est la première fois que vous utilisez Git avec GitHub en SSH, vous devrez peut-être valider l'empreinte de la clé SSH de GitHub. Tapez `yes` et appuyez sur Entrée.

Entrez dans le répertoire cloné :
```bash
cd awesome-python
```
Nous travaillons désormais dans une copie locale complète du projet, avec tout son historique.

### Créer une branche pour notre modification

Pour rester dans les bonnes pratiques, nous allons créer une branche dédiée à notre modification.

```bash
git checkout -b dev-doc-update
```
### Modifier le fichier `README.md`
Utilisez votre éditeur de texte préféré pour modifier le fichier `README.md` et y ajouter un commentaire ou une amélioration.

### Valider la modification avec un commit
Avant de procéder au commit, vérifions l'état de notre dépôt avec la commande `git status` pour s'assurer que le fichier modifié est bien pris en compte.

Une fois la modification effectuée, nous devons la valider à l’aide d’un commit.

On vérifie d'abord que nous sommes bien sur la branche `dev-doc-update` :

```bash
git branch
```

On ajoute le fichier modifié à l'index et on crée un commit :
```bash
git commit -a -m "Mise à jour de la documentation dans README.md"
```

Le paramètre -a permet à Git de prendre en compte automatiquement tous les fichiers suivis, ca évite de faire un `git add` avant le commit.

### Pousser la branche sur GitHub
Pour rendre notre branche accessible sur GitHub, nous devons la pousser vers le dépôt distant.

```bash
git push origin dev-doc-update
```

### Créer une Pull Request sur GitHub
Revenons sur GitHub dans la page de notre fork.

Vous verrez un bouton `Compare & pull request` pour notre branche récemment poussée.
![alt text](./images/06-PullRequestAsk.png "Créer une Pull Request")

Cliquez sur le bouton `Compare & pull request`.

Donnez un titre explicite à votre Pull Request, décrivez les modifications apportées, puis cliquez sur `Create pull request`.

Arrivé a cette étape vous devez être vigilants, il faut bien choisir vers quel dépôt vous souhaitez faire la Pull Request. 

Pour un projet open source forké, vous pouvez faire une PR vers le dépôt original (upstream) ou vers votre propre fork (head). L'objectif ici est de faire une PR vers le dépôt forké.

![alt text](./images/choix_pr_destination.png "Choisir la destination de la Pull Request")

Vous pouvez assigner des relecteurs si besoin, mais comme dans cette exemple vous êtes le seul contributeur, une fois la PR créee, vous pouvez la fusionner vous même en cliquant sur le bouton `Merge pull request`, en ajoutant un message explicite et clair. Confirmez le merge

Maintenant si vous allez sur l'onglet `Code` de votre dépôt forké, vous verrez que le fichier `README.md` a bien été mis à jour avec votre modification.

