---
title: HTL 全局对象
description: 了解 HTL 中可枚举对象、Java 支持的对象和 JavaScript 支持的对象。
exl-id: ca590b92-f1b3-4e44-a04a-a2c10dff256f
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '164'
ht-degree: 87%

---


# HTL 全局对象 {#htl-global-objects}

无需指定任何内容，HTL 允许开发人员访问许多对其十分有用的对象。 这些对象是对可能通过 [Use-API](java-use-api.md) 引入的任何对象的补充。

>[!NOTE]
>
>对于熟悉 AEM 中 JSP 开发的开发人员，HTL 允许访问 JSP 中包含 `global.jsp` 后通常可用的所有对象。

## 可枚举对象 {#enumerable-objects}

这些对象提供对常用信息的便捷访问。可以使用点表示法访问它们的内容，并且可以使用 `data-sly-list` 或 `data-sly-repeat` 循环访问它们。

| 变量名称 | 描述 | 支持方 |
|--- |--- |--- |
| `properties` | 当前资源的属性列表 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `pageProperties` | 当前页面的页面属性列表 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `inheritedPageProperties` | 当前页面的继承的页面属性列表 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |

## Java 支持的对象 {#java-backed-objects}

相应的Java对象支持以下每个对象。

| 变量名称 | 描述 |
|---|---|
| `component` | `com.day.cq.wcm.api.components.Component` |
| `componentContext` | `com.day.cq.wcm.api.components.ComponentContext` |
| `currentContentPolicy` | `com.day.cq.wcm.api.policies.ContentPolicy` |
| `currentContentPolicyProperties` | `com.day.cq.wcm.api.policies.ContentPolicy` |
| `currentDesign` | `com.day.cq.wcm.api.designer.Design` |
| `currentNode` | `javax.jcr.Node` |
| `currentPage` | `com.day.cq.wcm.api.Page` |
| `currentSession` | `javax.servlet.http.HttpSession` |
| `currentStyle` | `com.day.cq.wcm.api.designer.Style` |
| `designer` | `com.day.cq.wcm.api.designer.Designer` |
| `editContext` | `com.day.cq.wcm.api.components.EditContext` |
| `log` | `org.slf4j.Logger` |
| `out` | `java.io.PrintWriter` |
| `pageManager` | `com.day.cq.wcm.api.PageManager` |
| `reader` | `java.io.BufferedReader` |
| `request` | `org.apache.sling.api.SlingHttpServletRequest` |
| `resolver` | `org.apache.sling.api.resource.ResourceResolver` |
| `resource` | `org.apache.sling.api.resource.Resource` |
| `resourceDesign` | `com.day.cq.wcm.api.designer.Design` |
| `resourcePage` | `com.day.cq.wcm.api.Page` |
| `response` | `org.apache.sling.api.SlingHttpServletResponse` |
| `sling` | `org.apache.sling.api.scripting.SlingScriptHelper` |
| `slyWcmHelper` | `com.adobe.cq.sightly.WCMScriptHelper` |
| `wcmmode` | `com.adobe.cq.sightly.SightlyWCMMode` |
| `xssAPI` | `com.adobe.granite.xss.XSSAPI` |

## JavaScript 支持的对象 {#javascript-backed-objects}

可以使用 JavaScript 支持 HTL 逻辑。但是，首选或推荐的方法是使用[Sling模型](https://sling.apache.org/documentation/bundles/models.html)。
