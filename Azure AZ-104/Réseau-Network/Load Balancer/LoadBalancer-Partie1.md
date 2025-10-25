---
title: Load Balancer - P1
parent: Réseau-Network
grand_parent: Certification AZ-104
nav_order: 15
nav_exclude: true
---

# DNS dans Azure : services et fonctionnalités

Le Domain Name System (DNS) est le service essentiel qui traduit un nom de domaine (comme www.contoso.com) en adresse IP afin de permettre la communication entre les ressources réseau.

Dans Azure, cette fonction est assurée par Azure DNS, un service managé offrant hébergement, résolution et équilibrage DNS au sein de l’infrastructure mondiale de Microsoft.
Il permet d’administrer vos zones DNS avec la même sécurité, les mêmes outils et la même facturation que vos autres services Azure.


# Services DNS dans Azure

Il existe plusieurs **services et fonctionnalités DNS** disponibles dans Microsoft Azure :


### • **Azure DNS**  
Azure DNS est un **service managé** qui permet d’héberger vos domaines directement dans Azure. Il fournit des **serveurs de noms hautement disponibles** et offre la possibilité de **créer, gérer et résoudre** des enregistrements DNS (A, CNAME, MX, TXT, etc.) pour vos domaines.  

Vous pouvez y gérer vos enregistrements DNS (A, CNAME, MX, TXT, etc.) via le portail Azure, Azure CLI ou des API, avec la même gestion d’identité et de facturation que vos autres ressources Azure.
Les serveurs de noms d’Azure DNS sont distribués mondialement pour assurer rapidité, fiabilité et haute disponibilité.

Azure DNS prend en charge deux types de zones :  
- **Zones publiques** : permettent d’héberger vos domaines accessibles depuis Internet sans avoir à gérer de serveurs DNS externes.
Vous pouvez soit acheter un nom de domaine directement via App Service Domains (à ne pas confondre avec Active Directory Domain Services – ce sont deux services distincts),
soit transférer la gestion DNS d’un domaine déjà existant depuis votre registrar vers Azure DNS.
Ce service offre une gestion centralisée et sécurisée de vos enregistrements DNS publics tout en bénéficiant de la haute disponibilité et de la résilience de l’infrastructure Azure.
- **Zones privées** : assurent la résolution interne des noms dans vos réseaux virtuels (VNets) sans nécessiter de serveurs DNS personnalisés.
Les ressources Azure connectées au même réseau peuvent ainsi se découvrir automatiquement via des noms conviviaux plutôt que des adresses IP.
Ce service simplifie la gestion DNS interne tout en garantissant la sécurité et l’isolation du réseau. 

L’utilisation des zones privées DNS permet de gérer la résolution interne au sein de réseaux complexes sans déployer de serveur DNS.  
Ce service est **facturable** selon le nombre de zones et de requêtes DNS traitées.  
> Azure DNS est **couvert par la certification AZ-104**, qui inclut la création, la gestion et la liaison de zones publiques et privées.

---

### • **Azure Traffic Manager**  
Azure Traffic Manager est un **service DNS intelligent** utilisé pour la **gestion du trafic global**. Contrairement à Azure DNS, qui renvoie toujours la même réponse à une requête donnée, Traffic Manager peut fournir **des réponses dynamiques** selon plusieurs critères :  
- la **localisation géographique** de l’utilisateur,  
- la **latence réseau**,  
- la **disponibilité** des points de terminaison (endpoints).  

Il utilise DNS pour diriger les utilisateurs vers l’instance de service la plus performante ou la plus proche, que ce soit dans Azure, on-premises ou dans un autre cloud.  
Le service est **facturable** en fonction du nombre de requêtes traitées.  
> Bien que Traffic Manager fasse partie de l’écosystème DNS Azure, sa mise en œuvre n’est **pas couverte par l’examen AZ-104**.

---

### • **App Service Domains**  
Le service **App Service Domains** permet **l’achat et la gestion de noms de domaine** directement depuis le portail Azure. Ces domaines peuvent ensuite être hébergés dans **Azure DNS** pour la gestion des enregistrements.  

