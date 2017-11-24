## 传播方式
  * 单播: 只有一个发送方和一个接收方
  * 组播: 单个发送方对应一组接收方
  * 任意播: 任意发送方对应一组较为接近的接收方
  
## PIM
  * 由IDMR工作组设计
  * 不依赖于特定的单播路由协议，它可以利用各种单播路由协议建立的单播路由表完成
  * 在Internet范围内支持SPT和共享树，并使两者之间灵活切换，因而集中了他们的优点
  * PIM-DM(Dense-Mode): PIM密集模式，RFC3973
  * PIM-SM(Sparse-Mode): PIM稀疏模式，RFC2362  
  
## PIM-SM (RFC2362) 
  * A protocol for efficiently routing multicast groups that may span wide-area (and inter-domain) internets
  * Not depend on any particular unicast routing prrotocols
  * Designed to support sparse groups
  * A Designated Router (DR) send periodic Join/Prune message toward a group-specific Rendezvous Point (RP) for each group
  
  Route Entry refer to the state maintained in a router to represent the distribution tree. A Route entry may include such fields as sourse address, the group address, the incoming interface and the list of outgoing interfaces, timers, flag bits, etc
  
  When a data source first sends to a group, its DR unicast Register messages to the RP with the source's data packets encapculated within:
  * If the data rate is high, the RP can send source-specific Join/Prune messages back towards the source and the source's data packet will follow the resulting state and travel unencapsulated to the RP.
  * If the date rate warrants it, routers with local receiver can join a source-specific, shortest path, distribution tree and prune this source's packets off of the shared RP-centered tree.
  * For the low data rate sources, neither the RP, nor the last-hop routers need join a source-specific shortest path tree and data packets can be delivered via the shared, RP-tree
  
#### Local hots joining the group
  A host (R) conveys its membership information through the Internet Group Management Protocol ([IGMP]()).  
  From this point on we refer to such a host as receiver R of the group G.
  
  When a DR gets a membership indication from IGMP for a new group, G, the DR looks up the associated RP. The DR creates a wildcard multicast route entry for the group (\*, G), if there is no more specific match for a particular source, the packet will be forwarded accroding to this entry.
  
#### Establishing the RP-rooted shared tree
  Triggered by the (\*, G) state, the DR creates a Join/Prune message with the RP address in its join list and the wildcard bit (WC-bit) and RP-tree bit (RPT-bit) set to 1.
  * The WC-bit indicates that any source may match and be forwarded according to this entry if there is no longer match.
  * The RPT-bit indicates that this join is being sent up the shared, RP-tree.
  * The Prune list is left empty
  
  when the RPT-bit is set to 1, it indicates that the join is associate with the shared RP-tree and therefore the Join/Prune message is propagated along the RP-tree. The term RPT-bit is used to refer to both the RPT-bit flags associate with route entries, and the RPT-bit included in each encoded address in a Join/Prune message.
  When the WC-bit is set to 1, it indicates that the address is an RP and the downstream receiver expect to receive packet from all sources via this (shared tree) path.

  Each upstream router creates or updates its multicast route entry for (\*, G) when it receives a Join/Prune with the RPT-bit and WC-bit set.
  
#### Hosts sending to a group
  When a host starts sending multicast data packets to a group, initially its DR must deliver each packet to the RP for distribution down the RP-tree.  
  The Sender's DR initially encapsulates each data packet in a Register message and unicast it to the RP for that group.
  The RP decapsulates each Register message and forwards the enclosed data packet natively to downstream members on the shared RP-tree.
  
  If the data rate of the source warrants the use of a source-specific shortest path tree (SPT), the RP may construct a new multicast route entry that specific to the source.
  
  The RP triggers Register-Stop messages in response to Registers, if the RP has no downstream receivers for the group (or for that particular source), or if the RP has already joined the (S, G) tree and is receiving the data packets natively.
  
  Each source's DR maintains, per (S, G), a Register-Suppression -timer. The Register-Suppression-timer is started by the Register-Stop message, upon expiration, the source's DR resumes sending data packets to the RP, encapsulated in the Register message.
  
