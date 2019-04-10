---
title: HTL入门
seo-title: HTL入门
description: AEM支持的HTL代替JSP作为AEM中HTML首选和建议的服务器端模板系统。
seo-description: HTML模板语言- HTL-由Adobe Experience Manager支持，取代JSP作为AEM中的HTML首选服务器端模板系统。
uuid: 4a7d6748-8cdf-4280-a85 d-6c5319 ab487
content-type: 引用
topic-tags: introduction
discoiquuid: 3bf2ca75-0d68-489d-bd1 c-1d4 fd730 c61 a
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 271c355ae56e16e309853b02b8ef09f2ff971a2e

---


# HTL入门 {#getting-started-with-htl}

Adobe Experience Manager(AEM)支持的HTML模板语言(HTL)代替JSP(JavaServer页面)作为AEM中的HTML首选服务器端模板系统。

>[!NOTE]
>
>要运行此页面上提供的大多数示例，可使用名为 [Read Eval Print循环](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl) 的实时执行环境。
>
>AEM Community已生成一系列 [有关使用HTL的文章、视频和网络研讨会](related-community-articles.md) 。

## HTL over JSP {#htl-over-jsp}

建议新AEM项目使用HTML模板语言，因为与JSP相比，它具有多种优势。但是，对于现有项目，迁移只有在预计的工作比现有的JSP工作更省力的情况下才有意义。

但移动到HTL并不一定是一个全或全部选择，因为使用HTL编写的组件与使用JSP或ESP编写的组件兼容。这意味着现有项目不会对新组件使用HTL，同时保留现有组件的JSP。

