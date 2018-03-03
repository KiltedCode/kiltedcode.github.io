---
layout: inner
title: 'Code Bytes: Angular CLI with the Minimal Flag'
date: 2018-03-03 10:30:00
author: 'John Niedzwiecki'
categories: angular code-bytes development
tags: angular cli
featured_image: '/assets/img/code-bytes.jpg'
lead_text: 'Generage an Angular project without all the extra files and fluff with the --minimal flag.'
comments: true
---

I want to start a series of shorter blog posts, the for now I'm going to call **Code Bytes**. Obviously a cheesy play on words, but I like cheesy. And bacon, but I digress. The idea is they'll be shorter posts that are a quick read of limited scope. At least some of the first I have ideas aren't even enough to warrant a Github repo for examples. They'll be good with just some in post code examples. It's a little something new I'm trying out this year, so we'll see how it goes. For our first Code Byte, we turn to the Angular CLI.

The [Angular CLI](https://cli.angular.io/) is one of the killer tools that has made it so easy to get up and running with Angular. It really is a key piece when developing in Angular and it's abilities are only growing. One of the most basic uses is generating new projects and parts of the application, such as components and services. The CLI can give you all the bells and whistles. For example, for a new component it will give you the TypeScript file, and HTML template, a style file (CSS, SASS, etc), _and_ a unit test. Whether or not you want all those pieces every time is another case. You can use an inline template and inline styles. If you just want a simple project or are new to Angular and want to play around, this is a lot to immediately digest. But there is another way.

If you want to keep it simple, you can add a single flag to your command, the <code>--minimal</code> flag. This instructs the CLI to create the minimum necessary. To see the difference, create a default project and then a minimal project.

{% highlight shell %}
ng new default-app
ng new min-app --minimal
{% endhighlight %}

If we look at the difference at what is created, we can see fewer files below. I ran with the <code>--dry-run</code> flag to run without making changes which lists the files created. 

{% highlight shell %}
Repos$ ng new default-app --dry-run
  create default-app/README.md (1026 bytes)
  create default-app/.angular-cli.json (1246 bytes)
  create default-app/.editorconfig (245 bytes)
  create default-app/src/assets/.gitkeep (0 bytes)
  create default-app/src/environments/environment.prod.ts (51 bytes)
  create default-app/src/environments/environment.ts (387 bytes)
  create default-app/src/favicon.ico (5430 bytes)
  create default-app/src/index.html (297 bytes)
  create default-app/src/main.ts (370 bytes)
  create default-app/src/polyfills.ts (2667 bytes)
  create default-app/src/styles.css (80 bytes)
  create default-app/src/test.ts (1085 bytes)
  create default-app/src/tsconfig.app.json (211 bytes)
  create default-app/src/tsconfig.spec.json (304 bytes)
  create default-app/src/typings.d.ts (104 bytes)
  create default-app/e2e/app.e2e-spec.ts (293 bytes)
  create default-app/e2e/app.po.ts (208 bytes)
  create default-app/e2e/tsconfig.e2e.json (235 bytes)
  create default-app/karma.conf.js (923 bytes)
  create default-app/package.json (1316 bytes)
  create default-app/protractor.conf.js (722 bytes)
  create default-app/tsconfig.json (363 bytes)
  create default-app/tslint.json (2985 bytes)
  create default-app/src/app/app.module.ts (316 bytes)
  create default-app/src/app/app.component.css (0 bytes)
  create default-app/src/app/app.component.html (1139 bytes)
  create default-app/src/app/app.component.spec.ts (986 bytes)
  create default-app/src/app/app.component.ts (207 bytes)

NOTE: Run with "dry run" no changes were made.
Repos$ ng new min-app --minimal --dry-run
  create min-app/.angular-cli.json (1582 bytes)
  create min-app/src/assets/.gitkeep (0 bytes)
  create min-app/src/environments/environment.prod.ts (51 bytes)
  create min-app/src/environments/environment.ts (387 bytes)
  create min-app/src/index.html (293 bytes)
  create min-app/src/main.ts (370 bytes)
  create min-app/src/polyfills.ts (2667 bytes)
  create min-app/src/styles.css (80 bytes)
  create min-app/src/tsconfig.app.json (211 bytes)
  create min-app/src/typings.d.ts (104 bytes)
  create min-app/package.json (826 bytes)
  create min-app/tsconfig.json (363 bytes)
  create min-app/src/app/app.module.ts (316 bytes)
  create min-app/src/app/app.component.ts (1363 bytes)

NOTE: Run with "dry run" no changes were made.
{% endhighlight %}

That's a difference of 14 vs 28 files, mostly from dropping all the configuration related to testing. In addition when you create a new component in the default project you get 4 files compared to one in the minimal project. This is due to a change in the <code>angular-cli.json</code> configuration file. All the settings here are useful to understand, so you can configure your project what you want to generate. If we look at <code>"defaults"</code> we can see changes to what it should generate.

{% highlight json %}
"defaults": {
    "styleExt": "css",
    "class": {
      "spec": false
    },
    "component": {
      "inlineStyle": true,
      "inlineTemplate": true,
      "spec": false
    },
    "directive": {
      "spec": false
    },
    "guard": {
      "spec": false
    },
    "module": {
      "spec": false
    },
    "pipe": {
      "spec": false
    },
    "service": {
      "spec": false
    }
  }
  {% endhighlight %}

  You'll notice <code>"spec": false</code> appearing throughout. This tells the CLI to not generate a spec file when you generate a new piece of your application. In addition, it set <code>"inlineStyle": true</code> and <code>"inlineTemplate": true</code>, telling the CLI to set components to use inline instead of creating additional HTML and CSS files. Now, when you generate anything new within your project, you'll get just the minimum files, making it more compact and simple to play around. You're ready to rock and code.

  That concludes our first byte. Just a little code snack. A little nosh, if you would like. Hope you enjoyed.

  <small>Image credit: Ash Edmonds on [Unsplash](https://unsplash.com/photos/Koxa-GX_5zs), edited</small>