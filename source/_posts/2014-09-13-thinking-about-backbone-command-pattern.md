---
title: Bridging The Gap Between CRUDy Backbone.js and A CQRS Rest API: Commanding
tags:
  - backbone
  - js
  - ddd
  - commanding
  - rest api
  - webdev
  - CQRS
categories:
  - webdev
---

Lately I have been working with a commanding pattern and CQRS with sever-side code. In the context of a rest api a
commanding pattern fits quite nicely. Lets take the example of marking a product as active.

 > Traditional CRUD Rest API:
PUT /products/123 {active: true}

 > Commanding Rest API:
POST /products/123/activate

But this isn't the full story. Usually, when use the PUT verb, you have to send the entire entity. This leads to a verbose
HTTP request that obfuscates what you are actually doing. However, When you send the POST verb to send the activate
command, it is much more concise and expressive to what you are actually doing. But the POST verb is intended to create
entities you say? Well, aren't we just CREATING a command?

This paradigm of using the POST verb to create commands is really quite slick. It creates an API with expressive and
meaningful endpoints. For me, there was only one problem: My project is heavily invested in Backbone. Backbone is
designed to communicate with a rest api to Create, Update, and Delete entities. I've been thinking about a way to bridge
this gap between client side CRUD operations and server side commanding operations.

There are some examples of commanding pattern in JavaScript. These examples, in my opinion, are a little heavy handed.
I'm not looking to create a command bus in JavaScript. When I'm working with Backbone objects, I usually already have a
collection or model in that I am trying to manipulate. I’m thinking about extending Backbone to allow for commands to be
sent. Here is some pseudo code for what I’m thinking:

    var Commanding = {
        execute: function (command, context) {
            var self = this;
            this.trigger('command before:' + command, [self, command, context]);
            return $.ajax({
                url: _.result(this, 'url'),
                method: 'POST',
                data: context,
                always: function (data, status, xhr)
                {
                    var events = 'command: ' + status + ' ' + status + ':' + command;
                    self.trigger(events, [self, command, context, xhr, data, status]);
                }
            });
        }
    }

    _.extend(Backbone.Model.prototype, Commanding);
    _.extend(Backbone.Collection.prototype, Commanding);

    exampleModel.execute('activate');
    //triggers command and before:activate events
    //sends POST /products/{id}/activate ajax request
    //triggers command:success success:activate events
    //returns xhr object, so you can chain fail success/failure callbacks





