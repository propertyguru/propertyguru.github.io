---
layout: post
title:  "iOS versioning with agvtool"
date:   2014-07-16 02:08:34
---

When search around how to update version number of an iOS project, most of the result is about how to update the `info.plist` file using scripts.

Actually, apple provides a command line tool called [agvtool](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/agvtool.1.html) to update the version numbers for us. It’s simple and gets the task done.

The thing is, before you know it, it may not be obvious about how to start. So here are the steps.

**Steps**

* Setup Xcode project to use apple’s scheme for versioning  
In project build settings, set Versioning System to “Apple Generic”  
Set Current Project Version to the current build version of the project.

* In command line, check the current build version of the project:  
{% highlight bash %}
$ agvtool what-version
{% endhighlight %}

* Set the build version number based on what we got in step 2 and what need to set for next build version.
{% highlight bash %}
$ agvtool new-version -all new_version_number
{% endhighlight %}

* Check the current marketing version of the project:
{% highlight bash %}
$ agvtool what-marketing-version
{% endhighlight %}

* Set the new marketing version of the project based on what we got in step 4 and what we need for next marketing version.
{% highlight bash %}
$ agvtool new-marketing-version new_marketing_version_number
{% endhighlight %}

If you want to increase the version number to the next highest integral value, use `agvtool bump -all` instead.

**References**

* [OS X Man Page for agvtool](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/agvtool.1.html)
* [Setting iPhone Application Build Versions](http://useyourloaf.com/blog/2010/08/18/setting-iphone-application-build-versions.html)
* [Technical Q&A QA1827:Automating Version and Build Numbers Using agvtool](https://developer.apple.com/library/prerelease/ios/qa/qa1827/_index.html#//apple_ref/doc/uid/DTS40014500)