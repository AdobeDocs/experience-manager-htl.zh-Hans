---
title: HTL 快速入门
description: AEM 支持的 HTL 取代 JSP 作为 AEM 中适用于 HTML 的首选和推荐的服务器端模板系统。
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '2471'
ht-degree: 100%

---

# HTL 快速入门 {#getting-started-with-htl}

Adobe Experience Manager (AEM) 支持的 HTML 模板语言 (HTL) 是 AEM 中适用于 HTML 的首选和推荐的服务器端模板系统。它取代 AEM 的以前版本中使用的 JSP (JavaServer Pages)。

>[!NOTE]
>
>要运行本页面上提供的大多数示例，可以使用称为[读取-求值-打印循环](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl)的实时执行环境。

## HTL 优于 JSP {#htl-over-jsp}

建议新的 AEM 项目使用 HTML 模板语言，因为与 JSP 相比，它可提供多种好处。但是，对于现有项目，只有在估计迁移比未来几年维护现有 JSP 的工作量少时，迁移才有意义。

但迁移到 HTL 不一定是全有或全无的选择，因为使用 HTL 编写的组件与使用 JSP 或 ESP 编写的组件兼容。这意味着现有项目可以毫无问题地将 HTL 用于新组件，同时为现有组件保留 JSP。

甚至在同一个组件中，HTL 文件也可以与 JSP 和 ESP 一起使用。以下示例在&#x200B;**第 1 行**&#x200B;上显示如何从 JSP 文件中包括 HTL 文件，在&#x200B;**第 2 行**&#x200B;上显示如何从 HTL 文件中包括 JSP 文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常见问题解答{#frequently-asked-questions}

在开始使用 HTML 模板语言之前，让我们先回答一些和 JSP 与 HTL 主题相关的问题。

**HTL 是否有 JSP 没有的任何限制？** - 与 JSP 相比，HTL 真的没有限制，因为可以使用 JSP 完成的任务使用 HTL 应该也能够实现。但是，在多个方面，HTL 在设计上比 JSP 更严格，并且在单个 JSP 文件中可以全部实现的任务可能需要分离到 Java 类或 JavaScript 文件中才能在 HTL 中实现。但这通常用于确保在逻辑和标记之间良好地分离关注点。

