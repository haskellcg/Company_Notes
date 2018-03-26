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
  There have been a variety of IEEE protocols beyond the initial shared-media Ethernet variant, inluding:
  * 802.1D - added bridges and a spanning tree protocol
  * 802.1w - extension for rapid reconvergence of the spanning tree protocol (RTSP)
  * 802.1Q - added VLAN and priority support, where each link address maps to one VLAN
  * 802.1v - added VLANs where segments map to VLANs based link address together with network protocol and transport port
  * 802.1s - added support for multiple spanning trees, up to a maximum of 65, one per non-overlapping group of VLANs

  In addition, the following more recent extensions have been standardized to specify provider/carrier Ethernet services that can be effectively transparent to the previously specified customer Ethernet services. The TRILL problem as described in this document is limited to customer Ethernet services; however, there is no reason that a TRILL solution might not be easily applicable to both customer and provider Ethernet:
  * 802.1ad (Provider Bridges) - added support for a second level of VLAN tag, called a "service tag", and renamed the original 802.1Q tag "customer tag". Also known as **Q-in-Q** because of the stacking of 802.1Q VLAN tags
  * 802.1ah (Provider Backbone Bridges) - added support for stacking of MAC address by providing a tag to contain the original source and destination MAC address. Also known as **MAC in MAC**.

  It is useful to note that no extension listed above in this section addresses the issue of independent, localized routing in a single LAN -- which is the focus of TRILL.

  The TRILL problem and a sketch of a possible solution were presented to both the IETF (via a BoF) and IEEE 802 (via an IEEE 802 Plenary Meeting Tutorial). The IEEE, in response, approved a project callled **Shortest Path Bridging (IEEE Project P802.1ap)**, taking a different approach than that presented in [Pe04]. The current Draft of 802.1aq appears to describe two different techniques:
  * One, which does not use encapsulation, is , according to the IEEE Draft, limited in applicability to small networks of no more than 100 shortest path bridges
  * The other, which uses 802.1ah, is, according to the IEEE Draft, limited in applicability to networks of no more than 1000 shortest path bridges.

### 2.6. Problems Not Addressed
  There are other chanllenges to deploying Ethernet subnets that are not addressed in this document other than, in some cases, to mention relevant IEEE 802.1 documents, although it is possible for a solution to address one or more of these in addition to the TRILL problem:
  * increased Ethernet link subnet scale
  * increased node relocation
  * security of the Ethernet link subnet management protocol
  * flooding attacks on a Ethernet link subnet
  * support for "Provider" services such as Procider Bridges (802.1ad), Provider Backbone Bridges (802.1ah), or Provider Backbone Traffic Engneering (802.1Qay)

  Solutions to TRILL need not support deployment of larger scale of Ethernet link subnet than current broadcast domains can support.

  Similarly, solution to TRILL need not address link-layer node migration, which can complicate the caches in learning bridges. Similar challenges exist in the ARP, where link-layer forwarding is not updated appropriately when nodes move to ports on other bridges. Again, the compartmentalization available in the networl routing, like that of network-layer Autonomous Systems (ASes), can help hide the effect of migration. That is a side effect, however, and not a primary focus of this work.

  Current link-layer control-plane protocols, including Ethernet link subnet management (spanning tree) and link/network integration (ARP), are vulnerable to a variety of attacks. **Solutions to TRILL need not address these insecurities**. Similar attacks exist in the data plane.

  TRILL solutions need not address any of these issues, although it is cirtical that they do not introduce new vulnerabilities in the process.

## 3. Desired Properties of Solutions to TRILL
  This section describes some of the desirable or required properties of any system that would solve the TRILL problems, independent of the details of such a solution. Most of these are based on retaining useful properties of bridges, or maintaining those properties while solving the problems listed in section 2.

