---
title: HTL javaScript Use-API
seo-title: HTL javaScript Use-API
description: HTML模板语言- HTL - javaScript Use-API使HTL文件能够访问使用JavaScript编写的辅助代码。
seo-description: HTML模板语言- HTL - javaScript Use-API使HTL文件能够访问使用JavaScript编写的辅助代码。
uuid: 7ab34b10-30ac-44d6-926b-0234f52e5541
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 18871af8-e44b-4eec-a483-fcc765dae58f
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: bd1962e25d152be4f1608d0a83d8d5b3e728b4aa

---


# HTL javaScript Use-API {#htl-javascript-use-api}

HTML模板语言(HTL)JavaScript Use-API使HTL文件能够访问使用JavaScript编写的辅助代码。 这允许将所有复杂的业务逻辑封装在JavaScript代码中，而HTL代码只处理直接标记制作。

## 简单示例 {#a-simple-example}

我们定义一个组 `info`件，位于

**`/apps/my-example/components/info`**

它包含两个文件：

* **`info.js`**:定义use类的JavaScript文件。
* `info.html`:定义组件的HTL文件 `info`。 This code will use the functionality of `info.js` through the use-API.

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

我们还会在以下位置创建一个内容节 **`info`** 点：

**`/content/my-example`**, with properties:

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

相应的逻辑可以使用以下服务器端 ***JavaScript编写*** ，它位于模板旁 `component.js` 的文件中：

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

这会尝试从不同 `title` 的来源获取描述，并将描述裁切为50个字符。

## 依赖关系 {#dependencies}

让我们想象一下，我们有一个实用程序类，它已经装备了智能功能，如导航标题的默认逻辑或将字符串精心剪切到某个长度：

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

依赖关系模式还可用于扩展另一个组件的逻辑(通常是当 `sling:resourceSuperType` 前组件的逻辑)。

请想象一下，父组件已 `title`经提供，并且我们也要添 **`description`** 加：

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

## 将参数传递到模板 {#passing-parameters-to-a-template}

如果语句可 **`data-sly-template`** 以与组件无关，则将参数传递到关联的Use-API可能很有用。

因此，在我们的组件中，让我们调用一个位于其他文件中的模板：

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

相应的逻辑可以使用以下服务器端 ***JavaScript编写*** ，它位于模板文件 `template.js` 旁边的文件中：

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

传递的参数在关键字上设 `this` 置。
