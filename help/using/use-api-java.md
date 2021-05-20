---
title: HTL Java Use-API
description: HTML 模板语言 - HTL - Java Use-API 让 HTL 文件可以访问自定义 Java 类中的 Helper 方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '2558'
ht-degree: 1%

---

# HTL Java Use-API {#htl-java-use-api}

HTML模板语言(HTL)Java Use-API允许HTL文件通过`data-sly-use`访问自定义Java类中的Helper方法。 这允许将所有复杂的业务逻辑封装在Java代码中，而HTL代码仅处理直接标记生产。

Java Use-API对象可以是一个简单的POJO，由特定实施通过POJO的默认构造函数进行实例化。

Use-API POJO还可以使用以下签名公开一个名为init的公共方法：

```java
    /**
     * Initializes the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/
    public void init(javax.script.Bindings bindings);
```

`bindings`映射可以包含为当前执行的HTL脚本提供上下文的对象，Use-API对象可将其用于处理该脚本。

## {#a-simple-example}的简单示例

我们将从没有use类的HTL组件开始。 它由一个文件`/apps/my-example/components/info.html`组成

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我们还为此组件添加了一些内容以在`/content/my-example/`呈现：

### `http://<host>:<port>/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

访问此内容时，将执行HTL文件。 在HTL代码中，我们使用上下文对象`properties`访问当前资源的`title`和`description`并显示它们。 输出HTML将为：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加Use-Class {#adding-a-use-class}

**info**&#x200B;组件原样不需要use类即可执行其（非常简单）函数。 但是，在某些情况下，您需要执行无法在HTL中完成的操作，因此需要使用类。 但请记住以下几点：

>[!NOTE]
>
>仅当某些操作无法在HTL中完成时，才应使用use-class。

例如，假定您希望`info`组件显示资源的`title`和`description`属性，但全部以小写形式显示。 由于HTL没有用于小写字符串的方法，因此您将需要一个use类。 为此，我们可以添加Java use-class并按如下方式更改`info.html`:

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

在以下部分中，我们将介绍代码的不同部分。

### 本地与包Java类{#local-vs-bundle-java-class}

Java用例类可通过两种方式安装：**local**&#x200B;或&#x200B;**bundle**。 此示例使用本地安装。

在本地安装中，Java源文件与HTL文件一起放置在同一存储库文件夹中。 根据需要自动编译该来源。 无需单独的编译或打包步骤。

在包安装中，必须使用标准AEM包部署机制在OSGi包中编译和部署Java类（请参阅[捆绑的Java类](#bundled-java-class)）。

>[!NOTE]
>
>当use类特定于相关组件时，建议使用&#x200B;**本地Java use-class**。
>
>当Java代码实现从多个HTL组件访问的服务时，建议使用&#x200B;**包Java use-class**。

### Java包是存储库路径{#java-package-is-repository-path}

使用本地安装时，use-class的包名称必须与存储库文件夹位置的名称匹配，路径中的任何连字符在包名称中替换为下划线。

在这种情况下，`Info.java`位于`/apps/my-example/components/info`，因此包为`apps.my_example.components.info`:

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
>在AEM开发中，建议在存储库项目的名称中使用连字符。 但是，连字符在Java包名称中是非法的。 因此，必须将存储库路径中的&#x200B;**所有连字符转换为包名称**&#x200B;中的下划线。

### 扩展`WCMUsePojo` {#extending-wcmusepojo}

虽然有多种将Java类与HTL结合的方法（请参阅`WCMUsePojo`的替代方法），但最简单的方法是扩展`WCMUsePojo`类：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化类{#initializing-the-class}

从`WCMUsePojo`扩展use类时，通过覆盖`activate`方法来执行初始化：

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

通常， [activate](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)方法用于根据当前上下文（例如，当前请求和资源）预计算并存储HTL代码中所需的值（在成员变量中）。

`WCMUsePojo`类提供对与HTL文件中可用的上下文对象集的访问权限（请参阅[全局对象](global-objects.md)）。

在扩展`WCMUsePojo`的类中，可使用

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，常用的上下文对象可以通过相应的&#x200B;**方便方法**&#x200B;直接访问：

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [Designer](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [设计](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [样式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [组件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getInheritedProperties) |
| [资源](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResourceResolver](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter方法{#getter-methods}

use-class初始化后，将运行HTL文件。 在此阶段，HTL通常会提取use-class的各个成员变量的状态，并渲染它们以进行呈现。

要在HTL文件中提供对这些值的访问，您必须根据以下命名约定在use-class中定义自定义getter方法：

* 格式为`getXyz`的方法将在HTL文件中显示一个名为`xyz`的对象属性。

在以下示例中，方法`getTitle`和`getDescription`导致对象属性`title`和`description`在HTL文件的上下文中变为可访问：

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

### data-sly-use属性{#data-sly-use-attribute}

`data-sly-use`属性用于初始化HTL代码中的use类。 在本例中， `data-sly-use`属性声明我们要使用类`Info`。 我们可以仅使用类的本地名称，因为我们使用的是本地安装（将Java源文件放置在与HTL文件相同的文件夹中）。 如果使用捆绑安装，则必须指定完全限定的类名称。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本地标识符{#local-identifier}

在HTL文件中使用标识符`info`（在`data-sly-use.info`中的点之后）来标识类。 声明后，此标识符的范围在文件中是全局的。 它不限于包含`data-sly-use`语句的元素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 获取属性{#getting-properties}

然后，标识符`info`用于访问通过getter方法`Info.getTitle`和`Info.getDescription`公开的对象属性`title`和`description`。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 输出 {#output}

现在，当我们访问`/content/my-example.html`时，它将返回以下HTML:

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 超越基础知识{#beyond-the-basics}

在本节中，我们将介绍一些除上述简单示例之外的其他功能：

* 将参数传递给使用类。
* 捆绑的Java用类。
* `WCMUsePojo`的替代项

### 传递参数{#passing-parameters}

参数可在初始化时传递到use类。 例如，我们可以执行如下操作：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

下面我们将传递一个名为`text`的参数。 use-class然后将我们检索并显示结果的字符串上方加上`info.upperCaseText`。 以下是调整后的使用类：

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

通过`WCMUsePojo`方法[`<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)访问参数

在本例中，声明：

`get("text", String.class)`

然后，该字符串会通过以下方法进行反转并公开：

`getReverseText()`

### 仅从data-sly-template {#only-pass-parameters-from-data-sly-template}传递参数

虽然上述示例在技术上是正确的，但是当相关值在HTL代码的执行上下文中可用（或者，实际上，如上所述，该值是静态的）时，从HTL传递值以初始化使用类实际上并没有多大意义。

其原因是use-class将始终具有与HTL代码相同的执行上下文的访问权限。 这引出了最佳实践的导入点：

>[!NOTE]
>
>只有在`data-sly-template`文件中使用use-class时，才应将参数传递到use-class，该文件本身是从另一个HTL文件中调用的，其中包含需要传递的参数。

例如，让我们在现有示例旁边创建一个单独的`data-sly-template`文件。 我们将调用新文件`extra.html`。 它包含一个名为`extra`的`data-sly-template`块：

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

模板`extra`采用单个参数`text`。 然后，它使用本地名称`extraHelper`初始化Java use-class `ExtraHelper`，并将模板参数`text`的值作为use-class参数`text`传递给它。

模板正文将获取属性`extraHelper.reversedText`（在引擎罩下，该属性实际调用`ExtraHelper.getReversedText()`）并显示该值。

我们还会调整现有的`info.html`以使用此新模板：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info"
     data-sly-use.extra="extra.html">

  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>

  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

文件`info.html`现在包含两个`data-sly-use`语句，其中原始语句用于导入`Info` Java use-class，新语句用于导入本地名称`extra`下的模板文件。

请注意，我们本可以将模板块放在`info.html`文件中以避免第二个`data-sly-use`文件，但单独的模板文件更常见，更可重用。

与以前一样，使用`Info`类，通过其相应的HTL属性`info.lowerCaseTitle`和`info.lowerCaseDescription`调用其getter方法`getLowerCaseTitle()`和`getLowerCaseDescription()`。

然后，对模板`extra`执行`data-sly-call`，并将值`properties.description`作为参数`text`传递。

更改了Java use-class `Info.java`以处理新的文本参数：

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

使用`get("text", String.class)`检索`text`参数时，该值会反转，并通过getter `getReversedText()`用作HTL对象`reversedText`。

### 捆绑的Java类{#bundled-java-class}

对于包使用类，必须使用标准OSGi包部署机制在AEM中编译、打包和部署类。 与本地安装不同， use-class **package declaration**&#x200B;应正常命名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

和，`data-sly-use`语句必须引用完全限定的类名称，而不是仅引用本地类名称：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### `WCMUsePojo` {#alternatives-to-wcmusepojo}的替代项

创建Java用例类的最常见方式是扩展`WCMUsePojo`。 但是，还有许多其他选项。 要了解这些变体，了解HTL `data-sly-use`语句在引擎罩下的工作方式会有所帮助。

假设您有以下`data-sly-use`语句：

**`<div data-sly-use.`** `localName`**`="`**`UseClass`**`">`**

系统会按如下方式处理该语句：

(1)

* 如果本地文件`UseClass.java`与HTL文件位于同一目录中，请尝试编译并加载该类。 如果成功，请转至(2)。
* 否则，请将`UseClass`解释为完全限定的类名称，并尝试从OSGi环境加载该类名称。 如果成功，请转至(2)。
* 否则，将`UseClass`解释为HTL或JavaScript文件的路径，并加载该文件。 如果成功转到(4)。

(2)

* 尝试将当前`Resource`调整为`UseClass`。 如果成功，请转到(3)。
* 否则，请尝试将当前`Request`调整为`UseClass`。 如果成功，请转到(3)。
* 否则，请尝试使用零参数构造函数实例化`UseClass`。 如果成功，请转到(3)。

(3)

* 在HTL中，将新修改或创建的对象绑定到名称`localName`。
* 如果`UseClass`实现[`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)，则调用`init`方法，并传递当前执行上下文（以`javax.scripting.Bindings`对象的形式）。

(4)

* 如果`UseClass`是包含`data-sly-template`的HTL文件的路径，请准备模板。
* 否则，如果`UseClass`是JavaScript use-class的路径，请准备use-class（请参阅[JavaScript Use-API](use-api-javascript.md)）。

有关上述描述的几个要点：

* 任何可从`Resource`适应、可从`Request`适应或具有零参数构造函数的类都可以是使用类。 类不必扩展`WCMUsePojo`甚至不必实现`Use`。
* 但是，如果use-class *does*&#x200B;实现`Use`，则其`init`方法将自动使用当前上下文调用，从而允许您将初始化代码放置到依赖于该上下文的位置。
* 扩展`WCMUsePojo`的用类只是实现`Use`的一种特殊情况。 它提供了方便的上下文方法，并且其`activate`方法会从`Use.init`自动调用。

### 使用{#directly-implement-interface-use}直接实施接口

创建use-class的最常见方式是扩展`WCMUsePojo`，但也可以直接实现[`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)接口本身。

`Use`接口仅定义一种方法：

[`public void init(javax.script.Bindings bindings)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))

在初始化具有`Bindings`对象的类时，将调用`init`方法，该对象包含所有上下文对象以及传递到use-class的任何参数。

必须使用[`javax.script.Bindings`](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)对象显式实现所有其他功能（如`WCMUsePojo.getProperties()`的等效函数）。 例如：

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

当您希望将现有类的子类用作use类时，您可以自行实现`Use`接口而不是扩展`WCMUsePojo`。

### 从资源{#adaptable-from-resource}进行适应性调整

另一个选项是使用从`org.apache.sling.api.resource.Resource`适应的帮助程序类。

假设您需要编写一个HTL脚本来显示DAM资产的mimetype。 在本例中，您知道调用HTL脚本时，该脚本将位于`Resource`的上下文中，该将JCR `Node`与nodetype `dam:Asset`封装在一起。

您知道`dam:Asset`节点的结构如下：

### 存储库结构{#repository-structure}

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

在本例中，我们将显示作为示例项目geometrixx的一部分默认安装AEM的资产（JPEG图像）。 资产名为`jane_doe.jpg`，其mimetype为`image/jpeg`。

要从HTL中访问资产，您可以在`data-sly-use`语句中声明[`com.day.cq.dam.api.Asset`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)为类，然后使用`Asset`的get方法检索所需信息。 例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

`data-sly-use`语句指示HTL将当前`Resource`调整为`Asset`，并为其指定本地名称`asset`。 然后，它使用HTL getter简写调用`Asset`的`getMimeType`方法：`asset.mimeType`。

### 根据请求{#adaptable-from-request}进行适应性调整

也可以作为使用类使用任何可从[`org.apache.sling.api.SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)适应的类

与上述从`Resource`适应的使用类一样，可以在`data-sly-use`语句中指定从[`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)适应的使用类。 执行时，当前请求将适用于给定的类，并且生成的对象将在HTL中可用。
