---
title: HTL Java Use-API
description: 利用 HTML 模板语言 (HTL) Java Use-API，HTL 文件可以访问自定义 Java 类中的 helper 方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '2558'
ht-degree: 100%

---

# HTL Java Use-API {#htl-java-use-api}

利用 HTML 模板语言 (HTL) Java Use-API，HTL 文件可以通过 `data-sly-use` 访问自定义 Java 类中的 helper 方法。这样可以将所有复杂的业务逻辑封装在 Java 代码中，而 HTL 代码只处理直接标记生成。

Java Use-API 对象可以是一个简单的 POJO，由特定的实现通过 POJO 的默认构造器实例化。

Use-API POJO 也可以使用以下签名公开名为 init 的公共方法：

```java
    /**
     * Initializes the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/
    public void init(javax.script.Bindings bindings);
```

`bindings` 映射可以包含一些对象，这些对象向当前执行的 HTL 脚本提供上下文，以便 Use-API 对象可使用这些上下文进行处理。

## 简单示例 {#a-simple-example}

我们先来看一个没有 use 类的 HTL 组件。它由单个文件 `/apps/my-example/components/info.html` 组成

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我们还为该组件添加了一些内容，以便在 `/content/my-example/` 上呈现：

### `http://<host>:<port>/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

当用户访问此内容时，系统会执行 HTL 文件。在 HTL 代码内，我们使用上下文对象 `properties` 来访问当前资源的 `title` 和 `description` 并显示它们。输出 HTML 会是这样：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加一个 use 类 {#adding-a-use-class}

目前 **info** 组件不需要 use 类来执行其（非常简单的）功能。有些情况下，您需要做的事情无法在 HTL 中完成，因此需要 use 类。但请记住：

>[!NOTE]
>
>只有当某些事情无法单独在 HTL 中完成时，才应使用 use 类。

例如，假设您想让 `info` 组件显示资源的 `title` 和 `description` 属性且必须全部小写。由于 HTL 没有将字符串转换为小写的方法，所以您需要 use 类。我们的解决办法是，添加一个 Java use 类并更改 `info.html`，如下所示：

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

在以下部分中，我们将讲解代码的不同部分。

### 局部与捆绑 Java 类 {#local-vs-bundle-java-class}

Java use 类有两种安装方式：**局部**&#x200B;或&#x200B;**捆绑**。本例使用局部安装。

在局部安装中，Java 源文件与 HTL 文件放在一起，位于同一个存储库文件夹下。根据需要自动编译源。无需单独的编译或打包步骤。

在捆绑安装中，必须使用标准 AEM 捆绑部署机制在 OSGi 捆绑包中编译和部署 Java 类（参阅[捆绑的 Java 类](#bundled-java-class)）。

>[!NOTE]
>
>当 use 类特定于所针对的组件时，建议使用&#x200B;**局部 Java use 类**。
>
>当 Java 代码实现从多个 HTL 组件访问的服务时，建议使用&#x200B;**捆绑 Java use 类**。

### Java 包是存储库路径 {#java-package-is-repository-path}

使用局部安装时，use 类的包名称必须与存储库文件夹位置的名称匹配，在包名称中将路径中的所有连字符替换为下划线。

在此情形中，`Info.java` 位于 `/apps/my-example/components/info`，因此包名称为 `apps.my_example.components.info`：

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
>在 AEM 开发中，建议在存储库项目名称中使用连字符。但是在 Java 包名称中，连字符是无效的。因此，**存储库路径中的所有连字符在包名称中必须转换为下划线**。

### 扩展 `WCMUsePojo` {#extending-wcmusepojo}

尽管 Java 类与 HTL 结合的方法很多（请参阅 `WCMUsePojo` 的替代项），但更简单的方法是扩展 `WCMUsePojo` 类：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化类 {#initializing-the-class}

从 `WCMUsePojo` 扩展 use 类时，通过覆盖 `activate` 方法来执行初始化：

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

通常，[activate](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) 方法用于根据当前上下文（例如当前请求和资源）预先计算和存储（在成员变量中）HTL 代码中所需的值。

`WCMUsePojo` 类提供对 HTL 文件中同样的上下文对象集的访问（请参阅 [全局对象](global-objects.md)）。

在扩展 `WCMUsePojo` 的类中，可以使用名称访问对象

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，可以通过适当的&#x200B;**方便方法**&#x200B;直接访问常用的上下文对象：

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [Page](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [Page](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [Designer](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [Design](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [Style](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [Component](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getInheritedProperties) |
| [Resource](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResourceResolver](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter 方法 {#getter-methods}

一旦 use 类初始化，HTL 文件即会运行。在此阶段，HTL 通常会拉入 use 类的各种成员变量的状态，并渲染它们以便呈现。

要提供从 HTL 文件内访问这些值的权限，您必须根据以下命名惯例在 use 类中定义自定义 getter 方法：

* 表单 `getXyz` 的一种方法会在 HTL 文件内公开一个名为 `xyz` 的对象属性。

在下例中，方法 `getTitle` 和 `getDescription` 使对象属性 `title` 和 `description` 在 HTL 文件上下文内变为可访问。

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

### data-sly-use 属性 {#data-sly-use-attribute}

`data-sly-use` 属性用于初始化 HTL 代码中的 use 类。在我们的示例中，`data-sly-use` 属性声明我们需要使用类 `Info`：我们可以只使用该类的局部名称，因为我们使用的是局部安装（Java 源文件与 HTL 文件放在同一文件夹中）。如果我们使用捆绑安装，则须指定完全限定的类名称。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 局部标识符 {#local-identifier}

标识符 `info`（位于 `data-sly-use.info` 中的点后）用于在 HTL 文件中标识类。声明该标识符后，其作用范围是整个文件。这不限于包含 `data-sly-use` 语句的元素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 获取属性 {#getting-properties}

标识符 `info` 然后用于访问通过 getter 方法 `Info.getTitle` 和 `Info.getDescription` 公开的对象属性 `title` 和 `description`。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 输出 {#output}

现在，当我们访问 `/content/my-example.html` 时，它会返回以下 HTML：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 基本功能拓展 {#beyond-the-basics}

在本节中，我们来介绍一些超出上述简单示例的进一步的功能：

* 将参数传递给 use 类。
* 捆绑的 Java use 类。
* `WCMUsePojo` 的替代项。

### 传递参数 {#passing-parameters}

初始化时，可将参数传递给 use 类。例如，我们可以这样做：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

这里，我们传递一个名为 `text` 的参数。use 类然后通过 `info.upperCaseText` 将我们检索的字符串转换为大写并显示结果。以下是调整后的 use 类：

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

通过 `WCMUsePojo` 方法 [`<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html) 访问该参数

在我们的例子中，语句为：

`get("text", String.class)`

字符串然后通过以下方法反转并公开：

`getReverseText()`

### 仅从 data-sly-template 传递字符串 {#only-pass-parameters-from-data-sly-template}

尽管上述示例在技术上是正确的，但实际上，当相关值在 HTL 代码的执行上下文中可用（或者，很明显，像上面那样，值是静态的）时，从 HTL 传递一个值来初始化 use 类没有多大意义。

原因在于，use 类始终能够访问与 HTL 代码相同的执行上下文。由此引出一个最佳实践的导入点：

>[!NOTE]
>
>仅在以下情况才应向 use 类传递参数：使用 use 类的 `data-sly-template` 文件本身通过参数从另一个 HTL 文件调用，而这些参数需要传递。

例如，我们在现有示例之外又创建了一个单独的 `data-sly-template` 文件。该新文件名为 `extra.html`。它包含一个 `data-sly-template` 代码块，名为 `extra`：

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

模板 `extra` 采用单个参数 `text`。然后，它使用局部名称 `extraHelper` 初始化 Java use 类 `ExtraHelper`，向其传递模板参数 `text` 的值作为 use 类参数 `text`。

模板主体获取属性 `extraHelper.reversedText`（实质上是调用 `ExtraHelper.getReversedText()`）并显示该值。

我们还改写了现有 `info.html` 以使用这一新的模板：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info"
     data-sly-use.extra="extra.html">

  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>

  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

文件 `info.html` 现在包含两个 `data-sly-use` 语句，原始语句导入 `Info` Java use 类，新语句在局部名称 `extra` 下导入模板文件。

注意，我们本可以将模板代码块放在 `info.html` 文件内，以避免第二个 `data-sly-use`，但是单独的模板文件更常见，也更易于重用。

与之前一样，使用 `Info` 类，通过相应的 HTL 属性 `info.lowerCaseTitle` 和 `info.lowerCaseDescription` 调用 getter 方法 `getLowerCaseTitle()` 和 `getLowerCaseDescription()`。

然后，我们对模板 `extra` 执行 `data-sly-call`，向其传递值 `properties.description` 作为参数 `text`。

Java use 类`Info.java` 改为处理新的文本参数：

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

通过 `get("text", String.class)` 来检索 `text` 参数，通过 getter `getReversedText()` 反转值并用作 HTL 对象 `reversedText`。

### 捆绑的 Java 类 {#bundled-java-class}

对于捆绑的 use 类，必须使用标准 OSGi 捆绑包部署机制在 AEM 中编译、打包和部署类。与局部安装相反，应正常命名 use 类&#x200B;**包声明**：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

同时，`data-sly-use` 语句必须引用完全限定的类名称，而非仅局部类名称：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### `WCMUsePojo` 的替代项 {#alternatives-to-wcmusepojo}

创建 Java use 类的常用方法是扩展 `WCMUsePojo`。不过，也有其他许多选项。要了解这些变体，有必要了解 HTL `data-sly-use` 语句背后的工作原理。

假设有下面的 `data-sly-use` 语句：

**`<div data-sly-use.`** `localName`**`="`**`UseClass`**`">`**

系统按照如下方法处理该语句：

(1)

* 如果 HTL 文件所在的目录下存在局部文件 `UseClass.java`，则尝试编译和加载该类。如果成功，则转至 (2)。
* 否则，将 `UseClass` 解释为一个完全限定的类名称，尝试从 OSGi 环境加载它。如果成功，则转至 (2)。
* 否则，将 `UseClass` 解释为 HTL 或 JavaScript 文件的路径并加载该文件。如果成功，则转至 (4)。

(2)

* 尝试将当前 `Resource` 改为 `UseClass`。如果成功，则转至 (3)。
* 否则，尝试将当前 `Request` 改为 `UseClass`。如果成功，则转至 (3)。
* 否则，尝试使用零参数构造器初始化 `UseClass`。如果成功，则转至 (3)。

(3)

* 在 HTL 内，绑定新修改或创建的对象到名称 `localName`。
* 如果 `UseClass` 实现 [`io.sightly.java.api.Use`](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)，则调用 `init` 方法，传递当前执行上下文（以 `javax.scripting.Bindings` 对象形式）。

(4)

* 如果 `UseClass` 是包含 `data-sly-template` 的 HTL 文件的路径，则准备模板。
* 否则，如果 `UseClass` 是 JavaScript use 类的路径，则准备 use 类（请参阅 [JavaScript Use-API](use-api-javascript.md)）。

关于上述描述，我们总结了以下几个要点：

* 任何改写自 `Resource` 或 `Request` 或者具有零参数构造器的类都可以是 use 类。该类不必扩展 `WCMUsePojo`，甚至不必实现 `Use`。
* 但是，如果 use 类&#x200B;*的确*&#x200B;实现了 `Use`，则会自动使用当前上下文调用其 `init` 方法，从而允许您根据该上下文将初始化代码放在那里。
* 扩展 `WCMUsePojo` 的 use 类只是实现 `Use` 的一个特例。它提供方便的上下文方法，并且从 `Use.init` 自动调用其 `activate` 方法。

### 直接实现接口使用 {#directly-implement-interface-use}

尽管创建 use 类的更常见方法是扩展 `WCMUsePojo`，但也可以直接实现 [`io.sightly.java.api.Use`](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) 接口本身。

`Use` 接口只定义一个方法：

[`public void init(javax.script.Bindings bindings)`](https://helpx.adobe.com/cn/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))

将类初始化时，将通过一个 `Bindings` 对象来调用 `init` 方法，该对象保留所有上下文对象和传递给 use 类的任何参数。

必须使用 [`javax.script.Bindings`](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html) 对象显式实现所有额外的动能（如 `WCMUsePojo.getProperties()` 的同等功能）。例如：

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

通常，当您希望使用现有类的子类作为 use 类时，才会自己实现`Use` 接口而不是扩展 `WCMUsePojo`。

### 可从资源改写 {#adaptable-from-resource}

另一个办法是使用从 `org.apache.sling.api.resource.Resource` 改写的 helper 类。

假设您需要编写一个显示 DAM 资产的 mimetype 的 HTL 脚本。在这种情况下，您知道，当您的 HTL 脚本被调用时，它会在 `Resource` 的上下文内，该资源打包的 JCR `Node` 具有节点类型 `dam:Asset`。

您知道，`dam:Asset` 节点具有类似如下的结构：

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

这里我们显示该资产（一个 JPEG 图像），它附带在默认 AEM 安装中，作为示例项目 geometrixx 的一部分。该资产名为 `jane_doe.jpg`，其 mimetype 为 `image/jpeg`。

要从 HTL 内访问该资产，可以在 `data-sly-use` 语句中声明 [`com.day.cq.dam.api.Asset`](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html) 为类，然后使用 `Asset` 的 get 方法检索所需的信息。例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

`data-sly-use` 语句引导 HTL 将当前 `Resource` 改为 `Asset`，并赋予其局部名称 `asset`。然后，它使用 HTL getter 简写 `asset.mimeType` 来调用 `Asset` 的 `getMimeType` 方法。

### 可从请求改写 {#adaptable-from-request}

也可以将从 [`org.apache.sling.api.SlingHttpServletRequest`](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 改写的任何类作为 use 类。

像上面的利用从 `Resource` 改写的 use 类一样，可以在 `data-sly-use` 语句中指定从 [`SlingHttpServletRequest`](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 改写的 use 类。执行时，当前请求将改为给定类，生成的对象将在 HTL 内可用。