Ce service est **intégré à Azure App Service**, mais peut être utilisé pour tout type d’application, même en dehors de ce contexte.  
Il est **facturable** comme tout service d’enregistrement de domaine.  
> Ce service n’est **pas inclus dans le périmètre de l’examen AZ-104**, mais il est utile pour la mise en place rapide d’environnements web.

---

## 🔹 Fonctionnalités DNS intégrées à Azure

### • **Azure-provided DNS (DNS interne Azure)**  
Aussi appelé **DNS interne**, il est **fourni automatiquement** pour chaque réseau virtuel (VNet). Il permet aux **machines virtuelles** d’un même réseau de se découvrir et de communiquer entre elles à l’aide de leurs **noms d’hôte**.  

Les requêtes sont internes au VNet et ne sortent jamais de l’environnement Azure.  
Cette fonctionnalité n’est **pas facturable**, car elle fait partie du service de base du réseau virtuel.  

---

### • **DNS récursif (ou “Bring Your Own DNS”)**  
Azure offre par défaut un service de **résolution récursive** permettant à vos machines virtuelles de résoudre les noms publics sur Internet. Toutefois, vous pouvez choisir d’utiliser **votre propre serveur DNS** :  
- un **DNS on-premises** connecté via VPN,  
- un **contrôleur de domaine Active Directory**,  
- ou un **serveur DNS personnalisé** hébergé dans Azure.  

Cette approche, souvent appelée “**Bring Your Own DNS**”, permet d’intégrer Azure à des environnements DNS d’entreprise.  
Cette fonctionnalité n’est **pas facturable**, mais nécessite de maintenir vos propres serveurs DNS.  

---

### • **Reverse DNS (DNS inverse)**  
Le **Reverse DNS** permet la **résolution inverse** (IP → nom de domaine) en utilisant des enregistrements **PTR**. Cette fonctionnalité est disponible pour les **adresses IP publiques Azure** et peut aussi être utilisée dans des zones DNS internes.  

Les zones inverses (`in-addr.arpa` ou `ip6.arpa`) pour des blocs IP que vous possédez peuvent également être hébergées dans Azure DNS.  
Cette fonctionnalité n’est pas directement facturée, sauf si vous hébergez la zone PTR dans Azure DNS.  
> Le reverse DNS n’est **pas au programme de l’examen AZ-104**, mais reste utile pour les scénarios de messagerie et de vérification d’identité IP.

---

## 📘 Résumé

| Élément | Type | Facturation  | Description principale |
|:--|:--:|:--:|:--:|
| **Azure DNS** | Service | ✅ Oui | Hébergement DNS public et privé |
| **Azure Traffic Manager** | Service | ✅ Oui |Répartition DNS intelligente du trafic global |
| **App Service Domains** | Service | ✅ Oui | Achat et gestion de domaines dans Azure |
| **Azure-provided DNS** | Fonctionnalité | ❌ Non | DNS interne pour la résolution VM-à-VM |
| **DNS récursif / BYO DNS** | Fonctionnalité | ✅ Oui | Utilisation d’un DNS personnalisé ou d’entreprise |
| **Reverse DNS** | Fonctionnalité | ⚠️ Partiel | Résolution inverse IP → nom |

---
---
---

# 🌐 Gestion complète du DNS dans Azure

## 1. Création et délégation d’une zone DNS publique

### Comprendre le rôle d’une zone DNS
Une **zone DNS** dans Azure est une ressource qui héberge les enregistrements DNS d’un domaine. Lorsque vous créez une zone DNS dans **Azure DNS**, celle-ci se voit attribuer automatiquement **quatre serveurs de noms autoritatifs** (name servers) hébergés par Microsoft. Ces serveurs répondent aux requêtes DNS en fonction des enregistrements configurés dans la zone.

> 💡 **Remarque :** Vous pouvez créer une zone DNS sans posséder le nom de domaine correspondant. Cependant, pour la rendre accessible publiquement, une **délégation de domaine** depuis le registrar est nécessaire.