即使在同一组件内，也可以在JSP和ESP旁边使用HTL文件。以下示例显示了如何在 **** 第行中包含JSP文件中的HTL文件，第行在 **第行** 中介绍了如何从HTL文件中包括JSP文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常见问题{#frequently-asked-questions}

在开始使用HTML模板语言之前，我们首先回答与JSP与HTL主题相关的一些问题。

**HTL是否有任何JSP不存在的限制？**与JSP相比，HTL真的没有限制，因为在HTSP中，使用HTL可以实现哪些操作。然而，HTL是由几个方面的设计更严格，而在单个JSP文件中可以实现的内容，可能需要划分为Java类或JavaScript文件才能在HTL中实现。但通常需要确保逻辑与标记之间有良好的分离。

**HTL是否支持JSP标记库？**不，但 [如“加载客户端库](getting-started.md#loading-client-libraries) ”部分中所示， [模板和调用](block-statements.md#template-call) 语句提供类似模式。

**HTL功能是否可以在AEM项目上进行扩展？****否，但如 [“加载客户端库](getting-started.md#loading-client-libraries) ”部分中所示， [模板和调用](block-statements.md#template-call) 语句提供类似模式。
不能。HTL具有强大的扩展机制，用于重复使用逻辑- [使用API](getting-started.md#use-api-for-accessing-logic) 和标记( [模板和调用](block-statements.md#template-call) 语句)，可用于模块化项目代码。

**HTL相比JSP有哪些主要益处？**安全性和项目效率是主要优势， [概述概述](overview.md)。

**JSP最终会消失吗？**在当前日期，这些线上没有计划。

## HTL的基本概念 {#fundamental-concepts-of-htl}

HTML模板语言使用表达式语言将部分内容插入呈现的标记，以及HTML数据属性以定义通过标记块(如条件或迭代)定义语句。当HTL编译到Java Servlet中时，表达式和HTL数据属性都完全在服务器端评估，而在生成的HTML中不会保持可见。

### 块和表达式 {#blocks-and-expressions}

下面是第一个示例，它可能包含在 **`template.html`** 文件中：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以区分两种不同类型的语法：

* **[块语句](block-statements.md)**以条件显示 **&
amp；t；h& amp；gt；** 元素，使用 `[data-sly-test](block-statements.md#test)` HTML数据属性。HTL提供了多个此类属性，允许将行为附加到任何HTML元素，并且所有这些属性都是前缀 `data-sly`。

* **[表达式语言](expression-language.md)**HTL表达式由字符 `${` 和字符分隔 `}`。在运行时，对这些表达式进行评估，并将其值注入到传出HTML流中。

以上链接的两个页面提供可用于语法的详细功能列表。

### SLY元素 {#the-sly-element}

>[!NOTE]
>
>AEM6.1或HTL1.1已引入SLY元素。
>
>在此之前，必须使用 `[data-sly-unwrap](block-statements.md)` 该属性。

HTL的一个中央概念是提供重用现有HTML元素的可能性，以定义块语句，这避免了插入额外分隔符来定义语句开始和结束的需求。此标记不显眼的标记，可将静态HTML转换为有效的动态模板，从而提供不破坏HTML代码有效性的优点，因此依然可以正确显示，即使静态文件也是如此。

但是，有时可能不存在必须插入块语句的确切位置的现有元素。在这种情况下，可以插入一个特殊的SLY元素，它将自动从输出中删除，同时执行附加的块语句并相应地显示其内容。

因此，以下示例：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

将输出类似于HTML的内容，但前提条件仅是定义了一个、一个 **`jcr:title`** 和一 **`jcr:decription`** 个属性，并且其中没有一个属性为空：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

但要注意的一点是只在不使用块语句对现有元素添加注释的情况下使用SLY元素，因为SLY元素在使静态HTML成为动态HTML时不会更改静态HTML的值。

例如，如果上一示例已经在DIV元素内打包，则添加的SLY元素会受到谩骂：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

并且DIV元素可能已在条件中添加注释：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL注释 {#htl-comments}

以下示例显示了 **一** 个HTL注释，第 **行上显示** HTML评论：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL注释是HTML注释，其他类似JavaScript的语法。整个HTL注释，并且处理器中的任何内容都将完全忽略，并从输出中删除。

但是将通过标准HTML注释的内容，并将评估评论中的表达式。

HTML注释不能包含HTL注释，反之亦然。

### 特殊上下文 {#special-contexts}

为了能够充分利用HTL，很重要地了解它基于HTML语法的后果。

### 元素和属性名称 {#element-and-attribute-names}

表达式只能放置在HTML文本或属性值中，而不能放在元素名称或属性名称中，或者它不再是有效的HTML。为了动态设置元素名称，可以在所需元素上使用该 [`data-sly-element`](block-statements.md#element) 语句，并动态设置属性名称，甚至同时设置多个属性，可以使用 [`data-sly-attribute`](block-statements.md#attribute) 语句。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 无块语句的上下文 {#contexts-without-block-statements}

由于HTL使用数据属性定义块语句，因此无法在以下上下文中定义此类块语句，只能在此处使用表达式：

* HTML注释
* 脚本元素
* 样式元素

原因在于这些上下文的内容是文本而不是HTML，包含HTML元素将被视为简单的字符数据。因此，没有真正的HTML元素，也不能执行 **`data-sly`** 属性。

这听起来也很大，但它是一个需要的限制，因为HTML模板语言不应该被滥用来生成不是HTML的输出。[以下“访问逻辑](getting-started.md#use-api-for-accessing-logic) ”部分的使用API介绍了如何从模板调用其他逻辑，如果需要为这些上下文准备复杂输出，则可以使用该方法。例如，将数据从后端发送到前端脚本的一个简单方法是，使用该组件的逻辑生成一个JSON字符串，随后可以使用简单的HTL表达式放入数据属性中。

以下示例说明HTML注释的行为，但在脚本或样式元素中，会观察到相同的行为：

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

将输出类似HTML的内容：

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### 需要显式上下文 {#explicit-contexts-required}

如下 [自动上下文识别转义](getting-started.md#automatic-context-aware-escaping) 部分中所述，HTL的一个目标是通过自动将上下文感知转义应用到所有表达式来降低引入跨站点脚本(XSS)漏洞的风险。HTL可以自动检测放入HTML标记内的表达式上下文，但它不分析内嵌JavaScript或CSS的语法，因此依赖开发人员明确指定要应用于此类表达式的确切上下文。

由于没有在XSS漏洞中应用正确的转义，因此HTL因此删除了在未声明上下文的情况下在脚本和样式上下文中存在的所有表达式的输出。

下面是如何设置放置在脚本和样式内的表达式上下文的示例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

有关如何控制转义的详细信息，请参阅 [“表达式语言显示上下文](expression-language.md#display-context) ”部分。

### 提升特殊上下文的限制 {#lifting-limitations-of-special-contexts}

在特殊情况下，要绕过脚本、样式和注释上下文限制，可以将其内容分离为单独的HTL文件。位于其自身文件中的所有内容都将由HTL解释为普通HTML片段，从而获取可能已包含的限制上下文。

有关 [示例，请参阅进一步使用客户端模板](getting-started.md#working-with-client-side-templates) 部分。

>[!CAUTION]
>
>此技术可能引入跨站点脚本(XSS)漏洞，如果使用该技术，应仔细研究安全方面。通常情况下，实现与依赖此做法相同的功能通常更好。

## HTL的一般功能 {#general-capabilities-of-htl}

本节简要介绍了HTML模板语言的一般功能。

### 使用API访问逻辑 {#use-api-for-accessing-logic}

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

`logic.js` 服务器端执行的JavaScript文件放在它旁边：

```
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由于HTML模板语言不允许在标记内混合代码，因此它提供了使用API扩展机制，以便轻松地从模板执行代码。

上面的示例使用服务器端执行的JavaScript将标题缩短到10个字符，但它也可能已使用Java代码提供完全限定的Java类名称。通常，业务逻辑应使用Java构建，但如果组件需要从Java API提供的特定视图更改，则使用某些服务器端执行的JavaScript会很方便。

有关以下部分的更多信息：

* [data-sly-use语句中的部分](block-statements.md#use) 解释了使用该语句可以完成的所有操作。
* [使用API页面](use-api.md) 提供一些信息，用于帮助选择Java或JavaScript中的逻辑。
* 详细说明如何编写逻辑、 [JavaScript使用API](use-api-javascript.md) 和 [Java使用API](use-api-java.md) 页面。

### 自动上下文感知型转义 {#automatic-context-aware-escaping}

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多数模板语言中，此示例可能会创建跨站点脚本(XSS)漏洞，因为即使所有变量自动自动退出，该 `href` 属性仍必须明确地进行URL转义。这是最常见的错误之一，因为它可能被很容易遗忘，而且很难以自动化方式辨别。

为此，HTML模板语言会自动自动将每个变量自动转义到放置它的上下文。这是由于HTL理解HTML语法这一事实。

假定 `logic.js` 以下文件：

```
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

最初的示例将导致以下输出：

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

注意这两个属性如何以不同的方式转义，因为HTL知道， `href` 必须为URI上下文转义 `src` 属性。此外，如果开始使用 **`javascript:`**URI，则该属性将完全删除，除非上下文明确更改为其他内容。

有关如何控制转义的详细信息，请参阅 [“表达式语言显示上下文](expression-language.md#display-context) ”部分。

### 自动删除空属性 {#automatic-removal-of-empty-attributes}

请考虑以下示例：

```xml
<p class="${properties.class}">some text</p>
```

如果 `class` 该属性的值恰好为空，则HTML模板语言将自动从输出中删除整个 `class` 属性。

同样，这是可能的，因为HTL理解HTML语法，因此仅当其值不是空时才具有动态值的属性显示属性。这非常方便，因为它避免了在属性周围添加条件块，这会使标记无效且无法辨认。

此外，放入表达式中的变量类型非常重要：

* **String:**
   * **not ty：** 将字符串设置为属性值。
   * **空：** 完全删除属性。

* **数字：** 将该值设置为属性值。

* **布尔型:**
   * **true：** 显示不带值的属性(作为Boolean HTML属性)
   * **false：** 完全删除属性。

下面是一个Boolean表达式如何允许控制Boolean HTML属性的示例：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

对于设置属性，该 [`data-sly-attribute`](block-statements.md#attribute) 语句可能也很有用。

## HTL常见图案 {#common-patterns-with-htl}

本节介绍几个常见的情况，以及如何使用HTML模板语言最好地解决这些问题。

### 加载客户端库 {#loading-client-libraries}

在HTL中，客户端库通过AEM提供的辅助模板加载，可通过 [`data-sly-use`](block-statements.md#use)该模板访问。此文件中有三个模板，可通过以下方式调用 [`data-sly-call`](block-statements.md#template-call)：

* **`css`** - 仅加载引用的客户端库的CSS文件。
* **`js`** - 仅加载引用的客户端库的JavaScript文件。
* **`all`** - 加载引用的客户端库的所有文件(CSS和JavaScript)。

每个辅助模板都期望一个 **`categories`** 用于引用所需客户端库的选项。该选项可以是字符串值的数组，也可以是包含逗号分隔值列表的字符串。

以下是两个简短示例：

### 完全同时加载多个客户端库 {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 在页面的不同部分引用客户端库 {#referencing-a-client-library-in-different-sections-of-a-page}

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

在上面的第二个示例中，如果将HTML **`head`** 和 **`body`** 元素放入不同的文件中， **`clientlib.html`** 则必须在需要它的每个文件中加载模板。

[模板和调用](block-statements.md#template-call) 语句上的部分提供有关声明和调用此类模板的详细信息。

### 将数据传递给客户端 {#passing-data-to-the-client}

向客户一般传递数据最好、最流畅的方法是使用数据属性。

以下示例说明了逻辑(也可以使用Java编写的逻辑)非常方便地序列化要传递给客户端的对象，随后可以非常容易地放置到数据属性中：

```xml
<!--/* template.html file: */-->
<div data-sly-use.logic="logic.js" data-json="${logic.json}">...</div>
```

```
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

从此处，很容易想象客户端JavaScript如何访问该属性并再次解析JSON。这将作为相应的JavaScript放置到客户端库中：

```
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用客户端模板 {#working-with-client-side-templates}

一种特殊情况是，在“特殊上下文”部分 [的“提升特殊上下文](getting-started.md#lifting-limitations-of-special-contexts) ”部分中所述的技术可以合法地使用，即编写 **位于脚本** 元素内的客户端模板(例如“手持工具栏”)。这种技术可以安全地使用的原因在于 **，脚本** 元素不包含假设的JavaScript，而是进一步的HTML元素。下面是一个如何工作的示例：

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

如上所述，将包含在 **`script`** 元素中的标记可以包含HTL块语句，而且表达式无需提供明确的上下文，因为HandleBars模板的内容已被分离在其自身的文件中。此外，此示例显示服务器端执行HTL(如元素在 **`h2`** 元素上)如何与客户端执行的模板语言(如在 **`h3`** 元素上显示)混合。

然而，一个更现代化的技术完全是使用HTML **`template`** 元素，而好处在于它不需要将模板的内容分离为单独的文件。

**阅读下一步：**

* [表达式语言](expression-language.md) -详细了解可在HTL表达式中完成的操作。
* [块语句](block-statements.md) -找出HTL中可用的所有块语句，以及如何使用它们。
