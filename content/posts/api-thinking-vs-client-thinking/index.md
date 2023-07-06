---
author: michal
date: 2014-07-12T05:47:22.000Z
title: API thinking vs. client thinking
categories:
  - Technology
tags:
  - api
  - api design
  - programming
  - software architecture
image: /api-thinking-vs-client-thinking/apis-apis-everywhere.png
---

Have an API? No? So obscure. _Everybody_ has one these days as APIs were the foundation of online success in the last decade. But __building a good API is hard__. In fact, the mindset that's required is peculiar enough to consider separating people who will build it from those who will use it.

<!--more-->

APIs were all the rage that, along with [AJAX][wpajax], [kicked off Web 2.0][wpweb20]. By allowing others to tap into the features and data of your application, you could spark a whole community of clients and mash-ups, making you the __platform__. Twitter is a well known child of this era, where an API was built first, then Twitter's own clients as well as all the independent ones on top of it.

{{< figure src="/api-thinking-vs-client-thinking/apis-apis-everywhere.png" alt="APIs, APIs everywhere" >}}

This obviously takes away control of the application's future from its creators, putting it into the hands of a broader community. Example being again Twitter, where features such as [retweets were only added to the platform][twitterretweet] once they became widely used in independent clients. At some point Twitter decided to [reclaim control of its brand and user experience][twittercontrol], which started to diverge between applications. Certain requirements were imposed on how tweets may be displayed and what functions should be available. Break those and you may be kicked off the API completely.

For System Architects, APIs are the panacea in a multi-device world. With the variety of client applications being demanded - web, native, embedded, large-screen, tiny-screen etc. - we want to keep complexity low by reusing as much code as possible. A properly written API can be shared between all clients and even allow for gracefully dropping support for a legacy generation, like a browser that's becoming obsolete.

Trello makes excellent use of this graceful degradation pattern:

> [T]he website is just a face that chats with the Trello API and that the iOS and Android apps are also just faces and you can make your own face.
> (...)
> [T]here's a special face out there for people using Internet Explorer 9.

There's the API shared by all official and unofficial clients, each one called a "face", and there's a special, older version of the web face that's left to support the remaining users of Internet Explorer 9. Brilliant.

* _Yes! I want to build an API. How do I go about it?_
* _With foresight and planning._

APIs are a special case of [Separation of Concerns][wpseparationconcerns] and here's where I'm starting to think that APIs and clients should be built by different people:

* clients are focused on their immediate needs; I'm building feature X and need data A, B and C formatted this way.
* APIs are catering for many clients and their different, often incompatible needs.

If the same person writes the client and the API, or even if they're separate but on one, tightly knit team, they are much more likely to reconcile the conflict by __leaning towards the immediate need of the client__, away from the broader needs of the ecosystem. Every subsequent client that comes in with their needs will receive their own, special endpoints. Soon you'll have an explosion of similar, oddly named methods for very specific use cases, little reusability, where a simple change may require modifications to hundreds of lines of code. In other words, you'll have built a [monolith][wpmonolithsystem] where "API" will merely be a different name for the application's model layer, and since that model will be separate from the rest of the application, complexity becomes even worse.

Then, once it's in production, you're dead in the water, because:

> Public APIs are forever.
>
> <cite>Joshua Bloch, [How to Design a Good API][ythowtodesignapi]</cite>

Anyone can use a public API and you'll have to maintain backwards compatibility for a long, long time.

However, if you task different people with building APIs and clients, you'll get a lot of conversations, often conflicts, which are essential for getting the best result for the broadest amount of use cases.

Make sure the API [team][mpteamneverwas] consists of people who have as wide a perspective as possible. Keep thinking well beyond the immediate requests they receive, weighing those against all similar requests in the past and thinking forward into the future. What else could be required from this method later? What else might someone want to extract from this particular data set? Will it need filtering, sorting, paging?

Building a good API requires following guidelines, which are not the ones usually proposed for client design:

* __violate YAGNI__ - think of what might be useful in the future, but leave out things that are easy to add, because removing anything is much harder;
* __write a broader than usual set of features__ for the method, weighing the possible performance penalties against power;
* __displease everyone equally__ - clients will often times need to curb their requirements, to allow for broader reusability;
* __document extensively__ - your documentation will become a guide to understand the [contract][wpdesignbycontract] of each method - what it expects and returns. Without it, you'll be swamped by questions and complaints.

[Joshua Bloch][twitterjoshuabloch], creator of, among others, the [Java Collections API][javacollections], shares a number of excellent recommendations for building APIs in a Tech Talk he gave years ago at Google. It's well worth the hour to see it:

{{< youtube aAb7hSCtvGw >}}

If the API is done right, it's an investment that pays back many times the effort put into it. The multitude of clients that can use it, the flexibility to rapidly build features that weren't previously thought of. For any regular [software company][mpgeekheaven] it's possibly _the_ most complex task it will handle and you should put your best, brightest people on it. And make sure they __spark conflicts with all the developers building clients__, because that means they're __having real conversations__ about how to build the best solution for everyone.

[javacollections]: https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Collections.html
[mpgeekheaven]: {{< ref"posts/searching-for-geek-heaven/index.md" >}}
[mpteamneverwas]: {{< ref"posts/a-team-that-never-was/index.md" >}}
[twittercontrol]: https://blog.twitter.com/2012/delivering-consistent-twitter-experience
[twitterjoshuabloch]: https://twitter.com/joshbloch
[twitterretweet]: https://blog.twitter.com/2009/project-retweet-phase-one
[wpajax]: https://en.wikipedia.org/wiki/AJAX
[wpdesignbycontract]: https://en.wikipedia.org/wiki/Design_by_contract
[wpmonolithsystem]: https://en.wikipedia.org/wiki/Monolithic_system
[wpseparationconcerns]: https://en.wikipedia.org/wiki/Separation_of_concerns
[wpweb20]: https://en.wikipedia.org/wiki/Web_2.0
[ythowtodesignapi]: https://www.youtube.com/watch?v=aAb7hSCtvGw

*[AJAX]: Asynchronous JavaScript and XML
*[API]: Application Programming Interface
*[YAGNI]: You Aren't Gonna Need It
