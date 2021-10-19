---
title: HTL 块语句
description: HTML 模板语言 (HTL) 块语句是直接添加到现有 HTML 的自定义 data 属性。
exl-id: a517dcef-ab7a-4d4c-a1a9-2e57aad034f7
source-git-commit: 89b9e89254f341e74f1a5a7b99735d2e69c8a91e
workflow-type: ht
source-wordcount: '1555'
ht-degree: 100%

---

# HTL 块语句 {#htl-block-statements}

HTML 模板语言 (HTL) 块语句是直接添加到现有 HTML 的自定义 `data` 属性。这允许对原型静态 HTML 页面进行轻松且不造成干扰的注释，将其转换为正常使用的动态模板，而不会破坏 HTML 代码的有效性。

## 块概述 {#overview}

HTL 块插件由 HTML 元素上设置的 `data-sly-*` 属性定义。元素可以具有结束标记或自结束。属性可以具有值（可以是静态字符串或表达式），也可以只是布尔属性（没有值）。

```xml
<tag data-sly-BLOCK></tag>                                 <!--/* A block is simply consists in a data-sly attribute set on an element. */-->
<tag data-sly-BLOCK/>                                      <!--/* Empty elements (without a closing tag) should have the trailing slash. */-->
<tag data-sly-BLOCK="string value"/>                       <!--/* A block statement usually has a value passed, but not necessarily. */-->
<tag data-sly-BLOCK="${expression}"/>                      <!--/* The passed value can be an expression as well. */-->
<tag data-sly-BLOCK="${@ myArg='foo'}"/>                   <!--/* Or a parametric expression with arguments. */-->
<tag data-sly-BLOCKONE="value" data-sly-BLOCKTWO="value"/> <!--/* Several block statements can be set on a same element. */-->
```

将从生成的标记中删除所有已计算的 `data-sly-*` 属性。

### 标识符 {#identifiers}

块语句还可以后跟标识符：

```xml
<tag data-sly-BLOCK.IDENTIFIER="value"></tag>
```

块语句可以通过多种方式使用标识符，以下是一些示例：

```xml
<!--/* Example of statements that use the identifier to set a variable with their result: */-->
<div data-sly-use.navigation="MyNavigation">${navigation.title}</div>
<div data-sly-test.isEditMode="${wcmmode.edit}">${isEditMode}</div>
<div data-sly-list.child="${currentPage.listChildren}">${child.properties.jcr:title}</div>
<div data-sly-template.nav>Hello World</div>

<!--/* The attribute statement uses the identifier to know to which attribute it should apply it's value: */-->
<div data-sly-attribute.title="${properties.jcr:title}"></div> <!--/* This will create a title attribute */-->
```

顶级标识符不区分大小写（因为它们可以通过不区分大小写的 HTML 属性进行设置），但它们的所有属性都区分大小写。

## 可用的块语句 {#available-block-statements}

有许多可用的块语句。在同一元素上使用时，以下优先级列表定义了块语句的计算方式：

1. `data-sly-template`
1. `data-sly-set`, `data-sly-test`, `data-sly-use`
1. `data-sly-call`
1. `data-sly-text`
1. `data-sly-element`, `data-sly-include`, `data-sly-resource`
1. `data-sly-unwrap`
1. `data-sly-list`, `data-sly-repeat`
1. `data-sly-attribute`

当两个块语句具有相同的优先级时，它们的计算顺序是从左到右。

### use {#use}

`data-sly-use` 初始化一个帮助程序对象（在 JavaScript 或 Java 中定义）并通过变量公开它。

初始化一个 JavaScript 对象，其中源文件与模板位于同一目录中。请注意，必须使用文件名：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化一个 Java 类，其中源文件与模板位于同一目录中。请注意，必须使用类名，而非文件名：

```xml
<div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化一个 Java 类，该类作为 OSGi 捆绑包的一部分进行安装。请注意，必须使用其完全限定类名：

```xml
<div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