**HTL 是否支持 JSP 标记库？** - 不支持，但如[加载客户端库](getting-started.md#loading-client-libraries)部分中所示，[template 和 call](block-statements.md#template-call) 语句提供了类似模式。

**能否在 AEM 项目上扩展 HTL 功能？** - 不能。HTL 具有强大的扩展机制以重复使用逻辑 ([Use-API](getting-started.md#use-api-for-accessing-logic)) 和标记（[template 和 call](block-statements.md#template-call) 语句），它们可用于实现项目代码的模块化。

**HTL 与 JSP 相比有何主要好处？** - 安全性和项目效率是主要好处，在[概述](overview.md)中对此进行了详细介绍。

**JSP 最终会消失吗？** - 目前，还没有这些方面的计划。

## HTL 的基本概念 {#fundamental-concepts-of-htl}

HTML 模板语言使用表达式语言将内容片段插入到呈现的标记中，并使用 HTML5 数据属性来定义标记块上的语句（如条件或迭代）。在 HTL 编译为 Java Servlet 时，表达式和 HTL 数据属性都完全在服务器端进行计算，不会显示在生成的 HTML 中。

### 块和表达式 {#blocks-and-expressions}

下面是第一个示例，它可以按原样包含在 **`template.html`** 文件中：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以辨别出两种不同种类的语法：

* **[块语句](block-statements.md)** - 为有条件地显示 **&lt;h1>** 元素，使用了 [`data-sly-test`](block-statements.md#test) HTML5 数据属性。HTL 提供了多个此类属性，允许将行为附加到任何 HTML 元素，并且所有属性都以 `data-sly` 为前缀。

* **[表达式语言](expression-language.md)** - HTL 表达式通过字符 `${` 和 `}` 进行分隔。在运行时，计算这些表达式并将其值注入到传出的 HTML 流中。

上面链接的两个页面提供了可用于语法的功能的详细列表。

### SLY 元素 {#the-sly-element}

HTL 的一个中心概念是可以重复使用现有的 HTML 元素来定义块语句，从而无需插入额外的分隔符来定义语句的开始和结束位置。这种将静态 HTML 转换为正常使用的动态模板的不造成干扰的标记注释提供了不破坏 HTML 代码的有效性的好处，因此即使作为静态文件也能正确显示。

但是，有时在必须插入块语句的确切位置上可能没有现有元素。对于这种情况，可以插入一个特殊的 SLY 元素，将自动从输出中删除该元素，但会执行附加的块语句并相应地显示其内容。

因此，以下示例

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

将输出类似于以下 HTML 的内容，但前提是同时定义了 **`jcr:title`** 和 **`jcr:description`** 属性，并且它们都不为空：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

要记住的一件事是，仅当没有可使用块语句进行注释的现有元素时才使用 SLY 元素，因为 SLY 元素会阻止语言提供的值以在静态 HTML 变为动态时不改变静态 HTML。

例如，如果前面的示例已经封装在 DIV 元素中，那么添加的 SLY 元素将是滥用的：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

并且 DIV 元素可以使用条件进行注释：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL 注释 {#htl-comments}

以下示例在&#x200B;**第 1 行**&#x200B;上显示 HTL 注释，在&#x200B;**第 2 行**&#x200B;上显示 HTML 注释：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL 注释是使用其他类似于 JavaScript 的语法的 HTML 注释。整个 HTL 注释及其中的任何内容都将被处理器完全忽略并从输出中删除。

不过，将传递标准 HTML 注释的内容，并计算注释中的表达式。

HTML 注释不能包含 HTL 注释，反之亦然。

### 特殊上下文 {#special-contexts}

为了能够充分利用 HTL，很好地了解它基于 HTML 语法的结果很重要。

### 元素和属性名称 {#element-and-attribute-names}

表达式只能放置到 HTML 文本或属性值中，而不能放置到元素名称或属性名称中，否则它就不再是有效的 HTML。要动态设置元素名称，可以在所需的元素上使用 [`data-sly-element`](block-statements.md#element) 语句；要动态设置属性名称，甚至同时设置多个属性，可以使用 [`data-sly-attribute`](block-statements.md#attribute) 语句。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 没有块语句的上下文 {#contexts-without-block-statements}

由于 HTL 使用数据属性来定义块语句，因此无法在以下上下文中定义此类块语句，并且只能在这里使用表达式：

* HTML 注释
* Script 元素
* Style 元素

原因是这些上下文的内容是文本而不是 HTML，并且包含的 HTML 元素将被视为简单的字符数据。因此，没有真正的 HTML 元素，也无法执行 **`data-sly`** 属性。

这可能听起来像是一个很大的限制，但需要如此，因为不应滥用 HTML 模板语言来生成非 HTML 的输出。下面的[用于访问逻辑的 Use-API](getting-started.md#use-api-for-accessing-logic) 部分介绍了如何从模板调用其他逻辑，可以在需要为这些上下文准备复杂输出时使用它。例如，将数据从后端发送到前端脚本的简单方法是让组件的逻辑生成一个 JSON 字符串，然后可以使用简单的 HTL 表达式将其放在数据属性中。

以下示例说明了 HTML 注释的行为，但在 script 或 style 元素中，将观察到相同的行为：

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

将输出类似于以下 HTML 的内容：

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### 需要显式上下文 {#explicit-contexts-required}

如下面的[自动上下文感知转义](getting-started.md#automatic-context-aware-escaping)部分中所述，HTL 的一个目标是通过自动将上下文感知转义应用于所有表达式来降低引入跨站点脚本 (XSS) 漏洞的风险。虽然 HTL 可以自动检测放置在 HTML 标记内的表达式的上下文，但它不会分析内联 JavaScript 或 CSS 的语法，因此依赖开发人员显式指定必须将哪个确切上下文应用于此类表达式。

由于没有在 XSS 漏洞中应用正确的转义结果，因此 HTL 会在尚未声明上下文时删除脚本和样式上下文中的所有表达式的输出。

下面是如何为放置在脚本和样式中的表达式设置上下文的示例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

有关如何控制转义的更多详细信息，请参阅[表达式语言显示上下文](expression-language.md#display-context)部分。

### 解除特殊上下文的限制 {#lifting-limitations-of-special-contexts}

在需要绕过脚本、样式和注释上下文的限制的特殊情况下，可以将它们的内容隔离在单独的 HTL 文件中。位于自己文件中的所有内容都将由 HTL 解释为普通的 HTML 片段，而忘记了可能包含它的限制上下文。

请参阅下面的[使用客户端模板](getting-started.md#working-with-client-side-templates)部分来获取示例。

>[!CAUTION]
>
>这种技术可能会引入跨站点脚本 (XSS) 漏洞，如果使用它，应仔细研究安全方面。通常有比依赖这种做法更好的方法来实现同样的目的。

## HTL 的常规功能 {#general-capabilities-of-htl}

此部分快速介绍 HTML 模板语言的常规功能。

### 用于访问逻辑的 Use-API {#use-api-for-accessing-logic}

考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

以及放置在它旁边的以下 `logic.js` 服务器端执行的 JavaScript 文件：

```javascript
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由于 HTML 模板语言不允许在标记中混合代码，它提供了 Use-API 扩展机制来轻松地从模板执行代码。

上面的示例使用服务器端执行的 JavaScript 将标题缩短为 10 个字符，但它也可以使用 Java 代码通过提供完全限定的 Java 类名来执行相同的操作。通常，业务逻辑应该使用 Java 进行构建，但是当组件需要从 Java API 提供的内容中进行一些特定于视图的更改时，使用某个服务器端执行的 JavaScript 来执行该操作会很方便。

有关更多信息，请参阅以下部分：

* 有关 [`data-sly-use` 语句](block-statements.md#use)的部分说明可使用该语句完成的所有操作。
* [Use-API 页面](use-api.md)提供了一些信息来帮助选择使用 Java 还是 JavaScript 来编写逻辑。
* 要详细了解如何编写逻辑，[JavaScript Use-API](use-api-javascript.md) 和 [Java Use-API](use-api-java.md) 页面应该有帮助。

### 自动上下文感知转义 {#automatic-context-aware-escaping}

考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多数模板语言中，此示例可能会造成跨站点脚本 (XSS) 漏洞，因为即使所有变量都会自动进行 HTML 转义，`href` 属性仍必须经过专门的 URL 转义。这种遗漏是常见的错误之一，因为它很容易被遗忘，并且很难自动发现。

为帮助解决这个问题，HTML 模板语言会根据每个变量所在的上下文自动对每个变量进行转义。此功能的实现得益于 HTL 对 HTML 语法的理解。

假设以下 `logic.js` 文件：

```javascript
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

初始示例然后会生成以下输出：

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

请注意两个属性如何以不同的方式进行转义，因为 HTL 知道 `href` 和 `src` 属性必须针对 URI 上下文进行转义。此外，如果 URI 以 **`javascript:`** 开头，则除非上下文已显式更改为其他内容，否则将完全删除该属性。

有关如何控制转义的更多详细信息，请参阅[表达式语言显示上下文](expression-language.md#display-context)部分。

### 自动删除空属性 {#automatic-removal-of-empty-attributes}

考虑以下示例：

```xml
<p class="${properties.class}">some text</p>
```

如果 `class` 属性的值碰巧为空，则 HTML 模板语言将自动从输出中删除整个 `class` 属性。

此功能的实现同样得益于 HTL 对 HTML 语法的理解，因此只有当属性的值不为空时，才能有条件地显示具有动态值的属性。这非常方便，因为它避免了在属性周围添加条件块，这会使标记无效和不可读。

此外，放置在表达式中的变量的类型很重要：

* **字符串：**
   * **非空：**&#x200B;将字符串设置为属性值。
   * **空：**&#x200B;完全删除属性。

* **数字：**&#x200B;将值设置为属性值。

* **布尔型：**
   * **true：**&#x200B;显示没有值的属性（作为布尔 HTML 属性）
   * **false：**&#x200B;完全删除属性。

下面是布尔表达式如何允许控制布尔 HTML 属性的示例：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

要设置属性，[`data-sly-attribute`](block-statements.md#attribute) 语句可能也有用。

## 使用 HTL 的常见模式 {#common-patterns-with-htl}

此部分介绍一些常见场景以及如何使用 HTML 模板语言很好地解决它们。

### 加载客户端库 {#loading-client-libraries}

在 HTL 中，通过 AEM 提供的帮助程序模板来加载客户端库，可通过 [`data-sly-use`](block-statements.md#use) 访问模板。此文件中提供了三个模板，可通过 [`data-sly-call`](block-statements.md#template-call) 来调用它们：

* **`css`** - 仅加载引用的客户端库的 CSS 文件。
* **`js`** - 仅加载引用的客户端库的 JavaScript 文件。
* **`all`** - 加载引用的客户端库的所有文件（CSS 和 JavaScript）。

每个帮助程序模板都需要一个 **`categories`** 选项来引用所需的客户端库。该选项可以是字符串值的数组，也可以是包含逗号分隔值列表的字符串。

下面是两个简短示例：

### 同时完全加载多个客户端库 {#loading-multiple-client-libraries-fully-at-once}

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

在上面的第二个示例中，如果 HTML **`head`** 和 **`body`** 元素放置在不同的文件中，则 **`clientlib.html`** 模板必须加载到需要它的每个文件中。

有关 [template 和 call](block-statements.md#template-call) 语句的部分提供了有关声明和调用此类模板的工作原理的更多详细信息。

### 将数据传递给客户端 {#passing-data-to-the-client}

一般来说，将数据传递给客户端的更方便、更轻松的方式是使用数据属性，对于 HTL 来说更是如此。

以下示例显示了如何使用逻辑（也可以使用 Java 编写）来非常方便地将要传递给客户端的对象序列化为 JSON，然后可以很轻松地将其放置到数据属性中：

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

从那里，可以轻松地设想客户端 JavaScript 如何访问该属性并再次解析 JSON。例如，下面是要放置到客户端库中的相应 JavaScript：

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用客户端模板 {#working-with-client-side-templates}

可以合法使用[解除特殊上下文的限制](getting-started.md#lifting-limitations-of-special-contexts)部分中说明的技术的一种特殊情况是编写位于 **script** 元素内的客户端模板（例如 Handlebars）。在该情况下可以安全使用此技术的原因是，**script** 元素没有像假设的那样包含 JavaScript，而是包含更多的 HTML 元素。下面是有关它的工作原理的示例：

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

如上所示，将包含到 **`script`** 元素中的标记可以包含 HTL 块语句，并且表达式无需提供显式上下文，因为 Handlebars 模板的内容已被隔离在自己的文件中。此外，此示例显示了如何将服务器端执行的 HTL（例如在 **`h2`** 元素上）与 Handlebars 之类的客户端执行的模板语言（显示在 **`h3`** 元素上）混合。

不过，一种更现代的技术是改用 HTML **`template`** 元素，因为这样做的好处是无需将模板的内容隔离到单独的文件中。

**阅读下一篇文章：**

* [表达式语言](expression-language.md) - 详细了解可以在 HTL 表达式中完成的操作。
* [块语句](block-statements.md) - 了解 HTL 中可用的所有块语句，以及如何使用它们。
