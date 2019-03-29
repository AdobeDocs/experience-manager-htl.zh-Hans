---
title: HTL Java使用API
seo-title: HTL Java使用API
description: HTML模板语言- HTL- Java使用API使HTL文件能够访问自定义Java类中的辅助方法方法。
seo-description: HTML模板语言- HTL- Java使用API使HTL文件能够访问自定义Java类中的辅助方法方法。
uuid: b340f8f7-a193-45c8-aa39-5c6 e2 c0194 ea
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 126eBC9d-5f7b-47a4-aea2-c8840 d3464 c
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 796c55d3d85e6b5a3efaa5c04a25be1b0b4e54dd

---


# HTL Java使用API{#htl-java-use-api}

HTML模板语言(HTL) Java使用API使HTL文件能够访问自定义Java类中的辅助方法方法。这使所有复杂的业务逻辑都封装在Java代码中，而HTL代码只能进行直接标记制作。

## 简单示例 {#a-simple-example}

我们将从没有使用类的 *HTL* 组件开始。它由一个文件组成， `/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我们还为此组件添加一些内容，以在以下位置呈现 **`/content/my-example/`**：

### `http://localhost:4502/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

访问此内容时，将执行HTL文件。在HTL代码中，我们使用上下文对象****`properties`访问当前资源并 `title``description` 显示它们。输出HTML将为：

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加使用类 {#adding-a-use-class}

所站立 **的信息** 组件不需要使用类执行其(非常简单)函数。但是，有时您需要完成无法在HTL中完成的操作，因此需要使用类。但请记住：

>[!NOTE]
>
>*仅当不能在HTL中完成某些操作时使用类类。*

例如，假定您希望 `info` 组件显示资源的属性 `title` 和 **`description`** 属性，但全都以小写形式显示。由于HTL没有小写字符串的方法，因此您需要使用类。我们可以通过添加Java use-class并更改如下 **`info.html`** 所示：

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

在下面几节中，我们将遍历代码的不同部分。

### 本地与捆绑Java类 {#local-vs-bundle-java-class}

Java使用类可以通过以下两种方式安装： **本地** 或 **打包**。*此示例使用本地安装。*

在本地安装中，Java源文件位于同一个资源库文件夹中的HTL文件旁边。源会自动编译为按需编译。无需单独的编译或打包步骤。

在捆绑安装中，必须使用标准AEM捆绑部署机制在OSGi包内编译和部署Java类(请参阅 [绑定Java类](#bundled-java-class))。

>[!NOTE]
>
>在使用类特定于相关组件时，建议 **使用本地Java使用类** 。
>
>当Java代码实现从多个HTL组件访问的服务时，建议 **捆绑Java使用类** 。

### Java包是存储库路径 {#java-package-is-repository-path}

使用本地安装时，use-class的包名称必须与存储库文件夹位置的名称相匹配，路径中的任何连字符都替换为包名称中的下划线。

在此例 **`Info.java`** 中 **`/apps/my-example/components/info`** ，该包 **`apps.my_example.components.info`** 位于：

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
>在AEM开发过程中，建议使用存储库项目名称中的连字符。但是，连字符在Java包名称中是非法的。因此，存储库路径中 **的所有连字符必须转换为包名称**中的下划线。

### 扩展 `WCMUsePojo`{#extending-wcmusepojo}

尽管有许多方法将Java类与HTL结合(请参阅替代)，但最 `WCMUsePojo`简单的方法是扩展类 `WCMUsePojo` ：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    
    ...
}
```

### 初始化类 {#initializing-the-class}

当use-class从extended中进行扩展 **`WCMUsePojo`**时，将通过覆盖 **`activate`** 方法执行初始化操作：

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

通常， [激活](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html) 方法用于根据当前上下文(例如，当前的请求和资源)预计算和存储HTL代码中所需的值(成员变量)。

`WCMUsePojo` 类提供对同一组上下文对象的访问权限，如HTL文件中可用(请参阅 [全局对象](global-objects.md))。

在扩展 **`WCMUsePojo`**的类中，上下文对象可以 ** 通过

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

或者，可以通过相应 **的便捷方法直接访问常用上下文对象**：

|  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [页面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [valueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [valueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [设计人员](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [设计](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [样式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [组件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [valueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse#getInheritedProperties.html()) |
| [资源](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [resourceSolver](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [slinghttpservletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlinghttpservletResponse](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter方法 {#getter-methods}

使用类初始化后，将运行HTL文件。在此阶段HTL通常会拖入使用类的各种成员变量的状态并呈现它们以供演示。

要从HTL文件中访问这些值，您必须根据以下命名约定在use-类 **中定义custom getter方法**：

* 表单 **`getXyz`** 的方法将在HTL文件中公开名为 ***xyz***的对象属性。

例如，在以下示例中，这些方法 **`getTitle`** 在 **`getDescription`** 对象属性 **`title`** 中并 **`description`** 在HTL文件的上下文中变为可访问：

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

**`data-sly-use`** 该属性用于初始化HTL代码中的use-class。在我们的示例中，该 `data-sly-use` 属性声明我们要使用该类 **`Info`**。我们只能使用类的本地名称，因为我们使用本地安装(放置Java源文件所在的文件夹与HTL文件位于同一文件夹中)。如果我们使用的是捆绑安装，我们将必须指定完全限定的类名称(请参阅 [使用类绑定安装](#LocalvsBundleJavaClass))。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本地标识符 {#local-identifier}

标识符“**`info`**(位于点之后) **`data-sly-use.info`**用于HTL文件，用于标识类。在文件被声明后，此标识符的范围在文件中是全局的。它并不仅仅限于包含语句 `data-sly-use` 的元素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 获取属性 {#getting-properties}

标识符 `info` 随后用于访问对象属性 **`title`** ，并 **`description`** 使用getter方法 `Info.getTitle` 和 **`Info.getDescription`**getter方法。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 输出 {#output}

现在，当我们访问 **`/content/my-example.html`** 它时，将返回以下HTML：

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 超越基础 {#beyond-the-basics}

在本节中，我们将进一步介绍以上简单示例之外的其他功能：

* 将参数传递给use-class。
* 绑定Java使用类。
* 替代内容 `WCMUsePojo`

### 传递参数 {#passing-parameters}

初始化时可以将参数传递给use-类。例如，我们可以如下所示：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

在这里，我们将传递一个调用 **`text`**的参数。使用类之后，我们将更新检索并显示结果的字符串 `info.upperCaseText`。以下是调整后的使用类：

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

该参数通过该 `WCMUsePojo` 方法访问

[ `<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

在我们的案例中，声明

`get("text", String.class)`

随后，该字符串会通过该方法进行颠倒和公开

**`getReverseText()`**

### 只通过数据-sly-template传递参数 {#only-pass-parameters-from-data-sly-template}

尽管上面的示例从技术上来讲正确，但从HTL传递值到初始化use-类没有太大意义，当该值中的值可在HTL代码的执行上下文中可用时(或者，如果该值是静态的，则上面的值为static)。

其原因在于，使用类始终能够访问与HTL代码相同的执行上下文。这会带来最佳实践的导入点：

>[!NOTE]
>
>只应在使用类的data-sly-template文件中使用使用类时将参数传递给使用类， **该** 文件本身是通过需要传递的参数来自另一个HTL文件。

例如，我们在现有示例旁边创建一 `data-sly-template` 个单独的文件。我们将调用新文件 `extra.html`。它包含一个 **`data-sly-template`** 名为 **`extra`**：

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

模板 **`extra`**将采用单个参数 **`text`**。然后，它用本地名称初始化Java use-class `ExtraHelper`**`extraHelper`** ，并将模板参数 **`text`** 的值传递给use-class参数 **`text`**。

模板的正文将获取属性 `extraHelper.reversedText` (在实际调用下，实际调用 `ExtraHelper.getReversedText()`)并显示该值。

我们还可以调整现有 **`info.html`** 模板，以使用新模板：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info" 
     data-sly-use.extra="extra.html">
    
  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>
    
  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

该文件 `info.html` 现在包含两 **`data-sly-use`** 个语句，前者作为导入 **`Info`** Java使用类的原始语句，另一个用于导入在本地名称下导入模板文件的新语句 `extra`。

请注意，我们可能已将模板块放在 **`info.html`** 文件内以避免第二 **`data-sly-use`**个，但一个单独的模板文件更常见，而且更可重复使用。

**`Info`** 该类像之前一样使用，调用它的getter方法 **`getLowerCaseTitle()`** 以及 `getLowerCaseDescription()` 其相应的HTL属性 `info.lowerCaseTitle` 和 **`info.lowerCaseDescription`**。

然后，我们对 **`data-sly-call`** 模板 **`extra`** 执行一个操作，并将其值 `properties.description` 传递给该参数 **`text`**。

Java use-class `Info.java` 经过更改以处理新的文本参数：

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

检索 **`text`** 该参数，该值 **`get("text", String.class)`**被颠倒，并通过getter作为HTL对象 `reversedText` 提供 `getReversedText()`。

### 绑定Java类 {#bundled-java-class}

借助捆绑类类，必须使用标准OSGi捆绑部署机制在AEM中编译、打包和部署类。与本地安装相比，使用类 **包声明** 应正常命名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    ...
}
```

而且 `data-sly-use` ，语句必须引用 *完全限定的类名*，而不仅仅是本地类名称：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### 替代内容 `WCMUsePojo`{#alternatives-to-wcmusepojo}

创建Java使用类最常见的方法是扩展 `WCMUsePojo`。但是，还有许多其他选项。为了理解这些变体，理解HTL `data-sly-use` 语句在后台的工作方式有助于理解。

假定您有以下 `data-sly-use` 语句：

**`<div data-sly-use.`** `localName`**`="`** `UseClass`**`">`**

系统按如下方式处理语句：

(1)

* 如果在与HTL文件所在目录中存在本地文件 *useClass. java，* 请尝试编译和加载该类。如果成功转至(2)。

* 否则，将 *useClass解释* 为完全限定的类名，并尝试从OSGi环境加载它。如果成功转至(2)。
* 否则，将 *UseClass解释* 为HTL或JavaScript文件的路径，并加载该文件。如果成功，则为(4)。

(2)

* 尝试调整当前 **`Resource`** 版本 *`UseClass.`* 如果成功，请转至(3)。

* Otherwise, try to adapt the current **`Request`** to *`UseClass`*. 如果成功，请转至(3)。

* 否则，尝试 *`UseClass`* 使用零参数构造函数进行实例化。如果成功，请转至(3)。

(3)

* 在HTL中，将新改编或创建的对象绑定到名称 *`localName`*。
* 如果 *`UseClass`* 实现，则 [`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) 调用 `init` 方法，以传递当前执行上下文(以 `javax.scripting.Bindings` 对象的形式)。

