---
title: "使用 Azure 数据工厂从/向 Salesforce 复制数据 | Microsoft Docs"
description: "了解如何通过在 Azure 数据工厂管道中使用复制活动，将数据从 Salesforce 复制到支持的接收器数据存储，或者从支持的源数据存储复制到 Salesforce。"
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
ms.date: 11/24/2017
ms.author: jingwang
ms.openlocfilehash: d5bad9a3be9c3165e5d26001353b8955ff81a764
ms.sourcegitcommit: aaba209b9cea87cb983e6f498e7a820616a77471
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/12/2017
---
# <a name="copy-data-fromto-salesforce-using-azure-data-factory"></a>使用 Azure 数据工厂从/向 Salesforce 复制数据
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [版本 1 - GA](v1/data-factory-salesforce-connector.md)
> * [版本 2 - 预览版](connector-salesforce.md)

本文概述如何使用 Azure 数据工厂中的复制活动从/向 Salesforce 复制数据。 它是基于概述复制活动总体的[复制活动概述](copy-activity-overview.md)一文。

> [!NOTE]
> 本文适用于目前处于预览状态的数据工厂版本 2。 如果使用数据工厂服务第 1 版（已正式推出 (GA)），请参阅 [V1 中的 Salesforce 连接器](v1/data-factory-salesforce-connector.md)。

## <a name="supported-capabilities"></a>支持的功能

可将数据从 Salesforce 复制到任何受支持的接收器数据存储，或者将数据从任何受支持的源数据存储复制到 Salesforce。 有关复制活动支持作为源/接收器的数据存储列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)表。

具体而言，Salesforce 连接器支持：

- 以下版本的 Salesforce：**开发人员版、专业版、企业版或不受限制版**。
- 从/向 Salesforce **生产、沙盒和自定义域**复制数据。

## <a name="prerequisites"></a>先决条件