Il faut savoir que la zone DNS est par défaut Global, elle n'a donc pas de région spécifique de déploiement.

### Étapes de création d’une zone DNS publique
1. Créez une ressource **DNS Zone** dans Azure via le portail, Azure CLI ou PowerShell. 
2. Lors de la création de la zone, vous pouvez importer un fichier contenant les enregistrements de votre zone ou le rédiger manuellement. 
3. Azure attribue automatiquement quatre serveurs de noms (ex. `ns1-xx.azure-dns.com`, `ns2-xx.azure-dns.net`, etc.).  
4. Vérifiez les serveurs attribués dans le **portail Azure** ou à l’aide de la **CLI/PowerShell**. Chaque zone possède un ensemble de serveurs unique. `Azure Portal > VotreZoneDNS > Settings > Properties`
5. Configurez la délégation du domaine auprès de votre **registrar** (OVH, namecheap...):
   - Remplacez les enregistrements **NS** existants par ceux fournis par Azure DNS.  
   - N’utilisez **jamais** de glue records pointant directement vers les adresses IP des serveurs Azure DNS (ces IP peuvent changer).

> 🚫 Les **vanity name servers** (serveurs dont le nom correspond à votre domaine) ne sont pas pris en charge.

![alt text](image-1.png)

![alt text](image-7.png)

### Délégation d’une zone enfant
Azure traite les **zones enfants** comme des entités indépendantes. La délégation d’une sous-zone suit les mêmes étapes que pour une zone principale :
1. Créez la zone enfant dans Azure DNS.  
2. Identifiez les serveurs de noms associés à la zone enfant.  
3. Ajoutez des enregistrements **NS** dans la zone parente, pointant vers les serveurs de la zone enfant.  

> ⚠️ Si des enregistrements portant le même nom existent déjà dans la zone parente, ils deviendront inaccessibles une fois la délégation effectuée. Pensez à les **répliquer dans la zone enfant** avant la délégation.

---

## 2. Gestion des enregistrements DNS

Chaque enregistrement DNS dans Azure comprend :
- **Nom** : combiné avec le nom de la zone pour former le **FQDN** (ex. `www` dans `contoso.com` → `www.contoso.com`).  
- **Type** : détermine la nature des données (A, CNAME, MX, etc.).  
- **TTL (Time-To-Live)** : durée de mise en cache de l’enregistrement.  
- **RDATA** : contenu associé à l’enregistrement (adresse IP, nom de domaine, etc.).

Les enregistrements de même nom et type forment un **record set (RRSet)**. Chaque record set peut contenir jusqu’à **20 enregistrements individuels**. Pour le **root domain**, utilisez le symbole `@`.


### Création d’un enregistrement DNS
- Accédez à la zone DNS → DNS Management → **+ Record Set** → Add
- Renseignez les champs :
  - **Name** : `www`  
  - **Type** : A  
  - **Alias record set** : No  
  - **TTL** : 1 heure (modifiable)  
  - **IP Address** : ajoutez une ou plusieurs adresses IPv4  

### Exemple d’Alias Record au niveau du domaine racine
Pour un enregistrement au niveau du domaine (apex), utilisez :
- **Name** : `aliasdewww`  
- **Type** : A  
- **Alias Record Set** : Oui  
- **Azure resource** : vous avez le choix entre plusieurs options (Figure 1.2) ou  **Zone record set** : vous pouvez choisir un enregistrement dans la zone actuelle.
- **TTL** : 1 heure  

![alt text](image-3.png)
Figure 1.2

### Principaux types d’enregistrements pris en charge

