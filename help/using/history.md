---
title: HTL 历史记录
description: 对于 AEM 的长期用户，本文档提供了 HTL 的背景知识、它如何取代 JSP，以及 Sightly 的名称更改。
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '540'
ht-degree: 40%

---


# HTL 历史记录 {#history-of-htl}

对于 AEM 的长期用户，本文档提供了 HTL 的背景知识、它如何取代 JSP，以及 Sightly 的名称更改。

## 以前称为 Sightly {#sightly}

HTML 模板语言 (HTL) 是 Adobe Experience Manager 中适用于 HTML 的首选和推荐的服务器端模板系统。 它取代 AEM 的以前版本中使用的 JSP (JavaServer Pages)。

## HTL 优于 JSP {#htl-over-jsp}

Adobe建议对新的AEM项目使用HTML模板语言。 原因是与JSP相比，它提供了多种好处。 但是，对于现有项目，只有在估计迁移比未来几年维护现有 JSP 的工作量少时，迁移才有意义。

迁移到HTL不一定是全有或全无的选择，因为使用HTL编写的组件与使用JSP或ESP编写的组件兼容。 此方法意味着现有项目可以毫无问题地将HTL用于新组件，同时为现有组件保留JSP。

甚至在同一个组件中，HTL 文件也可以与 JSP 和 ESP 一起使用。以下示例在&#x200B;**第1**&#x200B;行上显示如何从JSP文件中包括HTL文件，在&#x200B;**第2**&#x200B;行上显示如何从HTL文件中包括JSP文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## 常见问题解答 {#frequently-asked-questions}

刚开始接触HTL的经验丰富的AEM开发人员通常会提出以下问题：

### HTL 是否有 JSP 没有的任何限制？ {#limitations}

与JSP相比，HTL没有限制，因为可以使用JSP完成的任务使用HTL应该也能够实现。 但是，在多个方面，HTL在设计上比JSP更严格。 可以在单个JSP文件中实现的任务可能需要分离到Java类或JavaScript文件中才能在HTL中实现。 但是这种方法通常用于确保在逻辑和标记之间良好地分离关注点。

### HTL 是否支持 JSP 标记库？ {#tag-libraries}

不适用。 但是，如“快速入门”文档的[加载客户端库](getting-started.md#loading-client-libraries)部分中所示， template和call语句提供了类似的模式。

### 能否在 AEM 项目上扩展 HTL 功能？ {#extended}

不适用。 HTL 具有强大的扩展机制以重复使用逻辑 ([Use-API](#use-api-for-accessing-logic)) 和标记（template 和 call 语句），它们可用于实现项目代码的模块化。

### HTL 与 JSP 相比有何主要好处？ {#benefits}

安全性和项目效率是主要好处，在[概述](overview.md)中对此进行了详细介绍。

### JavaServer Pages (JSP)是否将消失？ {#go-away}

不适用。 没有中断JSP的计划。

## 名称中包含什么？ {#what-is-in-a-name}

在AEM 6.0和6.1中，HTL称为&#x200B;**Sightly**。 Adobe已将其重命名为&#x200B;**HTML模板语言**&#x200B;或&#x200B;**HTL**，以阐明规范的用途并大体符合Adobe的命名准则。 此命名更改自 2016 年 8 月起生效，适用于 AEM 6.0 版及更高版本。

>[!NOTE]
>
>此命名更改不会影响代码或API，因此兼容性不受影响。 有关详细信息，请[参考此公告视频](https://helpx.adobe.com/cn/experience-manager/how-to/announce-htl.html)。

要了解有关HTL的更多信息，请参阅[HTML模板语言(HTL)快速入门指南](overview.md)。