(4)

* 如果 *`UseClass`* 是指向包含a的 `data-sly-template`HTL文件的路径，则准备该模板。

* 否则，如果 *`UseClass`* 是JavaScript使用类的路径，请准备use-class(请参阅 [JavaScript使用API](use-api-javascript.md))。

关于上述说明的几点要点：

* 可自适应自 `Resource`适应的、自适应的 `Request`或具有零参数构造函数的类可以是use-类。类不必扩展扩展 `WCMUsePojo` 甚至实现 `Use`。

* 但是，如果使用类 *确实* 实现， `Use`则将使用当前上下文自动调用其 **`init`** 方法，允许您放置依赖于该上下文的初始化代码。

* 扩展的 `WCMUsePojo` 使用类只是一个特例 **`Use`**。它提供方便的上下文方法及其 **`activate`** 方法 `Use.init`。

### 直接实施界面使用 {#directly-implement-interface-use}

尽管创建使用类最常见的方法是扩展 `WCMUsePojo`，但也可以直接实现 `[io.sightly.java.api.Use](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)`界面本身。

`Use` 该界面只定义一种方法：

`[public void init(javax.script.Bindings bindings)](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))`

将在类初始化时调用该 **`init`** 方法， **`Bindings`** 该对象包含所有上下文对象以及传入use-类的任何参数。

