---
layout: inner
title: About
permalink: /about/
---

I'm currently working on restarting my personal tech blog here, using Jekyll and GitHub Pages. More updates coming as I work through the kinks.
Plus, I'll probably put a little more information about me here.

For now, here's a function about me.

{% highlight javascript %}
function getDeveloperInfo() {
    var dev = {
        name: '{{ site.data.me.name }}',
        twitter: '{{ site.data.me.twitter }}',
        currentDevLoves: ['Angular', 'JavaScript', 'd3', 'CSS instead of JS'],
        nonDevAttr : {
            husband: true,
            father: true,
            hobbies: ['Running', 'Disney', 'Cooking', 'Video Games', 'Photography'],
            twitter: '{{ site.data.me.otherTwitter }}',
            web: '{{ site.data.me.otherWeb }}',
        }
    };
    return dev;
}

console.debug(getDeveloperInfo());
{% endhighlight %}