| Type | Description | Remarques |
|------|--------------|------------|
| **A / AAAA** | Mappe un nom vers une adresse IPv4 / IPv6. | Standard. |
| **CAA** | Indique les autorités de certification autorisées. | Configurable via CLI/PowerShell uniquement. |
| **CNAME** | Alias d’un nom vers un autre nom DNS. | Non autorisé au niveau du zone apex. |
| **MX** | Configure le routage des emails. | — |
| **NS** | Indique les serveurs de noms d’une zone. | Créé automatiquement à la création de la zone. |
| **PTR** | Effectue une recherche DNS inversée. | Utilisé pour le reverse DNS. |
| **SOA** | Spécifie les informations d’autorité de la zone. | Créé et supprimé automatiquement. |
| **SRV** | Permet la découverte de services. | Format : `_service._protocol.domain.com`. |
| **TXT** | Utilisé pour du SPF, la validation de domaine, etc. | Le type SPF est obsolète (RFC7208). |

### 🔗 Alias Records (Intégration Azure)
Les **alias records** permettent de référencer directement des ressources Azure (Public IP, Traffic Manager, ou autre enregistrement de la même zone). Ces enregistrements :
- S’actualisent automatiquement si la ressource change d’adresse IP.  
- Évitent les enregistrements orphelins lorsque les ressources sont supprimées.  
- Permettent la résolution dynamique avec **Traffic Manager** sans CNAME.  

Un alias record est supporté dans Azure DNS pour les types A, AAAA et CNAME.

- Pour le type A / AAAA : pointez vers une IP publique Azure (ressource Public IP). 
- Pour le type A / AAAA / CNAME : pointez vers un profil Traffic Manager — c’est particulièrement utile pour la gestion du trafic ou pour pouvoir utiliser un alias au niveau du domaine racine (zone apex), ce que les CNAME classiques ne permettent pas. 

**D’autres possibilités d’alias :**

- vers un point de terminaison CDN Azure (utile pour héberger des sites statiques) 
- vers un autre record dans la même zone (référence interne) 
- vers un endpoint Front Door Azure pour personnaliser un domaine sur un front global.


Dans le DNS classique, un enregistrement *peut rester actif* même si la ressource qu’il cible (IP, CNAME, etc.) a été supprimée.

Résultat : le DNS pointe vers une adresse invalide — voire réattribuée — créant un risque d’erreur ou de détournement de trafic.

Les alias records d’Azure DNS évitent ce problème en liant automatiquement le cycle de vie du DNS à celui de la ressource Azure.
Si la ressource (Public IP, Traffic Manager, etc.) change ou est supprimée :

le DNS est mis à jour automatiquement, ou l’enregistrement devient vide, sans intervention manuelle.

Cela garantit une résolution propre, sans enregistrements orphelins, ni risque de redirection accidentelle. ✅ 

> Les Alias Records est une extension non définie dans les RFC DNS classiques. Il se peut qu'en dehors d’Azure (chez d'autres registars), le comportement équivalent doit être simulé manuellement.

---

## 3. Configuration des paramètres DNS, Built-in et personnalisation :

### 🔧 Azure-provided DNS
Par défaut, chaque machine virtuelle dans un **Virtual Network (VNet)** reçoit via **DHCP** :
- Une **adresse IP privée**, et  
- Des **paramètres DNS** utilisant le service DNS interne d’Azure (`168.63.129.16`).

Ce service offre :
- Une **résolution interne** VM-à-VM dans le même réseau virtuel.  
- Une **résolution Internet** pour les domaines publics.  

Les noms internes suivent le format :  
`<nom_VM>.<suffixe_aléatoire_dns>.internal.cloudapp.net`  
Ces noms ne sont résolus qu’à l’intérieur du réseau virtuel.

![alt text](image-4.png)
Figure 1.3

Ces deux vm on étaient déployer sans aucune configuration personnalisé du DNS.

