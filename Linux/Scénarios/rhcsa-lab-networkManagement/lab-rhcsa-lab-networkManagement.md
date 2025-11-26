---
title: RHCSA-NetworkManagement
parent: Scénarios
grand_parent: Linux
nav_order: 30
---

# RHCSA - Network Management Scénarios

#### Partie A : Audit et Découverte

- Affichez la liste de tous les périphériques (devices) réseau reconnus par le système.
```bash
nmcli device status
```
- Affichez la liste de tous les profils de connexion (connections) existants.
```bash
nmcli connection show
```
- Identifiez quel profil de connexion est actuellement actif sur votre interface principale.
```bash
nmcli con show --active
```
- Affichez toutes les informations détaillées (IP, DNS, Routes) de la connexion active.
```bash
nmcli con show <nom_de_la_connexion>
```
- Vérifiez l'état général deu service NetworkManager (est-il en cours d'exécution ?).
```bash
sudo systemctl status NetworkManager
```
Attention Case sensitive !

- Utilisez nmtui pour visualiser la configuration actuelle sans rien modifier (navigation simple).
```bash
nmtui
```

#### Partie B : Résolution d'Incidents

- Créez un nouveau profil de connexion nommé lab-static pour votre interface principale (ex: enp1s0 ou eth0).
```bash
sudo nmcli con add con-name lab-static ifname ens18 type ethernet
```
- Attribuez l'adresse IP 192.168.10.201/24 à ce profil.
```bash
sudo nmcli con modify lab-static ipv4.addresses 192.168.10.201/24
```
- Définissez la passerelle par défaut (gateway) à 192.168.10.254.
```bash
sudo nmcli con modify lab-static ipv4.gateway 192.168.10.254
```
- Définissez la méthode de configuration IPv4 sur manual (statique).
```bash
sudo nmcli con modify lab-static ipv4.method manual
```
- Configurez le serveur DNS primaire sur 8.8.8.8.
```bash
sudo nmcli con modify lab-static ipv4.dns "8.8.8.8"
```
- Ajoutez un serveur DNS secondaire 1.1.1.1.
```bash
sudo nmcli con modify lab-static ipv4.dns "8.8.8.8 1.1.1.1"
```
- Désactivez l'autoconfiguration IPv6 pour ce profil (méthode ignore ou disabled).
```bash
sudo nmcli con modify lab-static ipv6.method ignore
```
- Assurez-vous que ce profil ne démarre pas automatiquement au démarrage du système (autoconnect no).
```bash
sudo nmcli con modify lab-static connection.autoconnect no
```

#### Partie C : Gestion opérationnelle

- Activez (montez) le profil lab-static que vous venez de créer.
```bash
sudo nmcli con up lab-static
```
- Vérifiez avec la commande ip addr que l'IP a bien changé.
```bash
ip addr show <interface>
```
- Testez la connectivité (ping) vers la passerelle (si elle existe, sinon simulez).
```bash
ping -c 4 192.168.10.254
```

- Modifier le profile de connexion lab-static pour une nouvelle interface (ex: ens19), avec une nouvelle ip (192.168.30.201/24), et une nouvelle gateway 192.168.30.254.
```bash
sudo nmcli con modify lab-static ifname ens19 ipv4.addresses 192.168.30.201/24 ipv4.gateway 192.168.30.254
```
- Appliquer le nouveau profile sur l'interface ens19.
```bash
sudo nmcli con up lab-static
```
- Modifiez le profil lab-static pour ajouter une seconde adresse IP 10.19.0.5/24 sur la même interface ens19.
```bash
sudo nmcli con modify lab-static +ipv4.addresses 10.19.0.5/24
```
- Rechargez la connexion pour prendre en compte la nouvelle IP sans couper l'interface complètement (si possible), ou redémarrez la connexion.
```bash
sudo nmcli con reload lab-static
```
- Utilisez nmcli pour désactiver temporairement l'interface réseau (disconnect device).
```bash
sudo nmcli device disconnect ens19
```
- Réactivez l'interface réseau.
```bash
sudo nmcli device connect ens19
```

#### Partie D : Noms d'hôtes et Scénarios avancés

- Affichez le nom d'hôte actuel (hostname) du système.
```bash
hostnamectl status
```
- Changez le nom d'hôte statique pour `client-rhel10` en utilisant hostnamectl.
```bash
sudo hostnamectl set-hostname client-rhel10
```
- Ajoutez le domaine de recherche DNS lab.local au profil lab-static.
```bash
sudo nmcli con modify lab-static ipv4.dns-search lab.local
```
- Modifiez le profil lab-static pour qu'il se connecte automatiquement au démarrage (autoconnect yes).
```bash
sudo nmcli con modify lab-static connection.autoconnect yes
```

#### Partie E : Nettoyage et Retour à la normale

- Repassez sur votre profil d'origine (celui identifié à la question 3).
```bash
sudo nmcli con up <nom_de_la_connexion_origine>
```

- Vérifiez que vous avez récupéré votre ancienne IP (souvent DHCP).
```bash
ip a
```

- Supprimez définitivement le profil lab-static.
```bash
sudo nmcli con delete lab-static
```

- Vérifiez qu'il ne reste plus de trace de ce profil dans la liste des connexions.
```bash
nmcli connection show
```

- Redémarrez le service NetworkManager pour vous assurer que tout est propre et stable.
```bash
sudo systemctl restart NetworkManager
```

