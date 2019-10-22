---
title: HTL 快速入门
seo-title: HTL 快速入门
description: AEM支持的HTL代替JSP作为AEM中HTML的首选和推荐的服务器端模板系统。
seo-description: Adobe Experience manager支持的HTML模板语言- HTL代替JSP作为AEM中HTML的首选和推荐的服务器端模板系统。
uuid: 4a7d6748-8cdf-4280-a85d-6c5319abf487
content-type: 参考
topic-tags: 简介
discoiquuid: 3bf2ca75-0d68-489d-bd1c-1d4fd730c61a
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 6de5ed20e4463c0c2e804e24cb853336229a7c1f

---


# HTL 快速入门 {#getting-started-with-htl}

Adobe Experience Manager(AEM)支持的HTML模板语言(HTL)代替JSP(JavaServer Pages)作为AEM中HTML的首选和推荐的服务器端模板系统。

>[!NOTE]
>
>要运行本页上提供的大多数示例，可使用称为“读取 [Eval打印循环](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl) ”的实时执行环境。
>
>AEM社区已生成一系列与使用HTL [相关的文章](related-community-articles.md) 、视频和网络研讨会。

## HTL over JSP {#htl-over-jsp}

建议新的AEM项目使用HTML模板语言，因为与JSP相比，该语言具有多种优势。 但是，对于现有项目，只有在预计未来几年花费的精力少于维护现有JSP的情况下，迁移才有意义。

但是，转向HTL并不一定是一个全选或全无的选择，因为用HTL编写的组件与用JSP或ESP编写的组件兼容。 这意味着现有项目可以无问题地对新组件使用HTL，同时保留现有组件的JSP。

