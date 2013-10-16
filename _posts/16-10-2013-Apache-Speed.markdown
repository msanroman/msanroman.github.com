---
layout: post
title: 'Some Apache and hosts settings that improved my "localhost" browsing speed'
permalink: 'settings-apache-hosts-localhost-speed'
published: true
---

These tips at least worked for me and some of my coworkers at [Runroom](http://www.runroom.com), if you know any other quick tip, or wanna tell me if it worked for you or not, just tweet me back and I'll update this post:

##Apache settings

Just replace (or create) the following lines in your Apache config file:

{% highlight apacheconf %}
Listen localhost:80
ServerName 127.0.0.1
{% endhighlight %}

##Hosts' file

In your hosts file, put your local server names in one line, and delete (or comment) your IPv6 line like this:

{% highlight bash %}
127.0.0.1 localhost.localdomain localhost servernamea.local servernameb.local servernamec.local
127.0.0.1 127.0.0.1
{% endhighlight %}

I don't know why or how it works, but these two lines worked like *pure magic*.

Let me know if it worked for you!