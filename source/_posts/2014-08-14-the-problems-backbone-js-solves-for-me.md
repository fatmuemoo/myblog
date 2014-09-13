---
title: The Problems Backbone.js Solves for Me
tags:
  - backbone
  - js
  - webdev
categories:
  - webdev
---

For my day job, I was tasked with writing some documentation regarding the way we use certain libraries to get the job
done. This was a very interesting task for me, simply because I "know" the problems that Backbone.JS is solving, but I
never stopped to really try to articulate it.

First a little background: We are writing a data centric single page web app. I have written single page apps using
jQuery in the past. Although plain jQuery works, and IMO its better then writing Vanilla JS, it lacks in many areas.
I have a one app that I still have to maintain, that essentially is a 15,000 line jQuery plug-in. What a headache!

Anyway, back to the point, the problems that Backbone.JS solves for me is three fold:

 1.  Routing/Front Controller using Backbone.Router
 2.  Views, Views, Views using Backbone.View
 3.  Data Storage and AJAX negotiation using Backbone.Collection and Backbone.Model
 4.  **BONUS:** Better separation of concerns and code organization.


Backbone.Router
---------------

The obvious use of the Backbone.Router component is mapping hashed URL elements to an action.
Code is worth a thousand words:

~~~js
var Router = Backbone.Router.extend({
    routes: {
        //define a route
        "help/:topic": "help"
    },
    //define the help action
    help: function (topic) {
        //build some model based on the the topic
        //....
        //pass the model to a view, render the view
    }
});
~~~

This is perfectly acceptable use of the Router, but when you do this, your router is going to get filled really
fast with routes and actions. In addition, you will end up having the main router polluted with module specific code.
For this reason, I liken my main router as more of Front Controller. The way I achieve this is I have a namespace the
app call Application.Sections. This object is populated with Section objects. Perhaps some code will help:

~~~js
var Router = Backbone.Router.extend({
    routes: {
        //define a route
        "sections/:section": "routeToSection"
    },
    //define the help action
    routeToSection: function (section) {
        if (typeof Application.Sections[section] === 'function') {
            new Application.Sections[section]();
        }
    }
});
~~~

Now in practice the Application.Sections objects are an extension of Backbone.View. These views take over after the
router creates them. Perhaps Front Controller description is not correct, but the point is that the router becomes a
thin mapper to a view layer. By modularizing the router, you can easily separate the section logic from the router,
enabling the code for sections to live outside of the context of the router. This makes the app easier to maintain and
organize.

Backbone.View
--------------

The Backbone.View documentation states:

> Backbone views are almost more convention than they are code — they don't determine anything about your HTML or CSS
for you, and can be used with any JavaScript templating library. The general idea is to organize your interface into
logical views, backed by models...

This is a really important statement. Backbone.View makes no assumptions for you. They provide some initialization code
that excepts a model, information for its root HTML node and some convention for delegating DOM events, but thats about
it. In practice, I use views to initiate new models/collections and create sub-views.

As an example, the Section View looks something like this:

~~~js
Application.Section.FooSection = Backbone.View.extend({
    initialize: function () { this.render(); },
    events: {
	'click button': function () {
            //do something when you click buttons
        }
    }
    render: function () {
        //build some dom, call template engine, etc
        ....
        //Create some subviews and collections/models specific to this section
    }
});
~~~

This is just outer layer of what Backbone.Views does for me. Lets look at some pretty ASCII art to explain the rest:

<pre>
..................................................
.                                                .
.             [main section view]                .
.  -------------------------   ----------------- .
.  |       [content view]   |  |  [side view]  | .
.  |                        |  |  ............ | .
.  |                        |  |  . [widget] . | .
.  |                        |  |  ............ | .
.  |                        |  |               | .
.  |                        |  |  ............ | .
.  |                        |  |  . [widget] . | .
.  |                        |  |  ............ | .
.  -------------------------   ----------------- .
..................................................
</pre>

Given the above scenario, the Main Section View is responsible for initializing the Content View and and the side view.
The side view is responsible for rending the widgets. Each parent view may also be responsible for initializing
Backbone.Collections or Backbone.Models for its children.

[How does the view build HTML? That is a topic for another post](/blog/2014/08/20/making-the-case-for-client-side-templates/)

Backbone.Collection / Backbone.Model
-------------------------------------

Writing a single page web app is essentially just putting a UI on top of a REST API. Lets take the classic example of
CRUD operations for products:


<table class="table">
<thead>
<tr>
  <th>Verb</th>
  <th>URL</th>
  <th></th>
</tr>
</thead>
<tbody>
<tr>
  <td>GET</td>
  <td>/api/products</td>
  <td>Gets a list of products</td>
</tr>
<tr>
  <td>GET</td>
  <td>/api/products{id}</td>
  <td>Get a product with the given ID</td>
</tr>
<tr>
  <td>POST</td>
  <td>/api/products</td>
  <td>Create a product</td>
</tr>
<tr>
  <td>PUT</td>
  <td>/api/products{id}</td>
  <td>Update a product for the given ID</td>
</tr>
</tbody>
</table>

Backbone excels at handling AJAX negotiations with REST endpoints. Under the hood, its really just a wrapper around
jQuery AJAX api, but that is exactly what is needed in order to keep our "Model" separate from our view logic. Lets see
and example of how to deal with these reset endpoints using backbone:

~~~js
    var Products = Backbone.Collection.extend({
        url: "/api/products"
    });
    var products = new Products();
~~~

GET /api/products

~~~js
    //fetch the products from /api/products
    products.fetch();
~~~

GET /api/products{id}

~~~js
    //gets product with ID 1, if it already exists in the collection.
    var p = products.get(1);
    //if it is undefined, lets add it and then fetch it
    if (typeof p === 'undefined') {
       p = products.add({id: 1});
       //fetch product info from /api/products/1
       p.fetch();
    }
~~~

POST /api/products

~~~js
    //sends post to /api/products
    var newProduct = products.create({
       name: 'foo',
       product_nbr: '123'
    });
~~~

PUT /api/products{id}

~~~js
    //update name attribute to foo
    p.set('name', 'foo');
    //sends PUT to /api/products/1
    p.save();
~~~

Summary
-------

In a nutshell, Backbone.JS provides component code that allows for clean separation of concerns. By calling external
objects in the Router, your router can be thin mapper to views. Views can be nested, simplifying the views and making
the views more reusable. By injecting models or collections into views, you can keep data "model" logic separate from
view logic. [In a related post](/blog/2014/08/20/making-the-case-for-client-side-templates/), I discuss how to render HTML with templates.

