---
title: "Filtrer le trafic réseau avec des groupes de sécurité réseau et d’application Azure (préversion) | Microsoft Docs"
description: "Découvrez comment créer des groupes de sécurité réseau et d’application (préversion) pour restreindre le type de trafic réseau vers et depuis les machines virtuelles."
services: virtual-network
documentationcenter: 
author: jimdial
manager: jeconnoc
editor: 
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/20/2017
ms.author: jdial
ms.custom: 
ms.openlocfilehash: 035eb44432081ef52c758a5d311b4d2ba2c6108d
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2017
---
# <a name="filter-network-traffic-with-network-and-application-security-groups-preview"></a>Filtrer le trafic réseau avec des groupes de sécurité réseau et d’application (préversion)

Dans ce didacticiel, découvrez comment créer des groupes de sécurité réseau pour filtrer le trafic réseau vers et depuis des groupes de machines virtuelles avec des groupes de sécurité d’application. Pour en savoir plus sur les groupes de sécurité réseau et sur les groupes de sécurité d’application, consultez [Vue d’ensemble de la sécurité](security-overview.md). 

Les sections suivantes incluent des mesures que vous pouvez prendre pour créer des groupes de sécurité réseau à l’aide de l’interface de ligne de commande Azure ([Azure CLI](#azure-cli)) et [d’Azure PowerShell](#powershell). Le résultat est le même, quel que soit l’outil choisi pour créer les groupes de sécurité réseau. Cliquez sur un lien d’outil pour accéder à la section correspondante du didacticiel. 

Cet article explique comment créer des groupes de sécurité réseau par le biais du modèle de déploiement Resource Manager, qui est le modèle de déploiement que nous recommandons à cette fin. Si vous devez créer un groupe de sécurité réseau (classique), consultez [Créer un groupe de sécurité réseau (classique)](virtual-networks-create-nsg-classic-ps.md). Si vous ne connaissez pas les modèles de déploiement Azure, consultez [Comprendre les modèles de déploiement Azure](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

> [!NOTE]
> Ce didacticiel utilise des fonctionnalités de groupe de sécurité réseau qui sont disponibles en préversion. Les fonctionnalités en préversion n’offrent pas les mêmes disponibilité et fiabilité que les fonctionnalités de la version générale. En préversion, les fonctionnalités ne sont disponibles que dans la région Ouest du centre des États-Unis. Si vous souhaitez implémenter des groupes de sécurité réseau en utilisant uniquement les fonctionnalités de la version générale, consultez [Créer un groupe de sécurité réseau](virtual-networks-create-nsg-arm-pportal.md). 

## <a name="azure-cli"></a>Interface de ligne de commande Azure

Les commandes d’Azure CLI sont identiques, que vous les exécutiez à partir de Windows, de Linux ou de macOS. Toutefois, il existe des différences de script entre les interpréteurs de commandes du système d’exploitation. Le script présenté dans les étapes suivantes s’exécute dans un interpréteur de commandes Bash. 

1. [Installer et configurer Azure CLI](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json).
2. Vérifiez que vous utilisez la version 2.0.18 ou une version ultérieure de l’interface Azure CLI en saisissant la commande `az --version`. Si ce n’est pas le cas, installez la version la plus récente.
3. Connectez-vous à Azure avec la commande `az login`.
4. Inscrivez-vous à la préversion en entrant les commandes suivantes :
    
    ```azurecli-interactive
    az feature register --name AllowApplicationSecurityGroups --namespace Microsoft.Network
    ``` 

5. Vérifiez que vous êtes inscrit à la préversion en saisissant la commande suivante :

    ```azurecli-interactive
    az feature show --name AllowApplicationSecurityGroups --namespace Microsoft.Network
    ```

    N’effectuez pas les étapes restantes tant que la mention *Registered* n’apparaît pas dans la section **state** de la sortie retournée par la commande précédente. Si vous continuez avant d’être inscrit, les étapes restantes échouent.
6. Exécutez le script bash suivant pour créer un groupe de ressources :

    ```azurecli-interactive
    #!/bin/bash
    
    az group create \
      --name myResourceGroup \
      --location westcentralus
    ```

7. Créez trois groupes de sécurité d’application, à raison d’un par type de serveur :

    ```azurecli-interactive
    az network asg create \
      --resource-group myResourceGroup \
      --name WebServers \
      --location westcentralus  

    az network asg create \
      --resource-group myResourceGroup \
      --name AppServers \
      --location westcentralus

    az network asg create \
      --resource-group myResourceGroup \
      --name DatabaseServers \
      --location westcentralus
    ```

8. Créez un groupe de sécurité réseau :

    ```azurecli-interactive
    az network nsg create \
      --resource-group myResourceGroup \
      --name myNsg \
      --location westcentralus
    ```

9. Créez des règles de sécurité dans le groupe de sécurité réseau, en définissant les groupes de sécurité d’application en tant que destination :
    
    ```azurecli-interactive    
    az network nsg rule create \
      --resource-group myResourceGroup \
      --nsg-name myNsg \
      --name WebRule \
      --priority 200 \
      --access "Allow" \
      --direction "inbound" \
      --source-address-prefixes "Internet" \
      --destination-asgs "WebServers" \
      --destination-port-ranges 80 \
      --protocol "TCP"

    az network nsg rule create \
      --resource-group myResourceGroup \
      --nsg-name myNsg \
      --name AppRule \
      --priority 300 \
      --access "Allow" \
      --direction "inbound" \
      --source-asgs "WebServers" \
      --destination-asgs "AppServers" \
      --destination-port-ranges 443 \
      --protocol "TCP"  

    az network nsg rule create \
      --resource-group myResourceGroup \
      --nsg-name myNsg \
      --name DatabaseRule \
      --priority 400 \
      --access "Allow" \
      --direction "inbound" \
      --source-asgs "AppServers" \
      --destination-asgs "DatabaseServers" \
      --destination-port-ranges 1336 \
      --protocol "TCP" 
    ``` 

10. Créez un réseau virtuel : 
    
    ```azurecli-interactive
    az network vnet create \
      --name myVnet \
      --resource-group myResourceGroup \
      --subnet-name mySubnet \
      --address-prefix 10.0.0.0/16 \
      --location westcentralus
    ```

11. Associez le groupe de sécurité réseau au sous-réseau du réseau virtuel :

    ```azurecli-interactive
    az network vnet subnet update \
      --name mySubnet \
      --resource-group myResourceGroup \
      --vnet-name myVnet \
      --network-security-group myNsg
    ```
    
12. Créez trois interfaces réseau, à raison d’une par type de serveur : 

    ```azurecli-interactive
    az network nic create \
      --resource-group myResourceGroup \
      --name myNic1 \
      --vnet-name myVnet \
      --subnet mySubnet \
      --network-security-group myNsg \
      --location westcentralus \
      --application-security-groups "WebServers" "AppServers"

    az network nic create \
      --resource-group myResourceGroup \
      --name myNic2 \
      --vnet-name myVnet \
      --subnet mySubnet \
      --network-security-group myNsg \
      --location westcentralus \
      --application-security-groups "AppServers"

    az network nic create \
      --resource-group myResourceGroup \
      --name myNic3 \
      --vnet-name myVnet \
      --subnet mySubnet \
      --network-security-group myNsg \
      --location westcentralus \
      --application-security-groups "DatabaseServers"
    ```

    Seule la règle de sécurité correspondante, créée à l’étape 9, est appliquée à l’interface réseau, en fonction du groupe de sécurité d’application dont l’interface réseau est membre. Par exemple, seul *WebRule* est efficace pour *myWebNic*, car l’interface réseau est membre du groupe de sécurité d’application *WebServers* et la règle spécifie le groupe de sécurité d’application *WebServers* comme destination. Les règles *AppRule* et *DatabaseRule* ne sont pas appliquées à *myWebNic*, car l’interface réseau n’est pas membre des groupes de sécurité d’application *AppServers* et *DatabaseServers*.

13. Créez une machine virtuelle pour chaque type de serveur, en attachant l’interface réseau correspondante à chaque machine virtuelle. Cet exemple crée des machines virtuelles Windows, mais vous pouvez remplacer *win2016datacenter* par *UbuntuLTS* pour créer des machines virtuelles Linux à la place.

    ```azurecli-interactive
    # Update for your admin password
    AdminPassword=ChangeYourAdminPassword1

    az vm create \
      --resource-group myResourceGroup \
      --name myWebVm \
      --location westcentralus \
      --nics myNic1 \
      --image win2016datacenter \
      --admin-username azureuser \
      --admin-password $AdminPassword

    az vm create \
      --resource-group myResourceGroup \
      --name myAppVm \
      --location westcentralus \
      --nics myNic2 \
      --image win2016datacenter \
      --admin-username azureuser \
      --admin-password $AdminPassword

    az vm create \
      --resource-group myResourceGroup \
      --name myDatabaseVm \
      --location westcentralus \
      --nics myNic3 \
      --image win2016datacenter \
      --admin-username azureuser \
      --admin-password $AdminPassword    
    ```

14. **Facultatif :** supprimez les ressources que vous avez créées dans ce didacticiel en suivant les étapes indiquées dans la section [Supprimer des ressources](#delete-cli).

## <a name="powershell"></a>PowerShell

1. Installez et configurez [PowerShell](/powershell/azure/install-azurerm-ps).
2. Vérifiez que le module AzureRm version 4.4.0 ou ultérieure est installé. Vous pouvez vérifier la version installée en entrant la commande `Get-Module -ListAvailable AzureRM`. Si vous devez installer ou mettre à niveau le module AzureRM, installez la dernière version du module à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/AzureRM).
3. Dans une session PowerShell, connectez-vous à Azure avec votre [compte Azure](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#account) à l’aide de la commande `login-azurermaccount`.
4. Inscrivez-vous à la préversion en entrant les commandes suivantes :
    
    ```powershell
    Register-AzureRmProviderFeature -FeatureName AllowApplicationSecurityGroups -ProviderNamespace Microsoft.Network
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
    ``` 

5. Confirmez que vous êtes inscrit à la préversion en entrant la commande suivante :

    ```powershell
    Get-AzureRmProviderFeature -FeatureName AllowApplicationSecurityGroups -ProviderNamespace Microsoft.Network
    ```

    N’effectuez pas les étapes restantes tant que *Registered* n’apparaît pas dans la colonne **RegistrationState** de la sortie retournée par la commande précédente. Si vous continuez avant d’être inscrit, les étapes restantes échouent.
        
6. Créez un groupe de ressources :

    ```powershell
    New-AzureRmResourceGroup `
      -Name myResourceGroup `
      -Location westcentralus
    ```

7. Créez trois groupes de sécurité d’application, à raison d’un par type de serveur :

    ```powershell
    $webAsg = New-AzureRmApplicationSecurityGroup `
      -ResourceGroupName myResourceGroup `
      -Name WebServers `
      -Location westcentralus
  
    $appAsg = New-AzureRmApplicationSecurityGroup `
      -ResourceGroupName myResourceGroup `
      -Name AppServers `
      -Location westcentralus

    $databaseAsg = New-AzureRmApplicationSecurityGroup `
      -ResourceGroupName myResourceGroup `
      -Name DatabaseServers `
      -Location westcentralus
    ```

8. Créez des règles de sécurité pour chaque type de serveur.
    
    ```powershell
    $webRule = New-AzureRmNetworkSecurityRuleConfig `
      -Name "WebRule" `
      -Access Allow `
      -Protocol Tcp `
      -Direction Inbound `
      -Priority 200 `
      -SourceAddressPrefix Internet `
      -SourcePortRange * `
      -DestinationApplicationSecurityGroupId $webAsg.id `
      -DestinationPortRange 80  

    $appRule = New-AzureRmNetworkSecurityRuleConfig `
      -Name "AppRule" `
      -Access Allow `
      -Protocol Tcp `
      -Direction Inbound `
      -Priority 300 `
      -SourceApplicationSecurityGroupId $webAsg.id `
      -SourcePortRange * `
      -DestinationApplicationSecurityGroupId $appAsg.id `
      -DestinationPortRange 443 

    $databaseRule = New-AzureRmNetworkSecurityRuleConfig `
      -Name "DatabaseRule" `
      -Access Allow `
      -Protocol Tcp `
      -Direction Inbound `
      -Priority 400 `
      -SourceApplicationSecurityGroupId $appAsg.id `
      -SourcePortRange * `
      -DestinationApplicationSecurityGroupId $databaseAsg.id `
      -DestinationPortRange 1336    
    ``` 

9. Créez un groupe de sécurité réseau :

    ```powershell
    $nsg = New-AzureRmNetworkSecurityGroup `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus `
      -Name myNsg `
      -SecurityRules $WebRule,$AppRule,$DatabaseRule
    ```

10. Créez une configuration de sous-réseau et associez-y le groupe de sécurité réseau :
    
    ```powershell
    $subnet = New-AzureRmVirtualNetworkSubnetConfig `
      -AddressPrefix 10.0.0.0/24 `
      -Name mySubnet `
      -NetworkSecurityGroup $nsg
    ```

11. Créez un réseau virtuel : 
    
    ```powershell
    $vNet = New-AzureRmVirtualNetwork `
      -Name myVnet `
      -AddressPrefix '10.0.0.0/16' `
      -Subnet $subnet `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus
    ```

12. Créez trois interfaces réseau, à raison d’une par type de serveur. 

    ```powershell
    $nic1 = New-AzureRmNetworkInterface `
      -Name myNic1 `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus `
      -Subnet $vNet.Subnets[0] `
      -NetworkSecurityGroup $nsg `
      -ApplicationSecurityGroup $webAsg,$appAsg

    $nic2 = New-AzureRmNetworkInterface `
      -Name myNic2 `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus `
      -Subnet $vNet.Subnets[0] `
      -NetworkSecurityGroup $nsg `
      -ApplicationSecurityGroup $appAsg

    $nic3 = New-AzureRmNetworkInterface `
      -Name myNic3 `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus `
      -Subnet $vNet.Subnets[0] `
      -NetworkSecurityGroup $nsg `
      -ApplicationSecurityGroup $databaseAsg
    ```

    Seule la règle de sécurité correspondante créée à l’étape 8 est appliquée à l’interface réseau, en fonction du groupe de sécurité d’application dont l’interface réseau est membre. Par exemple, seul *WebRule* est efficace pour *myWebNic*, car l’interface réseau est membre du groupe de sécurité d’application *WebServers* et la règle spécifie le groupe de sécurité d’application *WebServers* comme destination. Les règles *AppRule* et *DatabaseRule* ne sont pas appliquées à *myWebNic*, car l’interface réseau n’est pas membre des groupes de sécurité d’application *AppServers* et *DatabaseServers*.

13. Créez une machine virtuelle pour chaque type de serveur, en attachant l’interface réseau correspondante à chaque machine virtuelle. Cet exemple crée des machines virtuelles Windows, mais avant d’exécuter le script, vous pouvez remplacer *-Windows* par *-Linux*, *MicrosoftWindowsServer* par *Canonical*, *WindowsServer* par *UbuntuServer* et *2016-Datacenter* par *14.04.2-LTS* pour créer des machines virtuelles Linux à la place.

    ```powershell
    # Create user object
    $cred = Get-Credential -Message "Enter a username and password for the virtual machine."
   
    # Create the web server virtual machine configuration and virtual machine.
    $webVmConfig = New-AzureRmVMConfig `
      -VMName myWebVm `
      -VMSize Standard_DS1_V2 | `
    Set-AzureRmVMOperatingSystem -Windows `
      -ComputerName myWebVm `
      -Credential $cred | `
    Set-AzureRmVMSourceImage `
      -PublisherName MicrosoftWindowsServer `
      -Offer WindowsServer `
      -Skus 2016-Datacenter `
      -Version latest | `
    Add-AzureRmVMNetworkInterface `
      -Id $nic1.Id
    New-AzureRmVM `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus `
      -VM $webVmConfig

    # Create the app server virtual machine configuration and virtual machine.
    $appVmConfig = New-AzureRmVMConfig `
      -VMName myAppVm `
      -VMSize Standard_DS1_V2 | `
    Set-AzureRmVMOperatingSystem -Windows `
      -ComputerName myAppVm `
      -Credential $cred | `
    Set-AzureRmVMSourceImage `
      -PublisherName MicrosoftWindowsServer `
      -Offer WindowsServer `
      -Skus 2016-Datacenter `
      -Version latest | `
    Add-AzureRmVMNetworkInterface `
      -Id $nic2.Id
    New-AzureRmVM `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus `
      -VM $appVmConfig

    # Create the database server virtual machine configuration and virtual machine.
    $databaseVmConfig = New-AzureRmVMConfig `
      -VMName myDatabaseVm `
      -VMSize Standard_DS1_V2 | `
    Set-AzureRmVMOperatingSystem -Windows `
      -ComputerName mydatabaseVm `
      -Credential $cred | `
    Set-AzureRmVMSourceImage `
      -PublisherName MicrosoftWindowsServer `
      -Offer WindowsServer `
      -Skus 2016-Datacenter `
      -Version latest | `
    Add-AzureRmVMNetworkInterface `
      -Id $nic3.Id
    New-AzureRmVM `
      -ResourceGroupName myResourceGroup `
      -Location westcentralus `
      -VM $databaseVmConfig
    ```

14. **Facultatif :** supprimez les ressources que vous avez créées dans ce didacticiel en suivant les étapes indiquées dans la section [Supprimer des ressources](#delete-cli).

## <a name="delete"></a>Supprimer des ressources

Lorsque vous aurez terminé ce didacticiel, vous souhaiterez peut-être supprimer les ressources créées afin que leur utilisation ne soit pas facturée. La suppression d’un groupe de ressources supprime également toutes les ressources qu’il contient.

### <a name="delete-portal"></a>Portail Azure

1. Dans le champ de recherche du portail, entrez **myResourceGroup**. Dans les résultats de la recherche, cliquez sur **myResourceGroup**.
2. Dans le panneau **myResourceGroup**, cliquez sur l’icône **Supprimer**.
3. Pour confirmer la suppression, dans le champ **TAPEZ LE NOM DU GROUPE DE RESSOURCES :**, entrez **myResourceGroup**, puis cliquez sur **Supprimer**.

### <a name="delete-cli"></a>Interface CLI Azure

Dans une session CLI, entrez la commande suivante :

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

### <a name="delete-powershell"></a>PowerShell

Dans une session PowerShell, entrez la commande suivante :

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

