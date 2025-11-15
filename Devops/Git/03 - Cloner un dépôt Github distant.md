---
title: 03 - Cloner un dépôt Github distant
parent: Git
grand_parent: Devops
nav_order: 3
nav_exclude: false
---

# 03 - Cloner un dépôt GitHub distant et préparer un dépôt partagé local

Dans ce guide, nous allons mettre en place un workflow collaboratif local avec Git, en utilisant un dépôt distant hébergé sur GitHub comme source initiale.

- **Un administrateur** récupère un dépôt GitHub et crée un dépôt central partagé sur un répertoire commun.

- **Les développeurs** clonent ce dépôt central vers leurs répertoires personnels et poussent leurs modifications vers ce dépôt partagé.

Pour que cette collaboration fonctionne, il est indispensable d’utiliser un dépôt Git bare du côté serveur, afin que Git accepte les push sans risque d’incohérence.

Dans mon exemple, j'ai deux utilisateurs : vrm (administrateur) et camel et alice (développeur) dans le group dev_proj_a.

## Coté Administrateur :

### Préparer le dépôt partagé
Nous allons créer un espace partagé dans `/srv/projets` destiné à accueillir le dépôt central du projet. Le dépôt partagé utilisé comme point central doit être un dépôt bare, c’est-à-dire un dépôt Git sans working directory.


```bash
sudo mkdir -p /srv/projets
```

```bash
git init --bare /srv/projets/proj_a.git
```

Mise en place des permissions :
```bash
sudo chown -R vrm:dev_proj_a /srv/projets
sudo chmod -R 2770 /srv/projets
sudo chmod -R +t /srv/projets
```

Dans cet exemple, tous les développeurs du projet A appartiennent au groupe dev_proj_a.
Nous ajustons donc les permissions pour garantir :

- un accès restreint
- une collaboration fluide
- une gestion cohérente des permissions grâce au bit setgid

### Importer un projet GitHub dans le dépôt central
Depuis son home, l’administrateur récupère le dépôt GitHub dans sa zone personnelle :

```bash
git clone https://github.com/nom-utilisateur/nom-depot.git mon-depot-src
cd mon-depot-src
```

À ce stade, vous êtes dans votre copie locale du projet GitHub :

```bash
/home/vrm/mon-depot-src
```

Dans ce répertoire, Git conserve :

- votre espace de travail (fichiers visibles)
- le dossier .git/ contenant l’historique
- les branches
- les commits locaux

Ce clone va être utilisé pour pousser son contenu dans le dépôt central.

Or dans cette configuration, vous êtes dans le répertoire local du projet, et cette copie pour l'instant pas encore liée au dépôt partagé localement, elle est connectée uniquement au dépôt distant github. Et vous pouvez vérifier cela avec la commande:

```bash
git remote -v
```
Vous voyez que l'URL du dépôt distant github est bien référencée.

```bash
origin  https://github.com/nom-utilisateur/nom-depot.git (fetch)
origin  https://github.com/nom-utilisateur/nom-depot.git (push)
```

Si vous modifiez un fichier et que vous faites un `push` maintenant, vous pousserez vos modifications directement sur github.

Or, nous voulons pousser non pas vers GitHub, mais vers notre dépôt local /srv/projets/proj_a.git.

Pour avoir une structure hybrid, on peut ne pas effacer l'origine github, mais en ajouter une nouvelle pour le dépôt local, d'abord on renomme l'origine github en "github" :

```bash
git remote rename origin github
```

Ensuite on ajoute le dépôt local comme nouvelle origine "origin" :
```bash
git remote remove origin
git remote add origin /srv/projets/proj_a.git
```

ainsi on garde la possibilité de faire un push ou pull vers github si besoin, et on aura deux remotes différentes.

