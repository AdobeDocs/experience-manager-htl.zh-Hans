---
title: HTL 快速入门
description: AEM支持的HTL取代JSP，成为AEM中HTML的首选和推荐的服务器端模板系统。
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '2471'
ht-degree: 0%

---

# HTL 快速入门 {#getting-started-with-htl}

Adobe Experience Manager(AEM)支持的HTML模板语言(HTL)是AEM中HTML的首选和推荐的服务器端模板系统。 它取代了在AEM早期版本中使用的JSP(JavaServer Pages)。

>[!NOTE]
>
>要运行本页上提供的大多数示例，可以使用名为[读取评估打印循环](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl)的实时执行环境。

## JSP上的HTL {#htl-over-jsp}

建议新的AEM项目使用HTML模板语言，因为它与JSP相比具有多种好处。 但是，对于现有项目，只有在估计未来几年的工作量低于维护现有JSP时，迁移才有意义。

但是，迁移到HTL不一定是“一无是处”的选择，因为用HTL编写的组件与用JSP或ESP编写的组件兼容。 这意味着现有项目可以无问题地将HTL用于新组件，同时保留现有组件的JSP。

即使在同一组件中，HTL文件也可以与JSP和ESP一起使用。 以下示例显示在&#x200B;**行1**&#x200B;中如何从JSP文件包含HTL文件，以及在&#x200B;**行2**&#x200B;中如何从HTL文件包含JSP文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常见问题{#frequently-asked-questions}

在开始使用HTML模板语言之前，我们先回答一些与JSP与HTL主题相关的问题。

**HTL有JSP没有的任何限制吗？**  — 与JSP相比，HTL实际上没有任何限制，因为使用JSP可以执行的操作也应该可以通过HTL实现。但是，HTL的设计在几个方面比JSP更加严格，而在单个JSP文件中可以实现的内容，可能需要分为Java类或JavaScript文件，才能在HTL中实现。 但是，通常需要这样做，以确保逻辑和标记之间的关注事项得到很好的分离。

