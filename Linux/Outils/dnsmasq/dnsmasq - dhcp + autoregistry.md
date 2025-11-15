---
title: dnsmasq
parent: Outils
grand_parent: Linux
nav_order: 50
---

# dnsmasq

### Introduction
Dnsmasq est un logiciel léger qui fournit des services de serveur DNS, DHCP...


### Installation
Pour installer dnsmasq sur une distribution basée sur Debian/Ubuntu, utilisez la commande suivante :

```bash
sudo apt-get update
sudo apt-get install dnsmasq
```

## Configuration DHCP classique




## Configuration DHCP avec filtrage MAC




## Configuration DNS avec auto registry

Configuration du cache DNS :
`/etc/dnsmasq.conf`

```conf
# Activer et dimensionner le cache DNS
cache-size=1000

# Éviter les fuites DNS et les requêtes internes mal formées
domain-needed
bogus-priv

# Étendre automatiquement les noms courts avec le domaine local
expand-hosts

# Interface sur laquelle dnsmasq écoute
interface=eth0

# Domaine interne pour les entrées DNS et DHCP
domain=pxmx.edulabs.local

# Taille du cache DNS
cache-size=256
```

### Explication des options utilisées
- `cache-size=256` : Définit la taille du cache DNS à 256 entrées.
- `domain-needed` : Ce paramètre bloque la résolution des noms incomplets. Une requête pour `serveur` reste locale et n’est jamais envoyée aux résolveurs externes. En revanche, `serveur.exemple.com` peut être résolue via les upstream DNS. C’est un garde-fou pour éviter des fuites vers le FAI ou le DNS public.
- `expand-hosts` : Cette option permet d’ajouter automatiquement le domaine spécifié (ici `domain=px.edulabs.local`) aux noms d’hôtes locaux. Par exemple, si vous avez une entrée DHCP pour un hôte nommé `serveur`, avec cette option, dnsmasq le résoudra en `serveur.px.edulabs.local`.
- `bogus-priv` : Cette option empêche dnsmasq de répondre aux requêtes pour les adresses IP privées (comme 192.168.x.x, 10.x.x.x, etc.) qui ne sont pas dans le réseau local. Cela aide à éviter les fuites DNS vers l’extérieur pour les adresses privées.
- `interface=eth0` : Spécifie l’interface réseau sur laquelle dnsmasq doit écouter les requêtes DNS et DHCP. Remplacez `eth0` par l’interface appropriée de votre système. Sur des hôtes avec plusieurs interfaces, dont un accès WAN et LAN, cela permet de restreindre dnsmasq à l’interface LAN uniquement.
- `domain=px.edulabs.local` : Définit le domaine local pour les hôtes gérés par dnsmasq. Cela permet à dnsmasq de compléter automatiquement les noms d’hôtes locaux avec ce domaine.




