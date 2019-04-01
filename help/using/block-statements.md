---
title: HTL块语句
seo-title: HTL块语句
description: HTML模板语言(HTL)块语句是直接添加到现有HTML的自定义数据属性。
seo-description: HTML模板语言(HTL)块语句是直接添加到现有HTML的自定义数据属性。
uuid: 0624fb6e-6989-431b-aabc-1138325393f1
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 58aa6ea8-1d45-4f6f-a77 e-4819f593 a19 d
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 7a94b0b010461b29d2b74c9c717e3b218d0ca5a8

---


# HTL块语句 {#htl-block-statements}

HTML模板语言(HTL)块语句是直接添加到现有HTML的自定义 `data` 属性。这允许对静态HTML页面的原型进行简单而不显眼的注释，将其转换为有效的动态模板，而不破坏HTML代码的有效性。

## sy元素 {#sly-element}

& amp **；t；sly& amp；gt；元素** 不会显示在生成的HTML中，而是可以使用而不是数据释放。& amp的目标；t；sly& amp；gt；元素是为了使元素不会输出出来。如需，您仍可使用数据自由包裹。

```xml
<sly data-sly-test.varone="${properties.yourProp}"/>
```

与数据-缓慢包裹一样，尽量尽量减少使用。

## use {#use}

**`data-sly-use`**：初始化辅助对象(在JavaScript或Java中定义)并通过变量显示它。

初始化JavaScript对象，其中源文件与模板位于同一目录中。请注意，必须使用文件名：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化Java类，其中源文件与模板位于同一目录中。请注意，必须使用classname，而不是文件名：

```xml
        <div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化Java类，该类在OSGi包的一部分中安装。请注意，必须使用其完全限定的类名称：

```xml
        <div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

可以 *使用选项*将参数传递到初始化。通常，此功能只应由 `data-sly-template` 位于块中的HTL代码使用：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化其他HTL模板，然后使用 **`data-sly-call`**以下方法调用该模板：

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>有关使用API的详细信息，请参阅：
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript使用API](use-api-javascript.md)
>



## 取消绕排 {#unwrap}

**`data-sly-unwrap`**：从生成的标记中删除主机元素，同时保留其内容。这允许排除作为HTL演示逻辑一部分必需的元素，但实际输出中不需要这些元素。

但是，应谨慎使用此语句。通常，最好保持HTL标记尽可能接近预期输出标记。换句话说，在添加HTL块语句时，尽量尽可能简单地注释现有HTML，而无需引入新元素。

例如，

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

将生成

```xml
<p>Hello World</p>
```

然而，

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

将生成

```xml
Hello World
```

还可以有条件地取消绕排元素：

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

## 文本 {#text}

**`data-sly-text`**：用指定的文本替换其主机元素的内容。

例如，

```xml
<p>${properties.jcr:description}</p>
```

等同于

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

两者将显示作为段落 **`jcr:description`** 文本的值。第二种方法的优点是允许HTML不显眼的注释，同时保留原始设计人员的静态占位符内容。

## attribute {#attribute}

**data-sly-property**：向主机元素添加属性。

例如，

```xml
<div title="${properties.jcr:title}"></div>
```

等同于

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

两者都会将属性 `title` 设置 **`jcr:title`**为值。第二种方法的优点是允许HTML不显眼的注释，同时保留原始设计人员的静态占位符内容。

属性从左向右解析，属性最右侧的实例(通过文本或定义的属性) **`data-sly-attribute`**优先于定义为左侧的相同属性(定义或通过) **`data-sly-attribute`**的任何实例。

请注意，在最终标记 **`literal`** 中将删除 **`data-sly-attribute`**其值 *计算* 为空字符串的属性(或通过此属性设置)。此规则的一个例外是，将 *保留* 为 *文本* 空字符串设置的文本属性。例如，

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

生成、

```xml
<div></div>
```

但是，

```xml
<div class="" data-sly-attribute.id=""></div>
```

生成、

```xml
<div class=""></div>
```

要设置多个属性，请传递一个对应于属性及其值的键值对。例如，假设，

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

生成、

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

## 元素{#element}

**`data-sly-element`**：替换主机元素的元素名称。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

替换值的 **`h1`** 值 **`titleLevel`**。

由于安全原因， `data-sly-element` 只接受以下元素名称：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub 
sup table tbody td tfoot th thead time tr u var wbr
```

要设置其他元素，必须关闭XSS安全性( `@context='unsafe'`)。

## 测试 {#test}

**`data-sly-test`：** 以条件方式删除主机元素及其内容。值可 `false` 删除元素；值将 `true` 保留元素。

例如 `p` ，只有在以下情况下，才会渲染元素及其内容 `isShown``true`：

```xml
<p data-sly-test="${isShown}">text</p>
```

测试结果可以分配给以后可使用的变量。这通常用于构造“if else”逻辑，因为没有明确的else语句：

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

变量设置后，在HTL文件中具有全局范围。

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

## repeat {#repeat}

通过数据-sly-repreat，您可以基于指定的列表多次 *重复* 元素。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

这与数据列表的工作方式相同，只是您不需要容器元素。

以下示例演示了您还可以引用属性的 *项目* ：

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

## list {#list}

**`data-sly-list`**：在提供的对象中重复每个可枚举属性的主机元素的内容。

下面是简单的循环：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

列表的范围内提供以下默认变量：

**`item`**：迭代中的当前项目。

**`itemList`**：具有以下属性的对象：

**`index`**：从零开始的计数器( `0..length-1`)。

**`count`**：一个built计数器( `1..length`)。

`first`： `true` 如果当前项目是第一个项目。

**`middle`**： `true` 如果当前项目既不是第一个项目也不是最后一个项目。

**`last`**： `true` 如果当前项目是最后一个项目。

**`odd`**： `true``index` 如果为奇数。

**`even`**： `true` if `index` is even.

在 `data-sly-list` 语句上定义标识符可使您重命名 **`itemList`**`item` 和变量。**`item`** 将变为*** `<variable>`***并将 **`itemList`** 变为 **`*<variable>*List`**。

```xml
<dl data-sly-list.child="${currentPage.listChildren}">
    <dt>index: ${childList.index}</dt>
    <dd>value: ${child.title}</dd>
