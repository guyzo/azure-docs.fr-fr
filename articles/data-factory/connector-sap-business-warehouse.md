---
title: "Copier des données de SAP Business Warehouse à l’aide d’Azure Data Factory | Microsoft Docs"
description: "Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données de SAP Business Warehouse vers des banques de données réceptrices prises en charge."
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
ms.openlocfilehash: 03841e08d071eb5f846b6a3a9e5a08edaa895611
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2017
---
# <a name="copy-data-from-sap-business-warehouse-using-azure-data-factory"></a>Copier des données de SAP Business Warehouse à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [Version 1 - Disponibilité générale](v1/data-factory-sap-business-warehouse-connector.md)
> * [Version 2 - Préversion](connector-sap-business-warehouse.md)

Cet article décrit comment utiliser l’activité de copie dans Azure Data Factory pour copier des données d’un SAP Business Warehouse (BW). Il s’appuie sur l’article [présentant une vue d’ensemble de l’activité de copie](copy-activity-overview.md).

> [!NOTE]
> Cet article s’applique à la version 2 de Data Factory, actuellement en préversion. Si vous utilisez la version 1 du service Data Factory, qui est en Disponibilité générale, consultez [Connecteur SAP BW dans V1](v1/data-factory-sap-business-warehouse-connector.md).

## <a name="supported-scenarios"></a>Scénarios pris en charge

Vous pouvez copier des données de SAP Business Warehouse vers toute banque de données réceptrice prise en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur SAP Business Warehouse prend en charge ce qui suit :

- SAP Business Warehouse **version 7.x**.
- Copie de données **d’InfoCubes et de QueryCubes** (y compris des requêtes BEx) à l’aide de requêtes MDX.
- Copie de données en utilisant une authentification de base.

## <a name="prerequisites"></a>Composants requis

Pour utiliser ce connecteur SAP Business Warehouse, vous devez :

- Configurer un runtime d’intégration auto-hébergé. Pour plus d’informations, voir l’article [Runtime d’intégration auto-hébergé](create-self-hosted-integration-runtime.md).
- Installer la **bibliothèque SAP NetWeaver** sur l’ordinateur exécutant le runtime d’intégration. Vous pouvez obtenir la bibliothèque SAP Netweaver auprès de votre administrateur SAP, ou directement depuis le [Centre de téléchargement de logiciel SAP](https://support.sap.com/swdc). Recherchez la **Note SAP 1025361** pour obtenir l’emplacement de téléchargement de la version la plus récente. Assurez-vous que vous choisissez la bibliothèque SAP NetWeaver **64 bits** qui correspond à votre installation de runtime d’intégration. Ensuite, installez tous les fichiers inclus dans le Kit de développement logiciel (SDK) RFC de SAP NetWeaver en fonction de la note SAP. La bibliothèque de SAP NetWeaver est également incluse dans l’installation des outils clients SAP.

> [!TIP]
> Placer les DLL extraites du Kit de développement logiciel NetWeaver RFC dans le dossier system32.

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec l’activité de copie à l’aide du SDK .NET, du SDK Python, d’Azure PowerShell, de l’API REST ou du modèle Azure Resource Manager. Consultez le [Didacticiel de l’activité de copie](quickstart-create-data-factory-dot-net.md) pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie.

Les sections suivantes fournissent des informations détaillées sur les propriétés utilisées pour définir les entités Data Factory spécifiques du connecteur SAP Business Warehouse.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié SAP Business Warehouse sont les suivantes :

| Propriété | Description | Requis |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur **SapBw** | Oui |
| server | Nom du serveur sur lequel réside l’instance SAP BW. | Oui |
| systemNumber | Numéro de système du système SAP BW.<br/>Valeur autorisée : nombre décimal à deux chiffres représenté sous forme de chaîne. | Oui |
| clientId | ID client du client dans le système SAP W.<br/>Valeur autorisée : nombre décimal à trois chiffres représenté sous forme de chaîne. | Oui |
| userName | Nom de l’utilisateur ayant accès au serveur SAP. | Oui |
| password | Mot de passe pour l’utilisateur. Marquez ce champ comme SecureString. | Oui |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion à la banque de données. Un Runtime d’intégration autohébergé est nécessaire comme indiqué dans [Prérequis](#prerequisites). |Oui |

**Exemple :**

```json
{
    "name": "SapBwLinkedService",
    "properties":
    {
        "type": "SapBw",
        "typeProperties":
        {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les jeux de données. Cette section fournit la liste des propriétés prises en charge par le jeu de données SAP BW.

Pour copier des données de SAP BW, affectez la valeur **RelationalTable** à la propriété type du jeu de données. Tandis qu’aucune propriété spécifique d’un type n’est prise en charge pour le jeu de données SAP BW du type RelationalTable.

**Exemple :**

```json
{
    "name": "SAPBWDataset",
    "properties":
    {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<SAP BW linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source SAP BW.

### <a name="sap-bw-as-source"></a>SAP BW en tant que source

Pour copier des données de SAP BW, définissez **RelationalSource** comme type de source dans l’activité de copie. Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Requis |
|:--- |:--- |:--- |
| type | La propriété type de la source d’activité de copie doit être définie sur **RelationalSource** | Oui |
| query | Spécifie la requête MDX pour lire les données de l’instance SAP BW. | Oui |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromSAPBW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP BW input dataset name>",
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
                "type": "RelationalSource",
                "query": "<MDX query for SAP BW>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sap-bw"></a>Mappage de type de données pour SAP BW

Lors de la copie de données de SAP BW, les mappages suivants sont utilisés entre les types de données SAP BW et les types de données intermédiaires d’Azure Data Factory. Pour découvrir comment l’activité de copie mappe le schéma et le type de données la source au récepteur, voir [Mappages de schémas et de types de données](copy-activity-schema-and-type-mapping.md).

| Type de données SAP BW | Type de données intermédiaires de Data Factory |
|:--- |:--- |
| ACCP | int |
| CHAR | String |
| CLNT | String |
| CURR | Décimal |
| CUKY | String |
| DEC | Décimal |
| FLTP | Double |
| INT1 | Byte |
| INT2 | Int16 |
| INT4 | int |
| LANG | String |
| LCHR | String |
| LRAW | Byte[] |
| PREC | Int16 |
| QUAN | Décimal |
| RAW | Byte[] |
| RAWSTRING | Byte[] |
| STRING | String |
| UNITÉ | String |
| DATS | String |
| NUMC | String |
| TIMS | String |


## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [banques de données prises en charge](copy-activity-overview.md##supported-data-stores-and-formats).