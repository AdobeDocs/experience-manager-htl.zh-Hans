---
title: HTL Java Use-API
seo-title: HTL Java Use-API
description: 'HTML 模板语言 - HTL - Java Use-API 让 HTL 文件可以访问自定义 Java 类中的 Helper 方法。 '
seo-description: 'HTML 模板语言 - HTL - Java Use-API 让 HTL 文件可以访问自定义 Java 类中的 Helper 方法。 '
uuid: b340f8f7-a193-45c8-aa39-5c6e2c0194ea
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 126ebc9d-5f7b-47a4-aea2-c8840d34864c
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 48f09cfbdc5ef0bc5b951de1444ce40ea53dc07f

---


# HTL Java Use-API{#htl-java-use-api}

HTML模板语言(HTL)Java Use-API使HTL文件能够访问自定义Java类中的帮助程序方法。 这允许将所有复杂的业务逻辑封装在Java代码中，而HTL代码只处理直接标记制作。

## 简单示例 {#a-simple-example}

我们将从没有use类 *的* HTL组件开始。 它由一个文件组成， `/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我们还为此组件添加一些内容，以在以下位置进行渲染 **`/content/my-example/`**:

### `http://localhost:4502/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

访问此内容时，将执行HTL文件。 在HTL代码中，我们使用上下文对象**`properties`**访问当前资源并显 `title` 示 `description` 它们。 输出HTML将为：

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加Use类 {#adding-a-use-class}

info **组件** ，它现在不需要use类就可以执行其（非常简单）函数。 但是，在某些情况下，您需要做在HTL中无法完成的事情，因此您需要使用类。 但请记住以下几点：

>[!NOTE]
>
>*仅当某些操作无法在HTL中完成时，才应使用use类。*

例如，假设您希望组件 `info` 显示资源的 `title` 和 **`description`** 属性，但全部以小写形式显示。 由于HTL没有用于小写字符串的方法，因此您需要use-class。 我们可以通过添加Java use-class并按如下方式更改来 **`info.html`** 实现此操作：

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

在以下几节中，我们将遍历代码的不同部分。

### 本地与捆绑Java类 {#local-vs-bundle-java-class}

Java use-class可以通过两种方式安装：本 **地** 或 **捆绑**。 *此示例使用本地安装。*

In a local install, the Java source file is placed alongside the HTL file, in the same repository folder. 该源将根据需要自动编译。 无需单独的编译或打包步骤。

In a bundle install, the Java class must be compiled and deployed within an OSGi bundle using the standard AEM bundle deployment mechanism (see Bundled Java Class).[](#bundled-java-class)

>[!NOTE]
>
>A local Java use-class is recommended when the use-class is specific to the component in question.****
>
>A bundle Java use-class is recommended when the Java code implements a service that is accessed from multiple HTL components.****

### Java package is repository path {#java-package-is-repository-path}

When a local install is used, the package name of the use-class must match that of the repository folder location, with any hyphens in the path replaced by underscores in the package name.

In this case  is located at  so the package is  :**`Info.java`****`/apps/my-example/components/info`****`apps.my_example.components.info`**

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
>Using hyphens in the names of repository items is a recommended practice in AEM developement. However, hyphens are illegal within Java package names. 因此，必须将 **存储库路径中的所有连字符转换为包名称中的下划线**。

### Extending  `WCMUsePojo`{#extending-wcmusepojo}

While there are number of ways of incorporating a Java class with HTL (see Alternatives to ), the simplest is to extend the  class:`WCMUsePojo``WCMUsePojo`

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    
    ...
}
```

### Initializing the class {#initializing-the-class}

从扩展use-class时，通过覆盖 **`WCMUsePojo`**&#x200B;方法来执行初始化 **`activate`** 操作：

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

Typically, the activate method is used to precompute and store (in member variables) the values needed in your HTL code, based on the current context (the current request and resource, for example).[](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

The  class provides access to the same set of context objects as are available within an HTL file (see Global Objects).`WCMUsePojo`[](global-objects.md)

在扩展的类中， **`WCMUsePojo`**&#x200B;可以使用名称访问上 *下文对象*

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

或者，常用的上下文对象也可以通过相应的简便方法直 **接访问**:

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [设计人员](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [设计](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [样式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [组件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse#getInheritedProperties.html()) |
| [资源](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResourceResolver](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter方法 {#getter-methods}

use-class初始化后，将运行HTL文件。 在此阶段，HTL通常会拉入use-class的各个成员变量的状态，并呈现这些变量以供演示。

To provide access to these values from within the HTL file you must define custom getter methods in the use-class according to the following naming convention:****

* A method of the form  will expose within the HTL file an object property called xyz.**`getXyz`********

For example, in the following example, the methods  and  result in the object properties  and  becoming accessible within the context of the HTL file:**`getTitle`****`getDescription`****`title`****`description`**

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

### data-sly-use attribute {#data-sly-use-attribute}

The  attribute is used to initialize the use-class within your HTL code. **`data-sly-use`** In our example, the  attribute declares that we want to use the class . `data-sly-use`**`Info`** We can use just the local name of the class because we are using a local install (having placed the Java source file is in the same folder as the HTL file). If we were using a bundle install we would have to specify the fully qualified classname (See Use-class Bundle Install).[](#LocalvsBundleJavaClass)

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Local identifier {#local-identifier}

The identifier '' (after the dot in ) is used within the HTL file to identify the class. **`info`****`data-sly-use.info`**&#x200B;声明后，此标识符的范围在文件中是全局的。 它不限于包含语句的元 `data-sly-use` 素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 获取属性 {#getting-properties}

然后， `info` 该标识符用于访问通过getter方 **`title`** 法和 **`description`** 公开的对象属性 `Info.getTitle` 和 **`Info.getDescription`**。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 输出 {#output}

现在，当我们访问 **`/content/my-example.html`** 它时，它将返回以下HTML:

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

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

在初始化时，参数可以传递到use类。 例如，我们可以做这样的事：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

在此，我们传递一个名为的参数 **`text`**。 use-class然后将我们检索并显示结果的字符串加上大写 `info.upperCaseText`。 以下是调整后的使用类：

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

通过该方法访问该参 `WCMUsePojo` 数

[ `<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

就我们而言，这份声明

`get("text", String.class)`

然后，该字符串被反转并通过该方法公开

**`getReverseText()`**

### 仅从数据密码模板传递参数 {#only-pass-parameters-from-data-sly-template}

虽然上述示例在技术上是正确的，但实际上在HTL代码的执行上下文中存在相关值时（或者，从零开始，该值是静态的，如上所述），从HTL传递值来初始化use类并没有多大意义。

原因是use-class将始终有权访问与HTL代码相同的执行上下文。 这就引出了最佳实践的要点：

>[!NOTE]
>
>仅当在 **data-sly-template** 文件中使用use-class时，才应将参数传递到use-class，而该文件本身是从另一个HTL文件中调用的，并且参数需要传递。

例如，让我们在现有示例的旁边 `data-sly-template` 创建一个单独的文件。 我们将调用新文件 `extra.html`。 它包含一个 **`data-sly-template`** 名为 **`extra`**:

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

The template , takes a single parameter, . **`extra`****`text`**&#x200B;然后，它使用本地名 `ExtraHelper` 称初始化Java use-class, **`extraHelper`** 并将template参数的值传递给它 **`text`** 作为use-class参数 **`text`**。

The body of the template gets the property  (which, under the hood, actually calls ) and displays that value.`extraHelper.reversedText``ExtraHelper.getReversedText()`

We also adapt our existing  to use this new template:**`info.html`**

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info" 
     data-sly-use.extra="extra.html">
    
  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>
    
  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

该文 `info.html` 件现在包含两 **`data-sly-use`** 个语句，其中一个是导入 **`Info`** Java use-class的原始语句，另一个是导入本地名称下的模板文件的新语句 `extra`。

请注意，我们可能已将模板块放在文件中以避 **`info.html`** 免第二个模板块 **`data-sly-use`**，但单独的模板文件更常见、可重用。

The  class is employed as before, calling its getter methods  and  through their corresponding HTL properties  and .**`Info`****`getLowerCaseTitle()`**`getLowerCaseDescription()``info.lowerCaseTitle`**`info.lowerCaseDescription`**

然后，我们对模 **`data-sly-call`** 板执行操作， **`extra`** 并将值作为参 `properties.description` 数传递给它 **`text`**。

Java use-class已更 `Info.java` 改以处理新的text参数：

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

该参 **`text`** 数被检索为， **`get("text", String.class)`**&#x200B;值被反转，并通过getter作为HTL `reversedText` 对象可用 `getReversedText()`。

### 捆绑的Java类 {#bundled-java-class}

With a bundle use-class the class must be compiled, packaged and deployed in AEM using the standard OSGi bundle deployment mechanism. 与本地安装不同，use-class包声明 **应正常命** 名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    ...
}
```

并且，语 `data-sly-use` 句必须引用完 *全限定的类名*，而不只引用本地类名：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### 替代方 `WCMUsePojo` 法 {#alternatives-to-wcmusepojo}

创建Java用类的最常见方式是扩展 `WCMUsePojo`。 但是，还有其他许多选项。 要了解这些变体，了解HTL语句在内 `data-sly-use` 部的工作方式会有所帮助。

假设您有以下语 `data-sly-use` 句：

**`<div data-sly-use.`** `localName`**`="`** `UseClass`**`">`**

系统按如下方式处理该语句：

(1)

* 如果与HTL文件位于 *同一目录中存在本地文件UseClass.java* ，请尝试编译并加载该类。 如果成功，请转至(2)。

* 否则，将 ** UseClass解释为完全限定的类名，并尝试从OSGi环境加载它。 如果成功，请转至(2)。
* 否则，将 *UseClass解释为* HTL或JavaScript文件的路径，并加载该文件。 如果goto成功(4)。

(2)

* 尝试将当前版本调 **`Resource`** 整为“ *`UseClass.`* 如果成功，请转到(3)”。

* 否则，请尝试将当前版本调 **`Request`** 整为 *`UseClass`*。 如果成功，请转到(3)。

* 否则，请尝试使用 *`UseClass`* 零参数构造函数实例化。 If successful, go to (3).

(3)

* 在HTL中，将新调整或创建的对象绑定到名称 *`localName`*。
* 如果 *`UseClass`* 实现 [ ，则调用该方 `io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) 法，传递当前执行上下文(以对象的形式 `init``javax.scripting.Bindings` )。