* 在 Salesforce 中，必须启用 API 权限。 请参阅 [How do I enable API access in Salesforce by permission set?](https://www.data2crm.com/migration/faqs/enable-api-access-salesforce-permission-set/)（如何通过权限集启用 Salesforce 中的 API 访问权限？）

## <a name="salesforce-request-limits"></a>Salesforce 请求限制

Salesforce 对 API 请求总数和并发 API 请求均有限制。 请注意以下几点：

- 如果并发请求数超过限制，则将进行限制并且会看到随机失败。
- 如果请求总数超过限制，会阻止 Salesforce 帐户 24 小时。

在这两种情况下，还可能会收到“REQUEST_LIMIT_EXCEEDED”错误。 有关详细信息，请参阅 [Salesforce Developer Limits](http://resources.docs.salesforce.com/200/20/en-us/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf)（Salesforce 开发人员限制）文章中的"API 请求限制"一节。

## <a name="getting-started"></a>入门
可以使用 .NET SDK、Python SDK、Azure PowerShell、REST API 或 Azure 资源管理器模板创建包含复制活动的管道。 有关创建包含复制活动的管道的分步说明，请参阅[复制活动教程](quickstart-create-data-factory-dot-net.md)。

对于特定于 Salesforce 连接器的数据工厂实体，以下部分提供有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

Salesforce 链接的服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type |类型属性必须设置为：**Salesforce**  |是 |
| environmentUrl | 指定 Salesforce 实例的 URL。 <br> - 默认为 `"https://login.salesforce.com"`。 <br> - 要从沙盒复制数据，请指定 `"https://test.salesforce.com"`。 <br> - 要从自定义域复制数据，请指定 `"https://[domain].my.salesforce.com"`（以此为例）。 |否 |
| username |为用户帐户指定用户名。 |是 |
| password |指定用户帐户的密码。<br/><br/>可以选择将此字段标记为 SecureString 以将其安全地存储在 ADF 中，或在 Azure Key Vault 中存储密码，并允许复制活动在执行数据复制时从此处拉取 - 从[在 Key Vault 中存储凭据](store-credentials-in-key-vault.md)了解详细信息。 |是 |
| securityToken |为用户帐户指定安全令牌。 请参阅[获取安全令牌](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm)了解有关如何重置/获取安全令牌的说明。 若要了解有关安全令牌的一般信息，请参阅 [Security and the API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)（安全性和 API）。<br/><br/>可以选择将此字段标记为 SecureString 以将其安全地存储在 ADF 中，或在 Azure Key Vault 中存储安全令牌，并允许复制活动在执行数据复制时从此处拉取 - 从[在 Key Vault 中存储凭据](store-credentials-in-key-vault.md)了解详细信息。 |是 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如果未指定，则使用默认 Azure 集成运行时。 | 对于源为“No”，对于接收器为“Yes” |

>[!IMPORTANT]
>若要显式将数据复制到 Salesforce，请使用接近 Salesforce 的位置[创建 Azure IR](create-azure-integration-runtime.md#create-azure-ir)，并关联如下面示例中的链接服务。

**示例：将凭据存储在 ADF 中**

```json
{
    "name": "SalesforceLinkedService",
    "properties": {
        "type": "Salesforce",
        "typeProperties": {
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            },
            "securityToken": {
                "type": "SecureString",
                "value": "<security token>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**示例：在 Azure Key Vault 中存储凭据**

```json
{
    "name": "SalesforceLinkedService",
    "properties": {
        "type": "Salesforce",
        "typeProperties": {
            "username": "<username>",
            "password": {
                "type": "AzureKeyVaultSecret",
                "secretName": "<secret name of password in AKV>",
                "store":{
                    "referenceName": "<Azure Key Vault linked service>",
                    "type": "LinkedServiceReference"
                }
            },
            "securityToken": {
                "type": "AzureKeyVaultSecret",
                "secretName": "<secret name of security token in AKV>",
                "store":{
                    "referenceName": "<Azure Key Vault linked service>",
                    "type": "LinkedServiceReference"
                }
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 本部分提供 Salesforce 数据集支持的属性列表。

要从/向 Salesforce 复制数据，请将数据集的 type 属性设置为“SalesforceObject”。 支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | type 属性必须设置为：**SalesforceObject**  | 是 |
| objectApiName | 要从中检索数据的 Salesforce 对象名称。 | 对于源为“No”，对于接收器为“Yes” |

> [!IMPORTANT]
> 任何自定义对象均需要 API 名称的“__c”部分。

![数据工厂 - Salesforce 连接 - API 名称](media/copy-data-from-salesforce/data-factory-salesforce-api-name.png)

**示例：**

```json
{
    "name": "SalesforceDataset",
    "properties": {
        "type": "SalesforceObject",
        "linkedServiceName": {
            "referenceName": "<Salesforce linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "objectApiName": "MyTable__c"
        }
    }
}
```

>[!NOTE]
>为了向后兼容，从 Salesforce 中复制数据时，使用以前“RelationalTable”类型的数据集都将有效，而建议以切换到新的“SalesforceObject”类型。

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 数据集的 type 属性必须设置为：RelationalTable | 是 |
| tableName | 在 Salesforce 中表的名称。 | 否（如果指定了活动源中的“query”） |

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各个部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 Salesforce 源和接收器支持的属性列表。

### <a name="salesforce-as-source"></a>以 Salesforce 作为源

要从 Salesforce 复制数据，请将复制活动中的源类型设置为“SalesforceSource”。 复制活动**源**部分支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动源的 type 属性必须设置为：**SalesforceSource** | 是 |
| query |使用自定义查询读取数据。 可以使用 SQL-92 查询或 [Salesforce Object Query Language (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm)（Salesforce 对象查询语言 (SOQL)）查询。 例如：`select * from MyTable__c`。 | 否（如果指定了数据集中的“tableName”） |

> [!IMPORTANT]
> 任何自定义对象均需要 API 名称的“__c”部分。

![数据工厂 - Salesforce 连接 - API 名称](media/copy-data-from-salesforce/data-factory-salesforce-api-name-2.png)

**示例：**

```json
"activities":[
    {
        "name": "CopyFromSalesforce",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Salesforce input dataset name>",
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
                "type": "SalesforceSource",
                "query": "SELECT Col_Currency__c, Col_Date__c, Col_Email__c FROM AllDataType__c"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

>[!NOTE]
>为了向后兼容，从 Salesforce 中复制数据时，使用以前“RelationalSource”类型的复制源都将有效，而建议以切换到新的“SalesforceSource”类型。

### <a name="salesforce-as-sink"></a>以 Salesforce 作为接收器

要向 Salesforce 复制数据，请将复制活动中的接收器类型设置为“SalesforceSink”。 复制活动接收器部分中支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动接收器的 type 属性必须设置为：**SalesforceSink** | 是 |
| writeBehavior | 操作写入行为。<br/>允许的值为：**Insert** 和 **Upsert**。 | 否（默认值为 Insert） |
| externalIdFieldName | 更新插入操作的外部的 ID 字段名称。 指定的字段必须在 Salesforce 对象中，定义为“外部 Id 字段”和其相应的输入数据中不能有 NULL 值。 | 对于“Upsert”是必需的 |
| writeBatchSize | 每批中写入到 Salesforce 的数据行计数。 | 否（默认值为 5000） |
| ignoreNullValues | 指示是否忽略 null 值从输入数据期间写入操作。<br/>允许的值为：true 和 false。<br>- **true**：保留目标中的数据对象时进行更新插入/更新操作保持不变，并插入在执行插入操作时定义默认值。<br/>- **false**：执行 upsert/update 操作时为 NULL 更新目标对象中的数据和执行插入操作时插入 NULL 值。 | 否（默认值为 false） |

### <a name="example-salesforce-sink-in-copy-activity"></a>示例：复制活动中的 Salesforce 接收器

```json
"activities":[
    {
        "name": "CopyToSalesforce",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Salesforce output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SalesforceSink",
                "writeBehavior": "Upsert",
                "externalIdFieldName": "CustomerId__c",
                "writeBatchSize": 10000,
                "ignoreNullValues": true
            }
        }
    }
]
```

## <a name="query-tips"></a>查询提示

### <a name="retrieving-data-from-salesforce-report"></a>从 Salesforce 报表检索数据

可通过将查询指定为 `{call "<report name>"}` 从 Salesforce 报表检索数据。 示例：`"query": "{call \"TestReport\"}"`。

### <a name="retrieving-deleted-records-from-salesforce-recycle-bin"></a>从 Salesforce 回收站中检索删除的记录

若要从 Salesforce 回收站中检索软删除的记录，需要在查询中指定 **“IsDeleted = 1”**。 例如，

* 若要仅查询已删除的记录，请指定“select * from MyTable__c **where IsDeleted= 1**”
* 若要查询包括现有和已删除记录的所有记录，请指定“select * from MyTable__c **where IsDeleted = 0 or IsDeleted = 1**”

### <a name="retrieving-data-using-where-clause-on-datetime-column"></a>使用 DateTime 列上的 where 子句检索数据

当指定 SOQL 或 SQL 查询时，请注意 DateTime 的格式差异。 例如：

* **SOQL 示例**：`SELECT Id, Name, BillingCity FROM Account WHERE LastModifiedDate >= @{formatDateTime(pipeline().parameters.StartTime,'yyyy-MM-ddTHH:mm:ssZ')} AND LastModifiedDate < @{formatDateTime(pipeline().parameters.EndTime,'yyyy-MM-ddTHH:mm:ssZ')}`
* **SQL 示例**：`SELECT * FROM Account WHERE LastModifiedDate >= {ts'@{formatDateTime(pipeline().parameters.StartTime,'yyyy-MM-dd HH:mm:ss')}'} AND LastModifiedDate < {ts'@{formatDateTime(pipeline().parameters.EndTime,'yyyy-MM-dd HH:mm:ss')}'}"`

## <a name="data-type-mapping-for-salesforce"></a>Salesforce 的数据类型映射

从 Salesforce 复制数据时，以下映射用于从 Salesforce 数据类型映射到 Azure 数据工厂临时数据类型。 若要了解复制活动如何将源架构和数据类型映射到接收器，请参阅[架构和数据类型映射](copy-activity-schema-and-type-mapping.md)。

| Salesforce 数据类型 | 数据工厂临时数据类型 |
|:--- |:--- |
| 自动编号 |String |
| 复选框 |布尔 |
| 货币 |Double |
| 日期 |DateTime |
| 日期/时间 |DateTime |
| 电子邮件 |String |
| ID |String |
| 查找关系 |String |
| 多选择列表 |String |
| Number |Double |
| 百分比 |Double |
| 电话 |String |
| 选择列表 |String |
| 文本 |String |
| 文本区域 |String |
| 文本区域（长型值） |String |
| 文本区域（丰富） |String |
| 文本（加密） |String |
| 代码 |String |

## <a name="next-steps"></a>后续步骤
有关 Azure 数据工厂中复制活动支持作为源和接收器的数据存储列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)。