---
title: "使用 Azure Site Recovery 在 Azure 之间复制虚拟机的网络指南 | Microsoft Docs"
description: "Azure 虚拟机复制网络指南"
services: site-recovery
documentationcenter: 
author: sujayt
manager: rochakm
editor: 
ms.assetid: 
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 08/31/2017
ms.author: sujayt
ms.openlocfilehash: a2ea66f2ed3815d0375001571e64dad338f16e3e
ms.sourcegitcommit: cfd1ea99922329b3d5fab26b71ca2882df33f6c2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/30/2017
---
# <a name="networking-guidance-for-replicating-azure-virtual-machines"></a>Azure 虚拟机复制网络指南

>[!NOTE]
> 用于 Azure 虚拟机的 Site Recovery 复制当前处于预览状态。

本文详细介绍了使用 Azure Site Recovery 在不同区域之间复制和恢复 Azure 虚拟机的网络指南。 有关 Azure Site Recovery 要求的详细信息，请参阅[先决条件](site-recovery-support-matrix-azure-to-azure.md)一文。

## <a name="site-recovery-architecture"></a>Site Recovery 体系结构

借助 Site Recovery，可以简单轻松地将 Azure 虚拟机上运行的应用程序复制到另一个 Azure 区域，以便在主要区域发生中断时恢复这些应用程序。 详细了解[此方案和 Site Recovery 体系结构](concepts-azure-to-azure-architecture.md)。

## <a name="your-network-infrastructure"></a>网络基础结构

下图描绘了 Azure 虚拟机上运行的应用程序的典型 Azure 环境：

![customer-environment](./media/site-recovery-azure-to-azure-architecture/source-environment.png)

如果使用 Azure ExpressRoute 或从本地网络到 Azure 的 VPN 连接，则环境如下所示：

![customer-environment](./media/site-recovery-azure-to-azure-architecture/source-environment-expressroute.png)

通常，客户使用防火墙和/或网络安全组 (NSG) 保护他们的网络。 防火墙可以使用基于 URL 或基于 IP 的允许列表来控制网络连接。 NSG 允许使用 IP 范围控制网络连接的规则。

>[!IMPORTANT]
> 如果使用经过身份验证的代理控制网络连接，则不受支持，且无法启用 Site Recovery 复制。

以下部分讨论需要从 Azure 虚拟机进行的网络出站连接更改，以使 Site Recovery 复制正常工作。

## <a name="outbound-connectivity-for-azure-site-recovery-urls"></a>Azure Site Recovery URL 的出站连接

如果使用任何基于 URL 的防火墙代理控制出站连接，请确保将下列必需的 Azure Site Recovery 服务 URL 列入白名单：


**URL** | **用途**  
--- | ---
*.blob.core.windows.net | 必需，以便从 VM 将数据写入到源区域中的缓存存储帐户。
login.microsoftonline.com | 必需，用于向 Site Recovery 服务 URL 进行授权和身份验证。
*.hypervrecoverymanager.windowsazure.com | 必需，以便从 VM 进行 Site Recovery 服务通信。
*.servicebus.windows.net | 必需，以便从 VM 写入 Site Recovery 监视和诊断数据。

## <a name="outbound-connectivity-for-azure-site-recovery-ip-ranges"></a>Azure Site Recovery IP 范围的出站连接

