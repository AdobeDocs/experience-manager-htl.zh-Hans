---
title: HTL Use-API
seo-title: Adobe HTL Use-API
description: 有两个 API 可用于 HTL - 使用 Java 的 API 和使用 Javascript 的 API
seo-description: 有两个 API 可用于 Adobe HTL - 使用 Java 的 API 和使用 Javascript 的 API
uuid: ab44aa5c-ce7e-40b9-97fb-e86c6a28405c
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 89004426-eb59-4b63-913f-51bf98662773
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 5cbaf9c747acf748d12559c2c8e3aba4600cf9a4

---


# HTL Use-API {#htl-use-api}

下表概述了每个API的利弊。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **专业人员** | <ul><li>更快</li><li>可以使用调试器进行检查</li><li>易于单元测试</li></ul> | <ul><li>可由前端开发人员修改</li><li>位于组件中，使组件的视图逻辑与相应的模板保持接近</li></ul> |
| **缺点** | <ul><li>不能由前端开发人员修改</li></ul> | <ul><li>较慢</li><li>没有调试器（尚未）</li><li>单位测试难度</li></ul> |


对于页面组件，建议使用混合模型（所有模型逻辑都位于Java中），提供对视图中发生的任何事件（即组件内部）不可知的清晰API。 AEM附带了很好的默认模型，如应能覆盖大多数情况的页面或资源API。

特定于某个组件的所有视图逻辑都应作为JavaScript放置在该组件中，因为它属于该组件。
