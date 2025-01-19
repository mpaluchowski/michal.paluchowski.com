---
author: michal
date: 2017-11-19T10:32:07.000Z
title: The unexciting GOTO Berlin 2017
categories:
  - Technology
tags:
  - conferences
  - distributed systems
image: /the-unexciting-goto-berlin-2017/goto-berlin-badge.jpg
---

Unexciting. That's the one word I would use to describe this year's edition of GOTO Berlin. And I'm suspecting it's a sign of advancing maturity. Just not sure whether it's the industry's, or my own.

<!--more-->

This year was my second time at GOTO Berlin, after I attended in December 2015. I came back because the first time I was really satisfied with the content. Lots of fresh, practical, accessible ideas, well-presented (unlike, say, QCon London 2016). Plus, the keynotes were tremendous fun.

{{< figure src="/the-unexciting-goto-berlin-2017/goto-berlin-convention-room.jpg" alt="GOTO Berlin 2017 Convention Hall" >}}

Keynotes are always tricky for the organizers, because you want them to be somewhat abstracted from the low-level themes of sessions, but not too far away, so that the audience can still relate. Plus, they need really engaging speakers. GOTO Berlin pulled this off perfectly. The 2015 edition had space-themed keynotes about the [Apollo][gotober-2015-apollo] and [Space Shuttle][gotober-2015-space-shuttle] programs, _mighty_ interesting. This year featured a variety: Mars exploration, data science, autonomous systems and data security. Really, really good content.

This year's sessions, however, surprised me with __how little new stuff there was__. Nothing really bleeding-edge. Same topics I keep hearing about for the last two-three years: microservices, DDD, containers (and agile, of course, but I staid away from "soft" themes). Even serverless, that I thought would be the buzzword of this year, seems ubiquitous and long-tamed.

Not that any of these are boring---to the contrary, they've now reached mainstream adoption and presenters have a lot of real-world experience to share (as opposed to theories and hopeful evangelization). It's just... what happened to the breakneck speed of "innovation" in this industry?

Granted, there was a lot of talk about AI, which seems to be _the_ bleeding-edge stuff these days. But I staid away from those sessions too. We still need to prove we can deliver well-structured "dumb" software before we can think of constructing anything "smart".

So, what did I see?

[Rust][rust-lang] and [WebAssembly][webassembly] were the only real, new things for me. Emerging languages and platforms, the first one may help us make proper use of concurrency and multi-core processors:

{{<x user="mpaluchowski" id="931147623033180160">}}

and the [latest Firefox release already benefits from it][firefox-rust]. The second might help us build advanced applications for the web.

The rest of the sessions I attended were all about: how do you deal with the real world? Not some fresh, hot framework or cloud service, but the day-to-day grind of working in a business environment, modeling a highly irregular world in code, maintaining multi-year- and decade-old legacy code. All of this in a time of cheap networking, favoring distributed systems, where failure is the rule, not the exception.

It may be that the industry has changed. Reflected on itself and found that __technological novelties are fun, but often leave real-world problems unsolved__. Came back to Earth and asked itself "what are we _really_ trying to achieve?" Or... it may be that I'm the one who changed. I still enjoy new toys, but there's work to do down here. An approach that informed my choice of sessions to attend, and so influenced my impressions.

*[DDD]: Domain-Driven Design

[firefox-rust]: https://blog.rust-lang.org/2017/11/14/Fearless-Concurrency-In-Firefox-Quantum.html
[gotober-2015-apollo]: https://www.youtube.com/watch?v=l3XwpSKqNZw
[gotober-2015-space-shuttle]: https://www.youtube.com/watch?v=AyrRoKN_kvg
[rust-lang]: https://www.rust-lang.org/
[webassembly]: http://webassembly.org/

