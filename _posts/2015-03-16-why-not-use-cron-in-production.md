---
layout: post
title:  "Why not use Cron in production"
date:   2015-03-16 13:05:00
categories: ruby cron
comments: true
---
Cron is the simplest way to schedule tasks in \*nix systems. Since it's a
mature tool, most of developers and companies have been using it in production
without analyzing if it's suitable for distributed systems.

### Why not use Cron?

Most of the web applications today try to benefit from distributed systems
properties and I dare to say the most important ones on web are scalability and
reliability. So, does Cron provide the reliability expected on web applications?
See why not:

*    **Cannot run distributed**

     Cron jobs are setup in one single computer so, if that computer crashes, the
application will miss important scheduled tasks until that server is back up or
another server with same crontab configuration is spinned up.

*    **Does not feature backfill**

     When a server hosting Cron is recovered from a crash, it does not
automatically execute the jobs missed while the server was down. If a scheduled task is missed
while the server is down, a person will need to execute the missed tasks manually.

     [Anacron](http://en.wikipedia.org/wiki/Anacron) is a tool that works in conjunction
with Cron and does not assume the system is running continuously. However, Anacron
does not run distributed.

*    **Does not retry failures automatically**

     Cron jobs are not automatically retried on failure. I've seen cron jobs that
access a database or an external service. When these fail, a reliable solution
would retry the same job after some delay - usually an [exponential backoff formula](http://en.wikipedia.org/wiki/Exponential_backoff) is a good choice to define the delay intervals between retries.

### Solutions

*    For Ruby, there is this [Side<strong>t</strong>iq gem](https://github.com/tobiassvn/sidetiq)
that runs on top of [Side<strong>k</strong>iq](https://github.com/mperham/sidekiq).
Here is an ideal setup to avoid the issues mentioned above:

  * Use Side<strong>k</strong>iq's [automatically retries](https://github.com/mperham/sidekiq/wiki/Advanced-Options#workers).

  * Use Side<strong>t</strong>iq's [backfill feature](https://github.com/tobiassvn/sidetiq/wiki/Backfills).

  * Have at least **two** servers running Side<strong>k</strong>iq workers.

  * Notify the application maintainers when the retries are exhausted. Look how
to use Side<strong>k</strong>iq's `sidekiq_retries_exhausted` feature at
[Sidekiq Error Handling configuration](https://github.com/mperham/sidekiq/wiki/Error-Handling#configuration).

*    There are some SaaS solutions out there. I found this [Crondash](https://crondash.com)
tool which has retry support. It also reports errors which is useful to monitor the application
health. If you search for *online cron job* on web, you should find other similar SaaS available.

There should be many other solutions to the same problem. Just remember to choose
one that supports retry, backfill and running in multiple servers.

Please comment what solutions you are using for cron jobs that fulfills
distributed system properties.

Happy coding!
