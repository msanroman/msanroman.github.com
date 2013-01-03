---
layout: post
title: Maybe the problem isn't "Fat Model"
permalink: maybe-the-problem-is-not-fat-model
published: true
---

*(well, it isn't the root of it)*

## Madrid.rb

Since I am in Madrid during Christmas, I attended the event organized yesterday by [madrid.rb](https://madridrb.jottit.com/). If you don't know them, it's a Ruby user group in Madrid, and they share knowledge and beers on a monthly basis.

If you like Ruby, beers and it turns out you're in Madrid, I sincerely recommend you to step by one of their events. It's always a pleasure to have a little chat and share your point of view with the others.

Yesterday's talk was about Fat Models, or the inherent problems caused by following the "Skinny Controller, Fat Model" Ruby on Rails' practice. The speaker, [Luismi Cavall&eacute;](http://twitter.com/cavalle) did a great job talking about this problematic and the different solutions offered by the Rails community.

This is an issue in everyone's lips right now, and I loved watching the proactivity of the community sharing problems, tips and tricks.

## ActiveRecord feels wrong to me

The *Fat Model, Skinny Controller* thing is great under the point of view that business logic doesn't belong to the controller, but I'm sorry mixing persistence with business logic feels wrong to me. Why? Mainly because your classes:

+ Have now **two** responsibilities, persist and *do-whatever-they-say-they-do*
+ There won't be any unit test for them, since you are coupled to the database. As a side effect:
    + Your tests will be brittle, they can break now by a business logic problem or by a persistence problem.
    + Your test suite is gonna run up to 10/15/20 minutes. Are you gonna run them always after each change, each refactor?
    + Your tests won't be the safety net they are supposed to be.
+ You will end up being unable to manage that big, fat model.

[Almost all] applications aren't *only* a CRUD. They are much more. They have exclusive business rules and most of them don't have to do anything about a database, nor a deploying system, nor a framework. Those are implementation details, and you should decide about them in your last responsible moment.

And I think **Rails doesn't help with this point of view**.

Don't get me wrong. **I love Rails**, but being a database-centric framework makes unconfortable form me to try to *hack* against the *Rails-way* when I try to keep my domain logic and my persistence rules and validators separated.

Yeah, I know, you can keep that domain logic out of the model creating modules for that logic, or using the new ActiveSupport::Concern thing. It will make that logic easier to test, and your models will look skinnier than ever.

But you still have your models and domain coupled to the framework, and that looks dangerous to me. Specially if your test suite is not powerful enough to guarantee that after a change in the framework, or in the database (or any other *implementation detail*) your domain is still working.

## Not-so-skinny-controller? Really?

I agree with DHH on [one of his last posts'](http://david.heinemeierhansson.com/2012/emails-are-views.html) title: keep emails out of the models! But (IMHO) the controller is not the place to put them.

The controller must handle requests and return responses **nothing more**. They should call to certain domain modules depending on the request they are handling at that moment, and return the response that domain module sent them.

Sending emails or managing subscriptions doesn't seem like a controller's job.

## Wrap up

I still have to do more experiments and detect what's the real pain for me there. I just kept feeling like the topic we were discussing yesterday was mainly because we choose a framework when we start developing our application, and not when we really need it.

All in all, yesterday's meetup was great, and I had the pleasure to meet some people like [@ArturoHerrero](https://twitter.com/ArturoHerrero), [@cavalle](https://twitter.com/cavalle), [@ecomba](https://twitter.com/ecomba), [@eLobatoss](https://twitter.com/elobatoss), [@germandz](https://twitter.com/germandz), [@pasku1](https://twitter.com/pasku1) and much more. We didn't talk only about this problem, and each topic we talked was enriching for me.

So thanks to all of you!