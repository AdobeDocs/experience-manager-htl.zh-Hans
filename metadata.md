---
solution: Experience Manager
type: Documentation
product: adobe experience manager
git-repo: https://git.corp.adobe.com/AdobeDocs/experience-manager-htl.zh-Hans
index: y
source-git-commit: 89b9e89254f341e74f1a5a7b99735d2e69c8a91e
workflow-type: tm+mt
source-wordcount: '107'
ht-degree: 91%

---


# 内部使用的元数据

GitHub 创作系统中的元数据为层级式的，并定义了以下相对于前一项的递增级别。

1. metadata.md
1. ToC
1. 文章

在 metadata.md 文件中的定义的元数据应用到整个存储库，但可以在 ToC 和文章级别覆盖。任何覆盖元数据的操作应在尽可能最低的级别进行。

experience-manager-core-components.en 存储库中的元数据是最低要求。

metadata.md

* `product`
* `git-repo`
* `index: y`

已不再使用：

* `solution-title`
* `solution-hub-url`
* `getting-started-title`
* `getting-started-url`
* `tutorials-title`
* `tutorials-url`

ToC

* `sub-product`
* `user-guide-title`

文章

* `title`
* `description`
* `index: n`（仅适用于组件的以前版本）

有关元数据的其他信息可在[内部创作指南](https://experienceleague.adobe.com/docs/authoring-guide-exl/using/authoring/features/metadata.html#solution)中找到。
