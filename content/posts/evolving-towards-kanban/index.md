---
author: michal
date: 2012-08-05T14:44:00.000Z
title: Evolving towards Kanban
categories:
  - People
tags:
  - agile
  - kanban
  - management
  - software development
---

[Kanban](https://www.amazon.com/gp/product/B0057H2M70/) is the last stage of software delivery evolution. Right now, at least, as evolution never really stops. The path is usually the same, starting with a disorderly "let's just code", moving gradually towards a waterfall system, then painfully stumbling towards a formal [Agile]({{< ref"posts/what-does-it-mean-to-be-agile/index.md" >}}) framework, say Scrum, and once a Scrum team truly matures, they realize they're really doing Kanban with a bit of unnecessary paperwork.

<!--more-->

The jump from chaos to waterfall is rather obvious. So is usually the next step to go Scrum (or any other framework you like). We love all the Agile principles, user stories help [focus on business outcomes]({{< ref"posts/agile-is-the-business-approach-to-software-en/index.md" >}}), expecting changes makes Product Owners happy and programmers less stressed. Estimates made in story points help the management see when things will be done and ready, and make decisions based on that.

In Scrum for a new project you define a whole bag of user stories, estimate them, order by business value and call it a "Release". That Release gets split into Iterations, each of 1 or 2 weeks in length. When you're done with all the Iterations, you're done with the Release and ready to unleash your product onto the general audience.

We're often working, however, with software that's _relatively inexpensive_ to release. Like web applications, where you upload code to a server and all your users [instantly get the latest version](http://www.codinghorror.com/blog/2011/05/the-infinite-version.html). There's obviously the question of stability, quality and testing, but Scrum says strictly that the [definition of "done"](https://web.archive.org/web/20120830195739/http://www.scrumalliance.org/articles/106-definition-of-done-a-reference) is "designed, developed and tested." So, by that definition, you should have a release-quality product after every Iteration.

Add the two things together and you soon have a Product Owner asking you whether it's possible to do an interim-release, before the whole scope is done. Sure, it is. It takes a while to work out discipline and processes right, so that the quality really is good, but once you're there, you can release right after the Iteration summary meeting. And the Product Owner has a point: why do we wait another X weeks for the whole Release to be ready, when we can have something live now? Competition never sleeps and some of these things could make users seriously happy.

Here's where Kanban comes in: once you're ready to release after every Iteration, then the answer to "when will this be ready?" becomes "right after the Iteration you put it in." Long-term estimations become irrelevant, as they always were anyway. Just because you have a user story planned for 3 Iterations from now doesn't mean it'll really be done there and then. Things shift all the time.

Why bother with the "Release" planning then anyway? Switch to a higher-level roadmap of changes, so that the team continues to follow a vision for the product, but plan actual work only for the next Iteration. Then release it immediately. Now you're doing Kanban.
