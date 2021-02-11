---
title: HTL Use-API
description: 有两个 API 可用于 HTL - 使用 Java 的 API 和使用 Javascript 的 API
translation-type: tm+mt
source-git-commit: f7e46aaac2a4b51d7fa131ef46692ba6be58d878
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 8%

---


# HTL Use-API {#htl-use-api}

HTL不允许业务逻辑与标记混合，从而鼓励分散关注。 业务逻辑可通过Use-API实现。

下表概述了每个API的优缺点。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **优势** | <ul><li>更快</li><li>可通过调试器进行检查</li><li>易于单元测试</li></ul> | <ul><li>可由前端开发人员修改</li><li>位于组件中，使组件的视图逻辑与其相应的模板保持接近</li></ul> |
| **缺点** | <ul><li>前端开发人员无法修改</li></ul> | <ul><li>较慢</li><li>尚无调试器</li><li>单元测试更困难</li></ul> |

对于页面组件，建议使用混合模型，其中所有模型逻辑都位于Java中，提供与视图中发生的任何情况（即组件中）无关的清晰API。 AEM附带了大多数情况下都应能覆盖的Page或Resource API等出色的默认模型。

特定于某个组件的所有视图逻辑都应作为JavaScript放在该组件中，因为它属于该组件。
