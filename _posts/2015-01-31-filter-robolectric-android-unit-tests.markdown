---
layout: post
title:  "Filter Robolectric Android unit tests for faster verification"
date:   2015-01-31 02:20:34
summary: "Customizing Robolectric Gradle plugin to run only a subset of unit tests"
tags: android gradle robolectric unit-test
author: Ha Duy Trung
author_link: //twitter.com/hadt
---

[Robolectric](https://github.com/robolectric/robolectric) is pretty much the de facto unit test framework for Android nowadays. It's great, it's fast (much faster than the out-of-the-box Android instrumentation test), and it gives you back joy of writing unit tests.

So you enjoy it so much that over the time you add more and more tests to your test suite that it no longer feels that fast anymore everytime you rerun it for verification? Well, it still needs to launch a VM to execute your tests after all. To improve this, you can make use of [Robolectric's Gradle plugin](https://github.com/robolectric/robolectric-gradle-plugin#configuration-using-dsl) configuration to filter only tests that need to be rerun.

{% highlight groovy %}
robolectric {
    include project.hasProperty("testFilter") ? "**/*${project.ext.testFilter}*Test.class" : '**/*Test.class'
    exclude '**/espresso/**/*.class'

    // other robolectric configs
}
{% endhighlight %}

Now when you run your Gradle test command, throw in a `-PtestFilter=<filter>` command line parameter to instruct Robolectric to run only the tests specified by your filter:

{% highlight bash %}
$ ./gradlew test -PtestFilter=Activity
{% endhighlight %}

The above command will execute all tests with class name following `*Activity*Test` pattern. No parameter will default to running all tests. A drawback I found with this approach is that if your pattern has no matches, Robolectric executes all tests instead of no test (since there are none).

<div class="bs-callout bs-callout-primary">
    <h4>Update (Mar 21st, 2015)</h4>
    <p>Latest Robolectric Gradle plugin with support for Android Gradle plugin 1.1.0 allows one to filter tests by running <code>./gradlew testDebug --tests='*.MyTestClass'</code></p>
    <p>For more information, see <a href="http://tools.android.com/tech-docs/unit-testing-support">Unit testing docs for Android Gradle plugin</a>.</p>
</div>

Another tip to speed up your test run is to tell Robolectric to parallelize your test run using more processes if possible:

{% highlight groovy %}
robolectric {
    // Specify max number of processes (default is 1)
    maxParallelForks = 4

    // other robolectric configs
}
{% endhighlight %}
