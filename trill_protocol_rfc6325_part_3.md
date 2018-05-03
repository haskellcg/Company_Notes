## 5. RBridge Parameters
  This section lists parameters for RBridge. It is expected that the TRILL MIB will include many of the items listed in this section plus additional RBridge status and data including traffic and error counts.

  The default value and range are given for parameter added by TRILL. Where a parameter is defined as a 16-bit unsigned integer and an explicit maximum is not given, that maximum is 2\*\*16-1, for parameters imported from [802.1Q-2005], [802.1D], or IS-IS [ISO10589] \[RFC1195\], see those standards for default and range if nor given here.

### 5.1. Per RBridge
  The following parameters occur per RBridge:
  * Number of nicknames, which defaults to 1 and may be configured in range of 1 to 256
  * The desired number of distribution trees to be calculated by every Rbridge in the campus and a desired number of distribution trees for the advertising RBridge, both of which are unsgined 16-bit integers that default to 1
  * The maximum number of distribution trees the RBridge can compute. This is a 16-bit unsgined inteeger that is implementation and environment dependent and not sucject to management configuration.
  * Two list of nicknames, one designating the distribution trees to be computed and one desginating distribution trees to be used as discussed in section 4.5. By defaut, these lists are empty
  * The parameters Ageing Timer and Forward Delay with the default and range specified in [802.1Q-2005]
  * Two unsigned octets that are, respectively, the confidence in {MAC, VLAN, local port} triples learned from locally received native frams and the confidence in {MAC, VLAN, remote RBridge} triples learned from decapsulating frames. These each default to 0x20 and may each be configued to value from 0x00 to 0xFE.
  * The desired minimum acceptable inter-RBridge link MTU for the campus, that is, originatingLSPBufferSize. This is a 16-bit unsgined integr number of octets that defaults to 1470 bytes, which is the minimum valid value. Any lower value being advertised by an RBridge is ignored.
  * The number of failed MTU-probe before the RBridge conclusdes that a particular MRU is not supported, which defaults to 3 and may be configured between 1 and 255
  
  Static end-station address information and confidence in such end station information statically configured can also be configured with a default configence of 0xFF anf range of 0x00 to 0xFF. By Default, there is no such static address information. The quantity of such information that can be configured is implementation dependent.

### 5.2. Per Nickname Per RBridge
  The following is configration information per nickname at each Rbridge:
  * Priority to hold the nickname, which defaults to 0x40 if no specific value has been configured or 0xC0 if it is configured.
  * Nickname priority to be selected as distribution tree root, a 16-bit unsigned integer that defaults to 0x8000
  * Nickname value, an unsigned 16-bit quantity that defaults to the configured value if configured, elso to the last value held if the RBridge coming up after a reboot and that value is remembered, else to a random value; however, in all cases the reserved values 0x0000 and 0xFFC0 through 0xFFFF are excluded

### 5.3. Per Port Per RBridge
  An RBridge has the following per-port configuration parameters:
  * The same parameters as an [802.1Q-2005] port in terms of C-VLAN IDs. Inaddition, there is an announcing VLANs set that deafults to the enabled VLANs on the port, and ranges from the null set to the set of all legal VLAN IDs.
  * The same parameters as an [802.1Q-2005] port in terms of frame priority code point mapping
  * The inhibition time for the port when it observed a change in root bridge of an attached bridged LAN. This is in units of seconds, default to 30, and can be configured to any value from 0 to 30.
  * The desired Designated VLAN that the Rbridge will advertise in its TRILL Hellos if it is the DRB for the link via that port. This defaults to the lowest VLAN ID enabled on the port and may be configured to any valid VLAN ID that is enabled on the port.
  * Four per-port configuration bits: disable bit, disable end-station service, access port, and use P2P Hellos.
  * One bit per port such that, if the bit is set, it disables learning {MAC address, VLAN, port} triples from locally received native frames on the port. Default value is 0 == learning enabled.
  * The priority of the RBridge to be DRB and its Holding Time via the port with defaults and ranges as specific in IS-IS [RFC1195]
  * A bit that, when set, enables the receipt of TRILL-encapsulated frames from an Outer.MacSA with which the Rbridge does not have an IS-IS adjacency. Default value is 0 == disabled
  * Configuration for the optional send-BPDUs solution to the writing cloest topolgy problems as described in Appendix A.3.3. Default Bridge Address is the System ID of the Rbridge with lowest System ID. If RB1 and RB2 are part of a wiring closet topolgy, both need to be configured to know about this, and know the ID that should be used in the spanning tree protocol on the specified port.

