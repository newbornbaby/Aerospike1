# 1.简介
aerospike是高效的nosql数据库，采用纯内存或者内存+ssd方案存储数据，设计初衷达到了以下三个主要目的：
* 数据的快速、高效存取。
* 提供传统数据库的可靠性、健壮性、ACID能力。
* 灵活性、扩展性、容错能力。
  
# 2.架构设计
as采用**share nothing设计模式**，在各个处理单元中使用自己私有的cpu以及memory，不存在共享资源，类似于MMP（大规模并行处理）模式，各处理单元之间通过协议通信，并行处理和扩展能力强（典型的类似案例还有hadoop等），各节点处理完数据后，处理结果可以向上层汇总，或者在节点之间流转；此外，as的架构设计主要可以分为三层：
* **Client Layer（客户端）层**：实现as API，直接与集群通信、跟踪节点，发现节点的上下线、知道数据在节点中的位置。
* **Clustering and Data Distribution Layer（集群和数据分发）层**：管理集群、自动故障转移、高效分配任务、智能平衡数据、自动数据备份、数据迁移、跨数据中心同步数据。
* **Data Storage Layer（数据存储）层**：可靠地将数据存储到DRAM与FLASH中。（***as架构图待补充***）
                                
# 3.详细设计
## 3.1 Distribution Layer
分发层旨在通过系统自动化所有集群管理功能，来最大程度的消除手动操作，该层有三个重要模块：
### 3.1.1 Cluster Managerment Module（集群管理模块）：
**重点**：通过Paxos-base gossip-voting process算法来确认集群内节点，使集群中所有节点对当前集群成员身份达成一致（Paxos是一中在分布式设计中广泛使用的分布式一致性协议），同时通过heartbeat监控各个节点的状态。
* **Cluster View（集群视图）**：集群中的每一个一个node都会注册一个identifier，有涉及mac以及port的一个算法产生；集群视图由<cluster_key, succession_list>标识，其中cluster_key是一个8位的随机数，succession_list是集群成员标识符集合，当集群成员状态发生变动时，会产生一个新的cluster_key以及对应的succession_list，来标识新视图；由于每次成员变动都会影响集群重新配置视图，导致处理延迟和处理效率下降，所以有必要使用一个一致性机制，快速有效地使集群重新达到一致状态。
* **Cluster Discovery**：集群中的每个node通过向其他节点周期性地持续地发送心跳包来告诉其他节点自己的状态，每个节点维护一份存活着的node列表，里面记录了会向这个节点发送消息的其他节点的标识，心跳时间如果超过配置的超时时间还未收到，那么就移除该可能发生故障的节点。实际情况下，需要考虑到因网络抖动等问题，导致正常节点被排除的情况；此外，还需要避免一些不稳定节点频繁加入或者排除的情况。为了高效的使集群保持或者重新到达一致性状态，从以下两方面来考虑问题： ***（1）Surrogate heartbeats（辅助心跳）*** ：除主心跳外的心跳机制，例如，将副本写入作为辅助心跳，若副本写入完好，或者主心跳完好，可确保仅主心跳上出现的网络异常，不会影响集群视图。 ***（2）Node Health Score（节点健康分数）*** ：衡量节点健康情况的分数，当节点的score分数超过所有节点平均值的两倍时，则说明该节点存在异常。_（***公式待补充***）_
* **Cluster View Change**：首先阐述一下集群view改变过程，当集群进入view更改周期（集群更改周期可以设置一般不宜设置太短，也不宜设置太长，太短会太频繁，太长会导致集群异常状态持续太长，请求无法处理。）时，将新增节点放在节点列表的最上面，作为paxos请求申请者，然后整个集群按照paxos算法（***算法的具体过程待补充***）来判断是否通过该请求，若通过则开始重新平衡数据，在请求正常通过的情况下，paoxs将花费三个网络周期时间，为了应对频繁修改view的情况，特设集群更改周期为超时时间的两倍，可以减少多次修改view状态带来的开销，多次修改可以在一个周期内完成。
* **Data Distribution**：一个primary key通过RipeMD160算法被变成160位的摘要，摘要被分配到4096个分区空间中，分区是Aerospike中最小的数据存在单位。根据主键摘要来为记录分配分区。即使键在键空间中的分布是倾斜的，但是摘要在分配到分区空间中时是均匀的，此数据分区方案是Aerospike独有的，有助于避免在数据访问期间创建热点，实现高级别的规模和容错。as在运行读取或者查询操作时，将索引与对应的记录放在同一个节点上可以避免数据的跨节点读取，索引与数据并置，在搭配一个健壮性高的分布式hash算法，可以达到各节点数据均价分布且一致性的效果。采用此种方式分配数据的好处： ***（1）应用程序，工作负载在集群中均匀分布*** ； ***（2）数据库操作性能是可预测的*** ； ***（3）集群的向上和向下扩展很容易*** ； ***（4）实时集群重新配置和随后的重新平衡是简单的、无中断的、高效的。***
* **A Partition Assignment Algorithm（分区分配算法）**：该算法为每个分区生成一个replication list，该列表的第一个节点是主节点，第二个节点是第一个副本节点，以此类推，最后可能有未使用的节点，分区分配的结果被称为partition map（***分区图的具体示例可以参照图1，待补充***）。当一个请求过来时，首先往主节点发送，当请求为read请求时，也可以均匀的通过所有的节点包括副本节点（这可以在请求时动态配置），一个as集群支持任何数量的拷贝，只要其拷贝数量不超过集群的节点数即可。分区分配算法需要达到以下几个目的： ***（1）具有确定性，使分布式系统中每个节点均能单独地计算出相同的partition map*** ； ***（2）实现主节点和副本节点在集群中的均匀分布*** ； ***（3）在集群视图变动时，减小分区的变动*** 。算法的核心NODE_HAS_COMPUTE这个方法将nodeId、partitionId结合生成一个hash值，REPLICATION_LIST_ASSIGN，该方法是用来获取replication list的，它将循环遍历succession_list，每次遍历都调用NODE_HAS_COMPUTE方法，将nodeId与hash值关联存储起来，然后按照nodeId的hash值排序。（***算法的具体实现可以参照图2，待补充***）。到此，我们可以知道根据这个算法可以确定每个分区的节点分布图，且每个节点均有计算该分布图的能力，可以实现目的1；通过Jenkins one-at-a-time方法排列分区主副节点可以获取很好的散布性，可以实现目的2；由于分区图的确定，在增加或者减少节点时，可以尽可能的考虑重新分配分区，变动最小的方案，可以实现目的3。
### 3.1.2 Date Migration Module（数据迁移模块）
**重点概述**：将记录从一个节点迁移到另一个节点的过程称为迁移。每次集群视图更改后，数据迁移开始，其目标是在每个数据分区的当前主节点和副本节点上提供每个记录的最新版本。一旦在新的视图上达成共识，集群中的所有节点运行分区分配算法，并将主节点和一个或多个副本节点分配给每个分区。每个分区的主节点为该分区分配一个唯一的分区版本，此版本号将复制到副本节点。在集群视图更改之后，节点之间交换每个具有数据的分区的分区版本。因此，每个主节点都知道分区的每个副本的版本号。接下来详细描述节点挂了之后的数据迁移过程：当一个节点挂了之后，其心跳停止，其他节点在集群视图更改周期中发起移除申请，根据paxos算法，集群达成移除共识，集群的视图发生改变，seccession_list更新，各个节点运行分区分配算法，计算出新的统一分区分布图，分区重新分布，新分区中当前主节点和副本节点之间交换每个记录的最新版本，交换完成后主节点知道其他每个分区的版本号，它为分区生成一个统一的新的分区版本号，之后将该分区版本号传递给其他副本节点达成一致状态，副本数据分区可以通过copy的方式从当前主分区（可能是代理主分区，之后讨论何为代理主分区，在这里你就认为是一个临时主分区）获取数据，为了减少工作量，对比副本节点数据区数据和当前主节点数据区数据，只需要将不同的数据采用更新策略即可，一样的数据可不做处理，在所有分区的主副节点的数据区copy完成后，数据迁移结束。
* **Delta-Migrations（增量迁移）**：为了缩短数据迁移的时间，减少集群开销，as采取了一些优化策略来提高效率。在数据迁移的过程中，as会尝试先为每个分区的当前版本进行排序，对于同一分区，当node1的分区版本小于node2的分区版本时，显然node1的数据需要从当前主节点重新同步，当node2的版本恰好和当前分区版本一致时，node2就不需要同步了，节省了集群开销。但是，并不一定能够为所有的分区的版本都能被排序，当集群节点之间有网络分区时，有些节点无法参与排序，此时，集群中的一些节点可能无法保证此时该记录是集群的最新版本。
* **Operations During Migrations（数据迁移时的请求处理）**：在数据迁移期间，如果发生read请求，as保证返回最终获取胜利的记录，即迁移后的最终结果数据。对于部分的写操作，as保证记录将写入最终获取胜利的记录，即迁移后的最终结果数据。为了确保上述承诺的实现，在数据迁移期间，as会让操作进入duplicate resolution（重复判断）逻辑，在duplicate resolution期间，包含特定记录的分区当前主节点将读取其所有分区版本中的记录，并解析为该记录的一个副本（最新）。这是获胜的副本，今后将用于读或写事务。
* **Master Partition Without Data**：在集群视图变动时，可能出现部分分区新的副本列表主节点为空的情况，这时当读写请求进来后，进入duplicate resolution（重复判断）期间，在此期间时，会选取一个记录最多的节点当代理主节点，代理节点在迁移完成后结束生命周期，节点在同步未完成时会有一个DESYNC标志（未同步），当然如果请求主动声明不需要进入重复判断阶段，能够接受可能出现的返回数据不全或者数据过期或者空数据的情况，也可以。
* **Migration Ordering（数据迁移顺序涉及两个优化策略Smallest Partition First、Hottest Partition First）**：在数据迁移期间，as没有设定迁移操作的优先级以及其他操作（例如读取操作）的优先级谁高谁低，所以基本上这些操作会同时进行，为了减少在数据迁移期间处理其他操作带来的额外集群开销，需要尽可能快的完成数据迁移。as有以下两种策略来优化性能： ***（1）Smallest Partition First*** ：最小分区优先迁移原则； ***（2）Hottest Partition First*** ：热点数据优先迁移原则。
* **Summary**：数据的统一分布、相关的元数据（如索引）和事务工作负载使Aerospike集群的容量规划和上下扩展决策精确而简单。Aerospike只需要对集群成员的变化进行数据重新分配。这与基于备用键范围的分区方案形成了对比，后者要求在范围“大于”其节点上的容量时重新分配数据。

