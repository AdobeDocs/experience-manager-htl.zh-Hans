---
title: HTL 快速入门
description: 了解 HTL，AEM 中适用于 HTML 的首选和推荐的服务器端模板系统，并了解该语言的主要概念及其基本结构。
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---


# HTL 快速入门 {#getting-started-with-htl}

HTML 模板语言 (HTL) 是 Adobe Experience Manager 中适用于 HTML 的首选和推荐的服务器端模板系统。 像在所有 HTML 服务器端模板系统中一样，HTL 文件通过指定 HTML 本身、一些基本的表示逻辑和要在运行时计算的变量来定义发送到浏览器的输出。

本文档概述了 HTL 的用途，并介绍了该语言的基本概念和结构。

>[!TIP]
>
>本文档介绍了 HTL 的目的及其基本结构和概念。 如果您对特定语法有疑问，请参阅 [HTL 规范](specification.md)。

## HTL 图层 {#layers}

在 AEM 中，多个层定义了 HTL。

1. **[HTL 规范](specification.md)** – HTL 是一个开源、不依赖于平台的规范，任何人都可以自由实施。
1. **[Sling HTL 脚本引擎](specification.md)** – Sling 项目创建了 HTL 的参考实施，供 AEM 使用。
1. **[AEM 扩展](specification.md)** – AEM 构建在 Sling HTL 脚本引擎之上，以便为开发者提供 AEM 特有的方便功能。

本 HTL 文档侧重于使用 HTL 开发 AEM 解决方案。 因此，它涉及所有三层，必要时连接外部资源。

## HTL 的基本概念 {#fundamental-concepts-of-htl}

HTML 模板语言使用表达式语言将内容片段插入到呈现的标记中，并使用 HTML5 数据属性来定义标记块上的语句（如条件或迭代）。在 HTL 编译为 Java Servlet 时，表达式和 HTL 数据属性都完全在服务器端进行计算，不会显示在生成的 HTML 中。

