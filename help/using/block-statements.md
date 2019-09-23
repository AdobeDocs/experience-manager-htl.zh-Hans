---
title: HTL块语句
seo-title: HTL块语句
description: HTML模板语言(HTL)块语句是直接添加到现有HTML的自定义数据属性。
seo-description: 'HTML模板语言(HTL)块语句是直接添加到现有HTML的自定义数据属性。 '
uuid: 0624fb6e-6989-431b-abc-1138325393f1
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 58aa6ea8-1d45-4f6f-a77e-4819f593a19d
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: afc29cbad83caeb549097da3fc33fd9147f1157a

---


# HTL块语句 {#htl-block-statements}

HTML模板语言(HTL)块语句是直接添加到现 `data` 有HTML的自定义属性。 这样，原型静态HTML页面便于轻松、不显眼地进行注释，将其转换为可正常工作的动态模板，而不会破坏HTML代码的有效性。

## 狡猾元件 {#sly-element}

在生 **成的HTML中不显示** &lt;sly&gt;元素，并且可以使用该元素代替数据sly-unwrap。 该元素的目的是使元素不输出更明显。 如果您想要，您仍可以使用数据破解。

```xml
<sly data-sly-test.varone="${properties.yourProp}"/>
```

与数据秘密解包一样，请尽量减少对它的使用。

## use {#use}

**`data-sly-use`**:初始化帮助对象（在JavaScript或Java中定义），并通过变量显示它。

初始化JavaScript对象，其中源文件与模板位于同一目录中。 请注意，必须使用文件名：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化Java类，其中源文件与模板位于同一目录中。 请注意，必须使用类名，而不是文件名：

```xml
        <div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化Java类，该类作为OSGi包的一部分安装。 请注意，必须使用其完全限定的类名：

```xml
        <div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

可以使用选项将参数传递到初始 *化*。 通常，此功能仅应由块内的HTL代码使 `data-sly-template` 用：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化另一个HTL模板，然后可使用以下方式调用该模板 **`data-sly-call`**:

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>有关Use-API的详细信息，请参阅：
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)
>



## 取消绕排 {#unwrap}

**`data-sly-unwrap`**:从生成的标记中删除主机元素，同时保留其内容。 这允许排除作为HTL表示逻辑的一部分而在实际输出中不需要的元素。

但是，这一说法应少用一些。 通常，最好使HTL标记尽可能接近预期的输出标记。 换句话说，在添加HTL块语句时，尽可能地尝试只对现有HTML添加注释，而不引入新元素。

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

也可以有条件地取消元素绕排：

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

## 文本 {#text}

**`data-sly-text`**:将其主机元素的内容替换为指定的文本。

例如，

```xml
<p>${properties.jcr:description}</p>
```

等于

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

两者将显示段落文 **`jcr:description`** 本的值。 第二种方法的优点是允许HTML的不显眼的注释，同时保留原始设计人员的静态占位符内容。

## 属性 {#attribute}

**data-sly-attribute**:向主机元素添加属性。

例如，

```xml
<div title="${properties.jcr:title}"></div>
```

等于

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

两者都会将 `title` 属性设置为的值 **`jcr:title`**。 第二种方法的优点是允许HTML的不显眼的注释，同时保留原始设计人员的静态占位符内容。

属性从左到右解析，其最右侧的属性实例(文本或通过定义 **`data-sly-attribute`**)优先于在其左侧定义的相同属性实例(字面或通过定义 **`data-sly-attribute`**)。

