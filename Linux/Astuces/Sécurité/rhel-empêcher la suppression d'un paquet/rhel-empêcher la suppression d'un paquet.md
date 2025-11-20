---
title: RHEL - Empêcher la suppression d'un paquet
parent: Sécurité
grand_parent: Astuces
nav_order: 2
---

# Comment je protège un paquet (ex. dnsmasq) de la suppression sous Linux (Redhat) (tout en continuant à le mettre à jour) :

Quand vous travaillez sur un serveur Linux qui s’appuie sur un paquet comme `dnsmasq` pour fournir du DNS local, du DHCP ou du caching DNS, la dernière chose que vous souhaitez est qu’un administrateur distrait, ou un script automatisé un peu trop comment dirais-je, enthousiaste, désinstalle le paquet.

Dans cet article, je vous partage une approche fiable que j’utilise aujourd’hui dans les système Redhat10.

Il existe évidemment des méthodes agressives comme rendre des fichiers immuables ou créer des paquets factices, mais je voulais une solution élégante, intégrée dans l’écosystème DNF et facile à maintenir.

#### Pourquoi protéger dnsmasq ?

Quand dnsmasq disparaît, les symptômes sont immédiats :

- plus de DHCP,
- plus de cache DNS,
- plus de résolution locale,
- des conteneurs ou des appliances qui ne démarrent plus ou beuguent,
- le monitoring qui s'embale, et les arlertes qui pleuvent.

Une simple commande comme :
```bash
dnf remove dnsmasq
```
peut suffire à briser un environnement si elle est lancée par accident ou intégrée dans un script automatisé.

Je voulais donc une solution sûre, simple et surtout propre, qui ne casse pas le système et ne joue pas avec des fichiers sensibles.

#### La solution élégante : les protected_packages

DNF permet de définir une liste de paquets “protégés”.

Ces paquets :

- ne peuvent jamais être supprimés via dnf, yum, ou une transaction RPM,
- peuvent être mis à jour normalement,
- sont protégés dans toutes les opérations de maintenance.
- C’est un mécanisme utilisé par Red Hat pour éviter que les utilisateurs cassent leur système en supprimant des composants vitaux comme kernel, dnf, ou systemd.

Et la bonne nouvelle, c’est que je peux simplement y ajouter dnsmasq

#### Mise en place

J’édite le fichier :

```bash
nano /etc/dnf/dnf.conf
```

Puis, dans la section [main], j’ajoute :

```ini
protected_packages=dnsmasq
```

J’enregistre le fichier.
Et c’est tout : dnsmasq ne pourra plus jamais être supprimé par erreur.

#### Vérification

Pour vérifier que dnsmasq est bien protégé, j’essaie de le supprimer :

```bash
dnf remove dnsmasq
```

Je reçois un message d’erreur indiquant que le paquet est protégé et ne peut pas être supprimé.

```plaintext
root@localhost:~# dnf remove dnsmasq
Updating Subscription Management repositories.
Error: 
 Problem: The operation would result in removing the following protected packages: dnsmasq
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```

Pour vérifier que les mises à jour fonctionnent toujours :
```bash
dnf update dnsmasq
dnf upgrade dnsmasq
```

Ces commandes fonctionnent normalement, et dnsmasq est mis à jour si une nouvelle version est disponible.

#### Protéger dnsmasq contre les mise à jour :
Si vous souhaitez également empêcher les mises à jour de dnsmasq, vous pouvez utiliser la commande `dnf versionlock` :

```bash
dnf install 'dnf-command(versionlock)'
dnf versionlock add dnsmasq
```

Pour retirer le verrouillage de version plus tard, utilisez :

```bash
dnf versionlock delete dnsmasq
```

Il existe une autre méthode à base de `exclude` dans dnf.conf, que je n'ai pas testée ici.

#### Conclusion :

Protéger un paquet sans casser la gestion de paquets n’a pas besoin d’être compliqué. Grâce à protected_packages, vous pouvez garantir que votre paquet restera présent dans votre système, tout en continuant d’être mis à jour normalement.

C’est une approche claire, officielle et élégante, qui s’inscrit parfaitement dans les bonnes pratiques de Red Hat.

Si vous travaillez dans un environnement où les paquets sont sensibles, je vous recommande vraiment cette méthode. Elle m’a déjà épargné plus d’une mauvaise surprise.

##### Retirer la protection (si besoin)
```bash
sudo sed -i '/protected_packages/d' /etc/dnf/dnf.conf
```