>[!TIP]
>
>要运行本页面上提供的大多数示例，可以使用称为[读取-求值-打印-循环](https://github.com/adobe/aem-htl-repl)的实时执行环境。

### 块和表达式 {#blocks-and-expressions}

下面是第一个示例，它可以按原样包含在 `template.html` 文件中：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以辨别出两种不同种类的语法：

* **块语句** – 如果你想有条件地显示 `<h1>` 元素，请使用 `data-sly-test` HTML5 数据属性。 HTL 提供了多个此类属性，允许将行为附加到任何 HTML 元素，并且所有属性都以 `data-sly` 为前缀。
* **表达式语言** – HTL 表达式通过 `${` 和 `}` 字符进行分隔。 在运行时，计算这些表达式并将其值注入到传出的 HTML 流中。

有关两种语法的详细信息，请参阅 [HTL 规范](specification.md)。

### SLY 元素 {#the-sly-element}

HTL 的一个核心概念是提供重用现有 HTML 元素来定义块语句的可能性。这种重用避免了插入额外的分隔符来定义语句的开始和结束位置的需要。通过不干扰地注释标记，可以将静态 HTML 转换为动态模板，而不会破坏 HTML 有效性，从而确保即使是静态文件也能正确显示。

但是，有时在必须插入块语句的确切位置上可能没有现有元素。在这种情况下，您可以插入一个特殊的 `sly` 元素。在运行附加的块语句并相应地显示其内容时，此元素会自动从输出中删除。

以下示例：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

将输出类似于以下 HTML 的内容，但前提是同时定义了 `jcr:title` 和 `jcr:description` 属性，并且它们都不为空：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

要记住，只有在没有可用块语句注释现有元素时，才使用 `sly` 元素。 这是因为 `sly` 元素阻止该语言所提供的值，即在使其成为动态时不改变静态 HTML。

例如，如果前面的示例已经封装在 `div` 元素中，那么添加的 `sly` 元素将是滥用的：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

并且 `div` 元素可以使用条件进行注释：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL 注释 {#htl-comments}

以下示例在第 1 行上显示 HTL 注释，在第 2 行上显示 HTML 注释。

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL 注释是使用其他类似于 JavaScript 的语法的 HTML 注释。整个 HTL 注释及其中的任何内容都将被处理器完全忽略并从输出中删除。

不过，将传递标准 HTML 注释的内容，并计算注释中的表达式。

HTML 注释不能包含 HTL 注释，反之亦然。

### 特殊上下文 {#special-contexts}

为了能够充分利用 HTL，很好地了解它基于 HTML 语法的结果很重要。

有关详细信息，请参阅 HTL 规范的[显示上下文部分](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#121-display-context)。

### 元素和属性名称 {#element-and-attribute-names}

表达式只能放置到 HTML 文本或属性值中，而不能放置到元素名称或属性名称中，否则它就不再是有效的 HTML。 要动态设置元素名称，可以在所需的元素上使用 `data-sly-element` 语句；要动态设置属性名称，甚至同时设置多个属性，可以使用 `data-sly-attribute` 语句。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 没有块语句的上下文 {#contexts-without-block-statements}

由于 HTL 使用数据属性来定义块语句，因此无法在以下上下文中定义此类块语句，并且只能在这里使用表达式：

* HTML 注释
* Script 元素
* Style 元素

原因是这些上下文的内容是文本而不是 HTML，并且包含的 HTML 元素将被视为简单的字符数据。因此，没有真正的 HTML 元素，也无法执行 `data-sly` 属性。

这种方法听起来可能像是一个重大的限制。然而，它是首选，因为 HTML 模板语言应该只生成有效的 HTML 输出。下面的[用于访问逻辑的 Use-API](#use-api-for-accessing-logic) 部分介绍了如何从模板调用其他逻辑，可以在需要为这些上下文准备复杂输出时使用它。要将数据从后端发送到前端脚本，请使用组件的逻辑生成 JSON 字符串，并使用简单的 HTL 表达式将其放置在数据属性中。

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

如下面的[自动上下文感知转义](#automatic-context-aware-escaping)部分中所述，HTL 的一个目标是通过自动将上下文感知转义应用于所有表达式来降低引入跨站点脚本 (XSS) 漏洞的风险。HTL 检测 HTML 标记中表达式的上下文，但不分析内联 JavaScript 或 CSS，因此开发人员必须为这些表达式指定确切的上下文。

由于没有在 XSS 漏洞中应用正确的转义结果，因此 HTL 会在尚未声明上下文时删除脚本和样式上下文中的所有表达式的输出。

下面是如何为放置在脚本和样式中的表达式设置上下文的示例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

有关如何控制转义的更多详细信息，请参阅 HTL 规范中的[表达式语言显示上下文](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context)部分。

## HTL 的常规功能 {#general-capabilities-of-htl}

此部分快速介绍 HTML 模板语言的常规功能。

### 用于访问逻辑的 Use-API {#use-api-for-accessing-logic}

利用 HTML 模板语言 (HTL) Java Use-API，HTL 文件可以通过 `data-sly-use` 访问自定义 Java 类中的 helper 方法。这样可以将所有复杂的业务逻辑封装在 Java 代码中，而 HTL 代码只处理直接标记生成。

有关更多详细信息，请参阅 [HTL Java Use-API](java-use-api.md) 文档。

### 自动上下文感知转义 {#automatic-context-aware-escaping}

请仔细研究下面的示例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多数模板语言中，此示例可能会造成跨站点脚本 (XSS) 漏洞，因为即使所有变量都会自动进行 HTML 转义，`href` 属性仍必须经过专门的 URL 转义。这种遗漏是常见的错误之一，因为它容易被遗忘，并且很难自动发现。

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

请注意两个属性如何以不同的方式进行转义，因为 HTL 知道 `href` 和 `src` 属性必须针对 URI 上下文进行转义。 此外，如果 URI 以 `javascript:` 开头，则除非上下文已显式更改为其他内容，否则将完全删除该属性。

有关如何控制转义的更多详细信息，请参阅 HTL 规范中的[表达式语言显示上下文](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context)部分。

### 自动删除空属性 {#automatic-removal-of-empty-attributes}

请仔细研究下面的示例：

```xml
<p class="${properties.class}">some text</p>
```

如果 `class` 属性的值碰巧为空，则 HTML 模板语言将自动从输出中删除整个 `class` 属性。

此功能的实现同样得益于 HTL 对 HTML 语法的理解，因此只有当属性的值不为空时，才能有条件地显示具有动态值的属性。 这十分方便，因为它避免了在属性周围添加条件块，这会使标记无效和不可读。

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

要设置属性，`data-sly-attribute` 语句可能也有用。

## 使用 HTL 的常见模式 {#common-patterns-with-htl}

本节介绍几种常见的场景。它解释了如何使用 HTML 模板语言最好地解决这些情况。

### 加载客户端库 {#loading-client-libraries}

在 HTL 中，通过 AEM 提供的帮助程序模板来加载客户端库，可通过 `data-sly-use` 访问模板。此文件中提供了三个模板，可通过 `data-sly-call` 来调用它们：

* **`css`** - 仅加载引用的客户端库的 CSS 文件。
* **`js`** - 仅加载引用的客户端库的 JavaScript 文件。
* **`all`** - 加载引用的客户端库的所有文件（CSS 和 JavaScript）。

每个帮助程序模板都需要一个 `categories` 选项来引用所需的客户端库。该选项可以是字符串值的数组，也可以是包含逗号分隔值列表的字符串。

以下是两个简短示例。

#### 同时完全加载多个客户端库 {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

#### 在页面的不同部分中引用客户端库 {#referencing-a-client-library-in-different-sections-of-a-page}

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

在此示例中，如果 HTML `head` 和 `body` 元素位于单独的文件中，则必须在每个需要它的文件中加载 `clientlib.html` 模板。

[HTL 规范](specification.md)中有关 template 和 call 语句的部分提供了有关声明和调用此类模板的工作原理的更多详细信息。

### 将数据传递给客户端 {#passing-data-to-the-client}

一般来说，将数据传递给客户端的更方便、更轻松的方式是使用 `data` 属性，对于 HTL 来说更是如此。

下面的示例说明如何将对象序列化为 JSON（在 Java 中也可以）以传递给客户端。然后可以轻松地将其放入 `data` 属性中：

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

从那里，可以轻松地设想客户端 JavaScript 如何访问该属性并再次解析 JSON。此方法将相应的 JavaScript 放入客户端库中，例如：

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用客户端模板 {#working-with-client-side-templates}

可以合法使用[解除特殊上下文的限制](#lifting-limitations-of-special-contexts)部分中说明的技术的一种特殊情况是编写位于 `scrip` 元素内的客户端模板（例如 Handlebars）。在该情况下可以安全使用此技术的原因是，`script` 元素没有像假设的那样包含 JavaScript，而是包含更多的 HTML 元素。下面是有关它的工作原理的示例：

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

 `script` 元素的标记可以包含没有明确上下文的 HTL 块语句，因为 Handlebars 模板内容被隔离在其自己的文件中。此外，此示例显示了如何将服务器端执行的 HTL（例如在 `h2` 元素上）与 Handlebars 之类的客户端执行的模板语言（显示在 `h3` 元素上）混合。

不过，一种更现代的技术是改用 HTML `template` 元素，因为这样做的好处是无需将模板的内容隔离到单独的文件中。

### 解除特殊上下文的限制 {#lifting-limitations-of-special-contexts}

在需要绕过脚本、样式和注释上下文的限制的特殊情况下，可以将它们的内容隔离在单独的 HTL 文件中。 HTL 将其文件中的所有内容解释为标准 HTML 片段，忽略包含它的任何限制上下文。

请参阅下面的[使用客户端模板](#working-with-client-side-templates)部分来获取示例。

>[!CAUTION]
>
>该技术可能引入跨站点脚本（XSS）漏洞。如果使用这种方法，应该仔细研究安全性方面。通常有比依赖这种做法更好的方法来实施同样的目的。
