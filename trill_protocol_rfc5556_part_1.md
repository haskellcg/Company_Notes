# Transparent Interconnection of Lots of Links (TRILL): Problem and Applicability Statement

## Abstract
  Current IEEE 802.1 LANs use spanning tree protocols that have a number of chanllenges. These protocols need to strickly avoid loops. even temporary ones, during route propagation, because of the lack of header loop detection support. Routing tend not to take full advantage of alternate paths, or even non-overlapping pairwise paths (in the case of spanning trees).

## 1. Introduction
  Conventional Ethernet networks -- known in the Internet as Ethernet link subnet -- have a number of attractive features, allowing hosts and routers to relocate within the subnet without requiring renumbering, and support automatic configuration. **The basis of simplicity of these subnets is the spanning tree, which although simple and elegant, can have substantial limitations**.

  With spanning trees, the bandwidth across the subnet is limited because traffic flows over a subnet of links forming a single tree -- or, with the latest version of the protocol and significant additional configuration, over a small number of superimposed trees.

  The alternative to an Ethernet link subnet is often a network subnet. Network subnets can use link-state routing protocols that allow traffic to traverse least-cost paths rather than being aggregated on a spanning tree backbone, providing higher aggregate capacity and more resistance to link failures.

  Unfortunately, IP -- the dominant network layer technology -- requires that hosts be renumbered when relocated in different network subnets, interrupting network and transport associations that are in progress during the transition.

## 2. The TRILL Problem
  Ethernet subnets have evolved from "thicknet" to "thinnet" to twisted pair with hubs to twisted pair with switches, becoming increasingly simple to wire and manage. Each level has corresponding toponogy restrictions; thicknet is inherently linear, whereas thinnet and hub-connected twisted pair have to be wired as a tree. Switches, added in IEEE 802.1D, allow network managers to avoid thinking in trees, where the spanning tree protocol finds a valid tree automatically; unfortunately, this additional simplicity comes with a number of associated penalties.

  The spanning tree often results in inefficient use of the link topology; traffic is concertrated on the spanning tree path. The addition in IEEE 802.1Q of support for multiple spanning trees helps a little, but the use of multiple spanning trees requires additional configuration, the number of trees is limited, and these defects apply within each tree regardless.

  The spanning tree protocol reacts to certain small topology changes with large effects on the reconfiguration of links in use. Each of these aspects of the spanning tree protocol can cause problems for current link-layer deployments.

### 2.1. Inefficient Paths
  The STP helps breaks cycles in a set of interconnected bridges, but it also can limit the bandwidth among that set and cause traffic to take circuitous paths. 

                              [A]
                             // \    [C]
                            //   \   / \\  [D]
                           //     \ /   \\ //
                          [B]=====[H]=====[E]
                            \     //      ||
                             \   //       ||
                              \ //        ||
                               [G]--------[F]
           Figure 1: Bridged subnet with spanning tree shown

  One possible spanning tree is shown by double lines.

                                  [A]
                                  1           [C]
                                 1              1
                                1                1
                              [B]1111111[H]121212[E]
                                         2       2
                                        2        2
                                       2         2
                                      [G]       [F]
         Figure 2: Traffic from A..C (1) and G..F (2) share a link

  The spanning tree limits the capacity of the resulting subnet.

                                  [A]
                                  1           [C]
                                 1              1
                                1                1
                              [B]1111111[H]111111[E]

                                      [G]2222222[F]
       Figure 3: Traffic from A..C (1) and G..F (2) with full routing

  There are a number of features of modern layer 3 routing protocols which would be beneficial if available at layer 2, but which cannot practically be integrated into the spanning tree system such as multipath routing. Layer 3 routing typically optimizes paths between pairs of enpoints based on a cost metric, conventionally based on bandwidth, hop count, latency, and/or policy measures.

### 2.2. Multipath Forwarding
  The discussion above assumes that all traffic flowing from one point to another follows a single path. Using spanning trees reduces aggregate bandwidth by forcing all such paths onto one tree, while modern routing causes such paths to be selected based on a cost metric. However, extensions to modern routing protocols enable even greater aggregate bandwidth by permitting traffic flowing from one endpoint to another to be sent over multiple, typically equal-cost paths.

  Multipathing typically spreads the traffic more evenly over the available physically links. The addition of multipathing to a routed network would typically result in only a small improvement in capacity for a network with roughly equal traffic between all pairs of nodes, because in that situation traffic is already fairly well dispersed. 

  Conversely, multipathing can produce a dranamic improvment in a routed network where the traffic between a small number of pairs of nodes dominates, because such traffic can -- under the right sirsumstances -- be spread over multiple paths that might otehrwise be lightly loaded.

### 2.3. Convergence and Safety
  The spanning tree is dependent on the way a set of bridges are interconnected, i.e., the link-layer topology. **Small changed in this topology can cause large changes in the spanning tree**.Changes in the spanning tree can take time to propagate and converge, especially for older versions fo STP.

  One possible case occurs when one of the branches connected to the root bridge fails, causing a large number of ports to block and unblock before the network reconverges.

  The original IEEE 802.1 spanning tree protocol can impose 30-second delays in re-establishing data connectivity after a topology change in order to be sure a new topology has stabilized and been fully propagated.

  The spanning tree protocol is inherently global to an entire layer 2 subnet; there is no current way to contain, partition, or otherwise factor the protocol into a number of smaller, more stable subnets that interact as groups. Contrast this with Internet routing, which includes both interdomain and intradomain variants, split to provide exactly that containment and scalability within a domain while allowing domains to interact freely independent of what happens within a domain.

### 2.4. Stability of IP Multicast Optimization
  Although it is a layer violation, it is common for high-end bridges to snoop on IP multicast control messages for the purpose of optimizing the distribution of IP multicast data and of those control messages. When such snooping and optimization is performed by spanning-tree-based bridges, it done at each bridge based on the traffic observed on that bridge's ports. Changes in topology may reverse or otherwise change the required forwarding ports of messages for a multicast group. 

  Bridges must relearn the correct multicast forwarding from the receipt of multicast control messages on new ports. Such control messages are sent to establish multicast distribution state and then to refresh it, sometimes at intervals of seconds. If a bridging topology change has occured during such intervals, multicast data maybe disdirected and lost.

  However, a solution based on link-state routing, for example, can form and maintain a global view of the multicast group membership and multicast router situation in a similar fashion to that in which it maintains a global view of the status of links. Thus, such a solution can adjust the forwarding of multicast data and control traffic immediately as it sees the LAN topology change.

### 2.5. IEEE 802.1 Bridging Protocols
### 2.6. Problems Not Addressed

## 3. Desired Properties of Solutions to TRILL
### 3.1. No CHange to Link Capabilities
### 3.2. Zero Configuration and Zero Assumption
### 3.3. Forwarding Loop Mitigation
### 3.4. Spanning Tree Management
### 3.5. Multiple Attachments
### 3.6. VLAN Issues
### 3.7. Operational Equivalence
### 3.8. Optimizations
### 3.9. Internet Architecture Issues

## 4. Applicability

## 5. Security Considerations

## 6. Acknowledgments

## 7. Informative References
