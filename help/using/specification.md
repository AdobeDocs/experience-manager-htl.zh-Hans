---
title: HTL 规范
description: 有关详细语法信息，请参阅 HTL 规范。
exl-id: c0657476-4db6-4fad-ad87-9252b5003237
index: false
source-git-commit: 391c5279f0021dbedaffb0c63e67e037d6c782e1
workflow-type: tm+mt
source-wordcount: '135'
ht-degree: 85%

---


# HTL 规范 {#htl-specification}

HTML 模板语言 (HTL) 是适用于 HTML 的首选和推荐的服务器站点模板系统。

## HTL 图层 {#layers}

您可以通过多个层在 AEM 中定义 HTL。

1. **[HTL 规范](https://github.com/adobe/htl-spec)** – HTL 是一个开源、不依赖于平台的规范，任何人都可以自由实施。 其规范保存在 GitHub 存储库中。
1. **[Sling HTL 脚本引擎](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html)** – `Sling` 项目创建了 HTL 的参考实施，供 AEM 使用。 `Sling` 项目维护其文档。
1. **[AEM扩展](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-htl/content/aem-extensions)** - AEM构建在`Sling` HTL脚本引擎之上，以便为开发人员提供AEM特有的方便功能。 这些扩展作为本文档集的一部分进行记录。

按照以上链接，查阅 AEM 使用的所有 HTL 层的专用文档。
