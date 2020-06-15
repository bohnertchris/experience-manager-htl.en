---
title: HTL Block Statements
description: HTML Template Language (HTL) block statements are custom data attributes added directly to existing HTML.
---

# HTL Block Statements {#htl-block-statements}

HTML Template Language (HTL) block statements are custom `data` attributes added directly to existing HTML. This allows easy and unobtrusive annotation of a prototype static HTML page, converting it to a functioning dynamic template without breaking the validity of the HTML code.

## sly element {#sly-element}

The `<sly>` does not get displayed in the resulting HTML and can be used instead of `data-sly-unwrap`. The goal of the `<sly>` element is to make it more obvious that the element is not output. If you want you can still use `data-sly-unwrap`.

```xml
<sly data-sly-test.varone="${properties.yourProp}"/>
```

As with `data-sly-unwrap`, try to minimize the use of this.

## use {#use}

`data-sly-use` initializes a helper object (defined in JavaScript or Java) and exposes it through a variable.

Initialize a JavaScript object, where the source file is located in the same directory as the template. Note that the filename must be used:

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

Initialize a Java class, where the source file is located in the same directory as the template. Note that the classname must be used, not the file name:

```xml
<div data-sly-use.nav="Navigation">${nav.foo}</div>
```

Initialize a Java class, where that class is installed as part of an OSGi bundle. Note that its fully-qualified class name must be used:

```xml
<div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

Parameters can be passed to the initialization using options. Generally this feature should only be used by HTL code that is itself within a `data-sly-template` block:

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

Initialize another HTL template that can then be called using `data-sly-call`:

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>For more information on the Use-API, see:
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)
>

## unwrap {#unwrap}

`data-sly-unwrap` removes the host element from the generated markup while retaining its content. This allows the exclusion of elements that are required as part of HTL presentation logic but are not desired in the actual output.

However, this statement should be used sparingly. In general it is better to keep the HTL markup as close as possible to the intended output markup. In other words, when adding HTL block statements, try as much as possible to simply annotate the existing HTML, without introducing new elements.

For example, this

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

will produce

```xml
<p>Hello World</p>
```

Whereas this,

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

will produce

```xml
Hello World
```

It is also possible to conditionally unwrap an element:

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

## text {#text}

`data-sly-text` replaces the content of its host element with the specified text.

For example, this

```xml
<p>${properties.jcr:description}</p>
```

is equivalent to

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

Both will display the value of `jcr:description` as paragraph text. The advantage of the second method is that is allows the unobtrusive annotation of HTML while keeping the static placeholder content from the original designer.

## attribute {#attribute}

`data-sly-attribute` adds attributes to the host element.

For example, this

```xml
<div title="${properties.jcr:title}"></div>
```

is equivalent to

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

Both will set the `title` attribute to the value of `jcr:title`. The advantage of the second method is that is allows the unobtrusive annotation of HTML while keeping the static placeholder content from the original designer.

Attributes are resolved left to right, with the rightmost instance of an attribute (either literal or defined via `data-sly-attribute`) taking precedence over any instances of the same attribute (defined either literally or via `data-sly-attribute`) defined to its left.

Note that an attribute (either `literal` or set via `data-sly-attribute`) whose value evaluates to the empty string will be removed in the final markup. The one exception to this rule is that a literal attribute set to a literal empty string will be preserved. For example,

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

produces,

```xml
<div></div>
```

but,

```xml
<div class="" data-sly-attribute.id=""></div>
```

produces,

```xml
<div class=""></div>
```

To set multiple attributes, pass a map object hold key-value pairs corresponding to the attributes and their values. For example, assuming,

```xml
attrMap = {
    title: "myTitle",
    class: "myClass",
    id: "myId"
}
```

Then,

```xml
<div data-sly-attribute="${attrMap}"></div>
```

produces,

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

## element {#element}

`data-sly-element` replaces the element name of the host element.

For example,

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

Replaces the `h1` with the value of `titleLevel`.

For security reasons, `data-sly-element` accepts only the following element names:

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub 
sup table tbody td tfoot th thead time tr u var wbr
```

To set other elements, XSS security must be turned off ( `@context='unsafe'`).

## test {#test}

`data-sly-test` conditionally removes the host element and it's content. A value of `false` removes the element; a value of `true` retains the element.

For example, the `p` element and its content will only be rendered if `isShown` is `true`:

```xml
<p data-sly-test="${isShown}">text</p>
```

The result of a test can be assigned to a variable that can be used later. This is usually used to construct "if else" logic, since there is no explicit else statement:

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

The variable, once set, has global scope within the HTL file.

Following are some examples on comparing values:

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

With `data-sly-repeat` you can *repeat* an element multiple times based on the list that is specified.

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

This works the same way as `data-sly-list`, except that you do not need a container element.

The following example shows that you can also refer to the *item* for attributes:

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

## list {#list}

`data-sly-list` repeats the content of the host element for each enumerable property in the provided object.

Here is a simple loop:

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

The following default variables are available within the scope of the list:

* `item`: The current item in the iteration.
* `itemList`: Object holding the following properties:
* `index`: zero-based counter ( `0..length-1`).
* `count`: one-based counter ( `1..length`).
* `first`: `true` if the current item is the first item.
* `middle`: `true` if the current item is neither the first nor the last item.
* `last`: `true` if the current item is the last item.
* `odd`: `true` if `index` is odd.
* `even`: `true` if `index` is even.