所有其他功能(如等效于 `WCMUsePojo.getProperties()`)必须使用 ` [javax.script.Bindings](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)` 对象显式隐式隐含。例如：

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

实现 `Use` 接口而不是扩展 `WCMUsePojo` 的主要案例是当您希望使用现有类的子类作为use-类时。

### 自适应资源 {#adaptable-from-resource}

另一种选择是使用可自适应的辅助类 **`org.apache.sling.api.resource.Resource`**。

假设您需要编写一个HTL脚本，它显示DAM资产的MIME类型。在这种情况下，您知道在调用HTL脚本时，它将位于一个 **`Resource`** 包含nodeType的JCR **`Node`** 的环境中 **`dam:Asset`**。

您知道 **`dam:Asset`** 节点的结构如下：

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

在这里，我们将显示默认安装AEM作为示例项目geometrixx示例的资产(JPEG图像)。资产被调用 **`jane_doe.jpg`** ，其mimetype为 **`image/jpeg`**。

要从HTL中访问该资源，您可以在语句中声明 ` [com.day.cq.dam.api.Asset](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)` 为 **`data-sly-use`** 该类：然后使用一种获取 **`Asset`** 所需信息的方法。例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

`data-sly-use` 语句将HTL定向到HTL以使其 **`Resource`** 适应并 **`Asset`** 为其提供本地名称 **`asset`**。然后，调用 **`getMimeType`**`Asset` 使用HTL getter速记的方法： `asset.mimeType`。

### 根据请求改编 {#adaptable-from-request}

也可以将任何符合 **` [org.apache.sling.api.SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)`**

与上面的use-class适应性一样， `Resource`可以在语句中指定可 [`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 自适应的use-类 `data-sly-use` 。执行后，当前请求将调整为给定的类，并将在HTL中提供生成的对象。
