---
layout: post
title: "Batman as a CouchApp"
date: 2013-04-08 11:13
comments: true
categories: [Batmanjs, CouchDB, CouchApp]
---

# It was a dark and stormy night... #

Okay, not really.  It was actually really hot and only slightly hazy.  I was in Jordan field testing OpenDig, an iOS application which took advantage of [CouchDB](http://couchdb.apache.org) for easy replication, synchronization and portability of our archaeological field database.  It was at this point that I decided we really needed to use a  to have an easy way to view the data on the fly.  Sure, we have a full Ruby on Rails application, but I wanted to have some separation between the data being collected in the field and the data that was ready for publication.  It seemed like a CouchApp is the perfect solution; it travels with the data and can provide a structured way to read the data without having to deal with the raw JSON documents.

<!-- more -->

Of course, none of the solutions were appealing to me at the time.  Coming from the world of Ruby on Rails I've found myself loving the beauty of Ruby code and the baked in conventions of Rails.  The various Javascript frameworks I played with just left me feeling like I was missing something.  That was, until I discovered [BatmanJS](http://batmanjs.org) a few weeks ago.  I decided to try and make Batman work for my CouchApp... this is the story of that trial...

![always be batman](/images/batman.jpg)

### Batman to the Rescue! ###

I really love what BatmanJS is doing.  Unlike many of the other Javascript frameworks, it feels like I'm "home."  According to the developers, Batman is:

> ...concise and favors convention over configuration, and packs a powerful system of view bindings and observable properties. The API is designed with developer and designer happiness as its first priority, and while it has no real dependencies, it works quite well with Rails. 
>
> -- <cite>[BatmanJS Homepage](http://batmanjs.org)</cite>

But, as excited as I was about BatmanJS, I was even more excited to see if I could stuff it into a CouchApp and have a beautiful framework that I could use to create the basic data reader that was needed for the field.

I should say that I'm not really following all the principles of the framework as I actually am not using the models as they should be used.  Rather than using the *Batman.RestStorage* option, I just moved to using Ajax calls in the controller.  This will likely be rewritten later when I decide to have full model functionality, but as this is really just a data reader and not meant to handle editing or creating then the models just seemed a bit unnecessary at the moment.

### The tools ###

First I had to find the "right" couchapp tool.  If you go to the website, you'll see there are several tools listed there.  There seems to be a strong following for [Erica](https://github.com/benoitc/erica) and [Kanso](http://kan.so), and there is also a Ruby tool calls [Soca](https://github.com/quirkey/soca).  A quick glance shows that Erica has had the most recent development, although Kanso seems to be the most comprehensive.  Being a Rubyist, I opted for Soca simply because I knew I could easily hack on the code if I needed to.

Installing Soca is easy, you just install the gem:
```sh
gem install soca
```

You can setup a template for a soca app with:

```
soca generate
```

But you'd probably be better off just creating a few files rather than the whole structure since you are going to follow the Batman JS framework structure instead.

My key files are as follows:

```javascript config.js
{
  "id": "opendig_couch",
  "mapDirectories": {
    "config.js": "",
    "open_dig.js": "_attachments/open_dig.js",
    "index.html": "_attachments/index.html",
    "resources/favicon.ico": "_attachments/favicon.ico",
    "models": "_attachments/models",
    "controllers": "_attachments/controllers",
    "css": "_attachments/css",
    "img": "_attachments/img",
    "sass": false,
    "resources": "_attachments/resources",
    "views": "_attachments/views",
    "resources/descriptions.json": "_attachments/descriptions.json",
    "db": "",
    "*.dump": false,
    "Jimfile": false,
    "hooks": false,
    "build": false,
    "*.coffee": false,
    "Gemfile": false,
    "Gemfile.lock": false
  },
  "plugins": [
    "coffeescript"
  ]
}
```
The **false** option allows you to prevent a file from being pushed to the couchapp.  This is particularly useful if you don't want the coffee files, config files, or other random files being pushed.  When you need specific things pushed to specific areas of the couchapp (look at the above lines), then you can connect them directly to the attachments section of the design document.

That is really all one needs to do to make the app actually work and get pushed to a CouchApp.  However, there are a few other things that have to be dealt with to really make things work.  For example, the view templates are cached in CouchDB meaning you need to add a timestamp to keep the fresh.  Likewise, if you want pretty URLs, you will need to add a specific line in the application controller to add the necessary path information.

```coffeescript application_controller.coffee
class OpenDig.ApplicationController extends Batman.Controller
  # Add filters or functions you would like all your controllers to inherit to this controller.
  if "_design" in document.location.pathname.split("/")
    Batman.config.viewPrefix = "#{document.location.pathname.split('/')[1]}/_design/opendig_couch/views"
```

Now, here is an example of retrieving a map/reduce view through an ajax call in the controller

```coffeescript squares_controller.coffee
class OpenDig.SquaresController extends OpenDig.ApplicationController
  routingKey: 'squares'

  show: (params) ->
    response = null
    @square = params.id
    @square_label = "Square #{@square} - Loci"
    startkey = 'startkey=["' + @square + '"]'
    endkey = 'endkey=["' + @square + "\\ufff0" + '"]'
    $.ajax
      type: "GET",
      async: false,
      cache: false,
      url: '_view/by_square?group=true&' + startkey + '&' + endkey,
      success: (data, textStatus, jqHXR) ->
        response = JSON.parse(data).rows
    @loci = response
    @render source:"squares/show.html?#{(new Date).getTime()}"
```

There are still a few bugs I'm working out, but little adventure has been more about testing out the idea rather than making a polished product.  The code will likely be overhauled in the coming weeks as I patch the obvious bugs, figure out the best way to test this and start cleaning it up.

Hopefully this will help someone else who is trying to do something similar with Batman JS!

Check out the full project on [Github](https://github.com/neshmi/OpenDigCouch) 