Puis on pousse le code vers le dépôt bare (ma branche principale s'appelle main dans cet exemple) :
Vous pouvez vérifier le nom de votre branche principale avec la commande `git branch` avant de faire le push.

```bash
git push --all origin
git push --tags origin
```
Le dépôt central contient maintenant l’intégralité du projet.

On corriger le HEAD du bare si votre branche par défaut est main (ou master si votre projet utilise encore master):
```bash
cd /srv/projets/proj_a.git
git symbolic-ref HEAD refs/heads/main
```
Voilà coté administrateur c'est terminé pour l'instant, et on passe du coté du développeur.

### (Optionnel) Créer un working directory partagé
Si vous souhaitez disposer d’une copie lisible par tous (pour un outil interne, un serveur web ou une revue), vous pouvez générer un working directory séparé :
```bash
git clone /srv/projets/proj_a.git /home/admin/proj_a
```
Permissions :
```bash
chown -R root:dev_proj_a /srv/projets/proj_a
chmod -R 2770 /srv/projets/proj_a
```
Ce répertoire n'est pas un dépôt à utiliser pour travailler, mais une copie consultable.

## Coté Developpeur :
### Cloner le dépôt partagé en local vers son clone local
Chaque développeur doit travailler dans sa propre copie locale du dépôt.

Le développeur commence donc par créer un répertoire dédié dans le $HOME du développeur:

```bash
mkdir ~/proj_a
cd ~/proj_a
```
Ensuite, on déclare le dépôt partagé comme “safe directory” uniquement pour éviter les avertissements de sécurité Git :
```bash
git config --global --add safe.directory /srv/projets/proj_a.git
```
Puis on clone le dépôt bare :
```bash
git clone /srv/projets/proj_a .
```
Le `.` signifie que Git place le clone directement dans le répertoire courant.

À ce stade, le développeur peut modifier les fichiers du projet, par exemple :
```bash
nano README.md
```
Puis il enregistrez les modifications :
```bash
git add .
git commit -m "Premier commit local"
git push origin main
```

Chaque développeur travaille dans son propre espace local, en toute autonomie. Lorsqu’il pousse ses modifications, Git les enregistre dans le dépôt bare.
À ce stade, les changements sont bien présents dans le dépôt partagé… mais uniquement dans sa base de données Git interne.

Un dépôt bare n’a pas de copie directe des fichiers du projet :
- il stocke les commits, les arbres, les blobs, mais pas de working directory.
Autrement dit, le dépôt central connaît parfaitement les nouvelles versions, mais il ne possède aucun fichier lisible comme README.md ...

Git sépare donc clairement :

- le stockage interne des versions (dans le dépôt bare),
- et la mise à jour visible des fichiers (dans le working tree, via git pull).

Cette mécanique permet de travailler à plusieurs en conservant un dépôt central propre, fiable et adapté à la collaboration locale.

### Où sont les fichiers modifiés ?

Avec votre utilisateur faisant office de développeur vous avez :

- clone le projet depuis /srv/projets/mobile-app-quiz.git
- modifie README.md
- fait add → commit → push origin main


Et vous vous demandez :

- “Où est le fichier README.md que mon développeur a push ?”
- “Où sont les modifs ?”
- “Comment faire pour que les autres voient ces modifs ?”

**1. Où vit vraiment le fichier README.md ?**

Il y a deux lieux à distinguer :

- Le répertoire de travail de votre développeur
Par exemple : ~/mobile-app-quiz/README.md
Ça, c’est un vrai fichier sur le système de fichiers. Quand votre développeur édite le README, c’est ce fichier-là.

- Le dépôt Git (local ou bare)
Là, README.md n’existe pas comme fichier classique.
Il est stocké sous forme : d’un blob (contenu brut du fichier), référencé dans un tree (structure de dossier), pointé par une ref (la branche main par exemple).

C’est NORMAL que si vous faites un ls dans /srv/projets/mobile-app-quiz.git, vous ne voyez pas un fichier README.md :
Git “recompose” le contenu du projet à partir des objets internes uniquement dans un clone ou un checkout, pas dans un bare.

**Que fait exactement le push de camel ?**

Workflow logique :

- Camel modifie README.md dans son répertoire de travail.
- Il fait un add :
- Git prend le contenu de README.md, le met dans un blob, l’enregistre dans .git/objects.
- Il fait un commit :
- Git crée un objet commit qui pointe vers un tree (photo du projet), ce tree contient un chemin README.md → blob correspondant au fichier.
- Il fait un push vers /srv/projets/mobile-app-quiz.git :
    - Git envoie les nouveaux objets (blob, tree, commit) vers le dépôt bare, et met à jour la ref refs/heads/main dans ce dépôt.

### Conclusion :

Les modifs sont bien dans /srv/projets/mobile-app-quiz.git, mais sous forme d’objets Git, pas comme fichiers visibles. Si tu ne me crois pas, tu peux vérifier avec un troisième utilisateur qui clone le dépôt bare dans son propre répertoire de travail : il verra les modifs de camel.

Pour recevoir les mise à jour entre developpeurs, il faut faire un `git pull` dans un clone du dépôt bare.

