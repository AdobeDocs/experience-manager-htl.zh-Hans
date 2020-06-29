---
title: HTL 表达式语言
description: HTML模板语言使用表达式语言访问提供HTML输出动态元素的数据结构。
translation-type: tm+mt
source-git-commit: ee712ef61018b5e05ea052484e2a9a6b12e6c5c8
workflow-type: tm+mt
source-wordcount: '1848'
ht-degree: 0%

---


# HTL 表达式语言 {#htl-expression-language}

HTML模板语言使用表达式语言访问提供HTML输出动态元素的数据结构。 这些表达式以字符和 `${` 分隔 `}`。 为避免格式错误的HTML,表达式只能用于属性值、元素内容或注释中。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

表达式可以通过预挂字符进 `\` 行转义，例如 `\${test}` 将呈现 `${test}`。

>[!NOTE]
>
>要试用本页中提供的示例，可以使用名为“读取评 [估打印循环](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl) ”的实时执行环境。

表达式语法包 [括变量](#variables)、 [文本](#literals)、运 [算符](#operators) 和 [选](#options)项：

## 变量 {#variables}

变量是存储数据值或对象的容器。 变量的名称称为标识符。

无需指定任何内容，HTL即可提供对JSP中包含后通常可用的所有对象的访问 `global.jsp`。 “全 [局对象](global-objects.md) ”页提供HTL提供访问的所有对象的列表。

### 属性访问 {#property-access}

有两种方法可以访问变量的属性，使用点记号或括号记号：

```xml
${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}
```

大多数情况下最好使用更简单的点记号，括号记号应用于访问包含无效标识符字符的属性，或动态访问属性。 以下两章将提供这两个案例的详细信息。

被访问的属性可以是函数，但是不支持传递参数，因此只能访问不期望参数的函数，如getter。 这是一个理想的限制，旨在减少嵌入到表达式中的逻辑量。 如果需要， [`data-sly-use`](block-statements.md#use) 可以使用语句将参数传递给逻辑。

上例中还显示了Java getter函数(如 `getTitle()`，可以访问它，而 `get`不预先设置，并降低后面字符的大小写。

### 有效标识符字符 {#valid-identifier-characters}

变量名称（称为标识符）符合某些规则。 它们必须用字母(`A`-和`Z` - `a`)或下划线()开始，后续字符也可以是数字(-)或冒号(`z``_``0``9``:`)。 Unicode字母(如 `å` 和) `ü` 不能用于标识符。

由于冒号()`:`字符在AEM属性名称中是通用的，因此应强调它是一个方便有效的标识符字符：

`${properties.jcr:title}`

括号记号可用于访问包含无效标识符字符的属性，如以下示例中的空格字符：

`${properties['my property']}`

### 动态访问成员 {#accessing-members-dynamically}

```xml
${properties[myVar]}
```

### Null值的允许处理 {#permissive-handling-of-null-values}

```xml
${currentPage.lastModified.time.toString}
```

## 文字 {#literals}

文本是表示固定值的记号。

### 布尔型 {#boolean}

Boolean表示逻辑实体，可以有两个值： `true`和 `false`。

`${true} ${false}`

### 数字 {#numbers}

只有一个数字类型： 正整数。 其他数字格式（如浮点）在变量中受支持，但不能表示为文字。

`${42}`

### 字符串 {#strings}

字符串表示文本多次，可以是单个或引号：

`${'foo'} ${"bar"}`

除了普通字符外，还可以使用以下特殊字符：

* `\\` 反斜杠字符
* `\'` 单引号（或撇号）
* `\"` 多次报价
* `\t` 选项卡
* `\n` 新行
* `\r` 回车
* `\f` 表单源
* `\b` Backspace
* `\uXXXX` 由四个十六进制数字XXXX指定的Unicode字符。\
   一些有用的Unicode转义序列包括：

   * `\u0022`对象`"`
   * `\u0027`对象`'`

对于以上未列出的字符，在反斜杠字符前面将显示错误。

以下是如何使用字符串转义的一些示例：

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

这将导致以下输出，因为HTL将应用特定于上下文的转义：

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### 阵列 {#arrays}

数组是一组有序值，可以用名称和索引引用。 其元素类型可以混合。

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

## 运营商 {#operators}

### 逻辑运算符 {#logical-operators}

