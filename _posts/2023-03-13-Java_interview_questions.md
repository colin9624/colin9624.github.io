---
layout: mypost
title: Java interview questions
categories: [interview, Java]
---

# Collation of interview questions

## Accumulation point

1. Introduce the modules you are responsible for in the project.

2. The principle of Single Sign On, what information does your jwt contain?？
   * Single Sign On
   * Understand: Single login, multiple visits（In a system with multiple subsystems, you only need to log in to one subsystem, and then you do not need to log in again when you access other subsystems）
   * Process flow (Based on Cookie):
     * If the user visits system A for the first time, and system A discovers that the user is not logged in, it will be redirected to the SSO authentication center and carry the request url for login verification
     * The user verifies the user name and password at the SSO authentication center. After the login is successful, the server generates a `ticket`, then redirects to the source url of system A and appends the `ticket` to the url parameter.
     * System A gets the ticket in the url parameter and initiates ticket verification to SSO. If the verification is successful, system A releases it and stores the `ticket` in cookie.
     * When the user accesses system B, the `ticket` is already carried under the domain of system B, and `ticket` verification is initiated directly to SSO. If the verification is successful, the `ticket` is released and stored in cookie (update the expiration time of `ticket`)
     * Remove the cookie under domain when the user logs out ![01](01.png)
   * jwt：username, userid, expiration time.

3. When introducing a third-party login, how do you associate your own token with a third-party token?
   * Steps
     * Website A allows users to jump to GitHub
     * GitHub asks the user to log in, and then asks, "do you agree that website A asks for xx permission?"
     * If the user agrees, GitHub will redirect back to website A and send back an authorization code.
     * Website A uses authorization codes to request tokens from GitHub
     * GitHub returns token
     * Website A uses tokens to request user data from GitHub

4. Feign客户端的远程调用是怎么实现的?协议是什么？Hystrics熔断保护的实现原理？

