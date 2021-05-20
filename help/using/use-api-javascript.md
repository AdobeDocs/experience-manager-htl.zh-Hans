---
title: HTL JavaScript Use-API
description: HTML模板语言 — HTL - JavaScript Use-API允许HTL文件访问使用JavaScript编写的帮助程序代码。
exl-id: e98bfbd5-fa64-48c7-bd14-477d4c5e1788
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '324'
ht-degree: 2%

---

# HTL JavaScript Use-API {#htl-javascript-use-api}

HTML模板语言(HTL)JavaScript Use-API允许HTL文件访问使用JavaScript编写的帮助程序代码。 这允许将所有复杂的业务逻辑封装在JavaScript代码中，而HTL代码仅处理直接标记生产。

使用了以下约定。

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

## {#a-simple-example}的简单示例

我们定义一个组件`info`，位于

`/apps/my-example/components/info`

它包含两个文件：

* **`info.js`**:定义use类的JavaScript文件。
* **`info.html`**:用于定义组件的HTL文 `info`件。此代码将通过use-API使用`info.js`的功能。

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

我们还会在`info`

`/content/my-example`，其中包含属性：

* `sling:resourceType = "my-example/component/info"`
* `title = "My Example"`
* `description = "This is some example content."`

以下是生成的存储库结构：

### 存储库结构{#repository-structure}

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

可以使用以下服务器端JavaScript编写相应的逻辑，该JavaScript位于模板旁边的`component.js`文件中：

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

这会尝试从不同来源获取`title`，并将描述裁剪为50个字符。

## 依赖关系 {#dependencies}

假设我们有一个实用工具类，该类已经配备了智能功能，例如导航标题的默认逻辑或将字符串精心剪切到一定长度：

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

依赖关系模式还可用于扩展其他组件（通常是当前组件的`sling:resourceSuperType`）的逻辑。

假设父组件已经提供`title`，并且我们还要添加`description`:

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

## 将参数传递到模板{#passing-parameters-to-a-template}

对于与组件无关的`data-sly-template`语句，将参数传递到关联的Use-API会非常有用。

因此，在我们的组件中，让我们调用位于其他文件中的模板：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

下面是位于`template.html`中的模板：

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

可以使用以下服务器端JavaScript编写相应的逻辑，该JavaScript位于模板文件旁边的`template.js`文件中：

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

传递的参数在`this`关键字上设置。
