---
title: Cheatsheet - AWK
description: Référence rapide des commandes et options AWK pour le traitement de texte et l'analyse de données.
nav_order: 2
parent: Les bases
grand_parent: RHCSA
has_children: true
---

## AWK : Le Cheatsheet de Référence

AWK est une commande puissante pour le traitement de texte et l'analyse de données. Voici quelques notions de base pour utiliser AWK efficacement.

### Syntaxe de base
```bash
awk [options] -f fichier_script.awk fichier_input
awk [options] 'BEGIN { init } /pattern/ { action } END { final }' fichier_input
```

### Variables Prédéfinies (Système & Champs)

Ces variables définissent comment awk lit et écrit les données.

```bash
Variable,Description,Défaut
$0,La ligne (enregistrement) complète en cours.,
$1...$N,Les champs individuels (colonnes).,
FS,Input Field Separator. Délimiteur de lecture.,Espace
OFS,Output Field Separator. Délimiteur d'écriture.,Espace
RS,Input Record Separator. Délimiteur de ligne lue.,\n
ORS,Output Record Separator. Délimiteur de ligne écrite.,\n
NF,Number of Fields. Nombre de champs dans la ligne actuelle.,
NR,Number of Records. Numéro de ligne total (cumulé).,
FNR,File Number of Records. Numéro de ligne dans le fichier actuel.,
FILENAME,Nom du fichier en cours de traitement.,
ARGC,Nombre d'arguments passés à la ligne de commande.,
ARGV,Tableau des arguments de la ligne de commande (ARGV[0] est awk).,
ENVIRON,Tableau associatif contenant les variables d'environnement système.,"ex: ENVIRON[""HOME""]"
RSTART,Index de début trouvé par la fonction match().,
RLENGTH,Longueur de la chaîne trouvée par la fonction match().,
```


### Formatage de Sortie (printf) :
Similaire au C, printf permet un contrôle total (alignement, précision) contrairement à print.

Syntaxe : printf "Format", item1, item2

```bash
Spécificateur,Type de donnée,Exemple,Résultat
%s,Chaîne de caractères (String),"printf ""[%s]"", ""bonjour""",[bonjour]
%d ou %i,Entier (Integer),"printf ""%d"", 12.5",12
%f,Nombre flottant,"printf ""%.2f"", 10/3",3.33
%e,Notation scientifique,"printf ""%e"", 1000",1.000000e+03
%c,Caractère ASCII,"printf ""%c"", 65",A
%%,Affiche le symbole % littéral,"printf ""%d%%"", 50",50%
```


**Modificateurs (Padding & Alignement) :**

- %10s : Alignement à droite (largeur 10 espaces).
- %-10s : Alignement à gauche (largeur 10 espaces).
- %05d : Remplit avec des zéros (ex: 00123).

### Fonctions de Chaînes (String Functions)

Indispensables pour manipuler le texte à l'intérieur des colonnes.

```bash
Fonction,Description
"index(str, sub)",Retourne la position de sub dans str (0 si non trouvé).
length(str),Retourne la longueur de la chaîne.
"substr(str, start, len)",Extrait len caractères depuis start (commence à 1).
"split(str, arr, sep)",Découpe str dans le tableau arr selon le séparateur sep. Retourne la taille.
tolower(str),Convertit en minuscules.
toupper(str),Convertit en majuscules.
"match(str, regex)",Teste si str contient regex. Définit RSTART et RLENGTH.
"sub(regex, rep, str)",Remplace la première occurrence. Modifie str directement.
"gsub(regex, rep, str)",Remplace toutes les occurrences. Modifie str directement.
"sprintf(fmt, ...)",Comme printf mais retourne la chaîne au lieu de l'afficher.
```

### Contrôle de Flux (Flow Control)


**Conditions :**

```bash
if (condition) { ... } else if (cond2) { ... } else { ... }
# Ternaire :
var = (x > 5) ? "Grand" : "Petit"
```

**Boucles :**

```bash
# While
while (i < 10) { i++; print i }

# Do-While
do { print i; i++ } while (i < 10)

# For (style C)
for (i = 0; i < 10; i++) { print i }

# For (style Tableau)
for (key in array) { ... }
```