### 5.4. Per VLAN Per RBridge
  An RBridge has the following per-VLAN configuration parameters:
  * Per-VLAN ESADI Protocol participation flag, 7-bit priority, and Holding Time. Default participation flag is 0 == not participating. Default and range of priority and Holding Time as specified in IS-IS [RFC1195]
  * One bit per VLAN that, if set, disables learning {MAC address, VLAN, remote RBridge} triples from frames decapsulated in the VLAN. Defaults to 0 == learning enabled.

## 6. Security Considerations
  TODO

### 6.1. VLAN Security Considerations
  TODO

### 6.2. BPDU/Hello Denial-of-Service Considerations
  TODO

## 7. Assignment Considerations
  TODO

### 7.1. IANA Considerations
  A new IANA registri has been created for TRILL parameter with two subregistries as below.

  This initial contents of the TRILL Nicknames subregistry are as follows:
  * 0x0000 Reserved to indicate no nickname specified
  * 0x0001 - 0xFFBF Dynamic allocated by the RBridges within each Rbridge campus
  * 0xFFC0 - 0xFFBF Available for allocation by RFC Required (single value) or IETF Review (single or multiple values)
  * 0xFFFF Permanetly reserved

  The initial contents of the TRILL multicast address subregitry are as follows:
  * 01-80-C2-00-00-40 Assigned as All-RBridge
  * 01-80-C2-00-00-41 Assigned as All-IS-IS-RBridge
  * 01-80-C2-00-00-42 Assigned as All-ESADI-RBridge
  * 01-80-C2-00-00-43 to 01-80-C2-00-00-4F Available for allocation by IETF Review

### 7.2. IEEE Registration Authority Considerations
  The Ethertype 0x22F3 is assigned by the IEEE Registration Authority to the TRILL Protocol

  The Ethertype 0x22F4 is assigned by the IEEE registration Authority for L2-IS-IS.

  The block of 16 multicast MAC addresses from 01-80-C2-00-00-40 to 01-80-C2-00-00-4F is assigned by the IEEE Registration Authority for IETF TRILL protocol use.

## 8. Normative References
  * [802.1ak]
  * [802.1D]
  * [802.1Q-2005]
  * [802.3]
  * [ISO10589]
  * [RFC1112]
  * [RFC1195]
  * [RFC2119]
  * [RFC2464]
  * [RFC2710]
  * [RFC3376]
  * [RFC3419]
  * [RFC4286]
  * [RFC5226]
  * [RFC5303]
  * [RFC5305]
  * [RFC6165]
  * [RFC6326]

## 9. Informative References
  * [802.1AB]
  * [802.1ad]
  * [802.1ah]
  * [802.1aj]
  * [802.1AE]
  * [802.1AX]
  * [802.1X]
  * [RBridges]
  * [RFC1661]
  * [RFC3411]
  * [RFC4086]
  * [RFC4541]
  * [RFC4789]
  * [RFC5304]
  * [RFC5310]
  * [RFC5342]
  * [RFC5556]
  * [RFC1999]
  * [VLAN-MAPPING]

## Appendix A. Incremental Deployment Considerations
  Some aspects of partial RBridge deployment are described below for link cost determination (A.1) and possible congestion due to appointed forwarder bottlenecks (A.2). A particular example of a problem related to the TRILL use of a single appointed forwarder per link per LNA (the "wrieing closet topology") is explored in detail in A.3.

### A.1. Link Cost Determination
  With an RBridged campus having no bridges or repeaters on the links between RBridges, the Rbridges can accurately determine the number of physical hops involved in a path and the line speed of each hop, assuming this is reported by thier port logic. With intervening devices, this is no longer possible. For example, as shown in figure 12, the two bridges B1 and B2 can completely hide a slow link so that both RBridge RB1 and RB2 incorrectly believe the link is faster.

            +-----+        +----+        +----+        +-----+
            |     |   Fast |    | Slow   |    |   Fast |     |
            | RB1 +--------+ B1 +--------+ B2 +--------+ RB2 |
            |     |   Link |    | Link   |    |   Link |     |
            +-----+        +----+        +----+        +-----+

  Even in the case of a single intervening bridge, two RBridge may known they are connected but each sees the links as a different speed from how it is seen by the other.

  However, this problem is not unique to RBridges. Bridges can encounter similar situations due to links hidden by repeaters, and routers can encounter similar situations due to links hidden by bridges, repeaters, or RBridges.

