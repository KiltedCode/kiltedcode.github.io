---
layout: inner
title: 'Myths of Anuglar 2'
subtitle: 'What Angular Really Is'
date: 2017-05-16 22:45:00
author: 'John Niedzwiecki'
categories: angular development
tags: angular speaking devfestdc aot angular-cli typescript
featured_image: '/assets/img/angular-bigfoot.png'
lead_text: 'At ng-Europe in 2014, the Angular team showed off Angular 2. Unfortunately, the drastic changes were not received well by many. While things have changed since that announcement during the development of Angular 2, many people remember the things said there instead of what was actually done (damn first impressions).'
comments: true
---

I recently gave a talk at DevFest DC. This conference is one of my favorite every year, because of the amazing speakers and fantastic value for your money (but more on that in another post). This year, I gave a talk on Angular that had a two fold purpose. The first was to act as an introduction to Angular. The second was to dispell some of the misconceptions about what Angular is and isn't. This is my attempt to put a form of it into a blog.

## Why We're Here

At ng-Europe in 2014, the Angular team showed off Angular 2. Unfortunately, the drastic changes were not received well by many. 
While things have changed since that announcement during the development of Angular 2, many people remember the things said there instead of what was actually done (damn first impressions).

## What We'll Do

* [What is Angular?](#what-is-angular)
* [Version Soup](#semantic-version)
* [Mythbusting](#mythbusting)
* [Why Angular?](#why-angular)

# What is Angular? <a name="what-is-angular"></a>

The [official concise description](https://github.com/angular/angular) is "Angular is a development platform for building mobile and desktop web applications." Beyond that definition, Angular is an opinionated framework for building single page applications (SPA). It is built to be simple, fast, and align with the latest (and future) web standards.

Including the full set of code examples proved to be a little much in one post. If you'd like to take a look at some of the common and improved elements of Angular, hope over to [What is Angular?](/posts/2017/what-is-angular.html)

## Angular or AngularJS

With the release of Angular 2, the Angular team decided to give some naming convention to differentiate between version 1.x and the new versions 2+. 

To keep it simple:<br />
AngularJS == 1.x<br />
Angular == 2, 4, 6, 8...<br />
...who do we appreciate?

# Semantic Versioning <a name="semantic-version"></a>

With new Angular release, the team announced the switch to Semantic Versioning (SEMVER). The purpose of semantic versioning is to give meaning to how version numbers are assigned and incremented. Semantic versioning consists of 3 numbers, as shown here.

![alt text](/assets/img/semver.png "Semantic Versioning")

The important question is what does this mean for you and Angular. Beyond knowing what a version number change means, the Angular team wanted to give future changes in a predictable way with time based release cycles. The plan is to have a patch release each with with around 3 minor updates and one major update every 6 months. We've already seen that with the release of Angular 4 in March 2017.

While a major version change means breaking changes, it doesn't mean it's going to be like the Angular 1 to Angular 2 rewrite. Angular jumped from version 2 to 4 to align the core packages versions (Angular's router was already version 3). Future major updates will be more standard "breaking changes" with proper documented deprecation phases. For example, in Angular 4, template was deprecated in favor of ng-template. You'll receive warnings in your code, but everywhere template was used will continue as they did before the update. An actual breaking change was that all animations were moved from the core Angular into their own new experimental package.

# Mythbusting Time <a name="mythbusting"></a>

Now is the time we get to put on our fedoras and berets and take a look at some myths and misconceptions about Angular. Generally, these are things I've heard enough times that I felt they were worth setting the record straight.

## MYTH: Evergreen Browsers Only

Angular 2 was designed for the future. The team was looking ahead at the web and therefore Angular 2 was designed "targeting modern browsers and using ECMAScript 6" ([Source](https://blog.angularjs.org/2014/03/angular-20.html)). This mean only evergreen, auto-updating browsers. While this is a great desire, the internet is not quite there yet. And the Angular team knows this. That's why Angular supports a wealth of the most recent browsers, on both desktop and mobile.

If you take a look at the [official docs](https://angular.io/docs/ts/latest/guide/browser-support.html) you'll see support across the board, include back to IE 9, Safari / iOS 7, and Jelly Bean on Android. The Angular Github repo, can provide the latest continuous integration tests against the browsers as well. This is a snapshot from May 6, that matches checking this as of May 13th.

<img src="/assets/img/current-browser-support-may-6.png" style="max-width:100%" title="Browser support from CI" />

There's a challenge in supporting the larger range of browsers. Getting this kind of support requires polyfills. The Angular team offer mandatory and optional polyfills, depending on the browsers you want to support. What polyfills are needed, based on browser and features is all documented in the official docs. There are some base mandatory polyfills, like for ES6, and then some optional ones, such as for animations. If you use a tool like Angular-CLI, it provides you with this handy polyfills file, that you simply need to uncomment the files based on what you want to support.

{% highlight typescript %}
/**
 * This file includes polyfills needed by Angular and is loaded before the app.
 * You can add your own extra polyfills to this file.
 *
 * This file is divided into 2 sections:
 *   1. Browser polyfills. These are applied before loading ZoneJS and are sorted by browsers.
 *   2. Application imports. Files imported after ZoneJS that should be loaded before your main
 *      file.
 *
 * The current setup is for so-called "evergreen" browsers; the last versions of browsers that
 * automatically update themselves. This includes Safari >= 10, Chrome >= 55 (including Opera),
 * Edge >= 13 on the desktop, and iOS 10 and Chrome on mobile.
 *
 * Learn more in https://angular.io/docs/ts/latest/guide/browser-support.html
 */

/***************************************************************************************************
 * BROWSER POLYFILLS
 */

/** IE9, IE10 and IE11 requires all of the following polyfills. **/
import 'core-js/es6/symbol';
import 'core-js/es6/object';
import 'core-js/es6/function';
import 'core-js/es6/parse-int';
import 'core-js/es6/parse-float';
import 'core-js/es6/number';
import 'core-js/es6/math';
import 'core-js/es6/string';
import 'core-js/es6/date';
import 'core-js/es6/array';
import 'core-js/es6/regexp';
import 'core-js/es6/map';
import 'core-js/es6/set';

/** IE10 and IE11 requires the following for NgClass support on SVG elements */
import 'classlist.js';  // Run `npm install --save classlist.js`.

/** IE10 and IE11 requires the following to support `@angular/animation`. */
import 'web-animations-js';  // Run `npm install --save web-animations-js`.


/** Evergreen browsers require these. **/
import 'core-js/es6/reflect';
import 'core-js/es7/reflect';


/** ALL Firefox browsers require the following to support `@angular/animation`. **/
import 'web-animations-js';  // Run `npm install --save web-animations-js`.



/***************************************************************************************************
 * Zone JS is required by Angular itself.
 */
import 'zone.js/dist/zone';  // Included with Angular CLI.



/***************************************************************************************************
 * APPLICATION IMPORTS
 */

/**
 * Date, currency, decimal and percent pipes.
 * Needed for: All but Chrome, Firefox, Edge, IE11 and Safari 10
 */
import 'intl';  // Run `npm install --save intl`.
{% endhighlight %}

One important note, that comes straight from the official documentation for [browser support](https://angular.io/docs/ts/latest/guide/browser-support.html) is a sentiment I often share: "Note that polyfills cannot magically transform an old, slow browser into a modern, fast one." Remember that.

**MYTH: Evergreen Browsers Only**

<span class="myth-busted" title="BUSTED">&#0164;</span>

## MYTH: No Migration Path

Initially, no path to migration was announced. The team's goal was the best framework to build a web app without worrying backwards compatibility with existing APIs. The response was along the lines of "we will see". The most they offered was that the core concepts would be the same, carrying that knowledge forward. Obviously, people were less than happy for no concrete migration path.

Much like browser support, this is not the end implementation. Not only can you Not only can you migrate an app, you can upgrade _incrementally_. If you want to upgrade your app, you need to do some preparation. If you're already following the [Angular Style Guide](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md), you're off to a good start (and if you're not, why?).

* [Follow the Angular Style Guide](https://angular.io/docs/ts/latest/guide/upgrade.html#!#follow-the-angular-style-guide) (so your AngularJS will align better with Angular code)
* [Use a module loader](https://angular.io/docs/ts/latest/guide/upgrade.html#!#using-a-module-loader) (i.e. Webpack, SystemJS, Browserify)
* [Migrate to TypeScript](https://angular.io/docs/ts/latest/guide/upgrade.html#!#migrating-to-typescript) (if you plan to use it for Angular)
* [Use Component Directives](https://angular.io/docs/ts/latest/guide/upgrade.html#!#using-component-directives) (they're easier to migrate to Angular)

This prep could take you some time, but will make the rest much easier. Like any type of migration, sometimes the preparations take longer than the actual migration. Once you are prepped and ready to roll, you can take advantage of the Upgrade Module. This allows you run both versions of Angular side-by-side. They can interoperate via dependency injection, the DOM, and change detection. You just need to choose _how_ you want to migrate, will you have an Angular app with AngularJS pieces, or will your AngularJS app have Angular pieces? For example, the Upgrade module can upgrade an AngularJS components to work with Angular code, or downgrade your Angular code to work with the AngularJS code. You have to look at what will work best for you and your app. Maybe your base app to leverage the powerful new router but still use your AngularJS component directives and services. The choice is yours!

**MYTH: No Migration Path**

<span class="myth-busted" title="BUSTED">&#0164;</span>

## MYTH: Base Angular App is HUGE

After the release, I've heard many arguments about the size of Angular files. 
I often see it in comparisons of React vs Angular. You might see the comparison of Angular 2’s minified version (566K) to React JS with add-ons, minified version (144K) ([source](http://blog.debugme.eu/react-vs-angular2/)). This is a non-issue for an Angular app thanks to Ahead of Time (AOT) compilation. Angular built to take advantage of ES2015 module and tree shaking. "Tree shaking" is the process of removing unused code from final compiled bundles. With this, you'll only include the portions of the Angular base you need, reducing the footprint of your application. As a proof of concept, I've seen a Hello World that can be made in just 29KB ([source](https://www.lucidchart.com/techblog/2016/09/26/improving-angular-2-load-times/)).

Angular 4 has made even larger improvements to the ahead of time compilation. The team concentrated on implementing a new View Engine to generate less code. This has shown good savings in the bundle sizes. In a comparison of two medium size apps improvements were seen from 499Kb to 187Kb (68Kb to 34Kb after gzip) and from 192Kb to 82Kb (27Kb to 16Kb after gzip) ([source](http://blog.ninja-squad.com/2017/03/24/what-is-new-angular-4/)).

**MYTH: Base Angular App is HUGE**

<span class="myth-busted" title="BUSTED">&#0164;</span>

# Why Angular? <a name="why-angular"></a>

Finally, let's look at some of the key features that answer the question "Why Angular?"

## Speed and Performance

I loved AngularJS, but there were performance issues (I'm looking at you dirty checking and digest cycles). Part of the reason for the rewrite for Angular 2 was to get things right, from the ground up. They learned from themselves and of the rest of the web over the life of AngularJS. Angular sees faster checking of single bindings, improved change detection strategies, Immutables, Observables, use of zone.js, AOT compiling, and lazy loading modules. All of these pieces make for a faster, stronger Angular.

## Angular Anywhere

Angular is built to be used anywhere and everywhere. It was designed to be mobile first (another pain point of AngularJS). As we previously discussed, it works across a wealth of desktop browsers as well. Angular offers it's own mobile native option when [paired with NativeScript](https://docs.nativescript.org/tutorial/ng-chapter-0) or even with React Native through a [React Native renderer](http://angularjs.blogspot.com/2016/04/angular-2-react-native.html) (yes, you read that right). But why stop there? You can use Angular for the [Internet of Things (IoT)](https://medium.com/@urish/building-simon-with-angular2-iot-fceb78bb18e5). [Uri Shaked](https://github.com/urish/angular-iot) has created experimental technology to program physical hardware (buttons, LEDs, etc.) using Angular. It also includes a set of directives, such as <iot-button> and <iot-led> to interface with hardware.

## Angular Universal

Universal is more than just a theme park with some awesome Harry Potter lands. It's also the name of the _official_ package for server side rendering in Angular. Server side rendering can be important to single page apps (SPA). Everyone knows it matters for search engine optimization. It's also great to ensure you get that nifty little preview on social media.

[Angular Universal](https://universal.angular.io/) is also great for your _perceived_ performance. Angular Universal lets you render your app on the server, give it to the user, preboot a div to record their actions, and then replay and swap out once the app is bootstrapped. As little as 200 milliseconds in page load performance has an impact on user behavior, so quickly serving up a generated static page gets your user running faster.

## TypeScript: The thing you may be surprised to love

Frankly, I was worried about _needing_ to use TypeScript going into Angular.

Then I used it.

Now, I <img src="/assets/img/heart.png" class="inline-heart" alt="<3" title="<3" /> it.

I love having simple interfaces for my data and enforcing my types with them. Using TypeScript also "helped" some younger members of my team write better JS code (because they had no choice). TypeScript is a superset of ES6, so you can "use" TypeScript and just write JavaScript is you prefer. It comes with own great tooling that in turn helps make your Angular code easier to write. The code is easier to understand when written well, knowing what to give, what you'll get, and what's expected. This has made using other's components so easy much easier. Even if their documentation is only so-so, if the code is written well in TypeScript, you're IDE helps you to use it. Don't need to use TypeScript, but most example code you find, is.

## Fantastic Tooling

Speaking of great TypeScript tooling, Angular as a whole has some fantastic tooling. There is great IDE integration, for whatever your environment of choice is. As mentioned, the great TypeScript tooling in turn adds value to your Angular coding. Also discussed is the AOT compiling for Angular. The last bit of tooling, which we aluded to back in the [myth of evergreen browsers only](#mythbusting) is Angular-CLI. [Angular-CLI](https://cli.angular.io/) is an amazing command line tool for Angular applications. It gives you easy app scaffolding, code generation, support of SASS and LESS, support for development running using proxys, polyfills, unit tests, and even production builds with AOT compiling. With the release of Angular 4, Angular-CLI is now thankfully finally at a stable release, so is ready to rock and roll.

## Community

Angular 2 may have only been release in the later part of 2016, but Angular has an established community that has grown over many years. 

![alt text](/assets/img/hugs.gif "HUGS!")

It's a very supportive community, as I often find the development community is. Some of the contributors to Angular were part of the community of AngularJS. Even the official style guide for Angular is the the result of a lot of community work driven by John Papa, who spearheaded the unofficial style guide for AngularJS, that became the go to guide for 1.x. There is plenty of help to be found for whatever your problem through [StackOverflow](http://stackoverflow.com/questions/tagged/angular), [Google Groups](https://groups.google.com/forum/#!forum/angular), and Slack channels(a la [Angular Beers](http://slackin.angularbeers.org/)).

# Wrap Up

If you made it this far, wow, thank you for reading. If you didn't, thank you for what ever you may have read. The presentation at DevFest DC was well received, so I hope it translated well here in long form.

If you want to see the [slides](https://rhgeek.github.io/talks/myths-of-angular.html), they're [live](https://rhgeek.github.io/talks/myths-of-angular.html) under [talks](https://rhgeek.github.io/talks/). You can head to the  [mythbusters-angular repo](https://github.com/RHGeek/mythbusters-angular) for both example codes and the slides with reveal.js source. Unfortunately, there were videographer issues, so there is not video of the presentation to be watched (man, that would have saved you some time).