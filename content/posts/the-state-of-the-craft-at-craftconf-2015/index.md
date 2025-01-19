---
author: michal
date: 2015-05-04T12:47:16.000Z
title: The state of the craft at CraftConf 2015
categories:
  - Technology
tags:
  - agile
  - conferences
  - ddd
  - developers
  - microservices
  - software architecture
  - tdd
image: /the-state-of-the-craft-at-craftconf-2015/craftconf-stage.jpg
---

[CraftConf][craftconf] in sunny Budapest aims to be _the_ conference in Central Europe where developers share what's state of the art in building software. Joining the [2015 edition][craftconf15] was an easy choice, after the first edition a year earlier received rave reviews from attendees. Three days, 16 talks and a workshop later I emerged with 6,062 words worth of notes, distilled into a few broad trends that seem to represent the edge of the industry right now.

<!--more-->

{{< figure src="/the-state-of-the-craft-at-craftconf-2015/craftconf-stage.jpg" alt="CraftConf 2015 Stage" >}}

## Microservices entering the mainstream

At [SoCraTest 2014][mpsocrates], less than a year ago, a lot of people were still asking basic questions about microservices---what they are, how to build them, how are they different from any other SOA approach. Not anymore. CractConf saw a lot of companies sharing battlefield experiences with these architectures---what works, what doesn't, whether to consider the approach at all and how much it will cost.

* Microservices are a __means to scale the organization__---allow many people to work on the same system, reduce dependencies, and thus enable rapid development without having to coordinate huge releases or continuously breaking one another's code. They're routinely described as "_easier to reason about_" and built to "_minimize the amount of program one needs to think about_" by the very virtue of being small and focused on doing one task well.
* Microservice ecosystems at established companies __routinely consist of 50+ and even 100+ services__, usually built in a whole range of technologies. If a single service is small enough, say "_2,000 lines of code_", or "_takes 6 weeks to build from scratch by a newcomer_" (both actual definitions), then building one in a wholly new technology is easily sold as an inexpensive POC.
* All companies successfully implementing microservices __started out with monolith applications__ and afterwards split out the services one at a time. This approach was broadly recommended, even for greenfield projects, because monoliths are easier to refactor while the organization works to research and understand its product better.
* The __supporting tooling is already there__ and mostly quite mature. Early adopters, like Netflix, had to build [their own tools][ghnetflix], which they later shared and improved with the open source community. Meanwhile, commercial tools popped up, offered by specialized vendors, with support contracts and other conveniences that pointy-haired bosses like so much.

But most importantly:

{{<x user="martinfowler" id="504997343440408576">}}

To benefit from microservices without getting killed, you need to:

* __automate everything__---builds, testing, deployment, provisioning of virtual machines and container instances. It's the only way to manage hundreds of running service instances.
* __monitor everything__---each service must have its own health checks, showing [whatever information is most relevant][mpenoughlogging]. Typically numbers of transactions, timings, latency, hardware utilization, but often also business KPIs. You'll also need a means to track requests flowing through the system, ie. by IDs being passed in service calls. This information must be available to development teams _in real time_.
* __build for failure__---crashes, disconnections, load spikes, bugs, all of which will occur more often than with monoliths. Make sure failures are reported, tracked, and the system is self-healing, via techniques like circuit-breakers, bulkheads or resource pools. Work with business representatives to determine for each use case whether __consistency or availability__ are more important, because you cannot always offer both.

Not everyone was pleased with such a high degree of saturation with microservice themes at CraftConf, but that only proves that this architecture is well past the stage of early adoption and entering into the mainstream.

## Everybody has an API

The consequence of adopting microservices is that [APIs become the norm][mpapithinking], starting out internally, and often "_sooner than you think_" being made available to the outside world, either to support different devices or integrate with 3rd parties. Their costs also became more evident:

* __An API is for life__. Once it becomes public, it's very difficult to change, so early design mistakes become much more difficult to fix.
    - Use techniques that allow you to add new fields and methods to an API without disturbing clients.
    - __Semantic versioninig__, coupled with decent __deprecation procedures__, help move the process of making backwards incompatible changes.
    - You'll still have customers who "_missed the memo_", whom you may have to offer commercial legacy support for an additional fee.
* Create __good documentation__, generated and published automatically.
* Assign people who will be __monitoring community sites__, like [StackOverflow][stackoverflow], for questions regarding your API.

[@ade_oshineye][twade_oshineye] (of [Apprenticeship Patterns][apprenticeshippatterns] fame) was spot-on summarizing the process of deciding whether to create an API by showing an analogy to puppies---everybody wants one, but not everyone is ready.

{{<x user="rhwy" id="591591859388055552">}}

## Moving towards Domain-Driven Design

Frameworks these days tend to dictate the design of applications by suggesting organizing code by layers into packages like `models`, `controllers` or `views`. The consequence is that:

* modifications to a single business scenario require often changing code in many packages,
* it's all too easy for components to reach into layers outside their hierarchy, too deep down for their own good, while they should only be calling public APIs,
* there's very little information presented by the package structure on what the application does.