#### Switching from shared tree (RP-tree) to shortest path tree (SP-tree)
  The recommended policy is to initiate the switch to the SP-tree after receiving a significant number of data packets during a specified time interval from a particular source. 
  
  To realize this policy the route can monitor data packet from sources for which it has no source-specific multicast route entry and initiate such an entry when data rate exceeds the configured threhold.
  
  [PIM协议的图示](http://download.csdn.net/download/boostc/10118800)
  
  When the (S, G) entry is activated (and periodically so long as the state exists), a Join/Prune message is sent upstream towards the source, S, with S in the join list.

## PIM-SM (中文文档)
  协议无关组播，就是在做RPF检查以及发送特定的协议单播报文的时候利用单播路由表，而和具体采用何种单播路由协议并没有关系，该协议也不保持自己的路由表，为IP组播提供路由的单播路由协议可以是静态路由、RIP、OSPF、IS-IS、BGP等。

#### 概述
  * PIM-SM是稀疏模式的组播路由协议，主要用于成员分布相对分散、范围较广、大规模的网络
  * 协议假设：当组播源开始发送组播数据时，域内的所有节点都不需要接收数据
  * 该模型转发的核心任务就是构造并维护一颗单向共享树，共享树选择PIM中某一路由器作为公用根节点，成为汇聚点RP(Rendezvous Point)。组播数据通过RP沿共享树向接受者转发
  * 接受者发现DR(Designated Router)，由DR创建(\*, G)项并以Join消息发送到RP
  * 组播源同样发现DR(第一跳路由器)，并通过DR在RP上注册源信息
  * 同时包含两种树:共享树、原路径树
  * RPF检查根据树的类型进行:
    * 使用共享树进行数据接收转发时，使用RP地址作为检测地址
    * 使用原路径树进行数据接收转发时，使用组播源地址作为检测地址
  * BSR (BootStrap Router)在PIM-SM网络启动后，负责收集网络内的RP信息，为每个组播组选举RP，然后将RP集(即组-RP的映射数据库)发布到整个网络，PIM-SM域内只有一台BSR，可同时存在多台候选BSR

#### 关键点:
  * PIM-SM的基本原理
  * 共享树的加入和源的注册过程
  * RPT和SPT的切换

#### 字段解释
  版本|类型|保留|检验和
  ----|---|---|-----
  3|7|15|31
  
  * **_版本_**: 当前为2
  
  * **_类型_**：
    * 0: hello
    * 1: 注册 (only SM)
    * 2: 停止注册 (only SM)
    * 3: 加入/剪枝
    * 4: Bootstrap (only SM)
    * 5: Assert
    * 6: 嫁接 (only DM)  
    * 7: 嫁接回应 (only DM)
    * 8: 候选RP公告 (only SM)
  
  * **_保留_**
  
  * **_检验和_**
  
#### 协议机制
  * **邻居发现**: 组播路由需要使用Hello消息来发现邻居，并维护邻居关系
  * **DR选举**: 借助Hello消息可以为共享网络(如Ethernet)选举DR，DR将作为本网段中组播消息的唯一转发者，无论是和组播源连接的网络，还是和接收者连接的网络，只要网络为共享媒介都需要选举DR，接收者侧的DR向RP发送Join加入消息，组播源侧DR向RP发送Register注册消息
  * **BSR的选举**:
    * 如果域内只有一台C-BSR，该台路由器就是该域内的BSR
    * 如果域内存在多台C-BSR，则拥有最高优先级的路由器为BSR
    * 如果域内存在多台拥有相同优先级的C-BSR，则拥有最高IP地址的路由器为BSR
  * **RP发现**: 组播网络中，担当共享树根节点的节点就是RP，共享树里所有组播流都通过RP转发到接收者，RP可以负责几个或者所有组播组的转发，所以网络中可以有一个到多个RP负责不同的组播组:
    * 在DR和叶子路由器以及组播数据流将要经过的所有路由器上手工指定RP
    * 启动Bootstrap协议，利用自举机制动态选举RP
  * **利用BootStrap协议选举RP**:
    * 如果PIM-SM域内只有一个C-RP(Candidat-RP)，那么这个节点就是域内的RP
    * 如果域内存在多个C-RP并拥有不同的优先级，那么优先级最高额将会被选举为RP
    * 如果域内存在多个C-RP并都拥有相同的优先级时，则依靠Hash算法算出的树枝决定RP，数值最大的成为RP，Hash算法参数：组地址、掩码长度、C-RP地址
    * 如果域内存在多个C-RP并且拥有相同的优先级和Hash数值，则拥有最高IP地址的C-RP为该域的RP
  * **RP与BSR的关系**:
    * C-RP将声明发送给BSR，C-RP通过单播周期发送通告，BSR在RP集存储所有的C-RP通告
    * BSR周期性地向所有PIM路由器发送BSR消息，BSR消息包含整个RP-set和BSR地址，消息一跳一跳地自BSR向整个网络泛滥
    * 所有路由器使用收到的RP-set来确定RP，所有路由器使用相同的RP选择算法，选择的RP也是一样的
  * **RPT共享树加入**: IGMP, (\*, G)
  * **组播源注册**: (S, G)注册, (S, G)加入, 源树，共享树、数据流，**停止注册**， **组播流转发**
  * **SPT切换**: RPT向SPT切换，切换后组播的转发，切换后的剪枝，SPT的切换条件:
    * 当信息吞吐率超过预定值，PIM-SM就会从共享树切换到组播源路径树
    * Quidway系列设备中(VRP平台)，缺省情况下，连接接收者的路由器在探测到组播源之后，变立即加入最短路径树(源树)，即从RPT向SPT切换

#### 评价
  * 对于稀疏、密集模式应用都很有效
  * 数据流仅沿“加入”的分支向下发送
  * 可以根据流量条件等动态的切换到源树
  * 和具体的单播路由协议无关
  * 是域内组播路由的基础，和MGBP、MSDP共同结合使用可以完成跨域的组播

## PIM-SM (RFC4601)

#### Introduction
  PIM-SM v2  
  RFC 2117 -> RFC 2362 -> IETF Standards Track
  
#### Terminology
  * **Rendezvous Point (RP)**: An RP is a router that has been configured to be used as root of the non-source-specific distribution tree for a multicast group. Join messages from receivers for a group are sent towards the RP, and data from senders is sent to the RP so that receivers can discover who the senders are and start to receive traffic destined for the group.
  * **Designated Router (DR)**: A shared-media LAN like Ethernet may have multiple PIM-SM routers connected to it. A single one of these routers, the DR, will act on behalf of directly connected hosts with respect to the PIM-SM protocol. A single DR is elected per inetrface (LAN or otherwise) using a simple election process.
  * **MRIB**: Multicast Routing Information Base. this is multicast topology table, which is typically derived from the unicast routing table, or routing protocols such as Multiprotocol BGP (MBGP) that carry multicast-specific topology information. In PIM-SM, the MRIB is used to decide where to send Join/Prune messages. A secondary function of the MRIB is to provide routing metrics for destination address; these metrics are used when sending and processing Assert messages.
  * **RPF Neighbor**: RPF stands for "Reverse Path Forwarding". The RPF Neighbor of a router with respect to an address is the neighbor that the MRIB an indicates should be used to forward packets to that address. In the case of a PIM-SM multicast group, the RPF neighbor is the router that Join message for that group would be directed to, in the absence of modifying Assert state.
  * **TIB**: Tree Information Base. This is the collection of state at PIM router that has been created by receiving PIM Join/Prune messages, PIM Assert messages, and Internet Group Management Protocol (IGMP) or **Multicast Listener Discovery (MLD)** information from local hosts. It essentially stores the state of all multicast distribution trees at that router.
  * **MFIB**: Multicast Forwarding Information Base. The TIB holds all the state that is necessary to forward multicast packets at the router. However, although this specification defines forwarding in terms of TIB, to actually forward packets using TIB is very inefficient. Instead, a real router implementation will normally build an efficient MFIB from TIB state to perform forwarding. How this is done is implementation-specific and is not discussed in this document.
  * **Upstream**: Towards the root of the tree. The Root of tree may be either the source or the RP, depending on the context.
  * **Downstream**:  Away from the root of the tree.
  * **GenID**: Generation Identifier, used to detect reboots.
  * **PMBR**: PIM Multicast Border Router, joinin a PIM domain with another multicast domain.
  
#### PIM-SM Protocol Overview
  PIM relies on an underlying topology-gathering protocol to populate a routing table with routes. This table is called the MRIB. In contrast to the unicast RIB, which specifies the next hop that a data packet would take to get to some subnet, the <RIB gives reverse-path information and indicates the path that a multicast data packet would take from its origin subnet to the router that has the MRIB.
  
  Like all multicast routing protocols that implement the service model from RFC 1112, PIM-SM must be able to route data packet from sources to the receivers **without either the sources or receivers knowing a priori of the existence of the others**. This is essetially done in three phases, although as senders and receivers may come and go at any time, all three phases may occur suimultaneously.
  
  * Phase One: RP Tree
    * A multicast receiver expresses its interest in receiving traffic destined for a multicast group
    * One of the receiver's local routers is elected as the DR for that subnet
    * Then DR sends a PIM Join message towards the RP for that multicast group
    * This Join message is know as a (\*, G) Join because it joins group G for all sources to that group
    * Eventually, the (\*, G) Join either reaches the RP or reaches a router that already has (\*, G) Join state for that group
    * This is known as the **RP tree (RPT)**, and is also known as **the shared tree** because it is shared by all sources sending to that group
    * Join messages are **resent** periodically so long as the receiver remains in the group
    * When all receivers on a leaf-network leave the group, the DR will send a **PIM (\*, G) Prune message** towards the RP for that multicast group. However, if the Prune message is not sent for any reason, the **state will eventually time out**.
    * The **sender's local router (DR)** takes those data packets, unicast-encapsulates them, and sends then directly to the RP. The **RP receivers these encapsulated data packets**, decapsulates them, and forwards them onto the shared tree. The packets then follow the (\*, G) multicast tree state in the routers **on the RP tree**, beging replicated whereever the RP Tree beaches, and evetually reaching all the receivers for that multicast group. The process of encapsulating data packets to the RP is called **registering**, and the encapsulation packets are known as **PIM Register packets**.
    
  * Phase Two: Register Stop
    * Register-encapsultion of data packets is inefficient for two reasons: encapsulation and decapsulation; travel all the way to the RP and then back down the shared tree my result in packets traveling a relatively long distance to reach receivers that are close to the sender.
    * Although Register-encapsulation may continue indefinitely, for theses ressons, the RP will normally choose to switch to native forwarding. 
    * When the RP receives a register-encapsulated data packet from source S on group G, it will normally initiate an (S, G) source-specific Join towards S. This Join message travels hop-by-hop towards S, instantiating (S, G) multicast tree state in the routers along the path. (S, G) multicast tree state is used only to forward packets for group G if those packets come from source S.
    * Eventually the Join message reaches S's subnet or router that already has (S, G) multicast tree state, and then packets from S start to flow following the (S, G) tree state towards the RP. These data packets may also reach routers with (\*, G) along the path forwards the RP, **if they do**, they can **shortcut onto the RP tree at this point**.
    * While the RP is in the process of joining the source-specific tree for S, the data packets will continue being encapsulated to the RP. When packets from S also start to arrive natively at the RP, the RP will be reveiving two copies of each of these packets. At this point, the RP starts to **discard the encapsulated copy of the these packets**, and it sends a Register-Stop messages back to S's DR to prevent the DR from unnecessarily encapsulating the packets.
    * Where the two trees (source-specific tree and RP tree) intersect, traffic may transfer from the source-specific tree to the RP tree and thus avoid taking a long detour via the RP.
    
  * Phase Three: Shortest-Path Tree
    * To obtain lower latencies or more efficient bandwidth utilization, **a router on the receiver's LAN, typically the DR** may optionally initiate a transfer from the shared tree to a source-specific shortest-path tree (SPT)
    * To do this, it **issues an (S, G) Join towards S**. This instantiates state in the routers along the path to S. Eventually, this join either reach **S's subnet or reaches a router that already has (S, G) state**. When this happens, data packets from S start to flow following the (S, G) state until they reach the receiver.
    * At this point, the receiver (or a router upstream of the receiver will be receiving two copies of the data: **one from the SPT and one from the RPT**. When the first traffic starts to arrive from the SPT, the DR or upstream router starts to drop the packets for G from S that arrive via the RP tree. In addition, it sends an **(S, G) Prune** message towards the RP. This is known as (S, G, rpt) Prune. The towards the RP indicating that traffic from S for G should NOT be forwarded in this direction. The Prune is **propagated** until it reaches the RP or a router that still needs the traffic from S for other receivers.
    * In addition, the RP is receiving the traffic from S, but this traffic is no longer reaching the receiver along the RP tree. **As far as the receiver is concerned, this is the final distribution tree**.

#### Source-Specific Joins
  IGMPv3 permits a receiver to join a group and specify that it only wants to receive traffic for a group if that traffic comes from a particular source. If a receiver does this, and **no other receiver on the LAN requires all the traffic for tha group**, then DR may omit performing a (\*, G) join to set up the shared tree, and instead issue a source-specific (S, G) join only.
  
  The range of multicast addresses **from 232.0.0.0 to 232.255.255.255** is currently set aside for source-specific multicast in IPv4. For groups in this range, receivers should only issue source-specific IGMPv3 joins. If a PIM router receivers a non-source-specific join for a group in this range, it should ignore it.
  
#### Source-Specific Prune  
  IGMPv3 also permits a receiver to join a group and specify that it only wants to receive traffic for a group is that traffic does not come from a specific source or sources. In this case, **the DR will perform a (\*, G) join as normal, but may combine this with an (S, G, rpt) prune** for each of the sources that receiver does not wish to receive.
  
#### Multi-Access Transmit LANs 
  * The overview so far has concerned itself with point-to-point transit links. However, using multi-access LANs such as Ethernet for transit is not uncommon.
  * All of these problems are caused by there being more than one upstream router with join state for the group or source-group pair. PIM does not prevent such duplicate joins from occurning, instead, **when duplicate data packets appear on the LAN from different routers, these routers notice this and then elect a single forwarder**. This election is performed using PIM Assert message, which resolve the problem in favor of the upstream router has (S, G) state, or, if nethier or bother router has (S, G) state.
  
#### RP Discovery
  * PIM-SM routers need to know the address of the RP for each group for which they have (\*, G) state. This address is obtained automatically
  * Bootstrap mechanism, or through static configurtion
  * All the routers in the domain that are configured to be candidats to be RPs periodically unicast their candidacy to the **BSR**
  * From the candidates, the BSR picks an RP-set, and periodically announces this set in a **Bootstrap message**
  * to **map a group to an RP**, a router hashes the group address into RP-set using an **order-preserving hash function** (one that minimizes changes if the RP-Set Changes). The resulting RP is the one that it uses as the Rp for that group.
  
#### Protocol Specification

###### PIM Protocol State
  * The **protocol state** that a PIM implementation should maintain in order to function correctly. We term this state the Tree Information Base (TIB), as it holds the state of all the multicast distribution trees at this router. Most implementation will use this state to build a **multicast forwarding table**, which would then be updated when the relevant state in TIB changes.
  
  * We divide TIB state into four sections:
    * (\*, \*, RP) state: state that maintains per-RP trees, for all groups served by a given RP
    * (\*, G) state: state that maintains the RP tree for G
    * (S, G) state: state that maintains a source-specific tree for source S and group G
    * (S, G, rpt) state: state that maintains source-specific information about source S on the RP tree for G.
    
  * general purpose states:
    * Non-group-specific state
      * Effective Override Interval
      * Effective Propagation Delay
      * Suppression state: one of {"Enable", "Disable"}
    * Neighbor state
      * Information from neighbor's Hello
      * Neighbor's GenID
      * Neighbor Liveness Time (NLT)
    * Designate Router state
      * DR's IP address
      * DR's DR Priority
      
  * (\*, \*, RP) state:
    * PIM (\*, \*, RP) Join/Prune State
      * State: One of {"NoInfo" (NI), "Join" (J), "Prune Pending" (PP)}
      * Prune-Pending Timer (PPT)
      * Join/Prune Expiry Timer (ET)
    * Not interface specific
      * Upstream (\*, \*, RP) Join/Prune state: one of {"NotJoined(*, *, RP)", "Joined(*, *, RP)"}
      * Upstream Join/Prune Timer (JT)
      * Last RPF Neighbor towards RP that was used
      
  * (\*, G) state:
    * Local Membership
      * State: one of {"NoInfo", "Include"}
    * PIM (\*, G) Join/Prune State
      * State: one of {"NoInfo" (NI), "Join" (J), "Prune-Pending" (PP)}
      * Prune-Pending Timer (PPT)
      * Join/Prune Expiry Timer (ET)
    * (\*, G) Assert Winner State
      * State: one of {"NoInfo" (NI), "I lost Assert" (L), "I won Assert" (W)}
      * Assert Timer (AT)
      * Assert winner's IP Address (AssertWinner)
      * Assert winner's Assert Metric (AssertWinnerMetric)
    * Not interface specific
      * Upstream (\*, G) Join/Prune state: one of {"NotJoined(\*, G)", "Joined(\*, G)"}
      * Upstream Join/Prune Timer (JT)
      * Last RP Used
      * Last RPF Neighbor towards RP that was used
      
  * (S, G) state:
    * Local Membership
      * State: one of {"NoInfo", "Include"}
    * PIM (S, G) Join/Prune State
      * State: one of {"NoInfo" (NI), "Join" (J), "Prune-Pending" (PP)}
      * Prune-Pending Timer (PPT)
      * Join/Prune Expiry Timer (ET)
    * (S, G) Assert Winner State
      * State: one of {"NoInfo" (NI), "I lost Assert" (L), "I won Assert" (W)}
      * Assert Timer (AT)
      * Assert winner's IP Address (AssertWinner)
      * Assert winner's Assert Metric (AssertWinnerMetric)
    * Not interface specific
      * Upstream (S, G) Join/Prune State: one of {"NotJoined(S, G)", "Joined(S, G)"}
      * Upstream (S, G) Join/Prune Timer (JT)
      * Last RPF Neighbor towards S that was used
      * SPTbit (indicates (S, G) state is active)
      * (S, G) Keepalive Timer (KAT)
    * Additional (S, G) state at the DR
      * Register state: one of {"Join" (J), "Prune" (P), "Join-Pending" (JP), "NoInfo" (NI)}
      * Register-Stop timer
    * Additional (S, G) state at the RP
      * PMBR: the first PMBR to send a Register for this source with the Border bit set

  * (S, G, rpt) State
    * Local Membership
      * State: one of {"NoInfo", "Exclude"}
    * PIM (S, G, rpt) Join/Prune State
      * State: one of {"NoInfo", "Pruned", "Prune-Pending"}
      * Prune-Pending Timer (PPT)
      * Join/Prune Expiry Timer (ET)
    * Upstream (S, G, rpt) Join/Prune State
      * State: one of {"RPTnotJoined(G)", "NotPruned(S, G, rpt)", "Pruned(S, G, rpt)"}
      * Override Timer (OT)
      
###### State Summarization Macros
  Using this state, we define the following "macro" definitions, which we will use in the description of the state machines and  pseudocode in the following sections.
  
  The most important macros are those that define the outgoing interface list (or "olist") for the relevant state. An olist can be "immediate" or "inherited"
  
  immediate_olist(\*, \*, RP) = joins(\*, \*, RP)  
  immediate_olist(\*, G) = joins(\*, G) (+) pim_include(\*, G) (-) lost_assert(\*, G)  
  immediate_olist(S, G) = joins(S, G) (+) pim_include(S, G) (-) lost_assert(S, G)  
  
  inherited_olist(S, G, rpt) = (joins(\*, \*, RP(G)) (+) joins(\*, G) (-) prunes(S, G, rpt)) (+) (pim_include(\*, G) (-) pim_exclude(S, G)) (-) (lost_assert(\*, G) (+) + lost_assert(S, G, rpt))  
  inherited_olist(S, G) = inherited_olist(S, G, rpt) (+) joins(S, G) (+) pim_include(S, G) (-) lost_assert(S, G)

  pim_include(\*, G)  
  pim_include(S, G)  
  pim_exclude(S, G)  
  joins(\*, \*, RP)  
  joins(\*, G)  
  joins(S, G)  
  prune(S, G, rpt)  
  lost_assert(\*, G)  
  lost_assert(S, G, rpt)  
  lost_assert(S, G)
  
#### Data Packet Forwarding Rules
  The PIM-SM packet forwarding rules are defined below in pseudocode:
  1. iif is the incoming interface of the packet
  1. S is the source address of the packet
  1. G is the destination address of the packet
  1. RP is the address of the Rendezvous Point for this group
  1. RPF_interface(S) is the interface the MRIB indicates would be used to route packets to S
  1. RPF_interface(RP) is the interface the MRIB indicates would be used to route packets to RP, expect at the RP when it is the decapsulation interface
  
  ```c++
  if (DirectlyConnected(S) == TRUE AND iif == RPF_interace(S)){
      set KeepaliveTimer(S, G) to Keepalive_Period
      \\ Note: a register state transition or UpstreamJPState(S, G)
      \\ tramsition may happen as a result of restarting
      \\ KeepaliveTimer, and must be dealt with here
  }
  
  if (iif == RPF_interface(S) AND UpstreamJPState(S, G) == Joined AND inherited_olist(S, G) != NULL){
     set KeepaliveTimer(S, G) to Keepalive_Period
  }
  
  Update_SPTbit(S, G, iif)
  oiflist = NULL
  
  if (iif == RPF_interface(S) AND SPTbit(S, G) == TRUE){
      oiflist = inherited_olist(S, G)      
  } else if (iif == RPF_interface(RP(G)) AND SPTbit(S, G) == TRUE){
      oiflist = inherited_olist(S, G, rpt)
      CheckSwitchToSpt(S, G)
  } else {
      // Note: RPF check failed
      // A transition in an assert FSM may cause an Assert(S, G)
      // or Assert(*, G) message to be sent out interface iif
      if (SPTbit(S, G) == TRUE AND iif is in inherited_olist(S, G)){
          send Assert(S, G) on iif
      } else if (SPTbit(S, G) == FALSE AND iif is in inherited_olist(S, G, rpt)){
          send Assert(*, G) on iif
      }
  }
  
  oiflist = oiflist (-) iif
  forward packet on all interfaces in oiflist
  ```
  
###### Last-Hop Switchover to the SPT    
  In the sparse-mode PIM, last-hop routers join the shared tree towards the RP. One traffic from sources to joined groups arrives at a last-hop router, it has the option of switching to receive the traffic on a shortest path tree (SPT).
  
  ```c++
  void CheckSwitchToSpt(S, G)
  {
      if ((pim_include(*, G) (-) pim_exclude(S, G) (+) pim_include(S, G) != NULL)
          AND SwitchYoSptDesired(S, G)){
          // Note: Restarting the KAT will result in the SPT switch 
          set KeepaliveTimer(S, G) to Keepalive_Period
      }
  }
  ```
  
###### Setting and Clearing the (S, G) SPTbit
  The (S, G) SPTbit is used to distinguish whether to forward on (\*, \*, RP)/(\*, G) or on (S, G) state. When switching from the RP tree to the source tree, there is a transition period when data is arriving due to upstream (\*, \*, RP)/(\*. G). 
  
  ```c++
  void Update_SPTbit(S, G, iif)
  {
      if (iif == RPF_interface(S)
          AND JoinDesired(S, G) == TRUE
          AND (DirectlyConnected(S) == TRUE
               OR RPF_interface(S) != RPF_interface(RP(G))
               OR inherited_olist(S, G, rpt) == NULL
               OR RPF'(S, G) == RPF'(*, G)
               AND RPF'(S, G) != NULL)
          OR (I_Am_Assert_Loser(S, G, iif))){
          Set SPTbit(S, G) to TRUE
      }
  }
  ```
  
#### Designated Router (DR) and Hello Messages
  Because the distinction between LANs and point-to-point interfaces can sometimes be blurred, and because routers may also have multicast host functionality, the PIM-SM specification makes no distincton between two. Thus, DR election will happen on all interfaces, LAN or otherwise.
  
  DR election is performed using Hello messages. Hello messages are also the way that option negotiation take place in PIM, so that additional functionality can be enabled, or parameters tuned.
  
###### Sending Hello Messages
  PIM hello messages are sent periodically on each PIM-enabled interface. They allow a router to learn about the neighboring PIM routers on each interface. Hello message are also the mechanism used to elect a Designate Router (DR), and to negotiate additional capabilities. A router must record the Hello information received from each PIM neighbor.
  
  Hello message MUST be sent on all active interfaces, including physical point-to-point links, and are multicast to the "ALL-PIM-ROUTERS" group address.
  
  A **per-interface Hello Timer (HT(I))** is used to trigger sending Hello messages on each active interface. The Hello message of that interface is set to a random value between 0 and Triggered_Hello_Delay. This prevents synchronization of Hello messages if multiple routers are powered on simultaneously.
  
  Note that neighbor will not accept Join/Prune or Assert messages from a router unless they have first heard a Hello messages from that router. Thus, if a router needs to send a Join/Prune or Assert message with the currently on an interface on which it has not yet sent a Hello message with the currently configured IP address.
  
  **Dr_priority** option allows a network administrtor to give preference to a particular router in the DR election process.
  
  **Generation_Identifire (GenID)** option should be included in all Hello messages. The GenID option contains a randomly generated 32-bit value that is regenerated each time PIM forwarding is started or restarted on the interface, including when the router itself restarts. When a **Hello message with a new GenID** is received from a neighbor, and old Hello information about that neighbor should be discarded and supreseded by the information from a new Hello message. This may cause a new DR to be chosen on that interface.
  
  **The LAN Prune Delay** option should be include in all Hello messages sent on multi-access LANS. This option advertises a router's capability to use values other than the defaults for the **Propagation_Delay** and **Override_Interval**, which affect the setting of the Prune-Pending, Upstream Join, and Override Timers.
  
  **The Address List Option** advertises all the secondary addresses associated with the source interface of the router originating the message. The option must be included in all Hello messages if there are secondary addresses associated with the source interface and may omitted if no secondary addresses exist.
  
  When a Hello message is receiving from a new neighbor, or a Hello message with a new GenID is received from an existing neighbor, a new Hello messagee should be send on this interface after a randomized delay between 0 and **Triggered_Hello_Delay**. This triggered message need not change the timing of the scheduled periodic message.
  
  Before an interface **goes down or changes primary IP address**, a Hello message with **zero HoldTime** should be sent immediately (with the old IP address if the IP address changed). This will cause PIM neighbors to remove this neighbor (or its old IP address).
  
###### DR Election  
  When a PIM hello message is received on interface I, the following information about the sending neighbor is recorded:
  * neighbor.interface
  * neighbor.primary_ip_address
  * neighbor.genid
  * neighbor.dr_priority
  * neighbor.dr_priority_present
  * neighbor.timout: Neighbor Libeness Timer, is reset to **Hello_Holdtime** whenever a Hello message is received containing a Holdtime option, or to **Default_Hello_Holdtime** if the Hello message does not contain the Holdtime option. _Neighbor state is deleted when the neighbor timeout expires._
  
  ```c++
  host DR(I)
  {
      dr = me
      for each neighbor on interface I {
          if (dr_is_better(neighbor, dr, I) == TRUE){
              dr = neighbor
          }
      }
      
      return dr
  }
  
  bool dr_is_better(a, b, I)
  {
      if (there is a neighbor n on I for which n.dr_priority_present is false){
          return a.primary_ip_address > b.primary_ip_address
      } else {
          return ((a.dr_priority > b.dr_priority) OR (a.dr_priority == b.dr_priority AND a.primary_ip_address > b.primary_ip_address))
      }
  }   
  
  bool I_am_DR(I)
  {
      return DR(I) == me
  }
  ```
  
  A router's **idea of the current DR on an interface** can change when a PIM Hello message is received, when a neighbor times out, or when a router's own DR priority changes.
  
  If the router **becomes the DR or ceases to be the DR**, this will normally cause the DR Register state machine to change state.
  
###### Reducing Prune Propagation Delay on LANs
  In addition to the information recorded for the DR Election, the following per neighbor information is **obtained from the LAN Prune Delay Hello Option**:
  * neighbor.lan_prune_delay_present
  * neighbor.tracking_support
  * neighbor.propagation_delay
  * neighbor.override_interval
  
  The information provided in the LAN Prune Delay option is not used unless all neighbors on a link advertise the option.
  
  ```c++
  bool lan_delay_enabled(I)
  {
      for each neighbor on interface I{
          if (neighbor.lan_prune_delay_present == false){
              return false
          }
      }
      
      return true
  }
  ```
  
  Propagation Delay expresses the expected message propagation delay on the link and should be configurable by the system administrator. It is used by **upstream routers** to figure out how long they should wait for a Join override message before pruning an interface.
  
  When all routers on a link are in a position to negotiate a Propagation Delay different from the default, the **largest** value from those advertised by each neighbor is chosen.
  
  ```c++
  time_interval Effective_Propagation_Delay(I)
  {
      if (lan_delay_enabled(I) == false){
          return Propagation_delay_default
      }
      
      delay = Propagation_Delay(I)
      for each neighbor on interface I {
          if (neighbor.propagation_delay > delay){
              delay = neighbor.propagation_delay
          }
      }
      
      return delay
  }
  ```
