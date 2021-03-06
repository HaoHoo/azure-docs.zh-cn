---
title: "如何使用 Azure CLI 在经典模式下创建 NSG | Microsoft 文档"
description: "了解如何使用 Azure CLI 在经典模式下创建和部署 NSG"
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: tysonn
tags: azure-service-management
ms.assetid: 17d98950-5fbb-4653-bef6-d822ab37541e
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/02/2016
ms.author: jdial
ms.openlocfilehash: 115a1937a4c88ba2b986a40c84b1b759ed5e03b5
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="how-to-create-nsgs-classic-in-the-azure-cli"></a>如何在 Azure CLI 中创建 NSG（经典）
[!INCLUDE [virtual-networks-create-nsg-selectors-classic-include](../../includes/virtual-networks-create-nsg-selectors-classic-include.md)]

[!INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文介绍经典部署模型。 还可以[在 Resource Manager 部署模型中创建 NSG](virtual-networks-create-nsg-arm-cli.md)。

[!INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

下面的示例 Azure CLI 命令需要一个已经基于上述方案创建的简单环境。 如果想要运行本文档中所显示的命令，首先通过[创建 VNet](virtual-networks-create-vnet-classic-cli.md) 构建测试环境。

## <a name="how-to-create-the-nsg-for-the-front-end-subnet"></a>如何为前端子网创建 NSG
若要基于上述方案创建名为 **NSG-FrontEnd** 的 NSG，请执行下面的步骤。

1. 如果从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](../cli-install-nodejs.md)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **`azure config mode`** 命令以切换到经典模式，如下所示。
   
        azure config mode asm
   
    预期输出：
   
        info:    New mode is asm
3. 运行 **`azure network nsg create`** 命令创建 NSG。
   
        azure network nsg create -l uswest -n NSG-FrontEnd
   
    预期输出：
   
        info:    Executing command network nsg create
        info:    Creating a network security group "NSG-FrontEnd"
        info:    Looking up the network security group "NSG-FrontEnd"
        data:    Name                            : NSG-FrontEnd
        data:    Location                        : West US
        data:    Security group rules:
        data:    Name                               Source IP           Source Port  Destination IP   Destination Port  Protocol  Type      Action  Prior
        ity  Default
        data:    ---------------------------------  ------------------  -----------  ---------------  ----------------  --------  --------  ------  -----
        ---  -------
        data:    ALLOW VNET OUTBOUND                VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Outbound  Allow   65000
             true   
        data:    ALLOW VNET INBOUND                 VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Inbound   Allow   65000
             true   
        data:    ALLOW AZURE LOAD BALANCER INBOUND  AZURE_LOADBALANCER  *            *                *                 *         Inbound   Allow   65001
             true   
        data:    ALLOW INTERNET OUTBOUND            *                   *            INTERNET         *                 *         Outbound  Allow   65001
             true   
        data:    DENY ALL OUTBOUND                  *                   *            *                *                 *         Outbound  Deny    65500
             true   
        data:    DENY ALL INBOUND                   *                   *            *                *                 *         Inbound   Deny    65500
             true   
        info:    network nsg create command OK
   
    参数：
   
   * **-l（或 --location）**。 要在其中创建新 NSG 的 Azure 区域。 对于我们的方案，为 *westus*。
   * **-n（或 --name）**。 新 NSG 的名称。 对于我们的方案，即 *NSG-FrontEnd*。
4. 运行 **`azure network nsg rule create`** 命令以创建允许从 Internet 访问端口 3389 (RDP) 的规则。
   
        azure network nsg rule create -a NSG-FrontEnd -n rdp-rule -c Allow -p Tcp -r Inbound -y 100 -f Internet -o * -e * -u 3389
   
    预期输出：
   
        info:    Executing command network nsg rule create
        info:    Looking up the network security group "NSG-FrontEnd"
        info:    Creating a network security rule "rdp-rule"
        info:    Looking up the network security group "NSG-FrontEnd"
        data:    Name                            : rdp-rule
        data:    Source address prefix           : INTERNET
        data:    Source Port                     : *
        data:    Destination address prefix      : *
        data:    Destination Port                : 3389
        data:    Protocol                        : TCP
        data:    Type                            : Inbound
        data:    Action                          : Allow
        data:    Priority                        : 100
        info:    network nsg rule create command OK
   
    参数：
   
   * **-a（或 --nsg-name）**。 将在其中创建规则的 NSG 的名称。 对于我们的方案，为 *NSG-FrontEnd*。
   * **-n（或 --name）**。 新规则的名称。 对于我们的方案，为 *rdp-rule*。
   * **-c（或 --action）**。 规则的访问级别（拒绝或允许）。
   * **-p（或 --protocol）**。 规则的协议（Tcp、Udp 或 *）。
   * **-r（或 --type）**。 连接方向（入站或出站）。
   * **-y（或 --priority）**。 规则的优先级。
   * **-f（或 --source-address-prefix）**。 CIDR 中的源地址前缀或者使用默认标记。
   * **-o（或 --source-port-range）**。 源端口或端口范围。
   * **-e（或 --destination-address-prefix）**。 CIDR 中的目标地址前缀或者使用默认标记。
   * **-u（或 --destination-port-range）**。 目标端口或端口范围。
5. 运行 **`azure network nsg rule create`** 命令以创建允许从 Internet 访问端口 80 (HTTP) 的规则。
   
        azure network nsg rule create -a NSG-FrontEnd -n web-rule -c Allow -p Tcp -r Inbound -y 200 -f Internet -o * -e * -u 80
   
    预期输出：
   
        info:    Executing command network nsg rule create
        info:    Looking up the network security group "NSG-FrontEnd"
        info:    Creating a network security rule "web-rule"
        info:    Looking up the network security group "NSG-FrontEnd"
        data:    Name                            : web-rule
        data:    Source address prefix           : INTERNET
        data:    Source Port                     : *
        data:    Destination address prefix      : *
        data:    Destination Port                : 80
        data:    Protocol                        : TCP
        data:    Type                            : Inbound
        data:    Action                          : Allow
        data:    Priority                        : 200
        info:    network nsg rule create command OK
6. 运行 **`azure network nsg subnet add`** 命令以将 NSG 链接至前端子网。
   
        azure network nsg subnet add -a NSG-FrontEnd --vnet-name TestVNet --subnet-name FrontEnd
   
    预期输出：
   
        info:    Executing command network nsg subnet add
        info:    Looking up the network security group "NSG-FrontEnd"
        info:    Looking up the subnet "FrontEnd"
        info:    Looking up network configuration
        info:    Creating a network security group "NSG-FrontEnd"
        info:    network nsg subnet add command OK

## <a name="how-to-create-the-nsg-for-the-back-end-subnet"></a>如何为后端子网创建 NSG
若要基于上述方案创建名为 *NSG-BackEnd* 的 NSG，请执行下面的步骤。

1. 运行 **`azure network nsg create`** 命令创建 NSG。
   
        azure network nsg create -l uswest -n NSG-BackEnd
   
    预期输出：
   
        info:    Executing command network nsg create
        info:    Creating a network security group "NSG-BackEnd"
        info:    Looking up the network security group "NSG-BackEnd"
        data:    Name                            : NSG-BackEnd
        data:    Location                        : West US
        data:    Security group rules:
        data:    Name                               Source IP           Source Port  Destination IP   Destination Port  Protocol  Type      Action  Prior
        ity  Default
        data:    ---------------------------------  ------------------  -----------  ---------------  ----------------  --------  --------  ------  -----
        ---  -------
        data:    ALLOW VNET OUTBOUND                VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Outbound  Allow   65000
             true   
        data:    ALLOW VNET INBOUND                 VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Inbound   Allow   65000
             true   
        data:    ALLOW AZURE LOAD BALANCER INBOUND  AZURE_LOADBALANCER  *            *                *                 *         Inbound   Allow   65001
             true   
        data:    ALLOW INTERNET OUTBOUND            *                   *            INTERNET         *                 *         Outbound  Allow   65001
             true   
        data:    DENY ALL OUTBOUND                  *                   *            *                *                 *         Outbound  Deny    65500
             true   
        data:    DENY ALL INBOUND                   *                   *            *                *                 *         Inbound   Deny    65500
             true   
        info:    network nsg create command OK
   
    参数：
   
   * **-l（或 --location）**。 要在其中创建新 NSG 的 Azure 区域。 对于我们的方案，为 *westus*。
   * **-n（或 --name）**。 新 NSG 的名称。 对于我们的方案，即 *NSG-FrontEnd*。
2. 运行 **`azure network nsg rule create`** 命令以创建允许从前端子网访问端口 1433 (SQL) 的规则。
   
        azure network nsg rule create -a NSG-BackEnd -n sql-rule -c Allow -p Tcp -r Inbound -y 100 -f 192.168.1.0/24 -o * -e * -u 1433
   
    预期输出：
   
        info:    Executing command network nsg rule create
        info:    Looking up the network security group "NSG-BackEnd"
        info:    Creating a network security rule "sql-rule"
        info:    Looking up the network security group "NSG-BackEnd"
        data:    Name                            : sql-rule
        data:    Source address prefix           : 192.168.1.0/24
        data:    Source Port                     : *
        data:    Destination address prefix      : *
        data:    Destination Port                : 1433
        data:    Protocol                        : TCP
        data:    Type                            : Inbound
        data:    Action                          : Allow
        data:    Priority                        : 100
        info:    network nsg rule create command OK
3. 运行 **`azure network nsg rule create`** 命令以创建拒绝访问 Internet 的规则。
   
        azure network nsg rule create -a NSG-BackEnd -n web-rule -c Deny -p Tcp -r Outbound -y 200 -f * -o * -e Internet -u 80
   
    预期输出：
   
        info:    Executing command network nsg rule create
        info:    Looking up the network security group "NSG-BackEnd"
        info:    Creating a network security rule "web-rule"
        info:    Looking up the network security group "NSG-BackEnd"
        data:    Name                            : web-rule
        data:    Source address prefix           : *
        data:    Source Port                     : *
        data:    Destination address prefix      : INTERNET
        data:    Destination Port                : 80
        data:    Protocol                        : TCP
        data:    Type                            : Outbound
        data:    Action                          : Deny
        data:    Priority                        : 200
        info:    network nsg rule create command OK
4. 运行 **`azure network nsg subnet add`** 命令以将 NSG 链接至后端子网。
   
        azure network nsg subnet add -a NSG-BackEnd --vnet-name TestVNet --subnet-name BackEnd
   
    预期输出：
   
        info:    Executing command network nsg subnet add
        info:    Looking up the network security group "NSG-BackEndX"
        info:    Looking up the subnet "BackEnd"
        info:    Looking up network configuration
        info:    Creating a network security group "NSG-BackEndX"
        info:    network nsg subnet add command OK