**Instructions de saut :**

    break : Sort de la boucle.
    continue : Passe à l'itération suivante de la boucle.
    next : Spécifique AWK. Arrête le traitement de la ligne courante et passe à la ligne suivante du fichier d'entrée.
    exit : Arrête le script (exécute le bloc END s'il existe, sinon quitte).

### Opérateurs

- **Arithmétique** : +, -, *, /, % (modulo), ^ (puissance).

- **Incrément** : ++, -- (préfixe ou suffixe).

- **Assignation** : =, +=, -=, *=, /=, %=, ^=.
- **Relationnel** : ==, !=, >, <, >=, <=.

- **Matching (Regex)** :

    - ~ : Correspond à la regex ($1 ~ /^root/).
    - !~ : Ne correspond PAS à la regex.

- **Logique** : && (AND), || (OR), ! (NOT).



### Exemples :

- Afficher le nombre de champs (colonnes) dans chaque ligne :
```bash
awk '{print NF}' file.txt
```

- Afficher la première et la dernière colonne d'un fichier :
```bash
awk '{print $1, $NF}' file.txt
```

- Afficher le numéro des lignes et leur contenu :
```bash
awk '{print NR, $0}' file.txt
```

- Afficher les lignes 2 à 5 uniquement :
```bash
awk 'NR>=2 && NR<=5' file.txt
```


















`$NF` représente toujours le dernier champ, quel que soit le nombre de colonnes.
`NR` est le numéro de la ligne courante.

Le séparateur de champs par défaut est l'espace ou la tabulation. Pour définir un autre séparateur, utilisez l'option `-F`.

```bash
# Afficher uniquement la première colonne   
awk '{print $1}' file.txt

# Dupliquer une colonne (Afficher Col 1 deux fois)
awk '{print $1, $1, $2}' file.txt

# Réorganiser totalement une sortie CSV
awk -F ',' '{print $3, $1, $2}' data.csv

# Concaténer deux colonnes en une seule avec un underscore entre les deux
awk '{print $1"_"$2, $3}' file.txt

# Afficher les 3 dernières lignes d'un fichier en supprimant la 3ème colonne
awk -F: '{$AF; $3=""; print}' /etc/passwd | tail -n 3

# Afficher la première et la dernière colonne
awk -F: '{print $1, $NF}' /etc/passwd

# Afficher les lignes 5 à 10 uniquement
awk -F: 'NR>=5 && NR<=10' /etc/passwd

# Afficher les lignes paires uniquement (Utilise le modulo (reste de la division). Pour les impaires : NR % 2 == 1)
awk -F: 'NR % 2 == 0' /etc/passwd

# Ajouter des numéros de ligne devant chaque ligne (similaire à cat -n)
awk -F: '{print NR, $0}' /etc/passwd

# Afficher une ligne spécifique (ex: ligne 42) - commence par 1 et non par 0
awk -F: 'NR == 42' /etc/passwd

# Convertir un CSV en fichier séparé par des tabulations (TSV)
awk -F ',' 'BEGIN {OFS="\t"} {$1=$1; print}' data.csv



```















# Afficher le numéro de colonne avec son contenu
```bash
awk -F: '{for (i=1; i<=NF; i++) print "Colonne " i ": " $i}' /etc/passwd
```
- `-F:` → définit : comme séparateur de champs.
- `NF` → nombre de champs dans la ligne.
- Boucle `for` → parcourt chaque champ.
    - `i=1` → on commence au premier champ.
    - `i<=NF` → on continue tant que i est inférieur ou égal à NF.
    - `i++` → on incrémente i à chaque itération.
- `printf` → affiche le numéro de colonne et sa valeur.


### Afficher le nombre de champs et le numéro de ligne correspond :
Dans un cas ou j'ai un fichier qui doit contenir un certain nombre de champs (colonne) par ligne, et je veux vérifier si toutes les lignes respectent ce format. Ici, je vérifie le fichier `/var/log/audit/audit.log` pour voir combien de champs chaque ligne contient et si je trouve une qui ne correspond pas (ex. 16696) je vérifie avec son numéro de ligne.

```bash
awk -F: '{print NF, NR}' /var/log/audit/audit.log
awk -F: 'NR == 16696' /var/log/audit/audit.log

```