### 3.1. No Change to Link Capabilities
  There must be no change to the service that Ethernet subnets already provides as a result of deploying a TRILL solution. Ethernet supports unicast, broadcast, and multicast ntively. Although network protocols, notably IP, can tolerate link layers that do not provide all three, it would be useful to retain the support already in place.

  So called "zero configuration protocols" (also known as "zeroconf", e.g., as used to configure link-local addresses [RFC3927]), as well as existing bridge autoconfiguration, are also dependent on broadcast.

  Current Ethernet ensures in-order delivery for frames of the same priority and no duplicated frames, under normal operation (excepting transients during reconfiguration). These criteria apply in varying degrees to the different types of Ethernet, e.g., basic Ethernet up through basic VLAN (802.1Q) ensures that all frames with the same priority between two link addresses have both properties, but protocol/port VLAN (802.1v) ensures this only for packets with the requirement.

  Bridge autolearning already is susceptible to moving nodes between ports, because previously learned associatetions between the port and link address change. A TRILL solution could be similarly susceptible to such changes

### 3.2. Zero Configuration and Zero Assumption
  Both bridges and hubs are zero configuration devices; hubs having no configuration at all, and bridges being automatically self-configured. Bridge are further **zero-assumption** devices, unlike hubs.

  Bridges can be interconnected in arbitrary topologies, without regard for cycles or even self-attachment. Spanning tree protocols (STPs) remove the impact of cycles automatically, and port autolearning reduces unnecessary broadcast of unicast traffic.

  **A TRILL solution should strive to have a similar zero-configuration, zero-assumption operation**. This includes having TRILL solution components automatically discover other TRILL solution components and organize themselves, as well as to configure that organization for proper operation (plug-and-play). It also includes zero-configuration backward compatibility with existing bridges bridges and hubs, which may include interacting with some of the bridge protocols, such as spanning tree.

  VLANs add a caveat to zero configuration; a TRILL solution should support automatic use of a default VLAN (like non-VLAN bridges), but would undoubtedly require explicit configuration for VLANs where bridges require such configuration.

  Autoconfiguration extends to optional services, such as multicast support via Internet Group Management Protocol (IGMP) snooping, broadcast support via service copy, and support of multiple VLANs.

### 3.3. Forwarding Loop Mitigation
  Using spanning trees avoid forwarding loops by construction, although transient loops can occur, e.g., via the temporarily undetected appearance of new link connectivity or the loss a sufficient number of spanning-tree control frames. Solutions to TRILL are intended to use adapted network-layer routing protocols that may introduce transient loops during routing convergence. A TRILL solution thus needs to provide support for mitigating the effect of such routing loops.

  In the Internet, loop mitigation is provided by decrementing hop counts (TTL); in other networks, packets include a trace (sometimes referred to as 'serialized' or 'unioned') of visited nodes. In addition, there may be localized consistency checks, such as whether traffic is recieved on an unexpected interface, which indicates that routing is in flux and that such traffic should probably be discarded for safety. These types of mechanisms limit the impact of loops or detect them explicitly. Mechanisms with similar effect should be included in TRILL solutions

### 3.4. Spanning Tree Management
  In order to address convergence under reconfiguration and robustness to link interruption, participation in the spanning tree must be carefully managed. The goal is to provide the desired stability of the TRILL solution and of the entire Ethernet link subnet, which may include bridges using STP. This may involve a TRILL solution participating in the STP, where the protocol used for TRILL might dampen interactions with STP, or it may involved severing the STP into separate STPs on 'stub' external Ethernet link subnet segments.

  A requirement is that a TRILL solution must not require modifications or exceptions to the existing spanning tree protocols (STP, RSTP, MSTP).

### 3.5. Multiple Attachments
  In STP, a single node with multiple attachments to a single spanning tree segment will always get and send traffic over only one of the those attachment points. TRILL must manage all traffic, including multicast and broadcast traffic, so as not to create traffic loops involving Ethernet segments with multiple TRILL attachment points.

  This includes multiple attachments to a single TRILL node and attachments to multiple TRILL nodes. Support for multiple attachments can improve support for forms of mobility that induce topology changes, such as "make before break", although this is not a major goal of TRILL.

