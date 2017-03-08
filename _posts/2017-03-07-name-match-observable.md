---
layout: inner
title: 'Name Match Observable Lookup'
date: 2017-03-07 23:45:00
author: 'John Niedzwiecki'
categories: angular development
tags: angular javascript observables
lead_text: 'Some simple, useful fun with Observables.'
comments: true
---

One of the cool things in Angular is the use of Observables for the APIs. There are quite a bit of benefits of this (which is for a future post) but one of which is the ability to cancel the request. This ability makes Observables ideal for searches to happen as you type. The official [Angular Tour of Heroes tutorial](https://angular.io/docs/ts/latest/tutorial/) gives a good example under part 6, when implementing the hero search functionality. That's actually what I based this idea and work on.

In my latest project, I have an API made available to me to check if a site name is available. I wanted to implement this to check as you type, then showing an appropriate icon. This is a very simple user experience idea, saving the user time and giving them immediate feedback. With the ability to cancel the call in the UI, we don't need to manage the the order of requests, to make sure we're only showing the latest response.

# Background

## Code

All of the code shown below, and examples to run can be found in my GitHub repo [name-match-observable-example](https://github.com/RHGeek/name-match-observable-example). 

## Dependencies

[Angular-CLI](https://github.com/angular/angular-cli): I am currently using Angular-CLI to build my application, so the instructions (and example code) will assume this. You'll need to adapt for your own build methods. 
I personally find Angular-CLI very useful, though opinionated. It has additional dependencies itself of Node 4+ and NPM 3+.

## The API

First, we'll briefly discuss the API that I was given, as it took some extra consideration to get working quite the way I wanted. The API takes in a string, the site name, and then returns either a <code>200 OK</code> if the site name is not taken or <code>409 Conflict</code> if the name is already in use. Could the API be done differently in a way that makes the component and service simplier? Sure. But our goal is to use this API as it exists, so that's what we'll do. We'll just mock this for our service to call in the [mock-api.ts](https://github.com/RHGeek/name-match-observable-example/blob/master/src/app/shared/mock-api.ts).

# Making the Component

First, we'll create the component. We're just doing the bare bones here: a text box and some icons after to show the status. We'll just be using HTML codes for our icons here, though in app, I used [FontAwesome](http://fontawesome.io/). Use your graphics of choice. We'll create our component with the command <code>ng g component sitename-search</code>.

For the HTML template, we'll add a text box and some spans with ngIf to show the right icon. We'll check all against the nameAvailable variable that will give us YES, NO, ERROR, or LOADING. We'll need to use the async pipe, since we're dealing with Observables.

For the component, there's two main parts of the TypeScript code. First, we need to set up some logic in the <code>ngOnInit()</code>. We'll leverage rxjs/Subject to handle adding as we type and connecting to the service. The code snippet here below will set up everything we need. After, we'll discuss what each piece means.

{% highlight typescript %}
this.nameAvailable = this.siteNameTerms
            .debounceTime(300)
            .distinctUntilChanged()
            .switchMap(term => term
             ? this.sitesService.checkSiteName(term)
             : Observable.of<string>(''))
            .share();
{% endhighlight %}

First, we set a debounce time with <code>.debounceTime(300)</code>, which is the delay to pause in between events (in our case 300 ms). This keeps the component from firing off a request every character typed, and allows you to set a delay of how long to wait before sending off the next request. The next line <code>.distinctUntilChanged()</code> will ignore if you have the same search term mutliple times in a row. It's a simple optimization piece. We use <code>.switchMap(term => </code> to get a new Observable with each term. The switchMap function will stop emitting from earlier inner Observables and instead only emit from the new one. This essentially does all the work for you to cancel the old request and set up a new Observable. Within that, you'll see a condition that will call our service if there is any term, and when there is no term, return an Observable of the empty string. 

The last piece you see is the share function. In our template, we used the aync pipe several times. If we leave off this final call, when we run the code you'll notice 4 request every time, one for each async pipe. Each async pipe will get it's own subscription to the Observable, because an Observable is just a definition or representation of a set of values over time. Since we actually want just one get and subscription of data to be shared, we can simply add the <code>.share()</code> which will create a new Observable that will multicast the original Observable. Problem solved.

The second part we need to add to the component is a function to be used by the text field. All this function needs to do is add the term to our afore mentioned Subject siteNameTerms


{% highlight typescript %}
checkSiteAvailability(term: string): void {
    this.siteNameTerms.next(term);
}
{% endhighlight %}

That's it for the component. You can see the full code from Github [sitename-search.component.ts](https://github.com/RHGeek/name-match-observable-example/blob/master/src/app/sitename-search/sitename-search.component.ts). Next, we need to do a little work on the service to make the API endpoint work with our design.

# Making the Service

The final piece to complete our code is a service. Often, services are very simple: take in parameters for data (and the URL if working in a HATEOS environment) then call a GET or POST, return the result or an error. If we set up our service as such, when a site name is already in use, we'll be returning the error code back to the component. Normally, you'll want this. Unfortunately, an error will stop searching mechanism. I originally had the component handle the logic to catch the 409 and set the nameAvailable field accordingly, but once you had a match, which was an error, the logic would stop. Instead our service can do a little wrapping, catching the 409 as a known error as NO and only passing along any unexpected errors as ERROR. This will allow the lookup to continue functioning and making requests even after an API error is returned. The full code for the service is below, or check Github for the latest [sites.service.ts](https://github.com/RHGeek/name-match-observable-example/blob/master/src/app/shared/sites.service.ts). Additionally, for the unexpected errors, you'd likely want to add some extra error handling logic, but that would be dependent on your application.

{% highlight typescript %}
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs/Rx';
import { MockApi }  from './mock-api';

@Injectable()
export class SitesService {

  constructor(
    private mockApi: MockApi
  ) { }

  checkSiteName(siteName: string): Observable<string> {
    return this.mockApi.siteValid(siteName)
              .map((r: any) => 'YES')
              .catch(error => {
                  if(error.status==409) {
                      return Observable.of<string>('NO');
                  } else {
                      return Observable.of<string>('ERROR');
                  }
              });
    }

}
{% endhighlight %}

# How it works

Our codebase uses Angular-CLI, so to run our project, all we need to do run <code>ng serve</code>. This will start up locally and when it's ready, you simply navigat to [locahost:4200](http://localhost:4200/). For the mock-api, you'll find matches on any of the following values <code>['rhgeek', 'geek', 'geeks', 'realmenwherekilts']</code>. To force the backend to return a 500 error, enter <code>thisisnotasite</code>.

## What's Not Here

I unfortunately didn't get one small piece to work as I had hoped. I could not get the one variable to also work for displaying loading when a request was underway. I really wanted to have one variable set to the single status of the check. I tried several ways to set this value intermediately, but was unsuccessful. I was forced to do some wrapping with an extra loading variable used in conjunction with this, but was a very hacky approach I was not happy with. If you have an alternative suggestion to get this working with a loading status, I'd love to see it. If I find a solution on my own, I'll be sure to revisit and update this code.

# The Wrap Up

Overall, I accomplished my goal for the task. I had to add a little hack for the loading, but we'll see what we can do about that in the future. It was fun code to get working, with a couple of small 'gotcha' moments that hopefully having them here will help you out. Let me know what you think of the code or anyways you think it could be improved. I really do like what Observables have to offer and am slowly learning to use more of their power. Hopefully, that will yield a future post for you to read.