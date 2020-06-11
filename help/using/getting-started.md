---
title: Getting Started with HTL
description: HTL supported by AEM takes the place of JSP as the preferred and recommended server-side template system for HTML in AEM.
---

# Getting Started with HTL {#getting-started-with-htl}

The HTML Template Language (HTL) supported by Adobe Experience Manager (AEM) takes the place of JSP (JavaServer Pages) as the preferred and recommended server-side template system for HTML in AEM.

>[!NOTE]
>
>To run most examples provided on this page, a live execution environment called the [Read Eval Print Loop](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl) can be used.
>
>The AEM Community has generated a series of [articles, videos and webinars](related-community-articles.md) related to using HTL.

## HTL over JSP {#htl-over-jsp}

It is recommended for new AEM projects to use the HTML Template Language, as it offers multiple benefits compared to JSP. For existing projects though, a migration only makes sense if it is estimated to be less effort than maintaining the existing JSPs for the coming years.

But moving to HTL is not necessarily an all-or-nothing choice, because components written in HTL are compatible with components written in JSP or ESP. Meaning that existing projects can without a problem use HTL for new components, while keeping JSP for existing components.

Even within the same component, HTL files can be used alongside JSPs and ESPs. Following example shows on **line 1** how to include an HTL file from a JSP file, and on **line 2** how a JSP file can be included from an HTL file:

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### Frequently Asked Questions{#frequently-asked-questions}

Before getting started with the HTML Template Language, let's start with answering upfront some questions related to the JSP vs HTL topic.

**Does HTL have any limitations that JSP doesn't?** - HTL doesn't really have limitations compared to JSP in the sense that what can be done with JSP should also be achievable with HTL. However, HTL is by design stricter than JSP in several aspects, and what can be achieved all in a single JSP file, might need to be separated into a Java class or a JavaScript file to be achievable in HTL. But this is generally desired to ensure a good separation of concerns between the logic and the markup.

