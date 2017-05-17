---
layout: inner
title: 'What is Angular?'
date: 2017-05-16 21:00:00
author: 'John Niedzwiecki'
categories: angular development
tags: angular speaking devfestdc
lead_text: 'Angular is a development platform for building mobile and desktop web applications.'
comments: true
---

This year, I gave a talk on Angular at DevFest DC that had a two fold purpose. The first was to act as an introduction to Angular. The second was to dispell some of the misconceptions about what Angular is and isn't. Translating the talk to a blog yielded something a _wee bit_ too long. So I've split the "What is Angular?" portion of code snippets here.

# What is Angular?

The [official concise description](https://github.com/angular/angular) is "Angular is a development platform for building mobile and desktop web applications." Beyond that definition, Angular is an opinionated framework for building single page applications (SPA). It is built to be simple, fast, and align with the latest (and future) web standards.

Let's take a look at some of the common and improved elements of Angular.

## Components

First, let's look at the key piece of creating an application, a component. Angular components are your base building block for your UI. In the end, your Angular application is a tree of components, so, yeah, they're pretty important.

This first example is a component that will get data from a service and then use the Observable returned directly in the UI. You'll see a couple of things here. First, we import anything we need to use for our components. Second, when setting up the component, there are three things we're setting up: the custom tag to use, an HTML template, and a stylesheet. This aligns with [web components](https://www.webcomponents.org/introduction) that contain specs for creating custom elements as well as encapsulating style and markup. For Angular, these can either be inline or referenced files like this to be bundled during compilation. The rest is our implementation of the component, which uses the injected service when the component initialized (<code>ngOnInit</code>) to get the data and then sets it to an object to be used by the HTML template. You'll also notice the <code>ThemeParkGroup</code> type, thanks to the use of Typescript.

{% highlight typescript %}
/* /theme-parks/theme-parks-list/theme-parks-list.component.ts */
import { Component, OnInit } from '@angular/core';
import { Observable } from 'rxjs/Rx';
import { ParksService, ThemeParkGroup } from '../../shared';

@Component({
  selector: 'basics-theme-parks-list',
  templateUrl: './theme-parks-list.component.html',
  styleUrls: ['./theme-parks-list.component.css']
})
export class ThemeParksListComponent implements OnInit {

  themeParks: Observable<ThemeParkGroup[]>;

  constructor(
    private parksService: ParksService
  ) { }

  ngOnInit() {
    this.themeParks = this.parksService.getParks();
  }
}
{% endhighlight %}

Next, let's look at how to use the Observable directly in the HTML template. They key element is using the <code>async</code> pipe. The [async pipe]() subscribes and unsubscribes to an Observable (or a Promise) for you automatically. Then, it returns the latest value emitted. When a new value is emitted, it will mark your component to be checked for changes. We'll use this on our themeParks variable, which is a list we can iterate through with a <code>ngFor</code>. Finally, you'll also see us set up a link for our router with <code>routerLink</code>, instead of an href. We are using it in a basic form here, but it can be used to generate links in a much more dynamic fashion.

{% highlight html %}
<!-- theme-parks-list.component.html -->
<div *ngFor="let group of themeParks | async">
  <h2><a routerLink="../details/{% raw %}{{group.id}}{% endraw %}">{% raw %}{{group.company}}{% endraw %}</a></h2>
  <div>
    Location: {% raw %}{{group.location}}{% endraw %}
  </div>
  <div>
    # of Parks: {% raw %}{{group.parks.length}}{% endraw %}
  </div>
</div>
{% endhighlight %}

This second example is another component. This component will get an ID value from the route. We can access this parameter from an Observable of the route params and then pass it onto our service. Instead of using the Observable directly in the template, we'll subscribe to the Observable this time within the component. The <code>switchMap</code> function essentially allows you to chain the Observables together.

{% highlight typescript %}
/* /theme-parks/theme-park-details/theme-park-details.component.ts */
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Params, Router } from '@angular/router';
import { ParksService, ThemePark } from '../../shared';

@Component({
  selector: 'basics-theme-park-details',
  templateUrl: './theme-park-details.component.html',
  styleUrls: ['./theme-park-details.component.css']
})
export class ThemeParkDetailsComponent implements OnInit {

  themePark: ThemePark;

  constructor(
    private route: ActivatedRoute,
    private parksService: ParksService
  ) { }

  ngOnInit() {
    this.route.params
      .switchMap((params: Params) => this.parksService.getParkDetails(params['parkId']))
      .subscribe(
        (park: ThemePark) => {
          this.themePark = park;
        },
        error => {
          /* do error stuff here if you must */
        });
  }

}
{% endhighlight %}

When subscribing to the Observable, we need to make sure to make safe checks against our object. Instead of need to wrap everything in ngIfs, we can use <code>themePark?.company</code>. The safe navigation operator can be used and avoid any exceptions for themePark being undefined.

{% highlight html %}
<!-- theme-park-details.component.html -->
<a routerLink="/parks/list" class="link--back"><small>back</small></a>
<div class="park-card">
  <h2>{% raw %}{{themePark?.company}}{% endraw %}</h2>
  <div>
    Location: {% raw %}{{themePark?.location}}{% endraw %}
  </div>
  <div>
    Parks:
    <ul>
      <li *ngFor="let park of themePark?.parks">{% raw %}{{park.name}}{% endraw %}</li>
    </ul>
  </div>
</div>
{% endhighlight %}

## Services

Second, let's take a look at services. A service is an important part of any application, because they're what you most often write when you need to get data. Services are injectable and therefore shared across the components they are used in. This makes them useful for other things you want to _share_ between components, beyond just getting data from a server.

In our example application, we'll have one service to be used by both of our components. The key to making a service is to make it injectable. In the ParksService are two functions, both returning Observables. You'll notice that we map the json response to a type, that we define as an Interface. This helps consumers of your service to know what to expect from a service call. They each also have a catch on the http get call. This pattern is what you'll find in the Tour of Heroes tutorial. It allows you to have a single error handler shared for all calls. In reality, the <code>handleError</code> function could connect to a higher level error handler, that could decide when to log errors, when messages should be sent to the user, and implement a solution if you want to send errors back to the server for collection. We, however, are simply going to do a console.log for now and then throw the error down the line. Advanced, I know.

{% highlight typescript %}
/* /shared/parks.service.ts */
import { Injectable } from '@angular/core';
import { Http } from '@angular/http';
import { Observable } from 'rxjs/Rx';
import { ThemePark } from './theme-park.model';
import { ThemeParkGroup } from './theme-park-group.model';

@Injectable()
export class ParksService {

  constructor(private http: Http) { }

  getParks(): Observable<ThemeParkGroup[]> {
    let url: string = `/api/parks/list`;
    return this.http.get(url)
            .map(response => response.json() as ThemeParkGroup[])
            .catch(this.handleError);
  }

  getParkDetails(parkId: string): Observable<ThemePark> {
    let url: string = `/api/parks/${parkId}`;
    return this.http.get(url)
            .map(response => response.json() as ThemePark)
            .catch(this.handleError);
  }

  private handleError(error: any): Observable<any> {
      console.error('An error occurred', error);
      return Observable.throw(error.message || error);
  }
}
{% endhighlight %}

## Routing

Routing is one of the many areas that the Angular team learned there lesson in AngularJS. Angular routing allows you to define child routes, lazy load modules based on routes, have multiple router outlets (including named outlets), and define activation and deactivation guards.

In our routing, we'll see several of these features. If you look at the parks path, you'll see we have a component defined but also children routes, with their own components.

{% highlight typescript %}
/* /app-routing.module.ts */
import { NgModule }             from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { CanDeactivateGuard } from './guards/';
import { ThemeParksComponent } from './theme-parks/theme-parks.component';
import { ThemeParksListComponent } from './theme-parks/theme-parks-list/theme-parks-list.component';
import { ThemeParkDetailsComponent } from './theme-parks/theme-park-details/theme-park-details.component';

const routes: Routes = [
    { path: '', redirectTo: 'parks', pathMatch: 'full' },
    { path: 'parks', component: ThemeParksComponent,
        children: [
            { path: '', redirectTo: 'list', pathMatch: 'full' },
            { path: 'list', component: ThemeParksListComponent },
            { path: 'details/:parkId', component: ThemeParkDetailsComponent, canDeactivate: [ CanDeactivateGuard ] }
        ]
    },
    { path: 'about', loadChildren: 'app/about/about.module#AboutModule' }
];

@NgModule({
    imports: [ RouterModule.forRoot(routes) ],
    exports: [ RouterModule ],
    providers: [
        CanDeactivateGuard
    ]
})
export class AppRoutingModule { }
{% endhighlight %}

The ThemeParksComponents has it's own content plus a router outlet that will be used by it's children. The content here is just a greeting, but it can be a complex or as simple as you'd like. The ThemParksListComponent or ThemeParkDetailsComponent will be loaded in this outlet.

{% highlight html %}
<!-- theme-parks.component.html -->
<h1>{% raw %}{{greeting}}{% endraw %}</h1>
<router-outlet></router-outlet>
{% endhighlight %}

Another thing you may have noticed on the <code>details/:parkId</code> path is the addition of <code>canDeactivate: [ CanDeactivateGuard ]</code>. This allows you to run code when someone goes to leave a page of your app, through navigation within your app or with the browser controls. This guard will call <code>canDeactivate</code> in the component and use it's response to determine if the route change is allowed. The implemented canDeactivate will use a window confirmation that will resolve as a Promise of a boolean value.

{% highlight typescript %}
/* excerpt from /theme-parks/theme-park-details/theme-park-details.component.ts */

canDeactivate(): Promise< boolean > | boolean {
    return new Promise<boolean>(resolve => {
        return resolve(window.confirm('Are you sure you\'re ready to leave the park?'));
    });
}
{% endhighlight %}

The guard can accept an Observable, a Promise, or a simple boolean value. This allows you to either immediately determine if the change is allowed or if you need to do other work first. A key example would be a form. If the form is unchanged, and they want to navigate away, you can immediately return true. If the form is dirty, you'll want to otherwise confirm with the user if they want to leave and potentially lose changes.

## Lazy Loading Modules

As mentioned with the routing, we can create multiple modules and decide to load some portions of routes only when they're needed. This is useful to only load portions of your app when they're used, such as a seldom used sections (or only used by approved users) such as an administration section.

In our example application, we're loading the about module only if they user visits it. This is accomplished with <code>loadChildren</code> in our previous router. Then, you'll see the definition of the AboutModule, as well as it's own router, used for any children.

{% highlight typescript %}
/* from previous routing code */
{ path: 'about', loadChildren: 'app/about/about.module#AboutModule' }

/* /about/about.module.ts */
import { NgModule } from '@angular/core';
import { CommonModule }   from '@angular/common';
import { AboutComponent } from './about.component';
import { AboutRoutingModule } from './about-routing.module';

@NgModule({
  declarations: [
    AboutComponent
  ],
  imports: [
    CommonModule,
    AboutRoutingModule
  ]
})
export class AboutModule { }

/* /about/about-routing.module.ts */
import { NgModule }             from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { AboutComponent } from './about.component';

const routes: Routes = [
    { path: '', component: AboutComponent }
];

@NgModule({
    imports: [ RouterModule.forChild(routes) ],
    exports: [ RouterModule ]
})
export class AboutRoutingModule { }
{% endhighlight %}

## See it in action

You can see the example application in action. The full source code is available in the [mythbusters-angluar](https://github.com/RHGeek/mythbusters-angular) repository under <code>examples/angular-basics</code>. It is a project created with [Angular-CLI](https://github.com/angular/angular-cli) project (version 1.0.1). So you'll need that and their Node / npm prerequisites. If you have that, then all you need to do is run <code>ng serve</code> for a dev server and navigate to [http://localhost:4200/](http://localhost:4200/).

## More to go

This is not an exhaustive example of everything that makes Angular the framework it is, or even all the basics. It's meant to give you a good start and is also a part the talk I gave at DevFest DC. To read the rest of talk, continue with [Myths of Anuglar 2: What Angular Really Is](/posts/2017/myths-of-angular-devfest-dc.html).