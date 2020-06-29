---
title: HTL Java Use-API
description: 'HTML 模板语言 - HTL - Java Use-API 让 HTL 文件可以访问自定义 Java 类中的 Helper 方法。 '
translation-type: tm+mt
source-git-commit: ee712ef61018b5e05ea052484e2a9a6b12e6c5c8
workflow-type: tm+mt
source-wordcount: '2558'
ht-degree: 1%

---


# HTL Java Use-API {#htl-java-use-api}

The HTML Template Language (HTL) Java Use-API enables an HTL file to access helper methods in a custom Java class through `data-sly-use`. 这允许将所有复杂的业务逻辑封装在Java代码中，而HTL代码只处理直接标记生产。

Java Use-API对象可以是简单的POJO，通过POJO的默认构造函数由特定实现进行实例化。

Use-API POJO还可以使用以下签名公开一个名为init的公共方法：

```java
    /**
     * Initializes the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/
    public void init(javax.script.Bindings bindings);
```

映 `bindings` 射可包含为当前执行的HTL脚本提供上下文的对象，Use-API对象可以使用这些对象进行处理。

## 简单示例 {#a-simple-example}

我们将开始没有use类的HTL组件。 它由一个文件组成， `/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我们还为此组件添加一些内容，以在以下位置进行渲染 `/content/my-example/`:

### `http://<host>:<port>/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

访问此内容时，将执行HTL文件。 在HTL代码中，我们使用上下文 `properties` 对象访问当前资源并 `title` 显示 `description` 它们。 输出HTML将为：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加Use类 {#adding-a-use-class}

信 **息组** 件本身不需要use类来执行其（非常简单）函数。 但是，在某些情况下，您需要做在HTL中无法完成的事情，因此需要使用类。 但请记住以下几点：

>[!NOTE]
>
>仅当某些操作无法在HTL中完成时，才应使用use类。

例如，假定您希望组件 `info` 显示资源的 `title` 和 `description` 属性，但全部以小写形式显示。 由于HTL没有用于小写字符串的方法，因此您需要use-class。 我们可以通过添加一个Java use-class并按如下方式更改 `info.html` 来实现：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-1}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    private String lowerCaseTitle;
    private String lowerCaseDescription;

    @Override
    public void activate() throws Exception {
        lowerCaseTitle = getProperties().get("title", "").toLowerCase();
        lowerCaseDescription = getProperties().get("description", "").toLowerCase();
    }

    public String getLowerCaseTitle() {
        return lowerCaseTitle;
    }

    public String getLowerCaseDescription() {
        return lowerCaseDescription;
    }
}
```

在以下各节中，我们将遍历代码的不同部分。

### 本地与捆绑Java类 {#local-vs-bundle-java-class}

Java use-class可以通过两种方式安装： **本地** 或 **捆绑**。 此示例使用本地安装。

在本地安装中，Java源文件将放在HTL文件的旁边，位于同一存储库文件夹中。 该源将按需自动编译。 无需单独的编译或打包步骤。

在捆绑安装中，必须使用标准AEM捆绑部署机制在OSGi捆绑包中编译和部署Java类(请参 [阅捆绑Java类](#bundled-java-class))。

>[!NOTE]
>
>当 **use类特定于相** 关组件时，建议使用本地Java use-class。
>
>当 **Java代码实现从多个HTL组件** 访问的服务时，建议使用捆绑的Java使用类。

### Java包是存储库路径 {#java-package-is-repository-path}

当使用本地安装时，use-class的包名称必须与存储库文件夹位置的名称匹配，路径中的任何连字符都替换为包名称中的下划线。

在本例中， `Info.java` 位于以下位 `/apps/my-example/components/info` 置，因此软件包是 `apps.my_example.components.info`:

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-1}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {

   ...

}
```

>[!NOTE]
>
>在AEM开发中，建议在存储库项目名称中使用连字符。 但是，连字符在Java包名称中是非法的。 因此，必须将 **存储库路径中的所有连字符转换为包名称中的下划线**。

### 扩展 `WCMUsePojo` {#extending-wcmusepojo}

