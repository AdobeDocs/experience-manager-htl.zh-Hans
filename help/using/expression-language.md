---
title: HTL表达式语言
seo-title: HTL表达式语言
description: HTML模板语言使用表达式语言访问提供HTML输出动态元素的数据结构。
seo-description: HTML模板语言使用表达式语言访问提供HTML输出动态元素的数据结构。
uuid: 38b4a259-03b5-4847-91c6-e203777070
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 9ba37ca0-f318-48b0-a791-a944 a72502 eded
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 271c355ae56e16e309853b02b8ef09f2ff971a2e

---


# HTL表达式语言 {#htl-expression-language}

HTML模板语言使用表达式语言访问提供HTML输出动态元素的数据结构。这些表达式由字符 `${` 和 `}`字符分隔。为避免HTML格式错误，表达式只能用于属性值、元素内容中或注释中。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

例如，可以通过 **`\`** 字符前置表达式来转义 **`\${test}`** 表达式 **`${test}`**。

>[!NOTE]
>
>要试用此页面上提供的示例，可使用称为 [Read Eval Print循环](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl) 的实时执行环境。

表达式语法包括 [变量](#variables)、 [文本](#literals)、 [运算符](#operators) 和 [选项](#options)：

## 变量 {#variables}

变量是存储数据值或对象的容器。变量名称称为标识符。

除了指定任何内容之外，HTL还提供对JSP中常用的所有对象的访问权限，之后包括 `global.jsp`这些对象。[全局对象](global-objects.md) 页面提供了HTL访问的所有对象的列表。

### 属性访问 {#property-access}

有两种方法可访问变量属性、点记号或括号记号：

`${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}`

大多数情况下都应使用更简单的点记号，而且应使用括号记号访问包含无效标识符字符的属性或动态访问属性。以下两章将提供这两个案例的详细信息。

访问的属性可以是函数，但是传递参数不受支持，因此只有不希望参数被访问的函数才可以访问(getter)。这是一个期望的限制，它旨在减少嵌入到表达式中的逻辑量。如果需要， [`data-sly-use`](block-statements.md#use) 该语句可用于将参数传递到逻辑。

上述示例中还显示了Java getter函数(如 `getTitle()`可以访问)无需预先审核该 **`get`**函数，并通过降低后面的字符大小写来进行访问。

### 有效的缩进字符 {#valid-indentifier-characters}

变量名称(称为标识符)符合某些规则。它们必须以字母(-**`A`****`Z`** 和 **`a`**-)**`z`**开头，下划线(**`_`**)和后续字符也可以为数字(-**`0`****`9`**)或冒号(**`:`**)。Unicode字母等不 **`å`****`ü`** 能在标识符中使用。

鉴于AEM属性名称中的冒号(**：**)字符很常见，因此它是有效的标识符字符：

`${properties.jcr:title}`

括号记号可用于访问包含无效标识符字符的属性，如以下示例中的空格字符：

`${properties['my property']}`

### 动态访问成员 {#accessing-members-dynamically}

<!-- 

Comment Type: draft

<p>TODO: add description</p>

 -->

```xml
${properties[myVar]}
```

### 空值的权限处理 {#permissive-handling-of-null-values}

<!-- 

Comment Type: draft

<p>TODO: add description</p>

 -->

```xml
${currentPage.lastModified.time.toString}
```

## 文字 {#literals}

文本是表示固定值的记号。

### 布尔型 {#boolean}

Boolean表示一个逻辑实体，并且可以具有两个值： **`true`****`false`**和.

`${true} ${false}`

### 数字 {#numbers}

只有一种数字类型：正整数。其他数字格式(如浮点)在变量中受支持，但不能以文本形式表示。

`${42}`

### 字符串 {#strings}

它们代表文本数据，可以是单引号或双引号：

`${'foo'} ${"bar"}`

除了普通字符之外，还可以使用特殊字符：

* **`\\`** 反斜杠字符
* **`\'`** 单引号(或撇号)
* **`\"`** 双引号
* **`\t`** 表格
* **`\n`** 新行
* **`\r`** 回车符
* **`\f`** 表单源
* **`\b`** Backspace
* `\uXXXX` 由四个十六进制数字XXXX指定的Unicode字符。\
   一些有用的Unicode转义序列有：

   * **\ u0022** for **”**
   * **\ u0027** for **'**