请注意，其值计算 **`literal`** 为空字符串的属性( **`data-sly-attribute`**&#x200B;或通过设置 ** )将在最终标记中删除。 此规则的一个例外是将 *保留* 设置为文本 *空字符串的* 文本属性。 例如，

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

要设置多个属性，请传递一个映射对象，其中包含与这些属性及其值相对应的键值对。 例如，假设，

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

## 元素{#element}

**`data-sly-element`**:替换主机元素的元素名称。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

用 **`h1`** 的值替换 **`titleLevel`**。

出于安全原因， `data-sly-element` 仅接受以下元素名称：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub 
sup table tbody td tfoot th thead time tr u var wbr
```

要设置其他元素，必须关闭( `@context='unsafe'`)XSS安全性。

## 测试 {#test}

**`data-sly-test`** :有条件地删除主机元素及其内容。 值删除 `false` 元素；值将保 `true` 留元素。

例如，元素 `p` 及其内容仅在以下情况下才会 `isShown` 呈现 `true`:

```xml
<p data-sly-test="${isShown}">text</p>
```

测试结果可以指定给以后可以使用的变量。 这通常用于构造“if else”逻辑，因为没有明确的else语句：

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

设置变量后，该变量在HTL文件内具有全局范围。

以下是比较值的一些示例：

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

使用数据密码重复，您可 *以根据指定的列表* ，多次重复元素。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

这与数据密码列表的工作方式相同，只是您不需要容器元素。

以下示例显示您还可以参考项 *目* （属性）:

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

## list {#list}

**`data-sly-list`**:为提供的对象中的每个可枚举属性重复主机元素的内容。

下面是一个简单循环：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

在列表的范围内，有以下默认变量可用：

**`item`**:小版本中的当前项。

**`itemList`**:包含以下属性的对象：

**`index`**:零计数器( `0..length-1`)。

**`count`**:基于一的计数器( `1..length`)。

`first`:如果 `true` 当前项目是第一个项目。

**`middle`**:如果 `true` 当前项目既不是第一个项目也不是最后一个项目。

**`last`**:如果 `true` 当前项目是最后一个项目。

**`odd`**:如 `true` 果 `index` 奇数。

**`even`**:即 `true` 使 `index` 是扯平。

在语句上定义标 `data-sly-list` 识符允许您重命名和 **`itemList`** 变 `item` 量。 **`item`** 将变为*** `<variable>`**，并将变 **`itemList`** 为 **`*<variable>*List`**。

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

## 资源 {#resource}

**`data-sly-resource`**:包括通过sling分辨率和渲染过程渲染指示的资源的结果。

简单的资源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

选项允许许多其他变体：

处理资源路径：

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

默认情况下，AEM装饰标签处于禁用状态，decorationTagName选项允许将其重新显示，cssClassName则允许将类添加到该元素。

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM提供清晰、简单的逻辑，用于控制包含元素的装饰标签。 有关详细信息，请 [参阅开发组件文档中](https://helpx.adobe.com/experience-manager/6-4/sites/developing/using/decoration-tag.html) 的装饰标签。

## include {#include}

**`data-sly-include`**:将主机元素的内容替换为由指示的HTML模板文件（HTL、JSP、ESP等）生成的标记当它由其相应的模板引擎处理时。 包含文件的呈现上 *下文不包括* (包含文件的呈现上 **&#x200B;下文);因此，要包含HTL文件，必须 **`data-sly-use`** 在包含的文件中重复当前文件(在这种情况下，通常最好使用和 **`data-sly-template`** ) `data-sly-call`。

简单包括：

```xml
<section data-sly-include="path/to/template.html"></section>
```

JSP的包含方式与以下相同：

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

通过选项，您可以操作文件的路径：

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

您还可以更改WCM模式：

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

## 模板与呼叫 {#template-call}

`data-sly-template`:定义模板。 HTL不输出主机元素及其内容

`data-sly-call`:调用使用数据密集模板定义的模板。 调用的模板的内容（可选地参数化）将替换调用的主机元素的内容。

定义静态模板，然后将其调用：

```xml
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

定义动态模板，然后使用参数调用它：

```xml
<template data-sly-template.two="${ @ title}"><h1>${title}</h1></template>
<div data-sly-call="${two @ title=properties.jcr:title}"></div>
```

位于其他文件中的模板可用进行初始化 `data-sly-use`。 请注意，在这种情 `data-sly-use` 况下， `data-sly-call` 也可以放在同一元素上：

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

当您使用i18n和HTL时，您现在还可以传递自定义区域设置对象。

```xml
${'Hello World' @ i18n, locale=request.locale}
```

## URL处理 {#url-manipulation}

有一组新的url操作可用。

请参阅以下有关其用法的示例：

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

## AEM 6.3中支持的HTL功能 {#htl-features-supported-in-aem}

Adobe Experience Manager(AEM)6.3支持以下新的HTL功能：

### 数字／日期格式 {#number-date-formatting}

AEM 6.3支持数字和日期的本机格式化，无需编写自定义代码。 这还支持时区和区域设置。

以下示例显示先指定格式，然后指定需要格式的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>有关可以使用的格式的完整详细信息，请参阅 [HTL规范](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 与资源的数据密集使用 {#data-sly-use-with-resources}

这允许在HTL中直接获取资源并进行数据密集使用，并且无需编写代码即可获取资源。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

### 请求属性 {#request-attributes}

在 *data-sly-include**和data-sly-resource中* ，您现在可以传递requestAttributes ** ，以便在接收HTL-script中使用它们。

这允许您将参数正确传入脚本或组件。

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings" 
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Settings类的Java代码，Map用于传递requestAttributes:

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

在此示例中， *布局通过* Use类中的Map插入：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### @extension的修复 {#fix-for-extension}

@extension适用于AEM 6.3中的所有场景，此时您可能会得到www.adobe.com.ht *ml* 等结果，并检查是否添加扩展。

```xml
${ link @ extension = 'html' }
```
