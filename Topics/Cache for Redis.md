# [Azure Cache for Redis](https://docs.microsoft.com/en-us/azure/azure-cache-for-redis/)

Endpoint: `<account>.redis.cache.windows.net`.

Offers both the open-source OSS Redis and a commercial Redis Enterprise as a managed service.

## Common patterns

- **Data cache**: Load large databases into a cache as needed using the cache-aside pattern for faster access. Changes to data can also update the cache, which is shared with other clients.
- **Content cache**: Use an in-memory cache for static content like headers, footers, and banners of web pages. This provides quick access compared to backend datastores.
- **Session store**: Store user history data (e.g., shopping carts) associated with user cookies using an in-memory cache like Azure Cache for Redis for faster retrieval than a full relational database.
- **Job and message queuing**: Queue tasks that take time to execute, deferring longer operations to be processed in sequence, often by another server. This is known as task queuing.
- **Distributed transactions**: Execute a series of commands against a backend data-store as a single atomic operation using Azure Cache for Redis' support for transactions.

### [**Cache-Aside Pattern**](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)

A strategy to load data into a cache from a data store on demand, improving performance and maintaining data consistency. If data is not in the cache, it's retrieved from the data store and added to the cache. If data is updated, it's modified in the data store and the corresponding cache item is invalidated. Use Cache-Aside when a _cache doesn't provide native read-through and write-through operations_, or when _resource demand is unpredictable_. If the data is static and fits in the cache, better load it at startup and prevent it from expiring. Avoid this pattern for caching session state in a web farm to prevent client-server affinity dependencies.

Example:

```csharp
async Task<MyEntity> GetMyEntityAsync(int id)
{
    var key = $"MyEntity:{id}";
    var Connection = ConnectionMultiplexer.Connect("your-redis-connection-string");
    var cache = Connection.GetDatabase();
    var json = await cache.StringGetAsync(key);
    var value = string.IsNullOrWhiteSpace(json) ? default : JsonConvert.DeserializeObject<MyEntity>(json);
    if (value == null) // Cache miss
    {
        value = ...; // Retrieve from data store
        if (value != null)
        {
            await cache.StringSetAsync(key, JsonConvert.SerializeObject(value));
            await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(5));
        }
    }
    return value;
}
```

## [Caching](https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching)

| Feature           | Private Caching                                | Shared Caching                                           |
| ----------------- | ---------------------------------------------- | -------------------------------------------------------- |
| **Definition**    | Stores data locally on a specific computer.    | Common source accessible by multiple processes/machines. |
| **Accessibility** | Specific to one instance or user.              | Accessible to multiple instances or applications.        |
| **Speed**         | Generally faster (local access).               | May be slower (not held locally).                        |
| **Scalability**   | Limited; each instance has its own cache.      | Highly scalable; can be distributed across servers.      |
| **Consistency**   | Can lead to inconsistencies between instances. | Ensures consistent view of cached data.                  |
| **Complexity**    | Simpler to implement and manage.               | May add complexity (separate cache service).             |
| **Use Cases**     | Static data; single user/session specific.     | Frequently accessed data by multiple users/applications. |

### Cache expiration

Data in a cache is usually a copy of the original data. If the original changes, the cached data can become outdated. Caching systems often allow you to set expiration times to keep data fresh.

Eviction Policies:

- Most-Recently-Used: (LIFO) Assumes data won't be needed again.
- First-In-First-Out: Oldest data is removed first.
- Explicit Removal: Based on triggered events like data modification.

## Tiers

- **Basic**: A single VM running an OSS Redis cache. It is suitable for development/test and noncritical workloads, but it lacks a service-level agreement (SLA).
- **Standard**: This tier involves two VMs in a replicated configuration for the OSS Redis cache.
- **Premium**: Offering high performance, Premium caches use more powerful VMs than Basic or Standard caches. They provide higher throughput, lower latency, better availability, and additional features.
- **Enterprise**: These high-performance caches are powered by Redis Labs' Redis Enterprise software. They **support Redis modules** like RediSearch, RedisBloom, and RedisTimeSeries, and offer even greater availability than the Premium tier.
- **Enterprise Flash**: This tier provides cost-effective large caches powered by Redis Labs' Redis Enterprise software. It extends Redis data storage to nonvolatile memory on a VM, which is cheaper than DRAM, reducing the overall per-GB memory cost.

