+++
title = "Where am I?"
author = ["Ryan Cummings"]
date = 2020-05-08T21:18:00-04:00
lastmod = 2020-05-08T21:19:18-04:00
categories = ["personal"]
draft = false
+++

It's been a while since my last update, but I'm fine! For historical context, Coronavirus has pretty much swept the US at this point, and NYC has been in lockdown since March. Cases and deaths are down, and there is talk that a gradual return to normal will begin in a week, but I am not so sure we'll see it.

What have I been doing with all of this time? A few things:

-   Finally read [Gödel, Escher, Bach: An Eternal Golden Braid](https://www.amazon.com/G%C3%B6del-Escher-Bach-Eternal-Golden/dp/0465026567) by Douglas Hofstadter, an incredible book exploring recursion, ideal for the more left-brained
-   Brush up on [Django 3](https://www.djangoproject.com/), a python-based web framework
-   Learn to use the [Celery](http://www.celeryproject.org/) scheduler and [RabbitMQ](https://www.rabbitmq.com/) message broker in sync with Django to handle background processes for Django sites
-   Hammered through [Javascript 30](https://javascript30.com/), a free vanilla Javascript course composed of 30 small projects
-   Deployed two tiny projects:
    -   [SimpleSearch](http://app.ryanwcummings.com/simplesearch/), a quick way to search a Reddit subreddit for recent posts
    -   [Sudoku Solver](http://app.ryanwcummings.com/sudoku/), which does what it sounds like, written primarily in javascript
-   Brief aside with [Phaser 3](http://www.phaser.io), a JS game engine for web browsers, and Node.JS, a faster asynchronous (and ubiquitous) tool often used as an asynchronous web server.
-   Last but not least, [the Django REST Framework](https://www.django-rest-framework.org/), which lets you quickly create web APIs. Next up is [Vue.js](https://vuejs.org/), a javascript framework to make webapps. With these under my belt, it should be easy to design web services and websites/apps to access them.

My deep learning research elective was a huge success -- I achieved 90% accuracy training a neural network to identify TB on CXR. See some heatmaps below:

{{< figure src="/img/misc/CHNCXR_0329_1_heatmap.png" link="/img/misc/CHNCXR_0329_1_heatmap.png" >}}

{{< figure src="/img/misc/CHNCXR_0331_1_heatmap.png" link="/img/misc/CHNCXR_0331_1_heatmap.png" >}}

I may end up deploying this with Django. The results are good (not great, but good), so it may be worth it.

I guess this post turned into more of a diary, but it's always good to get some thoughts out there. Hopefully this is useful to at least one person looking for something to do. For now I'll keep stumbling down the path, hoping I end up somewhere interesting.
