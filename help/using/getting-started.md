---
title: HTL 快速入门
description: AEM支持的HTL取代JSP作为AEM中HTML的首选和推荐的服务器端模板系统。
translation-type: tm+mt
source-git-commit: f7e46aaac2a4b51d7fa131ef46692ba6be58d878
workflow-type: tm+mt
source-wordcount: '2471'
ht-degree: 0%

---


# HTL 快速入门 {#getting-started-with-htl}

Adobe Experience Manager(AEM)支持的HTML模板语言(HTL)是AEM中HTML的首选和推荐的服务器端模板系统。 它取代了先前版本的AEM中使用的JSP(JavaServer Pages)。

>[!NOTE]
>
>要运行本页上提供的大多数示例，可使用名为[读取评估打印循环](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl)的实时执行环境。

## HTL over JSP {#htl-over-jsp}

建议新的AEM项目使用HTML模板语言，因为与JSP相比，它优惠了多种优势。 但是，对于现有项目，仅在估计未来几年的工作少于维护现有JSP时，迁移才有意义。

但是，转向HTL并非一无是处的选择，因为用HTL编写的组件与用JSP或ESP编写的组件兼容。 这意味着现有项目可以无问题地对新组件使用HTL，同时保留现有组件的JSP。

即使在同一组件中，HTL文件也可以与JSP和ESP一起使用。 以下示例在&#x200B;**行1**&#x200B;中说明如何从JSP文件包含HTL文件，以及在&#x200B;**行2**&#x200B;中说明如何从HTL文件包含JSP文件：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常见问题{#frequently-asked-questions}

在开始使用HTML模板语言之前，让我们先开始回答与JSP与HTL主题相关的一些问题。

**HTL是否有JSP所没有的限制？** -与JSP相比，HTL实际上没有任何限制，因为JSP可以做的事情也应该通过HTL实现。但是，HTL在设计上要比JSP更严格，在单个JSP文件中可以实现的，可能需要将它们分成Java类或JavaScript文件，才能在HTL中实现。 但是，这通常是为了确保逻辑和标记之间的关注点的良好分离。

