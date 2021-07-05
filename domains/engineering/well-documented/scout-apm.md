---
title: Scout APM
nav_order: 6
parent: Well documented
grand_parent: Engineering
---

# Scout APM

## **What is APM?**

Application Performance Monitoring (APM), as the name suggests, is the process of monitoring the performance of the many aspects of your application (i.e., CPU processors, memory, storage of machines that host your web servers, APIs, databases, file systems, etc.).

## Common Use Cases for APMs

- Application Development
- Identifying Performance Bottlenecks
- Real-time Performance Alerts and Insights
- Monitor and Track End User Experience
- Cost optimisation

## Getting started with Scout

![/assets/images/scout.png](/assets/images/scout.png)

- **Response Time - MEAN**

    Average response time for a selected time frame(74.2).

- **Response Time - 95th**

    95% of requests response is below this value(127.6). Also one more thing to take note is this value should not exceed Mean response time * 4X (74.2*4 = 296.8)

- **Throughput**

    Total number of request per minute for a selected time frame

- **Errors**

    404, 500 errors happened during the selected time frame

- **Memory**

    Memory usage of the application

- **APDEX**

    Apdex is a measurement of  user's level of satisfaction based on the response time of request(s) when interacting with the application. 0-1 (0 being unsatisfactory and 1 being satisfactory). 

    ![/assets/images/apdex.png](/assets/images/apdex.png)

## What to look into Scout?

- **N+1 Insights**

    When we talk about the N+1 problem, what we are describing is the inefficient way of querying data from the database, resulting in an unnecessary amount of individual SQL queries being executed. A small inefficiency like this will often go unnoticed in development, but then suddenly cripple the application’s performance when deployed to production.

    Reference - [https://scoutapm.com/blog/finding-and-fixing-n1-queries](https://scoutapm.com/blog/finding-and-fixing-n1-queries)

    More information - [https://app.weet.co/play/0688e50a/weet-free-video-messaging](https://app.weet.co/play/0688e50a/weet-free-video-messaging)

- **Slow query Insights**

    Once your Rails app begins seeing consistent traffic, slow SQL queries will likely rear their ugly head. While PostgreSQL can log slow queries, it's difficult to get actionable information from this raw stream. The slow query logs lack application context.

    Possible solutions - indexes

    Reference - [https://scoutapm.com/blog/finding-slow-activerecord-queries-with-scout](https://scoutapm.com/blog/finding-slow-activerecord-queries-with-scout), [https://use-the-index-luke.com/](https://use-the-index-luke.com/)

    More information - [https://app.weet.co/play/5718938d/weet-free-video-messaging](https://app.weet.co/play/5718938d/weet-free-video-messaging)

- **Memory bloat Insights**

    Memory bloat is caused by loading too much into memory, usually as a result of poor memory management in the software, for example – when you load lots of unnecessary data objects into memory that are neither utilised by the application nor released from memory. Nowadays, with growing audiences and faster speed and data retrieval expectations, memory issues pose a huge threat to performance and can lead to huge losses in terms of customers and money.

    Possible solutions - Jemalloc, Frozen string literals

    Reference - [https://scoutapm.com/blog/memory-bloat](https://scoutapm.com/blog/memory-bloat) , [https://scoutapm.com/blog/manage-ruby-memory-usage](https://scoutapm.com/blog/manage-ruby-memory-usage), [https://www.mikeperham.com/2018/04/25/taming-rails-memory-bloat/](https://www.mikeperham.com/2018/04/25/taming-rails-memory-bloat/), [https://freelancing-gods.com/2017/07/27/an-introduction-to-frozen-string-literals.html](https://freelancing-gods.com/2017/07/27/an-introduction-to-frozen-string-literals.html), [https://freelancing-gods.com/2017/07/27/friendly-frozen-string-literals.html](https://freelancing-gods.com/2017/07/27/friendly-frozen-string-literals.html)

- **Errors**

    Errors happening in application. we can also integrate error tracking tools like rollbar, sentry into scout.