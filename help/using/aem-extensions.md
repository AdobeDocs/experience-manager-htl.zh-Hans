---
title: AEM 扩展
description: 为开发人员方便起见，AEM 提供了面向 AEM 的 HTL 规范扩展。
exl-id: d78cb84d-f958-45e2-9c6c-df86a68277d5
index: false
source-git-commit: a496d23277902a5cd573a6a8af770f27b0269f05
workflow-type: ht
source-wordcount: '228'
ht-degree: 100%

---


# AEM 扩展 {#aem-extensions}

与 HTL 规范的 [Apache Sling 扩展类似](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#extensions-of-the-htl-specification-1)，AEM 提供了其他表达式选项，使直接在 HTL 脚本中使用 AEM 概念更加容易。

## i18n {#i18n}

与 Apache Sling 中相同的[三个附加选项](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#i18n)，可以与 `i18n` 一起使用：

* `locale`
* `hint`
* `basename`

然而，在 AEM 中，HTL 的[国际化支持](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/implementing/developing/components/internationalization/i18n-dev)是在 `com.day.cq.i18n` 包中的 API 帮助下实施的。

## `data-sly-include` {#data-sly-include}

在 AEM 中，`data-sly-include` 可以采用附加的 `wcmmode` 选项，用于控制所包含脚本的 [WCM 模式](https://developer.adobe.com/experience-manager/reference-materials/cloud-service/javadoc/com/day/cq/wcm/api/WCMMode.html)。 允许的值是可用枚举常量的名称。

## `data-sly-resource` {#data-sly-resource}

除了路径和 `Resources` 之外，`data-sly-resource` 块元素还可以与 [`Maps`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) 或 [`Records`](https://github.com/apache/sling-org-apache-sling-scripting-sightly-runtime/blob/master/src/main/java/org/apache/sling/scripting/sightly/Record.java) 一起使用。对于这两种方法，必须提供 `resourceName` String 属性。其值用于创建包含在渲染上下文中的[合成资源](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.api/latest/org/apache/sling/api/resource/SyntheticResource.html)。 传递给 `data-sly-resource` 的 `Record` 或 `Map` 中的其余属性，将作为正常 `Resource` 属性使用。 如果此映射中缺少 `sling:resourceType` 属性，则资源类型将被假定为 `resourceType` [表达式选项](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#229-resource)的值，或者是驱动渲染的当前资源的资源类型。

给定脚本范围中可用的以下映射/记录属性为 `map`：

```javascript
{
    resourceName: "myText",
    "sling:resourceType": "core/wcm/components/text/v2/text",
    "text": "Hello World!"
}
```

并给出以下标记：

```html
<div class="outer" data-sly-resource="${map}"></div>
```

预计将产生以下输出：

```html
<div class="outer">
    <div class="myText">
        <div data-cmp-data-layer="{&quot;text-e58d65c472&quot;:{&quot;@type&quot;:&quot;core/wcm/components/text/v2/text&quot;,&quot;xdm:text&quot;:&quot;<p>Hello world!</p>&quot;}}" id="text-e58d65c472" class="cmp-text">
            <p>Hello world!</p>
        </div>
  </div>
</div>
```
