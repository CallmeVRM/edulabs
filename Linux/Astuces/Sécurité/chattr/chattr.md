---
title: J'utilise les attributs dès que je peux !
parent: Sécurité
grand_parent: Astuces
nav_order: 1
---

#### C'est quoi chattr :
`chattr` (pour change attribute) est une commande Linux qui permet de modifier les attributs "spéciaux" d’un fichier ou d’un répertoire, c'est un niveau plus bas que les simples permissions `rwx`.

En gros, là où `chmod` gère qui peut lire/écrire/exécuter, chattr peut imposer des comportements globaux que même root doit respecter… sauf s’il retire l’attribut.


##### Les usages les plus connus

Bien qu'il en existe une dizaine, je ne citerais que certains dans ce billet :

- +i (immutable) : Rend le fichier immuable : impossible d'écrire dedans, ou de le modifier, le renommer ou le supprimer. Aucun lien (hard link) ne peut être créé vers ce fichier, même le superutilisateur (root) ne peut pas effacer ou altérer son contenu tant que l’attribut est actif.
- +a (append only) : Autorise uniquement l’ajout à la fin du fichier (très pratique pour des logs).


##### Pourquoi j'utilise chattr dès que je peux ?
Sous Linux, les permissions ne sont pas un simple détail technique : elles constituent le garde‑fou qui protège vos fichiers, non seulement contre les acteurs malveillants, mais aussi contre vous‑même… ou contre ce collègue bien intentionné qui, par inadvertance, modifie le mauvais fichier.

Les droits classiques `(Lecture, Écriture, Exécution)` font partie de la première ligne de défense. Mais dans un environnement critique, ils ne suffisent pas : un utilisateur (ou script) disposant des droits sudo pourra tout de même modifier (override) un fichier protégé, même si ses permissions sont en mode `600`.

La persistance de certains malwares peut aussi être évitée en rendant un fichier immuable.

En pratique, cela empêche toute modification, même par un utilisateur disposant des droits root, sauf s'il retire explicitement cet attribut.

C’est une mesure simple mais redoutablement efficace : certains malware ne pourront pas écrire directement dans des fichiers critiques comme /etc/passwd ou /etc/ssh/sshd_config à votre insu.

###### Exemple :

Dès que je déploie un serveur, l’un des premiers fichiers que je configure est `/etc/ssh/sshd_config`. En général, j’utilise un template que je pousse via cloud-init ou Ansible. Une fois le fichier en place et les vérifications concluantes (tests de connexion, règles de sécurité validées), je le fige en appliquant un attribut d’immuabilité.

Cela empêche toute modification, volontaire ou accidentelle, même par root, tant que l’attribut n’est pas retiré.

```bash
sudo chattr +i /etc/ssh/sshd_config
```

Pour retirer cet attribut
```bash
sudo chattr -i /etc/ssh/sshd_config
```

chattr est un outil souvent méconnu, mais d’une efficacité redoutable pour renforcer la sécurité d’un système Linux. En ajoutant une couche de protection en dessous des permissions classiques, il permet de verrouiller des fichiers critiques et de réduire considérablement les risques de modification accidentelle ou malveillante.

Comme tout outil contraignant, il faut l'utilisé avec discernement, et toujours documenté pour éviter de se bloquer soi‑même. Il devient ainsi un allié précieux dans toute stratégie de durcissement (hardening) post‑déploiement. Que ce soit pour figer une configuration sensible, protéger des logs ou pour d'autres évènements, chattr mérite clairement sa place dans la boîte à outils de tout administrateur système.