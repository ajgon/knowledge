# Actionable Tips to Improve Web Performance with Rails

> Original article: <https://www.monterail.com/blog/actionable-tips-to-improve-web-performance-with-rails>

Recently, I had a pleasure of attending the Wroclove.rb conference in Wrocław where one talk in particular caught my attention more than others. [And not just because Ruby on Rails is my mother tongue](https://www.monterail.com/services/ruby-on-rails-development). "Web Performance with Rails" by [Stefan Wintermeyer](https://www.wintermeyer-consulting.de/) was, in my opinion, the best prepared, one of the most indispensable, and definitely the most useful of all the talks but, most importantly for me, it was a wellspring of new knowledge for me and a source of inspiration that ultimately drove me to write this post. 

In his talk, Stefan gave us a general overview of what Web performance is, why Web performance is so important, what are the qualities of a well-performing Web page, and why we should care about it. But for the purposes of this post, I'm going to focus on the second part of his presentation.

The second portion of the talk included a dense list of easily understandable processes and options to improve the Web performance of your (and not only Rails-based) page/application. These all are just low-hanging fruits, but still many developers don't know them, have forgotten about them, or they simply don't care. 

Today, I'll be giving you the second part of Stefan Wintermeyer's talk condensed into writing, with references, straightforward descriptions, and comments. All based on slides shared by the author and my notes from the conference. For the sake of logical and visual navigation, I grouped the content in two parts:

1.  Rails-related
2.  General Web performance tips

You can also check Stefan's very neat slide deck here: [WebPerformance with Rails 5.2](https://speakerdeck.com/wintermeyer/webperformance-with-rails-5-dot-2)

## Tools to Measure Web Performance

There are many tools, measurements, and qualities you can use to assess how your page works in terms of Web performance. In my experience, the two most popular ones currently are:

-   [WebPageTest](https://www.webpagetest.org/)
-   [Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)

Both tools are far from specific and allow you to assess the performance of every page using rather generic, broad indicators. 

**WebPageTest** is the more mature and battle-tested solution. Its author is a well-known and widely acclaimed figure in the Web performance world. It's easy-to-use and the results, based on multiple popular performance indicators, are presented in an accessible way. It's also open-source so you can deploy your own instance. Stefan Wintermeyer recommends it and so do I, as I used it many times for many different projects and it has never let me down.

**PageSpeed** Insights primarily uses Google's own Web performance indicator called *Speed Score* (and a host of other indicators, but this is the main one). It tests the performance of your page in both desktop and mobile environments. For each indicator, it proposes a set of guidelines to improve performance of specific elements of your page. The important thing is that Google's search engine takes results of these measurements into account. Better performance translates to a higher rank in Google's search results.

## Rails Ways of Improving Web Performance

### View: Fragment caching

This is a pretty straightforward technique. You can tell Rails to cache a part of the view based on, for example, an AR model or the association used to render it. Rails will know when to remove the cache using the updated_at property of the model. Stefan Wintermeyer demonstrated a nice example of nested fragment caching---single table rows and the whole table separately.

Reference: [RoR guides: Fragment Caching](http://guides.rubyonrails.org/caching_with_rails.html#fragment-caching)

### Database: Counter cache

If you often display a number of associated objects, you usually end up with using model.associated_objects.count which triggers the COUNT SQL query on the database. A small but forgettable improvement here entails using counter cache--- storing associated objects' count as the parent property. This way, when you set up a given association in an AR model with the counter_cache option on, the .count method will use this property instead of running additional SQL queries. The counter property is updated under the hood by AR callbacks.

Although a dedicated counter_cache option exists in ActiveRecord, Stefan demoed his own implementation of it. Don't know why. Maybe he prefers explicit solutions over implicit AR magic?

Reference: [RoR guides: Associations Basics](http://guides.rubyonrails.org/association_basics.html) (counter_cache is an option for associations definition in AR models)

### HTTP: etag and last_modified

I don't want to describe in detail how etag and last_modified HTTP headers work here, as that particular subject is broad enough to warrant a separate chapter in a relevant book. To put it simply: use these headers to manipulate HTTP response cache on the browser side.

You can leverage these features using Rails controller method called fresh_when. Basically, you can explicitly bind etag and/or last_modified headers of the response to the state of your application.

For example, if the response for GET /products/:id can be cached on the browser side and the cache should be invalidated only if the updated_at property of a given product changes, fresh_when is the way to go.

Reference: [RoR API: ActionController#fresh_when](http://api.rubyonrails.org/v5.1/classes/ActionController/ConditionalGet.html#method-i-fresh_when)

### Controller + server: Page caching

*The fastest page is delivered by Nginx without ever contacting Rails*

Example: 500.html, 404.html and similar pages are served quickly by a server because we don't need to bother Rails to do so. These views are already generated.

Rails allows us to do the same with controller responses using the caches_page feature. In general, you can tell the controller which page to pre-render and save in the public directory and what are conditions of invalidating this cached page (using the expire_page and expire_cache methods). We can also gzip cached pages on save. Nginx will use these and serve the pages quickly instead of engaging with the Rails app.

But wait...

This works nice for pages that change rarely (e.g. a public static landing page). But what if we have custom, dynamic pages, with current user data for example? During the talk, Stefan said that it's possible to cache these, too, but the process is much more complicated and requires much greater effort.

One option to consider in this case is using a header for the client to cache the endpoint and set a proper [Vary header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary).

Another thing to consider is the disk space required for storing the cache if you handle a sizeable number of users (but disk space is cheap nowadays, isn't it?).

Reference:

-   [RoR guides: Page caching](http://guides.rubyonrails.org/v3.2/caching_with_rails.html#page-caching)
-   [Stefan Wintermeyer: Caching in Ruby on Rails 5.2](https://medium.com/rubyinside/https-medium-com-wintermeyer-caching-in-ruby-on-rails-5-2-d72e1ddf848c)

### Uploads and images: ActiveStorage

Stefan Wintermeyer recommends using ActiveStorage to properly serve images. He probably just likes the common Railsy interface.

No matter the tool, the goal Stefan wants us to achieve is to serve different image resolutions and formats for different browsers and devices, to accomplish optimal fit. The ActiveStorage interface allows us to manipulate images pretty easily. To detect browser/device types, you'll need some other tools what weren't expressly brought up by the author.

This is just kind advice and a good practice to follow when Web performance is a high priority.

Reference: [RoR guides: ActiveStorage](http://edgeguides.rubyonrails.org/active_storage_overview.html)

Other Tips and tricks to Improve Web Performance
------------------------------------------------

### Use HTTP/2

Wintermeyer claims that simply switching from HTTP/1.1 to HTTP/2 provides an average Web performance boost of 20% right out of the gate. Sounds awesome, doesn't it? 

Unfortunately, he didn't provide any source for this and a quick Google search says that:

*It's possible to get great performance boost our of the box, but **it depends**.*

Well, it depends on a number of factors, some of which are not always obvious and can be as complex as our Web systems are. Moreover, current good practices may ultimately prove a hindrance to you in the new HTTP/2 world. Allow me to quote from The New Stack's [How to Use HTTP/2 to Speed Up Your Websites and Apps](https://thenewstack.io/take-advantage-http2-speed-web-sites-apps/) (emphasis mine):

***(With HTTP/1.1)** If you want to be as fast as possible, you don't use encryption, you shard domains, you never embed anything on pages, you try and concatenate things into as large files as possible, so you have fewer files. In this world **(HTTP/2)** it actually makes sense to slice things up into smaller pieces so you can cache smaller pieces and you can download everything in parallel, you have to have encryption on by default and sharding is the worst possible thing you can do.*

HTTP/2 itself doesn't require encryption by default, but browsers with HTTP/2 support require it to force best practice.

You probably will get a free boost after switch to HTTP/2, but results may vary.

### CDN with HTTP/2 are not so useful

There's no longer a need for CDNs if you have HTTP/2 enabled. According to Stefan, the reasons for that include the new features of the updated:

-   Multiplexing (multiple resources can be loaded in parallel over a single connection).
-   Server Push (your server can push resources if it concludes the client will need them before the browser parses the HTML and sends requests for embedded assets).

He recommends to simply omit CDN and serve assets directly from your Rails server.

While these are really great features and make some of CDN techniques unnecessary, Stefan probably forgot about something. There's one context related to CDN that HTTP/2 is simply not able to fix---geodistance and its influence on RTT (round trip time). To optimize this, we still need CDNs. We also can be sure that CDNs will introduce new optimization techniques to leverage HTTP/2 features even more. I wouldn't strip them quite so easily.

Reference: [Quora: Does HTTP/2 decrease or eliminate the need for CDN?](https://www.quora.com/Does-HTTP-2-decrease-or-eliminate-the-need-for-CDN)

### Prefer Brotli over Gzip

Brotli is a compression algorithm based on gzip, with some additions and improvements for better compression ratio thrown in. Current Web servers and browsers support it and adding it is very easy, yielding an average compression gain of 10 to 20%. This means almost free Web performance improvement.

You should set up your application to support both compression algorithms, but prefer brotli over gzip. In case of browsers not supporting brotli (see CanIuse reference below), take care to give the browser the ability to fall back to gzip.

Reference:

-   [CanIuse: brotli](https://caniuse.com/#feat=brotli)[ ](https://caniuse.com/#feat=brotli)[(wide](https://caniuse.com/#feat=brotli)[ support)](https://caniuse.com/#feat=brotli)
-   [Gzip vs Brotli](http://www.instantshift.com/2018/03/02/gzip-vs-brotli-compression/)

### Heroku vs Bare Metal

*Heroku is good for a quick start but has never been a good choice for good WebPerformance. Bare Metal is the way to go if you need maximum WebPerformance.*

*BTW: It's cheaper too.*

Let's explain this quote from Stefan's talk. He mentioned Heroku as it's the most popular tool, but he means all similar automagical cloud hosting solutions. They are very convenient, you can get a basic setup for your app running in minutes, but it's not the best fit in context of Web performance. You don't have enough control over infrastructure and machine setup. If Web performance is a really high priority for you, you should use a separate dedicated server, widely known as [bare metal](https://en.wikipedia.org/wiki/Bare-metal_server), and adjust it's config and Web performance techniques to fit your application's specific needs.

### Resource Hints: dns-prefetch

*Gives a hint to the browser to perform a DNS lookup in the background to improve performance.*

Example:

```html
<link rel="dns-prefetch" href="http://example-domain.com/">
```

Reference:

-   [W3C: Resource Hints#dns-prefetch](https://www.w3.org/TR/resource-hints/#dfn-dns-prefetch)
-   [CanIuse: dns-prefetch](https://caniuse.com/#feat=link-rel-dns-prefetch)[ ](https://caniuse.com/#feat=link-rel-dns-prefetch)[(wide](https://caniuse.com/#feat=link-rel-dns-prefetch)[ support)](https://caniuse.com/#feat=link-rel-dns-prefetch)

### Resource Hints: prefetch

*Informs the browsers that a given resource should be prefetched so it can be loaded more quickly.*

Example:

```html
<link rel="prefetch" href="(url)">
```

Reference:

-   [W3C: Resource Hints#prefetch](https://www.w3.org/TR/resource-hints/#dfn-prefetch)
-   [CanIuse: prefetch](https://caniuse.com/#feat=link-rel-prefetch)[ ](https://caniuse.com/#feat=link-rel-prefetch)[(good](https://caniuse.com/#feat=link-rel-prefetch)[ support w/o Safari)](https://caniuse.com/#feat=link-rel-prefetch)

### Resource Hints: prerender

*Gives a hint to the browser to render the specified page in the background, speeding up page load if the user navigates to it.*

Example:

```html
<link rel="prerender" href="(url)">
```

Reference:

-   [W3C: Resource Hints#prerender](https://www.w3.org/TR/resource-hints/#dfn-prerender)
-   [CanIuse: prerender](https://caniuse.com/#feat=link-rel-prerender)[ ](https://caniuse.com/#feat=link-rel-prerender)[(very](https://caniuse.com/#feat=link-rel-prerender)[ poor support)](https://caniuse.com/#feat=link-rel-prerender)

### Resource Hints: preconnect\
Stefan Wintermeyer didn't mention this one, but it's part of the same "Resource Hints family."

*Gives a hint to the browser to begin the connection handshake* *(DNS, TCP, TLS) in the background to improve performance.*

Example:

```html
<link rel="preconnect" href="https://example-domain.com/">
```

Reference:

-   [W3C: Resource Hints#preconnect](https://www.w3.org/TR/resource-hints/#dfn-preconnect)
-   [CanIuse: preconnect](https://caniuse.com/#feat=link-rel-preconnect)[ ](https://caniuse.com/#feat=link-rel-preconnect)[(average](https://caniuse.com/#feat=link-rel-preconnect)[ support)](https://caniuse.com/#feat=link-rel-preconnect)

## HTTP/2 PUSH

*HTTP/2 PUSH allows a Web server to send resources to a Web browser before the browser gets to request them. It is, for the most part, a performance technique that can help some websites load twice or thrice as fast.*

Reference: [A closer look to HTTP/2 PUSH](https://www.shimmercat.com/en/blog/articles/whats-push/#vw0)

### Set a time budget!

It is highly recommended you set up a time budget for features loaded by your page. If loading/rendering/whatever takes longer than the arbitrary limit you set, you should cancel loading rest of unnecessary features.

This is just a general rule, implementation details may differ on a case by case basis (example: abort trailing XHR requests).

To set proper arbitrary time limits, you should use well-researched and tested performance models like the [RAIL performance model by Google](https://developers.google.com/web/fundamentals/performance/rail).

### Turn on caching on dev mode

We often turn off caching on dev mode. Wintermeyer recommends turning it on in order to catch obvious caching bugs before they get to production. Trust Stefan and trust me: cache-related bugs are always a pain in the ass. Hard to identify, hard to describe, almost impossible to reproduce. Avoid them if you can.

### Read Ilya Grigorik's book!

If you need a deeper dive into Web performance (and the Web in general) basics, Stefan recommends Ilya Grigorik's book *High Performance Browser Networking*.

I read it and can't recommend it enough. It's a great read and contains the perfect amount of details, enough to give you an understanding of how it all works under the hood without bogging you down or confusing you. Although published in 2013, the contents are still relevant and feel up-to-date. Moreover, Ilya Grigorik has had a hand in drafting many of the modern W3C standards and documents.

Read it on [Safari Books: High Performance Browser Networking](https://www.safaribooksonline.com/library/view/high-performance-browser/9781449344757/).

## Final words

[Web performance is hard and tricky.](https://apiumhub.com/tech-blog-barcelona/web-performance-optimization-techniques/) A big part of the list above pertains to caching, making it even harder. Use it wisely.

Stefan Wintermeyer did some good work putting all these techniques and bits of advice together. I learned a great deal from his 45-minute-long talk and I believe you may just learn something useful from it as well.

--- *Radek Markiewicz*