即使在同一组件中，HTL文件也可以与JSP和ESP一起使用。 以下示例在第 **1行** （如何从JSP文件包含HTL文件）和第2 **行(如何从HTL文件包含JSP文件** )中演示：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常见问题{#frequently-asked-questions}

在开始使用HTML模板语言之前，我们先回答与JSP与HTL主题相关的一些问题。

**HTL是否有JSP不具备的任何限制？**
与JSP相比，HTL并没有任何限制，因为HTL也应该可以实现对JSP的使用。 但是，HTL在设计上比JSP在几个方面更加严格，而在单个JSP文件中可以实现的，可能需要将HTL分成Java类或JavaScript文件才能在HTL中实现。 但是，这通常是为了确保逻辑和标记之间的顾虑有良好的分离。

**HTL是否支持JSP标签库？**
否，但正如“加载客户端 [库”部分所示](getting-started.md#loading-client-libraries) ，模板和 [](block-statements.md#template-call) call语句提供类似的模式。

**HTL功能是否可以在AEM项目上扩展？**
否，但正如“加载客户端 [库”部分所示](getting-started.md#loading-client-libraries) ，模板和 [](block-statements.md#template-call) call语句提供类似的模式。
不，他们不能。 HTL具有用于重用逻辑的强大扩展机制- [Use-API](getting-started.md#use-api-for-accessing-logic) —— 和标记( [template &amp; call](block-statements.md#template-call) statements)，这些机制可用于模块化项目的代码。

**与JSP相比，HTL有哪些主要优势？**
安全性和项目效率是主要优势，详见概 [述](overview.md)。

**JSP最终会消失吗？**
在当前日期，没有这些计划。

## HTL的基本概念 {#fundamental-concepts-of-htl}

HTML模板语言使用表达式语言将内容片段插入渲染的标记中，HTML5数据属性用于定义标记块（如条件或迭代）上的语句。 当HTL编译为Java Servlets时，表达式和HTL数据属性将完全在服务器端计算，结果HTML中不会显示任何内容。

### 块和表达式 {#blocks-and-expressions}

以下是第一个示例，它可以像在文件中一样包 **`template.html`** 含：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以区分两种不同的语法：

* **[块语句](block-statements.md)**&#x200B;要有条件地显 **示&lt;h1&gt;元素** ，请使用 `[data-sly-test](block-statements.md#test)` HTML5数据属性。 HTL提供多个此类属性，这些属性允许将行为附加到任何HTML元素，所有这些属性都带有前缀 `data-sly`。

* **[表达式语](expression-language.md)**&#x200B;言HTL表达式由字符和 `${` 分隔 `}`。 在运行时，将计算这些表达式，并将其值注入传出的HTML流中。

上面链接的两个页面提供了可用于语法的详细功能列表。

### SLY元素 {#the-sly-element}

>[!NOTE]
>
>SLY元素已在AEM 6.1或HTL 1.1中引入。
>
>在此之前，必 `[data-sly-unwrap](block-statements.md)` 须改用属性。

HTL的一个核心概念是提供重用现有HTML元素来定义块语句的可能性，这避免了插入其他分隔符来定义语句的开始和结束位置的需要。 将静态HTML转换为有效动态模板的标记的这一不显眼的注释不会破坏HTML代码的有效性，因此即使作为静态文件也能正常显示。

但是，有时可能在必须插入块语句的确切位置不存在现有元素。 对于这种情况，可以插入将自动从输出中移除的特殊SLY元素，同时执行附加的块语句并相应地显示其内容。

下面是一个例子：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

将输出类似于以下HTML的内容，但仅当同时定义了一个和一 **`jcr:title`** 个属 **`jcr:decription`** 性，并且它们都不为空时：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

但需要注意的一点是，只有在没有现有元素可以用块语句进行注释的情况下才使用SLY元素，因为SLY元素阻止了语言提供的值，使静态HTML变为动态时不会改变。

例如，如果上一个示例已包装在DIV元素中，则添加的SLY元素将是谩骂：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

并且DIV元素可能已添加以下条件注释：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL注释 {#htl-comments}

以下示例显示 **在第1行** HTL注释和第2 **行** HTML注释：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL注释是HTML注释，带有其他类似于JavaScript的语法。 处理器将完全忽略整个HTL注释以及其中的任何内容，并从输出中删除。

但是，标准HTML注释的内容将被传递，注释中的表达式将被计算。

HTML注释不能包含HTL注释，反之亦然。

### 特殊上下文 {#special-contexts}

为了能够最好地利用HTL，必须更好地理解它基于HTML语法的后果。

### 元素和属性名称 {#element-and-attribute-names}

表达式只能放入HTML文本或属性值中，但不能放在元素名称或属性名称中，否则将不再是有效的HTML。 为了动态设置元素名称，可以对所需的元素使用该语句，并动态设置属性名称，即使同时设置多个属性，也可以使用该 [`data-sly-element`](block-statements.md#element)[`data-sly-attribute`](block-statements.md#attribute) 语句。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 无块语句的上下文 {#contexts-without-block-statements}

由于HTL使用数据属性来定义块语句，因此无法在以下上下文中定义此类块语句，并且只能在此处使用表达式：

* HTML注释
* 脚本元素
* 样式元素

其原因是这些上下文的内容是文本而不是HTML，包含的HTML元素将被视为简单的字符数据。 因此，没有真正的HTML元素，也无法执行 **`data-sly`** 属性。

这听起来可能像一个很大的限制，但是它是理想的限制，因为不应滥用HTML模板语言来生成非HTML的输出。 下面 [的“访问逻辑的使用](getting-started.md#use-api-for-accessing-logic) -API”部分介绍了如何从模板中调用其他逻辑，如果需要为这些上下文准备复杂的输出，则可以使用该模板。 例如，将数据从后端发送到前端脚本的一种简单方法是使组件的逻辑生成JSON字符串，然后该字符串可以用简单的HTL表达式放入数据属性中。

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

### 需要显式上下文 {#explicit-contexts-required}

如下面的“自动上下文感知型转义 [](getting-started.md#automatic-context-aware-escaping) ”部分中所述，HTL的一个目标是通过自动将上下文感知型转义应用于所有表达式，降低引入跨站点脚本(XSS)漏洞的风险。 虽然HTL可以自动检测置于HTML标记内的表达式的上下文，但它不分析内联JavaScript或CSS的语法，因此依赖开发人员明确指定应将这些表达式应用到的确切上下文。

由于未在XSS漏洞中应用正确的转义结果，因此，当尚未声明上下文时，HTL会删除脚本和样式上下文中的所有表达式的输出。

下面是一个有关如何为放置在脚本和样式中的表达式设置上下文的示例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

有关如何控制转义的详细信息，请参阅“表达式语言显 [示上下文](expression-language.md#display-context) ”部分。

### 特殊语境的解除限制 {#lifting-limitations-of-special-contexts}

在需要绕过脚本、样式和注释上下文限制的特殊情况下，可以在单独的HTL文件中隔离其内容。 HTL会将位于其自己文件中的所有内容解释为普通HTML片段，而忽略可能包含它的限制上下文。

有关示 [例，请参阅使用客户端模板](getting-started.md#working-with-client-side-templates) 。

>[!CAUTION]
>
>该技术可以引入跨站点脚本(XSS)漏洞，如果使用该漏洞，应仔细研究其安全性。 通常有比依靠这种做法更好的办法来实施同样的事情。

## HTL的一般功能 {#general-capabilities-of-htl}

本节将快速介绍HTML模板语言的一般功能。

### 用于访问逻辑的Use-API {#use-api-for-accessing-logic}

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

以及位 `logic.js` 于其旁边的服务器端执行的JavaScript文件：

```
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由于HTML模板语言不允许在标记内混合代码，因此它提供了Use-API扩展机制，以便从模板轻松执行代码。

上述示例使用服务器端执行的JavaScript将标题缩短为10个字符，但通过提供完全限定的Java类名称，它也可以使用Java代码执行相同操作。 通常，业务逻辑应该是使用Java构建的，但当组件需要从Java API提供的内容进行某些特定于视图的更改时，使用服务器端执行的JavaScript来实现这一操作会很方便。

以下各节中详细介绍了这一点：

* 数据密钥使 [用语句的部分介绍](block-statements.md#use) ，可以使用该语句执行的所有操作。
* “ [Use-API”页提供一些信息](use-api.md) ，可帮助选择使用Java或JavaScript编写逻辑。
* 要详细说明如何编写逻辑， [JavaScript Use-API](use-api-javascript.md) 和 [](use-api-java.md) Java Use-API页应该有帮助。

### 自动上下文感知转义 {#automatic-context-aware-escaping}

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多数模板语言中，此示例可能会创建跨站点脚本(XSS)漏洞，因为即使所有变量都是自动HTML转义的，该属性也必须 `href` 特别是URL转义的。 这是最常见的错误之一，因为很容易被遗忘，而且很难以自动方式发现。

为此，HTML模板语言会自动将每个变量相应地转义到放置该变量的上下文。 由于HTL了解HTML的语法，这是可能的。

假定以下 `logic.js` 文件：

```
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

初始示例将生成以下输出：

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

注意两个属性是如何以不同方式转义的，因为HTL知道 `href` URI上 `src` 下文必须转义和属性。 此外，如果URI以开头， **`javascript:`**&#x200B;则属性将被完全删除，除非上下文明确更改为其他内容。

有关如何控制转义的详细信息，请参阅“表达式语言显 [示上下文](expression-language.md#display-context) ”部分。

### 自动删除空属性 {#automatic-removal-of-empty-attributes}

请考虑以下示例：

```xml
<p class="${properties.class}">some text</p>
```

如果属性的值 `class` 恰好为空，则HTML模板语言将自动从输出中删除 `class` 整个属性。

同样，这也是可能的，因为HTL了解HTML语法，因此，仅当属性的值不为空时，才有条件地显示具有动态值的属性。 这非常方便，因为它避免在属性周围添加条件块，这会使标记无效且不可读。

此外，位于表达式中的变量类型也很重要：

* **String:**
   * **** 不为空：将字符串设置为属性值。
   * **** 空：完全删除属性。

* **** 数字：将值设置为属性值。

* **布尔型:**
   * **** true:显示无值属性（作为布尔HTML属性）
   * **** false:完全删除属性。

以下是一个Boolean表达式如何允许控制Boolean HTML属性的示例：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

对于设置属性，该 [`data-sly-attribute`](block-statements.md#attribute) 语句可能也很有用。

## 使用HTL的常见模式 {#common-patterns-with-htl}

本节介绍一些常见场景以及如何使用HTML模板语言最好地解决这些场景。

### 加载客户端库 {#loading-client-libraries}

在HTL中，客户端库通过AEM提供的帮助模板加载，可通过访问该模板 [`data-sly-use`](block-statements.md#use)。 此文件中提供了三个模板，可通过以下方式调用这些模板 [`data-sly-call`](block-statements.md#template-call):

* **`css`** -仅加载引用的客户端库的CSS文件。
* **`js`** -仅加载引用的客户端库的JavaScript文件。
* **`all`** -加载引用的客户端库的所有文件（CSS和JavaScript）。

每个帮助程序模板都需要 **`categories`** 一个用于引用所需客户端库的选项。 该选项可以是字符串值的数组，也可以是包含以逗号分隔的值列表的字符串。

以下是两个简短的示例：

### 一次完全加载多个客户端库 {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 在页面的不同部分中引用客户端库 {#referencing-a-client-library-in-different-sections-of-a-page}

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

在上面的第二个示例中，如果HTML和 **`head`** 元素 **`body`****`clientlib.html`** 被放置到不同的文件中，则模板随后必须加载到每个需要它的文件中。

模板和调用语 [句的部分提供了有关声明](block-statements.md#template-call) 和调用此类模板的工作方式的更多详细信息。

### 将数据传递到客户端 {#passing-data-to-the-client}

一般而言，将数据传递给客户端的最佳、最优雅的方法是使用数据属性，但使用HTL时更是如此。

以下示例演示如何使用逻辑（也可以用Java编写）非常方便地将要传递到客户端的对象序列化为JSON，然后将该对象非常容易地放入数据属性中：

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

从此处，可以轻松想象客户端JavaScript如何访问该属性并再次解析JSON。 例如，这将是放入客户端库中的相应JavaScript:

```
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用客户端模板 {#working-with-client-side-templates}

一个特殊情况是编写位于脚本元素中的客户端模板（如Handlebars），在 [Lifting Limitions of Special Contexts](getting-started.md#lifting-limitations-of-special-contexts) （特殊上下文的解除限制）一节中所述的技术可以合法使用 **** 。 在这种情况下可以安全地使用此技术的原因是 **script** 元素不像假定的那样包含JavaScript，而是包含更多HTML元素。 下面是如何实现这一目的的示例：

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

如上所示，将包含在元素中的标记可以包含HTL块语句，而表达式无需提供显式上下文，因为Handlebars模板的内容已隔离在其自己的文件中。 **`script`** 此外，此示例还说明如何将服务器端执行的HTL（如元素）与客户端执行的模板语言(如Handlebars（如元素中所示）)混 **`h2`****`h3`** 合使用。

但是，更现代的技巧是改用HTML元素，因为这样做的好处是，它不需要将模板的内容分离为单独的文件。 **`template`**

**阅读下一篇文章：**

* [表达式语言](expression-language.md) -详细了解在HTL表达式中可以执行哪些操作。
* [块语句](block-statements.md) -发现HTL中可用的所有块语句，以及如何使用它们。
