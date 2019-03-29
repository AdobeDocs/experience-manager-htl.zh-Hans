---
title: 概述
seo-title: 概述
description: AEM支持HTL的目的是提供一个高效的企业级Web框架，它可以提高安全性，并允许不带Java知识的HTML开发人员更好地参加AEM项目。
seo-description: HTML模板语言- HTL(由Adobe Experience Manager支持)旨在提供高效的企业级Web框架，它可以提高安全性，并允许不带Java知识的HTML开发人员更好地参加AEM项目。
uuid: 8f 486325-a1 b-4186-a998-96fc0034 c44 a
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/HTL
topic-tags: introduction
content-type: 引用
discoiquuid: 8f779e08-94c7-43bc-a6 e1-d81 a9 f818 c5 c
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 796c55d3d85e6b5a3efaa5c04a25be1b0b4e54dd

---


# 概述 {#overview}

HTML模板语言(HTL)是由Adobe Experience Manager(AEM)提供支持的，它旨在提供高效的企业级Web框架，从而提高安全性，并允许没有Java知识的HTML开发人员更好地参与AEM项目。

HTML模板语言已在AEM6.0中引入，并取代JSP(JavaServer页面)作为HTML首选的服务器端模板系统。对于需要构建可靠的企业网站的Web开发人员，HTML模板语言有助于提高安全性和开发效率。

## 增强的安全性 {#increased-security}

与JSP和大多数其他模板系统一样，HTML模板语言可提高在实施中使用它的站点的安全性，因为HTL能够自动将适当的上下文感知转义应用到输出到表示层的所有变量。HTL使这种情况成为可能，因为它理解HTML语法，并使用该知识根据它们在标记中的位置调整表达式所需的转义。这将导致放置置入 `href` 或 `src` 属性的表达式与放置在其他属性中的表达式或其他属性的转义不同。

尽管可以使用JSP等模板语言实现相同的结果，但是开发者必须手动确保对每个变量应用正确的转义。由于应用的转义可能足以导致跨站点脚本(XSS)漏洞，因此我们决定使用HTL自动执行此任务。如果需要，开发人员仍然可以在表达式上指定不同的转义，但使用HTL的默认行为更有可能与所需行为相对应，从而降低出错概率。

## 简化的开发 {#simplified-development}

HTML模板语言易于学习，其功能仅限于确保其简洁明了。它还具有用于标记标记和调用逻辑的强大机制，同时强制将注意力严格分离到标记和逻辑之间。HTL本身是标准HTML5，因为它使用表达式和数据属性注释标记，用所需的动态行为注释标记，这意味着它不会破坏标记的有效性并使其可读。请注意，对表达式和数据属性的评估完全在服务器端完成，在客户端不可见，在客户端可以使用任何所需的JavaScript框架。

这些功能使HTML开发人员无需Java知识，并且没有任何产品特定知识，可以编辑HTL模板，使他们成为开发小组的一部分，并简化与完全堆栈Java开发人员的协作。反之亦然，Java开发人员无需担心HTML就能专注于后端代码。

## 降低成本 {#reduced-costs}

更高的安全性、简化的开发和改进的团队协作，可减少工作量、缩短上市时间(TTM)以及降低总拥有成本(TCO)，从而提高团队协作的效率。

根据使用HTML模板语言重新实施Sites站点时观察到的情况，项目的成本和持续时间可减少约25%。

![](assets/chlimage_1.png)

上图显示了HTL可能带来的更高效率的后续改进：

* **HTML/CSS/JS：** 由于HTML开发人员能够直接编辑HTL模板，因此前端设计不必再单独与AEM项目实现，但可以直接在实际AEM组件上实现。这减少了与完全堆叠的Java开发人员的痛苦迭代。
* **JSP/HTL：** 由于HTL本身不需要任何Java知识并直接转发，因此任何具有HTML专业知识的开发者都可以编辑模板。
* **Java：** 由于使用HTL提供的Use-API清晰易用，因此明确了与业务逻辑的接口，这也有利于Java开发整体。

**阅读下一步：**

* [HTML模板语言入门](getting-started.md)

