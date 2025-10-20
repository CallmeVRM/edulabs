---
title: Passkeys 101 pour ingénieurs cloud : comprendre, adopter, sécuriser
parent: Articles
nav_order: 10
has_children: true
nav_exclude: false
---

# Passkeys : comprendre, adopter, sécuriser
Je suppose qu'en 2025, personne n'est dupe (normalement): les mots de passe sont une source majeure de vulnérabilités et d'inconfort utilisateur. Même complexes, ils se réinitialisent, se volent, se cassent et se réutilisent. C'est donc un passif opérationnel et sécuritaire important pour les entreprises.


### Pourquoi les mots de passe ne tiennent plus la route aujourd'hui ?

- Secret réutilisable : un mot de passe connu fonctionne partout, sur n’importe quel service, ce qui peut créer des compromissions en chaîne.
- Coût IT : réinitialisations, politiques de complexité, gestion des compromissions.
- Bruteforce & fuites : bases hachées attaquées, réutilisation inter-services.

Et pour résoudre le problème on a accouché du MFA (Multi-Factor Authentication) qui ajoute une couche de sécurité supplémentaire, mais qui n'est pas infaillible non plus, les attaquants peuvent toujours trouver des moyens de contourner ces systèmes, avec un peu de social engineering on peut récupérer les codes SMS/push (MFA Fatigue) ou via du phishing pur en créant des URLs trompeuses (homographes) entre autres.

Alors face à cette réalité, les passkeys émergent comme une solution prometteuse pour renforcer la sécurité tout en améliorant l'expérience utilisateur (ce n'est toujours pas la solution miracle, mais c'est mieux), basés sur la cryptographie à clé (privée/publique), offrant ainsi une alternative robuste aux mots de passe traditionnels.

Dans cet article, nous allons explorer les fondamentaux des passkeys, leur fonctionnement, leurs avantages, ainsi que les meilleures pratiques pour les adopter et les sécuriser dans vos applications cloud.

### Passkeys : principe et garanties

En pratique, une passkey est conservée dans un espace matériel sécurisé : sur iOS et macOS dans la Secure Enclave, sur Android également dans la Secure Enclave, sous Windows Hello dans un TPM (Trusted Platform Module), et sous Linux via un TPM. Lorsqu’aucun support matériel n’est disponible, elle est stockée dans un coffre logiciel chiffré.

Les passkeys remplacent totalement les mots de passe tout en garantissant une authentification forte : comme elles sont liées à un appareil (TPM, clé USB, etc.), leur utilisation nécessite un déverrouillage par PIN ou biométrie. Elles combinent donc quelque chose que vous possédez avec quelque chose que vous savez ou êtes.

Une passkey est liée à la fois à un compte et à un appareil. Dans certains cas, elle peut être synchronisée entre plusieurs appareils, mais en principe, chaque passkey est propre à un appareil pour un compte donné. Il est donc possible de disposer de plusieurs passkeys pour un même compte, par exemple une sur l’ordinateur professionnel et une autre sur le téléphone.

Contrairement aux mots de passe qui doivent être régulièrement renouvelés, une passkey reste valide tant qu’elle n’est pas supprimée. Elle offre une forte résistance aux attaques :

- Phishing : l’authentification exige la proximité entre l’appareil authentificateur et le client, ce qui empêche toute tentative à distance.
- Homographes : une passkey ne fonctionne que sur le domaine exact pour lequel elle a été créée, rendant inopérantes les imitations d’URL.
- Compromission du service : si le serveur (relying party) est piraté, seules les clés publiques sont stockées, ce qui ne permet pas à un attaquant de les exploiter directement.

Bien sûr, les attaquants chercheront toujours de nouvelles méthodes, mais les passkeys neutralisent efficacement la plupart des vecteurs classiques liés aux mots de passe, ainsi qu’une partie des faiblesses de l’authentification multifacteur traditionnelle.

### Fonctionnement :
Voyons comment cela fonctionne. Deux protocoles se complètent sous la bannière FIDO2 (ce qu’on entend généralement par “passkeys”).

FIDO2 repose sur deux protocoles complémentaires : WebAuthn et CTAP. Ensemble, ils permettent de remplacer les mots de passe par des passkeys, offrant une authentification forte, résistante au phishing et beaucoup plus simple pour l’utilisateur.

WebAuthn est le protocole qui relie le service (site ou application) et le client (navigateur ou app).

Lors de la création d’une passkey (enrôlement), le service envoie un challenge au client. L’authenticator génère une paire de clés, signe le challenge avec la clé privée et renvoie la clé publique au service.

