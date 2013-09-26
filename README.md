# Zend_Cache backend using Redis with full support for tags

This Zend_Cache backend allows you to use a Redis server as a central cache storage. Tags are fully supported
without the use of TwoLevels cache so this backend is great for use on a single machine or in a cluster.

## FEATURES

 - Uses the [phpredis PECL extension](https://github.com/nicolasff/phpredis) for best performance (requires **master** branch or tagged version newer than Aug 19 2011).
 - Falls back to standalone PHP if phpredis isn't available using the [Credis](https://github.com/colinmollenhour/credis) library.
 - Tagging is fully supported, implemented using the Redis "set" and "hash" datatypes for efficient tag management.
 - Key expiration is handled automatically by Redis.
 - Supports unix socket connection for even better performance on a single machine.
 - Supports configurable compression for memory savings. Can choose between gzip, lzf and snappy and can change configuration without flushing cache.
 - Uses transactions to prevent race conditions between saves, cleans or removes causing unexpected results.
 - __Unit tested!__

## INSTALLATION

 1. Install [redis](http://redis.io/download) (2.4+ required)
 2. Install [phpredis](https://github.com/nicolasff/phpredis) (optional)

   * For 2.4 support you must use the "master" branch or a tagged version newer than Aug 19, 2011.
   * phpredis is optional, but it is much faster than standalone mode
   * phpredis does not support setting read timeouts at the moment (see pull request #260). If you receive read errors (“read error on connection”), this might be the reason.
 3. Install using Composer (optional; see http://getcomposer.org)

```
{
    //...
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/bryanfagan/Cm_Cache_Backend_Redis"
        }
    ],
    // ...
    "require": {
        // ...
        "colinmollenhour/cache-backend-redis": "dev-master as 1.4.x-dev"
        // ...
    },
    // ...
}
```

## RELATED / TUNING

 - The recommended "maxmemory-policy" is "volatile-lru". All tag metadata is non-volatile so it is
   recommended to use key expirations unless non-volatile keys are absolutely necessary so that tag
   data cannot get evicted. So, be sure that the "maxmemory" is high enough to accommodate all of
   the tag data and non-volatile data with enough room left for the volatile key data as well.
 - Automatic cleaning is optional and not recommended since it is slow and uses lots of memory.
 - Occasional (e.g. once a day) garbage collection is recommended if the entire cache is infrequently cleared and
   automatic cleaning is not enabled. The best solution is to run a cron job which does the garbage collection.
   (See "Example Garbage Collection Script" below.)
 - Compression will have additional CPU overhead but may be worth it for memory savings and reduced traffic.
   For high-latency networks it may even improve performance.
   - gzip — Slowest but highest compression. Most likely you will not want to use above level 1 compression.
   - lzf — Fastest compress, fast decompress. Install: `sudo pecl install lzf`
   - snappy — Fastest decompress, fast compress. Download and install: [snappy](http://code.google.com/p/snappy/) and [php-snappy](http://code.google.com/p/php-snappy/)
 - Monitor your redis cache statistics with my modified [munin plugin](https://gist.github.com/1177716).
 - Enable persistent connections. Make sure that if you have multiple configurations connecting the persistent
   string is unique for each configuration so that "select" commands don't cause conflicts.
 - Use the `stats.php` script to inspect your cache to find oversized or wasteful cache tags.

## Release Notes

 - September 26, 2013: I forked this from Colon's code solely to make it a standalone package, vs a Magento plugin; the library is entirely the work of Colin Mollenhour
 - November 19, 2012: Added read_timeout option. (Feature only supported in standalone mode, will be supported by phpredis when pull request #260 is merged)
 - October 29, 2012: Added support for persistent connections. (Thanks samm-git!)
 - October 12, 2012: Improved memory usage and efficiency of garbage collection and updated recommendation.
 - September 17, 2012: Added connect_retries option (default: 1) to prevent errors from random connection failures.
 - July 10, 2012: Added password authentication support.
 - Mar 1, 2012: Using latest Credis_Client which adds auto-reconnect for standalone mode.
 - Feb 15, 2012: Changed from using separate keys for data, tags and mtime to a single hash per key.
 - Nov 10, 2011: Changed from using phpredis and redisent to Credis (which wraps phpredis). Implemented pipelining.

```
@copyright  Copyright (c) 2012 Colin Mollenhour (http://colin.mollenhour.com)
This project is licensed under the "New BSD" license (see source).
```
