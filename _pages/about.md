---
layout: inner
title: About the Geek
permalink: /about/
---

I'm currently working on restarting my personal tech blog here, using Jekyll and GitHub Pages. More updates coming as I work through the kinks.
Plus, I'll probably put a little more information about me here.

For now, here's a function about me.

{% highlight javascript %}
function getDeveloperInfo() {
    const dev = {
        name: '{{ site.data.me.name }}',
        desc: '{{ site.data.me.tag }}',
        twitter: '{{ site.data.me.twitter }}',
        github: '{{ site.data.me.github }}',
        web: '{{ site.data.me.web }}',
        email: '{{ site.data.me.email }}',
        currentDevLoves: ['Angular', 'JavaScript', 'NativeScript'. 'd3', 'CSS fun'],
        books: [
            {
                name: 'Angular 5 Companion Guide',
                link: 'https://www.packtpub.com/free-ebook/angular-5-companion-guide'
            }
        ],
        nonDevAttr : {
            husband: true,
            father: true,
            hobbies: ['running', 'disney', 'cooking', 'video games', 'photography'],
            twitter: '{{ site.data.me.otherTwitter }}',
            web: '{{ site.data.me.otherWeb }}',
        }
    };
    return dev;
}

console.log(getDeveloperInfo());
{% endhighlight %}
