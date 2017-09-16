---
title: "Azure 搜索中的分析器 | Microsoft Docs"
description: "将分析器分配到索引中的可搜索文本字段，以将默认标准 Lucene 替换为自定义、预定义或特定语言替代项。"
services: search
manager: jhubbard
author: HeidiSteen
documentationcenter: 
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 09/03/2017
ms.author: heidist
ms.translationtype: HT
ms.sourcegitcommit: ce0189706a3493908422df948c4fe5329ea61a32
ms.openlocfilehash: 6e6c4491b8f66011340d1246495dbded7caf2903
ms.contentlocale: zh-cn
ms.lasthandoff: 09/05/2017

---

# <a name="analyzers-in-azure-search"></a>Azure 搜索中的分析器

分析器是[全文搜索处理](search-lucene-query-architecture.md)的组成部分，负责对索引和查询工作负荷进行文本到词元的转换。 在索引期间，分析器会将文本转换为词元化的词语，并将其写入索引。 在查询期间，分析器会对读取操作执行相同的转换（将文本转换为词元化的词语）。 

以下是分析过程中的典型转换：

+ 删除非必需字（非索引字）和标点。
+ 将短语和用连字符连接的词语分解为组成部分。
+ 这些词语采用小写形式。
+ 将单词分解为词根形式，以便查找匹配项，而不考虑时态。

Azure 搜索提供默认分析器。 可使用替换选项逐字段替代。 本文旨在介绍可用的选项，并提供向搜索操作添加分析器的最佳做法。 它还演示了主要方案的分析器配置示例。

## <a name="how-analysis-fits-into-full-text-search-processing"></a>分析如何适应全文搜索处理

分析器对查询分析器传入的词语输入进行操作，并返回分析后的词语，以添加到查询树对象。

 ![Azure 搜索中 Lucene 查询体系结构的关系图][1]

分析器仅适用于单个词语查询或短语查询。 分析器不适用于使用不完整术语的查询类型（前缀查询、通配符查询、正则表达式查询）或模糊查询。 对于这些查询类型，词语将绕过分析阶段直接添加到查询树中。 针对这些类型的查询字词执行的唯一转换操作是转换为小写。

## <a name="supported-analyzers"></a>支持的分析器

下表介绍了 Azure 搜索支持的分析器。

