---
title: AEM HTL概述
seo-title: AEM HTL技术文档概述。
description: AEM支持的HTL旨在提供高效的企业级Web框架，该框架可提高安全性，并允许没有Java知识的HTML开发人员更好地参与AEM项目。
seo-description: 本文档阐述了Adobe Experience manager支持的HTML模板语言- HTL的原则和用途。 HTL是一个高效的企业级Web框架，它提高了安全性，并允许没有Java知识的HTML开发人员更好地参与AEM项目。
uuid: 8f486325-0a1b-4186-a998-96fc0034c44a
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: 简介
content-type: 引用
discoiquuid: 8f779e08-94c7-43bc-a6e5-d81a9f818c5c
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 1d4565a4cffa6e5d42d6a5242f7ce62203dc7c63

---


# 概述 {#overview}

HTML模板语言(HTL)由Adobe Experience Manager(AEM)支持，其目的是提供高效的企业级Web框架，该框架可提高安全性，并允许不具备Java知识的HTML开发人员更好地参与AEM项目。

AEM 6.0中引入了HTML模板语言，它取代了JSP(JavaServer Pages)，成为HTML的首选和推荐的服务器端模板系统。 对于需要构建强大企业网站的Web开发人员，HTML模板语言有助于提高安全性和开发效率。

## 增强的安全性 {#increased-security}

与JSP和大多数其他模板系统相比，HTML模板语言提高了在实现中使用它的站点的安全性，因为HTL能够自动将正确的上下文感知型转义应用于输出到表示层的所有变量。 HTL之所以能够实现这一点，是因为它了解HTML语法，并使用该知识根据表达式在标记中的位置调整表达式所需的转义。 例如，这会导致放入或属性中的表达式与放 `href` 入其他属 `src` 性或其他位置的表达式发生不同的转义。

虽然使用JSP等模板语言可以实现相同的结果，但开发人员必须手动确保将正确的转义应用于每个变量。 由于应用的转义中的一个遗漏或错误可能足以导致跨站点脚本(XSS)漏洞，因此我们决定使用HTL自动执行此任务。 如果需要，开发人员仍可以在表达式上指定不同的转义，但是使用HTL，默认行为更有可能与所需行为对应，从而降低出错的可能性。

## 简化的开发 {#simplified-development}

HTML模板语言易于学习，其功能有意限制，以确保其简单直接。 它还具有强大的机制来构造标记和调用逻辑，同时始终强制在标记和逻辑之间严格地分隔问题。 HTL本身是标准HTML5，因为它使用表达式和数据属性用所需的动态行为对标记进行注释，这意味着它不会破坏标记的有效性并使其可读。 请注意，表达式和数据属性的评估完全在服务器端完成，在客户端将不可见，在客户端，可以使用任何所需的JavaScript框架而不会受到干扰。

这些功能使没有Java知识且产品特定知识少的HTML开发人员能编辑HTL模板，使他们成为开发团队的一部分，并简化了与整个Java开发人员的协作。 反之，这使Java开发人员能专注于后端代码，而无需担心HTML。

## 降低成本 {#reduced-costs}

增强的安全性、简化的开发和改进的团队协作，以更少的工作量为AEM项目提供转换，缩短上市时间(TTM)，并降低总体拥有成本(TCO)。

具体而言，使用HTML模板语言重新实施Adobe.com站点时观察到的情况是，项目的成本和持续时间可减少约25%。

![](assets/chlimage_1.png)

上图显示了HTL可能提高的以下效率：

* **** HTML/CSS/JS:由于HTML开发人员能够直接编辑HTL模板，因此前端设计不必再与AEM项目分开实施，而是可以直接在实际AEM组件上实施。 这减少了与整个堆栈的Java开发人员之间痛苦的迭代。
* **** JSP/HTL:由于HTL本身不需要任何Java知识并且可以直接编写，因此任何具有HTML专业知识的开发人员都可以编辑模板。
* **** Java:由于HTL提供的Use-API使用简单明了，所以明确了与业务逻辑的接口，从而有利于Java的整体开发。

**阅读下一页：**

* [HTML模板语言快速入门](getting-started.md)

