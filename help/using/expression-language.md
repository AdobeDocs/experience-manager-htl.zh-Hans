---
title: HTL 表达式语言
description: HTML 模板语言使用表达式语言来访问提供 HTML 输出的动态元素的数据结构。
exl-id: 57e3961b-8c84-4d56-a049-597c7b277448
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '1854'
ht-degree: 100%

---

# HTL 表达式语言 {#htl-expression-language}

HTML 模板语言使用表达式语言来访问提供 HTML 输出的动态元素的数据结构。这些表达式通过字符 `${` 和 `}` 进行分隔。为避免格式错误的 HTML，只能在属性值、元素内容或注释中使用表达式。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

表达式可通过在前面追加 `\` 字符来进行转义，例如 `\${test}` 将呈现 `${test}`。

>[!NOTE]
>
>要试用本页面上提供的示例，可以使用称为[读取-求值-打印循环](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl)的实时执行环境。

表达式语法包括[变量](#variables)、[文本](#literals)、[运算符](#operators)和[选项](#options)：

## 变量 {#variables}

变量是存储数据值或对象的容器。变量的名称称为标识符。

无需指定任何内容，HTL 在包括 `global.jsp` 之后即可提供对 JSP 中所有常用对象的访问权限。[全局对象](global-objects.md)页面列出了 HTL 可访问的所有对象。

### 属性访问 {#property-access}

有两种方法可以访问变量的属性，即使用点表示法或使用中括号表示法：

```xml
${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}
```

大多数情况下应该首选较简单的点表示法，中括号表示法应该用于访问包含无效标识符字符的属性，或者动态访问属性。下面两章将详细介绍这两种情况。

访问的属性可以是函数，但不支持传递参数，因此只能访问不需要参数的函数，例如 getter。这是所需的限制，旨在减少嵌入到表达式中的逻辑量。如果需要，[`data-sly-use`](block-statements.md#use) 语句可用于将参数传递给逻辑。

上面的示例还显示，无需在前面追加 `get`，通过将后面的字符变为小写即可访问 Java getter 函数（如 `getTitle()`）。

### 有效的标识符字符 {#valid-identifier-characters}

称为标识符的变量的名称符合某些规则。它们必须以字母（`A`-`Z` 和 `a`-`z`）或下划线 (`_`) 开头，后续字符也可以是数字 (`0`-`9`) 或冒号 (`:`)。不能在标识符中使用 Unicode 字母，例如 `å` 和 `ü`。

考虑到冒号 (`:`) 字符在 AEM 属性名称中很常见，应当强调它通常是有效的标识符字符：

`${properties.jcr:title}`

中括号表示法可用于访问包含无效标识符字符（如以下示例中的空格字符）的属性：

`${properties['my property']}`

### 动态访问成员 {#accessing-members-dynamically}

```xml
${properties[myVar]}
```

### 空值的宽松处理 {#permissive-handling-of-null-values}

```xml
${currentPage.lastModified.time.toString}
```

## 文本 {#literals}

文本是一种用于表示固定值的表示法。

### 布尔型 {#boolean}

布尔型表示一个逻辑实体，可以具有两个值：`true` 和 `false`。

`${true} ${false}`

### 数字 {#numbers}

只有一种数字类型：正整数。虽然其他数字格式（如浮点）在变量中受支持，但不能表示为文本。

`${42}`

### 字符串 {#strings}

字符串表示文本数据，并且可以是用单引号或双引号括起来的：

`${'foo'} ${"bar"}`

除了普通字符外，还可以使用以下特殊字符：

* `\\` 反斜杠字符
* `\'` 单引号（或撇号）
* `\"` 双引号
* `\t` 制表符
* `\n` 换行符
* `\r` 回车符
* `\f` 换页符
* `\b` 退格符
* `\uXXXX` 由四个十六进制数字 XXXX 指定的 Unicode 字符。\
   一些有用的 unicode 转义序列是：

   * `\u0022` 表示 `"`
   * `\u0027` 表示 `'`

对于上面未列出的字符，在前面添加反斜杠字符将显示错误。

以下是如何使用字符串转义的一些示例：

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

它们将生成以下输出，因为 HTL 将应用特定于上下文的转义：

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### 数组 {#arrays}

数组是一组有序值，可使用名称和索引来引用它。可混合使用其元素类型。

```xml
${[1,2,3,4]}
${myArray[2]}
```

数组可用于从模板提供值列表。

```xml
<ul data-sly-list="${[1,2,3,4]}">
  <li>${item}</li>
</ul>
```

## 运算符 {#operators}

### 逻辑运算符 {#logical-operators}

这些运算符通常与布尔值一起使用，但是，就像在 JavaScript 中一样，它们实际上会返回指定操作数之一的值，因此当与非布尔值一起使用时，它们可能会返回非布尔值。

