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
## PIM-SM(RFC2362) 
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
  
  When the (S, G) entry is activated (and periodically so long as the state exists), a Join/Prune message is sent upstream towards the source, S, with S in the join list.


[PIM协议的图示](http://download.csdn.net/download/boostc/10118800)
