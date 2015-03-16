---
layout: post
title:  "Using context in Android testing"
date:   2014-06-02 02:08:34
summary: "Creating an interface for Android's Context to break direct dependency for unit testing"
tags: android tdd context mock patterns testing
author: Ha Duy Trung
author_link: //twitter.com/hadt
---

If you are keen on doing TDD with Android, you may have stumbled upon the dreaded `java.lang.RuntimeException: Stub!` when you try to test drive your code with different contexts as input. For us, most of the time we use context is to get some resources out of it, either string, bool, raw, you name it. These resources may change execution behavior, and thus we are required to test with different resources to cover all cases.

Soon you learn that it is not straightforward to get the default Android’s [`Context`](http://developer.android.com/reference/android/content/Context.html) out of the way, even with [`MockContext`](http://developer.android.com/reference/android/test/mock/MockContext.html). That is one of the reasons why we see test frameworks such as [Robolectric](http://robolectric.org/) (or mock framework such as [Mockito](https://code.google.com/p/mockito/), [PowerMock](https://code.google.com/p/powermock/)) getting popular. While it is convenient to let Robolectric build an activity for you and use it as the context, or mock up your own context using Mockito, there is a more structured way of making Context work for you in testing.

The reason why you can’t get `Context` out of your way in the first place is because it is a class, and you need to construct a specific instance to use it in testing, and it is just impossible to construct a real one! But what if it’s an interface? Then you can pass the interface around, and have a mock implementation of that interface to be used in testing instead of the real one?

With that in mind, we come up with the solution of having a `ContextWrapper`, which serves exact same purpose as Android’s `Context`, delegating all tasks to Android’s `Context`, and implements an interface, says `IContext`, that we will pass around instead of Android’s `Context` object. This `IContext` at a glance will look very much like what Android’s `Context` offers, and you can add in more as you go along with your project.

Below is a simplified example of how we implement this pattern. Now at least you can test `AppUtils`' method without worrying about how to construct a `Context`!

{% highlight java %}
public interface IContext {
    boolean getBoolean(int resId) throws Resources.NotFoundException;
}

public final class ContextWrapper implements IContext {
    private Context mContext;

    public ContextWrapper(Context context) {
        mContext = context;
    }
    @Override
    public boolean getBoolean(int resId) throws NotFoundException {
        return mContext.getResources().getBoolean(resId);
    }
}

public MyActivity extends Activity {
    private IContext mContext;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mContext = new ContextWrapper(this);
    }

    ...

    private boolean checkIsEnabled() {
        return AppUtils.isFeatureEnabled(mContext);
    }
}

public AppUtils {
    public static boolean isFeatureEnabled(IContext context) {
        try {
            return context.getBoolean(R.bool.is_feature_enabled);
        } catch (NotFoundException e) {
            return false;
        }
    }
}
{% endhighlight %}
