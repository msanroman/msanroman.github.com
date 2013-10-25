---
layout: post
title: Building a Backbone application with CoffeeScript and Jasmine (Pt. I)
permalink: 'pomodori-js-part-one'
published: true
---

<!-- > This is my first post in English, lately I've noticed that my English was getting kinda rusty, and decided to change it. In brief: _I'm sorry_ :p
 -->
## Prelude

A few weeks ago I went to the second edition of [We Love JS](http://welovejs.es/), and it was a blast! Not only [@cvillu](https://twitter.com/#!/cvillu) did show the awesomeness of CoffeeScript again, but also [@micho](https://twitter.com/#!/micho) did a great Backbone workshop that made me want to try out even more this framework.

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

{% highlight coffeescript %}
describe 'Task', ->

  it "should exist", ->
    expect(PomoJS.Models.Task).toBeDefined()
{% endhighlight %}

This spec will fail because there's nothing defined in our application yet! We should add a _Models_ collection or namespace and a _Task_ class in that collection.

So, in order to do it:

{% highlight coffeescript %}
#app.coffee
 
window.PomoJS = {
  Models: {}
}
 
#Task.coffee
 
class PomoJS.Models.Task
{% endhighlight %}

Now our first test should pass! Now we can start defining some default values for our tasks' attributes, like:

{% highlight coffeescript %}
describe 'Task', ->
 
  beforeEach ->
    @task = new PomoJS.Models.Task
 
  it "should exist", ->
    expect(PomoJS.Models.Task).toBeDefined()
 
  describe "default values for new tasks", ->
    it "should have an empty string as default name", ->
      expect(@task.get "name").toEqual ""
 
    it "shouldn't be completed", ->
      expect(@task.get "completed").toEqual false
 
    it "should have no pomodoros assigned as default estimation", ->
      expect(@task.get "estimation").toEqual 0
{% endhighlight %}

It will fail, announcing us that our object has no method _'get'_, so now we must make our class Task extend from Backbone.Model, and our tests will fail for the right reason: our attributes are not defined, and naturally have no default value, so we must define our _defaults_ like this:

{% highlight coffeescript %}
class PomoJS.Models.Task extends Backbone.Model
 
  defaults: {
    name: '',
    completed: false,
    estimation: 0
  }
{% endhighlight %}

Green again, now we can add an abstraction to this _get "element"_ call creating a getter for each of our attributes, and verify that this methods call the _get_ method with the right value:

{% highlight coffeescript %}
describe 'Task', ->
 
  #(...)
 
  describe "getters", ->
 
    describe "getId", ->
 
      it "should be defined", ->
        expect(@task.getId).toBeDefined()
      it "should return model's id", ->
        stub = sinon.stub(@task, 'get').returns 1
 
        expect(@task.getId()).toEqual 1
        expect(stub.calledWith('id')).toBeTruthy()
 
    describe "getName", ->
 
      it "should be defined", ->
        expect(@task.getName).toBeDefined()
 
      it "should return its name", ->
        spyOn(@task, 'get').andReturn 'Trololo!'
 
        expect(@task.getName()).toEqual 'Trololo!'
        expect(@task.get).toHaveBeenCalledWith('name')
 
    describe "isCompleted", ->
 
      it "should be defined", ->
        expect(@task.isCompleted).toBeDefined()
      it "should return value for the completed attribute", ->
        spyOn(@task, 'get').andReturn(false)
 
        expect(@task.isCompleted()).toEqual false
        expect(@task.get).toHaveBeenCalledWith('completed')
 
    describe "getEstimation", ->
 
      it "should be defined", ->
        expect(@task.getEstimation).toBeDefined()
      it "should return its estimation", ->
        spyOn(@task, 'get').andReturn 5
 
        expect(@task.getEstimation()).toEqual 5
        expect(@task.get).toHaveBeenCalledWith("estimation")
{% endhighlight %}
Then we just create these methods in our Task class:

{% highlight coffeescript %}
class PomoJS.Models.Task extends Backbone.Model
 
  defaults: {
    name: '',
    completed: false,
    estimation: 0
  }
 
  getId: -> @.get 'id'
 
  getName: -> @.get 'name'
 
  isCompleted: -> @.get 'completed'
 
  getEstimation: -> @.get 'estimation'
{% endhighlight %}

And now here comes one of the juiciest parts from our model's description: we must define the behaviour for the _save_ method of our model. It should send a request to our server, sending it a JSON object with our task's contents. For that, I use the marvelous sinon's fakeServer, and just capture what is being sent:

{% highlight coffeescript %}
describe "save", ->
 
    beforeEach ->
      @server = sinon.fakeServer.create()
 
    afterEach ->
      @server.restore()
 
    it 'sends valid data to the server', ->
      @task.save {name: 'new task name', estimation: 1}
 
      request = @server.requests[0]
 
      params = JSON.parse(request.requestBody)
      expect(params.task).toBeDefined()
      expect(params.task.name).toEqual 'new task name'
      expect(params.task.complete).toBeFalsy()
      expect(params.task.estimation).toEqual 1
{% endhighlight %}

We will get an error message telling us: _A "url" property or function must be specified_ - so lets create this property in our Task class, an let it be "/tasks", and now lets write a toJSON method for our class, because it will get called when calling the _save_ method:

{% highlight coffeescript %}
class PomoJS.Models.Task extends Backbone.Model
 
  url: "/tasks"
 
  defaults: {
    name: '',
    completed: false,
    estimation: 0
  }
 
  getId: -> @.get 'id'
 
  getName: -> @.get 'name'
 
  isCompleted: -> @.get 'completed'
 
  getEstimation: -> @.get 'estimation'
 
  toJSON: -> {task: @.attributes}
{% endhighlight %}

We have defined a value for our url property, but we should define it better for some different actions:

+ On task creation: POST "/tasks"
+ On task update: PUT "tasks/{task_id}"

{% highlight coffeescript %}
    describe 'server requests', ->
 
      describe 'on create', ->
 
        beforeEach ->
          new_task = new PomoJS.Models.Task()
          new_task.save()
          @request = @server.requests[0]
 
        it 'should be POST', ->
          expect(@request.method).toEqual 'POST'
 
        it 'should have /tasks as url', ->
          expect(@request.url).toEqual '/tasks'
 
      describe 'on update', ->
 
        beforeEach ->
          @task.save(id: 13)
          @request = @server.requests[0]
 
        it 'should be PUT', ->
          expect(@request.method).toEqual 'PUT'
 
        it 'should have /tasks/13 as url', ->
          expect(@request.url).toEqual '/tasks/13'
{% endhighlight %}

So let's modify our url property to be a method like this:

{% highlight coffeescript %}
class PomoJS.Models.Task extends Backbone.Model
 
  url: ->
    url = "/tasks"
    url += "/#{@.getId()}" unless @.isNew()
    return url
 
  defaults: {
    name: '',
    completed: false,
    estimation: 0
  }
 
  getId: -> @.get 'id'
 
  getName: -> @.get 'name'
 
  isCompleted: -> @.get 'completed'
 
  getEstimation: -> @.get 'estimation'
 
  toJSON: -> {task: @.attributes}
{% endhighlight %}

Now, we can define some attribute validation: we can't save a task with a negative id, nor one with a negative estimation:

{% highlight coffeescript %}
    it "won't save with negative id", ->
      @task.set({id: -1})
      expect(@task.isValid()).toBeFalsy()
 
    it "won't save with negative estimation", ->
      @task.set({estimation: -2})
      expect(@task.isValid()).toBeFalsy()
{% endhighlight %}

And to make it pass:

{% highlight coffeescript %}
class PomoJS.Models.Task extends Backbone.Model
 
  url: ->
    url = "/tasks"
    url += "/#{@.getId()}" unless @.isNew()
    return url
 
  defaults: {
    name: '',
    completed: false,
    estimation: 0
  }
 
  getId: -> @.get 'id'
 
  getName: -> @.get 'name'
 
  isCompleted: -> @.get 'completed'
 
  getEstimation: -> @.get 'estimation'
 
  validate: (attributes) -> 
    if @.getId() < 0
      return 'invalid id'
    if @.getEstimation() < 0
      return 'invalid estimation'
 
  toJSON: -> {task: @.attributes}
{% endhighlight %}

Yeah, I know, the validate method contains some ugly code duplication - we'll get into refactoring that later. But for now, I think that this post has grown far too long than I'd expected and that offers quite a bunch of information of how to test some Backbone models and how they work.

In my next post, I'll write about Backbone's collections and routers.

Stay tuned!