---
title: HTL历史
description: 对于AEM的长期用户，本文档介绍了HTL的背景、它如何替换JSP以及名称从Sightly更改。
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '542'
ht-degree: 45%

---


# HTL历史 {#history-of-htl}

对于AEM的长期用户，本文档介绍了HTL的背景、它如何替换JSP以及名称从Sightly更改。

## 以前称为 Sightly {#sightly}

HTML模板语言(HTL)是在Adobe Experience Manager中HTML的首选和推荐的服务器端模板系统。 它取代 AEM 的以前版本中使用的 JSP (JavaServer Pages)。

## HTL 优于 JSP {#htl-over-jsp}

建议新的 AEM 项目使用 HTML 模板语言，因为与 JSP 相比，它可提供多种好处。但是，对于现有项目，只有在估计迁移比未来几年维护现有 JSP 的工作量少时，迁移才有意义。

但迁移到 HTL 不一定是全有或全无的选择，因为使用 HTL 编写的组件与使用 JSP 或 ESP 编写的组件兼容。这意味着现有项目可以毫无问题地将 HTL 用于新组件，同时为现有组件保留 JSP。

甚至在同一个组件中，HTL 文件也可以与 JSP 和 ESP 一起使用。以下示例在&#x200B;**第 1 行**&#x200B;上显示如何从 JSP 文件中包括 HTL 文件，在&#x200B;**第 2 行**&#x200B;上显示如何从 HTL 文件中包括 JSP 文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## 常见问题解答 {#frequently-asked-questions}

这些是经验丰富的AEM开发人员对HTL不熟悉的一些常见问题。

### HTL 是否有 JSP 没有的任何限制？ {#limitations}

与JSP相比，HTL实际上没有任何限制，因为使用JSP可以执行的操作也应该可以通过HTL实现。 但是，HTL在设计上比JSP在设计上要严格。可以在单个JSP文件中实现的所有内容，可能需要将分隔为Java类或JavaScript文件，才能在HTL中实现。 但这通常用于确保在逻辑和标记之间良好地分离关注点。

### HTL 是否支持 JSP 标记库？ {#tag-libraries}

否，但如 [正在加载客户端库](getting-started.md#loading-client-libraries) 在快速入门文档的部分中，template和call语句提供了类似的模式。

### 能否在 AEM 项目上扩展 HTL 功能？ {#extended}

不，他们不能。 HTL具有强大的扩展机制，可重复使用逻辑( [Use-API](#use-api-for-accessing-logic))和标记（模板和调用语句），用于模化项目代码。

### HTL 与 JSP 相比有何主要好处？ {#benefits}

安全性和项目效率是主要优势，详见 [概述。](overview.md)

### JSP 最终会消失吗？ {#go-away}

没有这样的计划。

## 名字是什么？ {#what-is-in-a-name}

在AEM 6.0和6.1中，HTL称为 **Sightly**. Adobe将其重命名为 **HTML模板语言** 或 **HTL** 以阐明规范的用途，并总体上与Adobe的命名准则保持一致。 此命名更改自 2016 年 8 月起生效，适用于 AEM 6.0 版及更高版本。

>[!NOTE]
>
>此命名更改不会影响代码或 API，因此兼容性不受影响。欲知详情，请 [请参阅此公告视频。](https://helpx.adobe.com/cn/experience-manager/how-to/announce-htl.html)

要进一步了解HTL，并找到一个很好的入手点，我们的官员 [HTML模板语言(HTL)入门指南。](overview.md)
