---
title: HTL 规范
description: 有关详细语法信息，请参阅HTL规范。
exl-id: c0657476-4db6-4fad-ad87-9252b5003237
source-git-commit: 942c5249cc38e49a7d617ba4cf65a878f2e229ab
workflow-type: tm+mt
source-wordcount: '152'
ht-degree: 6%

---


# HTL 规范 {#htl-specification}

HTML模板语言(HTL)是首选和推荐的服务器站点模板系统，用于HTML。

## HTL图层 {#layers}

AEM中使用的HTL可由多个层定义。

1. **[HTL规范](https://github.com/adobe/htl-spec)** - HTL是一个与平台无关的开源规范，任何人都可以免费实施该规范。 其规范在其GitHub存储库中进行维护。
1. **[Sling HTL脚本引擎](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html)** - Sling项目已创建HTL的引用实施，供AEM使用。 其文档由Sling项目维护。
1. **[AEM扩展](aem-extensions.md)** - AEM以Sling HTL脚本引擎为基础构建，以便为开发人员提供特定于AEM的便捷功能。 这些扩展将作为此文档集的一部分进行记录。

请按照上面链接，找到AEM所用所有HTL层的专用文档。
