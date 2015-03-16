---
layout: post
title:  "A better analytics tracking for your app"
date:   2015-03-14 16:40:34
summary: "Migrating from Google Analytics to Google Tag Manager to future proof analytics tracking logic"
tags: google-analytics google-tagmanager android patterns
author: Ha Duy Trung
author_link: //twitter.com/hadt
---

[Google Analytics](https://www.google.com/analytics/) is great. I love it! People at my company love it! It tells you so many things about how your audience uses your app. But once your feet are deep into the analytics game, your application will probably end up with a bunch of clunky analytics tracking code as below:

{% highlight java %}
GoogleAnalytics.getInstance(context).send(new HitBuilders.EventBuilder()
        .setCategory("story")
        .setAction("view")
        .setLabel("1")
        .build());
GoogleAnalytics.getInstance(context).send(new HitBuilders.EventBuilder()
        .setCategory("story")
        .setAction("share")
        .setLabel("1")
        .build());
{% endhighlight %}

Don't get me wrong, the [Google Analytics API](https://developers.google.com/analytics/devguides/collection/android/v4/) is in no way bad, but the direct application of 'analytics tracking' may backfire you in the future, let's say when you switch to another provider, or when you simply want to change the name of a category/action. This calls for a layer of abstraction to future proof your tracking methods.

Coincidentally (or not!), [Google Tag Manager](https://www.google.com/tagmanager/) comes into the picture to help the abstraction of analytics tracking in a (not so) intuitive way. The idea behind Google Tag Manager is excellent, although it takes even an experienced engineer a while to logically put all its concepts in the correct place. That may explain why it does not get as [popular](https://stackoverflow.com/tags/google-tag-manager/info) as Google Analytics.

### Google Tag Manager explained

Let's take an example to explain how things work in Google Tag Manager. Assuming you want to repeat the Google Analytics logic above: send a request to Google Analytics, with category `story`, action `view` or `share`, whenever an object of type `story` encounters event `view`, or an object of type `story` encounters event `share`. So trivial!

In 'Google Tag Manager language', using its 3 main concepts: [*macro*](https://support.google.com/tagmanager/answer/2644341?hl=en&ref_topic=3441647), [*rule*](https://support.google.com/tagmanager/answer/2644396?hl=en&ref_topic=3441647) and [*tag*](https://support.google.com/tagmanager/topic/3281056?hl=en&ref_topic=3441647), this can be specified as follows:

<table class="table table-hover">
  <tr class="info">
    <td>macro <code>object_type</code></td>
    <td>provide object type, e.g. <code>story</code></td>
  </tr>
  <tr class="info">
    <td>macro <code>event</code></td>
    <td>provide event, e.g. <code>view</code> or <code>share</code></td>
  </tr>
  <tr class="warning">
    <td>rule <code>any story view</code></td>
    <td><code>object_type</code>=<code>story</code> &amp; <code>event</code>=<code>view</code> [&amp; <code>object_id</code>=*]</td>
  </tr>
  <tr class="warning">
    <td>rule <code>any story share</code></td>
    <td><code>object_type</code>=<code>story</code> &amp; <code>event</code>=<code>share</code> [&amp; <code>object_id</code>=*]</td>
  </tr>
  <tr class="success">
    <td>tag <code>track story view</code></td>
    <td>Universal Analytics track, category=story, action=view<br/>when <code>any story view</code> is triggered</td>
  </tr>
  <tr class="success">
    <td>tag <code>track story share</code></td>
    <td>Universal Analytics track, category=story, action=share<br/>when <code>any story share</code> is triggered</td>
  </tr>
</table>

In simple words, a *tag* connects to your analytics service, whenever *rule*s that compare data sent from your app - retrieved via *macro*s - are matched.

With this setup, an equivalent logic to the original Google Analytics example is as follows:

{% highlight java %}
TagManager.getInstance(context).getDataLayer().pushEvent("view",
        DataLayer.mapOf("object_type", "story", "object_id", "1"));
TagManager.getInstance(context).getDataLayer().pushEvent("share",
        DataLayer.mapOf("object_type", "story", "object_id", "1"));
{% endhighlight %}

<div class="bs-callout bs-callout-info">
  <h4>Tip</h4>
  If you have multiple Google Analytics trackers, you can also send the tracker ID you want to communicate with as a data variable that can be retrieved via <i>macro</i>
</div>

### Abstraction, Abtraction, Abstraction!

So now we have made some progress, converting our Google Analytics logic into Google Tag Manager logic, adding tons of complexity. But for what?

At a glance, the two code blocks before and after look almost identical. But looking closer, you can notice that we have managed to put an abstraction layer between our 'interaction definition' and Google Analytics 'event definition'.

We don't push an event to the tracking service anymore. We push our data model and its interaction to a 'data sink', that will decide how this information will be processed and sent to the tracking service. In Google Tag Manager, 'data model' is defined in the form of [`DataLayer`](https://developer.android.com/reference/com/google/android/gms/tagmanager/DataLayer.html).

If we generalize our data model and its interaction definition, treating anything that can be interacted with as a `Trackable` object, then we can just simply dump any interaction with a `Trackable` to Google Tag Manager, or an in-house data sink, delaying the decision of what to track for another day, and move forward with the project, without having to worry about fixing the code later to change tracking logic.

**Trackable.java**
{% highlight java linenos %}
public interface Trackable {
    enum ObjectType { story, comment, ... }
    String getTrackedObjectId();
    ObjectType getTrackedObjectType();
}
{% endhighlight %}

**Analytics.java**
{% highlight java linenos %}
public final class Analytics {
    public enum Event { view, share, ... }

    public static class Builder {
        private final Trackable mTrackable;
        private final List<Event> mEvents = new ArrayList<>();

        public static Builder createInstance(Trackable trackable) {
            return new Builder(trackable);
        }

        private Builder(Trackable trackable) {
            mTrackable = trackable;
        }

        public Builder add(Event event) {
            if (event != null) {
                mEvents.add(event);
            }

            return this;
        }

        public void send(Context context) {
            if (mTrackable == null) {
                return;
            }

            for (Event event : mEvents) {
                TagManager.getInstance(context).getDataLayer().pushEvent(
                        event.name(),
                        DataLayer.mapOf(
                                "object_type", mTrackable.getTrackedObjectType(),
                                "object_id", mTrackable.getTrackObjectId()
                        ));
            }

            // some other in-house data sink dumping
        }
    }
}
{% endhighlight %}

Using the above `Trackable` interface and the fluent-API `Analytic.Builder` class, one can dump interaction data to Google Tag Manager as follows:

{% highlight java %}
Analytics.Builder.createInstance(new Trackable() {
            @Override
            public String getTrackObjectId() {
                return "1";
            }

            @Override
            public ObjectType getTrackedObjectType() {
                return ObjectType.story;
            }
        })
        .add(Analytics.Event.view)
        .add(Analytics.Event.share)
        .send(context);
{% endhighlight %}

Of course, generalizing things and introducing abstraction layers will make things less flexible for some special cases. But in general, I always find that with some smart combinations of Google Tag Manager's *tag* and *rule*, the same can be achieved without losing the generality. And even better: the real tracking logic is now essentially broken free from our app!

<div class="bs-callout bs-callout-warning">
  <h4>Note</h4>
  It is not recommended to use both Google Analytics and Google Tag Manager at the same time. However, in case you must, I notice that Google Tag Manager will 'take over' all Google Analytics calls once it's initiated anyway.
</div>