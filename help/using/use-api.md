---
title: HTL使用API
seo-title: Adobe HTL使用API
description: 有两个 API 可用于 HTL - 使用 Java 的 API 和使用 Javascript 的 API
seo-description: 有两个 API 可用于 Adobe HTL - 使用 Java 的 API 和使用 Javascript 的 API
uuid: ab44aa5c-ce7 e-40b9-97fb-e86 c6 a28405 c
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 89004426-eb59-4b63-913f-51bf98662773
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 271c355ae56e16e309853b02b8ef09f2ff971a2e

---


# HTL使用API {#htl-use-api}

下表概述了每个API的优缺点。

|  | **Java Use-API** | **JavaScript使用API** |
|--- |--- |--- |
| **专业人员** | <ul><li>速度更快</li><li>可以使用调试器进行检查</li><li>易于单元测试</li></ul> | <ul><li>可由前端开发人员修改</li><li>位于组件内，使组件的视图逻辑靠近其相应的模板</li></ul> |
| **Cons** | <ul><li>无法被前端开发人员修改</li></ul> | <ul><li>slow</li><li>无调试器(尚未)</li><li>更难单元测试</li></ul> |


对于页面组件，建议使用混合模型(在Java中所有模型逻辑都位于Java中)，提供对视图中发生的任何事件(即组件内的任何事件)都具有敏感性的API。AEM附带了一些很好的默认模型，如页面或资源API，这些模型应能够涵盖大多数情况。

应将特定于某个组件的所有视图逻辑作为JavaScript放置在该组件中，因为它属于该组件。
