---
layout: inner
title: 'Running Angular from Docker with NGINX'
date: 2017-02-02 21:25:00
author: 'John Niedzwiecki'
categories: angular development
tags: angular javascript docker nginx config
lead_text: 'An Angular present with Docker wrapping paper and an NGINX bow on top.'
comments: true
---

In the latest project I've been working on at work, we're taking a new approach to deployment. We're using Docker containers for everything, which for our team is something new but useful.
We plan to deploy to the cloud (provider not solidified at this time) so it makes it easy to spin up more instances when your pieces are already in containers. 
It is also great for our test team, who can now simple grab the Docker images and run their own containers wherever and however they need. 
We already have Jenkins set up to build the images nightly and on demand, as well as then push them to our own Artifactory, so that process is fairly smooth.
This post is not about our decision to use Docker (the good, the bad, or the ugly). This is about my experience figuring out how to make a Docker container for the UI code, an Angular app (version 2+).

I'm going to mostly follow the progression that I did when getting to what would be our final product for our Docker container. 
In reality, this also follows the examples from simplest to adding more complexity with each step anyway. 
The base goal is simple: a Docker container with NGINX inside to serve up the Angular app. 
We'll build on each step, starting with a single Angular app inside a Docker container until the final example of a parameterized Docker container running two Angular apps while NGINX proxys all API calls.

## Dependencies