对于以上未列出的字符，反斜杠购物车之前将显示一个错误。

下面是一些如何使用字符串转义的示例：

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

这将导致以下输出，因为HTL将应用上下文特定的转义：

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### 数组 {#arrays}

数组是一组有序的值，可使用名称和索引引用。元素类型可以混合起来。

<!-- 

Comment Type: draft

<p>TODO: add description</p>

 -->

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

这些运算符通常与Boolean值一起使用，但像在JavaScript中一样，它们实际返回某个指定操作数的值，因此当使用非Boolean值时，它们可能返回非Boolean值。

如果可以将某个值转换为 **`true`**，则该值是所谓的粗体。如果可将值转换为， **`false`**则该值称为falcy。可转换为的值 **`false`** 包括：未定义的变量、null值、数字零和空字符串。

#### 逻辑NOT {#logical-not}

**`${!myVar}`** 返回， **`false`** 如果其单个操作数可转换 `true`为；否则，退货 **`true`**。

这可用于反转测试条件，如仅在没有子页面时显示元素：

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### 逻辑和 {#logical-and}

**`${varOne && varTwo}`** 退货 `varOne` ；否则返回 **Vartwo**。

此运算符可用于同时测试两个条件，如验证存在两个属性：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

逻辑AND运算符还可用于条件显示HTML属性，因为HTL会删除动态设置为false的值或空字符串的属性。因此，在下面的示例中，仅 **`class`** 当fey和If **`logic.showClass`****`logic.className`** exists存在且不是空时才显示该属性：

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### 逻辑OR {#logical-or}

**`${varOne || varTwo}`** 返回 **VarOne，** 如果它是真实的；否则返回 **Vartwo**。

此运算符可用于测试以下两种情况之一是否适用，如验证至少存在一个属性：

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

因为逻辑OR运算符返回的第一个变量非常真实，因此它也可以很方便地用于提供回退值。

有条件地显示HTML属性，因为HTL会删除由求值为false的表达式设置的值或空字符串。因此以下示例将显示****`properties.jcr:`标题(如果它存在且不为空)，否则它会回退到dislays **`properties.jcr:description`** (如果它存在且不为空)，否则它将显示消息“no title or description provided”：

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### 条件(termerary)运算符 {#conditional-ternary-operator}

**`${varCondition ? varOne : varTwo}`** 退货 **`varOne`****`varCondition`** ； **`varTwo`**否则返回。

此运算符通常可用于定义表达式中的条件，如根据页面的状态显示其他消息：

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

重要注意事项是，由于还可以在标识符中使用冒号字符，最好将三元运算符与空白分离，以使分析器清楚地显示：

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### 比较运算符 {#comparison-operators}

平等和不等于运算符仅支持类型相同的操作数。当类型不匹配时，会显示错误。

* 字符串在字符序列相同时是相等的。
* 如果数字相同，则数字相等
* 如果两者均为 **`true`** 或两者都相等，则布尔值相等 **`false`**。

* 空或未定义的变量彼此相等，或者彼此相等。

**`${varOne == varTwo}`** 返回 **`true`** 是否 **`varOne`****`varTwo`** 相等。

**`${varOne != varTwo}`** 返回 **`true`****`varOne`** 是否 **`varTwo`** 相等。

关系运算符仅支持数字操作数。对于所有其他类型，都会显示一个错误。

**`${varOne > varTwo}`** 退货 **`true`****`varOne`****`varTwo`**。

**`${varOne < varTwo}`** 返回 **`true`****`varOne`****`varTwo`**。

**`${varOne >= varTwo}`** 返回 **`true`****`varOne`** 大于或等于 **`varTwo`**。

**`${varOne <= varTwo}`** 返回 **`true`****`varOne`****`varTwo`**。

### 分组括号 {#grouping-parentheses}

分组运算符 **`(`****`)`** 控制表达式中评估的优先级。

`${varOne && (varTwo || varThree)}`

## 选项 {#options}

<!-- 

Comment Type: draft

<p>TODO: review text below.</p>

 -->

表达式选项可以在表达式上操作并修改它，或者在与块语句结合使用时用作参数。

下面是一个 **`@`** 选项：

```xml
${myVar @ optOne}
```

选项可以有一个值，它可能是变量或文本：

