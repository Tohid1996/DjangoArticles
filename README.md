# What is caching?
Caching is the process of storing copies of files in a **cache**, or temporary storage location, so that they can be accessed more quickly. Technically, a cache is any temporary storage location for copies of files or data, but the term is often used in reference to Internet technologies. Web browsers cache HTML files, JavaScript, and images in order to load websites more quickly, while **DNS** servers cache **DNS** records for faster lookups and **CDN** servers **cache** content to reduce **latency**.

To understand how caches work, consider real-world **caches** of food and other supplies. When explorer Roald Amundsen made his return journey from his trip to the South Pole in 1912, he and his men subsisted on the caches of food they had stored along the way. This was much more efficient than waiting for supplies to be delivered from their base camp as they traveled. Caches on the Internet serve a similar purpose; they temporarily store the 'supplies', or content, needed for users to make their journey across the web.
## What does a browser cache do?
Every time a user loads a webpage, their browser has to download quite a lot of data in order to display that webpage. To shorten **page load times**, browsers cache most of the content that appears on the webpage, saving a copy of the webpage's content on the device’s hard drive. This way, the next time the user loads the page, most of the content is already stored locally and the page will load much more quickly.

Browsers store these files until their **time to live (TTL)** expires or until the hard drive cache is full. (TTL is an indication of how long content should be cached.) Users can also clear their browser cache if desired.
## What does clearing a browser cache accomplish?
Once a browser cache is cleared, every webpage that loads will load as if it is the first time the user has visited the page. If something loaded incorrectly the first time and was cached, clearing the cache can allow it to load correctly. However, clearing one's browser cache can also temporarily slow page load times.
## What is CDN caching?
A **CDN**, or **content delivery network**, caches content (such as images, videos, or webpages) in proxy servers that are located closer to end users than **origin servers**. (A proxy server is a server that receives requests from **clients** and passes them along to other servers.) Because the servers are closer to the user making the request, a CDN is able to deliver content more quickly.

