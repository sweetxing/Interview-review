

## 五种基本类型

### STRINGS

- **自增命令**
  - INCR           加1
  - DECR          减一
  - INCRBY      加整数amount
  - DECRBY     减整数amount
  - INCRBYFLOAT    加浮点数amount
- **处理子串和二进制位**
  - APPEND
  - GETRANGE
  - SETRANGE
  - GETBIT
  - SETBIT
  - BITCOUNT
  - BITOP

### LIST

- **非阻塞列表命令**
  - RPUSH
  - LPUSH
  - RPOP
  - LPOP
  - LINDEX  key-name  offset
  - LRANGE  key-name  start  end
  - LTRIM  key-name start end 裁剪list
- **阻塞列表命令**
  - BLPOP  key-name [key-name ...] timeout
  - BRPOP 
  - RPOPLPUSH  source-key  dest-key   从source-key弹出位于最右端的元素，然后将这个元素推入dest-key列表的最左端
  - BRPOPLPUSH

### SET

- **常用几个命令**
  - SADD
  - SREM
  - SISMEMEBER  key-name  item  检查item是否存在于集合key-name中
  - SCARD  key-name  返回集合包含的元素的数量
  - SMEMBERS  key-name   返回集合包含的所有元素
  - SRANDMEMBER
  - SPOP  key-name  随机地移除集合中的一个元素
  - SMOVE  source-key  dest-key  item  如果集合source-key包含元素item，那么从集合source-key里面移除元素item，并将item添加到集合dest-key中；如果被成功移除，那么命令返回1，否则返回0

- **用于组合和处理多个集合**
  - SDIFF  key-name  [key-name]  返回那些存在于第一个集合、但不存在于其他集合中的元素
  - SDIFFSTORE  dest-key  key-name  [key-name]
  - SINTER  key-name  [key-name]  返回同时存在于所有集合中的元素
  - SINTESTORE
  - SUNION
  - SUNIONSTORE

### HASH

- **用于添加、删除键值对的散列操作**
  - HMGET
  - HMSET
  - HDEL
  - HLEN
- **展示Redis散列的更高级特性**
  - HEXISTS
  - HKEYS
  - HVALS
  - HGETALL
  - HINCRBY
  - HINCRBYFLOAT

### ZSET

- **常用的有序集合命令**
  - ZADD  key-name  score  member  [score  member]
  - ZREM  key-name  member  [member]
  - ZCARD  key-name
  - ZINCRBY  key-name increment  member
  - ZCOUNT  key-name  min  max  返回分值介于min和max之间的成员数量
  - ZRANK  key-name  member  返回成员member在有序集合中的排名
  - ZSCORE  key-name  member  返回成员member的分值
  - ZRANGE  key-name  start  stop  [WITHSCORES]  返回有序集合中排名介于start和stop之间的成员，如果给定了可选的WITHSCORES选项，那么命令会将成员的分值也一并返回
- **有序集合的范围型数据获取、删除命令，以及并集和交集**
  - ZREVRANK  key-name  member  返回有序集合里成员member的逆序排名
  - ZREVRANGE  key-name  start  stop  [WITHSCORES]   返回有序集合给定排名范围内的成员，成员由大到小排
  - ZRANGEBYSCORE  key  min  max  [WITHSCORES]  获取有序集合中分值介于min和max之间的所有成员
  - ZREVRANGEBYSCORE  key  max  min  [WITHSCORES]  [LIMIT  offset  count]
  - ZREMRANGEBYRANK  key-name  start stop
  - ZREMRANGEBYSCORE  key-name min max
  - ZINTERSTORE   dest-key key-count  key  [key ...]  [WEIGHTS  weight  [weight ...]]  [AGGREGATE  SUM|MIN|MAX]
  - ZUNIONSTORE  dest-key key-count  key  [key ...]  [WEIGHTS  weight  [weight ...]]  [AGGREGATE  SUM|MIN|MAX]

## 数据安全与性能保障

### 持久化选项

