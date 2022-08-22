---
title: HTL Java Use-API
description: HTL Java Use-API允许HTL文件访问自定义Java类中的Helper方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: da2375a2390f0254dba9745d9f4970e88788e5d5
workflow-type: tm+mt
source-wordcount: '1518'
ht-degree: 39%

---


# HTL Java Use-API {#htl-java-use-api}

HTL Java Use-API允许HTL文件访问自定义Java类中的Helper方法。

## 用例 {#use-case}

HTL Java Use-API允许HTL文件通过访问自定义Java类中的Helper方法 `data-sly-use`. 这样可以将所有复杂的业务逻辑封装在 Java 代码中，而 HTL 代码只处理直接标记生成。

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

此示例说明了Use-API的用法。

>[!NOTE]
>
>为了简化此示例，以便简单说明其用法。 在生产环境中，建议使用 [Sling模型。](https://sling.apache.org/documentation/bundles/models.html)

我们首先从一个名为 `info`，则没有use类。 它由单个文件 `/apps/my-example/components/info.html` 组成

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我们还为该组件添加了一些内容，以便在 `/content/my-example/` 上呈现：

```xml
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

当用户访问此内容时，系统会执行 HTL 文件。在HTL代码中，我们使用上下文对象 `properties` 访问当前资源的 `title` 和 `description` 并显示它们。 输出文件 `/content/my-example.html` 将为：

```html
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加一个 use 类 {#adding-a-use-class}

的 `info` 组件本身就不需要use类即可执行其简单功能。 有些情况下，您需要做的事情无法在 HTL 中完成，因此需要 use 类。但请记住：

>[!NOTE]
>
>只有当某些事情无法单独在 HTL 中完成时，才应使用 use 类。

例如，假设您想让 `info` 组件显示资源的 `title` 和 `description` 属性且必须全部小写。由于 HTL 没有将字符串转换为小写的方法，所以您需要 use 类。我们可以通过添加Java用例类并更改 `/apps/my-example/component/info/info.html` 如下所示：

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

此外，我们还创建 `/apps/my-example/component/info/Info.java`.

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

请参阅 [的Javaoc `com.adobe.cq.sightly.WCMUsePojo`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) 以了解更多详细信息。

现在，让我们浏览代码的不同部分。

### 局部与捆绑 Java 类 {#local-vs-bundle-java-class}

Java用例类可通过两种方式安装：

* **本地**  — 在本地安装中，Java源文件与HTL文件一起放置在同一存储库文件夹中。 根据需要自动编译源。无需单独的编译或打包步骤。
* **捆绑**  — 在包安装中，必须使用标准AEM包部署机制编译Java类并将其部署在OSGi包中（请参阅部分） [捆绑的Java类](#bundled-java-class))。

要了解何时使用哪种方法，请牢记以下两点：

* 当 use 类特定于所针对的组件时，建议使用&#x200B;**局部 Java use 类**。
* 当 Java 代码实现从多个 HTL 组件访问的服务时，建议使用&#x200B;**捆绑 Java use 类**。

本例使用局部安装。

### Java包是存储库路径 {#java-package-is-repository-path}

使用局部安装时，use 类的包名称必须与存储库文件夹位置的名称匹配，在包名称中将路径中的所有连字符替换为下划线。

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

虽然有许多将Java类与HTL结合的方法(请参阅 [替代 `WCMUsePojo`](#alternatives-to-wcmusepojo))，最简单的方法是 `WCMUsePojo` 类。 例如 `/apps/my-example/component/info/Info.java`:

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化类 {#initializing-the-class}

从 `WCMUsePojo`，则通过覆盖 `activate` 方法，在本例中为 `/apps/my-example/component/info/Info.java`

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

的 `WCMUsePojo` 类提供对与HTL文件中可用的上下文对象集的访问权限（请参阅文档） [全局对象。](global-objects.md))

在扩展 `WCMUsePojo` 的类中，可以使用名称访问对象

[`<T> T get(String name, Class<T> type)`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，常用的上下文对象可以直接使用此表中所列的相应方便方法进行访问。

| Object | 方便方法 |
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

### Getter方法 {#getter-methods}

一旦 use 类初始化，HTL 文件即会运行。在此阶段，HTL 通常会拉入 use 类的各种成员变量的状态，并渲染它们以便呈现。

要在HTL文件中提供对这些值的访问，您必须根据以下命名约定在use-class中定义自定义getter方法：

* 表单 `getXyz` 的一种方法会在 HTL 文件内公开一个名为 `xyz` 的对象属性。

在以下示例文件中 `/apps/my-example/component/info/Info.java`，方法 `getTitle` 和 `getDescription` 导致对象属性 `title` 和 `description` 在HTL文件的上下文中变得可访问。

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

`data-sly-use` 属性用于初始化 HTL 代码中的 use 类。在我们的示例中，`data-sly-use` 属性声明我们需要使用类 `Info`：我们可以只使用该类的局部名称，因为我们使用的是局部安装（Java 源文件与 HTL 文件放在同一文件夹中）。如果我们使用捆绑安装，则须指定完全限定的类名称。

请注意 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本地标识符 {#local-identifier}

标识符 `info`（位于 `data-sly-use.info` 中的点后）用于在 HTL 文件中标识类。声明该标识符后，其作用范围是整个文件。这不限于包含 `data-sly-use` 语句的元素。

请注意 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 获取资产 {#getting-properties}

标识符 `info` 然后用于访问通过 getter 方法 `Info.getTitle` 和 `Info.getDescription` 公开的对象属性 `title` 和 `description`。

请注意 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 输出 {#output}

现在，当我们 `/content/my-example.html` 它将返回以下 `/content/my-example.html` 文件。

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

>[!NOTE]
>
>为了简化此示例，以便简单说明其用法。 在生产环境中，建议使用 [Sling模型。](https://sling.apache.org/documentation/bundles/models.html)

## 超越基础知识 {#beyond-the-basics}

本节将介绍一些除前面介绍的简单示例之外的其他功能。

* 将参数传递到use类
* 捆绑的Java用类

### 传递参数 {#passing-parameters}

初始化时，可将参数传递给 use 类。例如，我们可以这样做：

有关详细信息，请参阅Sling [HTL脚本引擎文档。](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#passing-parameters-to-java-use-objects)

### 捆绑的 Java 类 {#bundled-java-class}

使用捆绑的使用类，必须使用标准OSGi包部署机制在AEM中编译、打包和部署类。 与本地安装不同， use-class包声明的名称应通常与此中的声明相同 `/apps/my-example/component/info/Info.java` 示例。

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

和 `data-sly-use` 语句必须引用完全限定的类名，而不是仅引用此中的本地类名 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```
