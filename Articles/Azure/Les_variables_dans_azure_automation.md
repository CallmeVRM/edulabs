---
title: Les variables dans Azure Automation
parent: Azure
grand_parent: Articles
nav_order: 11
has_children: true

---

# Les variables dans Azure Automation

## Introduction
Quand on parle d'Azure Automation, on pense souvent aux runbooks, à Powershell, aux workflows, mais beaucoup moins aux variables. Et pourtant, elles font partie de ces petits détails qui rendent une automation fluide, claire et agréable à maintenir.

C'est un peu comme un carnet de notes partagé entre vos scripts. Au lieu de réecrire la même information partout, vous la centralisez, et tout devient plus simple.

## Pourquoi s'en servir ?
Dans la vraie vie, les environnements changent, le nom d'un groupe de ressource, une clé API, un compte de stockage. Si vous avez hardcodé ces informations (nom du compte de stockage, nom du groupe de ressource...) dans vos runbooks, dans plusieurs endroits, eh bien le jour du changement ce n'est pas la chasse au oeuf que vous allez faire mais la chasse au texte. OK c'est plus facile aujourd'hui avec l'IA, mais pourquoi faire compliqué quand on peut faire plus simple (KISS protocol).

Les variables servent justement a vos sauver des minutes voir des heures  le jour J. On stock l'information dans une variable, et on fait appel a cette variable quand on a besoin, ainsi, le jour du changement, on aura que la ligne de la variable a changer, et le code se mettra à jour instatanement.

C'est plus propre, et surtout c'est plus élégant.

## Le principe :
Une variable, est une valeur stockée dans votre compte Azure Automation.
Elle peut être de type : 
- Texte (string)
- nombre (integer)
- booléen (true/false)
- date
- null (vide)

On peut créer les variables depuis le Portail Azure, ou via du Powershell/CLI, Bicep/ARM, ou même avec du Terraform.

## La sécurité dans tout ca ?
La question qui vient directement à l'esprit est : la variable est chiffré ou pas ?

Alors Azure propose deux types de variables :
- Non chiffrées : leur contenu est visible dans le portail et dans PowerShell.
- Chiffré : leur valeur est masquée et stockée de façon sécurisée.

Petite subtilité : une fois la variable créée, vous ne pouvez plus changer son statut de chiffrement. Si vous l'avez créée en clair, elle le restera.

## Créer une variable

### Depuis le **portail Azure**
- Ouvrez votre **Automation Account**
- Rendez-vous dans **Ressources partagées → Variables**
- Cliquez sur **Ajouter une variable**
- Choisissez le nom, le type, la valeur, et s’il faut la chiffrer
- Enregistrez.

Et voilà. C’est tout.
Une fois enregistrée, la variable devient immédiatement accessible depuis vos runbooks.

### En utilisant PowerShell

Pour créer une variable sans chiffrement :
```Powershell
New-AzAutomationVariable `
  -ResourceGroupName "nextcloud-prod" `
  -AutomationAccountName "OpsAccount" `
  -Name "DefaultRegion" `
  -Value "eastus" `
  -Encrypted $false
```

Pour créer une variable avec chiffrement :
```Powershell
New-AzAutomationVariable `
  -ResourceGroupName "nextcloud-prod" `
  -AutomationAccountName "OpsAccount" `
  -Name "DefLinuxAdminPassword" `
  -Value "MotdepasseFort123!" `
  -Encrypted $true
```

### Lire une variable dans un runbook

Dans un runbook PowerShell, la lecture est très simple :
```PowerShell
$region = Get-AutomationVariable -Name "DefaultRegion"
$password = Get-AutomationVariable -Name "AdminPassword"
``` 

La belle particularité, c’est que même les variables chiffrées peuvent être lues à l’intérieur du runbook — puisque celui-ci s’exécute avec les permissions nécessaires.

C’est ce qui vous permet de garder vos secrets sécurisés tout en automatisant sans contrainte.

### Modifier ou supprimer

Besoin de mettre à jour la valeur ?