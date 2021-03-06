---
title: Webhooks Azure Container Registry
description: "Découvrez comment utiliser des webhooks pour déclencher des événements quand certaines actions sont exécutées dans l’un des dépôts de votre registre."
services: container-registry
documentationcenter: 
author: neilpeterson
manager: timlt
editor: 
tags: acr, azure-container-registry
keywords: Docker, Conteneurs, ACR
ms.assetid: 
ms.service: container-registry
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/08/2017
ms.author: nepeters
ms.openlocfilehash: 5a9dab91aafb92f944b473f05144242143e36477
ms.sourcegitcommit: ccb84f6b1d445d88b9870041c84cebd64fbdbc72
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2017
---
# <a name="using-azure-container-registry-webhooks"></a>Utilisation de webhooks Azure Container Registry

Un registre Azure Container Registry stocke et gère des images conteneurs Docker privé, de la même manière qu’un Hub Docker stocke des images de Docker public. Utilisez des webhooks pour déclencher des événements quand certaines actions sont exécutées dans l’un des dépôts de votre registre. Les webhooks peuvent répondre à des événements au niveau du registre ou être réduits à une balise de dépôt spécifique.

Pour en savoir plus et connaître les concepts, consultez [À propos d’Azure Container Registry](./container-registry-intro.md).

## <a name="prerequisites"></a>Prérequis

- Registre managé de conteneurs Azure: créez un registre de conteneurs managé dans votre abonnement Azure. Par exemple, utilisez le [portail Azure](container-registry-get-started-portal.md) ou [Azure CLI](container-registry-get-started-azure-cli.md).
- Interface CLI Docker : pour configurer votre ordinateur local comme hôte Docker et accéder aux commandes de l’interface CLI Docker, installez le [moteur Docker](https://docs.docker.com/engine/installation/).

## <a name="create-webhook-azure-portal"></a>Créer un portail Azure webhook

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez au registre dans lequel vous souhaitez créer des webhooks.

2. Dans le panneau du conteneur, sélectionnez **Webhooks** sous **SERVICES**.

3. Dans la barre d’outils du panneau de webhook, sélectionnez **Ajouter**.

4. Remplissez le formulaire *Créer un webhook* avec les informations suivantes :

| Valeur | Description |
|---|---|
| Nom | Nom que vous voulez donner au webhook. Il peut contenir uniquement des caractères minuscules et des chiffres, et doit comporter de 5 à 50 caractères. |
| URI de service | URI auquel le webhook doit envoyer des notifications POST. |
| En-têtes personnalisés | En-têtes que vous souhaitez passer avec la demande POST. Ils doivent être au format « clé : valeur ». |
| Actions de déclencheur | Actions qui déclenchent le webhook. Actuellement, les webhooks peuvent être déclenchés par des actions d’envoi et/ou de suppression appliquées à une image. |
| État | État du webhook après sa création. Il est activé par défaut. |
| Étendue | Étendue à laquelle le webhook fonctionne. Par défaut, l’étendue couvre tous les événements dans le registre. Vous pouvez la spécifier pour un dépôt ou une balise en utilisant le format « dépôt:balise ». |

Exemple de formulaire webhook :

![Interface utilisateur de création de webhook ACR dans le portail Azure](./media/container-registry-webhook/webhook.png)

## <a name="create-webhook-azure-cli"></a>Créer un webhook avec Azure CLI

Pour créer un webhook à l’aide d’Azure CLI, utilisez la commande [az acr webhook create](/cli/azure/acr/webhook#create).

```azurecli-interactive
az acr webhook create --registry mycontainerregistry --name myacrwebhook01 --actions delete --uri http://webhookuri.com
```

## <a name="test-webhook"></a>Tester un webhook

### <a name="azure-portal"></a>Portail Azure

Avant d’utiliser le webhook sur des actions d’envoi ou de suppression appliquées à une image conteneur, vous pouvez le tester à l’aide du bouton **Ping**. La commande Ping envoie une demande POST générique au point de terminaison spécifié et enregistre la réponse. Cela peut vous aider à vérifier que vous avez correctement configuré le webhook.

1. Sélectionnez le webhook à tester.
2. Dans la barre d’outils supérieure, sélectionnez **Ping**.
3. Vérifiez la réponse du point de terminaison dans la colonne **État HTTP**.

![Interface utilisateur de création de webhook ACR dans le portail Azure](./media/container-registry-webhook/webhook-02.png)

### <a name="azure-cli"></a>Interface de ligne de commande Azure

Pour tester un webhook d’ACR avec Azure CLI, utilisez la commande [az acr webhook ping](/cli/azure/acr/webhook#ping).

```azurecli-interactive
az acr webhook ping --registry mycontainerregistry --name myacrwebhook01
```

Pour afficher les résultats, utilisez la commande [az acr webhook list-events](/cli/azure/acr/webhook#list-events).

```azurecli-interactive
az acr webhook list-events --registry mycontainerregistry08 --name myacrwebhook01
```

## <a name="delete-webhook"></a>Supprimer un webhook

### <a name="azure-portal"></a>Portail Azure

Vous pouvez supprimer chaque webhook en le sélectionnant, puis en appuyant sur le bouton **Supprimer** dans le portail Azure.

### <a name="azure-cli"></a>Interface de ligne de commande Azure

```azurecli-interactive
az acr webhook delete --registry mycontainerregistry --name myacrwebhook01
```