```xml
${myVar @ optOne=someVar}
${myVar @ optOne='bar'}
${myVar @ optOne=10}
${myVar @ optOne=true}
```

多个选项使用逗号分隔：

```xml
${myVar @ optOne, optTwo=bar}
```

也可能仅包含包含选项的参数表达式：

```xml
${@ optOne, optTwo=bar}
```

### String Formatting {#string-formatting}

替换枚举占位符{*n*}的选项，其中包含相应的变量：

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

### 国际化 {#internationalization}

使用当前词典将字符串翻译为当前 *源的* 语言(见 [下文](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/i18n-translator))。如果未找到翻译，则使用原始字符串。

```xml
${'Page' @ i18n}
```

提示选项可用于为翻译人员提供评论，用于指定使用文本的上下文：

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

该语言的默认来源为“资源”，这意味着文本被翻译为与内容相同的语言。这可以更改为“用户”，这意味着语言从浏览器区域设置或已登录用户的区域设置中获取：

```xml
${'Page' @ i18n, source='user'}
```

提供显式区域设置会覆盖源设置：

```xml
${'Page' @ i18n, locale='en-US'}
```

要将变量嵌入到译文字符串中，可以使用格式选项：

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### 数组加入 {#array-join}

默认情况下，将数组显示为文本时，HTL将显示逗号分隔值(不含间距)。

使用“加入”选项指定不同的分隔符：

```xml
${['one', 'two'] @ join='; '}
```

### 显示上下文 {#display-context}

HTL表达式的显示上下文引用HTML页面结构中的位置。例如，如果显示的表达式在渲染后会生成文本节点，则表示该表达式位于 **`text`** 上下文中。如果在属性的值内找到它，则表示它位于 **`attribute`** 上下文中，依此类推。

除了脚本(JS)和样式(CSS)上下文之外，HTL还会自动检测表达式上下文并相应地进行转义，以防止XSS安全问题。对于脚本和CSS，必须显式设置所需的上下文行为。此外，上下文行为还可以在需要覆盖自动行为的任何其他情况下显式设置。

这里，我们在三个不同的上下文中有三个变量： **`properties.link`** ( `uri` context)， **`properties.title`** (**`attribute`** context)和 **`properties.text`**(**`text`** context)。HTL将根据各自上下文的安全要求，转义每种类型的HTL。在通常情况下，不需要显式上下文设置(如此设置)：

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

要安全输出标记(即表达式本身的评估结果为HTML)，请使用 `html` 上下文：

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

也可以关闭转义和XSS保护：

```xml
<div>${myScript @ context='unsafe'}</div>
```

### 上下文设置 {#context-settings}

| 上下文 | 何时使用 | 它的用途 |
|--- |--- |--- |
| 文本 | 元素内内容的默认值 | 对所有HTML特殊字符进行编码。 |
| html | 安全输出标记 | 过滤器HTML以满足AntiSamy策略规则，删除与规则不符的内容。 |
| attribute | 属性值默认值 | 对所有HTML特殊字符进行编码。 |
| uri | 显示链接和路径默认值href和src属性值 | Validates用作href或src属性值的验证URI，如果验证失败，则输出任何内容。 |
| 数字 | 要显示数字 | 用于包含整数的Valides URI，如果验证失败，则输出零。 |
| AttributeName | 设置属性名称时的data-sly-属性默认值 | 验证属性名称，如果验证失败，则输出不输出。 |
| ElementName | data-sly-element默认值 | 验证元素名称，如果验证失败，则输出不输出。 |
| scriptToken | 对于JS标识符、字面编号或文本字符串 | 验证JavaScript令牌，如果验证失败，则输出不输出。 |
| scriptString | 在JS字符串内 | 对要跳出字符串的字符进行编码。 |
| ScriptComment | 在JS注释内 | 验证JavaScript注释，如果验证失败，则输出不输出。 |
| styleToken | 对于CSS标识符、数字、尺寸、字符串、十六进制颜色或函数。 | 验证CSS令牌，如果验证失败，则输出不输出。 |
| styleString | 在CSS字符串内 | 对要跳出字符串的字符进行编码。 |
| StyleComment | 在CSS注释内 | 验证CSS注释，如果验证失败，则输出不输出。 |
| 不安全 | 仅当以上任何一项均不起作用时 | 完全禁用退出和XSS保护。 |