```bash
# 快照持久化选项
save 60 10000                           # 多久执行一次快照操作
stop-writes-on-bgsave-error no          # 创建快照失败后是否仍然继续执行写命令
rdbcompression yes                      # 是否对快照文件进行压缩
dbfilename dump.rdb                     # 快照文件名
# AOF持久化选项
appendonly no                           # 是否使用AOF持久化
appendfsync everysec                    # 多久才将写入的内容同步到硬盘
no-appendsync-on-rewrite no             # 在对AOF进行压缩的时候是否能执行同步操作
auto-aof-rewrite-percentage 100         # 
auto-aof-rewrite-min-size 64mb          # 

dir ./                                  # 共享选项，决定了快照文件和AOF文件的保存位置。
```

- **快照持久化**
  - 客户端可以通过向Redis发送BGSAVE命令来创建。对于支持BGSAVE命令的平台来说（基本上所有平台都支持）

## 三种多机数据库实现

### 复制

### Sentinel

#### 1.哨兵

```c
struct sentinelState {
    uint64_t current_epoch;     /* 当前纪元，用于实现故障转移 */
    dict *masters;      /* key：主服务器的名字, value：指向sentinelRedisInstance结构的指针 */
    int tilt;           /* TILT模式? */
    int running_scripts;    /* 正在执行的脚本数量. */
    mstime_t tilt_start_time;   /* When TITL started. */
    mstime_t previous_time;     /* Last time we ran the time handler. */
    list *scripts_queue;    /* Queue of user scripts to execute. */
    char *announce_ip;      /* IP addr that is gossiped to other sentinels if
                               not NULL. */
    int announce_port;      /* Port that is gossiped to other sentinels if
                               non zero. */
} sentinel;
```

```c
typedef struct sentinelRedisInstance {
    int flags;      /* 标识值，记录了实例的类型，以及该实例的当前状态 */
    char *name;     /* Master name from the point of view of this sentinel. */
    char *runid;    /* 实例的运行ID */
    uint64_t config_epoch;  /* Configuration epoch. */
    sentinelAddr *addr; /* Master host. */
    redisAsyncContext *cc; /* Hiredis context for commands. */
    redisAsyncContext *pc; /* Hiredis context for Pub / Sub. */
    int pending_commands;   /* Number of commands sent waiting for a reply. */
    mstime_t cc_conn_time; /* cc connection time. */
    mstime_t pc_conn_time; /* pc connection time. */
    mstime_t pc_last_activity; /* Last time we received any message. */
    mstime_t last_avail_time; /* Last time the instance replied to ping with
                                 a reply we consider valid. */
    mstime_t last_ping_time;  /* Last time a pending ping was sent in the
                                 context of the current command connection
                                 with the instance. 0 if still not sent or
                                 if pong already received. */
    mstime_t last_pong_time;  /* Last time the instance replied to ping,
                                 whatever the reply was. That's used to check
                                 if the link is idle and must be reconnected. */
    mstime_t last_pub_time;   /* Last time we sent hello via Pub/Sub. */
    mstime_t last_hello_time; /* Only used if SRI_SENTINEL is set. Last time
                                 we received a hello from this Sentinel
                                 via Pub/Sub. */
    mstime_t last_master_down_reply_time; /* Time of last reply to
                                             SENTINEL is-master-down command. */
    mstime_t s_down_since_time; /* Subjectively down since time. */
    mstime_t o_down_since_time; /* Objectively down since time. */
    mstime_t down_after_period; /* Consider it down after that period. */
    mstime_t info_refresh;  /* Time at which we received INFO output from it. */

    /* Role and the first time we observed it.
     * This is useful in order to delay replacing what the instance reports
     * with our own configuration. We need to always wait some time in order
     * to give a chance to the leader to report the new configuration before
     * we do silly things. */
    int role_reported;
    mstime_t role_reported_time;
    mstime_t slave_conf_change_time; /* Last time slave master addr changed. */

    /* Master specific. */
    dict *sentinels;    /* 其他监听此服务器的哨兵 */
    dict *slaves;       /* Slaves for this master instance. */
    unsigned int quorum;/* Number of sentinels that need to agree on failure. */
    int parallel_syncs; /* How many slaves to reconfigure at same time. */
    char *auth_pass;    /* Password to use for AUTH against master & slaves. */

    /* Slave specific. */
    mstime_t master_link_down_time; /* Slave replication link down time. */
    int slave_priority; /* Slave priority according to its INFO output. */
    mstime_t slave_reconf_sent_time; /* Time at which we sent SLAVE OF <new> */
    struct sentinelRedisInstance *master; /* Master instance if it's slave. */
    char *slave_master_host;    /* Master host as reported by INFO */
    int slave_master_port;      /* Master port as reported by INFO */
    int slave_master_link_status; /* Master link status as reported by INFO */
    PORT_ULONGLONG slave_repl_offset; /* Slave replication offset. */
    /* Failover */
    char *leader;       /* If this is a master instance, this is the runid of
                           the Sentinel that should perform the failover. If
                           this is a Sentinel, this is the runid of the Sentinel
                           that this Sentinel voted as leader. */
    uint64_t leader_epoch; /* Epoch of the 'leader' field. */
    uint64_t failover_epoch; /* Epoch of the currently started failover. */
    int failover_state; /* See SENTINEL_FAILOVER_STATE_* defines. */
    mstime_t failover_state_change_time;
    mstime_t failover_start_time;   /* Last failover attempt start time. */
    mstime_t failover_timeout;      /* Max time to refresh failover state. */
    mstime_t failover_delay_logged; /* For what failover_start_time value we
                                       logged the failover delay. */
    struct sentinelRedisInstance *promoted_slave; /* Promoted slave instance. */
    /* Scripts executed to notify admin or reconfigure clients: when they
     * are set to NULL no script is executed. */
    char *notification_script;
    char *client_reconfig_script;
} sentinelRedisInstance;
```

