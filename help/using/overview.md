---
title: HTL 概述
description: 了解 AEM 如何支持 HTL（HTML 模板语言）以提供增强安全性的高效企业级 Web 框架。该框架让没有Java 知识的 HTML 开发人员能够更好地参与 AEM 项目。
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: 9735ca6ae09ca899c69edd9b63d2e85ad5d6f904
workflow-type: tm+mt
source-wordcount: '677'
ht-degree: 95%

---


# 概述 {#overview}

>[!TIP]
>
>**您是否考虑过AEM的Edge Delivery Services？**
>
>您可以继续将本文档中描述的方法用于现有项目。 但是，对于新项目，Adobe建议利用[Edge Delivery Services。](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-cloud-service/content/edge-delivery/overview)

HTML 模板语言 (HTL) 由 Adobe Experience Manager (AEM) 支持，旨在提供增强安全性的高效企业级 Web 框架。它还使没有Java 知识的 HTML 开发人员能够更好地参与 AEM 项目。

[在 AEM 6.0 中引入的HTML 模板语言 ](history.md)，是 AEM 中适用于 HTML 的首选和推荐的服务器端模板系统。 对于需要构建强大企业网站的 Web 开发者来说，HTML 模板语言有助于提高安全性和开发效率。

## 提高了安全性 {#increased-security}

HTML 模板语言 (HTL) 通过自动对所有输出变量应用上下文感知转义来增强站点安全性，使其比大多数其他模板系统更安全。HTL 使这种方法成为可能，因为它理解 HTML 语法，并由此根据表达式在标记中的位置调整表达式所需的转义。这种方法会导致置于 `href` 或 `src` 属性中表达式的转义与置于其他属性或其他位置中的表达式转义不同。

虽然使用 JSP 等模板语言可以实现同样的结果，但开发者必须手动确保将正确的转义应用于每个变量。 由于所应用转义中的单个遗漏或错误就可能足以造成跨站点脚本编制 (XSS) 漏洞，因此 Adobe 决定使用 HTL 自动执行这项任务。如果需要，开发者仍可以对表达式指定不同的转义，但是使用 HTL，默认行为更有可能与所需行为对应，从而降低出错的可能性。

## 简化了开发 {#simplified-development}

HTML 模板语言易于学习，且其功能经过刻意限制，可确保既简单又直接。它还拥有强大的机制来构造标记和调用逻辑，同时始终严格隔离标记和逻辑间的问题。HTL 是标准 HTML5，使用表达式和数据属性来注释具有动态行为的标记。这种方法保持了标记的有效性和可读性。表达式和数据属性的评估完全在服务器端完成，在客户端不可见，在客户端可以使用任何所需的 JavaScript 框架而不会产生干扰。

这些功能让没有Java 知识的 HTML 开发人员能够编辑 HTL 模板、融入开发团队并简化与全栈Java 开发人员的协作。反过来，这也让 Java 开发者能够专注于后端代码，而无需担心 HTML。

## 降低了成本 {#reduced-costs}

提高了安全性，简化了开发过程，同时加强了团队合作，这相应地减少了 AEM 项目的工作量，缩短了其上市时间 (TTM)，并降低了总体拥有成本 (TCO)。

使用 HTML 模板语言重新实现 Adobe.com 网站表明项目成本和工期减少了约 25%。

![效率提高，成本降低](assets/chlimage_1.png)

上图显示了 HTL 可能提供的以下效率改进：

* **HTML/CSS/JS：** HTML 开发人员可以直接编辑 HTL 模板，从而允许直接在 AEM 组件上实现前端设计，无需单独实现。这种方法减少了全栈 Java 开发者所面临的费时费力的迭代工作。
* **JSP / HTL：**&#x200B;因为 HTL 本身不需要任何 Java 知识并且可以直接编写，因此任何拥有 HTML 专业知识的开发者都可以编辑模板。
* **Java：**&#x200B;由于 HTL 提供的 Use-API 使用起来简单明了，因此明确了与业务逻辑的接口，这也有利于 Java 的整体开发。

## 视频介绍 {#video}

以下来自 [AEM Gems 会话的视频，](https://experienceleague.adobe.com/zh-hans/docs/events/experience-manager-gems-recordings/gems2014/aem-introduction-to-htl)概述了 HTL 的目的以及实施示例。

>[!VIDEO](https://video.tv.adobe.com/v/19504/?quality=9)

请注意，视频中是以[其原名 Sightly 指代 HTL](history.md)。

## 后续步骤 {#next-steps}

现在您已经了解了 HTL 的目标和优势，您可以开始使用该语言。请参见[HTML 模板语言快速入门](getting-started.md)。