(4)

* 如果 *`UseClass`* 是包含模板的HTL文件的路径， `data-sly-template`请准备模板。

* 否则， *`UseClass`* 如果是JavaScript use-class的路径，请准备use-class(请参阅 [JavaScript Use-API](use-api-javascript.md))。

有关上述说明的几点要点：

* 任何可从中调整、可从 `Resource`中调整或具有零参数构造 `Request`函数的类都可以是use-class。 类无需扩展甚至 `WCMUsePojo` 实现 `Use`。

* 但是，如果use-class实现 *了* ，则其方法将自动与当前上下文一起调用， `Use`**`init`** 从而允许您将初始化代码放置在依赖于该上下文的位置。

* 扩展的使用类只 `WCMUsePojo` 是实现的一个特殊情况 **`Use`**。 它提供了方便的上下文方法，并 **`activate`** 且其方法从中自动调用 `Use.init`。

### 直接实现接口使用 {#directly-implement-interface-use}

创建use类的最常见方法是扩展， `WCMUsePojo`但也可以直接实现接 `[io.sightly.java.api.Use](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)`口本身。

该接 `Use` 口仅定义一种方法：

`[public void init(javax.script.Bindings bindings)](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))`

在 **`init`****`Bindings`** 初始化类时，将调用该方法，该类的对象包含所有上下文对象以及传递到use-class的任何参数。