| 类别 | 说明 |
|----------|-------------|
| [标准 Lucene 分析器](https://lucene.apache.org/core/4_0_0/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html) | 默认。 自动用于索引和查询。 无需任何规范或配置。 这种通用分析器适用于大多数语言和方案。|
| 预定义分析器 | 作为按原样使用且具有有限自定义项的成品提供。 <br/>有两种类型：专用和语言特定。 由于按名称而非自定义项引用，因此被称为“预定义”。 <br/><br/>适用于需要专业处理或最小处理的文本输入的[专用（不限语言）分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search#AnalyzerTable)。 非语言预定义分析器包括 Asciifolding、Keyword、Pattern、Simple、Stop 和 Whitespace。<br/><br/>[语言分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)为多种语言提供丰富的语言支持。 Azure 搜索支持 35 种 Lucene 语言分析器和 50 种 Microsoft 自然语言处理分析器。 |
|[自定义分析器](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search) | 一种结合了现有元素的用户定义配置，由一个分词器（必需）和可选的筛选器（字符或词元）组成。|

可将预定义分析器（如 Pattern 或 Stop）自定义为使用[预定义分析器引用](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search#AnalyzerTable)中所述的替换选项。 可供设置选项的预定义分析器只有几个。 对于任何自定义项，请为你的新配置提供一个名称，例如 myPatternAnalyzer，以便将其与 Lucene Pattern 分析器区别开。

## <a name="how-to-specify-analyzer"></a>如何指定分析器

1. 仅适用于自定义分析器，在索引中创建一个 `analyzer` 定义。 有关详细信息，请参阅[创建索引](https://docs.microsoft.com/rest/api/searchservice/create-index)及[自定义分析器 > 创建](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search#create-a-custom-analyzer)。

2. 对于要使用分析器的每个字段，请在[索引中的字段定义](https://docs.microsoft.com/rest/api/searchservice/create-index)上，将 `analyzer` 属性设置为目标分析器的名称。 有效值包括预定义分析器、语言分析器，或先前在索引架构中定义的自定义分析器。

 如果不使用 `analyzer` 属性，还可以使用 `indexAnalyzer` 和 `searchAnalyzer` 字段参数为索引和查询设置不同分析器。 

3. 重新生成索引，调用新的文本处理行为。

## <a name="best-practices"></a>最佳实践

本部分提供更有效使用分析器的相关建议。

### <a name="one-analyzer-for-read-write-unless-you-have-specific-requirements"></a>除非特别要求，否则读写操作使用一个分析器

Azure 搜索允许通过附加的 `indexAnalyzer` 和 `searchAnalyzer` 字段参数来指定使用不同的分析器执行索引和搜索。 如果未指定，使用设置分析器`analyzer`属性用于索引编制和搜索。 如果未指定 `analyzer`，将使用默认标准 Lucene 分析器。

除非特别要求，否则索引和查询一般使用同一个分析器。 通常而言，如果创建词元的分析器与稍后在查询时用于查找词元的分析器相同，则效率会更高。 

### <a name="test-during-active-development"></a>在活动开发中进行测试

替换标准分析器需要重新生成索引。 如果可能，请先决定在活动开发中使用的分析器，然后再将索引应用到生产中。

### <a name="compare-analyzers-side-by-side"></a>并排比较分析器

建议使用[分析 API](https://docs.microsoft.com/rest/api/searchservice/test-analyzer)。 响应包含词元化的词语，由特定分析器针对提供的文本生成。 

> [!Tip]
> [搜索分析器演示](http://alice.unearth.ai/)演示了对标准 Lucene 分析器、Lucene 英语分析器和 Microsoft 英语自然语言处理器的并排比较。 对于提供的每个搜索输入，每个分析器的结果将显示在相邻窗格中。

## <a name="examples"></a>示例

下方示例演示了几个主要方案的分析器定义。

<a name="Example1"></a>
### <a name="example-1-custom-options"></a>示例 1：自定义选项

本示例说明了具有自定义选项的分析器定义。 字符筛选器、分词器和词元筛选器的自定义选项分别指定为命名构造，然后在分析器定义中引用。 预定义元素按原样使用，并可通过名称轻松引用。

分析此示例：

* 分析器是可搜索字段的字段类的属性。
* 自定义分析器是索引定义的一部分。 它支持轻度自定义（例如，在某个筛选器中自定义一个单独选项）或在多个位置自定义。
* 在这种情况下，自定义分析器为“my_analyzer”，它将反过来使用自定义的标准分词器“my_standard_tokenizer”和两个词元筛选器：小写的自定义 asciifolding 筛选器“my_asciifolding”。
* 它还定义一个自定义“map_dash”字符筛选器，以便在词汇切分前将所有短划线替换为下划线（标准分词器在短划线而非下划线上中断）。

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"my_analyzer"
        }
     ],
     "analyzers":[
        {
           "name":"my_analyzer",
           "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
           "charFilters":[
              "map_dash"
           ],
           "tokenizer":"my_standard_tokenizer",
           "tokenFilters":[
              "my_asciifolding",
              "lowercase"
           ]
        }
     ],
     "charFilters":[
        {
           "name":"map_dash",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["-=>_"]
        }
     ],
     "tokenizers":[
        {
           "name":"my_standard_tokenizer",
           "@odata.type":"#Microsoft.Azure.Search.StandardTokenizer",
           "maxTokenLength":20
        }
     ],
     "tokenFilters":[
        {
           "name":"my_asciifolding",
           "@odata.type":"#Microsoft.Azure.Search.AsciiFoldingTokenFilter",
           "preserveOriginal":true
        }
     ]
  }
~~~~

<a name="Example2"></a>
### <a name="example-2-override-the-default-analyzer"></a>示例 2：替代默认分析器

默认为标准分析器。 假设你希望将默认分析器替换为其他预定义分析器（如 Pattern 分析器）。 如果尚未设置自定义选项，则只需在字段定义中通过名称来指定它。

“分析器”元素逐字段替代标准分析器。 不能全局替代。 在本例中，`text1` 使用 Pattern 分析器和 `text2`，它不指定分析器，仅使用默认值。

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text1",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"pattern"
        },
        {
           "name":"text2",
           "type":"Edm.String",
           "searchable":true
        }
     ]
  }
~~~~

<a name="Example3"></a>
### <a name="example-3-different-analyzers-for-indexing-and-search-operations"></a>示例 3：用于索引和搜索操作的不同分析器

预览 API 包括为索引和搜索指定不同分析器的其他索引属性。 `searchAnalyzer` 和 `indexAnalyzer` 属性必须成对指定，并替换单个 `analyzer` 属性。


~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "indexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
     ],
  }
~~~~

<a name="Example4"></a>
### <a name="example-4-language-analyzer"></a>示例 4：语言分析器

包含其他语言字符串的字段可使用语言分析器，而其他字段将保留默认值（或使用某些其他预定义或自定义分析器）。 如果使用语言分析器，则必须将其同时用于索引和搜索操作。 使用语言分析器的字段不得对索引和搜索使用不同的分析器。

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "IndexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
        {
           "name":"text_fr",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"fr.lucene"
        }
     ],
  }
~~~~

## <a name="next-steps"></a>后续步骤

+ 请参阅 [Azure 搜索中全文搜索的工作原理](search-lucene-query-architecture.md)获取全面的说明。 本文通过示例来说明某些表面上看起来与预期不符的行为。

+ 通过[搜索文档](https://docs.microsoft.com/rest/api/searchservice/search-documents#examples)示例部分或者通过门户中“搜索资源管理器”的[简单查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)尝试其他查询语法。

+ 了解如何应用[语言特定的词法分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)。

+ [配置自定义分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)，针对单个字段尽量简化处理或者进行专门处理。

+ 在此演示网站的相邻窗格中[比较标准和英语分析器](http://alice.unearth.ai/)。 

## <a name="see-also"></a>另请参阅

 [搜索文档 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 

 [简单的查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) 

 [完整 Lucene 查询语法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search) 
 
 [处理搜索结果](https://docs.microsoft.com/azure/search/search-pagination-page-layout)

<!--Image references-->
[1]: ./media/search-lucene-query-architecture/architecture-diagram2.png
