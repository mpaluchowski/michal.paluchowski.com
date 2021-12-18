---
author: michal
date: 2014-03-30T05:43:59.000Z
title: The crazy state of structured data markup
categories:
  - Technology
tags:
  - html
  - search engines
  - seo
  - social networks
  - structured data
---

You want to look good for Google. You want it to *understand* your website, so that it comes up in results often and stands out. You also want Facebook and Twitter to display your links prominently in their crowded timelines. Because you want all that, you'll likely turn to structured data markup, like I did with the new design of Michał's Bites. And then you'll shake your head in disbelief.

<!--more-->

Designing a fresh look for Michał's Bites nudged me to look into [Schema.org](http://schema.org/) - a co-product of Bing, Google, Yahoo! and Yandex, meant to help them make sense of the contents of a page. In the words of its publishers:

> On-page markup enables search engines to understand the information on web pages and provide richer search results in order to make it easier for users to find relevant information on the web.

And while choosing the [right entity](http://schema.org/docs/full.html) for your particular page element isn't always straightforward, it's easy to mark it up:

```html
<article itemscope itemtype="http://schema.org/BlogPosting">
  <h1 itemprop="name headline">The crazy state of structured data markup</h1>
  <section itemprop="articleBody">
    <p>...</p>
  </section>
  <p>Written by <span itemprop="author" itemscope itemtype="http://schema.org/Person"><span itemprop="name">Michał Paluchowski</span></span></p>
</article>
```

It's *consistent*. Most entities will have a way to markup `name`, `url` or `description`. Some have unique attributes, like a `BlogPosting` has an `articleBody` above. It's readable for both computers and humans.

But wait, there's more.

Since I'm using [WordPress]({{< ref"posts/living-with-wordpress/index.md" >}}), some of its widgets output markup of the [microformats](http://microformats.org/) brand. These serve a similar purpose as Schema.org, but with a much smaller dictionary of entities. Little did I know that Google scanned it too and started complaining via Webmaster Tools that my implementation was incorrect:

{{< figure src="/the-crazy-state-of-structured-data-markup/webmaster-tools-microformats-error.png" alt="Google Webmaster Tools Microformats Error" title="Google Webmaster Tools Microformats Error" >}}

I complied, included the missing markup, and the code became:

```html
<article class="h-entry" itemscope="...">
  <h1 class="p-name" itemprop="...">
```

There's *still* more. Facebook developed its [OpenGraph](https://developers.facebook.com/docs/opengraph/), and Twitter has their [Cards](https://dev.twitter.com/cards), all of which - with some extra markup - allow me to control and improve the way content will appear in the services' respective timelines. Otherwise Facebook may display a random snippet of text with a link, starting with something as "meaningful" as "Comments closed".

These meant adding some more markup to my code:

```html
<meta property="og:type" content="article">
<meta property="og:title" content="The crazy state of structured data markup">
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="The crazy state of structured data markup">
```

Now my content was nicely highlighted on Twitter (note: doesn't always show up):

{{<tweet user="mpaluchowski" id="447115370965774336">}}

As a consequence, I have **the same data three-four times in the page**:

1. reader-visible HTML, with the overhead of Schema.org and Microformats markup,
2. META markup for Facebook,
3. META markup for Twitter.

It's like adding extra CSS for some older versions of Internet Explorer with `<!--[if lt IE 9]>`. Overhead and waste.

There is, perhaps, an end to this in sight. The W3C, just a few months ago, published a [draft specification of Microdata](https://www.w3.org/TR/microdata/), which essentially is Schema.org as part of HTML5.

I like the Schema.org specification best, because it's rich, consistent and impossible to confuse with other markup. Using the `class` attribute for structured data is logically sound (`<article class="blog-post">` speaks well of the type of the article), but if you place it in any slightly complex web layout, maintained by many people, it's easy to mix and confuse with styling values. That means it's likely to be accidentally removed or changed.

At the same time, Schema.org markup gets added right onto content markup, without the overhead of duplication in the `<head>` or elsewhere - again a maintenance nightmare waiting to happen.

Both Facebook and Twitter are certainly powerful enough to enforce their own solutions (think Facebook's latest [announcement of Hack](https://code.fb.com/developer-tools/hack-a-new-programming-language-for-hhvm/)), but if Google and Bing were able to come together and agree on one standard, I'm sure other big names can join the party too. The fewer standards on the market, the broader the adoption, easier parsing and ultimately better content is served to end users.

*[HTML]: HyperText Markup Language
*[CSS]: Cascading Style Sheets
*[W3C]: World Wide Web Consortium
