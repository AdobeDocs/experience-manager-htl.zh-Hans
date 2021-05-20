---
title: HTL 表达式语言
description: HTML模板语言使用表达式语言访问提供HTML输出动态元素的数据结构。
exl-id: 57e3961b-8c84-4d56-a049-597c7b277448
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '1854'
ht-degree: 0%

---

# HTL 表达式语言 {#htl-expression-language}

HTML模板语言使用表达式语言访问提供HTML输出动态元素的数据结构。 这些表达式以字符`${`和`}`分隔。 为避免格式不正确的HTML，表达式只能用于属性值、元素内容或注释中。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

表达式可以通过预挂`\`字符进行转义，例如，将呈现`\${test}`的`${test}`。

>[!NOTE]
>
>要试用本页中提供的示例，可使用名为[读取评估打印循环](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl)的实时执行环境。

表达式语法包括[变量](#variables)、[文字](#literals)、[运算符](#operators)和[选项](#options):

## 变量 {#variables}

变量是存储数据值或对象的容器。 变量的名称称为标识符。

无需指定任何内容，HTL即可访问JSP中在包含`global.jsp`之后通常可用的所有对象。 [全局对象](global-objects.md)页提供了HTL提供访问的所有对象的列表。

### 属性访问{#property-access}

有两种方法可访问变量的属性，即使用点表示法或使用括号表示法：

```xml
${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}
```

大多数情况下，最好使用更简单的点表示法，并且应使用括号表示法来访问包含无效标识符字符的属性，或动态访问属性。 以下两章将详细介绍这两种情况。

访问的属性可以是函数，但是不支持传递参数，因此只能访问不期望参数的函数，如getter。 这是一个理想的限制，其目的是减少嵌入到表达式中的逻辑量。 如果需要，可以使用[`data-sly-use`](block-statements.md#use)语句将参数传递到逻辑。

上例中还显示，Java getter函数（如`getTitle()`）可以访问，而无需预置`get`，并通过降低以下字符的大小写。

### 有效的标识符字符{#valid-identifier-characters}

变量的名称（称为标识符）符合特定规则。 它们必须以字母（`A`-`Z`和`a`-`z`）或下划线(`_`)开头，后续字符也可以是数字(`0`-`9`)或冒号(`:`)。 标识符中不能使用Unicode字母，如`å`和`ü`。

鉴于冒号(`:`)字符在AEM属性名称中是通用的，因此应当强调，它是有效的标识符字符：

`${properties.jcr:title}`

括号符号可用于访问包含无效标识符字符的属性，如以下示例中的空格字符：

`${properties['my property']}`

### 动态访问成员{#accessing-members-dynamically}

```xml
${properties[myVar]}
```

### 对空值{#permissive-handling-of-null-values}的许可处理

```xml
${currentPage.lastModified.time.toString}
```

## 文字 {#literals}

文本是表示固定值的符号。

### 布尔型 {#boolean}

布尔值表示逻辑实体，可以具有两个值：`true`和`false`。

`${true} ${false}`

### 数字 {#numbers}

只有一个数字类型：正整数。 而其他数字格式（如浮点）则支持在变量中，但不能表示为文字。

`${42}`

### 字符串 {#strings}

字符串表示文本数据，可以是单引号或双引号：

`${'foo'} ${"bar"}`

除了普通字符外，还可使用以下特殊字符：

* `\\` 反斜线字符
* `\'` 单引号（或撇号）
* `\"` 双引号
* `\t` 选项卡
* `\n` 新行
* `\r` 回车符
* `\f` 表单馈送
* `\b` Backspace
* `\uXXXX` 由四个十六进制数字XXXX指定的Unicode字符。\
   一些有用的Unicode转义序列包括：

   * `\u0022`对象`"`
   * `\u0027`对象`'`

对于上面未列出的字符，反斜线字符之前将显示错误。

以下是如何使用字符串转义的一些示例：

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

这会导致以下输出，因为HTL将应用特定于上下文的转义：

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### 阵列 {#arrays}

数组是一组有序的值，可以用名称和索引引用。 其元素类型可以混合使用。

```xml
${[1,2,3,4]}
${myArray[2]}
```

数组可用于提供模板中的值列表。

```xml
<ul data-sly-list="${[1,2,3,4]}">
  <li>${item}</li>
</ul>
```

## 运算符 {#operators}

### 逻辑运算符{#logical-operators}