可以使用选项将参数传递给初始化。通常，此功能只应由本身位于 `data-sly-template` 块中的 HTL 代码使用：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化另一个 HTL 模板，然后可以使用 `data-sly-call` 调用该模板：

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>有关 Use-API 的更多信息，请参阅：
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)


#### 对资源使用 data-sly-use {#data-sly-use-with-resources}

这允许使用 `data-sly-use` 直接在 HTL 中获取资源，并且无需编写代码即可获取它。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

>[!TIP]
>
>另请参阅[路径并非总是必需的](#path-not-required)部分。

### unwrap {#unwrap}

`data-sly-unwrap` 从生成的标记中删除主机元素，而保留其内容。这允许排除 HTL 表示逻辑中需要但在实际输出中不需要的元素。

但是，应谨慎使用此语句。一般来说，最好使 HTL 标记尽可能接近预期的输出标记。换句话说，在添加 HTL 块语句时，尽量尝试仅对现有 HTML 进行注释，而不引入新元素。

例如，此

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

将生成

```xml
<p>Hello World</p>
```

而此

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

将生成

```xml
Hello World
```

还可以有条件地解包元素：

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

### set {#set}

`data-sly-set` 定义具有预定义值的新标识符。

```xml
<span data-sly-set.profile="${user.profile}">Hello, ${profile.firstName} ${profile.lastName}!</span>
<a class="profile-link" href="${profile.url}">Edit your profile</a>
```

### text {#text}

`data-sly-text` 将其主机元素的内容替换为指定文本。

例如，此

```xml
<p>${properties.jcr:description}</p>
```

等同于

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

两者都会将 `jcr:description` 的值显示为段落文本。第二种方法的优点是允许对 HTML 进行不显眼的注释，同时保留原始设计者的静态占位符内容。

### attribute {#attribute}

`data-sly-attribute` 向主机元素中添加属性。

例如，此

```xml
<div title="${properties.jcr:title}"></div>
```

等同于

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

两者都会将 `title` 属性设置为 `jcr:title` 的值。第二种方法的优点是允许对 HTML 进行不显眼的注释，同时保留原始设计者的静态占位符内容。

从左到右解析属性，并且属性的最右边实例（文本或通过 `data-sly-attribute` 进行定义）优先于相同属性左边定义的任何实例（按字面或通过 `data-sly-attribute` 进行定义）。

请注意，将在最终标记中删除其值计算为空字符串的属性（`literal` 或通过 `data-sly-attribute` 进行设置）。此规则的一个例外是将保留设置为文本空字符串的文本属性。例如，

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

生成

```xml
<div></div>
```

但

```xml
<div class="" data-sly-attribute.id=""></div>
```

生成

```xml
<div class=""></div>
```

要设置多个属性，请传递一个包含与属性及其值对应的键值对的映射对象。例如，假定

```xml
attrMap = {
    title: "myTitle",
    class: "myClass",
    id: "myId"
}
```

那么

```xml
<div data-sly-attribute="${attrMap}"></div>
```

生成

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

### element {#element}

`data-sly-element` 替换主机元素的元素名称。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

将 `h1` 替换为 `titleLevel` 的值。

出于安全原因，`data-sly-element` 仅接受以下元素名称：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub
sup table tbody td tfoot th thead time tr u var wbr
```

要设置其他元素，必须禁用 XSS 安全性 (`@context='unsafe'`)。

### test {#test}

`data-sly-test` 有条件地删除主机元素及其内容。值 `false` 会删除元素；值 `true` 会保留元素。

例如，`p` 元素及其内容仅在 `isShown` 为 `true` 时才会呈现：

```xml
<p data-sly-test="${isShown}">text</p>
```

测试的结果可以分配给以后可以使用的变量。这通常用于构造“if else”逻辑，因为没有明确的 else 语句：

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

变量在设置之后在 HTL 文件中具有全局作用域。

以下是一些有关比较值的示例：

```xml
<div data-sly-test="${properties.jcr:title == 'test'}">TEST</div>
<div data-sly-test="${properties.jcr:title != 'test'}">NOT TEST</div>

<div data-sly-test="${properties['jcr:title'].length > 3}">Title is longer than 3</div>
<div data-sly-test="${properties['jcr:title'].length >= 0}">Title is longer or equal to zero </div>

<div data-sly-test="${properties['jcr:title'].length > aemComponent.MAX_LENGTH}">
    Title is longer than the limit of ${aemComponent.MAX_LENGTH}
</div>
```

### repeat {#repeat}

利用 `data-sly-repeat`，您可以根据指定的列表多次重复一个元素。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

这与 `data-sly-list` 的工作方式相同，只是您不需要容器元素。

以下示例显示您还可以引用属性的&#x200B;*项目*：

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

### list {#list}

`data-sly-list` 为提供的对象中的每个可枚举属性重复主机元素的内容。

以下是一个简单循环：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

以下默认变量在列表范围内可用：

* `item`：迭代中的当前项目。
* `itemList`：具有以下属性的对象：
* `index`：从 0 开始的计数器 (`0..length-1`)。
* `count`：从 1 开始的计数器 (`1..length`)。
* `first`：如果当前项目是第一个项目，则为 `true`。
* `middle`：如果当前项目既不是第一个项目也不是最后一个项目，则为 `true`。
* `last`：如果当前项目是最后一个项目，则为 `true`。
* `odd`：如果 `index` 为奇数，则为 `true`。
* `even`：如果 `index` 为偶数，则为 `true`。

在 `data-sly-list` 语句上定义标识符允许您重命名 `itemList` 和 `item` 变量。`item` 将成为 `<variable>`，`itemList` 将成为 `<variable>List`。

```xml
<dl data-sly-list.child="${currentPage.listChildren}">
    <dt>index: ${childList.index}</dt>
    <dd>value: ${child.title}</dd>
</dl>
```

您还可以动态访问属性：

```xml
<dl data-sly-list.child="${myObj}">
    <dt>key: ${child}</dt>
    <dd>value: ${myObj[child]}</dd>
</dl>
```

### resource {#resource}

`data-sly-resource` 包括通过 sling 解析和呈现过程呈现指示的资源的结果。

简单的资源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

#### 路径并非总是必需的 {#path-not-required}

请注意，如果您已经拥有资源，则无需将路径与 `data-sly-resource` 结合使用。如果您已经拥有资源，则可以直接使用它。

例如，下面的内容是正确的。

```xml
<sly data-sly-resource="${resource.path @ decorationTagName='div'}"></sly>
```

不过，下面的内容也是完全可接受的。

```xml
<sly data-sly-resource="${resource @ decorationTagName='div'}"></sly>
```

由于以下原因，建议尽可能直接使用资源。

* 如果您已经拥有资源，则使用路径重新解析是额外的、不必要的工作。
* 在您已经拥有资源时使用路径可能会引入意外结果，因为 Sling 资源可能已封装或可能是合成的，而不是在给定路径中提供。

#### 选项 {#resource-options}

选项允许许多其他变体：

处理资源的路径：

```xml
<article data-sly-resource="${ @ path='path/to/resource'}"></article>
<article data-sly-resource="${'resource' @ prependPath='my/path'}"></article>
<article data-sly-resource="${'my/path' @ appendPath='resource'}"></article>
```

添加（或替换）选择器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors='selector'}"></article>
```

添加、替换或删除多个选择器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors=['s1', 's2']}"></article>
```

向现有选择器中添加一个选择器：

```xml
<article data-sly-resource="${'path/to/resource' @ addSelectors='selector'}"></article>
```

从现有选择器中删除一些选择器：

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors='selector1'}"></article>
```

