---
title: HTL 历史记录
description: 对于 AEM 的长期用户，本文档提供了 HTL 的背景知识、它如何取代 JSP，以及 Sightly 的名称更改。
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: 88edbd2fd66de960460df5928a3b42846d32066b
workflow-type: tm+mt
source-wordcount: '544'
ht-degree: 100%

---


# HTL 历史记录 {#history-of-htl}

对于 AEM 的长期用户，本文档提供了 HTL 的背景知识、它如何取代 JSP，以及 Sightly 的名称更改。

## 以前称为 Sightly {#sightly}

HTML 模板语言 (HTL) 是 Adobe Experience Manager 中适用于 HTML 的首选和推荐的服务器端模板系统。 它取代 AEM 的以前版本中使用的 JSP (JavaServer Pages)。

## HTL 优于 JSP {#htl-over-jsp}

建议新的 AEM 项目使用 HTML 模板语言，因为与 JSP 相比，它可提供多种好处。但是，对于现有项目，只有在估计迁移比未来几年维护现有 JSP 的工作量少时，迁移才有意义。

但迁移到 HTL 不一定是全有或全无的选择，因为使用 HTL 编写的组件与使用 JSP 或 ESP 编写的组件兼容。这意味着现有项目可以毫无问题地将 HTL 用于新组件，同时为现有组件保留 JSP。

甚至在同一个组件中，HTL 文件也可以与 JSP 和 ESP 一起使用。以下示例在&#x200B;**第 1 行**&#x200B;上显示如何从 JSP 文件中包括 HTL 文件，在&#x200B;**第 2 行**&#x200B;上显示如何从 HTL 文件中包括 JSP 文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## 常见问题解答 {#frequently-asked-questions}

这些是 HTL 新手中经验丰富的 AEM 开发人员经常提出的问题。

### HTL 是否有 JSP 没有的任何限制？ {#limitations}

与 JSP 相比，HTL 真的没有限制，因为可以使用 JSP 完成的任务使用 HTL 应该也能够实现。 但是，在多个方面，HTL 在设计上比 JSP 更严格，并且在单个 JSP 文件中可以全部实现的任务可能需要分离到 Java 类或 JavaScript 文件中才能在 HTL 中实现。 但这通常用于确保在逻辑和标记之间良好地分离关注点。

### HTL 是否支持 JSP 标记库？ {#tag-libraries}

不支持，但如”快速入门“文档中[加载客户端库](getting-started.md#loading-client-libraries)部分所示，template 和 call 语句提供了类似模式。

### 能否在 AEM 项目上扩展 HTL 功能？ {#extended}

不，不能。 HTL 具有强大的扩展机制以重复使用逻辑 ([Use-API](#use-api-for-accessing-logic)) 和标记（template 和 call 语句），它们可用于实现项目代码的模块化。

### HTL 与 JSP 相比有何主要好处？ {#benefits}

安全性和项目效率是主要好处，在[概述](overview.md)中对此进行了详细介绍。

### JSP 最终会消失吗？ {#go-away}

还没有这些方面的计划。

## 名称中包含什么？ {#what-is-in-a-name}

在 AEM 6.0 和 6.1 中，HTL 称为 **Sightly**。 Adobe 已将其重命名为&#x200B;**“HTML 模板语言”**&#x200B;或&#x200B;**”HTL“**，以阐明规范的用途并大体符合 Adobe 的命名指南。此命名更改自 2016 年 8 月起生效，适用于 AEM 6.0 版及更高版本。

>[!NOTE]
>
>此命名更改不会影响代码或 API，因此兼容性不受影响。有关更多信息，请[参考此公告视频](https://helpx.adobe.com/cn/experience-manager/how-to/announce-htl.html)。

要了解有关 HTL 的更多信息和良好起点，请参阅我们官方的 [HTML 模板语言 (HTL) 快速入门指南。](overview.md)
