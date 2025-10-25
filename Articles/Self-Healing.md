---
title: Concevoir des systèmes en mode Self Heal sur Azure
parent: Articles
nav_order: 2
has_children: true
exclude_from_nav: true
---

# Concevoir des systèmes en mode Self Heal sur Azure


### Introduction :

Le cloud représente l'avenir de la tech, et ce constat ne fait normalement plus débat. Toutefois, il ne constitue pas une potion magique à tous les maux rencontrés dans les environnements on-premises.
Ce que le cloud offre aujourd'hui, ce sont des briques technologiques modulables, que vous pouvez assembler intelligemment pour faire évoluer votre infrastructure. Parmis ses nombreux avantages, il permet certainement le scaling, la modularité, mais il permet aussi de réduire les astreintes, et cela permet à vos équipes de se concentrer sur des tâches à plus forte valeur ajoutée, plutôt que de gérer des incidents récurrents ou des opérations manuelles.

D’où l’importance de concevoir des systèmes résilients, capables de s’auto-réparer (self-healing) face aux aléas.

Au début des architectures distribués, on croyait naïvement que les pannes allait disparaître, ou du moins être fortement atténués. Mais avec l'évolution des infrastrucutres, et la montée en compléxité, on s'est vite rendu compte que ce n'était pas le cas. Certes, certains problèmesd on été résolus - par exemple, la haute disponibilité des bases de donnée - mais l'introduction de ces technologies a aussi engendré de nouveaux défis : latence inter-services, gestion des secrets, propagation des erreurs...

Une des réponses apportée aujourd'hui par Microsoft Azure, est la possibilité de mettre en place une architecture auto-réparatrice (self-healed) et celle ci vise trois objectis : détecter vite, répondre sans intervention humaine, observer pour s'améliore.

Ce principe de self healing est central dans le pillier Fiabilité du Well-Architected Framework. Le degré de self healing doit correspondre au plus près à votre SLO (Disponibilité/latence) et à vos exigences RTO/RPO.

### 1. Détecter
> On ne répare pas une panne dont ne connais pas l'existence.
La première clé pour mettre en place une architecture qui s'auto répare, est la rapidé de détection, chaque seconde compte (dans certains métiers on parle de millisconde), la propagation de l'erreur vers les API, les bases de données... Plusieurs équipes qui se mobilisent pour finalement un seul service qui fait effet dominos parcequ'il n'a pas était détecter à temps.

Pour detecter dans Azure, on peut s'appuyer sur :
- **Health endpoints applicatifs** : Implémentez /health (liveness) et /ready (readiness). Le premier confirme que le processus vit, le second que le service peut accepter du traffic (dépendences essentielles OK). C'est ce que Microsoft appelle le Health Endpoint Monitoring Pattern.

- **Sondes de la plateforme** : 
    - **App Service - Health Check & Auto-Heal** : Retirent automatiquement une instance du pool ou la recucles si l'URL de santé échoue ou si la latence/mémoire dépassent vos seuils.
    - **Azure Front Door** : sonde vos originesdepuis le réseau edge et ne route que les backends sains, les échecs de probes sont journalisé pour le diagnostic.
    - **AKS** : consomme vos liveness/readiness/startup probes pour piloter les redémarrages et la distribution du trafic via (ingress ou gateway).

*Exemple* : un site front sur App Service expose /ready. Si la DB répond à +500 ms, /ready passe à 503. Health Check éjecte l’instance, Auto-Heal la recycle. Front Door continue à servir depuis les autres instances saines. Résultat : panne confinée, pas d’astreinte.