### 3.6. VLAN Issues
  A TRILL solution should support multiple customer VALNs (802.1Q, which includes 802.1v and 802.1s). This may involve ignorance, just as many bridge devices do not participate in the VALN supports. A TRILL solution may alternately furnish direct VLAN support, e.g., by proiding configurable support for VLAN-ignorant end stations equivalent to that provided by 802.1Q non-provider bridges.

  A TRILL solution might or might not be easily adaptable to handling provider VALNs.

### 3.7. Operational Equivalence
  As with any extension to an existing architecture, it would be useful -- though not strictly necessary -- to be able to describe or consider a TRILL solution as equivalent to an exsiting link layer component. Such equivalence provides a validation model for the architecture and a way for users to predict the effect of the use of a TRILL solution on a deployed Ethernet. In this case, 'user' refers to users of the Ethernet protocol, whether at the host (data segments), bridge (ST control segments), or VLAN (VLAN protocol).

  This provides a sanity check, i.e., "we got it right if we can exchange a TRILL solution component or components with an X" (where "X" might be a single bridge, a hub, or some other link layer abstraction). It does not matter whether "X" can be implemented on the same scale as the corresponding TRILL solution. It also does not matter if it can -- there may be utility to deploying the TRILL solution components incrementally, in ways that a single "X" could not be installed.

  For example, if a TRILL solution's components were equivalent to a single IEEE 802.1D bridge, it would mean that they would -- as a whole - participate in the STP. This need not require that TRILL solution components would propagate STP, any more than a bridge need do so in its on-board control. It would mean that the solution would interact with BPDUs at the edge, where the solution would -- again, as a whole - participate as if a single node in the spanning tree.

  Note that this equivalence is not required; a soluyion may act as if an IEEE 802.1 hub, or may not have a corresponding equivalent link layer component at all.

### 3.8. Optimizations
  There are a number of optimizations that may be applied to TRILL solutions. These must be applied in a way that does not affect functionality as a tradeoff for increase performance. Such optimizations may address broadcast and multicast frame distribution, VLAN support, and snooping of ARP and IPv6 neighbor discovery.

  In addition, there maybe optimizations which make the implementation of a TRILL solution easier than roughly equivalent existing bridge devices. For example, in many bridged LANs, there are topologies such that central bridges which have both a greater volume of traffic flowing through them as well as traffic to and from a larger variety of end station than do non-core bridges. Thus means that such core bridges need to learn a large number of end station addresses and need to do lookups based on such addresses very rapidly.

  This might require large high speed content addressable memory making implementation of such core bridges difficult. Although a TRILL solution need not provide such optimizations, it may reduce the need for such large, high speed content addressable memories or provide other similar optimizations.

### 3.9. Internet Architecture Issues
  TRILL solutions are intended to have no impact on the Internet network layer architecture. In particular, the Internet and higher layer headers should remain interact when traversing a deployed TRILL solution, just as they do when traversing any other link subnet technologies. This means that the IP TTL field cannot be co-opted for forwarding loop mitigation, as it would interfere with the Internet layer assuming that the link subnet was reachable with no changes in TTL. (Internet TTLs are changed only at routers, as per RFC 1812, and even if IP TTL were considered, TRILL is expected to support non-IP payload, and so requires a separate solution anyway).

  TRILL solutions should also have no impact on Internet routing or signaling, which also means that broadcast and multicast, both of which can pervade an entire Ethernet link subnet, must be able to transparently pervade a deployed TRILL solution. Changing how either of these capabilities behaves would have significant effects on a variety of protocols, including RIP (broadcast), RIP(multicast), ARP(broadcast), IPv6 neighbor discovery (multicast), etc.

  Note that snooping of network-layer packets may be useful, especially for certain optimizations. These include snooping multicast control-plane packets (IGMP) to tune link multicast to match the network multicast topology, as it already done in exsiting smart switches. This also includes snooping IPv6 neighbor discovery message to assist with governing TRILL solution edge configuration, as is the case in some smart learning bridges.

  Other layers may similarly be snooped, notablt ARP packets, for similar reasons as for IPv4.

## 4. Applicability

## 5. Security Considerations

## 6. Acknowledgments

## 7. Informative References
