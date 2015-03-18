# PropertyGuru Blog
[https://propertyguru.github.io](https://propertyguru.github.io)

**Requirements**
* Ruby >= 1.9.3

**Installation**
```bash
$ gem install github-pages
$ jekyll serve
```
By default, jekyll local server will be running at [http://localhost:400](http://localhost:400)

**Adding a new post**

Create a new file `yyyy-MM-dd-slugs.markdown` under [`_posts`](/_posts) with the following structure:

```markdown
---
layout: post
title:  "Blog title"
date:   yyyy-MM-dd hh:mm:ss
summary: "Blog summary"
author: Author
author_link: //twitter.com/author
---

Markdown content

{% highlight php %}
echo "Hello World!";
{% endhighlight %}
...
```

The new blog will be generated under `_site`.

**Update Tags**

We **MUST** run this command every time after added new tag that never have before from last generated tag

```sh
$ rake generate:tag
Creating tag file: tags/android.html
Creating tag file: tags/jacoco.html
Creating tag file: tags/coverage.html
Creating tag file: tags/robolectric.html
Creating tag file: tags/gradle.html
Creating tag file: tags/testing.html
Creating tag file: tags/tdd.html
Creating tag file: tags/context.html
Creating tag file: tags/mock.html
Creating tag file: tags/patterns.html
Creating tag file: tags/google-analytics.html
Creating tag file: tags/unit-test.html
Creating tag file: tags/google-tagmanager.html
```

**Jekyll & Github Pages documentations**

* [Jekyll](http://jekyllrb.com/docs/home/)
* [Github Pages](https://help.github.com/articles/using-jekyll-with-pages/)
