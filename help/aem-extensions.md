---
title: AEM扩展
description: AEM提供HTL规范的扩展，以便您作为开发人员使用。
source-git-commit: 6d97bc5d0ab89dffaf56a54c73c94d069bb31ca6
workflow-type: tm+mt
source-wordcount: '308'
ht-degree: 0%

---


# AEM扩展 {#aem-extensions}

与 [HTL规范的Apache Sling扩展，](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#extensions-of-the-htl-specification-1) AEM提供了一些其他表达式选项，使得直接在HTL脚本中使用AEM概念更加容易。

## i18n {#i18n}

相同 [其他三个选项](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#i18n) 与中一样，Apache Sling可以与 `i18n`:

* `locale`
* `hint`
* `basename`

但是，在AEM中， [国际化支持](https://experienceleague.adobe.com/docs/experience-manager-65/developing/components/internationalization/i18n-dev.html) for HTL是在API的帮助下实施的，该API位于 `com.day.cq.i18n` 包。

## data-sly-include {#data-sly-include}

在AEM中， `data-sly-include` 可以额外 `wcmmode` 控制 [WCM模式](https://developer.adobe.com/experience-manager/reference-materials/cloud-service/javadoc/com/day/cq/wcm/api/WCMMode.html) 的值。 允许的值是可用枚举常量的名称。

## data-sly-resource {#data-sly-resource}

除了路径和 `Resources`, `data-sly-resource` 块元素也可以与 [`Maps`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) 或 [`Records`.](https://github.com/apache/sling-org-apache-sling-scripting-sightly-runtime/blob/master/src/main/java/org/apache/sling/scripting/sightly/Record.java) 通过这两种方法， `resourceName` 必须提供字符串属性。 其值用于创建 [合成资源](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.api/latest/org/apache/sling/api/resource/SyntheticResource.html) 将包含在渲染上下文中。 其余的资产 `Record` 或 `Map` 被传递到 `data-sly-resource` 将正常使用 `Resource` 属性。 如果 `sling:resourceType` 此映射中缺少属性，则资源类型将假定为 `resourceType` [表达式选项](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#229-resource) 或驱动渲染的当前资源的资源类型。

将脚本范围中可用的以下映射/记录属性作为 `map`:

```javascript
{
    resourceName: "myText",
    "sling:resourceType": "core/wcm/components/text/v2/text",
    "text": "Hello World!"
}
```

给出了以下标记：

```html
<div class="outer" data-sly-resource="${map}"></div>
```

预计会出现以下输出：

```html
<div class="outer">
    <div class="myText">
        <div data-cmp-data-layer="{&quot;text-e58d65c472&quot;:{&quot;@type&quot;:&quot;core/wcm/components/text/v2/text&quot;,&quot;xdm:text&quot;:&quot;<p>Hello world!</p>&quot;}}" id="text-e58d65c472" class="cmp-text">
            <p>Hello world!</p>
        </div>
  </div>
</div>
```