这些运算符通常与布尔值一起使用，但是，与JavaScript中一样，它们实际上返回指定操作数之一的值，因此当与非布尔值一起使用时，它们可能返回非布尔值。

如果某个值可以转换 `true`为，则该值称为truthy。 如果某个值可以转换 `false`为，则该值称为falsy。 可转换为的值 `false` 是未定义的变量、空值、数字零和空字符串。

#### 逻辑非 {#logical-not}

`${!myVar}` 返回 `false` 其单个操作数是否可转换为 `true`; 否则，它将返回 `true`。

例如，这可用于反转测试条件，如仅在没有子页面时显示元素：

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### 逻辑与 {#logical-and}

`${varOne && varTwo}` 如 `varOne` 果是虚假的，则返回； 否则，它将返回 `varTwo`。

此运算符可用于同时测试两个条件，如验证是否存在两个属性：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

逻辑AND运算符还可用于有条件地显示HTML属性，因为HTL删除了值动态设置为false或空字符串的属性。 因此，在以下示例中， `class` 只有属性是真的且 `logic.showClass` 存在且不 `logic.className` 为空时，才显示该属性：

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### 逻辑或 {#logical-or}

`${varOne || varTwo}` 如 `varOne` 果真实，则返回； 否则，它将返回 `varTwo`。

此运算符可用于测试是否适用以下两种条件之一，如验证是否存在至少一个属性：

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

由于逻辑OR运算符返回第一个变量是真的，因此它也可以非常方便地用于提供回退值。

有条件地显示HTML属性，因为HTL删除了由表达式设置的、结果为false或空字符串的属性。 因此，以下示例将显 **`properties.jcr:`** 示标题（如果存在且不为空），否则它将返回显示 **`properties.jcr:description`** （如果存在且不为空），否则它将显示消息“未提供标题或说明”:

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### 条件（三元）运算符 {#conditional-ternary-operator}

`${varCondition ? varOne : varTwo}` 如 `varOne` 果真 `varCondition` 实，则返回； 否则它将返回 `varTwo`。

此运算符通常可用于定义表达式中的条件，如根据页面状态显示不同的消息：

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

>[!TIP]
>
>由于冒号字符也允许在标识符中使用，因此最好将三元运算符与空格分开，以便为分析器提供清晰性：

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### 比较运算符 {#comparison-operators}

等式和不等式运算符只支持相同类型的操作数。 类型不匹配时，将显示错误。

* 如果字符串具有相同的字符序列，则字符串相等。
* 数值相同时，数值相等
* 如果两者均为，则布尔 `true` 值相等，或两者 `false`均为。
* null或未定义的变量彼此相等。

`${varOne == varTwo}` 在 `true` 和 `varOne` 相等 `varTwo` 时返回。

`${varOne != varTwo}` 返 `true` 回 `varOne` (如 `varTwo` 果和不相等)。

关系运算符仅支持数字操作数。 对于所有其他类型，将显示错误。

`${varOne > varTwo}` 返 `true` 回 `varOne` 大于 `varTwo`。

`${varOne < varTwo}` 返 `true` 回 `varOne` 小于的 `varTwo`。

`${varOne >= varTwo}` 如 `true` 果 `varOne` 大于或等于，则返回 `varTwo`。

`${varOne <= varTwo}` 返 `true` 回 `varOne` 值(如果小于或等于 `varTwo`)

### 分组括号 {#grouping-parentheses}

分组运算符控 `()` 制表达式中评估的优先级。

`${varOne && (varTwo || varThree)}`

## 选项 {#options}

表达式选项可以对表达式执行操作并修改它，或在与块语句结合使用时用作参数。

之后的一 `@` 切都是一个选项：

```xml
${myVar @ optOne}
```

选项可以有一个值，它可以是变量或文本：

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

也可以使用仅包含选项的参数表达式:

```xml
${@ optOne, optTwo=bar}
```

### 字符串格式 {#string-formatting}

用相应的变量替换枚举的占&#x200B;*位符*{n}的选项：

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

## URL处理 {#url-manipulation}

有一组新的url操作可用。

请参阅以下示例的用法：

将html扩展添加到路径。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

向路径添加html扩展名和选择器。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

