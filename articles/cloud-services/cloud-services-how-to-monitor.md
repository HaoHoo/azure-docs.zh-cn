---
title: "如何监视云服务 | Microsoft Docs"
description: "了解如何使用 Azure 经典门户监视云服务。"
services: cloud-services
documentationcenter: 
author: thraka
manager: timlt
editor: 
ms.assetid: 5c48d2fb-b8ea-420f-80df-7aebe2b66b1b
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2015
ms.author: adegeo
ms.openlocfilehash: c369b22cf068a473343b006eb1b06fdd350d31db
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="how-to-monitor-cloud-services"></a>如何监视云服务
[!INCLUDE [disclaimer](../../includes/disclaimer.md)]

可以在 Azure 经典门户中监视云服务的 `key` 性能度量值。 可将每个服务角色的监视级别设置为“最低”和“详细”，并且可以自定义监视显示信息。 详细监视数据存储在存储帐户中，可在门户外部访问该存储帐户。 

Azure 经典门户中的监视显示信息是高度可配置的。 可在“监视”页上的度量值列表中选择要监视的度量值，并且可以选择要在“监视”页和仪表板上的度量值图表中显示的度量值。 

## <a name="concepts"></a>概念
默认情况下，将使用从角色实例（虚拟机）的主机操作系统收集到的性能计数器数据对新的云服务进行最低监视。 最低监视度量值限于“CPU 百分比”、“输入数据量”、“输出数据量”、“磁盘读取吞吐量”以及“磁盘写入吞吐量”。 通过配置详细监视，可根据虚拟机（角色实例）中的性能数据设定其他度量值。 详细监视度量值可对应用程序运行期间出现的问题进行进一步分析。

默认情况下，将每隔 3 分钟从角色实例中收集和传输一次性能计数器数据。 启用详细监视时，将每隔 5 分钟、1 小时和 12 小时为每个角色实例，以及每个角色的所有角色实例汇总一次原始性能计数器数据。 该汇总数据会在 10 天之后清除。

在启用详细监视后，汇总后的监视数据存储在存储帐户中的相应表中。 若要为某个角色启用详细监视，必须配置链接到该存储帐户的诊断连接字符串。 可对不同的角色使用不同的存储帐户。

启用详细监视将增加与数据存储、数据传输和存储事务相关的存储成本。 最低监视不需要存储帐户。 即使将监视级别设置为“详细监视”，在最低监视级别公开的度量值数据也不会存储在存储帐户中。

## <a name="how-to-configure-monitoring-for-cloud-services"></a>如何：为云服务配置监视
可使用下列步骤在 Azure 经典门户中配置详细监视或最低监视。 