这些运算符通常与布尔值一起使用，但与在JavaScript中一样，它们实际上会返回指定操作数之一的值，因此当与非布尔值一起使用时，它们可能会返回非布尔值。

如果某个值可以转换为`true`，则该值称为truthy。 如果某个值可以转换为`false`，则该值称为falsy。 可转换为`false`的值是未定义的变量、空值、数字零和空字符串。

#### 逻辑NOT {#logical-not}

`${!myVar}` 返回 `false` 其单个操作数是否可转换为 `true`;否则，将返回 `true`。

例如，这可用于反转测试条件，例如，仅当没有子页面时才显示元素：

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### 逻辑和{#logical-and}

`${varOne && varTwo}` 若 `varOne` 为falsy，则返回；否则，将返回 `varTwo`。

此运算符可用于同时测试两个条件，如验证是否存在两个属性：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

逻辑AND运算符还可用于有条件地显示HTML属性，因为HTL会删除其值动态设置为false或空字符串的属性。 因此，在以下示例中，仅当`logic.showClass`为truthy且`logic.className`存在且不为空时，才显示`class`属性：

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### 逻辑OR {#logical-or}

`${varOne || varTwo}` 如 `varOne` 果为truthy，则返回；否则，将返回 `varTwo`。

此运算符可用于测试是否应用以下两个条件之一，例如验证是否存在至少一个属性：

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

由于逻辑OR运算符会返回第一个真实的变量，因此它也可以非常方便地用于提供回退值。

它还可用于有条件地显示HTML属性，因为HTL会删除由计算为false或空字符串的表达式设置的值的属性。 因此，以下示例将显示&#x200B;**`properties.jcr:`**&#x200B;标题（如果存在且不为空），否则返回显示&#x200B;**`properties.jcr:description`**（如果存在且不为空），否则将显示消息“未提供标题或描述”：

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### 条件（三元）运算符{#conditional-ternary-operator}

`${varCondition ? varOne : varTwo}` 如 `varOne` 果 `varCondition` 为truthy;否则，返回 `varTwo`。

此运算符通常用于定义表达式中的条件，例如根据页面状态显示不同的消息：

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

>[!TIP]
>
>由于标识符中也允许使用冒号字符，因此最好将三元运算符与空格分开，以便为解析器提供清晰性：

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### 比较运算符{#comparison-operators}

等式和不等式运算符仅支持相同类型的操作数。 类型不匹配时，将显示错误。

* 如果字符串具有相同的字符序列，则字符串相等。
* 具有相同值时，数字相等
* 如果两者均为`true`或两者均为`false`，则布尔值相等。
* 空变量或未定义变量彼此相等。

`${varOne == varTwo}` 如果 `true` 和 `varOne` 相 `varTwo` 等，则返回。

`${varOne != varTwo}` 如果 `true` 和 `varOne` 不 `varTwo` 等于，则返回。

关系运算符仅支持数字操作数。 对于所有其他类型，将显示错误。

`${varOne > varTwo}` 如果 `true` 大 `varOne` 于，则返 `varTwo`回。

`${varOne < varTwo}` 如果 `true` 小 `varOne` 于，则返 `varTwo`回。

`${varOne >= varTwo}` 如 `true` 果 `varOne` 大于或等于，则返 `varTwo`回。

`${varOne <= varTwo}` 如 `true` 果 `varOne` 小于或等于，则返 `varTwo`回。

### 对括号{#grouping-parentheses}进行分组

分组运算符`()`控制表达式中求值的优先级。

`${varOne && (varTwo || varThree)}`

## 选项 {#options}

表达式选项可以对表达式执行操作并对其进行修改，或在与块语句结合使用时用作参数。

`@`之后的所有内容都是一个选项：

```xml
${myVar @ optOne}
```

选项可以具有一个值，该值可以是变量或文字：

```xml
${myVar @ optOne=someVar}
${myVar @ optOne='bar'}
${myVar @ optOne=10}
${myVar @ optOne=true}
```

多个选项之间用逗号分隔：

```xml
${myVar @ optOne, optTwo=bar}
```

也可以使用仅包含选项的参数表达式：

```xml
${@ optOne, optTwo=bar}
```

### 字符串格式{#string-formatting}

用于将枚举占位符{*n*}替换为相应变量的选项：

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

## URL操作{#url-manipulation}

提供了一组新的URL操作。

请参阅以下示例了解其用法：

将html扩展添加到路径。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

将html扩展和选择器添加到路径。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

