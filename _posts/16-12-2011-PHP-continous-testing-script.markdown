---
layout: post
title: 'PHP continous testing script for NotifyOSD systems'
permalink: 'php-autotest'
published: true
---

[@ArturoHerrero](https://twitter.com/#!/ArturoHerrero) published in [his blog](http://arturoherrero.com/2011/04/08/groovy-autotest-on-ubuntu/) a very interesting post about how he built a test-runner script using NotifyOSD, which is notification system used in Ubuntu, KUbuntu, Fedora and much more.

When I've started writing tests for a Symfony 2 project (maybe I should write more about them in another post) I also started missing a continous testing tool like Infinitest, so I decided to take a look at Arturo's post and build my own version for Fedora, which is the OS that I use lately.

So here's the code!

{% highlight bash linenos %}
#!/bin/sh
 
allTestsOkIcon=/usr/share/icons/gnome/48x48/status/weather-clear.png
allTestsOkTitle="All tests passed"
allTestsOkMessage="Arr! Ye rock mate!"
 
someTestsFailedIcon=/usr/share/icons/gnome/48x48/status/weather-showers.png
someTestsFailedTitle="Some tests failed"
someTestsFailedMessage="Avast! Ye broke sumthin'!"
 
folderBeingWatched=$1
phpunitConfigPath=$2
 
if [ ! -z $folderBeingWatched ]; then
 
  if [ -z $phpunitConfigPath ]; then
        phpunitConfigPath=app/
    fi
 
    while [ true ]; do
        inotifywait -qq -e modify $folderBeingWatched -r
        if phpunit -c $phpunitConfigPath | tail -n3 | grep "OK"; then
            notify-send --hint=int:transient:1 --icon=$allTestsOkIcon "$allTestsOkTitle" "$allTestsOkMessage" --expire-time=1500
        else
            notify-send --hint=int:transient:1 --icon=$someTestsFailedIcon "$someTestsFailedTitle" "$someTestsFailedMessage" --expire-time=1500
        fi
    done
else
    echo "USAGE: PHPAutotest folderToLookForModifications [phpUnitConfigPath]"
fi
{% endhighlight %}