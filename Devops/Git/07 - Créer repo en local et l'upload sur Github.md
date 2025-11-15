---
title: 07 - Créer un Dépôt Git Local et le Publier sur GitHub (SSH & HTTPS)
parent: Git
grand_parent: Devops
nav_order: 7
nav_exclude: false
---

# 07 - Créer un Dépôt Git Local et le Publier sur GitHub (SSH & HTTPS)

Dans cette section nous allons voir comment créer un dépôt Git local à partir de zéro, puis comment l’envoyer sur GitHub via deux méthodes possibles : **HTTPS** et **SSH**.

L’objectif est d’obtenir un workflow propre, capable de supporter le travail collaboratif autant que les projets personnels.

---

### Créer un répertoire local pour votre projet

Commençons par créer un projet appelé `my-webapp` dans votre répertoire personnel :

```bash
mkdir ~/my-webapp
cd ~/my-webapp
```

Créons un fichier simple, par exemple un README :

```bash
echo "# My Web App" > README.md
```

**Configurer Git (si pas encore fait)**
```bash
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@example.com"
```

### Initialiser un dépôt Git local
Initialisons un dépôt Git dans ce répertoire :

```bash
git init
```

Vérifions que le dépôt a été créé :

```bash
ls -la
```

Vérifions l’état du dépôt :

```bash
git status
```

### Ajouter et committer vos premiers fichiers
Ajoutons les fichiers à l’index :
```bash
git add .
```

Faisons notre premier commit :
```bash
git commit -m "Initial commit"
```

Votre dépôt local est maintenant opérationnel. Cependant le dépôt n’est pas encore lié à un dépôt distant sur GitHub, vous pouvez vérifier cela avec :

```bash
git remote -v
```

## Publier votre dépôt sur GitHub

Nous allons maintenant l’envoyer sur GitHub pour le rendre accessible partout.

Deux méthodes sont possibles :

- HTTPS : simple, fonctionne partout
- SSH : plus sécurisé et évite de saisir un mot de passe

Vous pouvez utiliser l’une ou l’autre selon vos préférences.

### Méthode HTTPS – La plus simple

**Créer un dépôt vide sur GitHub**
1. Connectez-vous sur GitHub
2. Cliquez sur New repository
3. Donnez-lui un nom, par exemple my-webapp, et une description optionnelle
4. Ne pas ajouter de Readme, ni de .gitignore
5. Choisir privé comme visibilité
5. Cliquez sur Create repository

GitHub vous propose maintenant plusieurs commandes. Nous allons utiliser celles adaptées à notre situation.

**Renommez la branche principale en "main"**
```bash
git branch -M main
```

**Déclarez le dépôt distant en utilisant l’URL HTTPS fournie par GitHub**
```bash
git remote add origin https://github.com/edulabs-fr/my-webapp.git
```
Maintenant si vous vérifiez les dépôts distants `git remote -v`, vous devriez avoir un retour.

**Enfin, poussez vos modifications vers GitHub avec la commande suivante**
```bash
git push -u origin main
```
Il vous demandera de vous authentifier avec votre nom d’utilisateur et mot de passe GitHub (ou token si vous avez activé l’authentification à deux facteurs).

Votre dépôt est maintenant en ligne.

Chaque fois que vous ferez un push, ou un pull, c'est le dépôt distant (GitHub) que vous utiliserez.

### Méthode SSH – Plus sécurisé
Si vous préférez une authentification sans mot de passe, passez par SSH.

Assurez-vous que votre clé SSH est installée. voir [Créer une clé SSH et l’ajouter à GitHub](./06%20-%20Configurer%20Git%20avec%20une%20clé%20SSH.md).

Pour vérifié que votre clé SSH fonctionne avec GitHub, utilisez la commande suivante :

```bash
ssh -T git@github.com
```

On suppose que vous partez d'un dossier vierge, donc on reprend depuis la création du répertoire et l’initialisation du dépôt Git local.

### Créer un répertoire local pour votre projet
Commençons par créer un projet appelé `my-webapp` dans votre répertoire personnel :

```bash
mkdir ~/my-webapp
cd ~/my-webapp
```

Créons un fichier simple, par exemple un README :

```bash
echo "# My Web App" > README.md
```

**Configurer Git (si pas encore fait)**
```bash
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@example.com"
```

### Initialiser un dépôt Git local
Initialisons un dépôt Git dans ce répertoire :

```bash
git init
```

Vérifions que le dépôt a été créé :

```bash
ls -la
```

Vérifions l’état du dépôt :

```bash
git status
```

### Ajouter et committer vos premiers fichiers
Ajoutons les fichiers à l’index :
```bash
git add .
```

Faisons notre premier commit :
```bash
git commit -m "Initial commit"
```

Votre dépôt local est maintenant opérationnel. Cependant le dépôt n’est pas encore lié à un dépôt distant sur GitHub, vous pouvez vérifier cela avec :

```bash
git remote -v
```

**Même procédure que pour HTTPS :**
1. Connectez-vous sur GitHub
2. Cliquez sur New repository
3. Donnez-lui un nom, par exemple my-webapp, et une description optionnelle
4. Ne pas ajouter de Readme, ni de .gitignore
5. Choisir privé comme visibilité
5. Cliquez sur Create repository


**Ajouter le dépôt distant (SSH)**
```bash
git remote add origin git@github.com:username/my-webapp.git
```

**Renommez la branche principale en "main"**
```bash
git branch -M main
```

**Enfin, poussez vos modifications vers GitHub avec la commande suivante**
```bash
git push -u origin main
```

Si c'est la première fois que vous utilisez SSH avec GitHub, vous devrez peut-être accepter l’empreinte de la clé, ainsi que la passphrase de votre clé SSH si vous en avez défini une.

Votre dépôt est maintenant en ligne via SSH sur la branch main.



### Conclusion
Vous avez maintenant un dépôt Git local fonctionnel, ainsi qu’un dépôt distant sur GitHub accessible via HTTPS ou SSH. Vous pouvez désormais collaborer avec d’autres développeurs ou simplement gérer vos projets personnels de manière efficace. N’oubliez pas de régulièrement faire des commits et de pousser vos modifications pour garder votre dépôt à jour !


### Pour les plus courageux :

Créer une nouvelle branch, faire des modifications, les committer, puis faire un pull request sur GitHub. Merge la branche une fois le PR validé.

Créer une nouvelle branche en local :
```bash
git branch dev
```

Utiliser la nouvelle branche :
```bash
git checkout dev
```

Faire des modifications sur le `README.md`, les ajouter, puis les committer :
```bash
git commit -a -m "Description des modifications"
```

Pousser la branche vers GitHub :
```bash
git push -u origin dev
```

Sur la page GitHub de votre dépôt, vous verrez qu'une nouvelle branche a été poussée. Cliquez sur "Compare & pull request", ajoutez une description, puis cliquez sur "Create pull request".

Si vous laissez le dépôt ainsi, vous aurez deux branches : `main` et `dev`, si vous faites des modifications en local, les fichiers modifiés seront sur la branche `dev` uniquement, ils ne seront pas visible sur la branche `main` tant que vous n’aurez pas fait le merge.


Après révision, vous pouvez fusionner la branch `dev` avec la branche `main` en cliquant sur `Merge pull request` pour fusionner les modifications dans la branche principale.