--- 
wordpress_id: "28"
layout: blog_post
title: Caching Filter Queries with Coherence
wordpress_url: http://martin.elwin.com/blog/?p=28
---
A pretty nice thing that I recently tried with a customer was storing a query result in a query cache.

For instance, consider the following method:

{% highlight java %}
public Collection getByFilter1(String cacheName, Filter f) {
    NamedCache c = CacheFactory.getCache(cacheName);
    return c.entrySet(f);
}
{% endhighlight %}

A query is executed across all nodes containing cache data in a cluster. The filter acts on the <em>values</em> in the cache, not the keys. And as all values are usually not available locally on every node the query needs to execute on the separate nodes.

The query above is correct, but not very efficient. In the above case we query and retrieve the data as well from the separate nodes. A more efficient way would be to only get the keys for the matching values from the nodes and then retrieve the values from the local near cache:

{% highlight java %}
public Collection getByFilter2(String cacheName, Filter f) {
    NamedCache c = CacheFactory.getCache(cacheName);
    Set keys = c.keySet(f);
    return c.getAll(keys).values();
}
{% endhighlight %}

However, in this case we still perform the query every time.

Now - the clever part. Not so much on my side, but the Coherence engineers have thought things through and made the Filter implementations have good hashCode and equals implementations. This together with the fact that they are serializable makes them possible to use as keys in a cache! Sweet! Without changing our method's interface we can add a query cache so that each query only is performed once.

{% highlight java %}
public Collection getByFilter3(String cacheName, Filter f) {
    NamedCache c = CacheFactory.getCache(cacheName);

    NamedCache queryC = CacheFactory.getCache(cacheName + ".querycache");
    Set keys = (Set)queryC.get(f);

    if(keys == null) {
        keys = c.keySet(f);
        queryC.put(f, keys);
    }

    return c.getAll(keys).values();
}
{% endhighlight %}

Note that we only save the keys in the query cache. This to avoid having several caches with the same data. When near caching is used, getting the data for the keys can still be a local only operation. Compared to getting the data from the separate nodes it's several orders of magnitude faster - depending on the usage patterns.

Of course if queries are different every time, the query cache will not help much. But in most high load applications the same data tends to be needed several times.

Additionally, the properties queried on should in most cases be indexed. This is important to avoid too much overhead when searching for an entry in a cache.

One thing to think about when adding query caches is: how is data updated?

As part of updating the value caches the query caches should preferably be emptied of the outdated cached queries. This could be done programmatically "manually" when the data is updated, or by hooking in code to clean the entries from the query cache using the map listener mechanism. The relevant code could do a query using the ContainsFilter.

To summarize: a little code can go a long way to improve performance without affecting the interface used by an application. Good when query heavy applications are adapted to use a distributed cache like Coherence.

/M

