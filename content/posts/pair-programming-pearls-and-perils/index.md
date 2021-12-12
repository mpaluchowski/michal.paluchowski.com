---
author: michal
title: "Pair programming pearls &\u00A0perils"
date: 2019-02-17T15:57:37Z
lastmod: 2019-02-20T20:25:23Z
categories:
  - People
tags:
  - communication
  - pair programming
  - programming
  - learning
---

If pair programming was easy, and universally better than working individually, we'd all be doing it *all* the time. But we don't. Most people I met need to make a conscious effort to pair and stay paired. All the while saying they "like it" and "need to do more of it". How come?

<!--more-->

The way practicioners talk about pair programming reminds me of how people talk about exercising: "yeah, I should do this more often". They *know* there are benefits, they may be able to recite some of them, but it certainly doesn't come naturally.

## The good parts

Perhaps the best part of working in a pair is that **you can't hide**:

> Pairs naturally arouse engagement, even intensity. In a larger group, an individual may lie low, phone it in. But **nobody can hide in a pair**. "The decisive characteristic of the dyad is that each of the two must actually accomplish something," wrote Georg Simmel, "and that in the case of failure only the other remains—not a supra-individual force, as prevails in a group even of three." [emphasis mine]
>
> <cite>Joshua Wolf Shenk, [Powers of Two: How Relationships Drive Creativity][amazon-powers-of-two]</cite>

The presence of another sets a pace, much like in a group of cyclists. You can't go slower than your partner. You can't procrastinate, idly browse Facebook or otherwise distract yourself. There's good focus and things move forward quicker.

It also helps, that two heads **bring together diverse experience**. Your partner may have seen a particular problem already and know the solution. You may have worked with a required technology before.

This brings up the point of **guided learning by doing**, but also the broader topic of **exchanging information**. You talk continuously and both of you are on the same page with regards to customer requirements, system architecture, standards, patterns and plans moving forward.

Finally, [as Linus likes to say][wikipedia-linus-law], "given enough eyeballs, all bugs are shallow". Pair programming is **a form of code review**---an informal, lightweight, continuous one. Your partner will often see bugs or consequences that you don't, or will claim that a particular section of your code is difficult to comprehend---a somewhat annoying remark, but one that eventually leads to better code.

## The parts that irk

The blessing is also the primary curse, because **in a pair, you can't hide**. Sometimes you need your time alone. Some personal space, perhaps to idly do some busywork, relieve your brain, or to process a particular problem in solitude and silence. That applies especially if you're an introvert and continuous communication drains your energy.

Your partner may have a **different pace of working**. Perhaps they'll take more, or fewer, breaks, setting a rhythm of work that will not be to your comfort. They may like to crunch quickly through sections of code that you prefer to contemplate longer. There's a related issue of **differences in temper**, where I personally like to think through solutions and understand available options before I touch the keyboard, I worked with people who are more "trigger happy" and prefer to try things out and see where it brings them.

Both of you **need to synchronize your schedules**. If you're starting work at 8:30 and your partner at 10:00, then what do you do until they arrive? What will *they* do when you leave early?

Finally, there's the same problem as when walking with someone who knows their way while you don't. Many people will be unable to learn to navigate on their own from following someone's lead. If you're new to the system and pairing with someone who's already familiar with it, there's a good chance you'll **need more time to find your way around** than if you were working on your own.

---

"Sometimes" and "some of the time", would be my answer to whether programmers should work in pairs. Certainly not mandatory, neither by policy nor peer pressure. And make it conscious: see which parts work for you, and which ones annoy you.

Notice with whom you're doing your best work---it *won't* be everybody. One can always make the argument that working with someone you consider "difficult" puts you out of your comfort zone and teaches you new things, but I find you can learn just as much, without the added tension, by working with people you're in tune with.

By all means, try it out if you haven't yet. [Don't ask your boss for permission][mb-dont-need-permission], just do it. You'll be happy you did, because pair programming *does* produce better code in the end, makes you a better craftsperson, and it takes doing to believe it.


[amazon-powers-of-two]: https://www.amazon.com/Powers-Two-Relationships-Drive-Creativity-ebook/dp/B00E9FYT0O/
[mb-dont-need-permission]: {{< ref"posts/you-dont-need-permission-you-need-to-act/index.md" >}} "You don't need permission, you need to act"
[wikipedia-linus-law]: https://en.wikipedia.org/wiki/Linus%27s_Law
