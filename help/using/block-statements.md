---
title: HTL 块语句
description: HTML模板语言(HTL)块语句是直接添加到现有HTML的自定义数据属性。
exl-id: a517dcef-ab7a-4d4c-a1a9-2e57aad034f7
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '1555'
ht-degree: 1%

---

# HTL 块语句 {#htl-block-statements}

HTML模板语言(HTL)块语句是直接添加到现有HTML的自定义`data`属性。 这样，原型静态HTML页面的注释就变得简单而直观，可以将其转换为正常工作的动态模板，而不会破坏HTML代码的有效性。

## 块概述 {#overview}

HTL块插件由在HTML元素中设置的`data-sly-*`属性定义。 元素可以具有结束标记或自结束标记。 属性可以具有值（可以是静态字符串或表达式），也可以只是布尔属性（不含值）。

```xml
<tag data-sly-BLOCK></tag>                                 <!--/* A block is simply consists in a data-sly attribute set on an element. */-->
<tag data-sly-BLOCK/>                                      <!--/* Empty elements (without a closing tag) should have the trailing slash. */-->
<tag data-sly-BLOCK="string value"/>                       <!--/* A block statement usually has a value passed, but not necessarily. */-->
<tag data-sly-BLOCK="${expression}"/>                      <!--/* The passed value can be an expression as well. */-->
<tag data-sly-BLOCK="${@ myArg='foo'}"/>                   <!--/* Or a parametric expression with arguments. */-->
<tag data-sly-BLOCKONE="value" data-sly-BLOCKTWO="value"/> <!--/* Several block statements can be set on a same element. */-->
```

所有评估的`data-sly-*`属性都将从生成的标记中删除。

### 标识符 {#identifiers}

块语句后面还可以跟一个标识符：

```xml
<tag data-sly-BLOCK.IDENTIFIER="value"></tag>
```

块语句可以通过各种方式使用标识符，以下是一些示例：

```xml
<!--/* Example of statements that use the identifier to set a variable with their result: */-->
<div data-sly-use.navigation="MyNavigation">${navigation.title}</div>
<div data-sly-test.isEditMode="${wcmmode.edit}">${isEditMode}</div>
<div data-sly-list.child="${currentPage.listChildren}">${child.properties.jcr:title}</div>
<div data-sly-template.nav>Hello World</div>

<!--/* The attribute statement uses the identifier to know to which attribute it should apply it's value: */-->
<div data-sly-attribute.title="${properties.jcr:title}"></div> <!--/* This will create a title attribute */-->
```

顶级标识符不区分大小写（因为它们可以通过HTML属性进行设置，而HTML属性不区分大小写），但其所有属性都区分大小写。

## 可用的块语句{#available-block-statements}

有许多块语句可用。 在同一元素上使用时，以下优先级列表定义了如何计算块语句：

1. `data-sly-template`
1. `data-sly-set`,  `data-sly-test`  `data-sly-use`
1. `data-sly-call`
1. `data-sly-text`
1. `data-sly-element`,  `data-sly-include`  `data-sly-resource`
1. `data-sly-unwrap`
1. `data-sly-list`, `data-sly-repeat`
1. `data-sly-attribute`

当两个块语句具有相同的优先级时，其评估顺序从左到右。

### use {#use}

`data-sly-use` 初始化帮助程序对象（在JavaScript或Java中定义）并通过变量公开它。

初始化JavaScript对象，其中源文件位于与模板相同的目录中。 请注意，必须使用文件名：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化Java类，其中源文件位于与模板相同的目录中。 请注意，必须使用类名，而不是文件名：

```xml
<div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化Java类，在该类中，该类将作为OSGi包的一部分安装。 请注意，必须使用其完全限定的类名：

```xml
<div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

可以使用选项将参数传递到初始化。 通常，此功能只应由位于`data-sly-template`块内的HTL代码使用：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化另一个HTL模板，然后可以使用`data-sly-call`调用该模板：

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>有关Use-API的更多信息，请参阅：
>
>* [Java Use-API](use-api-java.md)
* [JavaScript Use-API](use-api-javascript.md)


#### 对资源{#data-sly-use-with-resources}使用data-sly

这允许通过`data-sly-use`直接在HTL中获取资源，并且无需编写代码即可获取资源。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