如果一个值可以转换为 `true`，则该值就是所谓的真值。如果一个值可以转换为 `false`，则该值就是所谓的假值。可以转换为 `false` 的值是未定义的变量、空值、数字 0 和空字符串。

#### 逻辑非 {#logical-not}

`${!myVar}` 会在其单个操作数可转换为 `true` 时返回 `false`；否则，会返回 `true`。

例如，这可用于反转测试条件，例如仅在没有子页面时才显示元素：

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### 逻辑与 {#logical-and}

`${varOne && varTwo}` 会在 `varOne` 是假值时返回它；否则，会返回 `varTwo`。

此运算符可用于同时测试两个条件，例如验证两个属性是否存在：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

逻辑与运算符也可用于有条件地显示 HTML 属性，因为 HTL 会删除其动态设置的值计算为 false 或空字符串的属性。因此在以下示例中，仅当 `logic.showClass` 为真值，并且 `logic.className` 存在且非空时，才显示 `class` 属性：

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### 逻辑或 {#logical-or}

`${varOne || varTwo}` 会在 `varOne` 是真值时返回它；否则，会返回 `varTwo`。

此运算符可用于测试两个条件之一是否适用，例如验证是否至少存在一个属性：

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

由于逻辑或运算符返回第一个是真值的变量，因此它也可以非常方便地用于提供回退值。

它也可用于有条件地显示 HTML 属性，因为 HTL 会删除其通过表达式设置的值计算为 false 或空字符串的属性。因此以下示例会在 **`properties.jcr:`** 标题存在且非空时显示该标题，否则会在 **`properties.jcr:description`** 描述存在且非空时显示该描述，否则会显示消息“未提供标题或描述”：

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### 条件（三元）运算符 {#conditional-ternary-operator}

如果 `varCondition` 是真值，则 `${varCondition ? varOne : varTwo}` 会返回 `varOne`；否则，会返回 `varTwo`。

此运算符通常可用于在表达式中定义条件，例如根据页面状态显示不同的消息：

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

>[!TIP]
>
>由于标识符中也允许使用冒号字符，因此最好用空格分隔三元运算符，以向解析程序清楚阐明：

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### 比较运算符 {#comparison-operators}

相等和不等运算符仅支持相同类型的操作数。当类型不匹配时，会显示错误。

* 当字符串具有相同的字符序列时，它们是相等的。
* 当数字具有相同的值时，它们是相等的
* 如果两个布尔值都为 `true` 或都为 `false`，则它们是相等的。
* 空变量或未定义的变量与本身相等并且彼此相等。

如果 `varOne` 和 `varTwo` 相等，则 `${varOne == varTwo}` 会返回 `true`。

如果 `varOne` 和 `varTwo` 不相等，则 `${varOne != varTwo}` 会返回 `true`。

关系运算符仅支持数字操作数。对于所有其他类型，会显示错误。

如果 `varOne` 大于 `varTwo`，则 `${varOne > varTwo}` 会返回 `true`。

如果 `varOne` 小于 `varTwo`，则 `${varOne < varTwo}` 会返回 `true`。

如果 `varOne` 大于或等于 `varTwo`，则 `${varOne >= varTwo}` 会返回 `true`。

如果 `varOne` 小于或等于 `varTwo`，则 `${varOne <= varTwo}` 会返回 `true`。

### 分组括号 {#grouping-parentheses}

分组运算符 `()` 控制表达式中计算的优先级。

`${varOne && (varTwo || varThree)}`

## 选项 {#options}

表达式选项可作用于表达式并对其进行修改，或者在与块语句结合使用时充当参数。

`@` 之后的所有内容是一个选项：

```xml
${myVar @ optOne}
```

选项可以具有值，这可能是变量或文本：

```xml
${myVar @ optOne=someVar}
${myVar @ optOne='bar'}
${myVar @ optOne=10}
${myVar @ optOne=true}
```

以逗号分隔多个选项：

```xml
${myVar @ optOne, optTwo=bar}
```

仅包含选项的参数表达式也是可以的：

```xml
${@ optOne, optTwo=bar}
```

### 字符串格式 {#string-formatting}

将枚举占位符 {*n*} 替换为相应变量的选项：

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

## URL 操作 {#url-manipulation}

提供了一组新的 url 操作。

请参阅以下有关它们的用法的示例：

将 html 扩展添加到路径中。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

将 html 扩展和选择器添加到路径中。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