Lors de l’authentification, le service envoie un nouveau challenge. L’authenticator le signe après preuve de présence (PIN ou biométrie). Le service vérifie la signature avec la clé publique enregistrée : connexion réussie.


# Introduction
Les mots de passe ont atteint leurs limites : oubliés, réutilisés, volés, cassés. Même la MFA “classique” (codes SMS, push) reste vulnérable à l’ingénierie sociale. Les passkeys changent le modèle : on remplace un secret statique (secret partagé) par une preuve cryptographique locale liée à l’appareil et au domaine exact du service. Ce guide reprend fidèlement le contenu de la vidéo, en le rendant opérationnel pour un public cloud/DevOps.


### Pourquoi les mots de passe ne suffisent plus
- **Secret statique (secret partagé) :** Un mot de passe connu peut être réutilisé partout, sur n’importe quel système.
- **Coûts de gestion :** Les entreprises doivent réinitialiser les mots de passe oubliés ; en cas d’attaque, l’attaquant récupère des secrets et peut les exploiter.
- **Cassabilité :** La puissance de calcul qui ne fait qu'augmenter et l'arrivée de la quantique permettent d’attaquer des bases de mots de passe et de tester d’innombrables combinaisons.
- **Réutilisation humaine :** Les utilisateurs réemploient leurs mots de passe ; une compromission en entraîne d’autres.

La MFA améliore la posture (quelque chose que vous avez / êtes / savez) : ordinateur, téléphone, clé USB de sécurité ; biométrie (scan 3D du visage, empreinte, rétine) ; PIN. Le PIN semble plus simple, mais il est local à l’appareil (donc pas un secret partagé utilisable partout) et protégé par des mécanismes anti-bruteforce. Cependant, la MFA classique reste vulnérable au phishing (sites frauduleux) et aux attaques à distance (ingénierie sociale).


# Passkeys : définition et propriétés

Une passkey est un identifiant cryptographique basé sur une paire de clés :

- **Clé privée :** reste sur l’appareil de l’utilisateur (TPM, Secure Enclave ; sinon, coffre logiciel chiffré).
- **Clé publique :** stockée par le service (le relying party : Microsoft, Google, Amazon, etc.).

L’authentification consiste à signer un challenge avec la clé privée locale ; le service vérifie la signature avec la clé publique.

**Propriétés clés :**

- Remplacement complet du mot de passe, tout en gardant l’authentification forte : quelque chose que vous avez (appareil/clé) + quelque chose que vous savez/êtes (PIN/biométrie).

- Portée stricte au scope pré-défini : la passkey n’est utilisable qu'au sein d'un scope bien délimité, un nom de domaine exactement identique par exemple.

- Par compte et par appareil : on peut détenir plusieurs passkeys pour un même compte (une par appareil). Elles ne “tournent” pas automatiquement ; elles vivent jusqu’à suppression.

- Résistance au phishing et aux fuites : la clé privée ne quitte jamais l’appareil ; une base de clés publiques compromise est inutilisable seule.

- Proximité (selon le scénario) : impossible d’authentifier, à votre insu, une session lointaine.


“Passkey” est un mot simple pour un mécanisme sophistiqué : un identifiant cryptographique qui remplace le mot de passe. Derrière ce terme marketing se trouve une réalité technique précise : FIDO2. Quand on parle de passkeys, on parle en fait de FIDO2 (la norme), WebAuthn (le protocole côté navigateur/service) et CTAP (le dialogue avec l’authenticator, qu’il soit logiciel ou matériel). Sans FIDO2, pas de “passkey” au sens sécurité anti-phishing by design.

### Pourquoi évoquer FIDO lorsqu’on parle de passkeys

Parce que la promesse de la passkey —pas de secret statique, pas de hameçonnage, clé privée qui ne quitte jamais l’appareil— n’est vraie que si l’implémentation suit FIDO2. La norme apporte trois garanties structurantes :

- Liaison à l’origine (origin binding) : la passkey n’est utilisable que pour le domaine exact qui l’a enregistrée. Un site “presque identique” ne peut pas l’exploiter. C’est le cœur de la résistance au phishing via URL trompeuse.
- Preuve locale et possession réelle : l’usage de la clé privée impose une preuve de présence et une vérification utilisateur (PIN/biométrie) sur l’appareil qui détient la clé. Impossible de “relayer” à distance un défi d’authentification.
- Asymétrie et stockage sain : seule la clé publique est côté service ; une compromission du service ne livre rien d’exploitable pour se connecter ailleurs.

