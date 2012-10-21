---
layout: post
title: Building a Backbone application with CoffeeScript and Jasmine (Pt. I)
---

> This is my first post in English, lately I've noticed that my English was getting kinda rusty, and decided to change it. In brief: _I'm sorry_ :p

## Prelude

A few weeks ago I went to the second edition of [We ♥ JS](http://welovejs.es/), and it was a blast! Not only [@cvillu](https://twitter.com/#!/cvillu) did show the awesomeness of CoffeeScript again, but also [@micho](https://twitter.com/#!/micho) did a great Backbone workshop that made me want to try out even more this framework.

I've tried two technologies in order to build my app test-driving it:

+ Brunch & Mocha
+ Jasmine & sinon

I've found that Brunch is a really awesome HTML5 application assembler, agnostic to frameworks, libraries, even stylesheet and templating languages. It's easy to get started, and provides some watchers _out-of-the-box_ that build continously the application, launches a local server, etc. The fact that Mocha got me hooked seeing [@cvillu](https://twitter.com/#!/cvillu) using it at his CoffeeScript talk made me want to try it along with brunch.

There's just one problem - it's not as easy as it seems to make work Brunch, Backbone and Mocha together (or at least a _noob_ like me wasn't able to do it), mainly because right now there's no testing support on Brunch (it was planned for the 1.2 release, but has been postponed to the 1.3).

So, the main solution I found was using Jasmine HtmlSpecRunner, and add my compiled _.coffee_ files (spec and production code) to this HTML.

You might notice that I won't describe much on _how_ Backbone works, it's documentation is great and I'd just do a very poor job trying to show you that. Instead, I prefer showing you how was my flow of development with this tools.

## The application

One of the main examples for a Backbone application is a [_todo_ list](http://documentcloud.github.com/backbone/examples/todos/index.html): a simple application that lets you create tasks with an input field and mark them as completed on the task list. It seemed kinda cool, but as you know I'm a pomodoro guy, and one of the apps that I try to build when learning new things is a pomodoro task manager, so my app not only should save a list of items, but also save an estimation in pomodoros of each one and be able to launch the estimated pomodoros for a selected task (but lets leave it for the 1.1 release :P).

You can find my code at the [GitHub repo](http://github.com/msanroman/pomodori.js), so you'll be able to see there which is the set up for the application.

## Let's start coding!

Let's start defining our Task class:

<script src="https://gist.github.com/2690652.js"> </script>

This spec will fail because there's nothing defined in our application yet! We should add a _Models_ collection or namespace and a _Task_ class in that collection.

So, in order to do it:

<script src="https://gist.github.com/2690674.js"> </script>

Now our first test should pass! Now we can start defining some default values for our tasks' attributes, like:

<script src="https://gist.github.com/2690689.js"> </script>

It will fail, announcing us that our object has no method _'get'_, so now we must make our class Task extend from Backbone.Model, and our tests will fail for the right reason: our attributes are not defined, and naturally have no default value, so we must define our _defaults_ like this:

<script src="https://gist.github.com/2690702.js"> </script>

Green again, now we can add an abstraction to this _get "element"_ call creating a getter for each of our attributes, and verify that this methods call the _get_ method with the right value:

<script src="https://gist.github.com/2690712.js"> </script>

Then we just create these methods in our Task class:

<script src="https://gist.github.com/2690722.js"> </script>

And now here comes one of the juiciest parts from our model's description: we must define the behaviour for the _save_ method of our model. It should send a request to our server, sending it a JSON object with our task's contents. For that, I use the marvelous sinon's fakeServer, and just capture what is being sent:

<script src="https://gist.github.com/2690750.js"> </script>

We will get an error message telling us: _A "url" property or function must be specified_ - so lets create this property in our Task class, an let it be "/tasks", and now lets write a toJSON method for our class, because it will get called when calling the _save_ method:

<script src="https://gist.github.com/2690757.js"> </script>

We have defined a value for our url property, but we should define it better for some different actions:

+ On task creation: POST "/tasks"
+ On task update: PUT "tasks/{task_id}"

<script src="https://gist.github.com/2690789.js"> </script>

So let's modify our url property to be a method like this:

<script src="https://gist.github.com/2690794.js"> </script>

Now, we can define some attribute validation: we can't save a task with a negative id, nor one with a negative estimation:

<script src="https://gist.github.com/2690799.js"> </script>

And to make it pass:

<script src="https://gist.github.com/2690802.js"> </script>

Yeah, I know, the validate method contains some ugly code duplication - we'll get into refactoring that later. But for now, I think that this post has grown far too long than I'd expected and that offers quite a bunch of information of how to test some Backbone models and how they work.

In my next post, I'll write about Backbone's collections and routers.

Stay tuned!