虽然有多种将Java类与HTL集成的方法(请参阅替代方 `WCMUsePojo`法)，但最简单的方法是扩展 `WCMUsePojo` 类：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化类 {#initializing-the-class}

从扩展use类时，通过 `WCMUsePojo`覆盖方法来执行 `activate` 初始化：

### /apps/my-example/component/info/Info.java {#apps-my-example-component-info-info-java-3}

```java
...

public class Info extends WCMUsePojo {
    private String lowerCaseTitle;
    private String lowerCaseDescription;

    @Override
    public void activate() throws Exception {
        lowerCaseTitle = getProperties().get("title", "").toLowerCase();
        lowerCaseDescription = getProperties().get("description", "").toLowerCase();
    }

...

}
```

### 上下文 {#context}

通常， [激活方](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) 法用于根据当前上下文（例如，当前请求和资源）预计算和存储HTL代码中所需的值（在成员变量中）。

类 `WCMUsePojo` 提供对HTL文件中可用的同一组上下文对象的访问(请参阅 [全局对象](global-objects.md))。

在扩展的类中，可 `WCMUsePojo`以使用名称访问上下文对象

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，常用的上下文对象也可以通过相应的便捷方 **法直接访问**:

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [设计人员](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [设计](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [样式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [组件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getInheritedProperties) |
| [资源](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [资源解析器](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter方法 {#getter-methods}

use-class初始化后，将运行HTL文件。 在此阶段，HTL通常会拉入use-class的各个成员变量的状态，并呈现这些变量以进行演示。

要从HTL文件中访问这些值，必须根据以下命名约定在use-class中定义自定义getter方法：

* 表单的方法将 `getXyz` 在HTL文件中显示一个名为的对象属性 `xyz`。

在以下示例中，这些方 `getTitle` 法和 `getDescription` 结果导致对象属性 `title` ，并在 `description` HTL文件的上下文中变得可访问：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-4}

```java
...

public class Info extends WCMUsePojo {

    ...

    public String getLowerCaseTitle() {
        return lowerCaseTitle;
    }

    public String getLowerCaseDescription() {
        return lowerCaseDescription;
    }
}
```

### data-sly-use属性 {#data-sly-use-attribute}

该 `data-sly-use` 属性用于初始化HTL代码中的use类。 在我们的示例 `data-sly-use` 中，属性声明我们要使用类 `Info`。 我们只能使用类的本地名称，因为我们使用的是本地安装（已将Java源文件放在与HTL文件相同的文件夹中）。 如果使用捆绑安装，则必须指定完全限定的类名。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本地标识符 {#local-identifier}

HTL文 `info` 件中使用标 `data-sly-use.info`识符（在中的点之后）来标识类。 声明后，此标识符的范围在文件中是全局的。 它不限于包含语句的元 `data-sly-use` 素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 获取属性 {#getting-properties}

然后， `info` 该标识符用于访问对象属性 `title` , `description` 以及通过getter方法和公开的 `Info.getTitle` 对象 `Info.getDescription`。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 输出 {#output}

现在，当我们访 `/content/my-example.html` 问时，它将返回以下HTML:

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 超越基础 {#beyond-the-basics}

在本节中，我们将介绍一些超越上述简单示例的其他功能：

* 将参数传递到use类。
* 捆绑的Java用类。
* 替代 `WCMUsePojo`

### 传递参数 {#passing-parameters}

在初始化时，参数可以传递给use类。 例如，我们可以执行以下操作：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

在此，我们传递一个名为的参数 `text`。 use-class然后将我们检索的字符串加上并显示结果 `info.upperCaseText`。 以下是调整后的使用类：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-5}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {

    ...

    private String reverseText;

    @Override
    public void activate() throws Exception {

        ...

        String text = get("text", String.class);
        reverseText = new StringBuffer(text).reverse().toString();

    }

    public String getReverseText() {
        return reverseText;
    }

    ...
}
```

通过方法访问该参 `WCMUsePojo` 数 [`<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

就我们而言，声明：

`get("text", String.class)`

然后，该字符串通过以下方法进行反转和公开：

`getReverseText()`

### 仅从数据密码模板传递参数 {#only-pass-parameters-from-data-sly-template}

尽管上述示例在技术上是正确的，但实际上当相关值在HTL代码的执行上下文中可用时（或者，试用，该值是静态的，如上所述），从HTL传递值来初始化使用类并没什么意义。

其原因是use-class将始终具有与HTL代码相同的执行上下文的访问权限。 这就引出了最佳实践的一个要点：

>[!NOTE]
>
>仅当在文件中使用use-class时，才应将参数传递到use-class，该文件本身是从另一个HTL文件中调用的， `data-sly-template` 其中包含需要传递的参数。

例如，让我们在现有示例旁边 `data-sly-template` 创建一个单独的文件。 我们将调用新文件 `extra.html`。 它包含一 `data-sly-template` 个名为 `extra`:

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

模板 `extra`采用单个参数 `text`。 然后，它使用本地名 `ExtraHelper` 初始化Java `extraHelper` use-class，并将模板参数的值 `text` 传递给它作为use-class参数 `text`。

模板的主体获取属性( `extraHelper.reversedText` 实际调用该属性)并 `ExtraHelper.getReversedText()`显示该值。

我们还会调整现有模 `info.html` 板，以使用此新模板：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info"
     data-sly-use.extra="extra.html">

  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>

  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

该文 `info.html` 件现在包 `data-sly-use` 含两个语句，其中一个是导入Java `Info` use-class的原始语句，另一个是导入本地名称下的模板文件的新语句 `extra`。

请注意，我们可能已将模板块放在文件 `info.html` 中以避免第二个模板 `data-sly-use`，但单独的模板文件更常见、可重用。

类 `Info` 像以前一样使用，调用其getter方 `getLowerCaseTitle()` 法 `getLowerCaseDescription()` 并通过其相应的HTL属性 `info.lowerCaseTitle` 和 `info.lowerCaseDescription`。

然后，对模 `data-sly-call` 板执行 `extra` 一个操作，并将值 `properties.description` 作为参数传递 `text`。

Java use-class已更 `Info.java` 改为处理新文本参数：

### `/apps/my-example/component/info/ExtraHelper.java` {#apps-my-example-component-info-extrahelper-java}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class ExtraHelper extends WCMUsePojo {
    private String reversedText;
    ...

    @Override
    public void activate() throws Exception {
        String text = get("text", String.class);
        reversedText = new StringBuilder(text).reverse().toString();

        ...
    }

    public String getReversedText() {
        return reversedText;
    }
}
```

参 `text` 数被检索为 `get("text", String.class)`，值被反转，并通过getter作为HTL `reversedText` 对象可用 `getReversedText()`。

### 捆绑的Java类 {#bundled-java-class}

对于捆绑使用类，必须使用标准OSGi捆绑部署机制在AEM中编译、打包和部署该类。 与本地安装不同，use-class包声 **明应正** 常命名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

并且，语 `data-sly-use` 句必须引用完全限定的类名，而不是只引用本地类名：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### 替代 `WCMUsePojo` {#alternatives-to-wcmusepojo}

创建Java用类的最常见方法是扩展 `WCMUsePojo`。 但是，还有许多其他选项。 要了解这些变体，了解HTL语句在内 `data-sly-use` 部的工作方式会有所帮助。

假定您有以下语 `data-sly-use` 句：

**`<div data-sly-use.`** `localName`**`="`** `UseClass`**`">`**

系统按如下方式处理该语句：

(1)

* 如果与HTL文件位 `UseClass.java` 于同一目录中存在本地文件，请尝试编译并加载该类。 如果成功，请转到(2)。
* 否则，解 `UseClass` 释为完全限定的类名称，并尝试从OSGi环境加载它。 如果成功，请转到(2)。
* 否则，解 `UseClass` 释为HTL或JavaScript文件的路径并加载该文件。 如果goto成功(4)。

(2)

* 尝试调整当前 `Resource` 的 `UseClass`。 如果成功，请转到(3)。
* 否则，请尝试将当前版本调 `Request` 整为 `UseClass`。 如果成功，请转到(3)。
* 否则，尝试使用 `UseClass` 零参数构造函数实例化。 如果成功，请转到(3)。

(3)

* 在HTL中，将新改编或创建的对象绑定到名称 `localName`。
* 如 `UseClass` 果实 [`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) 现，则调用方 `init` 法，传递当前执行上下文(以对象的形 `javax.scripting.Bindings` 式)。

(4)

* 如 `UseClass` 果是包含模板的HTL文件的路 `data-sly-template`径，请准备模板。
* 否则， `UseClass` 如果是JavaScript use-class的路径，请准备use-class(请参 [阅JavaScript Use-API](use-api-javascript.md))。

有关上述描述的几个要点：

* 任何可从中调整、 `Resource`可从中 `Request`调整或具有零参数构造函数的类都可以是使用类。 类无需扩展甚至 `WCMUsePojo` 实现 `Use`。
* 但是，如果use-class实 *现了* ，则其方法将自 `Use``init` 动与当前上下文一起调用，从而允许您将初始化代码放置在依赖于该上下文的位置。
* 扩展的使用类 `WCMUsePojo` 只是实现的一个特殊情况 `Use`。 它提供了方便的上下文方法，并 `activate` 且自动调用其方法 `Use.init`。

### 直接实现接口使用 {#directly-implement-interface-use}

创建use类的最常见方法是扩展， `WCMUsePojo`但也可以直接实现接 [`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) 口本身。

该接 `Use` 口仅定义一种方法：

[`public void init(javax.script.Bindings bindings)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))

初始化 `init` 类时，将调用该方法，该类具有 `Bindings` 一个对象，该对象包含所有上下文对象以及传递到use-class的任何参数。

必须使用对象显式实现所有附 `WCMUsePojo.getProperties()`加功能(如等效 [`javax.script.Bindings`](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html) 功能)。 例如：

### `Info.java` {#info-java}

```java
import io.sightly.java.api.Use;

public class MyComponent implements Use {
   ...
    @Override
    public void init(Bindings bindings) {

        // All standard objects/binding are available
        Resource resource = (Resource)bindings.get("resource");
        ValueMap properties = (ValueMap)bindings.get("properties");
        ...

        // Parameters passed to the use-class are also available
        String param1 = (String) bindings.get("param1");
    }
    ...
}
```

当您希望将现有类 `Use` 的子类用 `WCMUsePojo` 作use-class时，自己实现接口而不是扩展。

### 从资源调整 {#adaptable-from-resource}

另一个选项是使用可调整的辅助类 `org.apache.sling.api.resource.Resource`。

假设您需要编写一个HTL脚本来显示DAM资产的mimetype。 在这种情况下，您知道调用HTL脚本时，它将位于用nodetype包装JCR `Resource` 的上下文 `Node` 中 `dam:Asset`。

您知道节点 `dam:Asset` 具有如下结构：

### 存储库结构 {#repository-structure}

```java
{
  "content": {
    "dam": {
      "geometrixx": {
        "portraits": {
          "jane_doe.jpg": {
            ...
          "jcr:content": {
            ...
            "metadata": {
              ...
            },
            "renditions": {
              ...
              "original": {
                ...
                "jcr:content": {
                  "jcr:primaryType": "nt:resource",
                  "jcr:lastModifiedBy": "admin",
                  "jcr:mimeType": "image/jpeg",
                  "jcr:lastModified": "Fri Jun 13 2014 15:27:39 GMT+0200",
                  "jcr:data": ...,
                  "jcr:uuid": "22e3c598-4fa8-4c5d-8b47-8aecfb5de399"
                }
              },
              "cq5dam.thumbnail.319.319.png": {
                  ...
              },
              "cq5dam.thumbnail.48.48.png": {
                  ...
              },
              "cq5dam.thumbnail.140.100.png": {
                  ...
              }
            }
          }  
        }
      }
    }
  }
}
```

此处，我们将作为示例项目geometrixx的一部分显示AEM默认安装附带的资产（JPEG图像）。 资产被调用， `jane_doe.jpg` 其mimetype被调 `image/jpeg`用。

要从HTL中访问资产，您可以在语 [`com.day.cq.dam.api.Asset`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html) 句中声明为 `data-sly-use` 类，然后使用get方法 `Asset` 检索所需的信息。 例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

该语 `data-sly-use` 句指示HTL将当前代码改 `Resource` 编为 `Asset` 一个代码，并为它指定本地名称 `asset`。 然后，它调用 `getMimeType` 使用 `Asset` HTL getter短格式的方法： `asset.mimeType`.

### 根据请求调整 {#adaptable-from-request}

还可以使用任何可适应 [`org.apache.sling.api.SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)

与上述情况一样，可从中调整 `Resource`的使用类在语句 [`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 中可指定 `data-sly-use` 。 执行时，当前请求将与给定的类相适应，并且生成的对象将在HTL中可用。