**HTL是否支持JSP标记库？**  — 否，但如加载客户端库 [部分](getting-started.md#loading-client-libraries) 中所示， [template和callstatement](block-statements.md#template-call) 提供了类似模式。

**能否在AEM项目上扩展HTL功能？**  — 不，他们不能。HTL具有强大的扩展机制来重复使用逻辑 — [Use-API](getting-started.md#use-api-for-accessing-logic) — 和标记（[template &amp; call](block-statements.md#template-call)语句），这些逻辑可用于模块化项目代码。

**与JSP相比，HTL的主要优势是什么？**  — 安全性和项目效率是主要优势，详见 [概述](overview.md)。

**JSP最终会消失吗？**  — 在当前日期，没有这些计划。

## HTL {#fundamental-concepts-of-htl}的基本概念

HTML模板语言使用表达式语言将内容段插入到渲染的标记中，而HTML5数据属性用于通过标记块（如条件或小版本）定义语句。 当HTL编译为Java Servlet时，表达式和HTL数据属性都将在服务器端完全计算，并且生成的HTML中不会显示任何内容。

### 块和表达式{#blocks-and-expressions}

以下是第一个示例，它可以像在&#x200B;**`template.html`**&#x200B;文件中一样包含：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以区分两种不同的语法：

* **[块语句](block-statements.md)**  — 根据条件显示  **&lt;h1>** 元素，则 [`data-sly-test`](block-statements.md#test) 会使用HTML5数据属性。HTL提供了多个此类属性，这些属性允许将行为附加到任何HTML元素，且所有属性都带有`data-sly`前缀。

* **[表达式语言](expression-language.md)**  - HTL表达式以字符和 `${` 分 `}`隔。在运行时，将计算这些表达式，并将其值插入传出HTML流中。

以上链接的两个页面提供了可用于语法的详细功能列表。

### SLY元素{#the-sly-element}

HTL的核心概念是提供重用现有HTML元素来定义块语句的可能性，这样就无需插入其他分隔符来定义语句的开始位置和结束位置。 这种用于将静态HTML转换为正常运行的动态模板的标记的不显眼的注释，不会破坏HTML代码的有效性，因此即使是作为静态文件，也能正常显示。

但是，有时可能不存在必须插入块语句的确切位置的现有元素。 对于这种情况，可以插入一个特殊的SLY元素，该元素将自动从输出中删除，同时执行附加的块语句并相应地显示其内容。

以下示例：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

将输出类似于以下HTML的内容，但仅当定义了&#x200B;**`jcr:title`**&#x200B;和&#x200B;**`jcr:description`**&#x200B;属性，且两者均为空时，才会输出：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

要记住的一点是，只有当没有现有元素可以用块语句进行注释时，才使用SLY元素，因为SLY元素会阻止语言提供的值在动态设置时不更改静态HTML。

例如，如果上一个示例已封装在DIV元素内，则添加的SLY元素将是辱骂性的：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

和DIV元素可能已添加条件注释：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL注释{#htl-comments}

以下示例显示在&#x200B;**行1**&#x200B;和HTL注释上，并在&#x200B;**行2**&#x200B;和HTML注释上：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL注释是使用其他类似JavaScript的语法的HTML注释。 整个HTL注释以及中的任何内容将被处理器完全忽略，并从输出中删除。

但是，标准HTML注释的内容将被传递，并且评估注释中的表达式。

HTML注释不能包含HTL注释，反之亦然。

### 特殊上下文{#special-contexts}

为了能够最好地使用HTL，请务必了解基于HTML语法的HTL的后果。

### 元素和属性名称{#element-and-attribute-names}

表达式只能置于HTML文本或属性值中，但不能置于元素名称或属性名称中，否则将不再是有效的HTML。 为了动态设置元素名称，可以对所需元素使用[`data-sly-element`](block-statements.md#element)语句，并动态设置属性名称，即使同时设置多个属性，也可以使用[`data-sly-attribute`](block-statements.md#attribute)语句。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 没有块语句的上下文{#contexts-without-block-statements}

由于HTL使用数据属性来定义块语句，因此无法在以下上下文中定义此类块语句，并且只能在此处使用表达式：

* HTML注释
* 脚本元素
* 样式元素

其原因是这些上下文的内容是文本而不是HTML，并且包含的HTML元素会被视为简单的字符数据。 因此，如果没有真正的HTML元素，也无法执行&#x200B;**`data-sly`**&#x200B;属性。

这听起来可能像一个很大的限制，但是它是必需的，因为不应滥用HTML模板语言来生成非HTML的输出。 下面的[用于访问逻辑的Use-API](getting-started.md#use-api-for-accessing-logic)部分介绍如何从模板中调用其他逻辑，如果需要为这些上下文准备复杂的输出，可以使用此模板。 例如，将数据从后端发送到前端脚本的一种简单方法是，使用组件的逻辑生成JSON字符串，该字符串随后可以使用简单的HTL表达式置于数据属性中。

以下示例说明了HTML注释的行为，但在脚本或样式元素中，会观察到相同的行为：

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

将输出类似于以下HTML的内容：

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### 需要{#explicit-contexts-required}的显式上下文

正如下面[自动上下文感知转义](getting-started.md#automatic-context-aware-escaping)部分中所述，HTL的一个目标是通过自动将上下文感知转义应用于所有表达式，来降低引入跨站点脚本(XSS)漏洞的风险。 虽然HTL可以自动检测置于HTML标记内的表达式的上下文，但它不会分析内联JavaScript或CSS的语法，因此依赖开发人员明确指定必须对此类表达式应用的确切上下文。

由于未应用正确的转义会导致XSS漏洞，因此当上下文未声明时，HTL会删除脚本和样式上下文中所有表达式的输出。

以下是如何为置于脚本和样式中的表达式设置上下文的示例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

有关如何控制转义的更多详细信息，请参阅[表达式语言显示上下文](expression-language.md#display-context)一节。

### 特殊上下文{#lifting-limitations-of-special-contexts}的解除限制

在需要绕过脚本、样式和注释上下文限制的特殊情况下，可以将其内容隔离到单独的HTL文件中。 位于其自身文件中的所有内容都将被HTL解释为正常的HTML片段，而忘记可能包含该片段的限制上下文。

有关示例，请参阅下面的[使用客户端模板](getting-started.md#working-with-client-side-templates)部分。

>[!CAUTION]
>
>此技术可能会引入跨站点脚本(XSS)漏洞，如果使用此漏洞，则应仔细研究安全方面。 与依赖此实践相比，通常有更好的方法来实施相同的内容。

## HTL {#general-capabilities-of-htl}的一般功能

本节将快速介绍HTML模板语言的一般功能。

### 用于访问逻辑{#use-api-for-accessing-logic}的Use-API

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

并在服务器端执行的`logic.js` JavaScript文件旁边放置：

```javascript
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由于HTML模板语言不允许在标记中混合使用代码，因此它提供了Use-API扩展机制，以便轻松执行模板中的代码。

上例使用服务器端执行的JavaScript将标题缩短为10个字符，但它也可以通过提供完全限定的Java类名称来使用Java代码实现这一点。 通常，业务逻辑应该是在Java中构建的，但是当组件需要从Java API提供的内容进行一些特定于视图的更改时，可以很方便地使用一些服务器端执行的JavaScript来执行此操作。

以下各节中详细介绍了这一点：

* [`data-sly-use`语句](block-statements.md#use)中的部分介绍了可以使用该语句执行的所有操作。
* [Use-API页面](use-api.md)提供了一些信息，可帮助在使用Java或JavaScript编写逻辑之间进行选择。
* 要详细了解如何编写逻辑，[JavaScript Use-API](use-api-javascript.md)和[Java Use-API](use-api-java.md)页面应该会有所帮助。

### 自动上下文感知转义{#automatic-context-aware-escaping}

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多数模板语言中，此示例可能会创建跨站点脚本(XSS)漏洞，因为即使所有变量都自动进行HTML转义，`href`属性仍必须具体进行URL转义。 这种遗漏是最常见的错误之一，因为它很容易被遗忘，而且很难自动发现。

为帮助实现此目的，HTML模板语言会根据放置该变量的上下文自动对每个变量进行转义。 这要归功于HTL了解HTML的语法。

假定在`logic.js`文件之后：

```javascript
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

初始示例随后将产生以下输出：

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

请注意这两个属性是如何以不同方式转义的，因为HTL知道必须对URI上下文的`href`和`src`属性进行转义。 此外，如果URI以&#x200B;**`javascript:`**&#x200B;开头，则属性将被完全删除，除非上下文被明确更改为其他内容。

有关如何控制转义的更多详细信息，请参阅[表达式语言显示上下文](expression-language.md#display-context)一节。

### 自动删除空属性{#automatic-removal-of-empty-attributes}

请考虑以下示例：

```xml
<p class="${properties.class}">some text</p>
```

如果`class`属性的值碰巧为空，则HTML模板语言将自动从输出中删除整个`class`属性。

同样，这是可能的，因为HTL了解HTML语法，因此，仅当属性的值不为空时，才能有条件地显示具有动态值的属性。 这非常方便，因为它避免在属性周围添加条件块，这会导致标记无效且不可读。

此外，置于表达式中的变量类型也很重要：

* **String:**
   * **不为空：** 将字符串设置为属性值。
   * **empty:** 完全删除属性。

* **数字：** 将值设置为属性值。

* **布尔型:**
   * **true:** 显示不带值的属性（作为布尔HTML属性）
   * **false:** 完全删除属性。

以下是布尔表达式如何允许控制布尔HTML属性的示例：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

对于设置属性，[`data-sly-attribute`](block-statements.md#attribute)语句可能也很有用。

## HTL {#common-patterns-with-htl}的常见模式

本节介绍一些常见情景以及如何使用HTML模板语言最好地解决它们。

### 正在加载客户端库{#loading-client-libraries}

在HTL中，客户端库通过AEM提供的帮助程序模板加载，该模板可通过[`data-sly-use`](block-statements.md#use)访问。 此文件中提供了三个模板，可通过[`data-sly-call`](block-statements.md#template-call)调用：

* **`css`**  — 仅加载引用的客户端库的CSS文件。
* **`js`**  — 仅加载引用的客户端库的JavaScript文件。
* **`all`**  — 加载引用的客户端库的所有文件（CSS和JavaScript）。

每个帮助程序模板都需要一个&#x200B;**`categories`**&#x200B;选项来引用所需的客户端库。 该选项可以是字符串值数组，也可以是包含逗号分隔值列表的字符串。

以下是两个简短的示例：

### 一次完全加载多个客户端库{#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 在页面{#referencing-a-client-library-in-different-sections-of-a-page}的不同部分中引用客户端库

```xml
<!doctype html>
<html data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html">
    <head>
        <!-- HTML meta-data -->
        <sly data-sly-call="${clientlib.css @ categories='myCategory'}"/>
    </head>
    <body>
        <!-- page content -->
        <sly data-sly-call="${clientlib.js @ categories='myCategory'}"/>
    </body>
</html>
```

在上面的第二个示例中，如果HTML **`head`**&#x200B;和&#x200B;**`body`**&#x200B;元素被放置到不同的文件中，则必须在每个需要它的文件中加载&#x200B;**`clientlib.html`**&#x200B;模板。

[template &amp; call](block-statements.md#template-call)语句中的部分提供了有关声明和调用此类模板如何工作的更多详细信息。

### 将数据传递到客户端{#passing-data-to-the-client}

通常而言，将数据传递到客户端的最佳、最优雅的方法是使用数据属性，但对于HTL而言，这种方法更为有效。

以下示例显示如何使用逻辑（也可以使用Java编写）非常方便地将要传递到客户端的对象序列化为JSON，然后，可以非常轻松地将该对象放入数据属性中：

```xml
<!--/* template.html file: */-->
<div data-sly-use.logic="logic.js" data-json="${logic.json}">...</div>
```

```javascript
/* logic.js file: */
use(function () {
    var myData = {
        str: "foo",
        arr: [1, 2, 3]
    };

    return {
        json: JSON.stringify(myData)
    };
});
```

从此处，您可以轻松想象客户端JavaScript如何访问该属性并再次解析JSON。 例如，这将是放置到客户端库中的相应JavaScript:

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用客户端模板{#working-with-client-side-templates}

在[Lipting Limitation of Special Contexts](getting-started.md#lifting-limitations-of-special-contexts)部分中介绍的技术可以合法使用的一个特殊情况是，编写位于&#x200B;**script**&#x200B;元素中的客户端模板（例如Handlebars）。 这种情况下可以安全地使用此技术的原因是，**script**&#x200B;元素随后不按假设包含JavaScript，而包含其他HTML元素。 以下是如何实现此操作的示例：

```xml
<!--/* template.html file: */-->
<script id="entry-section" type="text/template"
    data-sly-include="entry-section.html"></script>

<!--/* entry-section.html file: */-->
<div class="entry-section">
    <h2 data-sly-test="${properties.entrySectionTitle}">
        ${properties.entrySectionTitle}
    </h2>
    {{#each entry}}<h3><a href="{{link}}">{{title}}</a></h3>{{/each}}
</div>
```

如上所示，将包含在&#x200B;**`script`**&#x200B;元素中的标记可以包含HTL块语句，并且表达式不需要提供明确的上下文，因为Handlebars模板的内容已隔离在其自身的文件中。 此外，此示例还显示了如何将服务器端执行的HTL（如&#x200B;**`h2`**&#x200B;元素中的HTL）与客户端执行的模板语言(如Handlebars（如&#x200B;**`h3`**&#x200B;元素中所示）混合使用。

但是，更现代的技术将改为使用HTML **`template`**&#x200B;元素，因为这样可以带来好处，即无需将模板的内容分隔为单独的文件。

**阅读下一篇文章：**

* [表达式语言](expression-language.md)  — 详细了解在HTL表达式中可以执行的操作。
* [块语句](block-statements.md)  — 发现HTL中可用的所有块语句，以及如何使用它们。
