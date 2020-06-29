---
product: Adobe Experience Manager
git-repo: https://git.corp.adobe.com/AdobeDocs/experience-manager-htl.zh-Hans
index: y
solution-title: HTL学习和支持
solution-hub-url: https://docs.adobe.com/content/help/zh-Hans/experience-manager-cloud-service/sites/home.html
getting-started-title: AEM开发入门
getting-started-url: https://docs.adobe.com/content/help/zh-Hans/experience-manager-cloud-service/core-concepts/home.html
tutorials-title: AEM 教程
tutorials-url: https://docs.adobe.com/content/help/en/experience-manager-learn/cloud-service/overview.html
translation-type: tm+mt
source-git-commit: d3426d87dce09ac34ff1aca431ff2bfad2f7134a
workflow-type: tm+mt
source-wordcount: '139'
ht-degree: 20%

---


# 元数据供内部使用

GitHub创作系统中的元数据是分层的，定义为以下不断增加的先例级别。

1. metadata.md
1. ToC
1. 文章

metadata.md文件中定义的元数据适用于整个回购区，但可以在ToC和文章级别覆盖。 对元数据的任何覆盖都应在尽可能低的级别执行。

experience-manager-core-components.en repo中的元数据是最低要求。

metadata.md

* `product`
* `git-repo`
* `index: y`
* `solution-title`
* `solution-hub-url`
* `getting-started-title`
* `getting-started-url`
* `tutorials-title`
* `tutorials-url`

ToCs

* `sub-product`
* `user-guide-title`

文章

* `title`
* `description`
* `index: n` （仅适用于以前版本的组件）

有关元数据的其他信息，请参阅内 [部创作指南。](https://docs.adobe.com/help/en/collaborative-doc-instructions/collaboration-guide/markdown/metadata.html#solution-metadata)