Several speakers postulated that this should stop and instead __code should be packaged by business units__, with relevant models and controllers sharing the same packages. Encapsulation could further be improved by changing method and component access to `package` instead of the common `public`. It's an old theme that finally seems to be getting traction, with past support from developer celebrities like Uncle Bob:

> What do you see? If I were to show you this and I did not tell you that it was a Rails app, would you be able to recognize that it is, in fact, a Rails app? Yeah, looks like a Rails app.
>
> Why should the top-level directory structure communicate that information to you? What does this application do? There's no clue here!
>
> <cite>Robert Martin, [Architecture, the Lost Years][ytarchitecturelostyears]</cite>

Well worth watching.

{{< youtube WpkDN78P884 >}}

## TDD and Agile now taken for granted

Two themes notably _absent_ from the vast majority of CraftConf talks were TDD and [Agile][mpagile]. They implicitly became _accepted as defaults_---the baseline, cost of entry and survival in the game of software development.

{{<x user="sebcachia" id="591643649399717888">}}

- Microservices will only reach their full potential when their owning teams can work and [make decisions][mptellmewhattodo] __independently from central authority__, including frequent deployments to production.
- Frequent, decentralized deployments require comprehensive, __automated test coverage__.
- TDD drives the design of code, meaning every line exists only to __make a failing test pass__. "_Every new line of immediately becomes legacy code_" so code as little as possible.
- Google continues to build new tools to satisfy business problems they are facing---tools that often get adopted company-wide, but never enforced. They call it an __evolutionary approach__, where the best ideas will be adopted simply on merit, while the others will die in obscurity.
- [@cagan][twcagan] argued how the whole top-down oriented product design cycle is broken by being grossly inaccurate and too heavy, and companies need to adopt __bottom-up idea spawning__ and rapid testing instead.
- __[Transparency][mptransparent] is key__, so that everybody can [make better decisions][mpdecisions] based on as much information as possible.

Oh, and "doocracy" is a word now:

{{< x user="srdjanpoznic" id="591194137178808320">}}

## A vibrant community

It's been a blast to mingle with some of the 1,300 energetic attendees, meeting friends, old and new. [@MrowcaKasia][twmrowcakasia] and [@kubawalinski][twkubawalinski] turned from Twitter handles into real, live, and engaging persons. The slightly grumpy but ever competent [@maniserowicz][twmaniserowicz] is always a pleasure to meet, and then there's the whole SoCraTes gang of [@leiderleider][twleiderleider], [@c089][twc089], [@Singsalad][twSingsalad], [@egga_de][twegga_de] and [@rubstrauber][twrubstrauber], whose passion for community and craftsmanship continues to inspire.

The 2015 CraftConf was only the _second_ edition, but already organized with such painstaking detail, that I can easily call it the best conference I've ever been to. The team's already fishing out and working to improve what didn't quite work this year, so next year's event is bound to be even more polished.

[amazonoosoftwareengineering]: http://www.amazon.com/Object-Oriented-Software-Engineering-Approach/dp/0201544350/
[apprenticeshippatterns]: http://chimera.labs.oreilly.com/books/1234000001813/
[craftconf15]: http://craft-conf.com/2015
[craftconf]: http://craft-conf.com/
[ghnetflix]: https://github.com/Netflix/
[mpagile]: {{< ref"posts/what-does-it-mean-to-be-agile/index.md" >}}
[mpapithinking]: {{< ref"posts/api-thinking-vs-client-thinking/index.md" >}}
[mpdecisions]: {{< ref"posts/deciding-to-decide-when-the-time-is-right/index.md" >}}
[mpenoughlogging]: {{< ref"posts/just-enough-logging/index.md" >}}
[mpsocrates]: {{< ref"posts/the-unconference-experience-of-socrates-2014/index.md" >}}
[mptellmewhattodo]: {{< ref"posts/dont-tell-me-what-to-do/index.md" >}}
[mptransparent]: {{< ref"posts/agile-means-transparent/index.md" >}}
[stackoverflow]: http://stackoverflow.com/
[twade_oshineye]: https://twitter.com/ade_oshineye
[twc089]: https://twitter.com/c089
[twcagan]: https://twitter.com/cagan
[twegga_de]: https://twitter.com/egga_de
[twkubawalinski]: https://twitter.com/kubawalinski
[twleiderleider]: https://twitter.com/leiderleider
[twmaniserowicz]: https://twitter.com/maniserowicz
[twmrowcakasia]: https://twitter.com/MrowcaKasia
[twrubstrauber]: https://twitter.com/rubstrauber
[twSingsalad]: https://twitter.com/Singsalad
[ytarchitecturelostoobook]: https://youtu.be/WpkDN78P884?t=12m20s
[ytarchitecturelostyears]: https://www.youtube.com/watch?v=WpkDN78P884

*[API]: Application Programming Interface
*[DDD]: Domain-Driven Design
*[KPIs]: Key Performance Indicators
*[POC]: Proof of Concept
*[SOA]: Service-Oriented Architecture
*[TDD]: Test-Driven Development

