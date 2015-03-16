---
layout: post
title:  "Separate debug activities from your live Android analytics with Gradle"
date:   2015-01-29 10:44:34
summary: "Utilizing Gradle build types to inject different configurations for separating analytics tracking for different build types"
tags: android gradle google-analytics
author: Ha Duy Trung
author_link: //twitter.com/hadt
---

[Gradle plugin for Android](http://tools.android.com/tech-docs/new-build-system) is the official build tool for Android, and is getting better everyday with new features allowing you to customize your build process more and more. One of its feature is allowing one to inject resources that can have different values per build type/product flavor. We can utilize it to separate analytics data for development and production.

In your `build.gradle`:

{% highlight groovy %}
android {
    defaultConfig {
        applicationId "<appId>"
        minSdkVersion 9
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
        resValue "bool", "debug", "true"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            zipAlignEnabled true
            resValue "bool", "debug", "false"
        }

        <other build types>
    }
}
{% endhighlight %}

This script uses [`resValue`](https://plus.google.com/+XavierDucrohet/posts/UVKA58MZV3J) to set `debug` bool resource to `true` by default for all build types, and overriden by release build type. We can then access this bool resource from Java code to configure Google Analaytics based on `debug` value (I did it in my `Application` class in this case)

{% highlight java %}
public class Application extends android.app.Application {
    @Override
    public void onCreate() {
        super.onCreate();
        if (getResources().getBoolean(R.bool.debug)) {
            GoogleAnalytics.getInstance(this).getLogger().setLogLevel(Logger.LogLevel.VERBOSE);
            GoogleAnalytics.getInstance(this).setDryRun(true);
        }
        GoogleAnalytics.getInstance(this).newTracker(R.xml.ga_config);
    }
}
{% endhighlight %}

With the above setup, Google Analytics will show verbose `GAV4` logging but not sending real hit to Google Analytics server except for your release builds, which I assume are only used for production. More advanced settings can be applied in similar fashion to configure different GA tracker per build type, or some other setups based on your needs.