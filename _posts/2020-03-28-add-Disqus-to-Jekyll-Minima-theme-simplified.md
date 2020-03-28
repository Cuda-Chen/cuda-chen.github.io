---
layout: post
title: "Add Disqus to Jekyll Minima Theme Simplified"
category: [blogging]
tags: [Jekyll, Minima(Jekyll)]
---

Some of the Jekyll theme lack of comment system, as a result, we have to add plugin
to fulfill this task, and [Disqus](https://disqus.com/) is recommended by many people
because of its simplicity to install on your website. However, some of the themes have
some problems adding Disqus, so I write this guide to teach you Minima users to install
Disqus with no any problem.

Let's get started!

First create a [Disqus account](https://disqus.com/).

Then set your Jekyll website with Disqus by creating a unique [Disqus shortname](https://help.disqus.com/en/articles/1717111-what-s-a-shortname).

After setting your Disqus shortname, open your Jekyll sites `_config.yml` and add the 
following code. Note that you have to alter `my_disqus_shortname` to your Disqus
shortname you set.
```
# Disqus Comments
disqus:
    # Leave shortname blank to disable comments site-wide.
    # Disable comments for any post by adding `comments: false` to that post's YAML Front Matter.
    shortname: my_disqus_shortname

url: my_website_url
```

Mentioned in this [issue](https://github.com/jekyll/minima/issues/104), you have to explicitly add 
the `url` placeholder if you are using Minima theme just like me.

Next, create a file named `disqus_comments.html` in your Jekyll's `_includes` directory, and add 
the following content in the file:
{% raw %}
```
{% if page.comments != false and jekyll.environment == "production" %}

  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {
      this.page.url = '{{ page.url | absolute_url }}';
      this.page.identifier = '{{ page.url | absolute_url }}';
    };
    (function() {
      var d = document, s = d.createElement('script');
      s.src = 'https://{{ site.disqus.shortname }}.disqus.com/embed.js';
      s.setAttribute('data-timestamp', +new Date());
      (d.head || d.body).appendChild(s);
    })();
  </script>
  <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{% endif %}
```
{% endraw %}

The above code block is called [Disqus Universal Embed Code](https://cuda-chen.disqus.com/admin/universalcode/).
Note this code {% raw %}`{% if page.comments != false and jekyll.environment == "production" %}`{% endraw %} and 
{% raw %}`{% endif %}`{% endraw %}.
You simply add `comments: false` on any blog post [YAML front-matter](https://jekyllrb.com/docs/front-matter/)
if you want to disable Disqus comments. Additionally, it will prevent Disqus loading when
the environment of Jekyll is set to `development`.

At last, open your post layout file usually called `post.html` in your `_layouts` directory
and add the following liquid code just after the `</article>` tag. This tells Jekyll to 
load Disqus comments when your Jekyll environment is set to `production`.
{% raw %}
```
{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}
```
{% endraw %}

You can check the settings are effective or not by typing this command in your terminal:
```
$ JEKYLL_ENV=production bundle exec jekyll serve
```

That's all! After pushing these changes to your live site, the Disqus comment system
ought to work in no problem.

## Reference
https://desiredpersona.com/disqus-comments-jekyll/
