---
title: HTL 规范
description: 有关详细语法信息，请参阅 HTL 规范。
exl-id: c0657476-4db6-4fad-ad87-9252b5003237
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '138'
ht-degree: 73%

---


# HTL 规范 {#htl-specification}

HTML 模板语言 (HTL) 是适用于 HTML 的首选和推荐的服务器站点模板系统。

## HTL 图层 {#layers}

您可以在AEM中通过许多层定义HTL。

1. **[HTL 规范](https://github.com/adobe/htl-spec)** – HTL 是一个开源、不依赖于平台的规范，任何人都可以自由实施。 其规范保存在 GitHub 存储库中。
1. **[Sling HTL 脚本引擎](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html)** – Sling 项目创建了 HTL 的参考实施，供 AEM 使用。 Sling项目会维护其文档。
1. **[AEM扩展](aem-extensions.md)** - AEM构建在Sling HTL脚本引擎之上，为开发人员提供特定于AEM的方便功能。 这些扩展作为本文档集的一部分进行记录。

按照以上链接，查阅 AEM 使用的所有 HTL 层的专用文档。
