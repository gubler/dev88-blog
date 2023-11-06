---
title: "Backups, Cron, Email, and Locks"
date: 2019-02-18T05:05:00-05:00
series:
  - "Dokku & Symfony"
tags : [ "dokku", "symfony", "backups", "email", "cron"]
categories : [ "tutorial" ]
---

There are some other common topics for Symfony development that can be affected by using Dokku but either they are not large enough for their own articles or I do not have sufficient expertise to do more that scratch the surface.

<!--more-->

## Backups

There are four parts of our system that require backup.

{{% aside/danger "Encryption" %}}
I want to point out that none of our data is encrypted (in the database, files on disk, in our backups, or in use). If you are working with sensitive information, please research proper data security measures and/or hire a specialist.
{{% /aside/danger %}}

### Application Code

Our application code is currently on our development machine and in Dokku. We should also keep a copy of it in a separate remote repository. This can be one that you host on another server, or can one of the popular git hosting providers available online such as [GitLab](https://gitlab.com), [GitHub](https://github.com), or [BitBucket](https://bitbucket.org).

### Files

The easiest way to backup files uploaded to our server is enabling [DigitalOcean's automatic backups](https://www.digitalocean.com/docs/images/backups/). This makes a backup of your server every day. Other options would be custom scripts to copy the files to a remote server or even just zip up the files and copy them to a local computer.

### Database

If you have DigitalOcean backups configured for your server, that will also cover your database. If you need more frequent backups or want to create an additional backup, you can use the backup tasks that the [Dokku MariaDB plugin](https://github.com/dokku/dokku-mariadb) we are using provides.

Built into the plugin are commands to backup the database to an Amazon S3 bucket (see the `mariadb:backup` commands). If you want to dump the database to file, you can use the `mariadb:export` command.

### Dokku Configuration

Finally, you should also document and backup all the steps for setting up and deploying your application to Dokku. Everything from adding the git remote, to setting up the database on Dokku, to deploying and initializing your application.

This will ensure that if you ever need to rebuild your Dokku server, or even deploy to a new Dokku server, you know the exact series of steps to get your application up and running.

## Cron Tasks

Even Dokku's documentation will tell you that regularly scheduled tasks with Dokku can be a bit of a pain. [Read through the Dokku documentation for one-off processes](http://dokku.viewdocs.io/dokku/deployment/one-off-processes/) and decide the best way for your application to handle scheduled tasks. Using the Dokku server's cron process you can execute tasks within your app container. Long running or infrequent tasks can be handled with the `dokku run` command. Frequent tasks are probably better handled with the `dokku enter` command.

Pavel Karoukin also has [an article on setting up cron inside a second instance of your application](https://pavel.karoukin.us/blog/dokku-php-apps-and-crontab-tasks), which can be a good way to keep everything within your application container and not have to set up cron on Dokku server separately.

## Sending Emails

To send emails, we need to access to a SMTP server. Running our own SMTP server (using software like [Sendmail](https://en.wikipedia.org/wiki/Sendmail) or [Postfix](http://www.postfix.org/)) is hard to do securely and is also hard to maintain. There are also problems with sending a large number of emails from a local server - doing so can easily get our emails flagged by internet email providers (like Gmail) as spam.

The option is to use a transactional email provider [Mailgun](https://www.mailgun.com/), [Sendgrid](https://sendgrid.com/), or [Amazon SES](https://aws.amazon.com/ses/) (there a lot of options). Many providers supply a generous number of free emails per month (great for testing or tiny applications), deal with all the hassle of sending a huge amount of email and not getting flagged as spam, and handling software updates and security. I _highly_ recommend using one of these[^mailgun].

Once you have your SMTP server available (either set up on your own or from a SaaS provider), the you need to provide the connection information as an environment variable in Dokku.

```plain
dokku config:set MAILER_URL={your connection string}
```

[^mailgun]: I use [Mailgun](https://www.mailgun.com/).

### Mail Spools

Swiftmailer [has support for spooling emails](https://symfony.com/doc/current/email/spool.html) for sending later or in batches. For most providers, we should be able to use the standard `memory` spool (which sends emails right before Symfony's kernel terminates).

However, if our provider has a low rate limit (number of emails that can be sent in an amount of time) and our application grows past that limit, you may have to switch to the `file` spool. To store the spooled emails we can either put them in the application container _or_ we can mount a file store to hold them (as we did to [store uploaded files](#)).

Using the `file` spool, has some risks with Dokku and deploying new versions of our application:

- If we store the emails within the application container, when we deploy a new version of our application and spooled emails will get deleted.
- If we store the emails in a mounted directory, they will persist between deployments, but [the warning at the bottom of the spooling documentation](https://symfony.com/doc/current/email/spool.html) says there is a risk with spooling emails to file and clear the cache. Clearing the cache is a small part of deploying a new version of out application.

However, in my testing, I could not replicate the problem in the warning. If you need to spool emails to file, then definitely mount a directory to store your spooled emails, but be aware that you may be unable to send spooled emails if you deploy a new version of the application.

#### Sending Spooled Emails

Sending the spooled emails is best handled by setting up a cron job to run the console command:

```plain
php bin/console swiftmailer:spool:send
```

You will need to add either the `--message-limit` or `--time-limit` options to ensure you do not cross your provider's rate limit.

{{% aside/note "By CRON!" %}}
You can run this command by using the cron process we discussed above.
{{% /aside/note %}}

## Locking

{{% aside/warning "TL;DR" %}}
Resources (files and memory) are not shared between containers, and if you need to use Symfony's locks, you will need to use a [lock store](https://symfony.com/doc/current/components/lock.html#available-stores) that can be mounted to all of the containers for you application.
{{% /aside/warning %}}

Symfony provides a [Lock component](https://symfony.com/doc/current/components/lock.html) to prove exclusive access to shared resources. One example use of locks would be if we had a console command and we were running it with cron every minute. If the console command ran longer than a minute, then when cron ran the console command the next time, the two commands could try to do the same tasks (process the same data, send the same email, etc.). To prevent this, you can set a lock while the console command runs.

The Lock component offers several _stores_ - places where the component can store the lock information. Most of the available stores will not have any problems, but two stores that can are  the **FlockStore** and the **SemaphoreStore**.

FlockStore uses files and though the [Lock Component](https://symfony.com/doc/current/components/lock.html#flockstore) allows us to set where the lock file is stored, that setting is not exposed in the Symfony framework. This means the lock file is always stored in `/tmp` (or other default system temporary directory). This is not shared between containers, so different containers (ex. two cron containers) will not see the same directory or any existing lock files. This is the same problem for SemaphoreStore which uses shared memory for locking. Shared memory is not shared between containers.

{{% aside/note "Workarounds" %}}
There may be a way to redeclare the FlockStore service in your `config/services.yaml` file and define the location for the lock file. If so, then you can mount a persistent storage directory to all containers and use the FlockStore.
{{% /aside/note %}}
