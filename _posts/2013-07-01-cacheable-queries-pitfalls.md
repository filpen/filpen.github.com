---
layout: post
title: "Cacheable queries pitfalls"
description: ""
category:
tags: []
date: 2013-07-01 19:00 UTC
---
{% include JB/setup %}


Cacheable queries are a powerful tool in the belt of the caching-aware developer. They save the results of a query as a collection of identifiers, so that if the same query is executed again we can avoid hitting the database and just return the previously loaded results. Even though this sounds simple, there are two common pitfalls when using it: one concerns performance directly, and the other makes your cache return stale results.

The first pitfall, common to all second level cache implementations, is not to cache the entities that are returned by the query. In fact, when NHibernate finds any cached query results, it goes through the list of identifiers to fetch the entities one by one. However, if the entities themselves are not cached we run into a problem – supposing that we cached the results of a query returning 1000 entities, NHibernate is going to execute 1000 selects.

Concretely, to avoid this problem you must make sure to have configured the entities that will be returned by your query to be cached correctly. In this case it is evident that we are degrading the performance of our system instead of improving it. However, there is a lurking misconfiguration affecting the freshness of the cached data instead of raw performance, which is much more difficult to spot.

I have noticed that developers often avoid to define a cache region for cacheable queries – which is not necessarily an issue, especially when using SysCache. Despite the fact that there is no need to set the region, if you leave it out you should do so consciously. While this configuration works fine for SysCache, this deceivingly small detail has deep repercussions on how the cache is invalidated when using SysCache2.

Although there exists a default cache region containing the cached query results (NHibernate.Cache.StandardQueryCache), if you do not explicitly define a region for a cacheable query you get no automatic invalidation via query notifications. This is due to the fact that the standard query cache region is not configured to support query notifications and it might be an issue if the default relative expiration of 5 minutes of StandardQueryCache has been increased. As a matter of fact, we manually configured some of our cache regions to expire after a time frame of multiple hours, meaning that we were seeing several instances of this problem.

In case you are not familiar with query notifications, think of them as a service that receives a query and notifies you if the result of the query has changed due to changes in the underlying data. You can then keep the program running normally until SQL server sends you a notification, at which point you will want to react and, in case you are caching that data, invalidate your stale cache entry. This is all handled for you in the background by SysCache2 and a number of BCL classes. You might have noticed that in order to check the data for changes, SQL Server needs to know which query has to be monitored. It turns out that the default query cache has no such query defined, which makes sense, considering it is the cache region used for all queries and can't predict which entities will be involved in it.

In order to solve the staleness issue, you can use a custom cache region configured for query notifications. All things considered, if we are using SysCache2 we probably already defined cache regions containing either command- or table-based dependencies, which we can reuse for invalidating stale query results. More precisely, supposing that you are using QueryOver, your cacheable query will look like this:

{% highlight csharp %}
Session.QueryOver<Employee>().Where(...).Cacheable().CacheRegion("MyCacheRegion").List();
{% endhighlight %}

Sharing the cache region between queries and entities to be cached also has an additional benefit: suppose that you cached the results of the query in a different region, and your entity cache region is invalidated. The next time you are going to perform the query you are going to incur in the first problem that we met: the resulting ids of the query are still cached, but the entities themselves are not cached anymore. If you used the same cache region for both purposes this is not going to happen.

So as you have seen, caching queries is a way to squeeze some performance out of the system, but if not configured correctly it will backfire badly.