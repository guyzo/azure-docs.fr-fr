---
title: "Didacticiel Kubernertes sur Azure - Préparer une application | Microsoft Docs"
description: "Didacticiel ACS - Préparer une application"
services: container-service
documentationcenter: 
author: neilpeterson
manager: timlt
editor: 
tags: aks, azure-container-service
keywords: Docker, conteneurs, micro-services, Kubernetes, DC/OS, Azure
ms.assetid: 
ms.service: container-service
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/24/2017
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: db8169b1c3c30efa1b684e49e1f113bee6a7fd45
ms.sourcegitcommit: c5eeb0c950a0ba35d0b0953f5d88d3be57960180
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2017
---
# <a name="prepare-application-for-azure-container-service-aks"></a>Préparer une application pour Azure Container Service (ACS)

Dans ce didacticiel (premier d’une série de huit), vous allez préparer une application à plusieurs conteneurs à son utilisation dans Kubernetes. Les étapes terminées sont les suivantes :  

> [!div class="checklist"]
> * Clonage de la source de l’application à partir de GitHub  
> * Création d’une image conteneur à partir de la source de l’application
> * Test de l’application dans un environnement Docker local

Une fois ces étapes effectuées, l’application suivante est accessible dans votre environnement de développement local.

![Image du cluster Kubernetes sur Azure](./media/container-service-tutorial-kubernetes-prepare-app/azure-vote.png)

Dans les didacticiels suivants, l’image conteneur est chargée dans Azure Container Registry, puis exécutée dans un cluster ACS.

## <a name="before-you-begin"></a>Avant de commencer

Ce didacticiel suppose une compréhension élémentaire des concepts Docker principaux tels que les conteneurs, les images de conteneur et les commandes Docker de base. Si besoin, consultez [Bien démarrer avec Docker]( https://docs.docker.com/get-started/) pour apprendre les principes de base des conteneurs. 

Pour terminer ce didacticiel, il vous faut un environnement de développement Docker. Docker fournit des packages qui le configurent facilement sur n’importe quel système [Mac](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) ou [Linux](https://docs.docker.com/engine/installation/#supported-platforms).

Azure Cloud Shell n’inclut pas les composants Docker requis pour effectuer chaque étape de ce didacticiel. Par conséquent, nous recommandons d’utiliser un environnement de développement Docker complet.

## <a name="get-application-code"></a>Obtenir le code d’application

L’exemple d’application utilisé dans ce didacticiel est une application de votes de base. L’application est constituée d’un composant web frontal et d’une instance Redis principale. Le composant web est empaqueté dans une image conteneur personnalisée. L’instance Redis utilise une image non modifiée de Docker Hub.  

Utilisez git pour télécharger une copie de l’application dans votre environnement de développement.

```console
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

Modifiez les répertoires de manière à travailler à partir des répertoires clonés.

```console
cd azure-voting-app-redis
```

Dans le répertoire se trouvent le code source de l’application, un fichier Docker Compose précréé et un fichier manifeste Kubernetes. Ces fichiers sont utilisés tout au long de ce didacticiel. 

## <a name="create-container-images"></a>Créer des images de conteneur

[Docker Compose](https://docs.docker.com/compose/) peut être utilisé pour automatiser la génération à partir des images conteneur, ainsi que le déploiement des applications à plusieurs conteneurs.

Exécutez le fichier `docker-compose.yml` pour créer l’image conteneur, téléchargez l’image Redis, puis démarrez l’application.

```console
docker-compose up -d
```

Une fois terminé, utilisez la commande [docker images](https://docs.docker.com/engine/reference/commandline/images/) pour afficher les images créées.

```console
docker images
```

Notez que les trois images ont été téléchargées ou créées. L’image `azure-vote-front` contient l’application et utilise l’image `nginx-flask` comme base. L’image `redis` est utilisée pour démarrer une instance Redis.

```
REPOSITORY                   TAG        IMAGE ID            CREATED             SIZE
azure-vote-front             latest     9cc914e25834        40 seconds ago      694MB
redis                        latest     a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask   flask      788ca94b2313        9 months ago        694MB
```

Exécutez la commande [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) pour voir les conteneurs en cours d’exécution.

```console
docker ps
```

Output:

```
CONTAINER ID        IMAGE             COMMAND                  CREATED             STATUS              PORTS                           NAMES
82411933e8f9        azure-vote-front  "/usr/bin/supervisord"   57 seconds ago      Up 30 seconds       443/tcp, 0.0.0.0:8080->80/tcp   azure-vote-front
b68fed4b66b6        redis             "docker-entrypoint..."   57 seconds ago      Up 30 seconds       0.0.0.0:6379->6379/tcp          azure-vote-back
```

## <a name="test-application-locally"></a>Tester l’application localement

Accédez à http://localhost:8080 pour afficher le conteneur en cours d’exécution.

![Image du cluster Kubernetes sur Azure](./media/container-service-tutorial-kubernetes-prepare-app/azure-vote.png)

## <a name="clean-up-resources"></a>Supprimer des ressources

Maintenant que la fonctionnalité de l’application a été validée, les conteneurs en cours d’exécution peuvent être arrêtés et supprimés. Ne supprimez pas les images de conteneur. L’image `azure-vote-front` est chargée sur une instance Azure Container Registry dans le didacticiel suivant.

Exécutez le code suivant pour arrêter les conteneurs en cours d’exécution.

```console
docker-compose stop
```

Supprimez les conteneurs et les ressources arrêtés avec la commande suivante.

```console
docker-compose down
```

Une fois terminé, vous disposez de deux images conteneur contenant l’application Azure Vote.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, une application a été testée et les images de conteneur créées pour l’application. Les étapes suivantes ont été effectuées :

> [!div class="checklist"]
> * Le clonage de la source de l’application à partir de GitHub  
> * La création d’une image conteneur à partir de la source de l’application
> * Le test de l’application dans un environnement Docker local

Passez au didacticiel suivant pour en savoir plus sur le stockage d’images de conteneur dans Azure Container Registry.

> [!div class="nextstepaction"]
> [Envoyer des images à Azure Container Registry](./tutorial-kubernetes-prepare-acr.md)