删除所有选择器：

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors}"></article>
```

覆盖资源的资源类型：

```xml
<article data-sly-resource="${'path/to/resource' @ resourceType='my/resource/type'}"></article>
```

更改 WCM 模式：

```xml
<article data-sly-resource="${'path/to/resource' @ wcmmode='disabled'}"></article>
```

默认情况下，禁用 AEM 修饰标记，decorationTagName 选项允许恢复它们，cssClassName 允许向该元素添加类。

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM 可提供清晰而简单的逻辑来控制用于封装所包含元素的修饰标记。有关详细信息，请参阅开发组件文档中的[修饰标记](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/developing/full-stack/components-templates/decoration-tag.html?lang=zh-Hans)。

### include {#include}

`data-sly-include` 在由其相应的模板引擎处理时，将主机元素的内容替换为指示的 HTML 模板文件（HTL、JSP、ESP 等）所生成的标记。所含文件的呈现上下文将不包含当前 HTL 上下文（包含文件的上下文）；因此，要包含 HTL 文件，必须在包含的文件中重复当前的 `data-sly-use`（在这种情况下，通常最好使用 `data-sly-template` 和 `data-sly-call`）

简单的包含：

```xml
<section data-sly-include="path/to/template.html"></section>
```

可以通过相同的方式包含 JSP：

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

选项允许您处理文件的路径：

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

您还可以更改 WCM 模式：

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

### Request-attributes {#request-attributes}

在 `data-sly-include` 和 `data-sly-resource` 中，您可以传递 `requestAttributes` 以在接收 HTL 脚本中使用它们。

这允许您将参数正确传递到脚本或组件中。

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings"
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Settings 类的 Java 代码，使用 Map 传入 requestAttributes：

```xml
public class Settings extends WCMUsePojo {