5. What are the modes of Redis?

   * Stand-alone mode

     * pros:
       * Simple deployment, 0 cost
       * Low cost, no backup nodes, no other expenses
       * High performance, single machine does not need to synchronize data, natural data consistency
     * cons:
       * The reliability guarantee is not very good, and a single node has the risk of downtime.
       * The high performance of single machine is limited by the processing power of CPU, and redis is single-threaded.

   * Master-slave mode
     * Master-slave replication refers to copying the data from one Redis server to another Redis server.
       The former is called master node (master) and the latter is called slave node (slave); data replication is one-way and can only be from master node to slave node.

     * Advantages:
       * Once the master node goes down, the backup of the slave node as the master node can replace its work at any time.
       * Expand the reading ability of the master node, share the reading pressure of the master node, and support the separation of reading and writing.
       * High availability cornerstone: in addition to the above functions, master-slave replication is also the basis on which Sentinel mode and cluster mode can be implemented, so master-slave replication is the cornerstone of Redis high availability.
       * Slave can also accept connection and synchronization requests from other Slaves, which can effectively offload the synchronization pressure of Master.
       * Master Server provides services to Slaves in a non-blocking manner. So during Master-Slave synchronization, the client can still submit queries or modify requests
       * Slave Server also completes data synchronization in a non-blocking manner. During synchronization, if a client submits a query request, Redis returns the data before synchronization

     * Disadvantages:

       * Once the master node goes down, the slave node is promoted to the master node, and the address of the master node of the application side needs to be modified, and all slave nodes need to be ordered to copy the new master node. The whole process requires human intervention.
       * The writing ability of the master node is limited by a single machine.
       * The storage capacity of the master node is limited by a single machine.
       * If multiple Slave are disconnected, when you need to restart, try not to restart at the same time. Because as long as Slave starts, a sync request will be sent to fully synchronize with the host, and when multiple Slave restarts, it may cause a sharp increase in Master IO and lead to downtime

   * Sentinel mode

     * How it works: each Sentinel sends a PING command to all of its **master servers**, **slave servers** and other sentinel **instances** at a frequency of once per second.
       If an **instance** takes longer than the value specified by down-after-milliseconds from the last valid reply to the PING command, the instance will be marked as **subjective offline** by Sentinel.

       If a **master server** is marked as **subjective offline**, then all **Sentinel** nodes of the master server are being monitored to confirm whether the master server has indeed entered the **subjective offline** state at the frequency of **once per second**.

       If a master server is marked as subjectively offline and a sufficient number of Sentinel (at least up to the number specified in the configuration file) agrees with this judgment within the specified **time range**, then the master server is marked as **objective offline**.

       In general, each Sentinel sends INFO commands to all master and slave servers it knows about at a frequency of once every 10 seconds.

       When a **master server** is marked by Sentinel as **objective offline**, the frequency of Sentinel sending INFO commands to all slave servers of the offline master server will be changed from once per second to once per second.

       Sentinel和其他 Sentinel 协商 **主节点** 的状态，如果 主节点处于 **SDOWN`\**状态，则投票\**自动选出**新的主节点。将剩余的 **从节点** 指向 **新的主节点** 进行 **数据复制**。

       当没有足够数量的 Sentinel 同意 主服务器 下线时， 主服务器 的 **客观下线状态** 就会被移除。

       当 **主服务器** 重新向 Sentinel的PING命令返回 有效回复 时，主服务器 的 **主观下线状态** 就会被移除。

       ![02](02.jpg)

     * 功能：

       	* 监控(Monitoring)：哨兵会不断地检查主节点和从节点是否运作正常。
       	* 自动故障转移(Automatic failover)：当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点。
       	* 配置提供者(Configuration provider)：客户端在初始化时，通过连接哨兵来获得当前Redis服务的主节点地址。
       	* 通知(Notification)：哨兵可以将故障转移的结果发送给客户端。

     * 优点：

       * 哨兵模式是基于主从模式的，所有主从的优点，哨兵模式都具有
       * 主从可以自动切换，系统更健壮，可用性更高
       * Sentinel会不断地检查主服务器和从服务器是否正常运行。当被监控的某个Redis服务器出现问题，Sentinel通过API脚本向管理员或者其他的应用程序发送通知

     * 缺点：

       * Redis较难支持在线扩容，对于集群，容量达到上限时在线扩容会变得很复杂

   * Cluster mode

     * Redis Cluster 集群模式具有 **高可用**、**可扩展性**、**分布式**、**容错** 等特性
     * 原理：通过数据分片的方式来进行数据共享问题，同时提供数据复制和故障转移功能。之前的两种模式数据都是在一个节点上的，单个节点存储是存在上限的。集群模式就是把数据进行分片存储，当一个分片数据达到上限的时候，就分成多个分片
     * 所有的 redis 节点彼此互联(PING-PONG机制)，内部使用二进制协议优化传输速度和带宽
     * 节点的 fail 是通过集群中超过半数的节点检测失效时才生效
     * 客户端与 Redis 节点直连，不需要中间代理层.客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可

6. Redis的击穿、雪崩和穿透讲一下。

   * 缓存雪崩：当某一时刻发生大规模的缓存失效的情况，会有大量的请求进来直接打到DB上面
   * 解决方案：
     * key同时过期，设置随机失效时间
     * 设置热点数据不设置过期时间，有更新操作就更新缓存
     * 使用集群缓存，保证缓存服务的高可用
     * 开启Redis持久化机制，尽快恢复缓存集群，一旦重启，就能从磁盘上自动加载数据恢复内存中的数据
   * 缓存穿透：用户不断发起请求缓存和数据库中都没有的数据。
     * 举例：我们数据库的 id 都是1开始自增上去的，如发起为id值为 -1 的数据或 id 为特别大不存在的数据。每次都能绕开Redis直接打到数据库，数据库也查不到，每次都这样，并发高点就容易崩掉了
   * 解决方案：
     * 接口层增加校验（比如用户鉴权校验，参数做校验，不合法的参数直接代码Return，比如：id 做基础校验，id <=0的直接拦截等）
     * 缓存空值（之所以会发生穿透，就是因为缓存中没有存储这些空数据的key。从而导致每次查询都到数据库去了。那么我们就可以为这些key对应的值设置为null 丢到缓存里面去。后面再出现查询这个key 的请求的时候，直接返回null）
     * 布隆过滤器（Bloom Filter）（Bloom Filter是用于判断某个元素（key）是否存在于某个集合中。先把我们数据库的数据都加载到我们的过滤器中，在缓存之前在加一层BloomFilter，在查询的时候先去 BloomFilter 去查询 key 是否存在，如果不存在就直接返回，存在再走查缓存，然后查DB）
     * Nginx限制（对单个IP每秒访问次数超出阈值的IP都拉黑）
   * 缓存击穿：缓存击穿是指一个Key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个Key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个完好无损的桶上凿开了一个洞
   * 解决方案：
     * 设置热点数据永远不过期
     * 互斥锁：我们可以在第一个查询数据的请求上使用一个 互斥锁来锁住它。其他的线程走到这一步拿不到锁就等着，等第一个线程查询到了数据，然后做缓存。后面的线程进来发现已经有缓存了，就直接走缓存。

7. Redis中rdb、aof区别、效率

   * 介绍：由于Redis的数据都存放在内存中，如果没有配置持久化，redis重启后数据就全丢失了，于是需要开启redis的持久化功能，将数据保存到磁 盘上，当redis重启后，可以从磁盘中恢复数据
   * RDB持久化
     * 原理：将Reids在内存中的数据库记录定时 dump到磁盘上的RDB持久化
     * RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储
     * 优势：
       * RDB备份只会有一个文件，例如设置每小时归档最近24小时的文件，每天归档30天内的数据，这种方式在系统出现灾难性故障时，容易恢复
       * 文件容易压缩转移到其他地方
       * 性能最大，在进行RDB持久化时，Redis服务之需要fork一个子进程，之后再由子进程完成持久化相关操作，避免了服务器进程执行IO导致性能损耗
       * 对比AOF机制，如果数据集大，RDB启动效率高
     * 劣势：
       * 没有办法保证最大限度的数据安全性，存在着系统在执行定时持久化之前出现宕机时，没有写入磁盘的部分数据会丢失
       * 因为是定时任务，当需要持久化的数据集较大时，可能会影响或停止服务器(短时间：几百毫秒)。建议一般不要让RDB的间隔太长，否则每次生成的RDB文件太大了，对Redis本身的性能也会有影响
   * AOF(append only file)持久化
     * 原理：将Redis的操作日志以追加的方式写入文件
     * AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录
     * 优势：
       * 相比于RDB有着更高的数据安全性。
       * AOF采用的是append追加写入模式，即使在写入过程中宕机，也不会破坏日志文件汇总已经存在的内容(出现该问题后，可以在下一次启动redis之前通过redis-check-aof工具来解决数据一致性问题)
       * 如果日志过大，Redis 可以自动启用 rewrite 机制（后台重写，不影响客户端的读写）。即 Redis 以 append 模式不断的将修改数据写入到老的磁盘文件中，同时 Redis 还会创建一个新的文件用于记录此期间有哪些修改命令被执行。因此在进行 rewrite 切换时可以更好的保证数据安全性
       * AOF 日志文件的命令通过非常可读的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复(比如某人不小心用 flushall 命令清空了所有数据，只要这个时候后台 rewrite 还没有发生，那么就可以立即拷贝 AOF 文件，将最后一条 flushall 命令给删了，然后再将该 AOF 文件放回去，就可以通过恢复机制，自动恢复所有数据)
     * 劣势：
       * 同量数据，AOF文件更大
       * AOF数据恢复速度比较慢
       * 定期的备份需要自己手写脚本去做，做冷备份不太合适
       * AOF开启后，支持的写QPS会比RDB更低，因为要将命令写到AOF缓冲区，此外还要定时fsync一次日志文件
       * AOF相比于RDB这种完整数据快照的方式更脆弱，可能会出bug。不过 AOF 就是为了避免 rewrite 过程导致的 bug，因此每次 rewrite 并不是基于旧的指令日志进行 merge 的，而是基于当时内存中的数据进行指令的重新构建，这样健壮性会好很多
   * 实际应用选择标准：
     * 仅使用RDB会导致数据丢失问题
     * 仅使用AOF做冷备，宕机后恢复数据效率慢
     * 在复杂场景中建议综和使用，AOF作为第一选择，保证数据完整性；RDB做不同形式的冷备

8. Redis集群如何扩容

   * 步骤：准备新节点，加入集群，迁移槽和数据

     ```shell
     # 设置新节点参数
     port 6385                               //端口
     cluster-enabled yes                     //开启集群模式
     cluster-config-file nodes-6385.conf     //集群内部的配置文件
     cluster-node-timeout 15000              //节点超时时间，单位毫秒
     // 其他配置和单机模式相同
     
     # 开启一个该节点Redis服务
     $ sudo redis-server conf/redis-6385.conf
     # 进入原主节点将目标节点加入到集群中
     127.0.0.1:6379> cluster meet 127.0.0.1 6385
     OK
     127.0.0.1:6379> cluster nodes
     cb987394a3acc7a5e606c72e61174b48e437cedb 127.0.0.1:6385 master - 0 1496731333689 8 connected
     # 此时新加入的节点都是主节点，因为没有负责槽位，所以不能接受任何读写操作
     
     # 迁移槽和数据
     # 首先要为新节点指定槽的迁移计划，确保迁移后每个节点负责相似数量的槽，从而保证这些节点的数据均匀
     cluster reshard 127.0.0.1 6379
     ```

9. 使用Redis缓存如保障证与数据库一致性（延时双删、过期时间、binlog、分段加锁）

    * 定义

      	* 若缓存中有数据，则缓存的数据值需要和DB相同
      	* 若缓存无数据，则DB值必须是最新值

    * 类型

       * 读写缓存：对数据进行操作，在缓存中进行；同时根据采取的不同写会策略，决定是否同步写回DB

         	* 同步直写策略：写缓存时，也同步写数据库，缓存和数据库中的数据一致；使用此方法需要给操作添加事务，保障操作原子性
         	* 异步写回策略：写缓存时不同步写数据库，等到数据从缓存中淘汰时，在写回数据库；使用该操作时遇到缓存发生故障时数据库中就会没有最新数据

       * 只读缓存：数据新增，直接写入数据库；数据删改，将缓存中的数据表级为无效

          * 删改数据分析

             * 重试机制(仅应对两次操作无法保证原子性问题)：具体来说，可以把要删除的缓存值或者是要更新的数据库值暂存到消息队列中（例如使用Kafka消息队列）。当应用没有能够成功地删除缓存值或者是更新数据库值时，可以从消息队列中重新读取这些值，然后再次进行删除或更新

             * 两个操作第一次执行时都没有失败，当有大量并发请求时，应用还是有可能读到不一致的数据：

                * 类型一(先删除缓存，再更新数据库)：线程A删除缓存后，没有来得及操作数据库，线程B开始操作了

                   * 问题一：线程B读取到了旧值

                   * 问题二：线程B在缓存缺失的情况下，将数据库旧值写入缓存，造成后续读取操作获取的是错误的旧值

                   * 解决方法：延时双删

                     ```java
                     redis.delKey()	// 先删除缓存
                     db.update()			// 再写数据库
                     Thread.sleep()	// 休眠xx毫秒(根据业务时间来定)
                     redis.delKey()  // 再次删除缓存
                     ```

               * 类型二(先更新数据库值，再删除缓存值)：线程A删除了数据库中的值，没来得及删除缓存值，线程B就开始读取数据了

    * 常见一致性问题举例举例(数据更新问题)

       * 如果删除了缓存Redis，还没有来得及写库MySQL，另一个线程就来读取，发现缓存为空，则去数据库中读取数据写入缓存，此时缓存中为脏数据
       * 如果先写了库，在删除缓存前，写库的线程宕机了，没有删除掉缓存，则也会出现数据不一致情况

    * 延时双删

    * 设置缓存过期时间：理论上，设置缓存过期时间，是保证最终一致性的解决方案。所有的写操作以DB为准，只要到达缓存过期时间，则后面的读请求自然会从DB读取新值，然后回填缓存

    * binlog

       * 更新数据库数据
      * 数据库会将操作信息写入binlog日志当中
      * 订阅程序提取出所需要的数据以及key
      * 另起一段非业务代码，获得该信息
      * 尝试删除缓存操作，发现删除失败
      * 将这些信息发送至消息队列
      * 重新从消息队列中获得该数据，重试操作

   * 分布式锁：

     * Redisson实现分布式锁：[参考链接](https://www.cnblogs.com/yufeng218/p/13733205.html)

       ![03](03.png)

     * Redis实现简单分布式锁

       ```java
       String lockKey = "intelfaceName";
       // 随机线程id，保障只有本线程可以删锁
       String lockValue = UUID.randomUUID().toString();
       try {
         // 设置过期时间，防止死锁
         Boolean isLock = stringRedisTemplate.opsForVaule().setIfAbsent(lockKey, clientId, 30, TimeUnit.SECONDS);
       	if (!isLock) { return "error"; }
       	// 业务代码
       } finally {
         // 请求逻辑完成后删除lockKey(只能本线程删除，保证高并发下安全)
         if (clientId.equals(stringRedisTemplate.opsForVaule().get(lockValue))) {
           stringRedisTemplate.delete(lockKey);
         }
       }
       return "success";
       
       // 存在问题：存在过期时间问题，在业务代码中执行定时任务对锁进行续期可解决(使用Redisson框架)
       ```

       

   * 分段加锁：是对分布式锁的优化(分布式锁性能问题：高并发时只有第一个加锁成功的线程可以执行业务逻辑，其他线程等待)

     * 如果某个下单请求，咔嚓加锁，然后发现这个分段库存里的库存不足了，此时咋办？这时你得自动释放锁，然后立马换下一个分段库存，再次尝试加锁后尝试处理。这个过程一定要实现，避免分段库存不足而又无限持有锁导致其他用户无限期阻塞
     * 如果某个时刻，分段库存对于单个用户下单库存而言不足，但是总库存却大于所有请求下单库存量的和时，该如何处理这种有库存也无法下单的场景？(要解决这种场景问题，实现上比较麻烦)，但是就秒杀场景而言，出现几率较少，因为这种对用户下单的数量有限制，如每个用户最多购买一件商品
     * 实现成本高昂：
       * 首先，你得对一个数据分段存储，一个库存字段本来好好的，现在要分为20个分段库存字段
       * 其次，你在每次处理库存的时候，还得自己写挑选算法，挑选一个分段来处理
       * 最后，如果某个分段中的数据不足了，你还得自动切换到下一个分段数据去处理

     ![04](04.png)

10. Redis的哨兵机制，监控集群的状态：参考上文

11. 讲一下Servlet是什么东西？

     * Server Applet(运行在服务端的小程序)
     * Servlet只是狭义上一个接口(规范)，广义上指任何实现了这个Servlet的类
     * servlet主要用来扩展基于HTTP协议的Web服务器
     * 过程：
       * 客户端请求servlet
       * 加载Servlet到内存
       * 实例化并调用init()方法初始化该Servlet
       * 直行业务(doGet()、doPsot()、doHead()、doPut()、doTrace()、doDelete()、doOptions())

12. Spring和springboot的区别？

     * Spring是一款Java开发框架，胶水框架，解决了Java企业级开发的复杂度问题，有效降低耦合度；Spring框架有两大核心AOP、IOC，有效地管理了开发对象生命周期的问题；Spring能够于绝大多数框架整合到一起。
     * SpringBoot是Spring框架的扩展
       * 创建独立的Spring应用
       * 内置Tomcat服务器，支持通过jar包部署web应用
       * 提供了starter来简化Maven配置，解决依赖问题
       * 采用约定大于配置的思想，尽可能地自动配置spring应用，解决配置复杂度问题
       * 没有代码生成和XML配置要求

13. spring有哪些常用注解？

     * 组件类：@Component、@Controller、@Repository、@Service

     * 装配Bean时常用注解：

       * @Autowired
       * @Qualifier：此注解是和@Autowired一起使用的。使用此注解可以让你对注入的过程有更多的控制。@Qualifier可以被用在单个构造器或者方法的参数上。当上下文有几个相同类型的bean, 使用@Autowired则无法区分要绑定的bean，此时可以使用@Qualifier来指定名称。
       * @Resource(不属于spring的注解，而是来自于JSR-250位于java.annotation包下，使用该annotation为目标bean指定协作者Bean)、@Inject(由JSR-330提供)

     * 配置类常用注解：

       * @Configuration：注解在类上，声明当前类为配置类，相当于xml形式的Spring配置，其中内部组合了@Component注解，表明这个类是一个bean
       * @Bean 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式
       * @ComponentScan 用于对Component进行扫描
       * @WishlyConfiguration 为@Configuration与@ComponentScan的组合注解，可以替代这两个注解

      * AOP切面常用注解：

        * @Aspect 声明切面
          	* @After 后置
           	* @Before 前置
          * @Around 环绕
          * @PointCut 声明切点
          * 在java配置类中使用@EnableAspectJAutoProxy注解开启SpringduiAspectJ代理支持

      * @Bean的支持

        * @Scope 设置Spring容器如何新建Bean实例（方法上，得有@Bean），其设置类型包括：

          	* Singleton （单例,一个Spring容器中只有一个bean实例，默认模式）
          	* Protetype （每次调用新建一个bean）
          	* Request （web项目中，给每个http request新建一个bean）
          	* Session （web项目中，给每个http session新建一个bean）
          	* GlobalSession（给每一个 global http session新建一个Bean实例）

          @StepScope 在Spring Batch中还有涉及

          @PostConstruct 由JSR-250提供，在构造函数执行完之后执行，等价于xml配置文件中bean的initMethod

          @PreDestory 由JSR-250提供，在Bean销毁之前执行，等价于xml配置文件中bean的destroyMethod

     * 属性值与setter注解

       * @Value注解：
         * 注入普通字符：@Value("Michael Jackson")String name
         * 注入操作系统属性：@Value("#{systemProperties['os.name']}")String osName
         * 注入表达式结果：@Value("#{ T(java.lang.Math).random() * 100 }") String randomNumber
         * 注入其它bean属性：@Value("#{domeClass.name}")String name
         * 注入文件资源：@Value("classpath:com/hgs/hello/test.txt")String Resource file
         * 注入网站资源：@Value("http://www.github.cn")Resource url
         * 注入配置文件：@Value("${book.name}")String bookName
           * 注入配置使用方法：
             	1. 编写配置文件（test.properties）book.name=《三体》
              	2. @PropertySource 加载配置文件(类上)，@PropertySource("classpath:com/hgs/hello/test/test.propertie")
              	3. 还需配置一个PropertySourcesPlaceholderConfigurer的bean
        * @Required：此注解用于bean的setter方法上。表示此属性是必须的，必须在配置阶段注入，否则会抛出BeanInitializationExcepion

      * 环境切换：

        * @Profile：通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。（类或方法上）
        * @Conditional：Spring4中可以使用此注解定义条件话的bean，通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。（方法上）

      * 异步相关：

        * @EnableAsync：配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）
        * @Async 在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上_所有的方法都将异步_，需要@EnableAsync开启异步任务）

      * 定时任务相关：

        * @EnableScheduling 在配置类上使用，开启计划任务的支持（类上）
        * @Scheduled 来申明这是一个任务，包括cron,fixDelay,fixRate等类型（方法上，需先开启计划任务的支持）

      * Enable功能支持开启

        * @EnableAspectJAutoProxy 开启对AspectJ自动代理的支持
        * @EnableAsync 开启异步方法的支持
        * @EnableScheduling 开启计划任务的支持
        * @EnableWebMvc 开启Web MVC的配置支持
        * @EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持
        * @EnableJpaRepositories 开启对SpringData JPA Repository的支持
        * @EnableTransactionManagement 开启注解式事务的支持
        * @EnableCaching 开启注解式的缓存支持

      * 测试相关注解

        * @RunWith 运行器，Spring中通常用于对JUnit的支持
        * @RunWith(SpringJUnit4ClassRunner.class)
        * @ContextConfiguration 用来加载配置ApplicationContext，其中classes属性用来加载配置类
        * @ContextConfiguration(classes={TestConfig.class})

      * SpringMVC相关注解

        * @EnableWebMvc 在配置类中开启Web MVC的配置支持，如一些ViewResolver或者MessageConverter等，若无此句，重写WebMvcConfigurerAdapter方法（用于对SpringMVC的配置）
        * @Controller 声明该类为SpringMVC中的Controller
        * @RequestMapping 用于映射Web请求，包括访问路径和参数（类或方法上）
        * @ResponseBody 支持将返回值放在response内，而不是一个页面，通常用户返回json数据（返回值旁或方法上）
        * @RequestBody 允许request的参数在request体中，而不是在直接连接在地址后面（放在参数前）
        * @PathVariable 用于接收路径参数，比如@RequestMapping(“/hello/{name}”)申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法
        * @RestController 该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody
        * @ControllerAdvice 通过该注解，我们可以将对于控制器的全局配置放置在同一个位置，注解了@Controller的类的方法可使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上
          这对所有注解了 @RequestMapping的控制器内的方法有效
        * @ExceptionHandler 用于全局处理控制器里的异常
        * @CookieValue：此注解用在@RequestMapping声明的方法的参数上，可以把HTTP cookie中相应名称的cookie绑定上去
        * @CrossOrigin：此注解用在class和method上用来支持跨域请求
        * @MatrixVariable：此注解使用在请求handler方法的参数上，Spring可以注入matrix url中相关的值。这里的矩阵变量可以出现在url中的任何地方，变量之间用;分隔
        * @InitBinder 用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中
        * @ModelAttribute 本来的作用是绑定键值对到Model里，在@ControllerAdvice中是让全局的@RequestMapping都能获得在此处设置的键值对

      * SpringBoot相关注解

        * @EnableAutoConfiguration：主应用class，告诉Spring Boot自动基于当前包添加Bean、对bean的属性进行设置等
        * @SpringBootApplicaiton：主应用class，@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan这三个注解的组合
        * @ConfigurationProperties：用来加载额外的配置（如 .properties 文件，可用在@Configuration注解类，或者@Bean注解方法上面
        * @AutoConfigureAfter：用在自动配置类上面，表示该自动配置类需要在另外指定的自动配置类配置完之后
        * @AutoConfigureBefore：用在自动配置类上面，表示该自动配置类需要在另外指定的自动配置类配置完之前

14. spring事务失效：

     * 一般出现在自调用，原因是动态代理在调用自己方法的时候没有事务切面，没有拿到注解
       * 解决方案：
         * 在调用前在从spring工厂获取一次
         * [在事务中开启事务](https://mp.weixin.qq.com/s/1TEBnmWynN4nwc6Q-oZfvw)
     * 没有被Spring管理(例如没有加@Service注解来注入bean)
     * 方法不是public(使用AspectJ代理模式好像可以私有方法，来自官方说明)
     * 数据源配置配置事务管理器
     * 被异常吃掉，没有抛出异常进行回滚，或没有抛出指定类型回滚(事务默认回滚：RuntimeException)，如需其他异常出发回滚，需指定@Transactional(rollbackFor = Exception.class)(仅限于Throwable异常类及其子类)

15. AspectJ了解

     * 与SpringAOP的区别
       * SpringAOP只能用于由Spring管理的Bean容器内
       * AsprctJ提供了完整的AOP解决方案，健壮、复杂；可以在所有对象域中使用
       * 举例：在项目中使用AOP来记录日志访问信息

16. Spring IOC创建对象(bean)的作用域有哪些？

     * 类型：
       * 单例(singleton)(默认)：
         * 在Spring IOC容器中仅存在一个Bean实例，Bean以单例方式存在
         * 会在启动容器时实例化，可以配置lazy-init为true来实现第一次获取bean时再实例化(此时线程不安全)
       * 多例(prototype)：每次从容器中调用，都会返回一个新的实例，即每次调用getBean()时，相当于执行new XxxBean()
       * 请求(request)：每次HTTp请求都会创建一个新的Bean，该作用域仅适用于webApplicaiotnContext环境
       * 会话(session)：同一个HTTP Session共享一个Bean，不同Session使用不同Bean，仅适用于WebApplicationContext环境
       * 全局会话(global session)：一般用于Protlet环境，仅适用于WebApplicationContext环境

17. Spring如何去解决默认创建单例对象产生的线程安全问题？

      * 在Controller中使用ThreadLocal变量：ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景
      * ThreadLoacl与线程同步机制比较：
         * 同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂问题，程序设计和编写难度相对较大
         * ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突
         * 多线程下资源共享的问题，线程同步机制采用了以时间换空间的方式，而ThreadLocal采用了以空间换时间的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响
         * ThreadLocal是解决线程安全问题一个很好的思路，它通过为每个线程提供一份独立的变量副本解决了变量并发访问的冲突问题。在很多情况下，ThreadLocal比直接使用synchronized同步机制解决线程安全问题更简单、更方便，且结果程序拥有更高的并发性
      * 设置多例模式
      * 使用方法的参数局部变量
      * 判断已经创建好的单例对象是否安全(以下两个条件都满足则为线程不安全)
        * 看这个单例有没有全局变量(全局变量就是成员变量，成员变量又分为实例变量和静态变量)
        * 如果有全局变量，看他是不是只可以读取而不能写入(有没有发布set方法)
     * 有状态bean与无状态bean

18. Spring中使用全局变量需要注意的问题：参考上文

19. Spring后置处理器(BeanPostProcessor)出发点

      * 作用：在Bean对象实例化和依赖注入完毕后，在显示调用初始化方法的前后添加我们自己的逻辑。注意是Bean实例化完毕后及依赖注入完成后触发的
      * 方法：
         * postProcessBeforeInitialization：实例化、依赖注入完毕，在调用显示的初始化之前完成一些定制的初始化任务
         * postProcessAfterInitialization：实例化、依赖注入、初始化完毕时执行
      * InstantiationAwareBeanPostProcessor(继承BeanPostProcessor接口)

 20. Spring生命周期

     * 实例化(Instantiation)
     * 属性赋值(Populate)
     * 初始化(Initialization)
     * 销毁(Destruction)

21. 项目中的搜索功能是怎么实现的？ElasticSearch+kibana  ik分词； 

22. MQ怎么知道消息被指定的消费者消费？怎么使不同的生产者生产的消息被不同的消费者消费？

23. 讲一下为什么JVM要分为堆、方法区等？原理是什么？

24. GC算法都有哪些？他们之间的区别是什么？各自的适用场景？

25. String有最大长度限制吗？

     * 有，JVM对常量池有限制，String常量为u2，即2^16-1=65535
     * 编译期的限制：字符串的UTF8编码值的字节数不能超过65535，字符串的长度不能超过65534
     * 运行时限制：字符串的长度不能超过2^31-1(即int最大数值)，占用的内存数不能超过虚拟机能够提供的最大值

26. mysql对哪些建立索引？调优？聚集索引与非聚合索引

27. mq中一条消息出现了异常，怎么处理

28. elasticsearch的主从、字符串类型是哪个、nested类型是什么、聚合怎么写、查询某个id的语句、创建es的索引、时间类型怎么存的

29. int a = 1;jvm如何取得a的值

30. 知道哪些设计模式？

31. mq1000个消息始终不被消费怎么处理

32. 捕获异常在catch块里一定会进入finally吗？catch里能return吗？catch里return还会进finally吗？在try里return呢是什么情况？

33. jvm调优调的哪些参数？我说初始堆大小和最大堆大小一样，问这样有什么好处？在哪里写这些参数

34. 知道哪些锁？公平锁和非公平锁区别、底层实现？可重入锁是什么？

35. mq重复消费：重复消费一般是mq发生了在平衡，查看心跳时间，请求等待时间，调低拉取数量

36. String、StringBuilder、StringBuffer：

      ```
      线程安全性：
      线程安全 String 、StringBuffer
      非线程安全 StringBuilder
      
      执行效率：
      StringBuilder > StringBuffer > String
      
      存储空间：
      String 的值是不可变的，每次对String的操作都会生成新的String对象，
      效率低耗费大量内存空间，从而引起GC。
      StringBuffer和StringBuilder都是可变。
      
      使用场景：
      1如果要操作少量的数据用 String 
      2.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder 
      3.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer
      ```

      

37. jdk1.8 hashMap内部存储结构：链表和红黑树，链表长度超过8时自动转为红黑树

      ```
      1.内部存储结构：数组+链表+红黑树（JDK8）
      2.默认容量16，默认装载因子0.75。
      3.key和value对数据类型的要求都是泛型。
      4.key可以为null，放在table[0]中。
      5.hashcode：计算键的hashcode作为存储键信息的数组下标用于查找键对象的存储位置。equals：HashMap使用equals()判断当前的键是否与表中存在的键相同。
      为什噩梦扩容2倍
      ```

38. jvm的内存模型包括以下几部分：
      **方法区**：是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据
      **Java堆**：堆用来存储几乎所有对象、数组等都在此分配内存，在jvm内存中占的比很大，也是GC垃圾回收的主要地点
      **Java栈**：当jvm在执行方法时，会在此区域中创建一个栈帧来存放方法的信息，比如返回值，局部变量表和各种对象引用等，方法开始执行前就先创建栈帧入栈，执行完后就出栈
      **本地方法栈**：与Java栈类似，Java栈为虚拟机执行Java 方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native 方法服务
      **程序计数器**：占用很小的一片区域，我们知道JVM执行代码是一行一行执行字节码，所以需要一个计数器来记录当前执行的行数
      运行时常量池：是方法区的一部分，用于存放编译期生成的各种字面量和符号引用，在类加载后存放到方法区的运行时常量池中

      ![](https://img-blog.csdnimg.cn/20200903215850454.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p4eV9nbzE=,size_16,color_FFFFFF,t_70#pic_center)

39. jvm中Java堆上，分代的机制是怎么划分的？

      * 新生代： 新生成的对象优先存放在新生代中，新生代对象朝生夕死，存活率很低，在新生代中，常规应用进行一次垃圾收集一般可以回收70% ~ 95% 的空间，回收效率很高，新生代划分为三块，一块较大的Eden空间和两块较小的Survivor空间，采用复制算法来回收新生代
      * 老生代：在新生代中经历了多次GC后仍然存活下来的对象会进入老年代中，老年代中的对象生命周期较长，存活率比较高，在老年代中进行GC的频率相对而言较低，而且回收的速度也比较慢
      * 永久代：永久代储存类信息，常量，静态变量，即时编译器后的代码等数据，对这一区域而言，Java虚拟机规范指出可以不进行垃圾收集，一般而言不会进行垃圾回收

40. Java堆中新生代的垃圾回收机制：
      HotSpot将新生代划分为三块，一块较大的Eden空间和两块较小的Survivor空间，默认比例为8：1：1。划分的目的是因为HotSpot采用复制算法来回收新生代，设置这个比例是为了充分利用内存空间，减少浪费。新生成的对象在Eden区分配（大对象除外，大对象直接进入老年代），当Eden区没有足够的空间进行分配时，虚拟机将发起一次Minor GC。
      GC开始时，对象只会存在于Eden区和From Survivor区，To Survivor区是空的（作为保留区域）。GC进行时，Eden区中所有存活的对象都会被复制到To Survivor区，而在From Survivor区中，仍存活的对象会根据它们的年龄值决定去向，年龄值达到年龄阀值（默认为15，新生代中的对象每熬过一轮垃圾回收，年龄值就加1，GC分代年龄存储在对象的header中）的对象会被移到老年代中，没有达到阀值的对象会被复制到To Survivor区。接着清空Eden区和From Survivor区，新生代中存活的对象都在To Survivor区。接着， From Survivor区和To Survivor区会交换它们的角色，也就是新的To Survivor区就是上次GC清空的From Survivor区，新的From Survivor区就是上次GC的To Survivor区，总之，不管怎样都会保证To Survivor区在一轮GC后是空的。GC时当To Survivor区没有足够的空间存放上一次新生代收集下来的存活对象时，需要依赖老年代进行分配担保，将这些对象存放在老年代中。

41. CMS垃圾回收器优缺点：

      * 类型（串行/并行/并发）：并发，几乎不会暂停用户线程
      * 回收算法：标记清除
      * 适用场景：多cpu且与用户线程共存
      * 优点：并发收集、低停顿
      * 缺点：
        * CPU资源非常敏感：在并发阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致引用程序变慢，总吞吐量下降
        * 无法处理浮动垃圾：
        * 空间碎片：

42. G1垃圾回收器优缺点：

      缺点

      1、region 大小和大对象很难保证一致，这会导致空间的浪费；特别大的对象是可能占用超过一个 region 的。并且，region 太小不合适，会令你在分配大对象时更难找到连续空间，这是一个长久存在的情况。
      2、因为CSet、Rsets、CardTable等数据的存在，会比较耗费内存。

      优点
      1、可根据用户设置停顿时间，制定回收计划(但是也可能存在超出用户的停顿时间).  --- 最主要的目标
      2、局部上来看是基于“复制”算法实现的，无磁盘碎片。
      3、对GC停顿可以做更好的预测

      总结

      G1最大的特性/优点，就是对停顿时间的处理，一方面通过统计数据结构（会耗费更多内存）、参数配置实现控制和预测停顿时间，另一方面对于大内存的回收采取分批回收的方式，降低单次GC的停顿时间，避免影响主业务。

      G1怎么实现的回收时间可控
      1）追踪每一个Region的回收价值，所谓回收价值就是根据设定的预期系统停顿时间，来选择最少回收时间和最多回收对象的Region进行垃圾回收，保证每次回收都是最有效的回收
      2）如果需要被GC的对象很多，保证GC对系统停顿的影响在可控范围内，采取分批循环回收的策略，尽量保证单次服务停顿满足要求。
      核心思想：贪心算法 + 分批回收

      G1和CMS适用场景
      实际用cms的挺多，也有更多经验；
      如果都不熟，先看看g1能否达到你的延迟、吞吐目标；
      还有基础配置，如堆大小，比较大，比如16g以上，建议优先g1；30G以上慎用CMS
      如果gc是stw时间过长，一般也是通过G1来解决

43. 多台机器出现OOM异常，如何处理？

      1）分配的少了：比如虚拟机本身可使用的内存（一般通过启动时的VM参数指定）太少。

      2）应用用的太多，并且用完没释放，浪费了。此时就会造成内存泄露或者内存溢出。

      **内存泄露**：申请使用完的内存没有释放，导致虚拟机不能再次使用该内存，此时这段内存就泄露了，因为申请者不用了，而又不能被虚拟机分配给别人用。

      **内存溢出**：申请的内存超出了JVM能提供的内存大小，此时称之为溢出。

44. 频繁Full GC问题。

      Full GC可以清除老年代的垃圾本质上是好的行为，但是如果Full GC发生的频率高了，就会影响性能。同时，Full GC频繁发生，意味着你的内存分配机制存在问题，也许是内存泄露，有大量内存垃圾不断在老年代产生；也许是你的大对象（缓存）过多；也有可能是你的参数设置不好，minor GC清理不掉内存，导致每次minor GC都会触发Full GC；还有可能是你的老年代大小参数设置错误，老年代过小等等原因，合理排查处理才能最优化自己的代码。

45. jmap命令将内存使用的详细情况输出到文件

      jmap -dump:format=b,file=</opt/log/java12123.hprof> <pid>

46. netty了解

47. Reactor模式理解

      [参考](https://www.jianshu.com/p/2965fca6bb8f)

48. select poll epoll的区别

49. netty ByteBuffer讲解，实现原理

50. Dubbo SPI机制，与原生JDK SPI区别

51. Dubbo有几种负载算法

52. 将API给下游调用，应该遵循什么接口设计原则

     * 高可用：降级兜底、流量控制、快速失败、无状态服务、最少依赖、简单可靠、分散原则、隔离原则
     * 高性能：无锁化、批量处理、缓存、异步、池化
     * 易维护：充分必要、单一职责、内聚解藕、开笔原则、统一原则、用户重试、最小惊讶原则、不要传递无效请求至下游、入参校验

53. 领域建模了解

     领域模型”就是“解决方案空间”，是针对特定领域里的关键事物及其关系的可视化表现，是为了准确定义需要解决问题而构造的抽象模型，是业务功能场景在软件系统里的映射转化，其目标是为软件系统构建统一的认知

     - 帮助分析理解复杂业务领域问题，描述业务中涉及的实体及其相互之间的关系，是需求分析的产物，与问题域相关。 
     - 是需求分析人员与用户交流的有力工具，是彼此交流的语言。 
     - 分析如何满足系统功能性需求，指导项目后续的系统设计。

54. 项目中遇到的比较有挑战性的问题，然后如何解决  

55. 平时如何进行技术上的积累

56. 对于分布式的理解

     * 相比于单体：
       * 独立部署，网络通信
       * 每个服务，可做到弹性扩容，增删节点； 即：以一个集群的形式统一对外提供服务
       * 高可用，避免了单点故障
     * 优点：系统可用性提升、系统并发能力提升、系统容错能力提升、低延迟
     * 缺点：依赖网络、维护成本高、CAP(一致性、可用性、分区容错性)无法同时满足

57. 负载均衡模式

58. jdk动态代理和cglib动态代理实现、区别、性能优劣、使用场景

     1.   jdk：自己
           cglib：继承

     JDK动态代理：

     1. 根据ClassLoader和Interface来获取接口类（前面已经讲了，类是由ClassLoader加载到JVM的，所以通过ClassLoader和Interface可以找到接口类）
     2. 获取构造对象；
     3. 通过构造对象和InvocationHandler生成实例，并返回，就是我们要的代理类。

     ​	优点：

     ​		1.Java本身支持，不用担心依赖问题，随着版本稳定升级；

     ​		2.代码实现简单；

     ​	缺点：

     ​		1.目标类必须实现某个接口，换言之，没有实现接口的类是不能生成代理对象的；	

     ​		2.代理的方法必须都声明在接口中，否则，无法代理；

     ​		3.执行速度性能相对cglib较低；

     Cglib原理：

     1.通过字节码增强技术动态的创建代理对象；

     2.代理的是代理对象的引用；

     Cglib优缺点：

     优点：

     1.代理的类无需实现接口；

     2.执行速度相对JDK动态代理较高；

     缺点：

     1.字节码库需要进行更新以保证在新版java上能运行；

     2.动态创建代理对象的代价相对JDK动态代理较高；

     Tips：

     1.代理的对象不能是final关键字修饰的

59. MyBaits #、$区别、是否预编译

     1. #：字符串（会根据类型进行转换，防止SQL注入）
     2. $：直接显示（多用来传入表名、order by排序）
     3. 通过#{}来判断是否为动态语句
     4. ![05](05.png)

60. Mapple类与xml绑定：通过命名空间(namespace)

61. MyBaits嵌套查询与嵌套结果区别

62. 赃读、幻读、不可重复读

63. 死锁相关问题

64. 数据库优化

65. Restful风格介绍

        * 使用名词定位资源
        * 使用动词描述操作，规定方法幂等性
        * 使用Http状态码或自定义状态码传递server信息
        * 规定返回格式json或xml

66. mq问题，如何配置客户端

67. 常用集合框架

68. sychronized是什么层面（C语言），reentractlock（Java API层面）

69. sychronized的优化，轻量级锁升级重量级锁，如何判断锁类型

70. 锁信息：堆对象布局、类型、状态、GC标识、锁的信息

71. AQS理解、锁底层

72. 读写锁、读饥饿

73. CP、AP

74. MVCC保证事务隔离性中哪一个隔离级别？
        不可重复读，
        能解决幻读吗？
        使用Gap锁和行锁结合使用

        理解：RR级别mvcc可以避免幻“读”（快照在第一次读时生成，之后的快照读都用这个快照），但是无法避免幻“增”、幻“删”、幻“改”
        幻“增”：别的事务在当前事务开始后insert的数据，虽然当前事务selet不到（因为快照读），但insert时可能主键冲突报duplicate key
        幻“删”、幻“改”：delete、update的是实时数据（也就是当前读），因此select不到的数据可能被delete、update到
        还有个特殊的是，select不到的数据可以update到，再次select可以看到刚才被update的数据（符合mvcc可以看到当前事务开始前已经commit的以及当前事务自己修改的数据）

75. MySQL select for update和update加锁区别？

76. MySQL排它锁是表级锁还是行级锁？一定是表级锁吗？

77. 索引失效的10种场景
    1.  不满足最左匹配原则
    2.  使用了select *
    3.  索引列上有计算
    4.  索引列用了函数
    5.  字段类型不同
    6.  like左边包含%
    7.  列对比
    8.  使用or关键字
    9.  not in 和 not exists
    10. order by的坑

78. dp介绍

79. 线上死锁、线程卡住，使用什么shell命令排查、解决
        Jps、JCommander查询进程， top -h查询占用CPU最高线程，复制线程id转换16进制，jstack打线程栈

80. 线程堵塞，CPU占用率：比较低

81. 死锁：互相持有锁，又不释放

82. a转账给b、同时间b转账给a，会造成死锁问题，如何解决？

        * 单线程保障原子性
        * 有序性：每次转账先操作用户id大的，再操作用户id小的

83. 如何关闭一个TCP连接

84. 成员变量、局部变量、静态变量的区别

        |          | 成员变量       | 局部变量                   | 静态变量           |
        | -------- | -------------- | -------------------------- | ------------------ |
        | 定义位置 | 在类中、方法外 | 方法中，或者方法的形式参数 | 在类中、方法外     |
        | 初始化值 | 有默认初始化值 | 无，先定义，赋值后才能使用 | 有默认初始化值     |
        | 调用方式 | 对象调用       | -                          | 对象调用，类名调用 |
        | 储存位置 | 堆中           | 栈中                       | 方法区             |
        | 生命周期 | 与对象共存亡   | 与方法共存                 | 与类共存亡         |
        | 别名     | 实例变量       | -                          | 类变量             |

    ​    
    
85. nginx负载均衡策略，如何根据项目需求合理配置

## 模块化整理

### JVM

1. JVM内存模型,GC机制和原理
2. GC分哪两种, Minor GC和 Full GC有什么区别?
   什么时候会触发FIGC?分别采用什么算法?
3. JVM里有几种 classloader,为什么会有多种?
4. 什么是双亲委派机制?
   介绍一些运作过程,双亲委派模型的好处 什么情况下我们需要破坏双亲委派模型
5. 常见的JVM调优方法有哪些?
   可以具体到调整哪个参数,调成什么值?

### HashMap常见问题及延伸

1. HashMap内部的数据结构是什么? 底层是怎么实现的?
2. HashMap有什么并发问题？有可能还会拓展到 ConcurrentHashMap
3. 了解 LinkedHashMap的应用吗
4. 红黑树的实现原理和应用场景

### 多线程

1. 为什么需要线程池？ 创建线程池的方式有哪些？
2. 线程的生命周期，什么时候会出现进程僵死？
3. 什么是线程安全？如何实现线程安全？
4. 线程池几个重要参数？如何合理配置线程池的大小？
5. 分析线程池的实现原理和线程的调度过程
6. Threadlocal、 volatile的实现原理和使用场景
7. ThreadLocal什么时候会出现OOM的情况？为什么？
8. volatile, synchronized区别，synchronized锁粒度，模拟死锁场景，原子性与可见性

### 框架

1. spring有哪些优势呢?
2. 有没有读过 Spring?源码? 有没有进行解析进行再次封装?
3. SpringBoot比Spring做了哪些改进?Spring 5比Spring 4做了哪些改进?

### 分布式技术栈

1. 你有写过分布式的业务吗?分布式存储呢? 你觉得分布式的话会遇到什么问题呢?
2. 你了解几种消息中间件产品? 各产品的优缺点介绍
3. 消息中间件如何保证消息的一致性和 如何进行消息的重试机制?
4. Redis为什么这么快? Redis采用多线程会有哪些问题?
5. Redis分布式锁操作的原子性，Redis内部是如何实现的?
6. Spring Cloud对比下Dubbo什么场景下该使用 Spring Cloud?

### 数据库

1. 锁机制介绍:行锁、表锁、共享锁、排他锁
2. 乐观锁的业务场景及实现方式
3. 分布式事物的理解,常见的解决方案有哪些, 什么是两阶段提交、三阶段提交

### 多线程

```
可见性：一个线程对共享变量做了修改之后，其他的线程立即能够看到（感知到）该变量这种修改（变化）
	Java内存模型是通过将在工作内存中的变量修改后的值同步到主内存，在读取变量前从主内存刷新最新值到工作内存中，这种依赖主内存的方式来实现可见性的。
