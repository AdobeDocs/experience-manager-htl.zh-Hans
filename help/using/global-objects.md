---
title: HTL全局对象
seo-title: HTL全局对象
description: 无需指定任何内容，HTL即可提供对JSP中在包含global.jsp后通常可用的所有对象的访问。
seo-description: '无需指定任何内容，HTL即可提供对JSP中在包含global.jsp后通常可用的所有对象的访问。 '
uuid: e03affbb-a683-4323-8224-53d8ef59caef
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: fe071a7e-0dae-45c1-9f86-80c558483f87
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: c3beb0d02f18483b1b000c1bf70cd59a3dcc2035

---


# HTL全局对象{#htl-global-objects}

无需指定任何内容，HTL即可提供对JSP中在包含后通常可用的所有对象的访问 `global.jsp`。 这些对象是通过 [Use-API引入的对象之外的对象](use-api.md)。

## 可枚举的对象 {#enumerable-objects}

这些对象提供了对常用信息的便捷访问。 可以使用点记号访问其内容，也可以使用或重复 `data-sly-list` 访问 `data-sly-repeat`。

| 变量名称 | 描述 |
|--- |--- |
| 属性 | 当前资源的属性列表。 由 [org.apache.sling.api.resource.ValueMap支持](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| pageProperties | 当前页面的页面属性列表。 由 [org.apache.sling.api.resource.ValueMap支持](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.hmtl) |
| inheritedPageProperties | 当前页面的继承页面属性列表。 由 [org.apache.sling.api.resource.ValueMap支持](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) |


## Java支持的对象 {#java-backed-objects}

以下每个对象都由相应的Java对象备份。

下表中最有用的变量以粗体突出显示。

| 变量名称 | 描述 |  |
|---|---|---|
| `component` | `com.day.cq.wcm.api.components.Component` |  |
| `componentContext` | `com.day.cq.wcm.api.components.ComponentContext` |  |
| `currentDesign` | `com.day.cq.wcm.api.designer.Design` |  |
| `currentNode` | `javax.jcr.Node` |  |
| `currentPage` | `com.day.cq.wcm.api.Page` |  |
| `currentSession` | `javax.servlet.http.HttpSession` |  |
| `currentStyle` | `com.day.cq.wcm.api.designer.Style` |  |
| `designer` | `com.day.cq.wcm.api.designer.Designer` |  |
| `editContext` | `com.day.cq.wcm.api.components.EditContext` |  |
| `log` | `org.slf4j.Logger` |  |
| `out` | `java.io.PrintWriter` |  |
| `pageManager` | `com.day.cq.wcm.api.PageManager` |  |
| `reader` | `java.io.BufferedReader` |  |
| `request` | `org.apache.sling.api.SlingHttpServletRequest` |  |
| `resolver` | `org.apache.sling.api.resource.ResourceResolver` |  |
| `resource` | `org.apache.sling.api.resource.Resource` |  |
| `resourceDesign` | `com.day.cq.wcm.api.designer.Design` |  |
| `resourcePage` | `com.day.cq.wcm.api.Page` |  |
| `response` | `org.apache.sling.api.SlingHttpServletResponse` |  |
| `sling` | `org.apache.sling.api.scripting.SlingScriptHelper` |  |
| `slyWcmHelper` | `com.adobe.cq.sightly.WCMScriptHelper` |  |
| `wcmmode` | `com.adobe.cq.sightly.SightlyWCMMode` |  |
| `xssAPI` | `com.adobe.granite.xss.XSSAPI` |  |

## JavaScript支持的对象 {#javascript-backed-objects}

还有由JavaScript支持的可用对象。 但是，从AEM 6.2开始，这些对象仍然是试验性的，最好使用支持Java的对象，这样也可以。

<!-- 

Comment Type: draft

<p> </p> 
<p>JS-specific context variables: These supply access to asynchronous implementions of all the Java objects listed below). To write HTL code that is protable to granite.js, you must use the variables provided by aem and sly, not the native Java variables.</p> 
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
<p>JS specific non-HTL related variables. Present due to JS-implementaion. Generally not used in templating:</p> 
<ul> 
 <li>console: JS Object</li> 
 <li>exports: JS Object</li> 
 <li>module: JS Object</li> 
 <li>setImmediate: JS Function</li> 
 <li>setTimeout: JS Function</li> 
 <li>use: JS Function</li> 
</ul>
-->
