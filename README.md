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

**Jekyll & Github Pages documentations**

* [Jekyll](http://jekyllrb.com/docs/home/)
* [Github Pages](https://help.github.com/articles/using-jekyll-with-pages/)
