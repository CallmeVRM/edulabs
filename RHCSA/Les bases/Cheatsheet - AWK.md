---
title: Les bases
nav_order: 2
parent: Les bases
grand_parent: RHCSA
has_children: true
---

## Les bases - AWK

AWK est une commande puissante pour le traitement de texte et l'analyse de données. Voici quelques notions de base pour utiliser AWK efficacement.

### Syntaxe de base
```bash
awk 'pattern { action }' input_file
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


