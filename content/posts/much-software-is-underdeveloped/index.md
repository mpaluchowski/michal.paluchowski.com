---
author: michal
title: "Much software is underdeveloped"
date: 2019-07-15T18:25:00Z
categories:
  - Technology
tags:
  - software
  - human computers
  - software design
  - product owner
---

Software can do so much more than it does today, with mature programming languages, standard instructions and not a *hint* of deep learning. Just plain, old, "shallow" business logic, thoughtfully implemented, relying on existing data. And it's up to us, developers, to champion these improvements.

<!--more-->

Looking back at history, I see computers---software, really, because hardware stopped being important back in the 80s---undergo three stages of evolution:

1. replace humans in storing & retrieving data and performing computations,
2. provide obvious conclusions, based on data,
3. draw and provide non-obvious conclusions---what we'd call Artificial Intelligence.

Remember when the term "computer" meant "[person doing computations](https://en.wikipedia.org/wiki/Human_computer)"? Neither do I, because the last time this meaning applied was somewhere in the 1950s. "Computer Rooms" were places filled with people performing mathematical operations by hand, later with the help of mechanical calculators. It was considered *menial* work.

{{< figure src="computer-room.jpg" title="Human computers in the NACA High Speed Flight Station \"Computer Room\"" >}}

The first computers were built to replace all these people with machines that would reliably, repeatedly provide the same computing services, and do so faster. Digital data storage also is way more efficient, both in terms of space used and access speed.

{{<tweet user="BrianRoemmele" id="1147556213523021824">}}

You can argue that a lot of software today is still doing a lousy job at step one, working slowly, unreliably or offering a really bad user experience to those trying to retrieve data. Still, the features are broadly there, implemented, working.

Step two is where the trouble begins. I won't even go into discussing step three, because so little software is delivering on even this simple premise of offering *obvious* conclusions from the data it has access to. Cases in point:

* A company keeps customer accounts with credentials and logins. They organize an event for customers, with a mobile app that offers agenda and other features, and *requires a separate login*. Why? Why can't I reuse my credentials?
* A multi-level marketing system calculates bonus points for distributors, with a sophisticated setup of qualifications for various stages, events and prizes. All the rules are programmed into the system, but all the distributors ever see is a log of how many points they received, and are often puzzled why they received too little or received different types of points than expected, and hence didn't qualify for some level of performance.
* Same multi-level company, same system of bonus points and qualifications. People need to make a certain amount of points of a certain type by a certain date to qualify for an exclusive event. Why doesn't the system suggest the easiest path to reach the required level for those that are close-but-not-quite-there-yet? Ie. "if you make another order of $100 by next Monday, you'll make it to the Super-Duper Event!" These distributors are wasting tons of the company's time calling to ask questions, complain about their lack of qualifications etc.
* I make a reservation with the same hairdresser for the same haircut service at the same salon every month. Why doesn't the online reservation form remember my past choices? And if it doesn't, why do I have to register for an account with it at all?
* A search for doctors' appointment slots turns up no results, maybe because the selected doctor is on vacation, or fully booked, or the facility doesn't have the selected specialization. Why not *immediately* offer slots from other doctors at the same facility, or from the next nearest facility? Why offer just a bland suggestion to "change my criteria"?
* Phone company's payment system sends out bills to customers at the beginning of the month and then reminders three days before they're due. I always pay on the last day and *never* missed a deadline, but I keep getting spammed with these reminders. Why doesn't the system send them only to those, who paid late in the past?

I could go on, and on, and on. All these systems have all the data they need to help their users get more done in less time. No fancy neural networks or other forms of machine learning is needed. Just someone---usually the developer(s) behind the system, because nobody else knows better what their creations are capable of---who would raise these basic questions and implement these, rather basic, features. By making life easier for users, companies would either save money on customer support, or make money on increased sales, often both.

Truth be told, it's usually not the developers' fault. Not entirely, at least. Behind every development team, there's [some sort of Product Owner]({{< ref"posts/developer-turned-product-owner/index.md" >}}) who makes the calls on what gets done, how and when. Some developers will just do what they're told, offering little input of their own. Others will offer ideas, which may even be picked up by the PO and placed in the dreaded Backlog for "when we have time", among all the "nice to haves".

Whoever is the PO, formally or not, *is* responsible for existing and missing features of software. Too many of them see developers as mere coders rather than partners in building great systems. The result is a very shallow understanding of the software. Without developer input, POs will not be able to see beyond the user interface, and miss out on all the action in the business logic and data[^1].

I say, quoting Jack Welch, "**every brain in the game**". [Developers: speak up!]({{< ref"posts/you-dont-need-permission-you-need-to-act/index.md" >}}) Keep suggesting features that you *know* are perfectly possible and will be useful to the users. POs: listen! Use all the creativity and knowledge that's at your disposal, from the people you rely on so much for delivering your software. The closer you cooperate, the better the output and everybody profits. The users, the companies, and as a consequence, so will you---the creators.

[^1]: An often cited example is the Google search engine. The UI is a simple input field and two buttons, but the machinery behind delivering results is staggering.