```bash
azure@vm01:~$ ping vm01
PING vm01.pncdymwalfue3izxlcvqxqceth.bx.internal.cloudapp.net (10.0.0.4) 56(84) bytes of data.
64 bytes from vm01.internal.cloudapp.net (10.0.0.4): icmp_seq=1 ttl=64 time=0.018 ms
64 bytes from vm01.internal.cloudapp.net (10.0.0.4): icmp_seq=2 ttl=64 time=0.020 ms
64 bytes from vm01.internal.cloudapp.net (10.0.0.4): icmp_seq=3 ttl=64 time=0.019 ms

azure@vm01:~$ ping vm02
PING vm02.pncdymwalfue3izxlcvqxqceth.bx.internal.cloudapp.net (10.0.0.5) 56(84) bytes of data.
64 bytes from vm02.internal.cloudapp.net (10.0.0.5): icmp_seq=1 ttl=64 time=1.34 ms
64 bytes from vm02.internal.cloudapp.net (10.0.0.5): icmp_seq=2 ttl=64 time=2.38 ms
64 bytes from vm02.internal.cloudapp.net (10.0.0.5): icmp_seq=3 ttl=64 time=0.930 ms
```

### 🔧 Le principe du Bring Your Own DNS (BYODNS)
Vous pouvez remplacer le DNS fourni par Azure par vos **propres serveurs DNS**, hébergés :
- dans Azure (sur une VM),  
- sur site (on-premises), ou  
- sur Internet (ex. `8.8.8.8`).

Cela permet :
- la **résolution inter-VNet**,  
- la **résolution hybride Azure/On-prem**,  
- la **résolution inversée (PTR)**,  
- la **prise en charge d’Active Directory**.

> 🔒 Les serveurs DNS personnalisés doivent offrir un service **récursif**, sinon la résolution Internet échouera.

### 🔧 Configuration des DNS personnalisés
On peut modifier l'implémentation du DNS à plusieurs niveaux, par exemple :
- **Niveau VNet** : applique les DNS à toutes les VMs du réseau.  
- **Niveau interface réseau (NIC)** : par défaut, la carte réseau (NIC) hérite du DNS du VNet, quand on le passe sur le mode custom, il devient prioritaire sur le VNet.  
  > Si plusieurs VMs sont dans un *availability set*, les DNS configurés sur leurs interfaces sont fusionnés.

**Au niveau du vnet** :

![alt text](image-5.png)


**Au niveau de l'interface réseau (NIC)** :

![alt text](image-6.png)


> ⚠️ Il n’est **pas possible** de définir des DNS au niveau du **sous-réseau (subnet)**.

Après modification :
- Les VMs doivent **redémarrer** pour appliquer les nouveaux paramètres DNS.  
- Azure redémarre automatiquement les VMs concernées lors de la mise à jour au niveau de la NIC.

---

## 5. Zones DNS privées (Private DNS Zones)

En plus des zones DNS publics, Azure prend en charge les **zones DNS privées**, permettant la résolution de noms **au sein des réseaux virtuels Azure**.

### Avantages :
- Utiliser vos **propres noms de domaine** internes (sans suffixe Azure).  
- Bénéficier d’une **inscription automatique** des VMs dans la zone (pour le VNet enregistré).  
- Éviter la gestion manuelle de serveurs DNS internes avec toute sa complexité.

### Fonctionnement :
- Une zone privée peut être **liée à plusieurs VNets** :  
  - Le **VNet d’enregistrement** enregistre automatiquement ses VMs.  
  - Les autres VNets (appelés **VNets de résolution**) doivent être liés manuellement.  
- Les requêtes inter-VNet ne nécessitent pas de peering ou de VPN.

### Création via le portail Azure :
1. Recherchez **Private DNS Zones**.  
2. Cliquez sur **Créer**.  
3. Saisissez le **nom de domaine** (ex. `corp.local`) et le **groupe de ressources**.  
4. Une fois la zone créée, ajoutez un **Virtual Network Link** pour associer vos VNets.  

---

## 🧭 En résumé

Azure DNS offre une **plateforme complète et managée** pour :
- héberger vos zones publiques et privées,  
- déléguer des domaines depuis vos registrars,  
- configurer des enregistrements avancés (alias, SRV, CAA...),  
- et intégrer vos environnements hybrides (Azure / On-premises) sans infrastructure DNS dédiée.

> Avec ces fonctionnalités, Azure DNS constitue la **colonne vertébrale de la résolution de noms** dans les environnements cloud modernes et hybrides.