[Angular-CLI](https://github.com/angular/angular-cli): I am currently using Angular-CLI to build my application, so the instructions (and example code) will assume this. You'll need to adapt for your own build methods. 
I personally find Angular-CLI very useful, though opinionated. It has additional dependencies itself of Node 4+ and NPM 3+.

[Docker](https://www.docker.com/): This is pretty obvious. We want to make Docker containers, so we'll need Docker. The installation of Docker for Mac, was very simple.
Just follow their 'Get Started' link and find the appropriate installation for your environment.

## Code

All of the code shown below, and examples to run can be found in my GitHub Repo [angular-docker-nginx](https://github.com/RHGeek/angular-docker-nginx) (creative name, I know). 
At the end of each section, I'll point you to the right folder for the full code example.

# Angular + Docker + NGINX

My first attempt was to simply get a Docker container running with NGINX. Then, I needed to deploy a build of the Angular app within the container.
This was my first time working with Docker at all, so my research started with how to create a Docker container with NGINX. That proved to be quite easy.
To do this, we simply need a file name <code>Dockerfile</code> telling it to create the image from NGINX and to expose port 80 for web traffic.

{% highlight conf %}
FROM nginx

EXPOSE 80
{% endhighlight %}

Next, we need to add the UI code. For that, we first need to build the project. With Angular-CLI, we can target a production build with <code>ng build --prod</code>. 
Any ng build will produce output in the <code>dist</code> directory. If this is your first time working with the Angular-CLI project, you'll want to run <code>npm install</code> first. 
Now, we simply need to modify Dockerfile to copy this folder into the proper directory for NGINX.

{% highlight conf %}
FROM nginx

COPY dist /usr/share/nginx/html

EXPOSE 80
{% endhighlight %}

This alone _almost_ worked for me. See, if you start the container and go to the root, it will load the index.html and your app will run and function properly.
However, if you then try to refresh on a route, your app will fail to load. The problem is that NGINX sees the path and will try to load a file in that directory structure, 
instead of loading the index page and letting the Angular routing take over. This is a problem we can easily solve, by providing our own configuration for NGINX. 
What you'll see in our <code>default.conf</code> is a copy of the default.conf that the container creates, with a small addition, a mapping for the root location.

{% highlight nginx %}
location / {
    try_files $uri $uri/ /index.html;
}
{% endhighlight %}

This instructs NGINX to first try  what is provided, then try it with a trailing slash, and if both of those fail, use the index.html file on the root. '
This will allow any actual directory files to be loaded, such as any assets for extra JS files, but if no file is found, it will load the app and then assume it should be a route. 
The last piece need is to modify the Dockerfile to copy in our version of the default.conf.

{% highlight conf %}
FROM nginx

COPY dist /usr/share/nginx/html
COPY docker/nginx-conf /etc/nginx/conf.d/

EXPOSE 80
{% endhighlight %}

In order to create a Docker container, we first need to create a Docker image. To do this, we use the [docker build](https://docs.docker.com/engine/reference/commandline/build/) command. 
The link provide will show all the options you can use when building an image from a Dockerfile. For now, we'll simply add a tag, to give it a more friendly name.
Run the command <code>docker build -t simple-angular-nginx .</code>

This creates an image with the tag of simple-angular-nginx. Lastly, you need to run a container using this image. 
This can be done simply through <code>docker run --name simple-angular -p 80:80 -dit simple-angular-nginx</code>. 
This create a container to run, named simple-angular from the simple-angular-nginx image. 
It will bind the exposed port 80 to port 80, though you can change this by altering the first number, such as <code>8888:80</code>. 
Lastly, <code>-dit</code> are some options you can chose to pass or not. The <code>d</code> runs it in detached mode, meaning it will run in the background and not need the console to keep running. 
The <code>i</code> and the <code>t</code> each have their own meanings, but are often used together to allocate a tty for the container process. 
The short of it is either pass <code>-dit</code> or <code>-it</code>, your choice, but from my understanding, best to use one or the other. 
If you run with <code>-it</code>, you can simply use ctrl+c to stop the container. If you use <code>-dit</code>, you'll need to run <code>docker stop simple-angular</code>

The code for the first section can be found under <code>simple-angular-nginx</code>

# Adding API Proxy

The next step is to handle our API calls. Our application could very easily be configured where to point to for the REST endpoint. 
However, one of the nice points of using NGINX is the ability to set up proxys. 
We can configure NGINX to proxy all calls to <code>/api</code> to any other location. For this example, we'll just proxy them to another port locally.
Everything else about our build process and configuration will remain the same, but we'll add the /api location to our default.conf file.

{% highlight nginx %}
location /api {
    proxy_pass http://localhost:8080;
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    proxy_buffering off;
    proxy_set_header Accept-Encoding "";
}
{% endhighlight %}

The updated default.conf can be found under <code>angular-nginx-proxy</code>

# Adding Build Parameters

We've set our app up with a proxy, but the server needs to be set in the conf file provide. What if we want to set that dynamically, so we can pass in a server and port? 
Docker has you covered. First, we need to make some modifications to our default.conf file. 
We'll take the existing conf file, and template the portions we'll later want to replace with environment variables. 
The two values we want to be dynamic are the host and the port for the server of our REST API. 
In addition, we're also going to modify it to use and upstream name. This simplifies the file by listing the server at the top, but would also allow us to use multiple servers, if we so wished. 
You can read more about this from the NGINX documenation on [upstream](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream). 
We'll rename the new file <code>default.template</code> to make it clear this is no longer a valid conf file.

{% highlight conf %}
upstream ui_rest {
    server ${UI_REST_HOST}:${UI_REST_PORT};
}

server {
    # additional configuration... 

    location /api {
        proxy_pass http://ui_rest;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_buffering off;
        proxy_set_header Accept-Encoding "";
    }

    # additional configuration... 
}
{% endhighlight %}

The next step is to modify the Dockerfile to apply the enviornment variables to the template, when the container is run. 
In order to execute this, we use the <code>CMD</code> instruction. 
You can read more about it on the [Docker reference](https://docs.docker.com/engine/reference/builder/#/cmd) but it's main purpose is providing defaults to the container. 
This can be leveraged to take our template, and create a working conf file after applying the enviornment variables.

{% highlight conf %}
FROM nginx

COPY dist /usr/share/nginx/html
COPY docker/nginx-conf /etc/nginx/conf.d/

EXPOSE 80

CMD /bin/bash -c "envsubst < /etc/nginx/conf.d/default.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"
{% endhighlight %}

This is one of those moments where I can't say for certain if this is the _best_ way to do this, only that it is a way to do this. 
Remember, this is all a part of the first time I've used Docker, so if you know of a better or more standard way to do this, please let me know in the comments. 

The last step is to provide the environment variables. This is done during the run command. 
After building our image, which we'll assume we tagged as angular-build-params, we add our environment variables with the <code>-e</code> flag. 
Our run command will be <code>docker run --name angular-built -p 80:80 -e UI_REST_HOST='192.168.1.11' -e UI_REST_PORT=8080 -dit angular-build-params</code> 
You'll see both the host (I'm just using a local IP address this time) and the port being passed in that will be substituted into the upstream named ui-rest.

This all looks fine until you run it. Then, you'll notice one problem: deep linking to routes no longer works. Why is that? 
If we look at our environmental variables in our template, they look like <code>${UI_REST_HOST}</code>. 
The <code>envsubst</code> looks for the dollar sign and then replaces what is inside the curly brackets with a matching enviornment variable. 
If we take a look at the conf file within the container, we can see our problem. 
To do this, we can use [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) to run a command on a running container. 
If we run <code>docker exec -it angular-built bash</code>, we can navigate to the conf.d directory and look at our file. 
Since the problem is with the routes, and we know we originally fixed that in the first section with <code>location /</code>, we'll start our investigation there. 
Here, we see the problem. During the substitution, the dollar sign is dropped from $uri, therefore breaking our fix.

The solution I used for this? Another substitution! The modification made updates the $uri to ${DOLLAR}uri, which we'll then give an environmental variable of $. 
This definitely feels like a hacky (_read:_ non-elegant) solution, but it does help make a working configuration. Modify the default.template to include the new location block

{% highlight conf %}
location / {
    try_files ${DOLLAR}uri ${DOLLAR}uri/ /index.html;
}
{% endhighlight %}

Next, we'll need to rebuild the image to include the updated template, with the same commands as before. 
Then, we just pass in another environment variable, for our new DOLLAR variable, making our run command 
<code>docker run --name angular-built -p 80:80 -e UI_REST_HOST='192.168.1.11' -e UI_REST_PORT=8080 -e DOLLAR='$' -dit angular-build-params</code>

The updated default.template and Dockerfile can be found under <code>angular-docker-env</code>

# BOGO: Two Angular Apps in One Container

Let's say we want to run two separate Angular apps served up from one Docker container. We're saying this, because I needed to, so we will. 
We want our main app to continue living under the root of our container, but we have a secondary app, side-angular, that we want to live under the directory <code>/sideapp</code>. 
The two apps are just that: two separate applications. They may be sharing the same house, but they want their own space, so we're going to add a basement apartment.
How can we do this?

My first thought was to take the app, copy it's <code>dist</code> folder into it's own directory in NGINX, then configure another location to serve it's base HTML page for routing. 
Those changes are simple enough to make, and just build on what we've already done. First, we'll update the Dockerfile from our last example. 
We'll add an additional copy and update it to be relatively above both applications.

{% highlight conf %}
FROM nginx

COPY side-angular/dist /usr/share/nginx/html/sideapp
COPY main-angular/dist /usr/share/nginx/html
COPY docker/nginx-conf /etc/nginx/conf.d/

EXPOSE 80

CMD /bin/bash -c "envsubst < /etc/nginx/conf.d/default.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"
{% endhighlight %}

Next, we need to update the default.template to account for the new directory. 
It needs to know when to serve up the proper index.html and still keep the routing working properly. 
For that, we add a location, like we did with the route, only that last fallback should point to the index within the sideapp directory. 

{% highlight conf %}
location /sideapp/ {
    try_files ${DOLLAR}uri ${DOLLAR}uri/ /sideapp/index.html;
}
{% endhighlight %}

Now, this may seem like everything we need to get this working, but there is one gotcha. 
In order to have the second app work in a subdirectory, we need to build it with an extra parameter. 
We need to tell Angular-CLI to change the baseref of the application. 
Otherwise, when it tries to load its JS files and other assets, it is going to be looking at the root level, instead of within /sideapp. 
When we're first building our Angular apps to get the dist directory, we pass the --base-href flag (or --bh) with the base we want to use. 
For our side-angular app, it would be <code>ng build --prod --bh /sideapp/</code>

Now, build the Angular apps, create the docker image, and run the docker container (with enviornmental variables) and you'll have 
two Angular applications running in one Docker container that is proxying all API calls to another location via run time enviornment variables.

{% highlight shell %}
cd side-angular
ng build --prod --bh /sideapp/
cd ../
cd main-angular
ng build --prod
cd ../
docker build -t bogo-angular-apps .
docker run --name bogo-angular -p 80:80 -e UI_REST_HOST='192.168.1.11' -e UI_REST_PORT=8080 -e DOLLAR='$' -dit bogo-angular-apps
{% endhighlight %}

The code for the final section can be found under <code>bogo-angular</code>

# Wrap Up

Working with Docker has been an interesting experience so far. 
I've had some experience with NGINX on my last work project and there is a lot of tricky things you can do with the configuration. 
We leveraged a couple of them here in getting our Docker container to work properly. It makes for a simple solution to stand up your Angular code. 
In the end, I found a working solution for what I needed to accomplish. It took a lot of cobbling solutions together to get what I needed, so I felt it was worth the share.
I look forward to digging deeper to see if there are better ways to set this up. Are you experienced with Docker and have a better way? Please share.