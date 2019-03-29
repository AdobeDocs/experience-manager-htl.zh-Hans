---
title: HTL JavaScript使用API
seo-title: HTL JavaScript使用API
description: HTML模板语言- HTL- JAR JavaScript使用API允许HTL文件使用JavaScript编写的辅助代码。
seo-description: HTML模板语言- HTL- JAR JavaScript使用API允许HTL文件使用JavaScript编写的辅助代码。
uuid: 7ab34b10-30ac-44d6-926b-0234f52e5541
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 18871af8-e44 b-4eec-a483-cfci765 af58 f
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 796c55d3d85e6b5a3efaa5c04a25be1b0b4e54dd

---


# HTL JavaScript使用API {#htl-javascript-use-api}

HTML模板语言(HTL) JavaScript使用API允许HTL文件使用JavaScript编写的辅助代码。这使所有复杂的业务逻辑都封装在JavaScript代码中，而HTL代码只能进行直接标记制作。

## 简单示例 {#a-simple-example}

我们定义一个组件， `info`位于

**`/apps/my-example/components/info`**

它包含两个文件：

* **`info.js`**：用于定义use-class的JavaScript文件。
* `info.html`：定义组件 `info`的HTL文件。此代码将使用 `info.js` 使用API的功能。

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

我们还会在以下位置创建使用该 **`info`** 组件的内容节点：

**`/content/my-example`**具有属性：

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

相应的逻辑可以使用以下 ***服务器端*** JavaScript编写，位于紧靠模板旁边 `component.js` 的文件中：

```
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

这会尝试来自 `title` 不同来源，并将描述裁切为50个字符。

## 依赖关系 {#dependencies}

假设我们有一个已经配备智能功能的实用程序类，如导航标题的默认逻辑或将字符串精确剪辑到某个长度：

```
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

依赖关系模式还可用于扩展其他组件的逻辑(通常是当前组件的一 `sling:resourceSuperType` 部分)。

想象一下，父组件已经提供了该 `title`组件，我们也希望添加 **`description`** 一个：

```
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

**`data-sly-template`** 对于可以与组件无关的语句，将参数传递给关联的Use-API很有用。

因此，在我们的组件中，我们可以调用位于不同文件中的模板：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

然后，该模板 `template.html`位于以下位置：

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

相应的逻辑可以使用以下 ***服务器端*** JavaScript编写，位于紧靠模板文件旁边的 `template.js` 文件中：

```
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

传递的参数在 `this` 关键字上设置。
