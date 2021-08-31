---
title: HTL Use-API
description: HTL 有两个可用 API - Java Use-API 和 Javascript Use-API
exl-id: 8f95707e-952c-4efe-a44e-9a1ad501605e
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '183'
ht-degree: 100%

---

# HTL Use-API {#htl-use-api}

HTL 不允许业务逻辑与标记混合，鼓励将关系分开。可以通过 Use-API 实现业务逻辑。

下表概述了每个 API 的优缺点。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **优点** | <ul><li>较快</li><li>可以用调试器检查</li><li>很容易进行单元测试</li></ul> | <ul><li>前端开发人员可以修改</li><li>位于组件内，使组件的视图逻辑接近其相应的模板</li></ul> |
| **缺点** | <ul><li>前端开发人员无法修改</li></ul> | <ul><li>较慢</li><li>无调试器（截止目前）</li><li>较难进行单元测试</li></ul> |

对于页面组件，建议使用混合模型，其中所有模型逻辑都位于 Java 中，从而提供独立于视图内（即组件内）发生的任何情况的透明 API。AEM 附带像页面或资源 API 这样非常好的默认模型，应该能够覆盖大多数情形。

特定于某个组件的所有视图逻辑应作为 JavaScript 放置在该组件中，因为它属于该组件。
