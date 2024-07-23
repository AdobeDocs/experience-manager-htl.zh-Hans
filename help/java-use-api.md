---
title: HTL Java Use-API
description: HTL Java Use-API 让 HTL 文件可以访问自定义 Java 类中的 Helper 方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '1137'
ht-degree: 66%

---


# HTL Java Use-API {#htl-java-use-api}

HTL Java Use-API 让 HTL 文件可以访问自定义 Java 类中的 Helper 方法。

## 用例 {#use-case}

HTL Java Use-API 让 HTL 文件可以通过 `data-sly-use` 访问自定义 Java 类中的 Helper 方法。 此功能允许将所有复杂的业务逻辑封装在Java代码中，而HTL代码只处理直接标记生成。

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

此示例说明了 Use-API 的用法。

>[!NOTE]
>
>此示例经过了简化，仅为了说明其用途。 在生产环境中，Adobe建议使用[Sling模型](https://sling.apache.org/documentation/bundles/models.html)。

从名为`info`且没有use类的HTL组件开始。 它由单个文件 `/apps/my-example/components/info.html` 组成

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

现在，为该组件添加一些内容以在`/content/my-example/`上呈现：

```xml
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

访问此内容时，将运行HTL文件。 在HTL代码内，上下文对象`properties`用于访问当前资源的`title`和`description`并显示它们。 输出文件`/content/my-example.html`为：

```html
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加一个 use 类 {#adding-a-use-class}

目前 `info` 组件不需要 use 类来执行其（非常简单的）功能。 有些情况下，您需要做的事情无法在 HTL 中完成，因此需要 use 类。但请记住：

>[!NOTE]
>
>只有当某些事情无法单独在 HTL 中完成时，才应使用 use 类。

例如，假设您想让 `info` 组件显示资源的 `title` 和 `description` 属性且必须全部小写。由于HTL没有将字符串转换为小写的方法，因此您需要use类，方法是添加Java use类并更改`/apps/my-example/component/info/info.html`，如下所示：

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

此外，还创建`/apps/my-example/component/info/Info.java`。

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

有关详细信息，请参阅`com.adobe.cq.sightly.WCMUsePojo`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)的[Java文档。

现在，让我们浏览代码的不同部分。

### 局部与捆绑 Java 类 {#local-vs-bundle-java-class}

Java use 类有以下两种安装方式：

* **局部** – 在局部安装中，Java 源文件与 HTL 文件放在一起，位于同一个存储库文件夹下。 根据需要自动编译源。无需单独的编译或打包步骤。
* **捆绑** – 在捆绑安装中，必须使用标准 AEM 捆绑部署机制在 OSGi 捆绑包中编译和部署 Java 类（参阅[捆绑的 Java 类](#bundled-java-class)部分）。

要知道何时使用哪种方法，请记住以下两点：

* 当 use 类特定于所针对的组件时，建议使用&#x200B;**局部 Java use 类**。
* 当 Java 代码实现从多个 HTL 组件访问的服务时，建议使用&#x200B;**捆绑 Java use 类**。

本例使用局部安装。

### Java 包是存储库路径 {#java-package-is-repository-path}

使用本地安装时，use类的包名称必须与存储库文件夹位置匹配。 将路径中的任何连字符替换为包名称中的下划线。

在此情形中，`Info.java` 位于 `/apps/my-example/components/info`，因此包名称为 `apps.my_example.components.info`：

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

尽管 Java 类与 HTL 结合的方法很多，但最简单的方法是扩展 `WCMUsePojo` 类。对于此示例`/apps/my-example/component/info/Info.java`：

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化类 {#initializing-the-class}

从 `WCMUsePojo` 扩展 use 类时，通过覆盖 `activate` 方法来执行初始化，在本例中为 `/apps/my-example/component/info/Info.java`

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

通常，[activate](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) 方法用于根据当前上下文（例如当前请求和资源）预先计算和存储（在成员变量中）HTL 代码中所需的值。

`WCMUsePojo`类提供对HTL文件中同样的上下文对象集的访问（请参阅文档[全局对象](global-objects.md)）。

在扩展`WCMUsePojo`的类中，可以使用上下文对象的名称访问这些对象：

[`<T> T get(String name, Class<T> type)`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，您可以使用本表中列出的适当方便方法直接访问常用的上下文对象。

| 对象 | 方便方法 |
|---|---|
| [`PageManager`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/PageManager.html) | [`getPageManager()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getPageManager--) |
| [`Page`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/Page.html) | [`getCurrentPage()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentPage--) |
| [`Page`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/Page.html) | [`getResourcePage()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResourcePage--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getPageProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getPageProperties--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getProperties--) |
| [`Designer`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [`getDesigner()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getDesigner--) |
| [`Design`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Design.html) | [`getCurrentDesign()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentDesign--) |
| [`Style`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Style.html) | [`getCurrentStyle()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentStyle--) |
| [`Component`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/components/Component.html) | [`getComponent()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getComponent--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getInheritedProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getInheritedPageProperties--) |
| [`Resource`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/Resource.html) | [`getResource()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResource--) |
| [`ResourceResolver`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [`getResourceResolver()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResourceResolver--) |
| [`SlingHttpServletRequest`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [`getRequest()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getRequest--) |
| [`SlingHttpServletResponse`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [`getResponse()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResponse--) |
| [`SlingScriptHelper`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [`getSlingScriptHelper()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getSlingScriptHelper--) |

### Getter 方法 {#getter-methods}

use类初始化后，运行HTL文件。 在此阶段，HTL通常会拉入use类的各种成员变量的状态，并呈现它们以供呈现。

要提供从 HTL 文件内访问这些值的权限，您必须根据以下命名惯例在 use 类中定义自定义 getter 方法：

* 形式为`getXyz`的方法在HTL文件中公开名为`xyz`的对象属性。

在以下示例文件`/apps/my-example/component/info/Info.java`中，方法 `getTitle` 和 `getDescription` 使对象属性 `title` 和 `description` 在 HTL 文件上下文内变为可访问。

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

### `data-sly-use`属性 {#data-sly-use-attribute}

`data-sly-use` 属性用于初始化 HTL 代码中的 use 类。在此示例中，`data-sly-use`属性声明使用了类`Info`。 在这种情况下，您可以仅使用类的局部名称，因为您使用的是局部安装（已将Java源文件放在与HTL文件相同的文件夹中）。 如果您使用捆绑安装，则必须指定完全限定的类名称。

注意本 `/apps/my-example/component/info/info.html` 示例中的用法。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 局部标识符 {#local-identifier}

标识符 `info`（位于 `data-sly-use.info` 中的点后）用于在 HTL 文件中标识类。声明该标识符后，其作用范围是整个文件。这不限于包含 `data-sly-use` 语句的元素。

注意本 `/apps/my-example/component/info/info.html` 示例中的用法。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 获取属性 {#getting-properties}

标识符 `info` 然后用于访问通过 getter 方法 `Info.getTitle` 和 `Info.getDescription` 公开的对象属性 `title` 和 `description`。

注意本 `/apps/my-example/component/info/info.html` 示例中的用法。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 输出 {#output}

现在，当访问`/content/my-example.html`时，它会返回以下`/content/my-example.html`文件。

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

>[!NOTE]
>
>本示例经过简化以说明其用途。 在生产环境中，Adobe建议您使用[Sling模型](https://sling.apache.org/documentation/bundles/models.html)。

## 基本功能拓展 {#beyond-the-basics}

本节介绍一些超出前述简单示例的其他功能。

* 将参数传递给 use 类
* 捆绑的 Java use 类

### 传递参数 {#passing-parameters}

初始化时，可将参数传递给 use 类。

有关详细信息，请参阅Sling [HTL脚本引擎文档](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#passing-parameters-to-java-use-objects)。

### 捆绑的 Java 类 {#bundled-java-class}

对于捆绑的 use 类，必须使用标准 OSGi 捆绑包部署机制在 AEM 中编译、打包和部署类。 与局部安装相反，use 类包声明应按照该 `/apps/my-example/component/info/Info.java` 示例中的方式正常命名。

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

同时，`data-sly-use` 语句必须引用完全限定的类名称，而非该 `/apps/my-example/component/info/info.html` 示例中仅引用局部类名称：

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```
