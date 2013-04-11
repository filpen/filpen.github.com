---
layout: post
title: "Seemingly random clearing of NHibernate second level cache"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Imagine spending days carefully setting up and fine tuning NHibernate caches, the whole lot of them: session, second level and query cache. Suppose you have spent way too long in finding out how to set up SysCache2 and query notifications in SQL Server, how to create a perfect combination of permissions so that everything works smoothly. After the web app has been deployed to the test system, you receive a call asking you to check the behavior of the second level cache that may be the cause of a timeout.

Mh, sounds weird, but ok. I look into it and everything looks fine at first glance: reading the debugging output from the second level cache, it appears that everything is being cached and fetched from the cache correctly. Until I hit one button and try to reproduce that timeout, succeeding. The cache is being completely cleared and during the next attempt to fetch a cached model entity the cache is rebuilt again from scratch, blocking due to a transaction that was started by the operation itself. What? Why is it happening?

After all the time spent in the caching code, this defies everything I know about the second level cache. The cache region isn't expiring, it is being explicitly cleared even if there is no such explicit request from our code. A future me will have to admit that yes, we didn't have an explicit request to clear the cache, but we had something very close to it. A Clear() in disguise.

Stepping through the code I isolate the method call unleashing the total annihilation of the cache: it is a call to ISQLQuery.ExecuteUpdate. And then it dawns on me – we are performing a native update/delete/insert operation on the database and NHibernate has just to assume that we may have changed something that has already been cached. Which means that the second level cache is completely wiped clean in the name of consistency. Ouch.

While it may make sense for NHibernate to keep the caches consistent knowing nothing about their implementation, we are actually using SysCache2 which employs SQL Server's Query Notification system to be notified about changes in the data, automatically invalidating the related cache item. This is extremely cool – and a huge PITA to setup correctly – but now NHibernate has to ruin our fun and spoil the party. I start looking into options to limit the damage to the cache, supposing that at least it should be possible define some regions to be cleared, or maybe turn off the option completely.

While checking NHibernate sources reveals that there is an half-ported feature of Hibernate called query spaces that could be used to define regions that will be cleared, I wasn't able to leverage it and I assume it was not fully implemented in NHibernate, though I could be wrong. You can find the relevant code in BulkOperationCleanupAction.

{% highlight csharp %}
/// <summary>
/// Create an action that will evict collection and entity regions based on queryspaces (table names).  
/// </summary>
public BulkOperationCleanupAction(ISessionImplementor session, ISet<string> querySpaces)
{
    //from H3.2 TODO: cache the autodetected information and pass it in instead.
    this.session = session;

    ISet<string> tmpSpaces = new HashSet<string>(querySpaces);
    ISessionFactoryImplementor factory = session.Factory;
    IDictionary<string, IClassMetadata> acmd = factory.GetAllClassMetadata();
    foreach (KeyValuePair<string, IClassMetadata> entry in acmd)
    {
        string entityName = entry.Key;
        IEntityPersister persister = factory.GetEntityPersister(entityName);
        string[] entitySpaces = persister.QuerySpaces;

        if (AffectedEntity(querySpaces, entitySpaces))
        {
            if (persister.HasCache)
            {
                affectedEntityNames.Add(persister.EntityName);
            }
            ISet<string> roles = session.Factory.GetCollectionRolesByEntityParticipant(persister.EntityName);
            if (roles != null)
            {
                affectedCollectionRoles.UnionWith(roles);
            }
            for (int y = 0; y < entitySpaces.Length; y++)
            {
                tmpSpaces.Add(entitySpaces[y]);
            }
        }
    }
    spaces = new List<string>(tmpSpaces);
}

private bool AffectedEntity(ISet<string> querySpaces, string[] entitySpaces)
{
    if (querySpaces == null || (querySpaces.Count == 0))
    {
        return true;
    }

    for (int i = 0; i < entitySpaces.Length; i++)
    {
        if (querySpaces.Contains(entitySpaces[i]))
        {
            return true;
        }
    }
    return false;
}
{% endhighlight %}

As you see, it looks like there is a possibility to define query spaces so that the AffectedEntity method can discard only the relevant regions, yet I couldn't find a public method where to inject the query spaces. So no luck leveraging NHibernate features for me, I would have to solve the issue myself in the less hacky way possible. It turns out that there is a workaround to this issue that is also quite easy to implement.

The solution is simply to remove all the ISQLQuery.ExecuteUpdate calls and execute the relevant statements over ADO.NET, reusing the connection and enlisting the command in the existing transaction. Doing this we effectively perform the update/insert/delete statements behind NHibernate's back, however we will not need to worry about inconsistencies since SysCache2 is there to save the day. Thank you, SysCache2: the pain of setting up the permissions and double checking that you were working correctly was totally worth it.