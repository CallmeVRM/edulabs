---
title: RHCSA-NetworkManagement
parent: Scénarios
grand_parent: Linux
nav_order: 30
---

# RHCSA - Network Management Scénarios

#### Partie A : Audit et Découverte

- Affichez la liste de tous les périphériques (devices) réseau reconnus par le système.

nmcli device status

- Affichez la liste de tous les profils de connexion (connections) existants.

nmcli connection show

- Identifiez quel profil de connexion est actuellement actif sur votre interface principale.

nmcli con show --active

- Affichez toutes les informations détaillées (IP, DNS, Routes) de la connexion active.

nmcli con show <nom_de_la_connexion>

- Vérifiez l'état général deu service NetworkManager (est-il en cours d'exécution ?).

sudo systemctl status NetworkManager

Attention Case sensitive !

- Utilisez nmtui pour visualiser la configuration actuelle sans rien modifier (navigation simple).

nmtui


#### Partie B : Résolution d'Incidents

- Créez un nouveau profil de connexion nommé lab-static pour votre interface principale (ex: enp1s0 ou eth0).

sudo nmcli con add con-name lab-static ifname ens18 type ethernet

- Attribuez l'adresse IP 192.168.10.201/24 à ce profil.

sudo nmcli con modify lab-static ipv4.addresses 192.168.10.201/24

- Définissez la passerelle par défaut (gateway) à 192.168.10.254.

sudo nmcli con modify lab-static ipv4.gateway 192.168.10.254

- Définissez la méthode de configuration IPv4 sur manual (statique).

sudo nmcli con modify lab-static ipv4.method manual

- Configurez le serveur DNS primaire sur 8.8.8.8.

sudo nmcli con modify lab-static ipv4.dns "8.8.8.8"

- Ajoutez un serveur DNS secondaire 1.1.1.1.

sudo nmcli con modify lab-static ipv4.dns "8.8.8.8 1.1.1.1"

- Désactivez l'autoconfiguration IPv6 pour ce profil (méthode ignore ou disabled).

sudo nmcli con modify lab-static ipv6.method ignore

- Assurez-vous que ce profil ne démarre pas automatiquement au démarrage du système (autoconnect no).

sudo nmcli con modify lab-static connection.autoconnect no

Partie C : Gestion opérationnelle

- Activez (montez) le profil lab-static que vous venez de créer.

sudo nmcli con up lab-static

- Vérifiez avec la commande ip addr que l'IP a bien changé.

ip addr show <interface>

- Testez la connectivité (ping) vers la passerelle (si elle existe, sinon simulez).

ping -c 4 192.168.10.254


- Modifier le profile de connexion lab-static pour une nouvelle interface (ex: ens19), avec une nouvelle ip (192.168.30.201/24), et une nouvelle gateway 192.168.30.254.

sudo nmcli con modify lab-static ifname ens19 ipv4.addresses 192.168.30.201/24 ipv4.gateway 192.168.30.254

- Appliquer le nouveau profile sur l'interface ens19.

sudo nmcli con up lab-static

- Modifiez le profil lab-static pour ajouter une seconde adresse IP 10.0.0.5/24 sur la même interface ens19.

sudo nmcli con modify lab-static +ipv4.addresses 10.0.0.5/24

- Rechargez la connexion pour prendre en compte la nouvelle IP sans couper l'interface complètement (si possible), ou redémarrez la connexion.

- Utilisez nmcli pour désactiver temporairement l'interface réseau (disconnect device).

- Réactivez l'interface réseau.