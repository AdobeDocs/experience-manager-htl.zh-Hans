---
title: HTL 全局对象
description: 了解HTL中可枚举的对象、Java支持的对象和JavaScript支持的对象。
exl-id: ca590b92-f1b3-4e44-a04a-a2c10dff256f
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '200'
ht-degree: 32%

---


# HTL 全局对象 {#htl-global-objects}

HTL无需指定任何内容，即可访问对开发人员有用的许多对象。 这些对象除了可通过 [Use-API。](java-use-api.md)

>[!NOTE]
>
>对于熟悉AEM中JSP开发的开发人员，HTL提供对JSP中通常提供的所有对象的访问权限，这些对象在包括 `global.jsp`.

## 可枚举对象 {#enumerable-objects}

这些对象提供对常用信息的便捷访问。可以使用点表示法访问其内容，并可使用 `data-sly-list` 或 `data-sly-repeat`.

| 变量名称 | 描述 | 支持者 |
|--- |--- |--- |
| `properties` | 当前资源的属性列表 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `pageProperties` | 当前页面的页面属性列表 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `inheritedPageProperties` | 当前页面的继承页面属性列表 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |

## Java 支持的对象 {#java-backed-objects}

以下每个对象都受相应的 Java 对象支持。

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

可以使用 JavaScript 支持 HTL 逻辑。但是，首选或推荐的方法是使用 [Sling 模型](https://sling.apache.org/documentation/bundles/models.html)。