>[!TIP]
另请参阅[路径并非始终必需部分。](#path-not-required)

### 取消换行 {#unwrap}

`data-sly-unwrap` 从生成的标记中删除主机元素，同时保留其内容。这允许排除作为HTL表示逻辑的一部分所需但在实际输出中不需要的元素。

但是，应谨慎使用这一声明。 通常，最好将HTL标记保持尽可能接近预期的输出标记。 换言之，在添加HTL块语句时，尽量尝试只对现有HTML添加注释，而不引入新元素。

例如，

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

生产

```xml
<p>Hello World</p>
```

然而，

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

生产

```xml
Hello World
```

也可以有条件地取消元素绕组：

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

### set {#set}

`data-sly-set` 使用预定义的值定义新标识符。

```xml
<span data-sly-set.profile="${user.profile}">Hello, ${profile.firstName} ${profile.lastName}!</span>
<a class="profile-link" href="${profile.url}">Edit your profile</a>
```

### 文本 {#text}

`data-sly-text` 将其主机元素的内容替换为指定的文本。

例如，

```xml
<p>${properties.jcr:description}</p>
```

等于

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

二者都会将值`jcr:description`显示为段落文本。 第二种方法的优势在于，允许对HTML进行不显眼的注释，同时保留原始设计人员的静态占位符内容。

### 属性 {#attribute}

`data-sly-attribute` 将属性添加到主机元素。

例如，

```xml
<div title="${properties.jcr:title}"></div>
```

等于

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

二者都会将`title`属性设置为`jcr:title`的值。 第二种方法的优势在于，允许对HTML进行不显眼的注释，同时保留原始设计人员的静态占位符内容。

属性从左到右进行解析，其最右侧的属性实例（文字或通过`data-sly-attribute`定义）优先于在其左侧定义的同一属性实例（字面或通过`data-sly-attribute`定义）。

请注意，其值计算为空字符串的属性（`literal`或通过`data-sly-attribute`设置）将在最终标记中删除。 此规则的一个例外是，将保留设置为文字空字符串的文字属性。 例如，

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

生产，

```xml
<div></div>
```

但是，

```xml
<div class="" data-sly-attribute.id=""></div>
```

生产，

```xml
<div class=""></div>
```

要设置多个属性，请传递与属性及其值对应的映射对象保存键值对。 例如，假设

```xml
attrMap = {
    title: "myTitle",
    class: "myClass",
    id: "myId"
}
```

然后,

```xml
<div data-sly-attribute="${attrMap}"></div>
```

生产，

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

### 元素 {#element}

`data-sly-element` 替换主机元素的元素名称。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

将`h1`替换为值`titleLevel`。

出于安全原因，`data-sly-element`仅接受以下元素名称：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub
sup table tbody td tfoot th thead time tr u var wbr
```

要设置其他元素，必须关闭XSS安全性(`@context='unsafe'`)。

### 测试 {#test}

`data-sly-test` 有条件地删除主机元素及其内容。值`false`将删除该元素；值`true`将保留元素。

例如，`p`元素及其内容仅在`isShown`为`true`时才会呈现：

```xml
<p data-sly-test="${isShown}">text</p>
```

测试结果可以分配给稍后可以使用的变量。 由于没有明确的else语句，因此通常用于构建“if else”逻辑：

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

变量一经设置，即在HTL文件中具有全局范围。

以下是一些比较值的示例：

```xml
<div data-sly-test="${properties.jcr:title == 'test'}">TEST</div>
<div data-sly-test="${properties.jcr:title != 'test'}">NOT TEST</div>

<div data-sly-test="${properties['jcr:title'].length > 3}">Title is longer than 3</div>
<div data-sly-test="${properties['jcr:title'].length >= 0}">Title is longer or equal to zero </div>

<div data-sly-test="${properties['jcr:title'].length > aemComponent.MAX_LENGTH}">
    Title is longer than the limit of ${aemComponent.MAX_LENGTH}
</div>
```

### 重复 {#repeat}

使用`data-sly-repeat`，您可以根据指定的列表多次重复元素。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

其工作方式与`data-sly-list`相同，不同之处在于您不需要容器元素。

以下示例显示您还可以引用&#x200B;*item*&#x200B;来获取属性：

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

### 列表 {#list}

`data-sly-list` 为提供对象中的每个可枚举属性重复主机元素的内容。

下面是一个简单的循环：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

列表范围内提供以下默认变量：

* `item`:小版本中的当前项。
* `itemList`:包含以下属性的对象：
* `index`:零计数器( `0..length-1`)。
* `count`:基于一的计数器( `1..length`)。
* `first`: `true` 如果当前项目是第一个项目。
* `middle`: `true` 当前项目既不是第一个项目，也不是最后一个项目时。
* `last`: `true` 如果当前项目是最后一个项目。
* `odd`: `true` 如 `index` 果奇数。
* `even`: `true` 如 `index` 果为偶。

通过在`data-sly-list`语句中定义标识符，可以重命名`itemList`和`item`变量。 `item` 会变 `<variable>` 成 `itemList` 的 `<variable>List`。

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

### 资源 {#resource}

`data-sly-resource` 包括通过sling解析和渲染过程渲染指示的资源的结果。

简单的资源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

#### 路径并非始终必需{#path-not-required}

请注意，如果您已经拥有资源，则不需要使用具有`data-sly-resource`的路径。 如果您已经拥有资源，则可以直接使用它。

例如，以下内容是正确的。

```xml
<sly data-sly-resource="${resource.path @ decorationTagName='div'}"></sly>
```

但是，以下也是完全可以接受的。

```xml
<sly data-sly-resource="${resource @ decorationTagName='div'}"></sly>
```

由于以下原因，建议尽可能直接使用资源。

* 如果您已经拥有资源，则使用路径重新解析将是一项不必要的工作。
* 在您已经拥有资源时使用路径可能会引入意外结果，因为Sling资源可以封装或者是合成的，而不是在给定路径中提供。

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

更改WCM模式：

```xml
<article data-sly-resource="${'path/to/resource' @ wcmmode='disabled'}"></article>
```

默认情况下，AEM修饰标记处于禁用状态，decorationTagName选项允许将标记重新显示，而cssClassName则用于向该元素添加类。

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
AEM提供了清晰而简单的逻辑，用于控制封装所包含元素的修饰标记。 有关详细信息，请参阅开发组件文档中的[装饰标签](https://docs.adobe.com/content/help/en/experience-manager-65/developing/components/decoration-tag.html)。

### 包括 {#include}

`data-sly-include` 将主机元素的内容替换为由指示的HTML模板文件（HTL、JSP、ESP等）生成的标记当由相应的模板引擎处理时。 包含文件的呈现上下文将不包括当前HTL上下文（包含文件的上下文）；因此，要包含HTL文件，必须在包含的文件中重复当前的`data-sly-use`（在这种情况下，通常最好使用`data-sly-template`和`data-sly-call`）

简单包括：

```xml
<section data-sly-include="path/to/template.html"></section>
```

JSP的包含方式相同：

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

通过选项，您可以处理文件的路径：

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

您还可以更改WCM模式：

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

### 请求属性{#request-attributes}

在`data-sly-include`和`data-sly-resource`中，您可以传递`requestAttributes`，以便在接收HTL脚本中使用它们。

这允许您将相应的参数正确传递到脚本或组件中。

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings"
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Settings类的Java代码中，Map用于在requestAttributes中传递：

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

例如，通过Sling-Model，您可以使用指定`requestAttributes`的值。

在此示例中，布局通过Use类中的Map插入：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### 模板和调用{#template-call}

模板块的使用方式与函数调用类似：在他们的声明中，他们可以获取参数，随后在调用参数时可以传递这些参数。 它们还允许递归。

`data-sly-template` 定义模板。HTL不会输出主机元素及其内容

`data-sly-call` 调用使用data-sly-template定义的模板。调用模板的内容（可选进行参数化）将替换调用的主机元素的内容。

定义静态模板，然后调用该模板：

```xml
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

定义动态模板，然后使用参数调用该模板：

```xml
<template data-sly-template.two="${ @ title}"><h1>${title}</h1></template>
<div data-sly-call="${two @ title=properties.jcr:title}"></div>
```

位于其他文件中的模板可以使用`data-sly-use`进行初始化。 请注意，在这种情况下，`data-sly-use`和`data-sly-call`也可以放在同一元素上：

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

## Sly元素{#sly-element}

`<sly>` HTML标记可用于删除当前元素，仅允许显示其子元素。 其功能与`data-sly-unwrap`块元素类似：

```xml
<!--/* This will display only the output of the 'header' resource, without the wrapping <sly> tag */-->
<sly data-sly-resource="./header"></sly>
```

虽然不是有效的HTML 5标记，但可以使用`data-sly-unwrap`在最终输出中显示`<sly>`标记：

```xml
<sly data-sly-unwrap="${false}"></sly> <!--/* outputs: <sly></sly> */-->
```

`<sly>`元素的目标是使元素不输出这一点更加明显。 如果需要，您仍可以使用`data-sly-unwrap`。

与`data-sly-unwrap`一样，请尽量减少此功能的使用。