Clustering support (Premium+): Automatically split dataset among multiple nodes (shards), 10 shards max.

## [Session State Providers](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/cache-aspnet-session-state-provider)

- **In Memory**: Simple and fast. Not scalable, as it's not distributed.
- **SQL Server**: Allows for scalability and persistent storage. Can affect performance, though In-Memory OLTP can improve it.
- **Distributed In Memory** (e.g., Azure Cache for Redis): Combines simplicity, speed, and scalability. Must consider characteristics like transient network failures.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
  <providers>
    <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider"
         host=""
         accessKey=""
         ssl="true" />
  </providers>
</sessionState>
```

## Working with Redis

Create cache:

```sh
az redis create
    --name $name # must be globally unique
    --location $location # region where you want to create the cache
    --sku <sku> # pricing tier
    --vm-size <vm-size> # depends on the chosen tier
    --shard-count <shard-count> # number of shards to be used for clustering (Premium and Enterprise only). Max: 10
    --resource-group $resourceGroup
```

Redis Commands:

| Command                  | Description                              | Returns                    |
| ------------------------ | ---------------------------------------- | -------------------------- |
| `ping`                   | Checks server response.                  | "PONG"                     |
| `set [key] [value]`      | Stores a key-value pair.                 | "OK" if successful         |
| `get [key]`              | Retrieves value of a key.                | Value of the key           |
| `exists [key]`           | Checks if a key exists.                  | '1' for exist, '0' for not |
| `type [key]`             | Identifies value's type for a key.       | Type of the value          |
| `incr [key]`             | Increases key's value by 1.              | New value                  |
| `incrby [key] [amount]`  | Increases key's value by given amount.   | New value                  |
| `del [key]`              | Deletes a key's value.                   | -                          |
| `flushdb`                | Clears all database data.                | -                          |
| `expire [key] [seconds]` | Sets a key to expire after a given time. | -                          |
| `ttl [key]`              | Checks remaining time-to-live of a key.  | Time left in seconds       |

When the TTL expires, the key is auto-deleted, mimicking the `DEL` command. You can set the expiration time with up to 1-millisecond precision. Expiration data is replicated and saved on disk, preserving key expiration even if the Redis server stops.

Example:

```txt
> set counter 100
OK
> expire counter 5
(integer) 1
> get counter
100
... wait ...
> get counter
(nil)
```

`EXPIRE key seconds [NX | XX | GT | LT]`:

- NX: Set expiry only when the key has no expiry
- XX: Set expiry only when the key has an existing expiry
- GT: Set expiry only when the new expiry is greater than current one
- LT: Set expiry only when the new expiry is less than current one

## [Key eviction](https://redis.io/docs/reference/eviction/)

Redis's `maxmemory` directive controls memory usage, and its eviction policies (such as LRU and LFU) automatically remove old data as new data is added.

Example: `maxmemory 100mb`

- **noeviction**: Retains data up to memory limit; applies to primary databases with replication.
- **allkeys-lru**: Maintains recent keys; discards least recent.
- **allkeys-lfu**: Retains frequent keys; discards least frequent.
- **volatile-{lru/lfu}**: Removes least recent/frequent keys if _expiration is set_.
- **allkeys-random**: Deletes keys randomly for new data.
- **volatile-random**: Randomly removes keys if _expiration is set_.
- **volatile-ttl**: Removes keys with the shortest TTL (_expiration is set_).

Note: volatile is for when _expiration is set_.

## [Data persistence](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/cache-how-to-premium-persistence)

- **RDB**: Creates binary snapshots, stored in Azure Storage. Configurable save intervals. Restores cache from latest snapshot if both primary and replica fail.
- **AOF**: Logs write operations, saved at least once per second in Azure Storage. Rebuilds cache using logs if both primary and replica fail.

Note: These methods only restore data to the original cache, not to a new one.

## [Geo-replication](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/cache-configure#geo-replication)

Geo-replication links two Premium Azure Cache for Redis instances: a primary linked cache (read/write) and a secondary linked cache (read-only). Data written to the primary is replicated to the secondary, enabling cache replication across Azure regions.

## Access a Redis cache from a client

Retrieve the required information from the Azure portal under **Settings > Access Keys**:

- Host name: The public Internet address of your cache, created using the cache name (e.g., `sportsresults.redis.cache.windows.net`).
- Port: The port number for the Redis cache.
- Access key: Acts as a password for the cache, with both primary and secondary keys available for use.

```cs
var connectionString = "[cache-name].redis.cache.windows.net:6380,password=[password-here],ssl=True,abortConnect=False";
var redisConnection = ConnectionMultiplexer.Connect(connectionString); // needs to be kept alive
IDatabase db = redisConnection.GetDatabase(); // lightweight object, no need to keep

