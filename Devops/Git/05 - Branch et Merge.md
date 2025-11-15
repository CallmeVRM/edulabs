---
title: 05 - Branch et Merge
parent: Git
grand_parent: Devops
nav_order: 5
nav_exclude: false
---

# 05 - Brancher et Fusionner du Code avec Git

Dans cette section, nous allons explorer une fonctionnalité des plus intéressante de Git : les branches (`branch`).

Lorsque nous travaillons sur un projet versionné avec Git, il est essentiel de pouvoir isoler une modification, expérimenter ou corriger un bug sans impacter la branche principale.

Les branches permettent de travailler sur une fonctionnalité, une correction ou une expérimentation sans toucher directement à la branche principale (souvent master ou main).
Une fois le travail terminé, les modifications sont fusionnées dans la branche principale.

Ce flux de travail est fondamental dans tous les environnements professionnels

Nous allons partir du principe que le projet est déjà initialisé en tant que dépôt Git.

### Créer une branche pour isoler une modification

Créer une branche consiste à créer un nouvel espace de travail parallèle.
Cela nous permet de travailler sereinement, sans altérer directement la branche principale (master ou main).

Créons une branche dédiée à la correction d'un fichier répondant sous le nom de `program.js` :

```bash
git checkout -b fix-program-js
```
Cette commande crée la branche `fix-program-js` et nous y place immédiatement.
Nous travaillons désormais dans un contexte isolé, ce qui est la bonne pratique pour toute modification ciblée.

### Corriger le fichier `program.js`

Imaginons qu’une faute s’y soit glissée. Modifions le fichier :
```bash
nano program.js
```
Nous corrigeons l’erreur, puis nous enregistrons.


### Valider la correction dans la branche
Une fois la modification effectuée, nous devons la valider à l’aide d’un commit.

On vérifie d'abord que nous sommes bien sur la branche `fix-program-js` :

```bash
git branch
```

On aura un retour du type :
```bash
* fix-program-js
  main
```

On vérifie l'état de notre git avec la commande `git status` pour s'assurer que le fichier modifié est bien pris en compte.

Ensuite, on ajoute le fichier corrigé à l'index et on crée un commit :
```bash
git commit -a -m "Correction de la faute dans program.js"
```
Le paramètre -a permet à Git de prendre en compte automatiquement tous les fichiers suivis, ca évite de faire un `git add` avant le commit.

À ce stade, la branche fix-readme contient notre correction, parfaitement isolée du reste du projet.

### Revenir sur la branche principale

Pour intégrer notre travail, il faut d’abord revenir sur la branche principale :
```bash
git checkout main
```
Git nous replace dans le contexte principal du projet. On peut vérifieer avec `git branch` que nous sommes bien sur main.

### Fusionner la branche de correction dans la branche principale
Pour intégrer la correction effectuée dans la branche `fix-program-js`, nous utilisons la commande de fusion (`merge`) :
```bash
git merge fix-program-js
```
Si personne d’autre n’a modifié le même fichier en parallèle, la fusion est automatique.
Le fichier contient maintenant la modification validée précédemment.

### Conclusion

Nous avons ainsi parcouru les bases du travail avec les branches :

- créer une branche pour isoler un développement,

- y effectuer des modifications,

- valider le travail,

- revenir sur la branche principale,

- fusionner le résultat final.

Ce flux de travail est au cœur de Git.
Il permet de maintenir un projet propre, structuré, et de collaborer efficacement, que vous soyez seul ou au sein d’une équipe.

Ce mécanisme étant fondamental, il servira de base à des pratiques plus avancées comme la gestion des conflits, le rebase ou les pull requests.