  // used to pass is requestAttributes to data-sly-resource
  public Map<String, Object> settings = new HashMap<String, Object>();

  @Override
  public void activate() throws Exception {
    settings.put("layout", "flex");
  }
}
```

例如，通过 Sling-Model，您可以使用指定的 `requestAttributes` 的值。

在此示例中，通过 Map 从 Use 类注入 layout：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### template 和 call {#template-call}

template 块可以像函数调用一样使用：在其声明中，它们可以获得参数，然后可以在调用它们时传递参数。它们还允许递归。

`data-sly-template` 定义模板。HTL 不输出主机元素及其内容

`data-sly-call` 调用使用 data-sly-template 定义的模板。被调用模板的内容（可以选择进行参数化）替换调用的主机元素的内容。

定义一个静态模板，然后调用它：

```xml
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

定义一个动态模板，然后使用参数调用它：

```xml
<template data-sly-template.two="${ @ title}"><h1>${title}</h1></template>
<div data-sly-call="${two @ title=properties.jcr:title}"></div>
```

位于不同文件中的模板可以使用 `data-sly-use` 进行初始化。请注意，在此示例中，`data-sly-use` 和 `data-sly-call` 也可以位于同一元素上：

```xml
<div data-sly-use.lib="templateLib.html">
    <div data-sly-call="${lib.one}"></div>
    <div data-sly-call="${lib.two @ title=properties.jcr:title}"></div>
</div>
```

支持模板递归：

```xml
<template data-sly-template.nav="${ @ page}">
    <ul data-sly-list="${page.listChildren}">
        <li>
            <div class="title">${item.title}</div>
            <div data-sly-call="${nav @ page=item}" data-sly-unwrap></div>
        </li>
    </ul>
</template>
<div data-sly-call="${nav @ page=currentPage}" data-sly-unwrap></div>
```

## sly 元素 {#sly-element}

`<sly>` HTML 标记可用于删除当前元素，只允许显示其子级。它的功能类似于 `data-sly-unwrap` 块元素：

```xml
<!--/* This will display only the output of the 'header' resource, without the wrapping <sly> tag */-->
<sly data-sly-resource="./header"></sly>
```

虽然 `<sly>` 标记不是有效的 HTML 5 标记，但可以使用 `data-sly-unwrap` 在最终输出中显示：

```xml
<sly data-sly-unwrap="${false}"></sly> <!--/* outputs: <sly></sly> */-->
```

`<sly>` 元素的目标是突出显示该元素不是输出。如果您需要，仍然可以使用 `data-sly-unwrap`。

与 `data-sly-unwrap` 一样，请尝试尽量减少使用它。