var success = db.StringSet("favorite:flavor", "i-love-rocky-road");
var value = db.StringGet("favorite:flavor");
db.StringSet("userData", "JohnDoe123", TimeSpan.FromHours(1)); // Set TTL

redisConnection.Dispose();
```

### Getting and Setting binary values

- `RedisKey` keys. Seamlessly converts between `string` and `byte[]`.
- `RedisValue` values, also supporting `string` and `byte[]` conversions.
- `RedisResult` holds results, with properties like `Type` and `IsNull`. Retrieve value using `.ToString()`.

```cs
byte[] key = ...;
byte[] value = db.StringGet(key);

db.StringSet(key, value);
```

### [Common operations](https://github.com/StackExchange/StackExchange.Redis/blob/master/src/StackExchange.Redis/Interfaces/IDatabase.cs)

| Method              | Description                                                                                                                  | Return Type    |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------- |
| `CreateBatch`       | Creates a group of operations that will be sent to the server as a single unit, but not necessarily processed as a unit.     | `IBatch`       |
| `CreateTransaction` | Creates a group of operations that will be sent to the server as a single unit and processed on the server as a single unit. | `ITransaction` |
| `KeyDelete`         | Delete the key/value.                                                                                                        | `bool`         |
| `KeyExists`         | Returns whether the given key exists in cache.                                                                               | `bool`         |
| `KeyExpire`         | Sets a time-to-live (TTL) expiration on a key.                                                                               | `bool`         |
| `KeyRename`         | Renames a key.                                                                                                               | `bool`         |
| `KeyTimeToLive`     | Returns the TTL for a key.                                                                                                   | `TimeSpan?`    |
| `KeyType`           | Returns the string representation of the type of the value stored at key: string, list, set, zset, and hash.                 | `RedisType`    |

### Executing commands

```cs
RedisResult result = cache.Execute("ping"); // PONG
```

```cs
RedisResult result = await cache.ExecuteAsync("client", "list"); // This would output all the connected clients
// Type = BulkString
// Result = id=9469 addr=16.183.122.154:54961 fd=18 name=DESKTOP-AAAAAA age=0 idle=0 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=subscribe numops=5
// id=9470 addr=16.183.122.155:54967 fd=13 name=DESKTOP-BBBBBB age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=client numops=17
```

### Storing more complex values

```cs
string serializedValue = Newtonsoft.Json.JsonConvert.SerializeObject(obj);
bool added = db.StringSet("key", serializedValue);

var result = db.StringGet("key");
var stat = Newtonsoft.Json.JsonConvert.DeserializeObject<MyCustomClass>(result.ToString());
```

## Other operations

- `Set-AzRedisCache -ResourceGroupName ResourceGroupName -Name CacheName -Size 2.5GB` - change size