Autrement dit, la “passkey” n’est pas qu’un nouveau mot de passe plus sympa : c’est l’application des propriétés FIDO2 à l’expérience de connexion.

### Où se situent WebAuthn et CTAP dans ce tableau

WebAuthn est le “contrat cryptographique” entre le client (souvent un navigateur) et le service. Il définit comment un service enregistre une passkey (réception de la clé publique + preuve de possession) et comment il vérifie une connexion (challenge signé localement). Pas de recettes pas-à-pas : retenez que WebAuthn cadre qui parle à qui et quelles preuves doivent être fournies.

CTAP s’occupe du lien client ↔ authenticator quand l’authenticator n’est pas intégré au poste (smartphone, clé de sécurité USB/NFC/BLE, etc.). CTAP impose la proximité matérielle (USB/NFC/Bluetooth) et rend l’attaquant lointain inopérant, même s’il manipule l’utilisateur.

En combinant les deux, FIDO2 garantit que la bonne clé s’active au bon endroit pour le bon domaine, avec la bonne preuve.

**Clés matérielles (YubiKey & co.) :**

Les YubiKey et autres jetons FIDO2 jouent le rôle d’authenticators itinérants. Leur intérêt en contexte entreprise est double :

- Matériel dédié, surface d’attaque réduite : la clé privée reste dans le jeton ; l’accès nécessite preuve locale (PIN) et proximité physique. On évite les dérives des postes non maîtrisés.
- Cohérence des usages sensibles : beaucoup de clés supportent, en plus de FIDO2, PIV/Smart Card et OpenPGP. Même périphérique, multiples cas d’usage (connexion, signature, chiffrement), gouvernance unifiée.
- La passkey reste la même idée FIDO2 ; l’enveloppe matérielle ajoute une barrière opérationnelle très lisible pour l’utilisateur et pour l’audit.


### Quelques cas concrets pour étendre la confiance côté infra : passkeys + step-ca pour certificats éphémères

Les passkeys authentifient l’humain. Reste à distribuer de la confiance entre machines et outils d’accès :

Avec un CA privé comme step-ca, une authentification FIDO2 (WebAuthn dans le navigateur, YubiKey en preuve locale) peut conditionner l’émission de certificats courts (X.509 pour mTLS, certificats SSH).

On obtient des identités éphémères, attribuables et auditables : la preuve FIDO2 sert de porte d’entrée pour délivrer des clés d’usage jetables côté réseau et systèmes. Moins de clés longue durée, revocation-by-expiry, traçabilité propre.

Ce couplage relie l’identité humaine forte (passkey) et l’identité machine (certificats), sans fissure entre les couches

### Deux modèles coexistent, indépendants de la promesse FIDO2 :

- **Device-bound :** la clé privée ne sort pas de l’authenticator d’origine (Windows Hello for Business, clé matérielle FIDO2, certains modes de Microsoft Authenticator). Maîtrise forte de où résident les secrets ; l’accès sur un autre poste passe par la proximité (CTAP), pas par la copie.

- **Synced :** la passkey peut être synchronisée au sein d’un même écosystème (Apple↔Apple, Android↔Android) ou d’une même application. Meilleure récupération utilisateur, périmètre de diffusion plus large.

- **Grand public :** cap sur la récupération (synced). Entreprise : priorité au contrôle et à la réduction de surface d’attaque (device-bound), souvent avec clés matérielles pour standardiser l’expérience.

# Ce que change réellement l’approche passkeys/FIDO2

- **Anti-phishing par conception :** liaison stricte au domaine, preuve locale, proximité matérielle.
- **Réduction des secrets réutilisables :** moins de mots de passe à renouveler, moins de resets et de politiques acrobatiques.
- **Expérience sobre :** geste court (PIN/biométrie), souvent sans identifiant, y compris sur postes partagés via clés matérielles.
- **Chaîne de confiance continue :** en l’adossant à un CA (step-ca), la preuve FIDO2 irrigue mTLS, SSH et pipelines d’admin.

# Conclusion

Parler de passkeys sans parler de FIDO2 reviendrait à ignorer ce qui les rend sûres. La passkey est l’expression UX d’un cadre normatif (FIDO2) qui impose origin binding, preuve locale, asymétrie et proximité. Les YubiKey donnent à cette promesse un support matériel portable et gouvernable. En reliant le tout à un CA comme step-ca, la confiance s’étend jusqu’aux certificats éphémères qui sécurisent services et accès. Le résultat n’est ni un tutoriel, ni une simple MFA revisitée : c’est une architecture d’authentification et de distribution de confiance, conçue pour résister aux attaques courantes et pour tenir la distance côté opérations.