#### 2. 获取主服务器信息

### 集群

#### 1. 节点

```c
typedef struct clusterNode {
    mstime_t ctime; /* 节点创建时间 */
    char name[REDIS_CLUSTER_NAMELEN]; /*节点名子 */
    int flags;      /* 使用不同的标识值记录节点的角色（主/从节点; 在线/下线） */
    uint64_t configEpoch; /* 节点当前的配置纪元，用于实现故障转移；故障转移一次，就加一 */
    unsigned char slots[REDIS_CLUSTER_SLOTS/8]; /* slots handled by this node */
    int numslots;   /* 负责的槽节点数量 */
    int numslaves;  /* 从节点数量 */
    struct clusterNode **slaves; /* 从节点数组 */
    struct clusterNode *slaveof; /* 如果我是从节点，指向我的主节点 */
    mstime_t ping_sent;      /* Unix time we sent latest ping */
    mstime_t pong_received;  /* Unix time we received the pong */
    mstime_t fail_time;      /* Unix time when FAIL flag was set */
    mstime_t voted_time;     /* Last time we voted for a slave of this master */
    mstime_t repl_offset_time;  /* Unix time we received offset for this node */
    PORT_LONGLONG repl_offset;      /* Last known repl offset for this node. */
    char ip[REDIS_IP_STR_LEN];  /* Latest known IP address of this node */
    int port;                   /* Latest known port of this node */
    clusterLink *link;          /* 保存连接节点所需的有关信息 */
    list *fail_reports;         /* 记录了所有其他节点对该节点的下线报告 */
} clusterNode;
```

```c
typedef struct clusterLink {
    mstime_t ctime;             /* 连接的创建时间 */
    int fd;                     /* TCP套接字描述符 */
    sds sndbuf;                 /* 输出缓冲区，发送给其他节点的消息 */
    sds rcvbuf;                 /* 输入缓冲区，从其他节点接收到的信息 */
    struct clusterNode *node;   /* 与此连接相关联的节点 */
} clusterLink;
```