</dl>
```

您还可以动态地访问属性：

```xml
<dl data-sly-list.child="${myObj}">
    <dt>key: ${child}</dt>
    <dd>value: ${myObj[child]}</dd>
</dl>
```

## 资源 {#resource}

**`data-sly-resource`**：包括通过sling分辨率和渲染过程渲染指定资源的结果。

简单资源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

选项允许许多其他变体：

操作资源的路径：

```xml
<article data-sly-resource="${ @ path='path/to/resource'}"></article>
<article data-sly-resource="${'resource' @ prependPath='my/path'}"></article>
<article data-sly-resource="${'my/path' @ appendPath='resource'}"></article>
```

添加(或替换)选择器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors='selector'}"></article>
```

添加、替换或删除多个选择器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors=['s1', 's2']}"></article>
```

向现有选择器添加选择器：

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

默认情况下，AEM修饰标记处于禁用状态，允许使用修饰TagName选项返回它们，并允许cssClassName向该元素添加类。

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM提供了清晰、简单的逻辑，用于控制包含元素的装饰标记。有关详细信息，请参阅 [开发组件文档中的装饰标记](https://helpx.adobe.com/experience-manager/6-4/sites/developing/using/decoration-tag.html) 。

## includes {#include}

**`data-sly-include`**：替换主机元素的内容，使用由指示的HTML模板文件(HTL、JSP、ESP等)生成的标记。当其由其相应的模板引擎处理时。*包含文件* 的呈现上下文不包括当前HTL上下文(包括文件的 *包含)*；因此，为了包含HTL文件，必须 **`data-sly-use`** 在包含的文件中重复现有文件(在这种情况下，通常更好使用 **`data-sly-template`** 和 `data-sly-call`)

简单包括：

```xml
<section data-sly-include="path/to/template.html"></section>
```

JSP可以包含在以下方式中：

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

选项允许您操作文件的路径：

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

您还可以更改WCM模式：

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

## 模板和调用 {#template-call}

`data-sly-template`：定义模板。主机元素及其内容不会由HTL输出

`data-sly-call`：调用使用data-sly-template定义的模板。被称为模板的内容(可选)替换了调用主机元素的内容。

定义静态模板，然后调用它：

```xml
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

定义一个动态模板，然后使用参数调用它：

```xml
<template data-sly-template.two="${ @ title}"><h1>${title}</h1></template>
<div data-sly-call="${two @ title=properties.jcr:title}"></div>
```

可以使用其他文件中的模板初始化 `data-sly-use`。请注意，在这种情况 `data-sly-use` 下，也 `data-sly-call` 可以放在同一元素上：

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

## i18n和区域设置对象 {#i-n-and-locale-objects}

当您使用i18n和HTL时，您现在还可以传入自定义区域设置对象。

```xml
${'Hello World' @ i18n, locale=request.locale}
```

## URL操作 {#url-manipulation}

提供了一组新的url操作。

有关其用法，请参见以下示例：

将html扩展添加到路径。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

将html扩展名和选择器添加到路径。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

将html扩展名和片段(# value)添加到路径。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

## AEM6.3支持的HTL功能 {#htl-features-supported-in-aem}

Adobe Experience Manager(AEM)6.3支持以下新HTL功能：

### 数字/日期格式 {#number-date-formatting}

AEM6.3支持数字和日期的本机格式化，无需编写自定义代码。这还支持时区和区域设置。

以下示例显示了首先指定格式，然后显示需要格式化的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>有关您可以使用的格式的完整详细信息，请参阅 [HTL规范](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 数据与资源的紧密结合 {#data-sly-use-with-resources}

这允许在HTL中直接使用数据获取资源，无需编写代码即可获取资源。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

### 请求属性 {#request-attributes}

在 *data-sly-include* 和 *data-sly-资源* 中，您现在可以传递 *requestAttributes* ，以便在接收HTL脚本中使用它们。

这允许您将参数正确传递到脚本或组件中。

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings" 
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Settings类的Java代码，The Map用于传入requestAttributes：

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

例如，通过Sling-Model，您可以使用指定的requestAttributes的值。

在此示例中 *，布局* 通过使用类中的映射进行注入：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### 修复@扩展 {#fix-for-extension}

@ extension可在AEM6.3中的所有场景中工作，之前您的结果可能像 *www.adobe.com.html* ，而且还检查是否添加或未添加扩展名。

```xml
${ link @ extension = 'html' }
```
