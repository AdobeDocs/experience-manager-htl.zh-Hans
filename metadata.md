---
solution: Experience Manager
type: Documentation
product: adobe experience manager
git-repo: https://git.corp.adobe.com/AdobeDocs/experience-manager-htl.zh-Hans
index: y
translation-type: tm+mt
source-git-commit: 5b88f6255534ef5af0958681c80303ab3da112b5
workflow-type: tm+mt
source-wordcount: '108'
ht-degree: 6%

---


# 内部使用的元数据

GitHub创作系统中的元数据是分层的，定义为以下不断增加的先例级别。

1. metadata.md
1. ToC
1. 文章

metadata.md文件中定义的元数据适用于整个回购，但可以在ToC和文章级别覆盖。 对元数据的任何覆盖都应在尽可能低的级别执行。

experience-manager-core-components.en repo中的元数据是最低要求。

metadata.md

* `product`
* `git-repo`
* `index: y`

不再使用：

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
* `index: n` （仅适用于早期版本的组件）

有关元数据的其他信息，请参阅[内部创作指南。](https://docs.adobe.com/help/en/collaborative-doc-instructions/collaboration-guide/markdown/metadata.html#solution-metadata)