```c
// 保存在当前节点的视角下，集群所处的状态，例如集群是在线还是下面，集群包含多少个节点，集群当前的配置纪元
typedef struct clusterState {
    clusterNode *myself;  /* 指向当前节点的指针 */
    uint64_t currentEpoch;  /* 集群当前的配置纪元，用于实现故障转移 */ 
    int state;            /* 集群当前的状态：REDIS_CLUSTER_OK, REDIS_CLUSTER_FAIL, ... */
    int size;             /* 集群中至少处理着一个节点的数量 */
    dict *nodes;          /* 集群节点名单：Hash table of name(名字) -> clusterNode structures(节点) */
    dict *nodes_black_list; /* Nodes we don't re-add for a few seconds. */
    clusterNode *migrating_slots_to[REDIS_CLUSTER_SLOTS];     /*当前节点正在导出到其他节点的槽*/  
    clusterNode *importing_slots_from[REDIS_CLUSTER_SLOTS];   /*当前节点正在从其他节点导入的槽*/
    clusterNode *slots[REDIS_CLUSTER_SLOTS];
    zskiplist *slots_to_keys;
    /* The following fields are used to take the slave state on elections. */
    mstime_t failover_auth_time; /* Time of previous or next election. */
    int failover_auth_count;    /* Number of votes received so far. */
    int failover_auth_sent;     /* True if we already asked for votes. */
    int failover_auth_rank;     /* This slave rank for current auth request. */
    uint64_t failover_auth_epoch; /* Epoch of the current election. */
    int cant_failover_reason;   /* Why a slave is currently not able to
                                   failover. See the CANT_FAILOVER_* macros. */
    /* Manual failover state in common. */
    mstime_t mf_end;            /* Manual failover time limit (ms unixtime).
                                   It is zero if there is no MF in progress. */
    /* Manual failover state of master. */
    clusterNode *mf_slave;      /* Slave performing the manual failover. */
    /* Manual failover state of slave. */
    PORT_LONGLONG mf_master_offset; /* Master offset the slave needs to start MF
                                   or zero if stil not received. */
    int mf_can_start;           /* If non-zero signal that the manual failover
                                   can start requesting masters vote. */
    /* The followign fields are used by masters to take state on elections. */
    uint64_t lastVoteEpoch;     /* Epoch of the last vote granted. */
    int todo_before_sleep; /* Things to do in clusterBeforeSleep(). */
    PORT_LONGLONG stats_bus_messages_sent;  /* Num of msg sent via cluster bus. */
    PORT_LONGLONG stats_bus_messages_received; /* Num of msg rcvd via cluster bus.*/
} clusterState;
```

#### 2. 槽指派

```shell
CLUSTER ADDSLOTS <slot> [slot...]
```

clusterNode中的slots字段通过2048字节中的16384个二进制位表示，节点负责处理的的槽位。

**Notation:**  当且仅当所有的槽都有指派时集群才会上限

#### 3. 故障检测

```c
typedef struct clusterNodeFailReport {
    struct clusterNode *node;  /* 报告目标节点已经下线的节点 */
    mstime_t time;             /* 最后一次从node节点收到下线报告的时间；程序使用这个时间戳检查下线报告是否过期 */
} clusterNodeFailReport;
```

超过一半的主节点打小报告说你下线，那你就完了。

#### 4. 五种消息

- **MEET消息**
- **PING**
- **PONG**
- **FAIL**
- **PUBLISH**

```c
typedef struct {
    char sig[4];        /* Siganture "RCmb" (Redis Cluster message bus). */
    uint32_t totlen;    /* Total length of this message */
    uint16_t ver;       /* Protocol version, currently set to 0. */
    uint16_t notused0;  /* 2 bytes not used. */
    uint16_t type;      /* Message type */
    uint16_t count;     /* Only used for some kind of messages. */
    uint64_t currentEpoch;  /* The epoch accordingly to the sending node. */
    uint64_t configEpoch;   /* The config epoch if it's a master, or the last
                               epoch advertised by its master if it is a
                               slave. */
    uint64_t offset;    /* Master replication offset if node is a master or
                           processed replication offset if node is a slave. */
    char sender[REDIS_CLUSTER_NAMELEN]; /* Name of the sender node */
    unsigned char myslots[REDIS_CLUSTER_SLOTS/8];
    char slaveof[REDIS_CLUSTER_NAMELEN];
    char notused1[32];  /* 32 bytes reserved for future usage. */
    uint16_t port;      /* Sender TCP base port */
    uint16_t flags;     /* Sender node flags */
    unsigned char state; /* Cluster state from the POV of the sender */
    unsigned char mflags[3]; /* Message flags: CLUSTERMSG_FLAG[012]_... */
    union clusterMsgData data;
} clusterMsg;
```

```c
union clusterMsgData {
    /* PING, MEET and PONG */
    struct {
        /* Array of N clusterMsgDataGossip structures */
        clusterMsgDataGossip gossip[1];
    } ping;

    /* FAIL */
    struct {
        clusterMsgDataFail about;
    } fail;

    /* PUBLISH */
    struct {
        clusterMsgDataPublish msg;
    } publish;

    /* UPDATE */
    struct {
        clusterMsgDataUpdate nodecfg;
    } update;
};
```

