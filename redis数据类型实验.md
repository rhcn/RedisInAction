**实验说明**：
- 对于以下每个示例，你可以使用 `redis-cli` 命令行工具连接到 Redis 服务器，输入相应的命令进行操作。
- 确保 Redis 服务器正在运行，并且可以通过 `redis-cli` 访问，例如：
    ```bash
    dnf install redis -y
    systemctl start redis
    redis-cli -h localhost -p 6379
    ```
- 你可以在命令行中逐行输入上述命令，观察 Redis 如何存储和操作不同类型的数据。
- 在执行完每个操作后，使用相应的查询命令（如 `GET`、`LRANGE`、`HGETALL` 等）查看操作的结果。


以下是 Redis 每个数据类型的应用场景实践示例，通过命令行进行操作：

**一、字符串（String）**

**应用场景**：存储用户的访问令牌。

**实验步骤**：
1. **存储用户的访问令牌**：
    ```bash
    SET user:1:token "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
    ```
2. **获取用户的访问令牌**：
    ```bash
    GET user:1:token
    ```
3. **设置访问令牌的过期时间（例如，3600 秒）**：
    ```bash
    SETEX user:1:token 3600 "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
    ```


**二、列表（List）**

**应用场景**：实现一个简单的任务队列。

**实验步骤**：
1. **添加任务到队列头部**：
    ```bash
    LPUSH task_queue "Task 1" "Task 2" "Task 3"
    ```
2. **查看任务队列中的所有任务**：
    ```bash
    LRANGE task_queue 0 -1
    ```
3. **从队列尾部取出一个任务（模拟任务执行）**：
    ```bash
    RPOP task_queue
    ```


**三、哈希（Hash）**

**应用场景**：存储用户的详细信息。

**实验步骤**：
1. **存储用户信息**：
    ```bash
    HSET user:2 name "Jane Doe" age 28 email "jane@example.com"
    ```
2. **获取用户的某个属性（例如，年龄）**：
    ```bash
    HGET user:2 age
    ```
3. **获取用户的所有信息**：
    ```bash
    HGETALL user:2
    ```


**四、集合（Set）**

**应用场景**：存储用户的好友列表。

**实验步骤**：
1. **添加好友到用户的好友集合**：
    ```bash
    SADD user:3:friends "Friend1" "Friend2" "Friend3"
    ```
2. **检查某个用户是否是该用户的好友**：
    ```bash
    SISMEMBER user:3:friends "Friend2"
    ```
3. **获取用户的所有好友**：
    ```bash
    SMEMBERS user:3:friends
    ```


**五、有序集合（Sorted Set）**

**应用场景**：实现一个网站的文章点赞排行榜。

**实验步骤**：
1. **添加文章及其点赞数到排行榜**：
    ```bash
    ZADD article:likes 10 "Article1" 20 "Article2" 15 "Article3"
    ```
2. **查看点赞数最多的两篇文章**：
    ```bash
    ZRANGE article:likes 0 1 REV WITHSCORES
    # 查看所有文章及其点赞数
    ZRANGE article:likes 0 -1 REV WITHSCORES
    ```
3. **给某篇文章增加点赞数（例如，Article1 增加 5 个赞）**：
    ```bash
    ZINCRBY article:likes 5 "Article1"
    ```


**六、位图（Bitmap）**

**应用场景**：记录用户的每日签到情况。

**实验步骤**：
1. **记录用户在 2025 年 1 月 1 日的签到情况**：
    ```bash
    SETBIT user:4:checkin:202501 0 1
    ```
2. **检查用户是否在 2025 年 1 月 1 日签到**：
    ```bash
    GETBIT user:4:checkin:202501 0
    ```
3. **统计用户在 2025 年 1 月的签到次数**：
    ```bash
    BITCOUNT user:4:checkin:202501 0 30
    ```


**七、HyperLogLog**

**应用场景**：统计网站的每日独立访客数量。

**实验步骤**：
1. **记录每日的访客**：
    ```bash
    PFADD website:visitors:20250111 visitor1 visitor2 visitor3
    ```
2. **统计 2025 年 1 月 11 日的独立访客数量**：
    ```bash
    PFCOUNT website:visitors:20250111
    ```
3. **合并多天的访客数据（例如，合并 1 月 10 日和 1 月 11 日的数据）**：
    ```bash
    PFADD website:visitors:20250110 visitor1 visitor2 visitor4 visitor5
    PFMERGE website:visitors:20250110-11 website:visitors:20250110 website:visitors:20250111
    # 返回值5，可以用于统计DAU和MAU
    PFCOUNT website:visitors:20250110-11
    ```


通过这些实验，你可以亲身体验 Redis 不同数据类型在各种实际应用场景中的使用方式，以及如何使用相应的 Redis 命令来操作这些数据。这将有助于你更好地理解 Redis 的强大功能和灵活的使用方式，以便在实际开发中根据不同的业务需求选择合适的数据类型和操作命令。