### A.2. Appointed Forwarders and Bridged VLANs
  With partial RBridge deployment, the RBridge may partition a bridged LAN into a relatively small number of relatively small number of relatively large remant briged LANs, or possible not partition it at all so a single bridged LAN remains. Such configuration can result in the following problem.

  The requirement that native frames enter and leave a link via the link's appointed forwarder for the LAN of the frame can cause congestion or suboptimal routing. (Similar problems can occur within a bridged LAN due to the spanning tree algorithm.) The extent to which such a problem will occur is highly dependent on the network topology. For example, if a bridged LAN had a star-like structure with core bridges that connected only to other bridges and peripheral bridges that connected to end stations and are connected to core bridges, the replacement of all of the core bridges by RBridges without replacing the peripheral bridges would generally improve performance without inducing appointed forwarder congestion.

  Solutions to this problem are dicussed below and a particular exmple explored in A.3.

  Inserting RBridge so that all the bridged portions of the LAN stay connected to each other and have multiple RBridge connection is generally the least efficient arrangement.

  There are four techniques that may help if the problem above occurs and that can, to some extent, be used in combination:
  1. Replace more IEEE 802.1 customer bridges with RBridges so as to minimize the size of the remnant bridged LANs between RBridges. This requires no configuration of the RBridges unless the bridges they replace required configurations
  1. Re-arrange network topology to minimize the problem. If the bridges and RBridges involved are configured, this may require changes in their configration.
  1. Configure the RBridges and bridges so that end-station on a remnant bridged LAN are separated into different LANs that have different appointed forwarders. If the end stations were already assigned to different VLANs, this is straightforward. If the end station were on the same VLAN and have to be split into different VLANs, this techniques may lead to connectivity problems between end stations.
  1. Configure the RBridges such that their ports that are connected to the bridged LAN send spanning tree configuration BPDUs in such a way as to force the partition of the bridged LAN. (Note: A spanning tree is never formed through an RBridge but always terminates at Rbridges ports.) To use this techniques, the RBridges must support this optional feature, and would need to be configured to use it, but the bridges involved would rarely hav to be configured. This technique makes the bridged LAN unavailable for TRILL through traffic because the bridged LAN partitions.

  Conversely to item 3 above, there may be bridged LANs that use VLANs or use more VLANs than would be necessary, to support the multiple Spanning Tree Protocol or otherwise reduce the congestion that can be caused by single spanning tree. Replacing the IEEE 802.1 bridges in such LANs with Rbridges may enable a reduction in or elimination of VLANs and configuration complexity.

### A.3. Wiring Closet Topology
  If 802.1 bridges are present and RBridges are not properly configured, the bridge spanning tree or the DRB may make inappropriate decisions. Below is s specific example of the more general problem that can occur when a bridged LAN is connected to multiple RBridges.

  In cases where there are two or more groups of end nodes, each attached to a bridge (say, B1 and B2), each bridge is attached to an Rbridge (say, Rb1 and RB2, respectively), with an additional link connecting B1 and B2, it may be desirable to have the B1-B2 linm only as a backup in case one of RB1 or RB2 or one of the links B1-RB1 or B2-RB2 fails.

                +-------------------------------+
                |              |          |     |
                | Data      +-----+    +-----+  |
                | Center   -| RB1 |----| RB2 |- |
                |           +-----+    +-----+  |
                |              |          |     |
                +-------------------------------+
                               |          |
                               |          |
                +-------------------------------+
                |              |          |     |
                |           +----+     +----+   |
                | Wiring    | B1 |-----| B2 |   |
                | Closet    +----+     +----+   |
                | Bridged                       |
                | LAN                           |
                +-------------------------------+

  For example, B1 and B2 may be in a wiring closet and it may be easy provide a short, high-bandwidth, low-cost link between them while RB1 and RB2 are at a distant data center such that the RB1-B1 and RB2-B2 links are slower and more expensive.

  Default behavior might be that one of RB1 or RB2 would become DRB for the bridged LAN including B1 and B2 and appint itself forwarder for the VLANs on that bridged LAN. As a result, Rb1 would forward all traffic to/from the link, so end nodes attached to B2 would be connected to the campus via the path B2-B1-RB1, rather than the desired B2-RB2. This waste the bandwidth of the B2-RB2 path and cuts available bandwidth between the end-stations and the data center in half. The desired behavior would be to make sure of both the RB1-B1 and RB2-B2 links.

