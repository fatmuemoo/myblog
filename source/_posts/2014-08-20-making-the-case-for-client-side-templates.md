---
title: Making The Case For Client Side Templates
tags:
  - backbone
  - js
  - templates
  - twig.js
  - webdev
categories:
  - webdev
---

Context: We are building a single page web app with backbone. The Backbone.View object has a render method. The intent
of the render method is to provide a place for the developer to render the view, by whatever means the developer wants.

Question: Why use client side templates? The reason is simple: in the javascript world, you have 3 options for building HTML:

 1. Concatenate strings
 2. Build DOM programmaticly
 3. Use a template engine

Let's pose the following scenario: Given an array of products, loop through and display the products.

~~~js
var product = [
	{name: "Chicken Strips", product_nbr: "abc123"}
	{name: "Cow on a stick", product_nbr: "97208"}
];
~~~

The following 3 code blocks are psuedo code for what the render function might look like:

Concatenating Strings
---------------------

~~~js
var str = "<div><h1>Products</h1>";
for (i in products) {
    str += "<ul>;
    str += "<li>Name:";
    str += products[i].name;
    str += "</li><li>Nbr:";
    str += products[i].product_nbr;
    str += “</li></ul>”;
}
str += “</div>”

this.$el.html(str);
~~~

Building DOM (jQuery)
---------------------

~~~js
var $div = $("<div>");
$div.append("<h1>Products</h1>"
$.each(products, function () {
     $div.append(
        $("<ul>").append($("<li>").text("Name: " + this.name))
                 .append($("<li>").text("Nbr: " + this.product_nbr));
    );
});

this.$el.append($div);
~~~

Templating (Twig.JS)
--------------------

~~~js
this.$el.html(twig({ ref: "products" }).render({products:products}));
~~~

**Products Template**
~~~html
{% verbatim %}<div>
   <h1>Products</h1>
   {% for p in products %}
      <ul>
          <li>Name: {{ p.name }} </li>
          <li>Name: {{ p.product_nbr }}</li>
      </ul>
  {% endfor %}
</div>{% endverbatim %}
~~~

Concatenating strings gets ugly really fast. Building DOM is the least verbose, but it is error prone and incredibly
difficult to maintain  for anything but the simplest DOM structures. Just imagine changing the UL to table rows. It
would not be fun. Templating is the most concise, easiest to read, and easiest to maintain. In this example, there is
some bootstraping code that isn't shown for loading the templates from an external source (this is a topic for another post.)
 By loading templates as needed, you can keep your JavaScript code clean and concise. No need for your render method to
 be polluted with a bunch of DOM logic. Backbone.View then becomes a wrapper around your templating engine with
 functionality for listing to model changes and user events.

A Few Templating Engines
========================

>**Disclaimer:** Things move really fast. These template engine options may not be the best options out there
right now. These are just the ones I have evaluated.

My Requirements for a Templatint Engine are

 1. Some logic such as loops and conditionals
 2. Flexible loading from external files or inline strings
 3. Simple syntax

Below are a few of the Templating Engines I have evaluated and how to they work to render HTML.

Underscore.JS Templates
---------------------

Because [Underscore.JS](http://underscorejs.org/) is a dependency of Backbone, it ships with Backbone. It is pretty
simple and powerful. _.template takes the template as a string and returns a template function that will then take the
variables for template and return the rendered template.

~~~js
var compiled = _.template("hello: <%= name %>");
compiled({name: 'moe'});
// returns "hello: moe"
~~~

In order to get this to work in practical way inside a larger application, you will need a  way to load
the initial template strings. You could load the templates via ajax(not shown here) or have them inline using a script tag:


~~~html
<script type="text/template" id="template-hello-world">
    <h1>Hello: <%= name %></h1>
</script>
~~~
~~~js
var compiled = _.template($("#template-hello-world").html());
compiled({name: 'moe'});
// returns "hello: moe"
~~~

Conditionals and loops are achieved using JavaScript. While this is powerful, it really can be a pain in the butt:

~~~html
<script type="text/template" id="template-products">
    <div>
       <h1>Products</h1>
       <% _.each(products, function(p) { %>
          <ul>
              <li>Name: {{ p.name }} </li>
              <li>Name: {{ p.product_nbr }}</li>
          </ul>
       <% }); %>

       <% if(someConditional) { %>
           Print some string
       <% } %>
</script>
~~~

While using javascript inline is powerful, it isn't very easy to read and can get messy quick.

Mustache
--------

[Mustache](http://mustache.github.io/) is pretty neat. It has implementations in a lot of languages. Its syntax is
relatively easy to read. You can load the templates in similar fashion as our first example:
~~~html
<script type="text/template" id="template-hello-world">
    <h1>Hello: {{'{{ name }}'}}</h1>
</script>
~~~
~~~js
var templateString = $("#template-hello-world").html();
Mustache.render(templateString, {name: 'moe'});
// returns "hello: moe"
~~~

Looping is little loopy (no pun intended):

~~~html
<script type="text/template" id="template-products">
{% verbatim %}<div>
   <h1>Products</h1>
   {{#products}}
      <ul>
          <li>Name: {{ name }} </li>
          <li>Name: {{ product_nbr }}</li>
      </ul>
   {{/products}}
</div>{% endverbatim %}
</script>
~~~

Loops are a nice feature. But what about conditionals? The docs say:
>We call it "logic-less" because there are no if statements, else clauses, or for loops. Instead there are only tags. Some tags are replaced with a value, some nothing, and others a series of values.

So NOOP! No conditionals for you! It does make up for this by allowing you to pass in a function as a variable.

Handlebars
----------
[Handlebars.JS](http://handlebarsjs.com/) Is also a very good option. Again, Hello World:
~~~html
<script type="text/template" id="template-hello-world">
    <h1>Hello: {{'{{ name }}'}}</h1>
</script>
~~~
~~~js
var templateString = $("#template-hello-world").html();
var compiled = Handlebars.compile(templateString);
compiled({name: 'moe'});
// returns "hello: moe"
~~~

Like Mustache, Handlebars uses the a [tag based system for loops and functions](http://handlebarsjs.com/block_helpers.html),
but has conditionals build in:

~~~html
<script type="text/template" id="template-products">
{% verbatim %}<div>
   <h1>Products</h1>
   {{#with products}}
      <ul>
          <li>Name: {{ name }} </li>
          <li>Name: {{ product_nbr }}</li>
      </ul>
   {{/with products}}
   {{#if someConditional}}
        render something
   {{/if}
</div>{% endverbatim %}
</script>
~~~

Twig.JS
-------

[Twig.JS](https://github.com/justjohn/twig.js/wiki) is a JS implementation of the PHP Templateing engine
[Twig](http://twig.sensiolabs.org/). Twig.JS implements a subset of the PHP Twig features, but all the necessary ones
are here. Like the PHP implementation, you can have custom filters, functions and tags. If you already use Twig for
PHP templates, Twig.JS is a no brainer.

~~~html
<script type="text/template" id="template-hello-world">
    <h1>Hello: {{'{{ name }}'}}</h1>
</script>
~~~
~~~js
var templateString = $("#template-hello-world").html();
var compiled = twig({ data: templateString });
compiled.render({name: 'moe'});
// returns "hello: moe"
~~~

Summary
=======

In a large JavaScript application, templates make the presentation code easier to maintain and easier to read.
Most template engines are very similar but there differences are notable.
Take time to evaluate your options and pick the best one for your project.

In another post, I will provide details on how to abstract away the differences in different templating engines in
your Backbone.View.