将 html 扩展和片段 (#value) 添加到路径中。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

`@extension` 适用于所有场景，检查是否添加扩展。

```xml
${ link @ extension = 'html' }
```

### 数字/日期格式 {#number-date-formatting}

HTL 允许数字和日期的本机格式，而无需编写自定义代码。这还支持时区和区域设置。

以下示例显示先指定格式，然后指定需要设置格式的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>有关您可以使用的格式的完整详细信息，请参阅 [HTL 规范](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 国际化 {#internationalization}

使用当前[词典](https://docs.adobe.com/content/help/zh-Hans/experience-manager-65/developing/components/internationalization/i18n-translator.html)，将字符串翻译为当前&#x200B;*源* 的语言（参见下文）。如果找不到翻译，则使用原始字符串。

```xml
${'Page' @ i18n}
```

提示选项可用于为翻译工具提供注释，指定使用文本的上下文：

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

语言的默认源是 `resource`，这意味着文本翻译为与内容相同的语言。这可更改为 `user`，这意味着从浏览器区域设置或登录用户的区域设置中获取语言：

```xml
${'Page' @ i18n, source='user'}
```

提供显式区域设置会覆盖源设置：

```xml
${'Page' @ i18n, locale='en-US'}
```

要将变量嵌入到翻译后的字符串中，可以使用 format 选项：

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### 数组联接 {#array-join}

默认情况下，当将数组显示为文本时，HTL 将显示逗号分隔值（无空格）。

使用 join 选项指定不同的分隔符：

```xml
${['one', 'two'] @ join='; '}
```

### 显示上下文 {#display-context}

HTL 表达式的显示上下文是指它在 HTML 页面结构中的位置。例如，如果表达式出现在呈现之后会生成文本节点的位置，则称其在 `text` 上下文中。如果在属性值中找到它，则称其在 `attribute` 上下文中，依此类推。

除了脚本 (JS) 和样式 (CSS) 上下文之外，HTL 会自动检测表达式的上下文并适当地进行转义，以预防 XSS 安全问题。对于脚本和 CSS，必须显式设置所需的上下文行为。此外，还可以在需要覆盖自动行为的任何其他情况下显式设置上下文行为。

这里我们有三种不同上下文中的三个变量：

* `properties.link`（`uri` 上下文）
* `properties.title`（`attribute` 上下文）
* `properties.text`（`text` 上下文）

HTL 将根据各自上下文的安全要求，以不同方式对其中每一个进行转义。在正常情况下不需要显式上下文设置，例如：

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

要安全地输出标记（即，表达式本身的计算结果为 HTML），请使用 `html` 上下文：

```xml
<div>${properties.richText @ context='html'}</div>
```

必须为样式上下文设置显式上下文：

```xml
<span style="color: ${properties.color @ context='styleToken'};">...</span>
```

必须为脚本上下文设置显式上下文：

```xml
<span onclick="${properties.function @ context='scriptToken'}();">...</span>
```

也可以关闭转义和 XSS 保护：

```xml
<div>${myScript @ context='unsafe'}</div>
```

### 上下文设置 {#context-settings}

| 上下文 | 何时使用 | 作用 |
|--- |--- |--- |
| `text` | 元素中的内容的默认值 | 对所有 HTML 特殊字符进行编码。 |
| `html` | 安全输出标记 | 筛选 HTML 以符合 AntiSamy 策略规则，删除与规则不匹配的内容。 |
| `attribute` | 属性值的默认值 | 对所有 HTML 特殊字符进行编码。 |
| `uri` | 显示 href 和 src 属性值的链接和路径默认值 | 验证 URI 以作为 href 或 src 属性值写入，如果验证失败，则不输出任何内容。 |
| `number` | 显示数字 | 验证 URI 以包含一个整数，如果验证失败，则输出零。 |
| `attributeName` | 设置属性名称时 data-sly-attribute 的默认值 | 验证属性名称，如果验证失败，则不输出任何内容。 |
| `elementName` | data-sly-element 的默认值 | 验证元素名称，如果验证失败，则不输出任何内容。 |
| `scriptToken` | 对于 JS 标识符、文本数字或文本字符串 | 验证 JavaScript 令牌，如果验证失败，则不输出任何内容。 |
| `scriptString` | 在 JS 字符串内 | 对会脱离字符串的字符进行编码。 |
| `scriptComment` | 在 JS 注释内 | 验证 JavaScript 注释，如果验证失败，则不输出任何内容。 |
| `styleToken` | 对于 CSS 标识符、数字、维度、字符串、十六进制颜色或函数。 | 验证 CSS 令牌，如果验证失败，则不输出任何内容。 |
| `styleString` | 在 CSS 字符串内 | 对会脱离字符串的字符进行编码。 |
| `styleComment` | 在 CSS 注释内 | 验证 CSS 注释，如果验证失败，则不输出任何内容。 |
| `unsafe` | 仅当以上任何一项都不起作用时 | 完全禁用转义和 XSS 保护。 |