必须使用对象显式实现所有附 `WCMUsePojo.getProperties()`加功能(如等效的 ` [javax.script.Bindings](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)` 功能)。 例如：

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

当您希望将现有类的子类用 `Use` 作use-class时，自己实现接口而 `WCMUsePojo` 不是扩展的主要情况是。

### 从资源调整 {#adaptable-from-resource}

另一个选项是使用可适应的助手类 **`org.apache.sling.api.resource.Resource`**。

假设您需要编写一个HTL脚本，该脚本显示DAM资产的mimetype。 在这种情况下，您知道调用HTL脚本时，它将位于包含nodetype的JCR **`Resource`** 的上 **`Node`** 下文中 **`dam:Asset`**。

您知道节点 **`dam:Asset`** 具有如下结构：

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

在此，我们将显示作为示例项目geometrixx的一部分默认安装AEM附带的资产（JPEG图像）。 资产被调用， **`jane_doe.jpg`** 其mimetype被调用 **`image/jpeg`**。

要从HTL中访问资产，可以在语 ` [com.day.cq.dam.api.Asset](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)` 句中声明为类 **`data-sly-use`** :然后使用获取方法 **`Asset`** 检索所需信息。 例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

该语 `data-sly-use` 句指示HTL将当前版本改 **`Resource`** 编为 **`Asset`** 一个并为其指定本地名称 **`asset`**。 然后，它调用 **`getMimeType`** 短格式 `Asset` HTL getter的使用方法： `asset.mimeType`.

### 可适应性自请求 {#adaptable-from-request}

It is also possible to emply as a use-class any class that is adaptable from **` [org.apache.sling.api.SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)`**

与上述适用于自的use类的情况一样， `Resource`可在语句中指定适 [`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 用于自的use类 `data-sly-use` 。 执行时，当前请求将适应给定的类，并使得生成的对象在HTL中可用。