>[!NOTE]
> 若要在网络安全组上自动创建所需的 NSG 规则，可以[下载并使用此脚本](https://gallery.technet.microsoft.com/Azure-Recovery-script-to-0c950702)。

>[!IMPORTANT]
> * 在生产网络安全组上创建所需的 NSG 规则之前，建议先在测试网络安全组上创建这些规则，并确保没有任何问题。
> * 若要创建所需的 NSG 规则数，请确保订阅已列入允许列表。 联系支持人员，提高订阅中的 NSG 规则限制。

如果使用任何基于 IP 的防火墙代理或 NSG 规则控制出站连接，则需要将以下 IP 范围列入白名单，具体取决于虚拟机的源位置和目标位置：

- 对应于源位置的所有 IP 范围。 （可以下载 [IP 范围](https://www.microsoft.com/download/confirmation.aspx?id=41653)。）必须列入允许列表，才能从 VM 将数据写入到缓存存储帐户。

- 对应于 Office 365 [身份验证和标识 IP V4 终结点](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity)的所有 IP 范围。

    >[!NOTE]
    > 如果将来向 Office 365 IP 范围添加新的 IP，则需要创建新的 NSG 规则。

- Site Recovery 服务终结点 IP（[以 XML 文件形式提供](https://aka.ms/site-recovery-public-ips)），具体取决于目标位置：

   **目标位置** | **Site Recovery 服务 IP** |  **Site Recovery 监视 IP**
   --- | --- | ---
   东亚 | 52.175.17.132 | 13.94.47.61
   东南亚 | 52.187.58.193 | 13.76.179.223
   印度中部 | 52.172.187.37 | 104.211.98.185
   印度南部 | 52.172.46.220 | 104.211.224.190
   美国中北部 | 23.96.195.247 | 168.62.249.226
   欧洲北部 | 40.69.212.238 | 52.169.18.8
   欧洲西部 | 52.166.13.64 | 40.68.93.145
   美国东部 | 13.82.88.226 | 104.45.147.24
   美国西部 | 40.83.179.48 | 104.40.26.199
   美国中南部 | 13.84.148.14 | 104.210.146.250
   美国中部 | 40.69.144.231 | 52.165.34.144
   美国东部 2 | 52.184.158.163 | 40.79.44.59
   日本东部 | 52.185.150.140 | 138.91.1.105
   日本西部 | 52.175.146.69 | 138.91.17.38
   巴西南部 | 191.234.185.172 | 23.97.97.36
   澳大利亚东部 | 104.210.113.114 | 191.239.64.144
   澳大利亚东南部 | 13.70.159.158 | 191.239.160.45
   加拿大中部 | 52.228.36.192 | 40.85.226.62
   加拿大东部 | 52.229.125.98 | 40.86.225.142
   美国中西部 | 52.161.20.168 | 13.78.149.209
   美国西部 2 | 52.183.45.166 | 13.66.228.204
   英国西部 | 51.141.3.203 | 51.141.14.113
   英国南部 | 51.140.43.158 | 51.140.189.52
   英国南部 2 | 13.87.37.4| 13.87.34.139
   英国北部 | 51.142.209.167 | 13.87.102.68
   韩国中部 | 52.231.28.253 | 52.231.32.85
   韩国南部 | 52.231.298.185 | 52.231.200.144

## <a name="sample-nsg-configuration"></a>NSG 配置示例
本部分介绍配置 NSG 规则的相关步骤，配置后，Site Recovery 复制便可在虚拟机上正常工作。 如果使用 NSG 规则控制出站连接，请对所有必需的 IP 范围使用“允许 HTTPS 出站”规则。

>[!Note]
> 若要在网络安全组上自动创建所需的 NSG 规则，可以[下载并使用此脚本](https://gallery.technet.microsoft.com/Azure-Recovery-script-to-0c950702)。

例如，如果 VM 的源位置为“美国东部”，复制目标位置为“美国中部”，则按照接下来两部分中的步骤操作。

>[!IMPORTANT]
> * 在生产网络安全组上创建所需的 NSG 规则之前，建议先在测试网络安全组上创建这些规则，并确保没有任何问题。
> * 若要创建所需的 NSG 规则数，请确保订阅已列入允许列表。 联系支持人员，提高订阅中的 NSG 规则限制。

### <a name="nsg-rules-on-the-east-us-network-security-group"></a>美国东部网络安全组上的 NSG 规则

* 创建对应于[美国东部 IP 范围](https://www.microsoft.com/download/confirmation.aspx?id=41653)的规则。 必须执行此操作，才能从 VM 将数据写入到缓存存储帐户。

* 为对应于 Office 365 [身份验证和标识 IP V4 终结点](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity)的所有 IP 范围创建规则。

* 创建对应于目标位置的规则：

   **位置** | **Site Recovery 服务 IP** |  **Site Recovery 监视 IP**
    --- | --- | ---
   美国中部 | 40.69.144.231 | 52.165.34.144

### <a name="nsg-rules-on-the-central-us-network-security-group"></a>美国中部网络安全组上的 NSG 规则

必须创建这些规则，才能在故障转移后启用从目标区域到源区域的复制：

* 对应于[美国中部 IP 范围](https://www.microsoft.com/download/confirmation.aspx?id=41653)的规则。 必须创建这些规则，才能从 VM 将数据写入到缓存存储帐户。

* 针对对应于 Office 365 [身份验证和标识 IP V4 终结点](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity)的所有 IP 范围的规则。

* 对应于源位置的规则：

   **位置** | **Site Recovery 服务 IP** |  **Site Recovery 监视 IP**
    --- | --- | ---
   美国东部 | 13.82.88.226 | 104.45.147.24


## <a name="guidelines-for-existing-azure-to-on-premises-expressroutevpn-configuration"></a>现有 Azure 到本地 ExpressRoute/VPN 配置的相关准则

如果在本地和 Azure 中的源位置之间建立了 ExpressRoute 或 VPN 连接，请遵从本部分中的准则。

### <a name="forced-tunneling-configuration"></a>强制隧道配置

常见的客户配置是定义默认路由 (0.0.0.0/0)，以强制出站 Internet 流量流向本地位置。 但并不建议这样做。 复制流量和 Site Recovery 服务通信不应离开 Azure 边界。 若要解决此问题，可以为[这些 IP 范围](#outbound-connectivity-for-azure-site-recovery-ip-ranges)添加用户定义路由 (UDR)，使复制流量不流向本地。

### <a name="connectivity-between-the-target-and-on-premises-location"></a>目标位置与本地位置之间的连接

目标位置与本地位置之间的连接遵循以下准则：
- 如果应用程序需要连接到本地计算机，或者有通过 VPN/ExpressRoute 从本地连接到应用程序的客户端，请确保在目标 Azure 区域和本地数据中心之间至少有一个[站点到站点连接](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)。

- 如果预计有大量流量在目标 Azure 区域与本地数据中心之间流动，则应在目标 Azure 区域与本地数据中心之间再创建一个 [ExpressRoute 连接](../../expressroute/expressroute-introduction.md)。

- 如果想在故障转移后保留虚拟机的 IP，则将目标区域的站点到站点/ExpressRoute 连接保留为断开状态。 这样做可确保源区域的 IP 范围和目标区域的 IP 范围之间没有范围冲突。

### <a name="best-practices-for-expressroute-configuration"></a>ExpressRoute 配置最佳做法
请遵从以下 ExpressRoute 配置最佳做法：

- 需要在源区域和目标区域中分别创建 ExpressRoute 线路。 然后需要在以下对象之间创建连接：
  - 源虚拟网络和 ExpressRoute 线路。
  - 目标虚拟网络和 ExpressRoute 线路。

- ExpressRoute 标准规定，可以在同一地缘政治区域创建线路。 若要在不同的地缘政治区域创建 ExpressRoute 线路，则需使用 Azure ExpressRoute 高级版，这会增加成本。 （如果已在使用 ExpressRoute 高级版，则不必支付额外费用。）有关更多详细信息，请参阅 [ExpressRoute 位置文档](../../expressroute/expressroute-locations.md#azure-regions-to-expressroute-locations-within-a-geopolitical-region)和 [ExpressRoute 定价](https://azure.microsoft.com/pricing/details/expressroute/)。

- 建议在源区域和目标区域中使用不同的 IP 范围。 ExpressRoute 线路无法同时连接两个使用相同 IP 范围的 Azure 虚拟网络。

- 可以在这两个区域创建使用相同 IP 范围的虚拟网络，然后在这两个区域分别创建 ExpressRoute 线路。 发生故障转移时，断开源虚拟网络中的线路，连接目标虚拟网络中的线路。

 >[!IMPORTANT]
 > 如果主要区域完全关闭，断开连接操作可能会失败， 导致目标虚拟网络无法获取 ExpressRoute 连接。

## <a name="next-steps"></a>后续步骤
[复制 Azure 虚拟机](azure-to-azure-quickstart.md)，开始对工作负荷进行保护。