将html扩展和片段(#value)添加到路径中。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

`@extension`在所有情况下均可使用，并检查是否添加扩展。

```xml
${ link @ extension = 'html' }
```

### 数字/日期格式{#number-date-formatting}

HTL允许使用数字和日期的本机格式，而无需编写自定义代码。 这还支持时区和区域设置。

以下示例显示，首先指定格式，然后指定需要格式的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>有关可使用的格式的完整详细信息，请参阅[HTL-specification](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 国际化 {#internationalization}

使用当前[词典](https://docs.adobe.com/content/help/en/experience-manager-65/developing/components/internationalization/i18n-translator.html)将字符串转换为当前&#x200B;*source*&#x200B;的语言（请参阅下文）。 如果未找到翻译，则使用原始字符串。

```xml
${'Page' @ i18n}
```

提示选项可用于为翻译人员提供注释，以指定在其中使用文本的上下文：

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

该语言的默认来源为`resource`，这意味着文本将被翻译为与内容相同的语言。 这可以更改为`user`，这意味着语言是从浏览器区域设置或登录用户的区域设置中获取的：

```xml
${'Page' @ i18n, source='user'}
```

提供显式区域设置会覆盖源设置：

```xml
${'Page' @ i18n, locale='en-US'}
```

要将变量嵌入到已翻译的字符串中，可使用格式选项：

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### 阵列连接{#array-join}

默认情况下，将数组显示为文本时，HTL将显示逗号分隔值（不带间距）。

使用连接选项指定不同的分隔符：

```xml
${['one', 'two'] @ join='; '}
```

### 显示上下文{#display-context}

HTL表达式的显示上下文是指其在HTML页面结构中的位置。 例如，如果表达式在呈现后会生成一个文本节点，则该表达式会显示在`text`上下文中。 如果在属性的值中找到它，则表示它位于`attribute`上下文中，依此类推。

除脚本(JS)和样式(CSS)上下文外，HTL将自动检测表达式的上下文并相应地转义它们，以防止出现XSS安全问题。 对于脚本和CSS，必须明确设置所需的上下文行为。 此外，在需要覆盖自动行为的任何其他情况下，也可以显式设置上下文行为。

下面我们在三个不同的上下文中有三个变量：

* `properties.link` (上 `uri` 下文)
* `properties.title` (上`attribute` 下文)
* `properties.text` (上`text` 下文)

HTL将根据各自上下文的安全要求以不同方式对其中每个变量进行转义。 在正常情况下，如此一种情况下，无需显式上下文设置：

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

为了安全地输出标记（即，表达式本身的计算结果为HTML），将使用`html`上下文：

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

转义和XSS保护也可以关闭：

```xml
<div>${myScript @ context='unsafe'}</div>
```

### 上下文设置{#context-settings}

| 上下文 | 何时使用 | 它的作用 |
|--- |--- |--- |
| `text` | 元素内的内容默认 | 对所有HTML特殊字符进行编码。 |
| `html` | 安全输出标记 | 过滤HTML以符合AntiSamy策略规则，从而删除与规则不匹配的内容。 |
| `attribute` | 属性值的默认值 | 对所有HTML特殊字符进行编码。 |
| `uri` | 显示链接和路径：href和src属性值默认值 | 验证URI以写入href或src属性值，如果验证失败，则不会输出任何内容。 |
| `number` | 显示数字 | 验证包含整数的URI时，如果验证失败，则输出零。 |
| `attributeName` | 在设置属性名称时，data-sly-attribute的默认值 | 验证属性名称，如果验证失败，则不会输出任何内容。 |
| `elementName` | data-sly-element的默认值 | 验证元素名称，如果验证失败，则不会输出任何内容。 |
| `scriptToken` | 对于JS标识符、文字数字或文字字符串 | 验证JavaScript令牌，如果验证失败，则不会输出任何内容。 |
| `scriptString` | 在JS字符串内 | 会对字符串中断开的字符进行编码。 |
| `scriptComment` | 在JS注释内 | 验证JavaScript注释时，如果验证失败，则不会输出任何内容。 |
| `styleToken` | 用于CSS标识符、数字、维度、字符串、十六进制颜色或函数。 | 验证CSS令牌，如果验证失败，则不会输出任何内容。 |
| `styleString` | 在CSS字符串内 | 会对字符串中断开的字符进行编码。 |
| `styleComment` | 在CSS注释内 | 验证CSS注释，如果验证失败，则不会输出任何内容。 |
| `unsafe` | 只有上述情况都不能执行 | 完全禁用转义和XSS保护。 |
