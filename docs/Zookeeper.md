### Zookeeper

#### Zookeeper 概述

​    Zookeeper 是**分布式协调框架**，解决分布式集群中的一致性问题。本质上是一个**分布式**的**小文件存储系统**，提供类似文件系统的目录树方式的数据存储，可以对树中的节点进行管理，从而维护和监控数据的状态变化。通过监控这些数据状态的变化，从而达到数据的集群管理。类似统一命名服务、分布式配置管理、分布式消息队列、分布式锁、分布式协调登功能。

#### Zookeeper 特性

**全局数据一致性**：每个 server 保存一份数据=副本， client 无论连接到哪个 server 得到的数据都是一致的。

**可靠性**：如果消息被一台服务器接受，那么将被所以的服务器接受。

**顺序性**：包括全局有序和偏序两种：全局有序是指如果在一台服务器上消息 a 在消息 b 前发布，则在所以 server 上消息 a 都将在消息 b 前被发布；偏序是指如果一个消息 b 在消息 a 后被同一个发送者发布， a 必须排在 b 前面。

**数据更新原子性**：一次数据要么成功要么失败。

**实时性**: Zookeeper 保证客户端将在一个时间间隔内获得服务器的更新消息或者服务器失败的消息。

#### Zookeeper 集群角色

![zkservice](F:\JAVA\document\git-note\java-notes\gitbook\images\zkservice.jpg)

##### Leader

集群的核心，事务请求（写操作）的唯一调度和处理者，保证集群事务处理的顺序性，集群内部各个服务器的调度者，所有 create，setData， delete 等写操作请求会 统一转发给 Leader 处理， Leader 需要决定编号、执行操作，这个过程为一个事务。

##### Follower

处理客户端非事务（读操作）操作，转发事务请求给 Leader，参与集群 Leader 选举投票。

##### Observer

观察者角色，观察 Zookeeper 集群的最新状态变化并将这些状态同步过来，可以处理非事务请求，事务请求会转发给 Leader 。不会参与 Leader 选举投票，在不影响集群事务处理能力的前提下水平扩展集群。

#### Zookeeper 集群搭建

集群数量 `2n + 1` ，因为Paxos算法

### Zookeeper shell

ls [path] 节点列表

create -e -s [path] [data] 创建节点

- -s 序列化

- -e 序列化
- [path] 绝对路径
- [data] 节点数据

get [path] [watch] 获取数据

- [watch] 1 表示创建监听

set [path] [data] 更新节点数据

rmr [path] 移除节点

setquota -n|-b val [path] 限制节点，只是打印警告信息，没有实际限制作用

- -n 限制子节点数量
- -b 限制字节大小

history 显示所有的历史命令

redo [n] 执行历史命令，n代表历史命令的 id

### Zookeeper 数据模型

#### 数据结构

![zknamespace](F:\JAVA\document\git-note\java-notes\gitbook\images\zknamespace.jpg)

树形层次结构，树中的每个节点称为 Znode。Znode 包含三部分信息：stat 状态信息、data 数据、子节点。

- Znode 兼具文件和目录两种特点，既维护着数据、元信息、ACI（房访问控制列表）、时间戳等数据结构，又可以作为路径标识的一部分，并可以具有子 Znode。可以增删改查。
- Znode 具有原子性操作
- Znode 只能存储不超过 1M 的数据
- Znode 只能使用绝对路径引用，以斜杠开头的Unicode字符串组成

#### 节点类型

临时节点：会话结束自动结束，没有子节点

永久节点：一直存在直到被手动删除。

Znode 的序列化特性，如果创建时指定则该 Znode 的名字后面会自动追加一个不断增加的唯一序列号，以此表示节点创建的先后顺序，格式为 “%10d” （10位数字，没有值的位以 0 补充）

四种节点类型：

- PERSISTENT 永久节点
- EPHEMERAL 临时节点
- PERSISTENT_SEQUEnTIAL 序列化的永久节点
- EPHEMERAL_SEQUENTIAL 序列化的临时节点

#### 节点属性

使用命令 `get` 命令可以获取节点属性。

- dataVersion 数据版本号，每次 `set` 操作加一
- cversion 子节点版本号，Znode 的子节点变化时 cversion 值加一
- aclVersion ACL 的版本号
- cZxid 事务 id
- mZxid Znode 被修改时更新 mZxid
- ctime 节点创建时间戳
- mtime 节点最后一次更新时间
- ephemeralOwner 临时节点绑定的 session id，非临时节点值为 0

### Zookeeper Watcher

Zookeeper 引入 Watcher 机制来实现分布式的通知机制（发布/订阅）。Zookeeper 允许客户端向服务端注册一个 Watcher 监听，当服务端的事件触发这个 Watcher 则会向客户端发哦是那个一个事件通知来实现分布式的通知功能。

特点;

- 一次性触发：通知只发送一次
- 事件封装：一个事件包括三个属性：通知状态（keeperState）、事件类型（EventType）、节点路径（path）
- event 异步发送
- 先注册再触发

![zookeeper watcher 状态](F:\JAVA\document\git-note\java-notes\gitbook\images\zookeeper watcher 状态.png)

连接状态事件（type=None, path=null）不需要客户端注册，客户端只要有需要直接处理就行了。

API 使用参考 help 命令后的列表中有 `[watch]` 的命令都可以注册监听。

### Zookeeper 客户端API

Zookeeper 类主要方法

- **connect** - 连接到ZooKeeper集合
- **create(String path, byte[] data, List<ACL> acl, CreateMode createMode)**- 创建znode
- **exists(String path, boolean watcher)**- 检查znode是否存在及其信息
- **getData(String path, Watcher watcher, Stat stat)** - 从特定的znode获取数据
- **setData(String path, byte[] data, int version)** - 在特定的znode中设置数据
- **getChildren(String path, Watcher watcher)** - 获取特定znode中的所有子节点
- **delete(String path, int version)- 删除特定的znode及其所有子项
- **close** - 关闭连接
- **ZooKeeper(String connectionString, int sessionTimeout, Watcher watcher)** 创建连接

### Zookeeper 选举机制

leader选举的过程如下：

- 所有节点创建具有相同路径 /app/leader_election/guid_ 的顺序、临时节点。
- ZooKeeper集合将附加10位序列号到路径，创建的znode将是 /app/leader_election/guid_0000000001，/app/leader_election/guid_0000000002等。
- 对于给定的实例，在znode中创建最小数字的节点成为leader，而所有其他节点是follower。
- 每个follower节点监视下一个具有最小数字的znode。例如，创建znode/app/leader_election/guid_0000000008的节点将监视znode/app/leader_election/guid_0000000007，创建znode/app/leader_election/guid_0000000007的节点将监视znode/app/leader_election/guid_0000000006。
- 如果leader关闭，则其相应的znode/app/leader_electionN会被删除。
- 下一个在线follower节点将通过监视器获得关于leader移除的通知。
- 下一个在线follower节点将检查是否存在其他具有最小数字的znode。如果没有，那么它将承担leader的角色。否则，它找到的创建具有最小数字的znode的节点将作为leader。
- 类似地，所有其他follower节点选举创建具有最小数字的znode的节点作为leader。

### Zookeeper 典型应用

Hadoop

Solr

### 网络编程

