---
title: HTL 概述
description: 了解AEM如何支持HTL(HTML模板语言)以提供可增强安全性的高效企业级Web框架。 通过此框架，不了解Java的HTML开发人员可以更好地参与AEM项目。
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '645'
ht-degree: 26%

---


# 概述 {#overview}

HTML模板语言(HTL)由Adobe Experience Manager (AEM)提供支持，旨在提供可增强安全性的高效企业级Web框架。 它还可让不了解Java的HTML开发人员更好地参与AEM项目。

[在AEM 6.0](history.md)中引入了HTML模板语言，它是首选和推荐的服务器端模板系统，可用于AEM中的HTML。 对于需要构建强大企业网站的 Web 开发者来说，HTML 模板语言有助于提高安全性和开发效率。

## 提高了安全性 {#increased-security}

HTML模板语言(HTL)通过自动将上下文感知型转义应用于所有输出变量来增强站点安全性，使其比大多数其他模板系统更安全。 HTL之所以能实现这种方法，是因为它理解HTML语法，并由此根据表达式在标记中的位置调整表达式所需的转义。 此方法可能导致置于`href`或`src`属性中的表达式转义与置于其他属性或其他位置的表达式转义不同。

虽然使用 JSP 等模板语言可以实现同样的结果，但开发者必须手动确保将正确的转义应用于每个变量。 由于所应用转义中的单个遗漏或错误就可能足以造成跨站点脚本(XSS)漏洞，因此Adobe决定使用HTL自动完成此任务。 如果需要，开发者仍可以对表达式指定不同的转义，但是使用 HTL，默认行为更有可能与所需行为对应，从而降低出错的可能性。

## 简化了开发 {#simplified-development}

HTML 模板语言易于学习，且其功能经过刻意限制，可确保既简单又直接。它还拥有强大的机制来构造标记和调用逻辑，同时始终严格隔离标记和逻辑间的问题。HTL是标准HTML5，使用表达式和数据属性通过动态行为对标记进行注释。 此方法可保持标记的有效性和可读性。 表达式和数据属性的求值完全在服务器端完成，在客户端不可见，客户端可以不受干扰地使用任何所需的JavaScript框架。

利用这些功能，不了解Java的HTML开发人员可以编辑HTL模板、集成到开发团队中，并简化与全栈Java开发人员的协作。 反过来，它让Java开发人员能够专注于后端代码，而无需担心HTML。

## 降低了成本 {#reduced-costs}

提高了安全性，简化了开发过程，同时加强了团队协作，这相应地减少了AEM项目的工作量，缩短了其上市时间(TTM)，并降低了总体拥有成本(TCO)。

使用HTML模板语言重新实施Adobe.com站点表明，项目成本和持续时间最多可减少约25%。

![效率提高，成本降低](assets/chlimage_1.png)

上图显示了HTL可能提供的以下效率改进：

* **HTML/CSS/JS：** HTML开发人员可以直接编辑HTL模板，从而允许在AEM组件上直接实施前端设计，而无需单独的实施。 这种方法可减少全栈Java开发人员所面临的费时费力的迭代工作。
* **JSP / HTL：**&#x200B;由于HTL本身不需要任何Java知识并且可以直接编写，因此任何拥有HTML专业知识的开发人员都可以编辑模板。
* **Java：**&#x200B;由于 HTL 提供的 Use-API 使用起来简单明了，因此明确了与业务逻辑的接口，这也有利于 Java 的整体开发。

## 视频介绍 {#video}

以下来自[AEM Gems会话](https://experienceleague.adobe.com/en/docs/events/experience-manager-gems-recordings/gems2014/aem-introduction-to-htl)的视频概述了HTL的目的以及实施示例。

>[!VIDEO](https://video.tv.adobe.com/v/19504/?quality=9)

请注意，视频通过[其以前的名称Sightly](history.md)引用HTL。

## 后续步骤 {#next-steps}

现在您已经了解了HTL的目的和优势，您可以开始使用该语言了。 请参阅[HTML模板语言快速入门](getting-started.md)。
