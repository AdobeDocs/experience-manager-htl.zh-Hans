---
title: HTL JavaScript Use-API
description: HTML模板语言- HTL - JavaScript Use-API使HTL文件能够访问用JavaScript编写的帮助代码。
translation-type: tm+mt
source-git-commit: ee712ef61018b5e05ea052484e2a9a6b12e6c5c8
workflow-type: tm+mt
source-wordcount: '324'
ht-degree: 2%

---


# HTL JavaScript Use-API {#htl-javascript-use-api}

HTML模板语言(HTL)JavaScript Use-API使HTL文件能够访问用JavaScript编写的帮助代码。 这允许将所有复杂的业务逻辑封装在JavaScript代码中，而HTL代码只处理直接标记生产。

使用以下约定。

```javascript
/**
 * In the following example '/libs/dep1.js' and 'dep2.js' are optional
 * dependencies needed for this script's execution. Dependencies can
 * be specified using an absolute path or a relative path to this
 * script's own path.
 *
 * If no dependencies are needed the dependencies array can be omitted.
 */
use(['dep1.js', 'dep2.js'], function (Dep1, Dep2) {
    // implement processing
  
    // define this Use object's behavior
    return {
        propertyName: propertyValue
        functionName: function () {}
    }
});
```

## 简单示例 {#a-simple-example}

我们定义一个 `info`组件，位于

`/apps/my-example/components/info`

它包含两个文件：

* **`info.js`**: 定义use类的JavaScript文件。
* **`info.html`**: 定义组件的HTL文 `info`件。 此代码将通过use- `info.js` API使用功能。

### /apps/my-example/component/info/info.js {#apps-my-example-component-info-info-js}

```java
"use strict";
use(function () {
    var info = {};
    info.title = resource.properties["title"];
    info.description = resource.properties["description"];
    return info;
});
```

### /apps/my-example/component/info/info.html {#apps-my-example-component-info-info-html}

```xml
<div data-sly-use.info="info.js">
    <h1>${info.title}</h1>
    <p>${info.description}</p>
</div>
```

我们还会在以下位置创建一个使用该组 `info` 件的内容节点：

`/content/my-example`, with属性：

* `sling:resourceType = "my-example/component/info"`
* `title = "My Example"`
* `description = "This is some example content."`

下面是生成的存储库结构：

### 存储库结构 {#repository-structure}

```java
{
  "apps": {
    "my-example": {
      "components": {
        "info": {
          "info.html": {
            ...
          },
          "info.js": {
            ...
          }
        }
      }
    }
 },
 "content": {
    "my-example": {
      "sling:resourceType": "my-example/component/info",
      "title": "My Example",
      "description": "This is some example content."
    }
  }
}
```

请考虑以下组件模板：

```xml
<section class="component-name" data-sly-use.component="component.js">
    <h1>${component.title}</h1>
    <p>${component.description}</p>
</section>
```

相应的逻辑可以使用以下服务器端JavaScript编写，它位于模板 `component.js` 旁边的文件中：

```javascript
use(function () {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };

    var title = currentPage.getNavigationTitle() || currentPage.getTitle() || currentPage.getName();
    var description = properties.get(Constants.DESCRIPTION_PROP, "").substr(0, Constants.DESCRIPTION_LENGTH);

    return {
        title: title,
        description: description
    };
});
```

这会尝试从不同 `title` 来源获取描述，并将描述裁切为50个字符。

## 依赖关系 {#dependencies}

让我们想象一下，我们有一个实用程序类，它已经装备了智能功能，如导航标题的默认逻辑或将字符串精确地切割到某个长度：

```javascript
use(['../utils/MyUtils.js'], function (utils) {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };

    var title = utils.getNavigationTitle(currentPage);
    var description = properties.get(Constants.DESCRIPTION_PROP, "").substr(0, Constants.DESCRIPTION_LENGTH);

    return {
        title: title,
        description: description
    };
});
```

## 扩展 {#extending}

该依赖性模式还可用于扩展另一个组件(通常是当前组 `sling:resourceSuperType` 件的逻辑)的逻辑。

请想象一下，父组件 `title`已经提供，我们也 `description` 要添加：

```javascript
use(['../parent-component/parent-component.js'], function (component) {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };

    component.description = utils.shortenString(properties.get(Constants.DESCRIPTION_PROP, ""), Constants.DESCRIPTION_LENGTH);

    return component;
});
```

## 将参数传递到模板 {#passing-parameters-to-a-template}

对于可以 `data-sly-template` 独立于组件的语句，将参数传递到关联的Use-API可能很有用。

因此，在我们的组件中，让我们调用位于其他文件中的模板：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

这是位于以下位置的模板 `template.html`:

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

相应的逻辑可以使用以下服务器端JavaScript编写，它位于模板文 `template.js` 件旁的文件中：

```javascript
use(function () {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description"
    };

    var title = this.page.getNavigationTitle() || this.page.getTitle() || this.page.getName();
    var description = this.page.getProperties().get(Constants.DESCRIPTION_PROP, "").substr(0, this.descriptionLength);

    return {
        title: title,
        description: description
    };
});
```

传递的参数在关键字上 `this` 设置。