### 3.1.3 Transaction Processing Module（事务操作模块）
**重点概述**：读写数据时，as提供一致性和隔离性保证，此外，该模块提供以下保证。
* **Sync/Async Replication**：对于具有即时一致性写入，在返回和提交数据之前，将广播所有改变同步到所有备份节点。
* **Proxy**：在一些极少数的情况下，当集群重新配置时，客户端层可能暂时过时，事务处理模块可以透明地将请求代理到另一个节点执行。
* **Duplicate Replication**：当集群从分区中恢复时，需要解决不同数据副本之间的任何可能出现的冲突，该模块即可提供此功能，当一个集群启动后，可以在其他数据中心安装其他集群，并设置跨数据中心恢复，以确保如果数据中心关闭，远程集群将以最小或不中断用户请求的方式接管工作。
## 3.2 Client Layer
数据库不单独存在。因此，它们必须作为完整堆栈的一部分进行架构设计，以便端到端系统能够扩展。客户机层需要处理管理集群的复杂性。这里有各种各样的挑战需要克服，下面将讨论其中的一些。
* **Discovery**：客户机需要了解集群的所有节点及其角色。在as中，每个节点维护其adjacency list（相邻节点的列表）。此列表用于发现群集节点。客户机从一个或多个种子节点开始，并发现整个集群节点集。一旦发现了所有节点，客户机就需要知道每个节点的角色。节点的角色在分发层中解释过，就是，分区中对应的副本是主节点角色还是副节点角色。这个从分区到节点的映射（partition map）与客户机交换和缓存。与客户机共享分区映射对于使客户机-服务器交互至关重要。这就是为什么在as中，客户机对数据可以支持单跳访问的原因。在稳态下，as集群的扩展能力纯粹是客户机或服务器节点数量的函数。这保证了系统的线性可扩展性，只要系统的其他部分（如网络互连）能够正常运行即可。
* **Information Sharing**：每个客户机进程将分区映射存储在其内存中。为了使信息保持最新，客户机进程定期咨询服务器节点以检查是否有任何更新。它通过检查本地存储的版本与服务器的最新版本来实现这一点。如果有更新，它会请求完整的分区映射。像php cgi、node.js集群这样的框架可以在每台机器上运行多个客户机进程实例，以获得更多的并行性。由于客户机的所有实例都在同一台机器上，因此它们应该能够在自己之间共享这些信息。Aerospike使用共享内存和pthread库中强大的互斥代码的组合来解决这个问题。pthread mutex支持可跨进程使用的以下属性：PTHREAD_MUTEX_ROBUST_NP PTHREAD_PROCESS_SHARED，使用这些属性集在共享内存区域中创建锁。所有进程都周期性地竞争（每秒钟一次）以获取锁。然而，只有一个进程将获得锁。获取锁的进程从服务器节点获取分区映射，并通过共享内存与其他进程共享分区映射。如果持有锁的进程死了，当另一个进程尝试获取锁时，它将使用返回代码eownerdead获取锁。它应该调用pthread_mutex_consistent_np（）使锁保持一致，以便进一步使用。之后，一切照常进行。
* **Cluster Node Handling**：对于每个集群节点，在初始化时，客户机代表该节点创建一个内存中的结构，并存储其分区映射。它还为该节点维护一个连接池。当节点被声明为中断时，所有这些都将被销毁。初始化和销毁是一项代价高昂的操作，此外，如果出现故障，客户机需要有一个回退计划来处理故障，可以在同一个节点或群集中的不同节点上重试数据库操作，如果底层网络是脆弱的，并且这种情况反复发生，那么最终可能会降低整个系统的性能。这就需要有一种平稳的方法来识别集群节点的健康状况。as使用以下策略来实现这种平衡： ***（1）Health Score（健康指数）*** ：客户机单独使用事务响应状态码，作为度量集群状态的一种次优方案，已连接的节点可能暂时无法接受事务请求。或者可能是存在暂时的网络问题，而节点本身是正常的，为了减少这种情况的影响，客户机跟踪记录客户机在特定集群节点上操作时遇到的失败次数。只有当失败计数（也称为“幸福系数”）超过特定阈值时，客户机才会销毁集群节点的相应的信息。对该节点的任何成功操作都会将失败计数重置为0； ***（2）Cluster Consultation（集群轮询）***  ：脆弱的网络往往难以处理，单向网络故障（A见B，B不见A）更为严重，在某些情况下，集群节点可以彼此看到，但客户机无法直接看到某些集群节点（例如X），在这些情况下，客户机查询集群中对其自身可见的所有节点，并查看这些节点中是否有任何节点的adjacency list（相邻节点的列表）中有X，如果集群中的客户机可见节点报告X在其中，则客户机不执行任何操作，否则，客户机将等待一个阈值时间，然后通过销毁节点的数据结构来永久删除该节点。经过几年的部署，我们发现该方案大大提高了整个系统的稳定性。
## 3.3 STORAGE
**重点概述**：as底层存储使用了SSD技术，将内存和SSD两者作为存储，即可以利用内存快速存取数据的优点，也可以利用SSD的大容量的优点，在保证吞吐量的情况下，可以处理兆字节数据。下面将详细描述存储系统的设计思路以及使用技术。
* **Storage Management**：as实现了一个混合模型（SSD+DRAM），***其中索引纯粹在内存中（不是持久的），数据只在持久存储（SSD）上***，直接从磁盘读取。访问索引不需要磁盘I/O，这使得性能可以预测。这种设计是可行的，因为无论是随机的还是顺序的，SSD中I/O的读取延迟特性都是相同的，对于这种模型，可以使用第3.1.2节中，定时维护知识点描述的优化，来避免磁盘扫描重建索引的系统开销。更新记录时，先从磁盘（这里说的磁盘指代SSD）读取记录的副本，加载到内存中，在内存中执行更新，最后将更新的副本写入写缓冲区，写缓冲区在填满时被刷新到磁盘中。读取单元rblocks的大小为128字节。这就增加了可寻址空间，并且可以容纳一个最大为2TB的存储设备。以wblock为单位写入（可配置，通常为1MB）可优化磁盘寿命。（***rblocks、wblock参考图3-1，待补充***）。as基于一个健壮的哈希函数，在多个存储设备之间分配数据，允许并行访问数据，同时避免任何热点区域的产生。
* **Defragmentation（碎片整理）**：as使用一个日志结构的文件系统，该系统具有一种copy-on-write（写时复制机制）。它通过连续运行后台碎片整理进程，来回收空间。每个设备存储一个块（wblock）的映射以及与每个块的填充因子（使用系数）相关信息。块的填充因子有效记录块的使用情况。在启动时，这些信息被加载，并在每次写入时保持更新。当块的填充因子低于某个阈值时，该块将成为碎片整理的候选块，然后排队等待碎片整理进程处理它。对块进行碎片整理时，先读取有效记录，再将其移动到新的写缓冲区，当该写缓冲区满时，将刷新到磁盘。为了避免新的写入和旧的写入混合，as维护两个不同的写入缓冲队列，一个用于正常的客户端写入，另一个用于碎片整理时存放移动的记录。在系统运行时，这些候选块不断地被送入这个队列进行碎片整理，这增加了磁盘的写入速率。设置一个非常高的填充因子阈值（通常为50%）会增加设备的烧坏率，而设置较低会降低空间利用率。根据可立即被写入占用的可用磁盘缓冲区空间（完全清空wblock），调整碎片整理阀值以确保有效的空间利用率。
* **Performance and Tuning（性能和调整）**：性能和调整主要分为以下两方面： ***（1）Post Write Queue***：as不维护LRU页面缓存，而是维护所谓的写后队列。队列存储最近写入的（LRW）数据缓存。有许多应用程序模式，其中写入的数据会立即以时间位置进行读取。复制服务提供了最近更改的记录，具有这种行为特征。此外，该缓存不需要额外的缓存空间，在用于对磁盘执行写操作的写块缓存上方。后写队列提高了缓存命中率，并减少了存储设备上的I/O负载。 ***（2）Shadow Device（影子设备）*** ：在云环境中，不同的设备具有不同的I/O和延迟特性。例如，在AmazonEC2实例中，可以通过使用临时磁盘（在进程重新启动之后生存，而不是实例重新启动）和EBS（在进程和实例重新启动之后生存）在不同程度上实现持久性。Ephemeral devices（临时磁盘）很快，直接连接到实例。相比之下，EBS设备速度较慢，并通过网络连接到实例。为了在这样的系统中扩大规模，as采用了一种阴影设备技术，在这种技术中，写入同时应用于本地的临时存储，并远程应用于EBS。但是，读取总是从临时存储中完成的，它们可以支持比EBS更高的随机访问率。