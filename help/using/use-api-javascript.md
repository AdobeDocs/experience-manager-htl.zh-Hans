---
title: HTL JavaScript Use-API
description: 利用 HTML 模板语言 (HTL) JavaScript Use-API，HTL 文件可以访问使用 JavaScript 编写的帮助程序代码。
exl-id: e98bfbd5-fa64-48c7-bd14-477d4c5e1788
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '324'
ht-degree: 100%

---

# HTL JavaScript Use-API {#htl-javascript-use-api}

利用 HTML 模板语言 (HTL) JavaScript Use-API，HTL 文件可以访问使用 JavaScript 编写的帮助程序代码。这样可以将所有复杂的业务逻辑封装在 JavaScript 代码中，而 HTL 代码只处理直接标记生成。

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

我们定义一个组件 `info`，位于

`/apps/my-example/components/info`

它包含两个文件：

* **`info.js`**：定义 use 类的 JavaScript 文件。
* **`info.html`**：定义组件 `info` 的 HTL 文件。此代码将通过 use-API 使用 `info.js` 的功能。

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

我们还创建一个 content 节点，它在以下位置使用 `info` 组件：

`/content/my-example`，属性为：

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

考虑以下组件模板：

```xml
<section class="component-name" data-sly-use.component="component.js">
    <h1>${component.title}</h1>
    <p>${component.description}</p>
</section>
```

相应的逻辑可以使用位于模板旁边的 `component.js` 文件中的以下服务器端 JavaScript 进行编写：

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

这会尝试从不同源获取 `title` 并将 description 裁剪为 50 个字符。

## 依赖关系 {#dependencies}

假设我们有一个实用程序类，它已经配备了智能功能，例如导航标题的默认逻辑或很好地将字符串剪切为特定长度：

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

依赖模式还可用于扩展另一个组件（通常是当前组件的 `sling:resourceSuperType`）的逻辑。

假设父组件已经提供了 `title`，我们还想添加一个 `description`：

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

## 将参数传递给模板 {#passing-parameters-to-a-template}

对于可以独立于组件的 `data-sly-template` 语句来说，将参数传递给关联的 Use-API 可能很有用。

因此，在我们的组件中，让我们调用位于不同文件中的模板：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

下面是位于 `template.html` 中的模板：

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

相应的逻辑可以使用位于模板文件旁边的 `template.js` 文件中的以下服务器端 JavaScript 进行编写：

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

在 `this` 关键词上设置传递的参数。
