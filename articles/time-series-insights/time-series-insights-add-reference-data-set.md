---
title: "Ajouter un jeu de données de référence à votre environnement Azure Time Series Insights | Microsoft Docs"
description: "Dans ce didacticiel, vous ajoutez un jeu de données à votre environnement Time Series Insights"
keywords: 
services: time-series-insights
documentationcenter: 
author: venkatgct
manager: almineev
editor: cgronlun
ms.assetid: 
ms.service: tsi
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/29/2017
ms.author: venkatja
ms.openlocfilehash: b94ca172dba71b407ee5e9a40c283a97602efd17
ms.sourcegitcommit: 6acb46cfc07f8fade42aff1e3f1c578aa9150c73
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2017
---
# <a name="create-a-reference-data-set-for-your-time-series-insights-environment-using-the-ibiza-portal"></a>Créer un jeu de données de référence pour votre environnement Time Series Insights à l’aide du portail Ibiza

Un jeu de données de référence est une collection d’éléments à laquelle s’ajoutent les événements issus de votre source d’événements. Le moteur d’entrée Time Series Insights joint un événement à partir de votre source d’événements à un élément dans votre jeu de données de référence. Cet événement ajouté est ensuite disponible pour la requête. Ce type de jointure repose sur les clés définies dans votre jeu de données de référence.

## <a name="steps-to-add-a-reference-data-set-to-your-environment"></a>Procédure d’ajout d’un jeu de données de référence à votre environnement

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur « Toutes les ressources » dans le menu de gauche du portail Azure.
3. Sélectionnez votre environnement Time Series Insights.

    ![Créer le jeu de données de référence Time Series Insights](media/add-reference-data-set/getstarted-create-reference-data-set-1.png)

4. Sélectionnez « Jeux de données de référence », puis cliquez sur « + Ajouter ».

    ![Créer le jeu de données de référence Time Series Insights - détails](media/add-reference-data-set/getstarted-create-reference-data-set-2.png)

5. Spécifiez le nom du jeu de données de référence.
6. Spécifiez le nom de la clé et son type. Ce nom et ce type sont utilisés pour récupérer la propriété appropriée à partir de l’événement dans votre source d’événement. Par exemple, si vous indiquez le nom de clé « DeviceId » et le type « String », le moteur d’entrée Time Series Insights recherche une propriété portant le nom « DeviceId » et de type « String » dans l’événement entrant. Vous pouvez indiquer plusieurs clés à joindre à l’événement. Le nom de la clé est sensible à la casse.

     ![Créer le jeu de données de référence Time Series Insights - détails](media/add-reference-data-set/getstarted-create-reference-data-set-3.png)

7. Cliquez sur « Créer ».

## <a name="next-steps"></a>Étapes suivantes

* [Gérez les données de référence](time-series-insights-manage-reference-data-csharp.md) par programme.
* Pour obtenir des informations de référence d’API complètes, consultez le document [API de données de référence](/rest/api/time-series-insights/time-series-insights-reference-reference-data-api).