Defining an identifier on the `data-sly-list` statement allows you to rename the `itemList` and `item` variables. `item` will become `<variable>` and `itemList` will become `<variable>List`.

```xml
<dl data-sly-list.child="${currentPage.listChildren}">
    <dt>index: ${childList.index}</dt>
    <dd>value: ${child.title}</dd>
</dl>
```

You can also access properties dynamically:

```xml
<dl data-sly-list.child="${myObj}">
    <dt>key: ${child}</dt>
    <dd>value: ${myObj[child]}</dd>
</dl>
```

## resource {#resource}

`data-sly-resource` includes the result of rendering the indicated resource through the sling resolution and rendering process.

A simple resource include:

```xml
<article data-sly-resource="path/to/resource"></article>
```

Options allow a number of additional variants:

Manipulating the path of the resource:

```xml
<article data-sly-resource="${ @ path='path/to/resource'}"></article>
<article data-sly-resource="${'resource' @ prependPath='my/path'}"></article>
<article data-sly-resource="${'my/path' @ appendPath='resource'}"></article>
```

Add (or replace) a selector:

```xml
<article data-sly-resource="${'path/to/resource' @ selectors='selector'}"></article>
```

Add, replace or remove multiple selectors:

```xml
<article data-sly-resource="${'path/to/resource' @ selectors=['s1', 's2']}"></article>
```

Add a selector to the existing ones:

```xml
<article data-sly-resource="${'path/to/resource' @ addSelectors='selector'}"></article>
```

Remove some selectors from the existing ones:

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors='selector1'}"></article>
```

Remove all selectors:

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors}"></article>
```

Overrides the resource type of the resource:

```xml
<article data-sly-resource="${'path/to/resource' @ resourceType='my/resource/type'}"></article>
```

Changes the WCM mode:

```xml
<article data-sly-resource="${'path/to/resource' @ wcmmode='disabled'}"></article>
```

By default, the AEM decoration tags are disabled, the decorationTagName option allows to bring them back, and the cssClassName to add classes to that element.

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM offers clear and simple logic controlling the decoration tags that wrap included elements. For details see [Decoration Tag](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/decoration-tag.html) in the developing components documentation.

## include {#include}

`data-sly-include` replaces the content of the host element with the markup generated by the indicated HTML template file (HTL, JSP, ESP etc.) when it is processed by its corresponding template engine. The rendering context of the included file will not include the current HTL context (that of the including file); Consequently, for inclusion of HTL files, the current `data-sly-use` would have to be repeated in the included file (In such a case it is usually better to use `data-sly-template` and `data-sly-call`)

A simple include:

```xml
<section data-sly-include="path/to/template.html"></section>
```

JSPs can be included the same way:

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

Options let you manipulate the path of the file:

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

You can also change the WCM mode:

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

## template & call {#template-call}

`data-sly-template` defines a template. The host element and its content are not output by HTL

`data-sly-call` calls a template defined with data-sly-template. The content of the called template (optionally parameterized) replaces the content of the host element of the call.

Define a static template and then call it:

```xml
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

Define a dynamic template and then call it with parameters:

```xml
<template data-sly-template.two="${ @ title}"><h1>${title}</h1></template>
<div data-sly-call="${two @ title=properties.jcr:title}"></div>
```

Templates located in a different file, can be initialised with `data-sly-use`. Note that in this case `data-sly-use` and `data-sly-call` could also be placed on the same element:

```xml
<div data-sly-use.lib="templateLib.html">
    <div data-sly-call="${lib.one}"></div>
    <div data-sly-call="${lib.two @ title=properties.jcr:title}"></div>
</div>
```

Template recursion is supported:

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

## i18n and locale objects {#i-n-and-locale-objects}

When you are using i18n and HTL, you can now also pass in custom locale objects.

```xml
${'Hello World' @ i18n, locale=request.locale}
```

## URL manipulation {#url-manipulation}

A new set of url manipulations is available.

See the following examples on their usage:

Adds the html extension to a path.

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

Adds the html extension and a selector to a path.

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

Adds the html extension and a fragment (#value) to a path.

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

## HTL Features Supported in AEM 6.3 and Later{#htl-features-supported-in-aem}

The following new HTL features are supported in Adobe Experience Manager (AEM) 6.3 and later:

### Number/Date-formatting {#number-date-formatting}

HTL allows native formatting of numbers and dates, without writing custom code. This also supports timezone and locale.

The following examples show that the format is specified first, then the value that needs formatting:

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>

```

>[!NOTE]
>
>For complete details on the format you can use, refer to [HTL-specification](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md).

### data-sly-use with resources {#data-sly-use-with-resources}

This allows to get resources directly in HTL with data-sly-use and does not require to write code to get the resource.

For example:

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

### Request-attributes {#request-attributes}

In the `data-sly-include` and `data-sly-resource` you can now pass `requestAttributes` in order to use them in the receiving HTL-script.

This allows you to properly pass-in parameters into scripts or components.

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings" 
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Java-code of the Settings class, the Map is used to pass in the requestAttributes:

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

For example, via a Sling-Model, you can consume the value of the specified `requestAttributes`.

In this example, layout is injected via the Map from the Use-class:

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### Fix for @extension {#fix-for-extension}

The `@extension` works in all scenarios in AEM 6.3 and later. In previous versions you could have a result such as `www.adobe.com.html`. It also checks whether to add the extension or not.

```xml
${ link @ extension = 'html' }
```
