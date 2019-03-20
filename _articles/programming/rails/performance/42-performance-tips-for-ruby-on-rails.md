# 42 performance tips for Ruby on Rails

> Original article: <https://www.mskog.com/posts/42-performance-tips-for-ruby-on-rails/>

Since Ruby on Rails is not the fastest web framework out there you sometimes need to improve the performance in order to keep up. This post will list 42 of my favorite performance tips for making your app faster.

## TLDR

This is very long so here are my personal favorites if you don't feel like reading the whole thing:

-   [Use rack-mini-profiler](#use-rack-mini-profiler)
-   [Puma tuning](#puma-tuning)
-   [Use the Google pagespeed module](#try-the-pagespeed-module-from-google)
-   [Eliminate N+1 queries](#eliminate-n-1-queries)
-   [Use proper indexes for full text search](#use-proper-indexes-for-full-text-search)
-   [Page and Action caching](#page-and-action-caching)
-   [Use a fast JSON gem](#use-a-fast-json-gem)
-   [Try a timed cache](#does-it-have-to-be-real-time-no-try-a-timed-cache)
-   [Use a faster language for jobs](#use-a-faster-language-for-jobs)

## Let's get started!

### Temper your expectations

Ruby and by extension Ruby on Rails is not known for its performance. View rendering in particular is quite slow so you should not expect to get to those sub-millisecond view renderings that you can get in Elixir for example. A good benchmark is to aim for the classic [100ms target](https://www.speedshop.co/2015/05/27/100-ms-to-glass-with-rails-and-turbolinks.html).

### Use APM

Having metrics like average page load times, how much of the request is spent in the database and so on is vital to performance optimizations. [New Relic](https://newrelic.com/) is a good way to do this but it is quite expensive for a side project. [Skylight](https://www.skylight.io/) is another option that is free for the first 100k requests each month.

### Use rack-mini-profiler

[rack-mini-profiler](https://github.com/MiniProfiler/rack-mini-profiler) is an amazing way to get helpful metrics. Once installed it will display a badge in your browser with the total request time. If you click it you get metrics for each view renderedas well as every database query that was run. It will also display how much time was spent rendering templates and how much time was spent in the database.

### Measure twice, optimize once

Always make sure that you know what you are optimizing and why. It makes little sense to do view optimization when the issue is in the database.

### 80⁄20 rule

Also known as the [Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle). Simply put, it says that 80% of the effects come from 20% of the causes. The takeaway here is that you should not spend your time on microoptimizations when there are bigger things to fix. It is for example probably not worth spending hours to shave another millisecond off that database query when the view rendering takes a second to render that list.

### Use updated versions of Ruby and Ruby on Rails

This goes without saying but performance is better in each version of Ruby. Make sure you use the latest versions. Ruby on Rails 6.0 just went beta and that has parallel testing built in for example. Definitely worth checking out!

### Make sure to measure in production mode

By default Rails will for example not cache things in the development environment so make sure to either use the production environment or manipulate the configuration manually to get the same setup as the production application.

### Run things in the background

Use [Activejob](https://edgeguides.rubyonrails.org/active_job_basics.html) and [Sidekiq](https://github.com/mperham/sidekiq) for fast background jobs. Don't send emails and such in the main thread. Offload it to workers!

### Further reading and resources

* <https://www.skylight.io/>
* <https://www.speedshop.co/2017/07/11/is-ruby-too-slow-for-web-scale.html>
* <http://engineering.appfolio.com/appfolio-engineering/2018/12/13/a-short-update-how-fast-is-ruby-260rc1>

## Servers and hardware

### Use servers with good single core performance

Having a CPU with a high clock rate will do wonders for the performance of a Ruby on Rails application. On [DigitalOcean](https://m.do.co/c/457e363d283d) for example you can get a VPS with ~30% higher clock speed if you use optimized droplets instead of regular ones. They run at ~2.6GHz. If you want a dedicated server and don't need ECC RAM then you can get a server with an [i7-8700k at Hetzner](https://www.hetzner.com/dedicated-rootserver/matrix-ex). It has a base clock speed of 3.7GHz which will do wonders. Again, make sure that you benchmark this before committing to something that might not make a difference.

### Puma tuning

Puma is the default web server for Ruby on Rails since version 5.0. By default it will launch a single worker process and use threads to process requests. To get optimal performance you should tinker with this. A good starting point is to have one worker for each CPU core on your server. You can change this in `config/puma.rb`and what you are looking for is the `workers`configuration. Keep in mind that each worker is a forked process and this will use a lot of RAM so make sure that you don't run out.

### Use HTTP2

If you are using Nginx then make sure you [enable HTTP2](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-with-http-2-support-on-ubuntu-18-04) for optimal performance. If you use [Dokku](https://github.com/dokku/dokku) then it should do that by itself if your Nginx installation supports it.

### Try the pagespeed module from Google

Google has [pagespeed modules](https://developers.google.com/speed/pagespeed/module/) for both Apache and Nginx. They can do all sorts of optimizations for you but the one I will recommend here is the one that does inline CSS. What it does is that it will attempt to find the CSS rules needed to render the page correctly and then inline that CSS instead of loading it from an external file. This means that the client does not have to wait for the external CSS to load before the page is rendered and this can increase loading times. You need to compile Nginx with support for this first and the configuration then looks a little something like this in the http block:

```console
pagespeed on;
pagespeed FileCachePath /var/ngx_pagespeed_cache;
pagespeed EnableFilters prioritize_critical_css;
pagespeed Domain  *.mydomain.com;
pagespeed InlineResourcesWithoutExplicitAuthorization Stylesheet;
```

### Make sure the server is not swapping

This goes without saying but make sure you have enough RAM. Rails is very memory hungry and it adds up quickly with multiple Puma processes and background workers.

### Have server metrics

You need to know how your servers are doing. Are they swapping? How much CPU load do you currently have etc. The tools I use for this is Telegraf+Influxdb+Grafana. Telegraf will collect metrics from my servers and push them to Influxdb. Grafana will then graph this setup for me and can even send warnings to Slack for example if something is wrong.

### Further reading

* <https://www.speedshop.co/2017/10/12/appserver.html>
* <https://lkhill.com/telegraf-influx-grafana-network-stats/>

## Database

### Eliminate N+1 queries

This is a classic problem. If you load a User with `user = User.first`and then loop through the comments by doing for example `user.comments.each...`then Rails will execute a query for each comment. You can eliminate this by doing `user = User.includes(:comments)`. The [Bullet](https://github.com/flyerhzm/bullet) gem can find these problems automatically for you. You can also use the rack-mini-profiler for this but Bullet is superior for this.

### Missing database indexes

The rack-mini-profiler above might flush out slow queries and a lot of the time this is caused by missing or suboptimal indexes. To easily find which indexes are missing you can use the [lol\_dba](https://github.com/plentz/lol_dba) gem. It will find any simple cases where an index is missing and I highly recommend using it as a baseline.

### Use the database Luke!

Make sure that you use the database whenever it is convenient to do so. Keeping giant arrays in memory and sorting them, grouping them, etc can take up a lot of CPU time if done with Ruby. The database can do this much faster but sometimes you will have to drop down to using raw SQL to get it done. If you are uncomfortable with SQL or simply don't want to use it unless you have to then you can probably skip this hint. However, this leads us to the next one....

### Learn SQL!

This one is controversial but I find that while you can certainly create complex applications without knowing any SQL in Rails you might still want to learn how to write SQL queries. If you know what you are doing you can do wonderful optimizations or certain things by using SQL. You will also be able to figure out **why**that Activerecord-query is so slow if you know things like how to use the [EXPLAIN command](https://www.postgresql.org/docs/9.1/sql-explain.html).

### Beware of pagination

If your query is slow then pagination can be a huge problem, especially if you are grouping in the query. You can often get good results by modifying how the pagination is done by not including a row count. If the pagination doesn't have to do a count to find the total number of rows then that will often improve performance significantly. You should also take a look at the modern [Pagy gem](https://github.com/ddnexus/pagy) that is much faster than other solutions.

#### Update

I was informed by the author of the Pagy gem that they have recenty added the [countless extra](https://ddnexus.github.io/pagy/extras/countless) to not have to execute an additional count query like I mention above.

### Faster Postgres counting

If you have a very large amount of data then simply counting the rows can be problematic, especially if it is a big query. You can use [Count estimates](https://wiki.postgresql.org/wiki/Count_estimate) to solve this problem but it is a bit of a pain to use.

### Eager or preloaded queries

You should always use `includes`to improve the performance of relationships in ActiveRecord. This will however always be a least one extra query. You can avoid this by using `eager_load`which will make a single query instead. However, this is not always faster and it can lead to significant memory usage. Benchmark this carefully before using it. For more details you can read [this article](http://blog.scoutapp.com/articles/2017/01/24/activerecord-includes-vs-joins-vs-preload-vs-eager_load-when-and-where)

### Use proper indexes for full text search

If you just do the simple `ILIKE '%foobar%'`in Postgres for example then that will be very slow when there is a lot of data to search. Postgres can be [**very**performant](http://rachbelaid.com/postgres-full-text-search-is-good-enough/) for full text searches if you setup the proper indexes for it. You can also use the [pg_search gem](https://github.com/Casecommons/pg_search) to make this easier. Personally I find that [Elasticsearch](https://www.elastic.co/) with the [Searchkick gem](https://github.com/ankane/searchkick) is easier to use. However, do not immediately reach for it the second you need full text search. It's another thing to maintain and there are other drawbacks when using this solution.

### Don't insert massive amounts of data using ActiveRecord

If you just loop over a giant JSON blog for example and insert records using ActiveRecord then you might find that it is very slow. I recommend using the [activerecord-import gem](https://github.com/zdennis/activerecord-import) to make this much much faster.

## Caching

The best way to make code faster is to not run it at all. So caching your data and executions makes sense in any language but especially one that is not the best performer like Ruby. There is a lot of stuff to cover here so let's get started.

### Use a proper cache store

A cache store is where the cached data is stored in a Rails application. The default is to store the cache in the file system which is much too slow for production use. You can have an in-process memory cache store by using `ActiveSupport::Cache::MemoryStore`but that is not shared between processes and will not survive a restart. A production Rails application will most likely have multiple processes serving requests, by using Puma in clustered mode for example, and having a cache that is not shared between processes is quite bad. My recommendation is to use [Memcache](https://guides.rubyonrails.org/caching_with_rails.html#activesupport-cache-memcachestore) or [Redis](https://guides.rubyonrails.org/caching_with_rails.html#activesupport-cache-rediscachestore).

If you are doing background work with Sidekiq then you already have a Redis server available so you can then use that as the cache. Be careful though if you shared the Redis instance for caching with the one for Sidekiq. Redis can easily be filled up with cache data leaving no room for the Sidekiq jobs. This is easily solved in Redis 4 though by using the `volatile-lfu`eviction policy. It will use smart algorithms to evict the best data possible. Since all the cache will have the volatile flag set it will remove the cached data to make room for Sidekiq jobs if need be.

### Use HTTP caching but be careful

HTTP caching is a bit complicated so you really should read a dedicated article about it. I recommend the one at [Heroku](https://devcenter.heroku.com/articles/http-caching-ruby-rails#conditional-cache-headers). Using `expires_in`can be a very powerful tool whenever it is ok that the data is not completely fresh. A list of articles in a blog for example does not necessarily need to be fresh on every request. Setting a 5 minutes cache on this together with a proxy server and/or a CDN can really speed things up, especially when you use it in an API so you don't have to worry about sessions.

### Page and Action caching

Page and Action caching were both removed in Rails 4.0. Page caching saves the entire HTTP response and the next one did not even hit the Rails stack at all. Action caching hits the controller but does not execute the controller action if it finds cached data. You can use the [actionpack-action_caching gem](https://github.com/rails/actionpack-action_caching) to use this in Rails 4+.

Page caching is mostly about caching static pages such as the 404 or the 500 page and perhaps some long lived static pages in your app. It is much too heavy handed for other things since it will for example ignore any query parameters.

Action caching can be very powerful since it is very fast. You can also control the cache behavior by using the `cache_path`option. Perhaps you want the cache only JSON responses for example or only cache data for users that are not logged in to your application. You can use the cache keys described in the Fragment caching section as well here. It is much faster than fragment caching so I recommend using this whenever possible, especially for mostly static applications like a blog or a CMS.

### Fragment caching

This is also known as "Russian doll caching". This is done in the view templates by using the `cache`method. You can cache each layer of the view. For example if you have a blog with multiple posts then you use something like `render partial: 'posts/post', collection: @posts, cached: true`to cache the rendering of the posts. Rails will then cache each post with it's own key as well as with a key that will change if any of the posts change. When nothing has changed it will fetch the cache with a single call to the cache and if a single post has changed it will only re-render that one. The others will be fetched from the cache like normal.

You can then nest these caches by using `cache`blocks to cache for example the comments for each post or expensive partial rendering within each post. Since the Ruby on Rails view rendering is quite slow then using this technique can really speed things up.

### Raw caching

Another way to cache things is to use the `Rails.cache.fetch`method. This will cache the result of the block given to it like so:

```ruby
Rails.cache.fetch('my_cache_key', expires_in: 1.hour) do
  SomeApi.fetch_posts
end
```

The next time you run this code with the same cache key Rails will instead fetch the result of the block from the cache. This can be used anywhere in the Rails application and it is a good way to cache external APIs and other slow data fetching stuff.

### Further reading

* <https://guides.rubyonrails.org/caching_with_rails.html><https://devcenter.heroku.com/articles/caching-strategies><https://blog.appsignal.com/2018/03/20/fragment-caching-in-rails.html>

## Misc

### Use a fast JSON gem

If you have an API for example and use the ActiveModelSerializers gem then you should try the [fast\_jsonapi gem](https://github.com/Netflix/fast_jsonapi) from Netflix. It has much better performance. You can also try the [oj gem](https://github.com/ohler55/oj) for regular JSON work. This can make a huge difference for large amounts of JSON data.

#### Update

As a friendly individual pointed out in an email there is also the [Nativeson gem](https://gitlab.com/nativeson/nativeson) which generates the JSON data in the database itself. This is thus very fast!

### Use Turbolinks

If you use the [Stimulus](https://stimulusjs.org/handbook/origin) to handle Javascript or perhaps [React on Rails](https://github.com/shakacode/react_on_rails) or something similar, then you no longer have to fear the dreaded multi bindings of events like you did for the JQuery spaghetti of the past. Turbolinks will make the app significantly faster and I think that it is worth it.

### Use link prefetching

If you start loading the next page when the user hovers over a link you can make the site appear faster. See [this article](https://www.mskog.com/posts/instant-page-loads-with-turbolinks-and-prefetch/) for more details.

### Do not render with a loop

If you have an `each`-loop in your views for example to render a partial for each item in a collection then this can get very expensive. Instead, use the collection rendering feature like so:`<%= render partial: 'item', collection: @item, as: :item %>`. This is a lot faster because in that case Rails will only initialize the template once rather than once for each item if rendered with a loop.

### Use find\_each instead of find for big database collections

If you want to loop over a big chunk of data then you don't have to load all of it into memory at once. Simply use the `ActiveRecord#find_each`method instead of `all`for example to batch the loading. By default Rails will then load 1000 items at the time.

### Use a CDN for your assets

Assets like Javascript, CSS and Images can easily be cached thanks to the built in fingerprinting in Rails. These are much better off being served by a CDN for optimal performance. You can use [Cloudflare](https://www.cloudflare.com/) for free to load all your cachable assets super fast. If you want an alternative that is not free then I recommend [BunnyCDN](https://bunnycdn.com/) as a low cost alternative.

### Minimize Javascript bundle sizes

If you just toss all the Javascript in the world into your app then the `application.js`file can be huge and then it doesn't matter much if you save 100ms on the request time since the site will still be slow. I recommend using [Bundlephobia](https://bundlephobia.com/) to find the size of any npm package before you add it to your site.

### Does it have to be real time? No? Try a timed cache!

If you for example have an expensive calculation or a slow external API call in your application that doesn't need to be real time then you can use a timed cache like so:

```ruby
Rails.cache.fetch('some_key', expires_in: 1.hour) do
  # some expensive thing
end
```

Then your app will perform the expensive operation only once an hour.

## Advanced

These tips are a bit more advanced and time consuming to setup correctly. For emergency use only!

### Try JRuby

JRuby is a a Java implementation of Ruby that runs on the JVM and it should be faster than MRI for most applications. It will consume more memory however. Keep an eye on [Truffleruby](https://github.com/oracle/truffleruby) for more exotic performance improvements.

### Use a faster language for jobs

If you have a lot of background work and Ruby simply isn't fast enough then you can use another language to process the queues. There is a [version of Sidekiq that uses Crystal](https://github.com/mperham/sidekiq.cr) and that is much faster than Ruby. There is also [Faktory](https://github.com/contribsys/faktory) which allows you to use multiple programming languages to process the jobs. You can also use [Amazon SQS](https://aws.amazon.com/sqs/) with any language(s) of your choice. This can be integrated with your application by using the [Shoryuken gem](https://github.com/phstc/shoryuken). Faktory and Shoryuken both support ActiveJob so you don't have to change much to use them. I would personally start with Crystal since that is a language that has a similar syntax to Ruby and it is very fast.

### Asynchronous partial rendering

As we previously discussed, view rendering in Rails can take up quite a bit of the request time. If you have parts of the site that are slow to render then you can try loading them asynchronously instead. This means that the server will load the partials with ajax. You can use the [render_async gem](https://github.com/renderedtext/render_async) to help you with this.

### Prefetch links

You can load the next page in the background using ajax which will make the page load seem instantaneous. There are several ways to do this like using [Instantclick](http://instantclick.io/) to load the next page when the user hovers a link or [quicklink](https://github.com/GoogleChromeLabs/quicklink) to load new pages while the browser is idle. You can also combine Turbolinks with Instantclick by using a few tricks that I write about [in this post](https://www.mskog.com/posts/instant-page-loads-with-turbolinks-and-prefetch/).

### Use materialized database views

A database view is sort of like a virtual table that is created by an SQL query. You can save a big query as a view and then access it as if it was a table. A materialized view is like a view but it also saves the result of the query. It is thus sort of like a cached dataset. Obviously this is very fast but the data will be old. Used sparingly it can do wonders for the performance however. You can use the [scenic gem](https://github.com/scenic-views/scenic) to handle all your database view needs. Make sure you use a recent version of PostgreSQL though(>= 9.4).

### Give up

Well here we are. You've tried everything and nothing works. Perhaps it is time to give up then? Give Node a shot perhaps? Elixir is kind of hot right now and its framework Phoenix is quite similar to Rails so that might fit? Or just give up completely and go full Javascript with an SPA. Good luck!