**Does HTL support JSP Tag Libraries?** - No, but as shown in the [Loading Client Libraries](getting-started.md#loading-client-libraries) section, the [template & call](block-statements.md#template-call) statements offer a similar pattern.

**Can the HTL features be extended on an AEM project?** - No, they cannot. HTL has powerful extension mechanisms for reuse of logic - the [Use-API](getting-started.md#use-api-for-accessing-logic) - and of markup (the [template & call](block-statements.md#template-call) statements), which can be used to modularize the code of projects.

**What are the main benefits of HTL over JSP?** - Security and project efficiency are the main benefits, which are detailed on the [Overview](overview.md).

**Will JSP eventually go away?** - At the current date, there are no plans along these lines.

## Fundamental Concepts of HTL {#fundamental-concepts-of-htl}

The HTML Template Language uses an expression language to insert pieces of content into the rendered markup, and HTML5 data attributes to define statements over blocks of markup (like conditions or iterations). As HTL gets compiled into Java Servlets, the expressions and the HTL data attributes are both evaluated entirely server-side, and nothing remains visible in the resulting HTML.

### Blocks and Expressions {#blocks-and-expressions}

Here's a first example, which could be contained as is in a **`template.html`** file:

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

Two different kind of syntaxes can be distinguished:

* **[Block Statements](block-statements.md)** - To conditionally display the **&lt;h1&gt;** element, a [`data-sly-test`](block-statements.md#test) HTML5 data attribute is used. HTL provides multiple such attributes, which allow to attach behavior to any HTML element, and all are prefixed with `data-sly`.  

* **[Expression Language](expression-language.md)** - HTL expressions are delimited by characters `${` and `}`. At runtime, these expressions are evaluated and their value is injected into the outgoing HTML stream.

The two pages linked above provide the detailed list of features available for syntax.

### The SLY Element {#the-sly-element}

>[!NOTE]
>
>The SLY element was introduced with AEM 6.1, or HTL 1.1.
>
>Prior to that, the [`data-sly-unwrap`](block-statements.md) attribute had to be used instead.

A central concept of HTL is to offer the possibility of reusing existing HTML elements to define block statements, which avoids the need of inserting additional delimiters to define where the statement starts and ends. This unobtrusive annotation of the markup to transform a static HTML into a functioning dynamic template offers the benefit of not breaking the validity of the HTML code, and therefore to still properly display, even as static files.

However, sometimes there might not be an existing element at the exact location where a block statement has to be inserted. For such cases, it is possible to insert a special SLY element that will be automatically removed from the output, while executing the attached block statements and displaying its content accordingly.

So following example:

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

will output something like following HTML, but only if there are both, a **`jcr:title`** and a **`jcr:decription`** property defined, and if neither of them are empty:

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

One thing to keep in mind is to only use the SLY element when no existing element could have been annotated with the block statement, because SLY elements deter the value offered by the language to not alter the static HTML when making it dynamic.

For example, if the previous example would have been wrapped already inside a DIV element, then the added SLY element would be abusive:

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

and the DIV element could have been annotated with the condition:

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL Comments {#htl-comments}

Following example shows on **line 1** an HTL comment, and on **line 2** an HTML comment:

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL comments are HTML comments with an additional JavaScript-like syntax. The whole HTL comment, and anything within will be entirely ignored by the processor and removed from the output.

The content of standard HTML comments however will be passed through and expressions within the comment will be evaluated.

HTML comments cannot contain HTL comments and vice versa.

### Special Contexts {#special-contexts}

In order to be able to make the best use of HTL, it is important to understand well the consequences of it being based on the HTML syntax.

### Element and Attribute Names {#element-and-attribute-names}

Expressions can only be placed into HTML text or attribute values, but not within element names or attribute names, or it wouldn't be valid HTML anymore. In order to set element names dynamically, the [`data-sly-element`](block-statements.md#element) statement can be used on the desired elements, and to dynamically set attribute names, even setting multiple attributes at once, the [`data-sly-attribute`](block-statements.md#attribute) statement can be used.

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### Contexts Without Block Statements {#contexts-without-block-statements}

As HTL uses data attributes to define block statements, it is not possible to define such block statements inside of following contexts, and only expressions can be used there:

* HTML comments
* Script elements
* Style elements

The reason for it is that the content of these contexts is text and not HTML, and contained HTML elements would be considered as simple character data. So without real HTML elements, there also cannot be **`data-sly`** attributes executed.

This may sound like a big restriction, however it is a desired one, because the HTML Template Language shouldn't be abused to generate output that isn't HTML. The [Use-API for Accessing Logic](getting-started.md#use-api-for-accessing-logic) section below introduces how additional logic can be called from the template, which can be used if it is needed to prepare complex output for these contexts. For instance, an easy way to send data from the back-end to a front-end script, is to have the component's logic to generate a JSON string, which can then be placed in a data attribute with a simple HTL expression.

Following example illustrates the behavior for HTML comments, but in script or style elements, the same behavior would be observed:

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

will output something like following HTML:

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### Explicit Contexts Required {#explicit-contexts-required}

As explained in the [Automatic Context-Aware Escaping](getting-started.md#automatic-context-aware-escaping) section below, one objective of HTL is to reduce the risks of introducing cross-site scripting (XSS) vulnerabilities by automatically applying context-aware escaping to all expressions. While HTL can automatically detect the context of expressions placed inside of HTML markup, it doesn't analyze the syntax of inline JavaScript or CSS, and therefore relies on the developer to specify explicitly what exact context has to be applied to such expressions.

Since not applying the correct escaping results in XSS vulnerabilities, HTL does therefore remove the output of all expressions that are in script and style contexts when the context has not been declared.

Here is an example of how to set the context for expressions placed inside scripts and styles:

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

For more details about how to control the escaping, refer to the [Expression Language Display Context](expression-language.md#display-context) section.

### Lifting Limitations of Special Contexts {#lifting-limitations-of-special-contexts}

In the special cases where it is needed to bypass the restrictions of the script, style and comment contexts, it is possible to isolate their content in a separate HTL file. Everything located in its own file will be interpreted by HTL as a normal HTML fragment, forgetting the limiting context from which it might have been included.

See the [Working with Client-Side Templates](getting-started.md#working-with-client-side-templates) section further down for an example.

>[!CAUTION]
>
>This technique can introduce cross-site scripting (XSS) vulnerabilities, and the security aspects should be carefully studied if this gets used. There usually are better ways to implement the same thing than to rely on this practice.

## General Capabilities of HTL {#general-capabilities-of-htl}

This section quickly walks through the general features of the HTML Template Language.

### Use-API for Accessing Logic {#use-api-for-accessing-logic}

Consider following example:

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

And following `logic.js` server-side executed JavaScript file placed next to it:

```
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

As the HTML Template Language doesn't allow to mix code inside the markup, it offers instead the Use-API extension mechanism to easily execute code from the template.

The example above uses server-side executed JavaScript to shorten the title to 10 characters, but it could also have used Java code to do the same by providing a fully qualified Java class name. Generally, business logic should rather be built in Java, but when the component needs some view-specific changes from what the Java API provide, it can be convenient to use some server-side executed JavaScript to do that.

More about that in following sections:

* The section on the [`data-sly-use` statement](block-statements.md#use) explains everything that can be done with that statement.
* The [Use-API page](use-api.md) provides some information to help choose between writing the logic in Java or in JavaScript.
* And to detail how to write the logic, the [JavaScript Use-API](use-api-javascript.md) and the [Java Use-API](use-api-java.md) pages should help.

### Automatic Context-Aware Escaping {#automatic-context-aware-escaping}

Consider following example:

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

In most template languages, this example would potentially create a cross-site scripting (XSS) vulnerability, because even when all variables are automatically HTML-escaped, the `href` attribute must still be specifically URL-escaped. This omission is one of the most common errors, because it can be very easily forgotten, and it is difficult to spot in an automated manner.

To help with that, the HTML Template Language automatically escapes each variable accordingly to the context in which it is placed. This is possible thanks to the fact that HTL understand the syntax of HTML.

Assuming following `logic.js` file:

```
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

The initial example will then result in following output:

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

Notice how the two attribute got escaped differently, because HTL knows that `href` and `src` attributes must be escaped for URI context. Also, if the URI started with **`javascript:`**, the attribute would have been removed entirely, unless the context were explicitly changed to something else.

For more details about how to control the escaping, refer to the [Expression Language Display Context](expression-language.md#display-context) section.

### Automatic Removal of Empty Attributes {#automatic-removal-of-empty-attributes}

Consider following example:

```xml
<p class="${properties.class}">some text</p>
```

If the value of the `class` property happens to be empty, the HTML Template Language will automatically remove the entire `class` attribute from the output.

Again, this is possible, because HTL understand the HTML syntax and can therefore conditionally show attributes with dynamic values only if their value isn't empty. This is extremely convenient as it avoids to add a condition block around attributes, which would have made the markup invalid and unreadable.

Additionally, the type of the variable placed in the expression matters:

* **String:**
  * **not empty:** Sets the string as attribute value.
  * **empty:** Removes the attribute altogether.

* **Number:** Sets the value as attribute value.  

* **Boolean:**
  * **true:** Displays the attribute without value (as a Boolean HTML attribute
  * **false:** Removes the attribute altogether.

Here's an example of how a Boolean expression would allow to control a Boolean HTML attribute:

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

For setting attributes, the [`data-sly-attribute`](block-statements.md#attribute) statement might also be useful.

## Common Patterns with HTL {#common-patterns-with-htl}

This section introduces a few common scenarios and how to best solve them with the HTML Template Language.

### Loading Client Libraries {#loading-client-libraries}

In HTL, client libraries are loaded through a helper template provided by AEM, which can be accessed through [`data-sly-use`](block-statements.md#use). Three templates are available in this file, which can be called through [ `data-sly-call`](block-statements.md#template-call):

* **`css`** - Loads only the CSS files of the referenced client libraries.
* **`js`** - Loads only the JavaScript files of the referenced client libraries.
* **`all`** - Loads all the files of the referenced client libraries (both CSS and JavaScript).

Each helper template expects a **`categories`** option for referencing the desired client libraries. That option can be either an array of string values, or a string containing a comma separated values list.

Here are two few short examples:

### Loading multiple client libraries fully at once {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### Referencing a client library in different sections of a page {#referencing-a-client-library-in-different-sections-of-a-page}

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

In the second example above, in case the HTML **`head`** and **`body`** elements are placed into different files, the **`clientlib.html`** template would then have to be loaded in each file that needs it.

The section on the [template & call](block-statements.md#template-call) statements provides more details about how declaring and calling such templates work.

### Passing Data to the Client {#passing-data-to-the-client}

The best and most elegant way to pass data to the client in general, but even more with HTL, is to use data attributes.

Following example shows how the logic (which could also be written in Java) can be used to very conveniently serialize to JSON the object that is to be passed to the client, which can then very easily be placed into a data attribute:

```xml
<!--/* template.html file: */-->
<div data-sly-use.logic="logic.js" data-json="${logic.json}">...</div>
```

```
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

From there, it is easy to imagine how a client-side JavaScript can access that attribute and parse again the JSON. This would for instance be the corresponding JavaScript to place into a client library:

```
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### Working with Client-Side Templates {#working-with-client-side-templates}

One special case, where the technique explained in the section [Lifting Limitations of Special Contexts](getting-started.md#lifting-limitations-of-special-contexts) can legitimately be used, is to write client-side templates (like Handlebars for instance) that are located within **script** elements. The reason this technique can safely be used in that case, is because the **script** element then doesn't contain JavaScript as assumed, but further HTML elements. Here's an example of how that would work:

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

As shown above, the markup that will be included into the **`script`** element can contain HTL block statements and the expressions don't need to provide explicit contexts, because the content of the Handlebars template has been isolated in its own file. Also, this example shows how server-side executed HTL (like on the **`h2`** element) can be mixed with a client-side executed template language, like Handlebars (shown on the **`h3`** element).

A more modern technique altogether, would however be to use the HTML **`template`** element instead, as the benefit would then be that it doesn't require to isolate the content of the templates into separate files.

**Read next:**

* [Expression Language](expression-language.md) - to learn in detail what that can be done within HTL expressions.
* [Block Statements](block-statements.md) - to discover all the block statements available in HTL, and how to use them.
