---
title: dnsmasq
parent: Outils
grand_parent: Linux
nav_order: 50
---

# dnsmasq

### Introduction
Dnsmasq est un logiciel léger qui fournit des services de serveur DNS, DHCP...

Dans cet article, nous allons mettre en place dnsmasq pour fournir des serv


### Installation
Pour installer dnsmasq sur une distribution basée sur Debian/Ubuntu, utilisez la commande suivante :

```bash
sudo apt update
sudo apt install dnsmasq
```

Si vous avez systemd-resolved actif (Ubuntu récent), assurez vous qu’il ne vient pas interférer sur le port 53.

**Ce que nous allons configurer :**
    DHCP classique
    DHCP avec réservations MAC
    DNS avec auto-enregistrement des clients DHCP
    Enregistrements DNS manuels
    Le tout limité à une seule interface.



### Partir avec un dnsmasq propre :

On va tout mettre dans /etc/dnsmasq.conf + un fichier pour les hosts manuels dans /etc/dnsmasq.d/ :

```bash
mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak
touch /etc/dnsmasq.conf
mkdir -p /etc/dnsmasq-hosts
```

### Fichier `/etc/dnsmasq.conf` complet (DHCP + DNS sur 1 interface)

Editez le fichier `/etc/dnsmasq.conf` 

```bash
sudo nano /etc/dnsmasq.conf
```

Collez y la configuration suivante :

```conf
########################################
#  dnsmasq - DHCP + DNS pour Proxmox  #
#  Réseau : 192.168.20.0/24           #
########################################

# --- Limiter dnsmasq à l’interface LAN des VMs ---
# Interface dans le conteneur (branchée sur le bridge Proxmox)
interface=eth2

# Se binder strictement à cette interface
bind-interfaces

# N’écouter que sur l’adresse IP locale de cette interface (optionnel mais propre)
listen-address=192.168.20.200

# --- Domaine interne ---
# Domaine du LAN
domain=px.edulabs.local

# Ajoute automatiquement le domaine aux noms courts venant de /etc/hosts et addn-hosts
expand-hosts

# Ne pas forwarder les noms sans point vers l’upstream DNS
domain-needed

# Ne jamais forwarder les IP privées (RFC1918) vers l’upstream DNS
bogus-priv

# Taille du cache DNS
cache-size=512

# --- Serveurs DNS upstream (optionnel : sinon /etc/resolv.conf est utilisé) ---
# Par exemple Cloudflare + Google
server=1.1.1.1
server=8.8.8.8

# --- DHCPv4 : plage dynamique ---
# Réseau : 192.168.20.0/24
# Plage dynamique pour les VMs "non critiques"
# Format : dhcp-range=<start>,<end>,<netmask>,<lease-time>
dhcp-range=192.168.20.50,192.168.20.200,255.255.255.0,12h

# Passerelle par défaut (gateway du LAN)
# Option 3 = router
dhcp-option=3,192.168.20.254

# DNS envoyé aux clients (on se met soi-même)
# Option 6 = DNS server
dhcp-option=6,192.168.20.200

# Domaine envoyé côté DHCP
dhcp-option=15,px.edulabs.local

# --- DHCP : réservations MAC -> IP + hostname ---
# VM critique 1
# MAC de la VM, IP fixe, hostname, durée du bail
dhcp-host=AA:AA:AA:AA:AA:01,192.168.20.10,vm1,24h

# VM critique 2
dhcp-host=AA:AA:AA:AA:AA:02,192.168.20.11,db1,24h

# VM d’admin
dhcp-host=AA:AA:AA:AA:AA:03,192.168.20.12,admin1,24h

# Remarque :
# - Ces IP (10,11,12) sont EN DEHORS de la plage dynamique (50-200),
#   mais dans le même subnet : c’est un pattern classique.

# --- Fichier d’entrées DNS manuelles supplémentaires ---
# Format type /etc/hosts
addn-hosts=/etc/dnsmasq-hosts/local-hosts

# Exemple :
#   192.168.50.5   nas
#   192.168.50.6   backup
#
# Avec expand-hosts + domain, ça deviendra :
#   nas.pxmx.edulabs.local
#   backup.pxmx.edulabs.local

# --- Logs (optionnel, utile pour debug) ---
log-queries
log-dhcp
#log-facility=/var/log/dnsmasq.log
```

### Fichier d’entrées DNS manuelles : /etc/dnsmasq-hosts/local-hosts
On crée ce fichier pour les enregistrements non-DHCP (NAS, routeurs, services divers, etc.) :

```bash
sudo nano /etc/dnsmasq-hosts/local-hosts
```

voici son contenu, que pouvez adapter selon vos besoins :

```conf
# Enregistrements DNS manuels supplémentaires
192.168.20.2   nas
192.168.20.3   backup
192.168.20.4   registry
```

Grâce à `expand-hosts` + `domain=px.edulabs.local`, dnsmasq vous répondra :

- `nas.px.edulabs.local` → 192.168.20.2
- `backup.px.edulabs.local` → 192.168.20.3
- `registry.px.edulabs.local` → 192.168.20.4

### Vérifier la conf, démarrer et activer dnsmasq

Vérifiez la syntaxe de votre fichier de configuration :

```bash
sudo dnsmasq --test
```

Si tout est OK, vous aurez le message `dnsmasq: syntax check OK.`, redémarrez le service dnsmasq :

```bash
sudo systemctl restart dnsmasq
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
domain=px.edulabs.local
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