将html扩展名和片段(#value)添加到路径。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

在所 `@extension` 有情况下均可使用，检查是否添加扩展。

```xml
${ link @ extension = 'html' }
```

### 数字／日期格式 {#number-date-formatting}

HTL允许数字和日期的本机格式化，无需编写自定义代码。 这还支持时区和区域设置。

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
>有关可使用格式的完整详细信息，请参 [阅HTL规范](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 国际化 {#internationalization}

使用当前词典将字符串转换 *为当前* 源的语言(请参 [见下)](https://docs.adobe.com/content/help/en/experience-manager-65/developing/components/internationalization/i18n-translator.html)。 如果未找到转换，则使用原始字符串。

```xml
${'Page' @ i18n}
```

提示选项可用于为翻译人员提供注释，指定使用文本的上下文：

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

该语言的默认来源 `resource`是，这意味着文本将翻译为与内容相同的语言。 这可以更改 `user`为，这意味着语言是从浏览器区域设置或登录用户的区域设置中获取的：

```xml
${'Page' @ i18n, source='user'}
```

提供显式区域设置将覆盖源设置：

```xml
${'Page' @ i18n, locale='en-US'}
```

要将变量嵌入到已翻译的字符串中，可使用格式选项：

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### 阵列连接 {#array-join}

默认情况下，当将数组显示为文本时，HTL将显示以逗号分隔的值（无间距）。

使用连接选项指定不同的分隔符：

```xml
${['one', 'two'] @ join='; '}
```

### 显示上下文 {#display-context}

HTL表达式的显示上下文指其在HTML页面结构中的位置。 例如，如果表达式显示在渲染后会生成文本节点的位置，则表示它位于上下文 `text` 中。 如果在属性的值中找到它，则它被称为在上 `attribute` 下文中，依此类推。

除脚本(JS)和样式(CSS)上下文外，HTL将自动检测表达式的上下文并相应地进行转义，以防止XSS安全问题。 对于脚本和CSS，必须显式设置所需的上下文行为。 此外，在需要覆盖自动行为的任何其他情况下也可以显式设置上下文行为。

这里，我们在三个不同的上下文中有三个变量：

* `properties.link` (上 `uri` 下文)
* `properties.title` (上`attribute` 下文)
* `properties.text` (上`text` 下文)

HTL将根据各自上下文的安全要求以不同方式逃避这些漏洞。 正常情况下无需显式上下文设置，如下：

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

要安全输出标记(即，表达式自身的计算结果为HTML)，请使用 `html` 上下文：

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

还可以关闭逃生和XSS保护：

```xml
<div>${myScript @ context='unsafe'}</div>
```

### 上下文设置 {#context-settings}

| 上下文 | 何时使用 | 它的用途 |
|--- |--- |--- |
| `text` | 元素内容的默认值 | 对所有HTML特殊字符进行编码。 |
| `html` | 安全输出标记 | 过滤器HTML以符合AntiSamy策略规则，删除与规则不匹配的内容。 |
| `attribute` | 属性值的默认值 | 对所有HTML特殊字符进行编码。 |
| `uri` | 显示链接和路径href和src属性值的默认值 | 验证URI以写入href或src属性值，如果验证失败则不输出任何内容。 |
| `number` | 显示数字 | 验证包含整数的URI，如果验证失败，则输出零。 |
| `attributeName` | 在设置属性名称时数据密钥属性的默认值 | 验证属性名称，如果验证失败则不输出任何内容。 |
| `elementName` | 数据密钥元素的默认值 | 验证元素名称，如果验证失败则不输出任何内容。 |
| `scriptToken` | 对于JS标识符、文本编号或文本字符串 | 验证JavaScript令牌，如果验证失败则不输出任何内容。 |
| `scriptString` | 在JS字符串中 | 对字符串中的字符进行编码。 |
| `scriptComment` | 在JS注释中 | 验证JavaScript注释，如果验证失败，则不输出任何注释。 |
| `styleToken` | 用于CSS标识符、数字、尺寸、字符串、十六进制颜色或函数。 | 验证CSS令牌，如果验证失败则不输出任何内容。 |
| `styleString` | 在CSS字符串中 | 对字符串中的字符进行编码。 |
| `styleComment` | 在CSS注释中 | 验证CSS注释，如果验证失败则不输出任何内容。 |
| `unsafe` | 只有上述任何一项都没有 | 完全禁用逃逸和XSS保护。 |
