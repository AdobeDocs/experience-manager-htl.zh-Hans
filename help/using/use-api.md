---
title: HTL Use-API
description: 有两个 API 可用于 HTL - 使用 Java 的 API 和使用 Javascript 的 API
exl-id: 8f95707e-952c-4efe-a44e-9a1ad501605e
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 8%

---

# HTL Use-API {#htl-use-api}

HTL不允许将业务逻辑与标记混合，从而鼓励了问题的分离。 业务逻辑可通过Use-API实施。

下表概述了每个API的优缺点。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **优势** | <ul><li>更快</li><li>可通过调试器进行检查</li><li>易于单元测试</li></ul> | <ul><li>前端开发人员可以修改</li><li>位于组件内，将组件的视图逻辑保留在靠近其相应模板的位置</li></ul> |
| **缺点** | <ul><li>前端开发人员无法修改</li></ul> | <ul><li>慢</li><li>尚未调试器</li><li>单元测试难度</li></ul> |

对于页面组件，建议使用混合模型，其中所有模型逻辑都位于Java中，从而提供与视图中发生的任何事件（即组件内的事件）不相关的清晰API。 AEM提供了大多数情况下都应该能够涵盖的优秀默认模型，例如页面或资源API。

特定于某个组件的所有视图逻辑都应作为JavaScript放置在该组件中，因为它属于该组件。