原子性：一个操作不能被打断，要么全部执行完毕，要么不执行。
	Java内存模型所保证的是，同线程内，所有的操作都是由上到下的，但是多个线程并行的情况下，则不能保证其操作的有序性。
有序性：在本线程内观察，操作都是有序的；如果在一个线程中观察另外一个线程，所有的操作都是无序的。
```

1. 多线程有什么用？
   * 发挥多核CPU的优势
   * 防止阻塞
   * 便于建模
2. Java如何开启线程？如何保证线程安全？

   ```
   * 线程和进程区别：进程是操作系统进行资源分配的最小单元，线程是操作系统进行任务分配的最小单元，线程隶属于进程
    * 指令重排：系统优化CPU处理线程效率，改变了代码正常的执行顺序
   ```

    * 如何开启线程（概括点，具体操作需进行详细回答）：
      * 继承Thread类，重写run方法（常用）
      * 实现Runnable接口，实现run方法（常用）
        * 实现Callable接口，实现call方法，通过FutureTask创建一个线程，获取到线程执行的返回值
        * 通过线程池来开启线程
    * 怎样保证线程安全（定义：当你有多个线程对同一资源进行操作的时候，就会设计到线程安全）->加锁：
      * 使用JWM提供的锁，也就是Synchronized关键字
      * JDK提供的各种锁Lock

3. Volatile和Synchronized有什么区别？Volatile能不能保证线程安全？DCL(Double Check Lock)单例为什么要加Volatile？

   * Syncchronized关键字，用来加锁；Volatile只是保持变量的线程可见性；通常适用于一个线程写、多个线程读的场景。
   * 不能。Volatile关键字只能保证线程可见性，不能保证原子性。
   * Volatile防止指令重排。在DCL中，防止高并发情况下，指令重排造成的线程安全问题。

4. Java线程锁机制是怎样的？偏向锁、轻量级锁、重量级锁有什么区别？锁机制是如何升级的？

   * JAVA的锁就是在对象的Markword中记录一个锁状态。无锁、偏向锁、轻量级锁、重量级锁对应不同的锁状态。

     ![06](06.png)

   * JAVA的锁机制就是根据资源的竞争的激烈程度不断进行锁升级的过程

   * 

5. 谈谈你对AQS的理解。AQS如何实现可重入锁？

   ```
   AbstractQueuedSynchronizer:抽象队列同步器
   AQS定义了两种资源共享模式：
   独占式，每次只能有一个线程持有锁，例如ReentrantLock实现的就是独占式的锁资源。
   共享式，允许多个线程同时获取锁，并发访问共享资源，ReentrantWriteLock和CountDownLatch等就是实现的这种模式。
   ```

   * AQS是一个JAVA线程同步的框架。是JDK中很多锁工具的核心实现框架。
   * 在AQS中，维护了一个信号量state和一个线程组成的双向链表队列。其中，这个线程队列，就是用来给线程排队的，而state就像是一个红绿灯，用来控制线程排队或者放行的。在不同的场景下，有不同的意义。
   * 在可重入锁场景下，state就用来表示加锁次数。0表示无锁，没加一次锁，state就加1，释放锁就减1.

6. 有A、B、C三个线程，如何保证三个线程同时执行？如何在并发情况下保证三个线程依次执行？如何保证三个线程有序交错执行？

   并发工具：CountDownLatch,CylicBarrier,Semaphore

   * CountDownLatch

     CountDownLatch类位于 java.util.concurrent 包下，**利用它可以实现类似计数器的功能**。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了

   * CylicBarrier

     

   * Semaphore

   

7. 如何对一个很长的字符串进行快速排序？

   Fork/Join框架（分而治之）
   ![07](07.png)

8. 核心线程数设置多少，看任务是 IO密集还是计算密集型，来配置。

9.  主线程获取十个子线程的返回值，怎么优化
   completablefurture


### 网络

1. TCP和UDP有什么区别？TCP为什么是三次握手，而不是两次？
   **TCP** Transfer Contorl protocol 是一种面向连接的、可靠的、传输层通信协议
   	* 类似打电话：面向连接、点对点通信，高可靠的，效率比较低，占用的系统资源比较多

   **UDP** User Datagram Protocol 是一种无连接的、不可靠的、传输层通信协议

   * 类似广播：不需要连接，发送方不管接受方有没有准备好，直接发消息；可以进行广播发送的；传输不可靠，有可能丢失消息；效率高，协议简单、占用的系统资源比较少

   建立连接三次握手、断开连接四次挥手

2. JAVA有哪几种IO模型？有什么区别？

   * BIO：同步阻塞IO

     客户端发请求后，就一直等着服务端响应。客户端：阻塞，请求：同步

     特点：可靠性差，吞吐量低，适用于连接比较少且比较固定的场景；JDK1.4之前唯的选择；编程模型最简单

   * NIO：同步非阻塞IO
     客户端发了请求之后，就去干别的事情了，时不时检查服务端是否给出响应。客户端：非阻塞，请求：同步

     特点：可靠性比较好，吞吐量比较高，适用于连接比较多并且连接比较短（轻操作），例如聊天室；JDK1.4开始支持；编程模型最复杂

   * AIO：异步非阻塞IO
     客户端发了请求之后，就去干别的事情，等到服务器给出响应后，再过来处理业务逻辑。客户端：非阻塞，请求：异步

     特点：可靠性最好，吞吐量非常高，适用于连接标多并且连接比较长（重操作），例如相册服务器，视频流点播室；JDK1.7开始支持；编程模型比较简单，但需要操作系统来支持

     

3. JAVA NIO的几个核心组件是什么？有什么区别？

   * Channel、Buffer、Selector 核心三大组件
   * Channel：类似于一个流，没个channel对应一个buffer缓冲区，channel会注册到selector上面
   * selector会根据channel上发生的读写事件，将请求交由某个空闲线程处理，selector对应一个或多个线程
   * buffer、channel都是可读可写的

4. select，poll和epoll有什么区别（涉及底层概念）？

   * 它们是NIO中多路复用的三种实现机制，是由Linux操作系统提供的

   * 用户空间和内核空间：操作系统为了保护系统安全，将内核划分为两个部分：一个是用户空间，一个是内核空间。用户空间不能直接访问底层的硬件设备，必须通过内核空间。

   * 文件描述符FIle Descriptor(FD)：是一个抽象的概念，形式上是一个整数，实际上是一个索引值，指向内核中为每个进程维护进程所打开的文件的记录表。当程序打开一个文件或是创建一个文件时，内核就会向进城返回一个FD（Unix、Linux）

   * select机制：会维护一个DF的结合fd_set。将fd_set从用户空间复制到内核空间，激活socket。数量限制(x64最大2048)，fd_set是一个数组结构

   * poll机制：和select机制是差不多的，把fd_set进行了优化，FD集合的大小就突破了操作系统的限制，pollfd结构来代替fd_set，通过链表实现

   * epoll(Event Poll)机制：epoll不再扫描所有的FD，只将用户关心的FD事件存放到内核的一个事件表中，这样可以减少用户空间与内核空间之前需要拷贝的数据

   * 简单总结

     |        | 操作方式 | 底层实现 | 最大连接数 | IO效率 |
     | ------ | -------- | -------- | ---------- | ------ |
     | select | 遍历     | 数组     | 受限于内核 | 一般   |
     | poll   | 遍历     | 链表     | 无上限     | 一般   |
     | epoll  | 事件回调 | 红黑树   | 无上限     | 高     |

   * Java当中使用的是哪种机制？

     操作系统不同：windows默认poll(WindowsSelectorProvider)，linux内核高版本(2.6以上)会使用受支持的epoll(EPollSelectorProvider)，否则就是默认的PollSelectorProvider。通过nio.ch.sctp包DefaultSelectorprovider类源码查看

   * Select:1984年出现，poll:1997出现，epoll:2002出现

5. 描述下HTTP和HTTPS的区别（回答要点：概念、运行机制、区别、优缺点）。

   HTTP：是互联网上应用最为广泛的一种网络通信协议，基于TCP，可以使浏览器工作更为高效，减少网络传输。

   HTTPS：是HTTP的加强版，可以认为是HTTP+SSL(Secure Socket Layer)。在HTTP的基础哈桑增加了一系列的安全机制，一方面保证数据传输安全，另一方面对访问者增加了验证机制。是目前现行架构下，最为安全的解决方案。

   主要区别：

   * HTTP的连接是简单无状态的，HTTPS的数据传输是经过证书加密的，安全性更高。
   * HTTP是免费的，而HTTPS需要申请证书，而证书通常是收费的。
   * 传输协议不同，所以两者使用的的端口不一样，HTTP默认80端口，HTTPS默认是443端口。

   HTTPS的缺点：

   * HTTPS的握手协议比较费时，所以会影响服务的响应速度及吞吐量(举例：三次握手时会增加证书验证步骤)
   * HTTPS也并不是完全安全的。它的证书体系其实并不是完全安全的，并且HTTPS在面对DDOS这样的攻击时，几乎起不到任何作用，而且可能会起到反作用。
   * 证书需要费钱，并且功能越强大的证书费用越高。

### JVM调优篇

1. 说一说JVM的内存模型

2. JAVA类加载的全过程是怎样的？什么是双亲委派机制？有什么作用？一个对象从加载到JVM，再到GC被清除，都经历了什么过程？

3. 怎么确定一个对象到底是不是垃圾？什么是GC Root？

### Nginx
1. 什么是Nginx？
   轻量级/高性能的反向代理Web服务器，用于 HTTP、HTTPS、SMTP、POP3 和 IMAP 协议
2. Nginx 有哪些优点？
   
3. Nginx应用场景？
Nginx怎么处理请求的？
Nginx 是如何实现高并发的？
什么是正向代理？
什么是反向代理？
反向代理服务器的优点是什么?
Nginx目录结构有哪些？
Nginx配置文件nginx.conf有哪些属性模块?
cookie和session区别？
为什么 Nginx 不使用多线程？
nginx和apache的区别
什么是动态资源、静态资源分离？
为什么要做动、静分离？
什么叫 CDN 服务？
Nginx怎么做的动静分离？
Nginx负载均衡的算法怎么实现的?策略有哪些?
如何用Nginx解决前端跨域问题？
Nginx虚拟主机怎么配置?
location的作用是什么？
限流怎么做的？
漏桶流算法和令牌桶算法知道？
Nginx配置高可用性怎么配置？
Nginx怎么判断别IP不可访问？
在nginx中，如何使用未定义的服务器名称来阻止处理请求？
怎么限制浏览器访问？
Rewrite全局变量是什么？
Nginx 如何实现后端服务的健康检查？
Nginx 如何开启压缩？
ngx_http_upstream_module的作用是什么?
什么是C10K问题?
Nginx是否支持将请求压缩到上游?
如何在Nginx中获得当前的时间?
用Nginx服务器解释-s的目的是什么?
如何在Nginx服务器上添加模块?
生产中如何设置worker进程的数量呢？
nginx状态码