---
title: "Copier des données de Table web à l’aide d’Azure Data Factory | Microsoft Docs"
description: "Découvrez le connecteur Table web d’Azure Data Factory, qui vous permet de copier des données d’une Table web vers des banques de données prises en charge par Data Factory en tant que récepteurs."
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: spelluru
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/18/2017
ms.author: jingwang
ms.openlocfilehash: fba3de916fc3fb0b83b300cc2cf78ef556b35556
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2017
---
# <a name="copy-data-from-web-table-by-using-azure-data-factory"></a>Copier des données de Table web à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [Version 1 - Disponibilité générale](v1/data-factory-web-table-connector.md)
> * [Version 2 - Préversion](connector-web-table.md)

Cet article décrit comment utiliser l’activité de copie dans Azure Data Factory pour copier des données d’une base de données de Table web. Il s’appuie sur l’article [présentant une vue d’ensemble de l’activité de copie](copy-activity-overview.md).


> [!NOTE]
> Cet article s’applique à la version 2 de Data Factory, actuellement en préversion. Si vous utilisez la version 1 du service Data Factory, qui est en Disponibilité générale, consultez [Connecteur Table web dans V1](v1/data-factory-web-table-connector.md).

## <a name="supported-scenarios"></a>Scénarios pris en charge

Vous pouvez copier des données d’une base de données de Table web vers toute banque de données réceptrice prise en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur Table web prend en charge l’**extraction de contenu de table d’une page HTML**. Pour récupérer des données à partir d’un point de terminaison HTTP/S, utilisez plutôt le [Connecteur HTTP](connector-http.md).

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec l’activité de copie à l’aide du SDK .NET, du SDK Python, d’Azure PowerShell, de l’API REST ou du modèle Azure Resource Manager. Consultez le [Didacticiel de l’activité de copie](create-self-hosted-integration-runtime.md) pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie.

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques du connecteur Table web.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Table web sont les suivantes :

| Propriété | Description | Requis |
|:--- |:--- |:--- |
| type | La propriété de type doit être définie sur **Web** |Oui |
| url | URL de la source web |Oui |
| authenticationType | Valeur autorisée : **Anonyme**. |Oui |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion à la banque de données. Vous pouvez utiliser runtime d’intégration Azure ou un runtime d’intégration auto-hébergé (si votre banque de données se trouve dans un réseau privé). À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. |Non |

**Exemple :**

```json
{
    "name": "WebLinkedService",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "url" : "https://en.wikipedia.org/wiki/",
            "authenticationType": "Anonymous"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les jeux de données. Cette section fournit la liste des propriétés prises en charge par le jeu de données de l’objet Table web.

Pour copier des données de Table web, affectez la valeur **RelationalTable** à la propriété type du jeu de données. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Requis |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur **WebTable** | Oui |
| path |URL relative de la ressource qui contient la table. |Non. Quand le chemin d’accès n’est pas spécifié, seule l’URL spécifiée dans la définition du service lié est utilisée. |
| index |Index de la table dans la ressource. Pour savoir comment obtenir l’index d’une table dans une page HTML, consultez la section [Obtenir l’index d’une table dans une page HTML](#get-index-of-a-table-in-an-html-page) . |Oui |

**Exemple :**

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source de Table web.

### <a name="web-table-as-source"></a>Table web en tant que source

Pour copier des données de Table web, définissez le type de source dans l’activité de copie sur **WebSource**. Aucune autre propriété n’est prise en charge.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromWebTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Web table input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "WebSource"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="get-index-of-a-table-in-an-html-page"></a>Obtenir l’index d’une table dans une page HTML

1. Lancez **Excel 2016** et basculez vers l’onglet **Données**.
2. Cliquez sur **Nouvelle requête** dans la barre d’outils, pointez sur **À partir d’autres sources** et cliquez sur **À partir du web**.

    ![Menu Power Query](./media/copy-data-from-web-table/PowerQuery-Menu.png)
3. Dans la boîte de dialogue **À partir du web**, entrez **l’URL** que vous utiliseriez dans le service lié JSON (par exemple : https://en.wikipedia.org/wiki/), ainsi que le chemin d’accès à spécifier pour le jeu de données (par exemple : AFI%27s_100_Years...100_Movies), puis cliquez sur **OK**.

    ![Boîte de dialogue À partir du web](./media/copy-data-from-web-table/FromWeb-DialogBox.png)

    URL utilisée dans cet exemple : https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies
4. Si la boîte de dialogue **Accéder au contenu web** apparaît, sélectionnez **l’URL** et **l’authentification** adéquates, puis cliquez sur **Se connecter**.

   ![Boîte de dialogue Accéder au contenu web](./media/copy-data-from-web-table/AccessWebContentDialog.png)
5. Cliquez sur un élément de **table** dans l’arborescence pour afficher le contenu de la table, puis sur le bouton **Modifier** du bas.  

   ![Boîte de dialogue Navigateur](./media/copy-data-from-web-table/Navigator-DialogBox.png)
6. Dans la fenêtre **Éditeur de requête**, cliquez sur **Éditeur avancé** dans la barre d’outils.

    ![Bouton Éditeur avancé](./media/copy-data-from-web-table/QueryEditor-AdvancedEditorButton.png)
7. Dans la boîte de dialogue Éditeur avancé, le numéro en regard de « Source » est l’index.

    ![Éditeur avancé - Index](./media/copy-data-from-web-table/AdvancedEditor-Index.png)

Si vous utilisez Excel 2013, utilisez [Microsoft Power Query pour Excel](https://www.microsoft.com/download/details.aspx?id=39379) pour obtenir l’index. Pour plus d’informations, consultez l’article [Se connecter à une page web](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8) . Les étapes sont identiques si vous utilisez [Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/).


## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [banques de données prises en charge](copy-activity-overview.md##supported-data-stores-and-formats).