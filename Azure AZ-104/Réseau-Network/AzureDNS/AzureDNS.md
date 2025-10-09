---
title: Azure DNS
parent: Réseau-Network
grand_parent: Certification AZ-104
nav_order: 10
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
