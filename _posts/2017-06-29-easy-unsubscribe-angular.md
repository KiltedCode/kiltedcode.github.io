---
layout: inner
title: 'Managing Subscriptions without Unsubscribe in Angular'
date: 2017-06-29 23:45:00
author: 'John Niedzwiecki'
categories: angular development
tags: angular javascript rxjs
featured_image: '/assets/img/gollum.jpg'
lead_text: 'Dont manage a stack of unsubscribes for your RxJS Observables. Manage them all with the takeUntil operator. One Subject to rule them all.'
comments: true
---

Angular manages unsubscribing from certain observable subscriptions like the HTTP service or the async pipe. However, it doesn't clean up them all. The rest are up to you. RxJS is powerful and provides you several ways to handle your observables.

# Manual Unsubscribe

The straightforward way to handle a subscription is to manually unsubscribe from it. The blueprint is an easy one: save the <code>Subscription</code> then, when the time comes call <code>unsubscribe()</code>. For our example, I'll make a service request and set up a timer. The service is representative of any service that returns an Observable that won't be handled by Angular, such as custom observable based code or working with a custom library. We'll unsubscribe inside of ngOnDestroy, a [lifecycle hook](https://angular.io/guide/lifecycle-hooks) provided by Angular. As it names implies, it is called just before Angular destroys the component or directive. It's purpose is to "cleanup just before Angular destroys the directive/component" including to "unsubscribe Observables and detach event handlers to avoid memory leaks." 

{% highlight typescript %}
import { Component, OnInit, OnDestroy } from '@angular/core';

import { Observable } from 'rxjs/Observable';
import { Subscription } from 'rxjs/Subscription';
import 'rxjs/add/observable/timer';

import { ProcessingService } from './shared';

@Component({
  selector: 'rh-manual-unsubscribe',
  templateUrl: './manual-unsubscribe.component.html'
})
export class ManualUnsubscribeComponent implements OnInit, OnDestroy {
    private procSub: Subscription;
    private manTimer: any;

    constructor(
        private procService: ProcessingService
    ) { }

    ngOnInit() {
        this.procSub = this.procService.checkAllTheThings()
            .subscribe(
            (resp: any) => {
                /* One ring to rule them all, 
                   one ring to find them, 
                   One ring to bring them all 
                   and in the darkness bind them. */
            }, 
            error => {
                /* One does not simply walk into Mordor */
            });
        this.manTimer = Observable.timer(10000, 10000)
            .subscribe((t: any) => {
                console.log('Walking...');
            });
    }

    ngOnDestroy() {
        if(this.procSub) {
            this.procSub.unsubscribe();
        }
        if(this.manTimer) {
            this.manTimer.unsubscribe();
        }
    }
}
{% endhighlight %}

As you can see, with one or two items, it's not too much work. You should also be able to see how it can get stacked up quickly, managing a variable and an unsubscribe (with an option safe check to make sure it exists) for each Observable. The safe check can really come into play with timers that may not always be running. If you look at this and think "there must be an easier way", then you're right.

# takeUntil Operator

The second method to manage your subscriptions and their clean up is to use the <code>takeUntil</code> operator. To do this, we need to compose our Observables differently, adding the <code>takeUntil</code> operator and passing it a <code>Subject</code> that you'll update during the <code>ngOnDestroy</code>. I'll modify the previous code, but this time with takeUntil.

{% highlight typescript %}
import { Component, OnInit, OnDestroy } from '@angular/core';

import { Observable } from 'rxjs/Observable';
import { Subject } from 'rxjs/Subject';
import 'rxjs/add/observable/timer';

import { ProcessingService } from './shared';

@Component({
  selector: 'rh-manual-unsubscribe',
  templateUrl: './manual-unsubscribe.component.html'
})
export class ManualUnsubscribeComponent implements OnInit, OnDestroy {
    private ngUnsubscribe: Subject<void> = new Subject<void>();
    private manTimer: any;

    constructor(
        private procService: ProcessingService
    ) { }

    ngOnInit() {
        this.procService.checkAllTheThings()
            .takeUntil(this.ngUnsubscribe)
            .subscribe(
            (resp: any) => {
                /* One ring to rule them all, 
                   one ring to find them, 
                   One ring to bring them all 
                   and in the darkness bind them. */
            }, 
            error => {
                /* One does not simply walk into Mordor */
            });
        this.manTimer = Observable.timer(10000, 10000)
            .takeUntil(this.ngUnsubscribe)
            .subscribe((t: any) => {
                console.log('Walking...');
            });
    }

    ngOnDestroy() {
        this.ngUnsubscribe.next();
        this.ngUnsubscribe.complete();
    }
}
{% endhighlight %}

What we can see here is much simpler code. For each Observable we use, we compose it with the takeUntil operator and pass each the _same_ Subject, in our code named ngUnsubscribe. We don't need any other reference to the subscription, though for the case of the timer, I'm saving it in case we want to be able to stop it at any time. The purpose of the [takeUntil operator](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-takeUntil) is to "emit the values emitted by the source Observable until a notifier Observable emits a value." The Subject (which extends Observable) we provide is the "notifier" so that when it says it is complete, the condition is met and _all_ Observables using our ngUnsubscribe will complete. That is what we are seeing in ngOnDestroy. When the component is destroyed we call next and complete on the subject, which will in turn tell the observables that they are complete. Now, we can add as many new Observables as we want without needing to add more Subscription variables and more manual unsubscribes.

# tl;dr

Don't store subscriptions and manually unsubscribe. Instead, give every observable the same subject through <code>takeUntil</code> and complete the subject in ngOnDestroy to manage all observables at once.