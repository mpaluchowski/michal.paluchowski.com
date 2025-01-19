---
author: michal
date: 2014-04-09T17:01:35.000Z
title: Just enough logging
categories:
  - Technology
tags:
  - debugging
  - logging
  - programming
---

Debugging is a lot like police forensics. You're chasing the villain (bug) by analyzing [eye-witness accounts][mbnoteveryone] (users' reports), inspecting the crime scene (source code), and combing through often the most helpful resource: CCTV recordings (application logs), if only their quality allows.

<!--more-->

I got upset lately, looking for the needle in a stack of log *spam*:

{{<x user="mpaluchowski" id="445855130433634304">}}

where *everything* was logged and only mildly structured, making tracking application flow a nightmare. Clearly just dumping data into a text file doesn't make debugging easier.

The two most common faults with logging are:

1. logging to the **wrong level**, e.g. everything is `INFO`, forcing someone to dig through tens of thousands lines of text in Mega- to Gigabyte-sized files,
1. collecting the **wrong logs** per environment, e.g. logging `DEBUG` in production, which slows and breaks stressed systems due to "out of disk space" or other errors.

The bigger your organization, the more important it becomes to get logging right. You can't ignore the time wasted searching overblown logs, nor throw more hardware at the problem. [BBC's preparations for the London 2012 Olympics][velocitybbc], for instance, revealed incorrect logging as one of the top performance killers:

> * Monitoring Thresholds
> * **Verbose logging, everywhere**
> * Timeouts
> * No data
> * Volumetrics
> * Unfair load balancing
>
> <cite>Andrew Brockhurst, [The BBC's experience of the London 2012 Olympics][velocitybbc]</cite>

Most organizations' findings would likely be identical. Clearly, we can do better.

## Log the right things, right

With 5 standard levels of logging, it's not always easy to choose the right one. There are good rules of thumb though, and one of the best write-ups I've seen [comes from StackOverflow][stackloglevels]:

> * **error**: the system is in distress, customers are probably being affected (or will soon be) and the fix probably requires human intervention. The "2AM rule" applies here- if you're on call, do you want to be woken up at 2AM if this condition happens? If yes, then log it as "error".
>
> * **warn**: an unexpected technical or business event happened, customers may be affected, but probably no immediate human intervention is required. On call people won't be called immediately, but support personnel will want to review these issues asap to understand what the impact is. Basically any issue that needs to be tracked but may not require immediate intervention.
>
> * **info**: things we want to see at high volume in case we need to forensically analyze an issue. System lifecycle events (system start, stop) go here. "Session" lifecycle events (login, logout, etc.) go here. Significant boundary events should be considered as well (e.g. database calls, remote API calls). Typical business exceptions can go here (e.g. login failed due to bad credentials). Any other event you think you'll need to see in production at high volume goes here.
>
> * **debug**: just about everything that doesn't make the "info" cut... any message that is helpful in tracking the flow through the system and isolating issues, especially during the development and QA phases. We use "debug" level logs for entry/exit of most non-trivial methods and marking interesting events and decision points inside methods.
>
> * **trace**: we don't use this often, but this would be for extremely detailed and potentially high volume logs that you don't typically want enabled even during normal development. Examples include dumping a full object hierarchy, logging some state during every iteration of a large loop, etc.
>
> <cite>ecodan, [Logging levels - Logback - rule-of-thumb to assign log levels][stackloglevels]</cite>

While you're still coding:

* make sure you don't flatten stack traces - let them spill into logs in their usual, multi-line form, which then creates a visual pattern that's easy to scan for,
* avoid concatenating log messages - the (computationally expensive) code would execute even when a given level is not logged. Your logging library will often help you out, for instance [SLF4J][slf4j] will let you replace:

    ~~~
    log.debug("Loaded User: " + user);
    ~~~

    with

    ~~~
    log.debug("Loaded User: {}", user);
    ~~~

    and automatically compose the message *only when* the given level should be logged.

## Collect the right logs

Different application environments have their own logging needs. The general rules to follow are:

1. the more traffic, the less logging, but longer persistence,
1. the more active development, the more logging, but shorter persistence.

This translates into the following setup:

* **development/sandbox** `DEBUG` or even `TRACE` levels, with logs that are deleted within 24-48 hours.
* **testing** `DEBUG` most of the time, to efficiently [follow-up bug reports][mbqafromvenus], with logs deleted after 2-3 days.
* **staging** `INFO` to match production closely, with the occasional fallback on `DEBUG` if you need to trace problems, logs lasting for as long as the staging phase takes.
* **production** `INFO` or even `WARN` level if you have a very high-traffic system, with log storage for at least a week or longer.

The environment should also configure the correct [appender][log4jappenders]:

* use text files wherever possible. They're open, easy to parse and can be saved to disk without relying on intermediaries, like queues or database connections, to behave correctly (logs stored in databases won't help much if it's the database failing);
* log asynchronously. You really don't want the application to wait until each and every log message is written.

## Bonus: Decide explicitly, or be decided for

Make sure you have *explicit* logging configuration everywhere, even if some module doesn't use logging at all. Otherwise a newly pulled in dependency might come with its own setup, which your logging library or other dependencies will happily accept. Just last week we spent a few hours tracking down why our integration test suite suddenly started blasting out `DEBUG` level logging from all libraries, slowing running speed to a crawl. It was a new mocking library we had added to the [Maven dependency list][mavendependencies], which brought its own logging configuration, overwriting ours.

&nbsp;

Thanks to Tim Barnett for [motivating me][timstweet] to write this post.

[mbnoteveryone]: {{< ref"posts/not-everyone-knows-what-theyre-talking-about/index.md" >}}
[mbqafromvenus]: {{< ref"posts/developers-are-from-mars-qa-are-from-venus/index.md" >}}
[stackloglevels]: http://stackoverflow.com/a/8021604/121448
[velocitybbc]: https://web.archive.org/web/20130730225229/https://velocityconf.com/velocityeu2012/public/schedule/detail/25792
[timstweet]: https://twitter.com/tgbarnett/status/445905567618711553
[slf4j]: http://www.slf4j.org/
[log4jappenders]: http://logging.apache.org/log4j/2.x/manual/appenders.html
[mavendependencies]: http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html

*[CCTV]: Closed-Circuit Television

