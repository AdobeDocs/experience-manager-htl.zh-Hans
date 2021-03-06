---
title: HTL 全局对象
description: 无需指定任何内容，HTL 在包括 global.jsp 之后即可提供对 JSP 中所有常用对象的访问权限。
exl-id: ca590b92-f1b3-4e44-a04a-a2c10dff256f
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '220'
ht-degree: 100%

---

# HTL 全局对象 {#htl-global-objects}

无需指定任何内容，HTL 在包括 `global.jsp` 之后即可提供对 JSP 中所有常用对象的访问权限。这些对象是对可能通过 [Use-API](use-api.md) 引入的任何对象的补充。

## 可枚举对象 {#enumerable-objects}

这些对象提供对常用信息的便捷访问。可以使用点表示法访问它们的内容，并且可以使用 `data-sly-list` 或 `data-sly-repeat` 循环访问它们。

| 变量名称 | 描述 |
|--- |--- |
| `properties` | 当前资源的属性列表。受 [org.apache.sling.api.resource.ValueMap](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) 支持 |
| `pageProperties` | 当前页面的页面属性列表。受 [org.apache.sling.api.resource.ValueMap](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) 支持 |
| `inheritedPageProperties` | 当前页面的继承的页面属性列表。受 [org.apache.sling.api.resource.ValueMap](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) 支持 |

## Java 支持的对象 {#java-backed-objects}

以下每个对象都受相应的 Java 对象支持。

下表中最有用的变量以粗体突出显示。

| 变量名称 | 描述 |
|---|---|
| **`component`** | `com.day.cq.wcm.api.components.Component` |
| `componentContext` | `com.day.cq.wcm.api.components.ComponentContext` |
| **`currentDesign`** | `com.day.cq.wcm.api.designer.Design` |
| `currentNode` | `javax.jcr.Node` |
| **`currentPage`** | `com.day.cq.wcm.api.Page` |
| `currentSession` | `javax.servlet.http.HttpSession` |
| `currentStyle` | `com.day.cq.wcm.api.designer.Style` |
| `designer` | `com.day.cq.wcm.api.designer.Designer` |
| `editContext` | `com.day.cq.wcm.api.components.EditContext` |
| `log` | `org.slf4j.Logger` |
| `out` | `java.io.PrintWriter` |
| `pageManager` | `com.day.cq.wcm.api.PageManager` |
| `reader` | `java.io.BufferedReader` |
| **`request`** | `org.apache.sling.api.SlingHttpServletRequest` |
| `resolver` | `org.apache.sling.api.resource.ResourceResolver` |
| **`resource`** | `org.apache.sling.api.resource.Resource` |
| **`resourceDesign`** | `com.day.cq.wcm.api.designer.Design` |
| **`resourcePage`** | `com.day.cq.wcm.api.Page` |
| `response` | `org.apache.sling.api.SlingHttpServletResponse` |
| `sling` | `org.apache.sling.api.scripting.SlingScriptHelper` |
| `slyWcmHelper` | `com.adobe.cq.sightly.WCMScriptHelper` |
| **`wcmmode`** | `com.adobe.cq.sightly.SightlyWCMMode` |
| `xssAPI` | `com.adobe.granite.xss.XSSAPI` |

## JavaScript 支持的对象 {#javascript-backed-objects}

可以使用 JavaScript 支持 HTL 逻辑。但是，首选或推荐的方法是使用 [Sling 模型](https://sling.apache.org/documentation/bundles/models.html)。

<!-- 

Comment Type: draft

<p> </p> 
<p>JS-specific context variables: These supply access to asynchronous implementations of all the Java objects listed below). To write HTL code that is portable to granite.js, you must use the variables provided by aem and sly, not the native Java variables.</p> 
<ul> 
 <li>wcm
  <ul> 
   <li>currentPage</li> 
   <li>nativePage: [com.day.cq.wcm.apiPage]</li> 
   <li>properties: {<i>enumerable</i>}</li> 
  </ul> </li> 
 <li>granite
  <ul> 
   <li>request
    <ul> 
     <li>parameters: {<i>enumerable</i>}</li> 
     <li>nativeRequest: [org.apache.sling.scripting.core.impl.helper.OnDemandReaderRequest]</li> 
     <li>pathInfo
      <ul> 
       <li>nativePathInfo: [SlingRequestPathInfo: path='/content/geometrixx/en/jcr:content/par/text', selectorString='null', extension='html', suffix='null']</li> 
      </ul> </li> 
    </ul> </li> 
   <li>resource
    <ul> 
     <li>nativeResource: [Paragraph, path=/content/geometrixx/en/jcr:content/par/text, type=wcm/foundation/components/text, cssClass=default, column=0/0, diffInfo=[null], resource=[JcrNodeResource, type=wcm/foundation/components/text, superType=null, path=/content/geometrixx/en/jcr:content/par/text]]</li> 
     <li>path: "/content/geometrixx/en/jcr:content/par/text"</li> 
     <li>properties: {sling:resourceType,jcr:created,jcr:lastModified,jcr:createdBy, textIsRich,jcr:lastModifiedBy,jcr:primaryType}</li> 
    </ul> </li> 
   <li>properties: {sling:resourceType,jcr:created,jcr:lastModified,jcr:createdBy, textIsRich,jcr:lastModifiedBy,jcr:primaryType}</li> 
  </ul> </li> 
</ul> 
<p>JS specific non-HTL related variables. Present due to JS-implementation. Generally not used in templating:</p> 
<ul> 
 <li>console: JS Object</li> 
 <li>exports: JS Object</li> 
 <li>module: JS Object</li> 
 <li>setImmediate: JS Function</li> 
 <li>setTimeout: JS Function</li> 
 <li>use: JS Function</li> 
</ul>
-->
