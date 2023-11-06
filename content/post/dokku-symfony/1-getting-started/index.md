---
title: "Getting Started with Symfony and Dokku"
date: 2019-02-18T05:00:00-05:00
series:
  - "Dokku & Symfony"
tags: [ "dokku", "symfony"]
categories: [ "tutorial" ]
---

This is the first in a series of posts on integrating Dokku and [Symfony](https://symfony.com) and will cover setting up a barebones Symfony app and get it running on Dokku.

<!--more-->

{{% aside/note "A PaaS of Our Own" %}}
In a previous post ([A PaaS of Our Own: Dokku]({{< ref "/post/paas-of-our-own">}})) we walked though installing Dokku using DigitalOcean's One-Click Installer and configuring it. This series will build off the work we did in that post.
{{% / aside/note %}}

## Barebones Symfony

Before getting _creative_, we want to just get the most barebones of Symfony applications running. No database, no Encore, just a single controller with two routes and two views.

{{% aside/warning "Operating System" %}}
All of the commands, unless otherwise stated, are run on our local computer. I am using a Mac (macOS 10.14 Mojave), though everything should still run fine on Linux. I honestly don't know how to set this up on Windows.
{{% /aside/warning %}}

### Creating the Project

To get started, let's create a new Symfony project called `sf-demo`.

```bash
composer create-project symfony/website-skeleton sf-demo

# ... a whole bunch of installing

cd sf-demo
```

## First Thing: Source Control

{{% aside/warning %}}
> **Tip 23: Always Use Source Code Control—Always.**
>
> Even if you are a single-person team on a one-week project. Even if it's a "throw-away" prototype. Even if the stuff you're working on isn't source code. Make sure that *everything* is under source control -- documentation, phone number lists, memos to vendors, makefiles, build and release procedure, that little shell script that burns the CD master—everything.
>
> — The Pragmatic Programmer, Hunt / Thomas

{{% /aside/warning %}}

Before we do _anything_ else, we need to set up out git repository. Not only will we need to git to push our application to Dokku later, it is just a smart thing to do.

```bash
git init

# Initialized empty Git repository in /Users/dev88/Projects/sf-demo/.git/

git add .
git commit -m 'Initial Commit'
```

## Just a Homepage

For our barebones app we are going to add a home page and a "more information" page, with both pages being handled by a single Home controller.

### Start Our Working Branch

{{% aside/note "Branching Out" %}}
Before we work on a feature or fix bugs, we will create a [git branch](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging) for our work and then merge that code back into our `master` branch when we are done.

This is a good practice to be in and works well with both [GitLab Flow](https://about.gitlab.com/2014/09/29/gitlab-flow/) and [GitHub Flow](https://guides.github.com/introduction/flow/).
{{% /aside/note %}}

To start up a new branch for building our homepage, we run the following commands:

```bash
git checkout -b homepage
```

This creates a new `homepage` branch and then switches to the new branch.

{{% aside/note "CLI vs. GUI" %}}
In this series I will be using the `git` command for working with our repository. There are [a lot of good git GUI apps](https://git-scm.com/downloads/guis/) that you can use if you are not a fan of the command line. On macOS I like [Fork](https://git-fork.com/) (free) and [Tower](https://www.git-tower.com/) (paid) (and use both regularly).
{{% /aside/note %}}

### `make`ing a Controller

The `website-skeleton` project we created includes [the Symfony MakerBundle](https://symfony.com/doc/current/bundles/SymfonyMakerBundle/index.html) that we can use to create a new controller.

{{% aside/note "bin/console Alias" %}}
I have set up `sf` as [an alias](http://www.tldp.org/LDP/abs/html/aliases.html) for `bin/console` to save a lot of typing. You can create the same alias by running `alias sf="bin/console"`. To make the alias permanent, need to add that command in your `.bashrc` or `.zshrc`.
{{% /aside/note %}}

```plain
$ sf make:controller
```

Enter `HomeController` as the name for your controller class when prompted. Once the controller is created, we can edit the contents of `src/Controller/HomeController.php` to be the following:

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class HomeController extends AbstractController
{
    /**
     * @Route("/", name="home")
     */
    public function index()
    {
        return $this->render('home/index.html.twig');
    }

    /**
     * @Route("/more", name="more_info")
     */
    public function more()
    {
        return $this->render('home/more.html.twig');
    }
}
```

### Twig Templates

The MakerBundler also created the file `templates/home/index.html.twig`. Update the contents of this file to be:

```
{% extends 'base.html.twig' %}

{% block title %}Hello Dokku!{% endblock %}

{% block body %}
    <h1>Hello Dokku</h1>

    <p>
        <a href="{{ path('more_info') }}">
            More information
        </a>
    </p>
{% endblock %}
```

Finally, create a new file `templates/home/more.html.twig` and make its contents:

```
{% extends 'base.html.twig' %}

{% block title %}More Information{% endblock %}

{% block body %}
    <h1>More Information</h1>

    <p>This is some more information.</p>

    <p>
        <a href="{{ path('home') }}">
            Back Home
        </a>
    </p>
{% endblock %}
```

### Test Our Pages

We can now test are two pages by starting up [a web server](https://symfony.com/doc/current/setup/built_in_web_server.html):

```bash
sf server:run
```

Go to `http://localhost:8000` and we should see our "Hello Dokku" title, and a working link to the "More information" page (which should link back to the homepage). Once you have checked and made sure everything works, you can kill the server with ^C (control-C).

{{% aside/warning "run vs. start" %}}
The Symfony documentation instructs you to use `server:start` and then run `server:stop` when you are done. The benefit to this is that the server is run in the background and you can keep using the console. `server:run` blocks the console and you see all of the logging as you use your site.

I have problems with `server:start`, and I will admit that its _something_ on my computer. If `server:start` works for you, feel free to use it. For now, I have to use `server:run`.
{{% /aside/warning %}}

## Commit and Merge Our Changes

If everything looks good, then you should commit your changes to the repository:

```bash
git add .
git commit -m 'Add home and more info pages'
```

and then merge our changes into `master` and remove the `homepage` branch:

```bash
git checkout master
git merge homepage
git branch -D homepage
```

## Dokku Prep

{{% aside/note "A Big Thank You" %}}
[This article by Matt Brunt](https://mfyu.co.uk/post/symfony-and-dokku) was a _huge_ help in figuring this out. A giant "thank you" to Matt for the groundwork.
{{% /aside/note %}}

Before we can push our Symfony app to Dokku, we need to get Dokku ready.

### Local Dokku Client

Before we go any further, set up the [Dokku Client](http://dokku.viewdocs.io/dokku/community/clients/) so you do not have to SSH into the server to run all of the commands for app setup.

To get the client to talk to your server you either need to set the `DOKKU_HOST` environment variable or be in a git repository with a remote named `dokku`. We are going to set the `DOKKU_HOST` environment variable the first time we run it here so that both the app and the remote will get created.

{{% aside/note "Server Name" %}}
My server is `dev88.xyz` - use whatever domain name (or IP address) you have configured for your Dokku server.
{{% /aside/note %}}

```plain
DOKKU_HOST=dev88.xyz dokku apps:create sf-demo
   -----> Dokku remote added at dev88.xyz called dokku
   -----> Application name is sf-demo
   -----> Creating sf-demo... done
```

Then we can check our git remotes and make sure the Dokku has been added:

```plain
git remote -v
   dokku   dokku@dev88.xyz:sf-demo (fetch)
   dokku   dokku@dev88.xyz:sf-demo (push)
```

Now our server is set up, we need to prepare our app for deployment.

### Nginx Configuration

To set up Nginx to run Symonfy's rewrite rules, we add a `nginx.conf` file to the root our app with the following content:

```nginx
location / {
    # try to serve file directly, fallback to rewrite
    try_files $uri @rewriteapp;
}

location @rewriteapp {
    # rewrite all to index.php
    rewrite ^(.*)$ /index.php/$1 last;
}

location ~ ^/index\.php(/|$) {
    try_files @heroku-fcgi @heroku-fcgi;
    internal;
}
```

### Add the Procfile

We also need to create a file named `Procfile` in the root of our app. This file will tell Dokku to use the `nginx.conf` file.

```plain
web: $(composer config bin-dir)/heroku-php-nginx -C nginx.conf public
```

### Setting Environment Variables

Finally we set the environment variables Symfony needs:

```plain
dokku config:set APP_ENV=prod
dokku config:set APP_SECRET=$(head /dev/urandom | LC_ALL=C tr -dc A-Za-z0-9 | head -c 32)
```

### And DEPLOY!

Everything is now ready and we can push our application up to Dokku:

```bash
git push dokku master
```

#### (Maybe) Things Go Horribly Wrong

When I did this, I got the following:

```plain
!     ERROR: Failed to download minimal PHP for bootstrapping!
!
!     This is most likely a temporary internal error. If the problem
!     persists, make sure that you are not running a custom or forked
!     version of the Heroku PHP buildpack which may need updating.
! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'dokku@dev88.xyz:sf-demo'
```

I had to log into my server and run an upgrade (there commands are all on the server):

```bash
sudo apt-get update
sudo apt-get upgrade
```

The culprit was needing a newer version of Herokuish. Once that was updated, everything worked. Once the upgrade completed, I could push my app to Dokku.

#### And Possibly _More_ Errors

When I tried to do `apt-get update`, I received an error about not having the proper GPG key for the packagecloud.io repository that handles Dokku. To fix this, I followed the [manual install instructions for Dokku's packagecloud.io page](https://packagecloud.io/dokku/dokku/install#manual-deb). Specifically the following command:

```bash
curl -L https://packagecloud.io/dokku/dokku/gpgkey | sudo apt-key add -
```

After that command, I could `apt-get update` and `apt-get upgrade`.

## Adding HTTPS

You should now be able to visit your app at `http://sf-demo.dev88.xyz` (replace `dev88.xyz` with whatever your domain is). However, we really want that `http` to be `https`. To do that we can run:

```bash
dokku letsencrypt sf-demo
```

Let that command run and then refresh your site. It should automatically forward you the `https` version.

{{% aside/note "What is Available on the Server?" %}}
When building new features, we may need specific PHP extensions installed. To find out what is _already_ installed by the PHP buildpack we are using, we can run the following command:

```plain
dokku run 'php -i'
```

This will dump the PHP configuration and we can check installed extensions, server and environment variables, and all other PHP configuration values.
{{% /aside/note %}}

## Logging

Currently, our app is logging its errors to `var/logs/prod.log`. This is _fine_, however, those logs will get destroyed during each deploy when our app's container gets rebuilt. There are two ways to handle this:

1. [Mounting persistent storage](http://dokku.viewdocs.io/dokku/advanced-usage/persistent-storage/) to our app and writing the logs to there. We will cover mounting persistent storage for file uploads in a later part of this series.
1. Use a log management provider like [Papertrail](https://papertrailapp.com/) or [Loggly](https://www.loggly.com/).
1. If you only care about errors, use a error tracking service such as [Rollbar](https://docs.rollbar.com/docs/symfony), [Bugsnag](https://docs.bugsnag.com/platforms/php/symfony/), or [anything supported by Monolog](https://github.com/symfony/monolog-bundle/blob/master/DependencyInjection/Configuration.php#L25).

For this tutorial, we will just be leaving the logs in `var/logs/` and not worrying about them being deleted on each deployment.

## SUCCESS!

We should now have a barebones Symfony up and running on our Dokku server, it should be served over HTTPS, and we are ready to flesh it out.

{{% aside/success "What Next?" %}}
Now that we have a basic Symfony site up and running we can start adding features to experiment with getting them working on Dokku.

In the next post in this series, add a some style to our site with Encore and Bootstrap.

Or start experimenting on your own. I have confidence in you.
{{% /aside/success %}}