### <a name="before-you-begin"></a>开始之前
* 创建用于存储监视数据的经典存储帐户。 可对不同的角色使用不同的存储帐户。 有关详细信息，请参阅[如何创建存储帐户](../storage/common/storage-create-storage-account.md#create-a-storage-account)。
* 为云服务角色启用 Azure 诊断。 请参阅[配置云服务诊断](cloud-services-dotnet-diagnostics.md)。

确保“角色”配置中存在诊断连接字符串。 只有先启用 Azure Diagnostics 并在“角色”配置中包含了诊断连接字符串，才能启用详细监视。   

> [!NOTE]
> 面向 Azure SDK 2.5 的项目不会将诊断连接字符串自动加入项目模板。 对于这些项目，需要将诊断连接字符串手动添加到“角色”配置。
> 
> 

**手动将诊断连接字符串添加到“角色”配置**

1. 在 Visual Studio 中打开云服务项目
2. 双击“角色”打开“角色”设计器，并选择“设置”选项卡
3. 查找名为 **Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString** 的设置。 
4. 如果此设置不存在，请单击“添加设置”按钮将该设置添加到配置中，并将新设置的类型更改为 **ConnectionString**
5. 单击“...”按钮设置连接字符串的值。 这会打开用于选择存储帐户的对话框。
   
    ![Visual Studio 设置](./media/cloud-services-how-to-monitor/CloudServices_Monitor_VisualStudioDiagnosticsConnectionString.png)

### <a name="to-change-the-monitoring-level-to-verbose-or-minimal"></a>将监视级别更改为详细监视或最低监视
1. 在 [Azure 经典门户](https://manage.windowsazure.com/)中，打开云服务部署的“配置”页。
2. 在“级别”中，单击“详细”或“最少”。 
3. 单击“保存” 。

在打开详细监视后，应能在Azure 经典门户中看到一小时内的监视数据。

原始性能计数器数据和汇总监视数据存储在存储帐户中由角色的部署 ID 限定的表中。 

## <a name="how-to-receive-alerts-for-cloud-service-metrics"></a>如何：接收云服务度量值的警报
根据云服务监控指标，可能收到警告。 在 Azure 经典门户上的“管理服务”页上，可创建在所选度量值达到指定值时触发警报的规则。 也可选择在触发警报时发送电子邮件。 有关详细信息，请参阅[如何：在 Azure 中接收警报通知和管理警报规则](http://go.microsoft.com/fwlink/?LinkId=309356)。

## <a name="how-to-add-metrics-to-the-metrics-table"></a>如何：向度量值表中添加度量值
1. 在 [Azure 经典门户](http://manage.windowsazure.com/)中，打开云服务的“监视”页。
   
    默认情况下，度量值表显示可用度量值的子集。 下图中显示云服务的默认详细监视度量值，该度量值仅由 Memory\Available MBytes 性能计数器提供，并且数据在角色级别汇总。 使用“添加度量值”可选择要在 Azure 经典门户中监视的其他汇总和角色级别的度量值。
   
    ![详细显示](./media/cloud-services-how-to-monitor/CloudServices_DefaultVerboseDisplay.png)
2. 向度量值表中添加度量值：
   
   1. 单击“添加度量值”打开“选择度量值”，如下所示。
      
       将展开第一个可用度量值以显示可用选项。 对于每个度量值，最上面的选项显示所有角色的汇总监视数据。 此外，还可以选择要显示有关其数据的独立角色。
      
       ![添加度量值](./media/cloud-services-how-to-monitor/CloudServices_AddMetrics.png)
   2. 选择要显示的度量值
      
      * 单击度量值旁边的下拉箭头，展开监视选项。
      * 选中要显示的每个监视选项对应的复选框。
        
        度量值表中最多可显示 50 个度量值。
        
        > [!TIP]
        > 在详细监视中，度量值列表可以包含几十个度量值。 要显示滚动条，请将鼠标指针悬停于对话框右侧。 若要筛选列表，请单击搜索图标，并在搜索框中输入文本，如下所示。
        > 
        > 
        
        ![添加度量值搜索](./media/cloud-services-how-to-monitor/CloudServices_AddMetrics_Search.png)
3. 在选择度量值后，请单击“确定”（复选标记）。
   
    所选度量值将添加到度量值表中，如下所示。
   
    ![监控指标](./media/cloud-services-how-to-monitor/CloudServices_Monitor_UpdatedMetrics.png)
4. 要从度量值表中删除某个度量值，请单击该度量值以选中它，并单击“删除度量值”。 （在选择度量值后，只会看到“删除度量值”。）

### <a name="to-add-custom-metrics-to-the-metrics-table"></a>向度量值表中添加自定义度量值
“详细”监视级别提供了可在门户上监视的默认度量值的列表。 除此之外，还可通过门户监视应用程序定义的任意自定义度量值或性能计数器。

以下步骤假设已打开“详细”监视级别，并已配置好应用程序，可收集和传输自定义性能计数器。 

若要在门户中显示自定义性能计数器，需要更新 wad-control-container 中的配置：

1. 在诊断存储帐户中打开 wad-control-container Blob。 可以使用 Visual Studio 或任何其他存储资源管理器执行此操作。
   
    ![Visual Studio 服务器资源管理器](./media/cloud-services-how-to-monitor/CloudServices_Monitor_VisualStudioBlobExplorer.png)
2. 使用 **DeploymentId/RoleName/RoleInstance** 模式导航 Blob 的路径，以查找角色实例的配置。 
   
    ![Visual Studio 存储资源管理器](./media/cloud-services-how-to-monitor/CloudServices_Monitor_VisualStudioStorage.png)
3. 下载角色实例的配置文件，并更新该文件以包含所有自定义性能计数器。 例如，若要监视 C 盘的磁盘写入字节数/秒，请在 **PerformanceCounters\Subscriptions** 节点下添加以下内容
   
    ```xml
    <PerformanceCounterConfiguration>
    <CounterSpecifier>\LogicalDisk(C:)\Disk Write Bytes/sec</CounterSpecifier>
    <SampleRateInSeconds>180</SampleRateInSeconds>
    </PerformanceCounterConfiguration>
    ```
4. 保存所做的更改，然后将配置文件上传回到相同位置，并覆盖 Blob 中的现有文件。
5. 在 Azure 经典门户配置中切换到“详细”模式。 如果已处于“详细”模式，请切换到“最低”，再切换回到“详细”模式。
6. 自定义性能计数器即会出现在“添加度量值”对话框中。 

## <a name="how-to-customize-the-metrics-chart"></a>如何：自定义度量值图表
1. 在度量值表中，最多可选择 6 个要显示在度量值图表上的度量值。 要选择度量值，请单击其左侧的复选框。 若要从度量值图表中删除度量值，请在度量值表中清除其复选框。
   
    当选择度量值表中的度量值时，这些度量值将添加到度量值图表中。 在较窄的显示器上，由“更多 n 项”下拉列表包含显示器无法显示的度量值标题。
2. 若要切换显示相对值（仅显示每个度量值的最终值）和绝对值（显示 Y 轴），请在图表顶部选择“相对”或“绝对”。
   
    ![相对或绝对](./media/cloud-services-how-to-monitor/CloudServices_Monitor_RelativeAbsolute.png)
3. 若要更改度量值图表显示的时间范围，请在图表顶部选择 1 小时、24 小时或者 7 天。
   
    ![监视器显示时段](./media/cloud-services-how-to-monitor/CloudServices_Monitor_DisplayPeriod.png)
   
    在仪表板度量值图表中，显示度量值的方式是不同的。 将显示一组标准度量值，并且可通过选择度量值标题来添加或删除度量值。

### <a name="to-customize-the-metrics-chart-on-the-dashboard"></a>在仪表板上自定义度量值图表
1. 打开云服务的仪表板。
2. 从图表中添加或删除度量值：
   
   * 若要显示新的度量值，请在图表标题中选择该度量值对应的复选框。 在收缩视图中，单击“n 个度量值”旁的向下箭头会显示图表标题区域无法显示的度量值。
   * 要删除显示在图表上的某个度量值，请清除其标题旁的复选框。
   
3. 在“相对”和“绝对”视图之间切换。
4. 选择要显示 1 小时、24 小时还是 7 天的数据。

## <a name="how-to-access-verbose-monitoring-data-outside-the-azure-classic-portal"></a>如何：在 Azure 经典门户外部访问详细监视数据
详细监视数据存储在为每个角色指定的存储帐户中的相应表中。 对于每个云服务部署，将为角色创建 6 个表。 即，将为每 5 分钟、1 小时和 12 小时的数据创建 2 个表。 其中一个表存储角色级别的汇总数据；其他表存储角色实例的汇总数据。 

表名称采用以下格式：

```
WAD*deploymentID*PT*aggregation_interval*[R|RI]Table
```

其中：

* deploymentID 是分配给云服务部署的 GUID
* aggregation_interval = 5 分钟、1 小时或 12 小时
* R = 角色级别的汇总
* RI = 角色实例的汇总

例如，下表将存储在 1 小时时间间隔内汇总的详细监视数据。

```
WAD8b7c4233802442b494d0cc9eb9d8dd9fPT1HRTable (hourly aggregations for the role)

WAD8b7c4233802442b494d0cc9eb9d8dd9fPT1HRITable (hourly aggregations for role instances)
```
