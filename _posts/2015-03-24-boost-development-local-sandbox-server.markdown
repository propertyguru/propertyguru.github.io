---
layout: post
title:  "Boost up development with a local sandbox server"
date:   2015-03-24 0:40:34
summary: "Quickly constructing a local sandbox server mirroring production web services for faster app development"
tags: android ruby api sinatra testing sandbox
author: Ha Duy Trung
author_link: //twitter.com/hadt
---

When we are doing unit testing, more often than not we will probably stumble upon external dependencies that we wish we could just get them out of the way to focus on the testing task at hand. That is when [mocking, stubbing, faking](http://martinfowler.com/articles/mocksArentStubs.html) comes in handy.

If we apply these techniques in a broader context, says for app testing, where our external dependencies are web services, how do we consistently and reliably test our app against the ever changing data returned by real API calls? That is when we need a [sandbox](http://en.wikipedia.org/wiki/Sandbox_%28software_development%29) server.

> The term *sandbox* is commonly used for the development of Web services to refer to a mirrored production environment for use by external developers. Typically, a third-party developer will develop and create an application that will use a web service from the sandbox, which is used to allow a third-party team to validate their code before migrating it to the production environment.
>
> -- <cite>Wikipedia</cite>

Now, when we talk about *sandbox* here, I mean *our own, self-developed, local sandbox server*, not an external sandbox server as provided by our service provider (e.g. [Paypal](https://www.sandbox.paypal.com/), [eBay](http://sandbox.ebay.com/), [Google](https://developers.google.com/wallet/instant-buy/test-flows) sandbox environment). Why? Because if we use an external sandbox server, it is still an external server, populated with test data from other users. We are often forced to use one so that we do not polute the service provider's production data, at the cost of real network latency (if not worse).

<img src="/assets/img/chuck-norris-test-production.jpg" class="img-responsive center-block" />

### Eat your own dog food
The fact that we depend on someone else's service does not mean that we have to depend on it during development as well, at least for sections of app that consume data versus those that produce data. Well, we may still need it at some point before migrating to production; but for day-to-day development, one can totally make do without it.

We start by mirroring the real service, by constructing a fake one, producing just enough information that our app needs to consume to be functional (not essentially in a logically correct way).

Let's take an example of a social news reader app that uses a few services from the public [Hacker News API](https://github.com/HackerNews/API):

* GET `/v0/topstories.json` - to retrieve list of top stories
* GET `/v0/item/<id>.json` - to retrieve a specific story with `id`

I like Ruby, it makes rapid prototyping easy; and [sinatra](https://github.com/sinatra/sinatra) seems to be the perfect pick to quickly construct a dumb sandbox server. But any web frameworks in any languages should be capable of doing the same thing here.

{% highlight ruby %}
require 'sinatra'

get '/v0/topstories.json' do
  status 200
  content_type :json
  %/[ 1, 2, 3 ]/
end

get '/v0/item/:id.json' do |id|
  status 200
  content_type :json
  %/{ "by": "hidro", "id": #{id}, "title": "Hello World", "type": "story" }%/
end
{% endhighlight %}

Done and dusted! Now every GET request to `/v0/topstories.json` will give us 3 story IDs, and every `/v0/item/<id>.json` will give us the same 'Hello World' story.

Let's make it a little bit smarter in the next prototyping iteration.

{% highlight ruby %}
require 'sinatra'

get '/v0/topstories.json' do
  status 200
  content_type :json
  request.path_info = '/v0/topstories'
  mock_response
end

get '/v0/item/:id.json' do |id|
  status 200
  content_type :json
  request.path_info = '/v0/item'
  mock_response %/#{id}/
end

def mock_response json='default'
  dir_path = %-#{Dir.pwd}/responses#{request.path_info}/-
  file_path = %-#{dir_path}#{json}.json-
  file_path = %-#{dir_path}default.json- if not File.file? file_path
  File.open(file_path)
end
{% endhighlight %}

Now whenever the sandbox server receives a request, it will return a JSON response as saved under the `/responses/` folder:

{% highlight bash %}
GET /v0/topstories.json
-> /responses/v0/topstories/default.json

GET /v0/item/1.json
-> /responses/v0/item/1.json
or /responses/v0/item/default.json
{% endhighlight %}

<div class="bs-callout bs-callout-info">
    <h4>Tip</h4>
    Keep the sandbox responses well-organized. They can serve as handy examples / quick documentations of what we can expect from external services.
</div>

What if we want to simulate different scenarios, e.g. when the server encounters high traffic load and fails to return proper response?

{% highlight ruby %}
require 'sinatra'

$responses_path = %-#{Dir.pwd}/responses/default/-

...

get '/_sandbox/:dataset' do |dataset|
  $responses_path = %-#{Dir.pwd}/responses/#{dataset}/-
end

def mock_response json='default'
  dataset = $dataset
  dir_path = %-#{$responses_path}#{request.path_info}/-
  file_path = %-#{dir_path}#{json}.json-
  file_path = %-#{dir_path}default.json- if not File.file? file_path
  File.open(file_path)
end

{% endhighlight %}
Now when we want to test scenario 'empty', send a request to `/_sandbox/empty`, the subsequent requests will look for response under 'empty' dataset:

{% highlight bash %}
GET /v0/topstories.json
-> /responses/empty/v0/topstories/default.json

GET /v0/item/1.json
-> /responses//empty/v0/item/1.json
or /responses/empty/v0/item/default.json
{% endhighlight %}

To revert to the normal 'default' dataset, send a request to `/_sandbox/default`.

This can keep going until we have a sandbox server that balances our needs and its simplicity.

<div class="bs-callout bs-callout-warning">
    <h4>Note</h4>
    It is very important to keep <i>simplicity</i> in mind, as we do not want to end up building comprehensive web services that is production-ready! The dumber the sandbox server is, the easier it is to maintain/update it.
</div>

###Switching to local sandbox server
With our sandbox server ready to be used for development and testing, we need to find a way to build our app against different servers: a sandboxed one for `debug` build, and a real one for `release` build. Here [http://localhost:4567](#) is the default sinatra host.

**build.gradle**
{% highlight groovy %}
apply plugin: 'com.android.application'

android {
    ...

    buildTypes {
        release {
            ...
            resValue "string", "service_host", "https://hacker-news.firebaseio.com"
        }

        debug {
            ...
            resValue "string", "service_host", "http://localhost:4567"
        }
    }

    ...
}

...
{% endhighlight %}

**RestServiceFactory.java**
{% highlight java %}
import com.squareup.okhttp.OkHttpClient;
import retrofit.Callback;
import retrofit.RestAdapter;
import retrofit.client.OkClient;

public class RestServiceFactory {
    interface RestService {
        @GET("/v0/topstories.json")
        void topStories(Callback<int[]> callback);
        @GET("/v0/item/{itemId}.json")
        void item(@Path("itemId") String itemId, Callback<Item> callback);
    }

    public static RestService create(Context context) {
        return new RestAdapter.Builder()
                .setEndpoint(context.getString(R.string.service_host))
                .setClient(new OkClient(new OkHttpClient()));
                .build()
                .create(RestService.class);
    }
}

...

RestServiceFactory.create(activity);
{% endhighlight %}

<div class="bs-callout bs-callout-warning">
    <h4>Note</h4>
    <p>You may need to point your app to an intranet IP address, as 'localhost' means different thing from the device's poinst of view. For <a href="https://www.genymotion.com/">Genymotion</a> emulator, the address should be <a href="#">http://10.0.3.2:4567</a>.</p>
    <p>Or even better, get a <a href="http://www.raspberrypi.org/products/">Raspberry Pi</a> and use it to power a web server!</p>
</div>

###Conclusion
A local sandbox server makes it very quick and convenient to develop/test/demo our app.

* It saves us from being blocked from development on the dark days when the internet connection is slow, or when the real server runs into troubles.
* It facilitates pipelining of full-stack development, allowing app development with the assumption that the 'to-be-developed' web server will function in certain ways, without having to wait for it to be ready first.

However, keep in mind that it is just a mirror of the real thing. When the real thing changes, if we have no way to get ourselves notified of the changes, we would still be working against an outdated mirror of it!

<div class="bs-callout bs-callout-info">
    <h4>Tip</h4>
    Use an API validator, or API response schema test, and have it run against both the real and sandbox servers frequently, to ensure that we get notified as soon as they are not in sync.
</div>

We also need to constantly remind ourselves that our local sandbox server is a dumb one. It only produces fake responses for development purpose, not executing real server logic with sandbox data like those provided by Paypal or eBay. The decision to make it smarter or dumber is in our hands, and it is a trade-off between how much we want to mirror the real thing vs. how quickly we want to get it out of the way.

Now say good bye to the real web server, and start using a local sandbox server for a boosted development experience!