#### A.3.1. The RBridge Solution
  Of course, if B1 and B2 are replaced with RBridges, the right thing will happen without configuration (other than VlAN support), but this may not be immediately practical if bridges are being incrementally replaced by RBridges.

#### A.3.2. The VLAN Solution
  If the end station attached to B1 and B2 are already divided among a number of VLANs, Rb1 and RB2 could be configured so that whichever becomes DRB for this link will appointed itself forwarder for some of these VLANs and appoint the other RBridge for the remainning VLANs. Should either of the RBridges fail or become disconnected, the other will have only itself to appoint as forwarder for all the VLANs.

  If the end stations are all on a single VLAN, then it would be necessary to assign them between at least two VLANs to use this solution. This may lead to connectivity problems that might requied further measures to rectify.

#### A.3.3. The Spanning Tree Solution
  Another solution is to configure the relevant ports on RB1 and RB2 to be part of a "wiring closet group", with a configured per-RBridge port "Bridge Address" Bx (which may be RB1 or RB2's System ID). Both RB1 and RB2 emit spanning tree BPDUs on their configured ports as highest priority root Bx. This cause the spanning tree to logically partition the bridged LAN as desired by blocking B1-B2 link at one end or the other (unless one of the bridges is configured to also have highest priority and has a lower ID, which we consider to be a misconfiguration). With the B1-B2 link blocked, RB1 and RB2 cannot see each other's TRILL-Hellos via tht link and each acts as Designated RBridge and appointed forwarder for its respective partition. Of course, with this partition, no TRILL through traffic can flow through the RB1-B1-B2-RB2 path.

  In the spanning tree configuration BPDU, the Root is "Bx" with highest priority, cost to Root is 0, Designated Bridge ID is "RB1" when RB1 transmits and "RB2" when RB2 transmits, and port ID is a value chosen independently by each of RB1 and RB2 to distinguish each of its own ports. The topology change flag is zero, and the topology change BPDU has been received on the port since the last configuration BPDU was transmitted on the port. (If Rb1 and RB2 were actually bridges on the same shared medium with no bridges between them, the result would be that the one with the larger ID sees "better" BPDUs (because of the tiebreaker on the third field: the ID of the transmitting bridge), and would turn off its port.)

  Should either RB1 or the RB1-B1 link or RB2 or the RB2-B2 link fail, the spanning tree algorithm will stop seeing one of the RBx roots and will unblock the B1-B2 link maintaining connectivity of all the end stations with the data center.

  If the link RB1-B1-B2-RB2 is on the cut set of the campus and RB2 and RB1 have been configured to believe they are part of a wiring closet group, the campus becomes partitioned as the link is blocked.

#### A.3.4. Comparison of Solutions
  Replacing all 802.1 customr bridges with RBridge is usually the best solution with the least amount of configuration required, possibly none.

  The VLAN solution works well with a relatively small amount of configuration if the end stations are already divided among a number of VLANs. If they are not, it becomes more complex and problematic.

  The spanning tree solution does quit well in this particular case. But it depends on both RB1 and RB2 having implemented the optional feature of being able to configure a port to emit spanning tree BPDUs as described in A.3.3 above. It also makes the bridged LAN whose partition is being forced unavailable for through traffic. Finally, while in this specific example, it neatly breaks the link between the two bridges B1 and B2, if there were a more complex bridged LAN, instead of exactly two bridges, there is no guarantee that it would paritition into roughly equal pieces. In such case, you might end up with a highly unbalanced load on the RB1-B1 link and the RB2-B2 link although this is still better than using only one of these links exclusively.

## Appendix B. Trunk and Access Port Configuration
  Many modern bridged LANs are organized into a core and access model, the core bridges have only point-to-point links to other bridges while the access bridges connect to end stations, core bridges, and possibly other access bridges. It seems likely that some Rbridge campuses will be organized in a similar fashion.

  An RBridge port can be configured as a trunk port, that is, a link to another Rbridge or RBridges, by configuring it to disable end-station support. These is no reason for such a port to have more than one VLAN enabled and in its Announcing Set on the port. Of course, the RBridge (or RBridges) to which it is connected must have the same VLAN enabled. There is no reason for this VLAN to be other than the default VLAN 1 unless the link is actually over carrier Ethernet or other facilities that only provides some other specific VLAN or the like, Such Configuration minimizes wasted TRILL-Hellos and eliminates useless decapsulation and transmission of multi-destination traffic in native form onto the link.

  An RBridge access port would be expected to lead to a link with end stations and possibly one or more bridges. Such a link might also have more than one Rbridge connected to it to provide more reliable service to the end stations. It would be a goal to minimize or elimite transit traffic on such a link as it is intended for end-station native traffic. This can be accomplished by turning on the access port configuration bit for the RBridge port or ports connected to the link as further detailed in section 4.9.1.

  When designing Rbridge configuration user interfaces, consideration should be given to making it convenient to coffigure ports as trunk and access ports.

## Appendix C. Multipathing
  RBridges support multipathing of both known unicast and multi-destination traffic. Implementation of multipathing is optional.

                                +---+
                                |RBy|---------------+
                                +---+               |
                               /  |  \              |
                             /    |    \            |
                           /      |      \          |
                        +---+   +---+   +---+       |
                        |RB1|---|RB2|---|RB3|       |
                        +---+   +---+   +---+\      |
                          |       |       |    \    |
                        +---+   +---+   +---+    \+---+
                        |RB4|---|RB5|---|RB6|-----|RBz|
                        +---+   +---+   +---+    /+---+
                          |       |       |    /
                        +---+   +---+   +---+/
                        |RB7|---|RB8|---|RB9|
                        +---+   +---+   +---+

  Multi-destination traffic can be multipthed by using different distribution tree roots for different frames. For example, assume that in Figure 14 end stations attached to RBy are the source of various multicast streams each of which has multiple listenders attached to various of RB1 through RB9. Assuming equal bandwidth links, a distribution tree rooted at RBy will predominantly use the vertical links among RB1 through RB9 while one rooted at RBz will predominantly use the horizontal. if RBy chooses its nickname as the distribution tree root for half of this traffic and an RBz nickname as the root for the other half, it may be able to substantially increase the aggregate bandwidth by making use of both the vertical and horizontal links among RB1 through RB2.

  Since the distribution trees an RBridge must calculate are the same for all RBridges and transit RBridges must respect the tree root specific by the ingress RBridge, a campus will operate correctly with a mix of RBridges some of which use different roots for different multi-destination frames they ingress and some of which use a single root for all such frames.

                                      +---+       double line = 10 Gbps
                        -----      ===|RB3|---    single line = 1 Gbps
                       /     \   //   +---+   \
                   +---+     +---+            +---+
                ===|RB1|-----|RB2|            |RB5|===
                   +---+     +---+            +---+
                       \     /   \    +---+  //
                        -----     ----|RB4|===
                                      +---+

  Known unicast Equal Cost Multipthing (ECMP) can occur at an RBridge if, instead of using a tiebreaker criterion when building SPF paths, information is retained about ports thrrough which euqal cost paths are available. Different unicast frames can then be sent through those different ports and will be forwarded by equal cost paths. For example, in Figure 15, which shows only RBridges and omits any Rbridges present, there are three equal cost path between RB1 and RB2 and two equal cost paths beetween Rb2 and RB5. Thus, for traffic transiting this part of the campus from left to right, RB1 may be able to perform three way ECMP and RB2 may be able to perform two-way ECMP.

  A transit Rbridge receiving a known unicast frame forwards it towards the egress RBridge and is not concerned with whether it believes itself to be on any parrticular path from the ingree Rbridge or a previous transit RBridge. Thus, a campus will operate correctly with a mix of RBridge some of which implement ECMP and some of which do not.

  There are actually three possibilities for the parallel paths between Rb1 and RB2 as follows:
  1. if two or three paths have pseudonodes, then all three will be distinctly visible in the campus-wide link state and ECMP as described above is applicable.
  1. If the paths use P2P Hellos or otherwise do not have pseudonodes, these three paths would appear as a single adjacency in the link state. In this case, multipathing across them would be aan entirely local matter for RB1 and RB2. It can be freely done for known unicast frames but not for multi-destination frames as described in section 4.5.2.
  1. If and only if the three paths between Rb1 and RB2 are single hop equal bandwidth links with no intervening bridges, then it would be permissible to combine them into one logical link through the [802.1AX] "link eggregation" feature. RBridge may implement link aggregation since operaates below TRILL

  When multipathing is used, frames that follow different paths will be subject to different paths will be subject to different delays and may be re-ordered. While some traffic may be oder/delay insensitive, typically most traffic consists of flows of frames where re-ordering within a flow is damaging. How to determine flows or what granularity flows should have is beyong the scope of this document.

## Appendix D. Determination of VLAN and Priority
  A high-level, informative summary of how VLAN ID and priority are determined for incoming native frames, omitting some details, is given in the bulleted items below. For more detailed information see [802.1Q-2005].
  * When an untagged native frame arrives, an unconfigured RBridge associates the default priority zero and the VLAN ID 1 with it. It actually sets the VLAN for the untagged frame to be the "port VLAN ID" associated with that port. The port VLAN ID defaults to VLAN ID 1 but may be configured on a per-port basis to discard such determination of the VLAN ID. An Rbridge may be also configured on a per-port basis to discard such frames or to associate a different priority code point with them. Determination of the VLAN ID associated with an incoming untagged non-control frame may also be made dependent on the Ethertype or NSAP of the arriving frame, the source MAC address, ot other local rule.
  * When a priority tagged native frame arrives, an unconfigured RBridge associated with it both the port VLAN ID, which defaults to 1, and the priority code point provided in the priority tag in the frame. An RBridge may be configured on a per-port basis to discard such frames or to associate them wwith a different VLAN ID as described in the point immediately above. It may also be configured to map the priority code point provided in the frame by specifying, for each of the eight possible values that might be in the frame, what actual priority code point will be associated with the frame, what actual priority code point will be associate with frame by the RBridge.
  * When a C-tahhed (formerly called Q-tagged) native frame arrives, an unconfigured Rbridge associates with it the VLAN ID and priority in the C-tag. An RBridge may be configured on a per-port per-LAN basis to discard such frames. It may also be configured on a per-port basis to map the priority value as specified above for priority tagged frames.

  In 802.1, the process of associating a priority code point with a frame, including mapping a priority provided in the frame to another priority, is refered to as priority "regeberation".

## Appendix E. Support of IEEE 802.1Q-2005 Amendments
  This informational appendix briefly comments on Rbridge support for completed and in-process amendments to IEEE [802.1Q-2005]. There is no assurance that existing Rbridge protocol specifications or existing bridges will support not yet specified future [802.1Q-2005] amendments just as there is no assurance that existing bridge protocol specifications or existing Rbridge will support not yet specified future TRILL amendments.

  The information below is frozen as of 25 October 2009. For the latest status, see the IEEE 802.1 working group.

### E.1. Complete Amendments
  * 802.1ad-2005 Provider Bridges - Sometime called "Q-in-Q", because VLAN tags used to be called "Q-tags", 802.1ad specifies Provider Rbridges that tunnel customer bridge traffic within service VLAN tags (S-tags). If the customer LAN is an Rbridge campus, that traffic will be bridged by Provider Bridges. Customer bridge features involving Provider bridge awareness, such as the ability to configure a customer bridge port to add an S-tag to a frame before sending it to a Provider bridge, are below the EISS layer and can be supported in RBridge ports without modification to the TRILL protocol.
  * 802.1ag-2007 Connectivity Fault Management (CFM) - TODO
  * 802.1ak-2007 Multiple Registration Protocol - TODO
  * 802.1ah-2008 Provider Backbone Bridges - TODO
  * 802.1Qaw-2009 Management of Data-Driven and Data-Dependent Connectivity Fault - TODO
  * 802.1Qay-2009 Provider Backbone Bridge Traffic Engineering - TODO

### E.2. In-Process Amendments

## Appendix F. Acknowledgements