![CDN caching](https://www.cloudflare.com/img/learning/cdn/what-is-a-cdn/what-is-a-cdn.png)

Think of a CDN as being like a chain of grocery stores: Instead of going all the way to the farms where food is grown, which could be hundreds of miles away, shoppers go to their local grocery store, which still requires some travel but is much closer. Because grocery stores stock food from faraway farms, grocery shopping takes minutes instead of days. Similarly, CDN caches 'stock' the content that appears on the Internet so that webpages load much more quickly.
When a user requests content from a website using a CDN, the CDN fetches that content from an origin server, and then saves a copy of the content for future requests. Cached content remains in the CDN cache as long as users continue to request it.
## What is a CDN cache hit? What is a cache miss?
A **cache hit** is when a client device makes a request to the cache for content, and the cache has that content saved. A cache miss occurs when the cache does not have the requested content.

A cache hit means that the content will be able to load much more quickly, since the CDN can immediately deliver it to the end user. In the case of a cache miss, a CDN server will pass the request along to the origin server, then cache the content once the origin server responds, so that subsequent requests will result in a cache hit.
## Where are CDN caching servers located?
CDN caching servers are located in **data centers** all over the globe. Cloudflare has CDN servers in 200 cities spread out throughout the world in order to be as close to end users accessing the content as possible. A location where CDN servers are present is also called a data center.
## How long does cached data remain in a CDN server?
When websites respond to CDN servers with the requested content, they attach the content’s TTL as well, letting the servers know how long to store it. The TTL is stored in a part of the response called the **HTTP** header, and it specifies for how many seconds, minutes, or hours content will be cached. When the TTL expires, the cache removes the content. Some CDNs will also purge files from the cache early if the content is not requested for a while, or if a CDN customer manually purges certain content.
## How do other kinds of caching work?
**DNS caching** takes place on DNS servers. The servers store recent DNS lookups in their cache so that they do not have to query nameservers and can instantly reply with the **IP address** of a domain.

**Search engines** may cache webpages that frequently appear in search results in order to answer user queries even if the website they are attempting to access is temporarily down or unable to respond.

----
One of the often overlooked but critical components of a functioning web application is its performance. Not only does it have a dramatic impact on the user experience of a front-end user, it also has implications on the reliability and cost-effectiveness of the deployment of the application. This article will cover how to effectively use different caching techniques to improve the performance of a python / Django web application.

# Why is caching important?

For several reasons:

- **A better user experience**. Frustration levels decrease and conversion rates increase as page performance improves.
- **SEO**. Google and other search engines give preferential ranking to websites that have a lower average response time.
- **Cost-effective infrastructure**. Utilizing caching to reduce server workload can let you built out a less costly, more reliable web application.
- **Slow performance can cause outages**. This can’t be stressed enough; if enough requests are stalled in line waiting for a slow page to load, you can have an outage. Load balancers will
    start responding with 503 messages and your website will be down.

It almost never makes sense not to cache. In most cases, an invalidation strategy can be created that makes caching transparent. However, some workloads (write-heavy and read light) may incur more overhead from caching and invalidation than savings.
## Types of cache

There are several different types of caches, at different layers that can be utilized in a python / Django web application. Although the list below is not comprehensive, it does cover the most
common caching techniques used.

- **Upstream caches**: These caches intercept the HTTP request before it is sent to the application server and attempt to service the request from the cache before sending it to the back end. Some
    upstream caches are sophisticated enough to run complicated logic to determine which pages can be cached and which can’t or even return different versions of caches depending upon
    certain user conditions. Common upstream caches include Varnish, Squid and hosted CDNs like Akamai.
- **Downstream caches**: Typically a middleware that intercepts a request before it is routed further into the application and attempts to server it from the cache. Can be very powerful in that
    it’s done in application code, so very sophisticated caching logic can be implemented. Its downside is that it ties up an application server process to serve a cached page.
- **ORM Cache**: This is a cache that stores the result of queries made to the ORM and tries to serve them from the cache whenever possible. The primary purpose of an ORM cache is to
    relieve query pressure from the database. Examples include johnny cache and cachalot.
- **Template fragment caching**: Caches a chunk of rendered HTML. Built into Django core.
- **Manual**: The built-in caching primitives exposed in Django's caching api. Can be used to cache expensive, repeated computations.
    
## ORM Caching

In Django, the ORM provides a great layer in which you can inject caching mechanisms across all ORM calls to a database server. Two projects exist currently that do this for you: johnny cache and cachalot. These use a clever mechanism to reduce query overhead to your database server:

- Whenever a write is made to any table, a cache key is set with both the table name and the time of the write. This marks the table as “dirty” for any cached query made before this write.
- Any time a SQL query meets certain conditions (is cacheable), a hash is made of the query and the query, the time it was generated, and the database tables it depends upon are cached
- When a query is about to be run, the cache is checked first to see if a cached result exists for that query.
- If a cached result exists, the time the result was generated is compared against all of the tables that the result depends on and the last time they were written to. If the tables have not been modified since the query was cached, the cached result is used and the database backend is not queried. If a table has been modified, the cached result is not used and the query is sent to the database for processing.

What’s nice about ORM caching is it’s plug and play. Simply enabling the ORM caching middleware gains you the performance benefit without having to do any further coding (other than testing). The only thing to keep in mind when using a caching engine like this is the additional overhead you incur for tables that are write-heavy; it’s best to “blacklist” these tables if that is the case so the ORM cache doesn’t attempt to cache them. Another benefit of this type of caching is that it’s not time-based upon it’s expiration; if your data never changes, the cache never expires. Certain workloads can benefit greatly from this type of caching.

One thing to keep in mind when using an ORM cache is to make sure that your queries can be cached. Sometimes you can make minor modifications to your application to make queries cacheable that have little to no user impact:

- Round timestamps to the nearest 5 minutes. If you’re checking for something to have expired, most of the time it’s OK if that window is 5 minutes off. This will help like requests in the same 5-minute window to share a cache key, instead of generating a new query because of a difference of seconds or even milliseconds.
- Don’t use certain SQL operators that cannot be cached if you can void it (NOW(), for instance)
- Don’t use random sorting in SQL; not only is it bad for caching, but it’s also bad for performance for large result sets.
- Optimize expressions whenever possible so cache keys overlap. For example, if SQL is written as X + 5 > 10in one place and X > 5 in another, that will generate two different cache keys.
- Use model methods so SQL queries are the same. If you re-implement SQL queries in different views, minor differences in how the query is generated may cause cache misses. It also has the side benefit of improving your code quality.
- Avoid user-defined functions and stored procedures.
- Avoid using subqueries (this also adds a performance benefit in most cases). 

## Template fragment caching

Template fragment caching is the ability to cache, given a key and possible vary parameters, a chunk of django template:

    1234{% load cache %}{% cache 500 sidebar request.user.username %} .. sidebar for logged in user ..{% endcache %}

This can be useful for several reasons:

- Expensive template rendering can be cached between pages so crawlers don’t have a negative impact on performance
- Pages that render the same item more than once on a page (a buy box, for instance) can share the cached versions between the first and second renders.
- Your upstream cache may have a lower cache time (5 minutes, for example) than needed for some content. You can selectively wrap content that can live with a higher cache time in template fragment caching.

A few common use cases where this can be useful include shared navigation between pages, or a shared header or footer. The Django cache tag supports multiple various parameters as well, which gives you plenty of flexibility when setting up fragment caching. Although it’s good to cache in layers, it doesn’t really make sense to fragment cache entire pages; an upstream cache is more effective for this.

Cache backends

Once you implement template fragment caching, ORM caching, and even native caching techniques into your application, it’s important to make sure that you use the correct cache backend to support your workload. There are several that Django support; we’ve listed a few here with comments around the positives and negatives of each. This list is not comprehensive, but it does cover some of the common backends you may see.

- **Memcached**: The gold standard for caching. An in-memory service that can return keys at a very fast rate. Not a good choice if your keys are very large in size
- **Redis**: A good alternative to Memcached when you want to cache very large keys (for example, large chunks of rendered JSON for an API)
- **Dynamodb**: Another good alternative to Memcached when you want to cache very large keys. Also scales very well with little IT overhead.
- **Localmem**: Only use for local testing; don’t go into production with this cache type.
- **Database**: It’s rare that you’ll find a use case where the database caching makes sense. It may be useful for local testing, but otherwise, avoid it.
- **File system**: Can be a trap. Although reading and writing files can be faster than making SQL queries, it has some pitfalls. Each cache is local to the application server (not shared), and if you have a lot of cache keys, you can theoretically hit the file system limit for the number of files allowed.
- **Dummy**: A great backend to use for local testing when you want your data changes to be made immediately without caching. Be warned: permanently using dummy caching locally can hide bugs from you until they hit an environment where caching is enabled.
