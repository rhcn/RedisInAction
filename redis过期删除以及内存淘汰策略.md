**一、Redis 删除机制概述**

Redis 使用了多种策略来删除不再需要的数据，主要包括定时删除、惰性删除和定期删除。这些策略共同确保 Redis 能够高效地管理内存，并清除过期的数据，以防止内存泄漏。

**二、过期数据的设置方式**

在 Redis 中，可以通过以下几种方式设置键的过期时间：
- `EXPIRE key seconds`：设置键 `key` 在 `seconds` 秒后过期。
- `EXPIREAT key timestamp`：设置键 `key` 在 `timestamp`（Unix 时间戳）后过期。
- `PEXPIRE key milliseconds`：设置键 `key` 在 `milliseconds` 毫秒后过期。
- `PEXPIREAT key timestamp_ms`：设置键 `key` 在 `timestamp_ms`（Unix 毫秒时间戳）后过期。


**三、删除策略**

**1. 惰性删除（Lazy Expiration）**
- 惰性删除是 Redis 最基本的删除策略之一。当客户端尝试访问一个已经过期的键时，Redis 会检查该键是否过期，如果过期，则删除该键并返回空结果。
- 这种方式的优点是对 CPU 友好，因为只有在访问过期键时才会删除，避免了额外的 CPU 开销。但缺点是过期键可能会在内存中驻留很长时间，直到被访问，会占用额外的内存空间。

**示例代码**：
```bash
# 设置一个键并设置其过期时间
SET mykey "value"
EXPIRE mykey 10
# 等待 10 秒后，尝试获取该键
GET mykey
```
在上述示例中，当 `GET mykey` 操作发生时，如果 `mykey` 已经过期，Redis 会删除该键并返回 `nil`。


**2. 定期删除（Active Expiration）**
- Redis 会定期运行一个后台任务，对部分过期键进行主动检查和删除。该任务会在一定的时间间隔内随机抽取一部分键，检查它们是否过期。如果过期，则删除这些键。
- 这个时间间隔和抽取的键的数量由 Redis 的配置决定，例如 `hz` 参数会影响检查的频率，`active-expire-effort` 会影响检查的比例。
- 优点是可以在一定程度上避免过期键在内存中长时间驻留，平衡了 CPU 消耗和内存使用；缺点是仍然可能存在部分过期键未被及时删除，且会占用一定的 CPU 资源。


**示例代码（伪代码）**：
```python
def active_expiration():
    # 假设 Redis 服务器运行时会定期调用此函数
    keys_to_check = sample_keys()  # 随机抽取一部分键
    for key in keys_to_check:
        if is_expired(key):  # 检查键是否过期
            delete_key(key)  # 删除过期键
```


**3. 定时删除（Volatile-ttl）**
- 对于设置了过期时间的键，Redis 会在内部维护一个按过期时间排序的字典，过期时间最近的键会先被删除。
- 在某些情况下，Redis 会优先删除过期时间最近的键，以释放内存空间，特别是在内存使用达到一定阈值时。


**四、Redis 的内存淘汰策略（Eviction Policies）**

当 Redis 的内存使用量达到 `maxmemory` 限制时，即使有些键尚未过期，Redis 也会根据配置的内存淘汰策略删除一些键以释放内存。以下是一些常见的内存淘汰策略：
- **volatile-lru**：从设置了过期时间的键中，使用 LRU（Least Recently Used）算法删除最近最少使用的键。
- **allkeys-lru**：从所有键中，使用 LRU 算法删除最近最少使用的键。
- **volatile-lfu**：从设置了过期时间的键中，使用 LFU（Least Frequently Used）算法删除最不频繁使用的键。
- **allkeys-lfu**：从所有键中，使用 LFU 算法删除最不频繁使用的键。
- **volatile-random**：从设置了过期时间的键中随机删除键。
- **allkeys-random**：从所有键中随机删除键。
- **volatile-ttl**：从设置了过期时间的键中，删除即将过期的键。
- **noeviction**：不删除任何键，当内存使用达到上限时，新的写操作会导致 Redis 报错。


**示例配置**：
在 `redis.conf` 中设置内存淘汰策略：
```
maxmemory-policy volatile-lru
```


**五、总结**

- **过期键的处理**：
    - 过期的数据键并非立即删除，而是通过惰性删除、定期删除或定时删除的方式进行处理。
    - 惰性删除是在访问时检查并删除，定期删除是通过后台任务随机抽取部分键进行检查和删除，定时删除是在某些条件下优先删除即将过期的键。


- **内存淘汰策略**：
    - 当 Redis 内存达到上限时，会根据内存淘汰策略删除键，以释放内存空间，不同的策略适用于不同的使用场景。


**六、使用建议**

- **配置 `maxmemory` 和内存淘汰策略**：
    - 根据服务器的内存大小和应用需求，合理设置 `maxmemory` 并选择合适的内存淘汰策略。


- **监控内存使用**：
    - 可以使用 `INFO memory` 命令监控 Redis 的内存使用情况：
        ```bash
        redis-cli INFO memory
        ```


- **性能优化**：
    - 考虑调整 `hz` 和 `active-expire-effort` 参数，以优化定期删除的性能，但要注意平衡 CPU 开销和内存占用。


通过这些删除和淘汰策略，Redis 可以在内存管理和性能之间取得平衡，有效地处理过期键和内存使用问题，确保 Redis 服务的高效运行。在实际应用中，根据业务需求和服务器资源，合理选择和调整 Redis 的删除和淘汰策略至关重要。
