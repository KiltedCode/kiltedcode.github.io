---
layout: inner
title: 'Cleaning out Docker Images'
date: 2017-09-28 12:00:00
author: 'John Niedzwiecki'
categories: docker development
tags: docker
featured_image: '/assets/img/erase-me.jpg'
lead_text: 'Some quick commands to remove all Docker containers and images.'
comments: true
---


I currently work in an environment leveraging [Docker](https://www.docker.com). This has made me a huge fan of containers, no matter who you use. In learning Docker, I've also found my own tips and tricks I use. I previously shared with you my experience running two [Angular apps side by side in a Docker container with NGINX](/posts/2017/angular-docker-nginx.html). That was my first step really diving into Docker. For our full system, we have everything set up with a [docker-compose](https://docs.docker.com/compose/) file, configuring all the containers and dependencies we need making it easy to setup, update, and run a full environment. From this, I've found several commands to make it really easy to manage and clean up after myself.

I run a VM with the full system on so I can have a version of the latest full code and for the backend to proxy my local UI code to as I develop in Angular (thank you Angular-CLI for making that [so easy](https://github.com/angular/angular-cli/blob/master/docs/documentation/stories/proxy.md)). Day to day, I usually just update my containers to have the latest code. To do that, I just run

{% highlight shell %}
docker-compose stop
docker-compose pull
docker-compose up -d
watch 'docker-compose ps'
{% endhighlight %}

This will stop all the running containers. Second, it will pull all the latest images, if there are any updates available. Third, it will bring up the containers again, with updates, in [detached mode](https://docs.docker.com/compose/reference/up/) so they will run in the background without me needing to leave the console up. Finally, the last command is just so I can see the state of the containers to make sure everything starts up correctly. Once I'm satisfied, I just ctrl + c and I'm good to go. Nothing ground breaking here.

Every once and a while I like to start clean or sometimes I need to if things go _very_ wrong. To do that I need to remove all the containers and all the images to ensure everything is fresh. That is what this next set of commands will do. Now, this is very useful in my VM as there is nothing else there but this system. If you run this locally, you will get rid of **all** images so be aware that this will act globally on your system, for whatever the projects you may have. That's my one word of caution.

{% highlight shell %}
docker rm $(docker ps -a -q)
docker rmi $(docker images -q)
docker volume rm `docker volume ls -q -f dangling=true`
{% endhighlight %}

This code will remove all the containers, all the images, and lastly the volumes. Three quick and easy commands for when you want to start fresh. These commands are very handy for me and hope they end up being helpful for you too.

<small>Image credit: Jose Aljovin on [Unsplash](https://unsplash.com/photos/Kfk-TPxLVL0), edited</small>