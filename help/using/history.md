---
title: HTL 历史记录
description: 对于 AEM 的长期用户，本文档提供了 HTL 的背景知识、它如何取代 JSP，以及 Sightly 的名称更改。
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: addc69e4b4e56a9b1c5f91ce9af26fa2d326d981
workflow-type: ht
source-wordcount: '530'
ht-degree: 100%

---


# HTL 历史记录 {#history-of-htl}

对于 AEM 的长期用户，本文档提供了 HTL 的背景知识、它如何取代 JSP，以及 Sightly 的名称更改。

## 以前称为 Sightly {#sightly}

HTML 模板语言 (HTL) 是 Adobe Experience Manager 中适用于 HTML 的首选和推荐的服务器端模板系统。 它取代 AEM 的以前版本中使用的 JSP (JavaServer Pages)。

## HTL 优于 JSP {#htl-over-jsp}

Adobe 建议对于新的 AEM 项目使用 HTML 模板语言。原因是与 JSP 相比，它提供了多种优势。但是，对于现有项目，只有在估计迁移比未来几年维护现有 JSP 的工作量少时，迁移才有意义。

迁移到 HTL 不一定是全有或全无的选择，因为使用 HTL 编写的组件与使用 JSP 或 ESP 编写的组件兼容。这种方法意味着现有项目可以将 HTL 用于新组件，同时为现有组件保留 JSP。

甚至在同一个组件中，HTL 文件也可以与 JSP 和 ESP 一起使用。下面的示例在&#x200B;**第 1 行**&#x200B;上显示如何从 JSP 文件中包括 HTL 文件，在&#x200B;**第 2 行**&#x200B;上显示如何从 HTL 文件中包括 JSP 文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## 常见问题解答 {#frequently-asked-questions}

刚接触 HTL 的经验丰富的 AEM 开发人员通常会问以下问题：

### HTL 是否有 JSP 没有的任何限制？ {#limitations}

与 JSP 相比，HTL 没有任何限制，因为使用 JSP 完成的任务使用 HTL 应该也能够实现。然而，HTL 在设计上在多个方面都比 JSP 更严格。在单个 JSP 文件中可以实现的功能可能需要分成 Java 类或 JavaScript 文件才能在 HTL 中实现。但这种方法通常用于确保在逻辑和标记之间良好地分离关注点。

### HTL 是否支持 JSP 标记库？ {#tag-libraries}

不会。但是，如”快速入门“文档中[加载客户端库](getting-started.md#loading-client-libraries)部分所示，模板和调用语句提供了类似的模式。

### 能否在 AEM 项目上扩展 HTL 功能？ {#extended}

不会。HTL 具有强大的扩展机制以重复使用逻辑 ([Use-API](#use-api-for-accessing-logic)) 和标记（template 和 call 语句），它们可用于实现项目代码的模块化。

### HTL 与 JSP 相比有何主要好处？ {#benefits}

安全性和项目效率是主要好处，在[概述](overview.md)中对此进行了详细介绍。

### JavaServer Pages（JSP）会消失吗？ {#go-away}

不会。目前没有停止 JSP 的计划。

## 名称中包含什么？ {#what-is-in-a-name}

在 AEM 6.0 和 6.1 中，HTL 被称为 **Sightly**。Adobe 已将其重命名为&#x200B;**HTML 模板语言**&#x200B;或&#x200B;**HTL**，以阐明规范的用途并大体符合 Adobe 的命名指南。此命名更改自 2016 年 8 月起生效，适用于 AEM 6.0 版及更高版本。

>[!NOTE]
>
>此命名更改不会影响代码或 API，因此兼容性不受影响。

<!-- LINK IS 404
For more information, watch [this announcement video](https://helpx.adobe.com/experience-manager/how-to/announce-htl.html). -->

要了解有关 HTL 的更多信息，请参阅 [HTML 模板语言 (HTL) 入门指南](overview.md)。