**HTL是否支持JSP标签库？** -否，但如加载客户端库 [部分所](getting-started.md#loading-client-libraries) 示，模板和调 [用语句](block-statements.md#template-call) 优惠类似的模式。

**HTL功能能否在AEM项目上扩展？** -不，他们不能。HTL具有用于重用逻辑的强大扩展机制- [Use-API](getting-started.md#use-api-for-accessing-logic)和标记（[template &amp; call](block-statements.md#template-call)语句），可用于模块化项目代码。

**与JSP相比，HTL有哪些主要优势？** -安全性和项目效率是主要优势，详见概 [述](overview.md)。

**JSP最终会消失吗？** -在当前日期，没有这些计划。

## HTL{#fundamental-concepts-of-htl}的基本概念

HTML模板语言使用表达式语言将内容片段插入渲染的标记中，HTML5数据属性用于定义标记块（如条件或迭代）上的语句。 当HTL编译为Java Servlet时，表达式和HTL数据属性都在服务器端进行评估，结果HTML中不会显示任何内容。

### 块和表达式{#blocks-and-expressions}

以下是第一个示例，它可以像在&#x200B;**`template.html`**&#x200B;文件中一样包含：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以区分两种不同的语法：

* **[块语句](block-statements.md)** -要有条件地显示  **&lt;h1>** 元素，则 [`data-sly-test`](block-statements.md#test) 使用HTML5数据属性。HTL提供多个此类属性，这些属性允许将行为附加到任何HTML元素，所有属性都带有`data-sly`前缀。

* **[表达式语](expression-language.md)** - HTL表达式由字符和 `${` 字符分 `}`隔。在运行时，将评估这些表达式，并将其值注入传出的HTML流中。

上面链接的两页提供了可用语法功能的详细列表。

### SLY元素{#the-sly-element}

HTL的一个核心概念是优惠重用现有HTML元素来定义块语句的可能性，这避免了插入附加分隔符来定义语句开始和结束的位置。 将静态HTML转换为有效动态模板的标记的这一不显眼的注释优惠了不破坏HTML代码有效性的好处，因此，即使作为静态文件，也能正常显示。

但是，有时可能在必须插入块语句的确切位置不存在现有元素。 对于这种情况，可以插入将自动从输出中移除的特殊SLY元素，同时执行附加的块语句并相应地显示其内容。

下面的示例：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

将输出类似于以下HTML的内容，但仅当同时定义了&#x200B;**`jcr:title`**&#x200B;和&#x200B;**`jcr:description`**&#x200B;属性，并且两者均为空时：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

要记住的一点是，当没有现有元素可以用块语句添加注释时，只使用SLY元素，因为SLY元素阻止语言提供的值在使静态HTML变为动态时不改变它。

例如，如果上一个示例已包装在DIV元素中，则添加的SLY元素将是谩骂：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

并且DIV元素可能已添加条件注释：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL注释{#htl-comments}

以下示例显示在&#x200B;**行1**&#x200B;上的HTL注释，以及在&#x200B;**行2**&#x200B;上的HTML注释：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL注释是HTML注释，其附加的类似JavaScript的语法。 处理器将完全忽略整个HTL注释以及其中的任何内容，并从输出中删除。

但是，将传递标准HTML注释的内容，并评估注释中的表达式。

HTML注释不能包含HTL注释，反之亦然。

### 特殊上下文{#special-contexts}

为了能够最好地利用HTL，很重要的是要了解它基于HTML语法的后果。

### 元素和属性名称{#element-and-attribute-names}

表达式只能放置到HTML文本或属性值中，但不能放在元素名称或属性名称中，否则将不再是有效的HTML。 为了动态设置元素名称，[`data-sly-element`](block-statements.md#element)语句可用于所需元素，并动态设置属性名称，甚至一次设置多个属性，也可以使用[`data-sly-attribute`](block-statements.md#attribute)语句。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 上下文语无块语句{#contexts-without-block-statements}

由于HTL使用数据属性定义块语句，因此无法在以下上下文中定义此类块语句，并且只能在这里使用表达式:

* HTML注释
* 脚本元素
* 样式元素

其原因在于这些上下文的内容是文本而不是HTML，包含的HTML元素将被视为简单的字符数据。 因此，如果没有真实的HTML元素，也不能执行&#x200B;**`data-sly`**&#x200B;属性。

这听起来可能是一个很大的限制，但是它是理想的限制，因为不应滥用HTML模板语言生成非HTML的输出。 下面的[访问逻辑的Use-API](getting-started.md#use-api-for-accessing-logic)部分介绍了如何从模板调用其他逻辑，如果需要为这些上下文准备复杂输出，可使用该模板。 例如，将数据从后端发送到前端脚本的一种简单方法是，使组件的逻辑生成JSON字符串，然后使用简单的HTL表达式将该字符串放在数据属性中。

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

### 需要显式上下文符{#explicit-contexts-required}

正如下面的[自动上下文感知转义](getting-started.md#automatic-context-aware-escaping)部分所述，HTL的一个目标是通过自动将上下文感知转义应用于所有表达式，降低引入跨站点脚本(XSS)漏洞的风险。 虽然HTL可以自动检测置于HTML标记内的表达式的上下文，但它不分析内联JavaScript或CSS的语法，因此需要开发人员明确指定哪些上下文应用于此类表达式。

由于未在XSS漏洞中应用正确的转义结果，因此，HTL会删除未声明上下文时脚本和样式上下文中所有表达式的输出。

下面是一个如何为放置在脚本和样式中的表达式设置上下文的示例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

有关如何控制转义的详细信息，请参阅[表达式语言显示上下文](expression-language.md#display-context)部分。

### 特殊上下文的提升限制{#lifting-limitations-of-special-contexts}

在需要绕过脚本、样式和注释上下文限制的特殊情况下，可以将其内容分离到单独的HTL文件中。 HTL会将位于其自己文件中的所有内容解释为普通HTML片段，而忽略可能包含该片段的限制上下文。

有关示例，请参见[使用客户端模板](getting-started.md#working-with-client-side-templates)一节。

>[!CAUTION]
>
>该技术可能引入跨站点脚本(XSS)漏洞，如果使用该漏洞，应仔细研究其安全性。 通常有比依靠这种做法更好的方法来实施同样的事。

## HTL {#general-capabilities-of-htl}的一般功能

本节快速介绍HTML模板语言的一般功能。

### 用于访问逻辑{#use-api-for-accessing-logic}的Use-API

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

在服务器端执行的位于其旁边的`logic.js` JavaScript文件下：

```javascript
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由于HTML模板语言不允许在标记中混合代码，因此它将优惠Use-API扩展机制，以便轻松地从模板中执行代码。

以上示例使用服务器端执行的JavaScript将标题缩短为10个字符，但它也可能使用Java代码通过提供完全限定的Java类名来实现这一操作。 通常，业务逻辑应该是使用Java构建的，但是当组件需要从Java API提供的内容进行一些视图特定的更改时，使用一些服务器端执行的JavaScript可以很方便。

以下各节中有关此的更多信息：

* [`data-sly-use`语句](block-statements.md#use)中的部分介绍了可以使用该语句完成的所有操作。
* [Use-API页面](use-api.md)提供一些信息，帮助您选择使用Java或JavaScript编写逻辑。
* 要详细说明如何编写逻辑，[JavaScript Use-API](use-api-javascript.md)和[Java Use-API](use-api-java.md)页面应会有所帮助。

### 自动上下文感知转义{#automatic-context-aware-escaping}

请考虑以下示例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多数模板语言中，此示例可能会创建跨站点脚本(XSS)漏洞，因为即使所有变量都自动以HTML形式转义，`href`属性也必须仍具体为URL转义。 这种疏漏是最常见的错误之一，因为它很容易被遗忘，而且很难以自动方式发现。

为此，HTML模板语言会相应地将每个变量自动转义到放置该变量的上下文。 由于HTL了解HTML的语法，这是可能的。

假定文件为`logic.js`:

```javascript
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

注意两个属性是如何以不同方式转义的，因为HTL知道`href`和`src`属性必须为URI上下文转义。 此外，如果URI以&#x200B;**`javascript:`**&#x200B;开头，则属性将被完全删除，除非上下文被显式更改为其他内容。

有关如何控制转义的详细信息，请参阅[表达式语言显示上下文](expression-language.md#display-context)部分。

### 自动删除空属性{#automatic-removal-of-empty-attributes}

请考虑以下示例：

```xml
<p class="${properties.class}">some text</p>
```

如果`class`属性的值碰巧为空，HTML模板语言将自动从输出中删除整个`class`属性。

同样，这也是可能的，因为HTL了解HTML语法，因此只有属性值不为空时，才能有条件地显示具有动态值的属性。 这非常方便，因为它避免在属性周围添加条件块，这会使标记无效和不可读。

此外，放置在表达式中的变量类型也很重要：

* **String:**
   * **不为空：** 将字符串设置为属性值。
   * **空：完** 全删除属性。

* **Number:** 将值设置为属性值。

* **布尔型:**
   * **true：显** 示无值属性（作为布尔HTML属性）
   * **false：完** 全删除属性。

下面是一个布尔表达式如何允许控制布尔HTML属性的示例：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

对于设置属性，[`data-sly-attribute`](block-statements.md#attribute)语句可能也很有用。

## HTL {#common-patterns-with-htl}的常见模式

本节介绍一些常见情形，以及如何使用HTML模板语言最好地解决这些情况。

### 正在加载客户端库{#loading-client-libraries}

在HTL中，客户端库通过AEM提供的帮助模板加载，可通过[`data-sly-use`](block-statements.md#use)访问该模板。 此文件中有三个模板，可通过[`data-sly-call`](block-statements.md#template-call)调用：

* **`css`** -仅加载引用的客户端库的CSS文件。
* **`js`** -仅加载引用的客户端库的JavaScript文件。
* **`all`** -加载引用的客户端库的所有文件（CSS和JavaScript）。

每个帮助程序模板都需要一个&#x200B;**`categories`**&#x200B;选项来引用所需的客户端库。 该选项可以是字符串值的数组，也可以是包含逗号分隔值列表的字符串。

以下是两个简短示例：

### 一次完全加载多个客户端库{#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 在页面{#referencing-a-client-library-in-different-sections-of-a-page}的不同部分引用客户端库

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

在上面的第二个示例中，如果将HTML **`head`**&#x200B;和&#x200B;**`body`**&#x200B;元素放置到不同的文件中，则必须在需要它的每个文件中加载&#x200B;**`clientlib.html`**&#x200B;模板。

[template &amp; call](block-statements.md#template-call)语句中的部分提供了有关声明和调用此类模板的工作方式的详细信息。

### 将数据传递到客户端{#passing-data-to-the-client}

一般而言，将数据传递给客户端的最佳且最优雅的方式，是使用数据属性。

以下示例说明如何使用逻辑（也可以用Java编写）非常方便地将要传递到客户端的对象序列化为JSON，然后将该对象非常容易地放入数据属性中：

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

从这里，可以很容易地想象客户端JavaScript如何访问该属性并重新分析JSON。 例如，这将是放入客户端库的相应JavaScript:

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用客户端模板{#working-with-client-side-templates}

在[特殊上下文的提升限制](getting-started.md#lifting-limitations-of-special-contexts)一节中所述的技术可以合法使用的一个特殊情况是编写位于&#x200B;**脚本**&#x200B;元素中的客户端模板（如Handlebars）。 在这种情况下，此技术可以安全使用的原因是，**script**&#x200B;元素不会像假定的那样包含JavaScript，而是包含更多HTML元素。 下面是一个如何实现该功能的示例：

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

如上所示，将包含在&#x200B;**`script`**&#x200B;元素中的标记可以包含HTL块语句，而表达式无需提供显式上下文，因为Handlebars模板的内容已隔离在其自己的文件中。 此外，此示例还说明如何将服务器端执行的HTL（如&#x200B;**`h2`**&#x200B;元素）与客户端执行的模板语言（如Handlebars）混合在一起（如&#x200B;**`h3`**&#x200B;元素中所示）。

但是，更现代的技术是改用HTML **`template`**&#x200B;元素，因为这样做的好处是不需要将模板的内容分离为单独的文件。

**阅读下一篇文章：**

* [表达式语](expression-language.md) -详细了解在HTL表达式中可以完成哪些工作。
* [块语句](block-statements.md) -发现HTL中可用的所有块语句，以及如何使用它们。
