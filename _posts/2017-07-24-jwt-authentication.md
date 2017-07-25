---
layout: inner
title: 'JWT Authentication in Angular'
date: 2017-07-24 22:00:00
author: 'John Niedzwiecki'
categories: angular development
tags: angular javascript jwt
featured_image: '/assets/img/whats-the-token.jpg'
lead_text: 'JSON Web Tokens are a compact, self-contained, and verifiable object perfect for authentication.'
comments: false
---

We all have dreams. We just don't all go sharing those dreams with every thug and ruffians we meet at the local watering hole, even if it is a five star joint. Sometimes, we keep those touchy-feely dreams to ourselves. For those private dream[app]s, we need authentication. One of the popular and easy ways to add authentication is through JSON Web Token (JWT).

# What is JSON Web Token?

JSON Web Token (JWT) is "an open standard that defines a compact and self-contained way for securely transmitting information between parties as a JSON object." ([source](https://jwt.io/introduction/)) The great thing about JWT is they are digitally signed, so the information can be trusted and verified. They can use either a secret or a public and private key pair, depending on if you use HMAC or RSA accordingly. Know all about JWTs? [Jump ahead..](#using-angular)

While JSON Web Tokens can offer a number of uses, the most common case is authentication. A user logs in, gets a token back, and then uses it for every request thereafter. Because it is verifiable, while anyone can _read_ the information it contains, it would have needed to be created by a trusted source. This makes it an ideal method for single sign on solutions.

JWTs consist of three parts: the header, the payload, and the signature separated by dots. The header tells what algorithm was used, looking something like this:

{% highlight json %}
{
  "alg": "HS256",
  "typ": "JWT"
}
{% endhighlight %}

The payload gives you all the information you need and want to know. Here, you'll find some predefined items like expiration time (exp), issuer (iss), subject (sub), and audience (aud), as well as your own information you want to include like email, name, user roles. An example of the payload may look like this:

{% highlight json %}
{
  "exp": 1500951059,
  "iat": "1500944400",
  "sub": "rider@swashbuckler.net",
  "name": "Eugene Fitzherbert",
  "role": "ADMIN",
  "jti": "a5ef7189-66a6-4202-9ace-947017284907"
}
{% endhighlight %}

The last piece of the token is the signature. This is used to verify the token. In the case of HMAC you take the header and the payload, encode them to base 64 and then use the secret with the algorithm to create a signature. It'd work a little bit like this: 

{% highlight javascript %}
secret = "TheSmoulder";
signature = HMACSHA256(encodeBase64Url(header) + "." + encodeBase64Url(payload), secret);
{% endhighlight %}

To get your token, you just put it all together 

{% highlight javascript %}
token = encodeBase64Url(header) + "." + encodeBase64Url(payload) + "." + encodeBase64Url(signature);
// token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDA5NTEwODEsImlhdCI6IjE1MDA5NDQ0MDAiLCJzdWIiOiJyaWRlckBzd2FzaGJ1Y2tsZXIubmV0IiwibmFtZSI6IkV1Z2VuZSBGaXR6aGVyYmVydCIsInJvbGUiOiJBRE1JTiJ9.cTKNHk3Kq3E7grwd56aeZVChILyg273zkx837d_K3xg
{% endhighlight %}

Now that we know what a token is, we need to know how to use it for simple authentication.

# Using Tokens <a name="using-angular"></a>

Using JWT is actually quite simple. The flow comes down to three pieces. The [first step](#getting-token) is setting up your authentication, logging in, receiving a token, and storing it for later. The [second step](#guard-routes) is to protect your routes for only logged in users. The [third step](#authenticated-apis) is passing that token with all requests to your APIs.

## Getting your token <a name="getting-token"></a>

We start by logging in. To do so, we'll need an AuthService. We'll add a simple function to send our credentials to an API and expect a token back, if successful. Then, the service will store that token to local storage, so we can access it when we need it. 

{% highlight typescript %}
import { Injectable } from '@angular/core';
import { Headers, Http } from '@angular/http';

import { Observable } from 'rxjs/Rx';
import 'rxjs/add/operator/map';

@Injectable()
export class AuthService {

  constructor(
    private http: Http
  ) {}

  /**
   * Logins a user.
   * Sends credentials then stores returned token.
   * @param login: user's login: email, username, etc
   * @param password: password for logging in 
   */
  login(login: string, password: string): Observable<any> {
    let basic = btoa(`${login}:${password}`);
    let headers = new Headers({'Authorization': 'Basic ' + basic});
    return this.http.post('/api/v1/login', '', { headers: headers })
            .map((response: any) => {
              let data = response.json();
              localStorage.setItem('token', data['token']);
            });
  }

  /**
   * Logouts a user.
   * Removes token from local storage.
   */
  logout() {
    localStorage.removeItem('token');
  }
}
{% endhighlight %}

## Guard your routes <a name="guard-routes"></a>

Now that we _can_ login, we need to know when we _need_ to log in. To accomplish this, we need to check that we have a token that is still good (not expired) and a guard to enforce the check. Guards add to the route configuration, allowing you to define additional checks affecting navigation, such as if the user can go to (activate) or leave (deactivate) a route. We need to update the AuthService to include a <code>loggedIn</code> function. In order to check the expiration on the token, we'd need to decode payload, then check the <code>exp</code> against the current date and time. We could write all the logic ourself, or we could use a library. Since we've discussed what the JWT, I'm going to assume you could do that if you want to. Instead, I'm going to include [angular2-jwt](https://github.com/auth0/angular2-jwt) from Auth0. I'll leave the setup better to their README. When we use their helper library, our loggedIn is very simple to add.

{% highlight typescript %}
// other imports...
import { tokenNotExpired } from 'angular2-jwt';

@Injectable()
export class AuthService {
  // ...

  /**
   * Helper function to check if user is logged in.
   * User is logged in if stored token is not expired.
   */
  loggedIn() {
    return tokenNotExpired('token');
  }
}
{% endhighlight %}

Now that we have our own helper function in our AuthService, we can create a AuthGuard leveraging this check. Our guard should implement CanActivate. This function will check if the user is logged in and if not, navigate them to the login page and pass a query parameter of where to return after a successful log in. Alternatively, you could simply show a login form.

{% highlight typescript %}
import { Injectable } from '@angular/core';
import { Router, CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable()
export class AuthGuard implements CanActivate {

  constructor(
    private authService: AuthService, 
    private router: Router
  ) { }

  /**
   * Checks if route can activate, if user is logged in.
   * Redirects to login page if not logged in
   */
  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
    if(this.authService.loggedIn()) {
      return true;
    } else {
      this.router.navigate(['${siteName}/login'], { queryParams: { returnUrl: state.url }});
      return false;
    }
  }
}
{% endhighlight %}

Lastly, we just need to add the guard to any routes we want to be authenticated.

{% highlight typescript %}
// your routing module ...

{ path: 'tower', component: HiddenTowerComponent, canActivate: [ AuthGuard ]  }
{% endhighlight %}

## Call authenticated APIs  <a name="authenticated-apis"></a>

The last step is making authenticated API calls. To authenticate with JWT token, we need to add an Authorization header with the Bearer schema. The header will look like:

{% highlight text %}
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDA5NTEwODEsImlhdCI6IjE1MDA5NDQ0MDAiLCJzdWIiOiJyaWRlckBzd2FzaGJ1Y2tsZXIubmV0IiwibmFtZSI6IkV1Z2VuZSBGaXR6aGVyYmVydCIsInJvbGUiOiJBRE1JTiJ9.cTKNHk3Kq3E7grwd56aeZVChILyg273zkx837d_K3xg
{% endhighlight %}

To accomplish this is easy with <code>angular2-jwt</code>. The library provides <code>AuthHttp</code>, to allow you to send authenticated requests. You simply use it the same way as you use HTTP. In addition, with the release of Angular 4.3, HttpInterceptors. You can use this to add the token to every request yourself. I'll have more on this on my next post covering role based Authorization. Angular2-jwt is being written to use interceptors as well, which you can get currently via their 1.0 prerelease branch.

# Wrap Up

As you can see, JSON Web Tokens are a simple and easy to use. There are [tons of libraries](https://jwt.io/#libraries) to make it easy to roll your own in the language of your choice or plenty of services that can manage it for you, like [Auth0](https://auth0.com). There are also great helper libraries like [angular2-jwt](https://github.com/auth0/angular2-jwt) to make it easy to implement on the front end.