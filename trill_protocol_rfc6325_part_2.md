### 4.3. Inter-RBridge Link MTU Size
  There are two reaons why it is important to known what size of frame each inter-RBridge link in the campus can support:
  * RBridge RB1 must known the size of link state information messages it can generate that will be guaranteed to be forwardable across all inter-RBridge links in the campus.
  * If traffic engineering tools knowwn which links support larger than minimally acceptable data packet size, paths can be computed that can support large data packets.

#### 4.3.1. Determining Campus-Wide TRILL IS-IS MTU Size
  In a stable campus, there must ultimately be agreement among all RBridge on the value of "Sz", the minimum acceptable inter-RBridge link size for the campus, for the proper operation of TRILL IS-IS. All RBridges must format their link state information messages to be in chunks of size no larger than what they believe Sz to be. Also, every RBridge RB1 should test each of its RBridge adjacencies, say, to RB2, tn ensure that the RB1-RB2 link can forward packets of at least size Sz.

  Sz has no direct on end stations and is not directly related to any end-station-to-end-station "path MTU". Methods of using Sz or any link MTU information gathered by TRILL IS-IS in the traffic engineering of routes or the determination of any oath MTU is beyond the scope of this document. Native frames that, after TRILL encapsulation, exceed the MTU of a link on which  they are sent will generally be discarded.

  Sz is determined by having each RBridge (optionally) advertise, in its LSP, its assumption of the value of the campus-wide sz. This LSP element is known in IS-IS as the originatingLSPBufferSIze, TLV #14. The default and minimum value for Sz, and the implicitly advertised value of Sz if the TLV is adsent, is 1470 octets. This length (which is also the maximum size of a TRILL-Hello) was chosen to make it extremely unlikely that a TRILL control frame, even with reasonable additional headers, tags, and/or encapsulation, would encounter MTU problems on an inter-RBridge link.

  The campus-wide value of Sz is the smallest value of Sz advertised by any RBridge.

#### 4.3.2. Testing Link MTU Size
  There are two new TRILL IS-IS message types for use between pairs of RBrudge neighbors to test the bidirectional packet size capacity of their connection. Thes messages are:
  * \-\- MTU-probe
  * \-\- MTU-ack

  Both the MTU-probe and the MTU-ack are padded to the size being tested.

  Sending of MTU-probes is optional; however, an RBridge RB2 that receives an MTU-probe from RB1 must respond with an MTU-ack padded to the same size as the MTU-probe. The MTU-probe may be multicast to All-RBridges, or unicast to a specific RBridge. The MTU-ack is normally unicast to the source of the MTU-probe wo which it responds but may be multicast to the All-RBridge.

  If RB1 fails to receive an MTU-ack to a probe of size X from RB2 after k tries (where k is a configurable parameter whose default is 3), then RB1 assumes the RB1-RB2 link cannot support isze X. If X is not greater than Sz, then RB1 sets the "failed minimum MTU test" flag for RB2 in RB1's hello. If size X succeeds, and X > Sz, then RB1 advertises the largest tested X for each adjacency in the TRILL Hellos RB1 sends on that link, and RB1 may advertise X as attributes of the link to RB2 in RB1's LSP.

### 4.4. TRILL-Hello Protocol
  The TRILL-Hello Protocol is a little different from the Layer 3 IS-IS LAN Hello Protocol and uses a new type of IS-IS messages knwon as a TRILL-Hello.

#### 4.4.1. TRILL-Hello Rationale
  The reason for definninf this new type of link in TRILL is that in Layer 3 IS-IS, the LAN Hello protocol may elect multiple Desinated Routers (DRs) since, when choosing a DR, routers ignore other routers with whom they do not have 2-way connectivity. Also, Layer 3 IS-IS LAN hellos are padded, to avoid forming adjacencies between neighbors that can't speak the maximum-sized packet to each other. This means, in Layer 3 IS-IS, the neighbors that have connectivity to each other, but with an MTU on that connection less than what they percieve as maximum sized packets, will not see each other's hellos. The result is that routers might from cliques, resulting in the link turnning into multiple pseudonodes.

  This behavior is fine for Layer 3, but not for Layer 2, where loops may form if there are multiple DRBs. Thereform, the TRILL-Hello protocol is a little different from Layer 3 IS-IS's LAN Hello protocol.

  One other issue with TRILL-Hellos is to ensure that subnets of the information can appear in any single message, and be processable, in the spirit of IS-IS LSPs and CSNPs. TRILL-Hello frames, even though they are not padded, can become very large. An example where this might be the case is when some sort of backbone technology interconnects hundreds of TRILL sites over what would appear to TRILL to be a giant Ethernet, where the RBridge connected to that cloud will perceive that backbone to be a single link with hundreds of neighbors.

  In TRILL (Unlike in Layer 3 IS-IS), the DRB is selected based solely on priority and MAC address. In other words, if RB2 receives a TRILL-Hello from RB1 with higher (priority, MAC), RB2 defers to RB1 as DRB, regardless of whether RB1 lists RB2 in RB1's TRILL-Hello.

  Although the neighbor list in a TRILL-Hello does not influence the DRB election, it does determine what is announced in LSPs. RB1 only reports links to RBridges with which it has two way connectivity. If RB1 is the DRB on a link, and for whatever reason (MTU mismatch, or one-way connectivity) RB1 and RB2 to not have two-way connectivity, then RB2 does not report a link to RB1 (or the psedunode), and RB1 (or RB1 on behalf of the pseudonode) does not report a link to RB2.

#### 4.4.2. TRILL-Hello Contents and Timing
  The TRILL-Hello has a new IS-IS message type. It starts with the same fixed header as an IS-IS LAN Hello, which includes the 7-bit priority for the issuing RBridge to be DRB on that link. TRILL-Hellos are sent with the same timming as IS-IS LAN Hellos.

  TRILL-Hello messages, including their Outer.MacDA and Outer.MacSA, but excluding any other Outer.VLAN or other tags, must not exceed 1470 octets in length and should be padded. The folloing information appear in every TRILL-Hello. References to "TLV" may actually be a "sub-TLV" as specified in separate documents [RFC6165] \[RFC6326\].

  1. The VLAN ID of the Designated VLAN for the link.
  1. A copy of the Outer.VLAN ID with which the Hello was tagged on sending.
  1. A 16-bit port ID assigned by the sending RBridge to the port the TRILL-Hello is sent on such that no two ports of that RBridge have the same port ID.
  1. A nickname of the sending RBridge
  1. Two flags as follows:
      1. A flag that, if set, indicates that the sender has detected VLAN mapping on the link, within the past 2 of its Holding Times.
      1. A flag that, if set, indicates that the sender believes it is appointed forwarder for the VLANand port on which the TRILL Hello was sent.

  The Following information may appear:
  1. The set of VLANs for which end-station service is enabled on the port.
  1. Several flags as follows:
      1. A flag that, if set, indicates that the sender's port was configured as an access port
      1. A flag that, if set, indicates that the sender's port was configured as a trunk port
      1. A bypass pseudonode flag, as described below in this section
  1. If the sender is the DRB, the RBridges (excluding itself) that it appoints as forwarder for link and the VLANs for which it appoints them. As describe below, this TLV is designed so that not all the appointment information need be included in each Hello. Its absence means that appointed forwarders should continues as previously assigned.
  1. The TRILL neighbor list. This is a new TLV, not the same as the IS-IS Neighbor TLV, in order to accomodate fragmentation and reporting MTU on the link

  The Appointed Forwarders TLV specifies a range of VLANs and, within that range, sepcifies within RBridge, if any, other than the DRB, is appointed forwarder for the VLANs in that range [RFC6326]. Appointing an RBridge as forwarder on a port for a VLAN that is not enable on that port has no effect.

  It is anticipated that many links between RBridges will be point-to-point, in which case using a pseudonode merely adds to the complexity. If the DRB specifie the bypass pseudonode bit in its TRILL-Hellos, the RBridges on the link just report their adjacencies as point-to-point. This has no effect on how LSPs are flooded on a link. It only effects what LSPs are generated.

  For example, if RB1 and RB2 are the only RBridges on the link and RB1 is the DRB, then if RB1 creates a pseudonode that is used, there are 3 LSPs: for, say, RB1.25 (the pseudonode), RB1, and RB2, where RB1.25 reports connectivity to RB1 and RB2, and Rb1 and RB2 each just say they are connected to RB1.25. Whereas if DRB RB1 sets the bypass pseudonode bit in its Hellos, then there will be only 2 LSPs: RB1 and RB2 each reporting connectivity to each other.

  A DRB should set the bypass pseudonode bit for its link unless, for a particular link, it has seen at least two simultaneous adjacencies on the link at somee point since it last rebooted.

##### 4.4.2.1. TRILL Neighbor List
  The new TRILL Neighbor TLV includes following information for each neighbor is lists:
  1. The neighbor's MAC address.
  1. MTU size to this neighbor as a 2-octet unsigned integer in units of 4-octets chunks. The value zero indicates that the MTU is untested.
  1. A flag for "failed minimum MTU test".

  To allow partial reporting of neighbors, the neighbor IDs must be sorted by ID. If a set of neighbors {X1, X2, X3, ..., Xn} is reported in RB1's Hello, then X1 < X2 < X3, ..., Xn. If RBridge RB2's ID is between X1 and Xn, and does not in RB1's Hello, then RB2 knowns that RB1 has not heard RB2's Hello.

  Additonally there are two overall TRILL Neighbor List TLV flags: "the smallest ID I reported in this hello is the smallest ID of any neighbor" and "the largest ID I reported in this hello is the largest ID of any neighbor". If all the neighbors fit in RB1's TRILL-Hello, both flags will be set.

  If RB1 reports {X1, ..., Xn} in its Hello, with the "smallest" flag set, and RB2's ID is smaller than X1, then RB2 knowns that RB1 was not heard RB2's Hello. Similarly, if RB2's ID is larger than Xn and the "largest" flag is set, then RB2 knwons that RB1 has not heard RB2's Hello.

  To Ensurn that any RBridge RB2 can definitively determine whether RB1 can hear RB2, RB1's neighbor list must eventually cover every possible range of IDs, that is, within a period that depends on RB1's policy and not necessarily within any specific period such as the holding time. In other words, if X1 is the smallest ID reported in one of RB1's neighbor lists, and the "smallest" flag is not set, then X1 must also appears as the largest ID reported in a different TRILL-Hello Neighbor list. Or, fragments may overlap, as long as there is no gap, such that some range, say between Xi and Xi, never appears in any fragment.

#### 4.4.3. TRILL MTU-Probe and TRILL Hello VLAN Tagging
  The MTU-probe mechanism is designed to determine the MTU for transmissions between RBridges. MTU-probe and probe acknownledgements are only sent on the Designated VLAN.

  An RBridge RBn maintains for each port the same VLAN information as a customer IEEE [802.1Q-2005] bridge, including the set of VLANs enabled for output through that port. In addition, RBn maintains the following TRILL-specific VLAN parameters per port:
  * Desired Designated VLAN: TODO
  * Designate VLAN: TODO
  * Announcing VLANs set. TODO
  * Forwarding VLANs set: TODO

  On each of its ports that is not configured to use P2P Hellos, an RBridge sends TRILL-Hellos Outer.VLAN tagged with each VLAN in a set of VLANs. This set depends on the RBridge's DRB status and the above VLAN parameters. RBridges send TRILL Hellos Outer.VLAN tagged with the Designated VLAN, unless that VLAN is not enabled on that port. In addition, the DRB sends TRILL Hellos Outer.VLAN tagged with  each enabled VLAN in ite announcing VLAN set. All non-DRB RBridge send TRILL-Hellos Outer.VLAN tagged with all enabled VLANs that are in the intersection of their Forwarding VLANs set and their Announcing VLANs set. More symbolically, TRILL-Hello frames, when sent, are sent as follows:
  ```
  If sender is DRB:
    intersection (Enabled VLANs,
                  union(Designated VLAN, Announcing VLANs))
  If sender is not DRB:
    intersection (Enabled VLANs,
                  union(Designated VLAN,
                        intersection(Forwarding VLANs, Announcing VLANs)))
  ```

  Configuring the Announcing VLANs set to be null minimizes the number of TRILL-Hellos. In that case, TRILL Hellos are only tagged with the Designated VLAN. Greate care should be taken in configuring an RBridge to not send TRILL Hellos on any VLAN where that RBridge is appointed forwarder as, under some circumstances, failure to send such Hellos can lead to loops.

  The number of TRILL-Hellos is maximized, within this specification, by configuring the Annoucning VLANs set to be the set of all enabled VLAN IDs, which is the default. In that case, the DRB will send TRILL-Hello frames tagged with all its enabled VLAN tags; in addition, any non-DRB RBridge RBn will send TRILL-Hello frames tagged with the Designated VLAN, if enabled, and tagged with all VLANs for which RBn is an appointed forwarder. (it is possible to send even more TRILL-Hellos. In particular, non-DRB RBridges could send TRILL-Hellos on enabled VLANs for which they are not an appointed forwarder and which are not the Designated VLAN. This would cause no harm other than a further communications and processing burden.)

  When an RBridge port comes up, until it has heard a TRILL-Hello from a Higher priority RBridge, it considers itself to be DRB on that port and sends TRILL-Hellos on that basis. Similarly, even if it has at some time recognized some other RBridge one the link as DRB, if it receives no TRILL-Hellos on that port from an RBridge with highwe priority as DRB for a long enough time, as specified by IS-IS, it will revert to believing itself DRB.

#### 4.4.4. Multiple Ports on the Same Link
  It is possible for an RBridge RB1 to have multiple ports to the same link. It is important for RB1 to recognize which of its ports are on the same link, so, for instance, if RB1 is appointed  forwarder for VLAN A, RB1 knowns that only one of its ports acts as appointed forwarder for VLAN A on that link.

  RB1 detects this condition based on receiving TRILL-Hello messages with the same IS-IS pseudonode ID on multiple ports. RB1 might have one set of ports, say, {p1, p2, p3} on one link, and another set of ports {p4, p5} on a second link, and yet other ports, say, p6, p7, p8, that are each on distinct link. Let us call a set of ports on the same link a "port group".

  If RB1 detects that a set of ports, say, {p1, p2, p3}, is a port group on a link, then RB1 must ensure that **it does not cause loops** when it encapsulates and decapsulates traffic from/to that link. If RB1 is appointed forwarder for VLAN A on that Ethernet link, RB1 must encapsulate/decapsulate VLAN A on only one of the ports. However, if RB1 is appointed forwrader for more than one VLAN, RB1 may choose to load split among its ports, using one port for some set of VLANs, and another port for a disjoint set of VLANs.

  If RB1 detects VLAN mapping occuring, then RB1 must not load split as appointed forwarder, and instead must act as appointd VLAN forwarder on that link on only one of its ports in the port group.

  When forwarding TRILL-encapsulated multi-destination frames to/from a link on which RB1 has a port group, RB1 may choose to load split among its ports, provided that it does not duplicate frames, and procided that it keeps frames for the same flow on the same port. If RB1's neighbor on that link, RB2, accepts multi-destination frames on that tree on that link from RB1, RB2 must accept the frame from any of RB2's adjacencies to RB1 on that link.

  If an RBridge has more than one port connected to a link and those ports have the same MAC address, they can be distinguished by the port ID contained in TRILL-Hellos.

#### 4.4.5. VLAN Mapping within a Link
  IEEE [802.1-2005] does not provide for bridges changing the C-tag VLAN ID for a tagged frame they received, that is, mapping VLANs. Nevertheless, some bridges products provide this capability and , in any case, bridge LANs can be configured to display this behavior. For example, a bridge port can be configured to strip VLAN tags on output and send the resulting untagged frames onto a link learning to another bridge's port configured to tag these frames with a different VLAN. Although each port's configuration is legal under [802.1Q-2005], in the aggregate they perform manipulations not permitted on a single customer [802.1Q-2005] bridge. Since RBridge ports have the same VLAN capabilities as customer [802.1Q-2005] bridges, this can occur even in the absense of bridges. (VLAN mapping is referred to in IEEE 802.1 as "VLAN ID translation".)

  RBridge include the Outer.VLAN ID insidde every TRILL-Hello message. When a TRILL-Hello is received, RBridges compare this saved copy with the Outer.VLAN ID information associated with the received frame. If these differ and the VLAN ID inside the Hello is X and the Outer.VLAN is Y, it can be assumed that VLAN ID X is being mapped into VLAN ID Y.

  When non-DRB RB2 detects VLAN mapping, based on receiving a TRILL Hello where the VLAN tag in the body of the Hello differs from the one in the outer header, it sets a flag in all of its TRILL-Hellos for a period of two of its Holding Times since the last time RB2 detected VLAN mapping. When DRB RB1 is informed of VLAN mapping, either because of seeing the "VLAN mapping detected" flag in a neighbors TRILL-Hello on the link, RB1 re-assigns VLAN forwarders to ensure there is only a single forwarder on the link for all VLANs.

### 4.5. Distribution Trees
  RBridges use distribution trees to forward mult-destination frames. Distribution trees are bidirectional. Although a single tree is logically sufficient for the entire campus, the computation of additional distribution trees is warranted for the following reasons:
  * It enables multipathing of multi-destination frames
  * enables the choice of a tree root closer to or, in limit, identical with the ingress RBridge

  Such a closer tree root improves the efficiency of the delivery of multi-destination frames that are being delivered to a subnet of the links in the campus and reduces out-of-order delivery when a unicast address transitions between unkown and known. If applications are in use where occasional out-of-order unicast frames due to such transitions are a problem, the RBridge campus should be engineered to make sure they are of extremely low probability, such as by using the ESADI protocol, configuring addresses to eliminate unknown destination unicast frames, or keeping alive frames.

  An additonal level of flexibility is the ability of an RBridge to acquire multiple nicknames, and therefore have multiple trees rooted at the same RBridge. Since the tree number is used as a tirbreaker for equal cost path, the different trees, even if rooted at the same RBridge, wil likely utilize different equal cost paths.

  **How an ingress RBridge chooses the distribution tree or trees that it uses for multi-destination frames is beyond the scope of this document**. However, for the reasons state above, in the absence of other factors, a good choice is the tree whose root is least cost from ingress RBridge and that is the default for an ingress RBridge that uses a single tree to distribute multi-destination frames.

  RBridge will precompute all the trees that might be used, and keep state for Reverse Path Forwarding Check filters, also, since the tree number is used as a tiebreaker, it is important for all RBridge to known:
  * how many trees to compute
  * which trees to compute
  * what the tree number for each tree is
  * which trees each ingress RBridge might choose (For building Reverse Path Forwarding Check filters)

  Each RBridge advertises in its LSP a "tree root" priority for its nickname or for each of its nickname if it has been configured to have more than one. This is a 16-bit unsigned integer that defaults, for an unconfigured RBridge, to 0x8000. Tree roots are ordered with highest numerical priority being highest priority, then with system ID of the RBridge (numerically higher = higher priority) as tiebreaker, and if that is equal, by numerically higher nickname value, as an unsigned integer, having priority.

  Each RBridge advertises in its LSP the maximum number of distribution trees that it can compute and the number of trees that it wants all RBridges in the campus to compute. The number of trees, k, that are computed for the campus is the number wanted by the RBridge RB1, which has the nickname with the highest "tree root" priority, but no more than the number of trees supported by the RBridge in the campus that supports the fewest trees. If RB1 does not specify the specific distribution trees are the trees that will be computed by all RBridges, Note that some of these k highest priority trees might be rootted at the same RBridge, if that RBridge has multiple nicknames.

  If an RBridge specifies the number of trees it can compute, or the number of trees it wants computed for the campus, as 0, it is treated as specifying them as 1. Thus, k default to 1.

  In addition, the RBridge RB1 having the highest root priority nickname might explicityly advertise a set of s trees by providing a list of s nicknames. In that case, the first k of those s trees will be computed. Is s is less than k, or if any of the s nicknames associated with the trees RB1 is advertising does not exist within the LSP database, then the RBridge still compute k trees, but the remianning trees they select are the highest priority trees, such that k trees are computed,

  There are two exceptions to the above, which can cause fewer distribution trees to be computed, as follows:
  * A nickname whose tree root priority is zero is not selected as a tree root based on priority, although it may be selected by being listed by the RBridge holding the highest priority tree root nickname. The one exception to this is that if all nicknames have priority zero, then the highest priority among them as determined by the tiebreaker is used as a tree root so that there is always guardanteed to be at least one distribution tree.
  * As a transient condition, two or more identical nicknames can appear in the list of roots for trees to be computed. In such a case, it is useless to compute a tree for the nickname(s) that are about to be losy by the RBridge holding them. So a distribution tree is only computed for the instance where the priority to hold that nickname value is highest, recuding the priority total number of trees computed. (It would also be of little use to go further down the priority ordered list of possible tree root nicknames to miantain the number of trees as the additional tree roots found this way would only be valid for a very brief nickname transition period.)

  The k trees calsulated for a campus are ordered and numbered form 1 to k. In addition to advertising the number k, RB1 might explicitly advertise a set of s trees by providing a list of s nicknames as described above.
  * If s == k, then the trees are numbered in the order that RB1 advertises them.
  * If s == 0, then the trees are numbered in the order of decreasing priority. For example, if RB1 advertises only that k=2, then the highest priority tree is number 1 and the 2nd highest priority tree is number 2.
  * If s < k, then those advertised by RB1 are numbered from in the order advertised. Then the remaider are chosen by priority order from among the remainning possible trees with the numbering continuing. For example, if RB1 advertises k=4, advertises {Tx, Ty} as the nicknames of the root of the trees, and the campus-wide priority ordering of trees in decreasing order is Ty > Ta > Tc > Tb > Tx, the numbering will be follows: Tx is 1 and Ty is 2 since that is the order they are advertised in RB1. Then Ta is 3 and Tc is 4 because they are highest priority trees that have not already been numbered.

#### 4.5.1. Distribution Tree Calculation
  RBridges do not use spanning tree to calculate distribution trees. Instead, distribution trees are calculated **based on the link state information**, selecting a particular RBridge nickname as the root. Each RBridge RBn independently calculates a tree rooted at RBi by performing the SPF (Shortest Path First) calculation with RBi as the root without requiring any additional exchange of information.

  It is important, when building a distribution tree, that all RBridge choose the same link for that tree. Therefore, when there are equal cosr paths for a particular tree, all RBridge need to use the same tiebreakers. It is also desirable to allow splitting of traffic on as many links as possible. For this reason, a simple tiebreaker such as "always choose the parent with lower ID" would not be desirable. Instead, **TRILL uses the tree numbre as a parameter in the tiebreaking algorithm**.

  When building the tree number j, remember all possible equal cost parents for node N. After calsulation the entire "tree" (actually, directed graph), for each node N, if N has "p" parents, then order the parents in ascending order according to the 7-octet IS-IS ID considered as an unsigned integer, and number them starting at zero. For tree j, choose N's parent as choice j mod p.
  
  Note that there might be multiple equal cost link between N and potential parent P that have no pseudonodes, because they are either point-to-point links or pseudonode-suppressed links. Such links will be treated as a single link for purpose of tree building, because they all have the sam parent P, whose IS-IS ID is "P.0".

  In other words, the set of potential parents for N, for the tree rooted at R, consists of those that give equally minimal cost paths from N to R and have distinct IS-IS IDs, based on what is reported in LSPs.

#### 4.5.2. Multi-Destination Frame Checks
  When a multi-destination TRILL-encapsulated frame is received by an RBridge, there are four checks performed, each of which may cause frame to be discarded:
  * Tree Adjacency Check: Each RBridge RBn keeps a set of adjacencies ({port, neighbor} pairs) for each distribution tree it is calculating. One of there adjacencies is toward the tree root RBi, and the others are toward the leaves. Once the adjacencies are chosen, it is irrelevant which ones are towards the root RBi and which are away from RBi. RBridge must drop a multi-destination frame that arrives at a port from an RBridge that is not an adjacency for the tree on which the frame is being distributed. Lets suppose that RBn has calculated that adjacencies a, c, and f are in the RBi tree. A multi-destination frame for the distribution tree is received only from one of the adjacencies a, c, or f (otherwise it is discarded) and forwarded to the other two adjacencies. Should RBn have multiple ports on a link, a multi-destination frame it sends on one of these port will be received by the others but will be discarded as an RBridge is not adjacent to itself.
  * RPF Check: Another technique used by RBridge for avoid temporary multicast loops during topology changes is **the Reverse Path Forwarding Check**. It involves checking that a multi-destination frame, based on the tree and the ingress RBridge, arrives from the expected link. RBridge must drop multi-destination frames that fail the RPF check.
  
  To limit the amount of state necessary to perform the RPF check, each RBridge RB2 must announce which trees RB2 choose when RB2 ingresses a multi-destination packet. TODO
  * Parallel Links Check: If the tree-building and tiebreaking for a particular multi-destination frame distribution tree selects a non-pseudonode link between RB1 and RB2, that "RB1-RB2 link" might actually consist of multiple links. There parallel links would be visible to RB1 and RB2, but not to the rest of the campus (because the links are not representd by pseudonodes). If this bundle of parallel links is included in a tree, it is important for RB1 and RB2 to decide which link to use, but is irrelevant to other RBridges, and therefore, the tiebreaker algorithm need not be visible to any RBridges other than RB1 and RB2. In this case, RB1-RB2 adjacencies are ordered as follows, with the one "most prefered" adjacencies being the one on which RB1 and RB2 transmit to and receive multi-destination frames from each other.
    * Most preferred are those established by P2P Hellos. Tiebreaker among those is based on preferring the one with numerically highest Extended Circuit ID as associated with the adjacency by the RBridge with the highest System ID
    * Next considered are those established through TRILL-Hello frames, with suppressed pseudonodes. Note that the pseudonode is suppressed in LSPs, but still appears in the TRILL-Hello, and therefore is available for this tiebreaking. Among These links, the one with the numerically largest pseudonode ID is preferred.
  * Port Group Check: If an RBridge has multiple ports attached to the same link, a multi-destination frame it is receiving will arrive on all of them. All but one received copy of such a frame must be discarded to avoid duplication. All such frames that are part of the same flow must be accepted on the same port to avoid reordering.

  When a topology change occurs (including apparent changes during start up), an RBridge must adjust its input distribution tree filters no later than it adjust its output forwarding.

#### 4.5.3. Pruning the Distribution Tree
  Each distribution tree should be pruned per VLAN, eliminating branches that have no potential receivers downstream. Multi-destination TRILL Data frames should only be forwarded on branches that are not pruned.

  Further pruning should be done in two cases:
  * IGMP [RFC3376], MLD [RFC2710], and MRD [RFC4286] messages, where these are to be delivered only to links with an IP multicast routers
  * other multicast frames derived from an IP multicast address that should be delivered only to links that have registered listeners, plus links that have IP multicast routers, except for IP multicast addresses that must be broadcast. 

  Each of these cases is scoped per VLAN.

  Let's assume that RBridge RBn knows that adjacencies (a, c, and f) are in the nickname1 distribution tree. RBn marks pruning information for each of the adjacencies in the nickname1-tree. For each adjacency and for each tree, RBn marks:
  * the set of VLANs reachable downstream
  * for each one of those VLANs, flags indicating whether there are IPv4 or IPv6 multicast routers downstream, and
  * the set of Layer 2 multicast addresses derived from IP multicast groups for which there are receivers downstream

#### 4.5.4. Tree Distribution Optimization
  RBridge must determine the VLAN associated with all native frames they ingress and properlt enfore VLAN rules in the emission of native frames at egress RBridge ports according to how those ports are configured and designated as appointed forwarders. RBridges should also prune the distribution tree of multi-destiantion frames according to VLAN. But, since they are not required to do such pruning, they may receive TRILL data or ESADI frames that should have been VLAN pruned earlier in the tree distribution. They silently discard such frames. A campus may contain some RBridge that prune distribution trees on VLAN and some that do not.

  The distribution is more complex for multicast. RBridge should analyze IP-derived native multicast frames, and learn and announce listeners and IP multicast routers for such frames. And they should prune the distribution of IP-derived multicast frames based on such learning and annoucements. But, they are not required to prune based on IP multicast listener and router attachment state. And, unlike VLANs, where VLAN attachment state of ports must be maintained and honored, RBridge are not required to maintain IP multicast listener and router attachment state.

  An RBridge that does not examine native IGMP [RFC3376], MLD [RFC2710], or MRD [RFC4286] frames that it ingresses must advertise that it has IPv4 and IPv6 IP multicast routers attached for all the VLANs for which it is an appointed listeners. This will cause all IP-derived multicast traffic to be sent to this RBridge for those VLANs. It then egresses that traffic onto the links for which it is appointed forwarder where the VLAN of the traffic matches the VLAN for which it is appointed forwarder on that link. (This may cause the supression of certain IGMP membership report messages from end stations, but that is not significant because any multicast traffic that such reports would be request will be sent to such end station under these circumstances.)

  A campus may contain a mixture of RBridges with different levels of IP-derived multicast optimization. An RBridge may receive IP-derived multicast frames that should have pruned earlier in the tree distribution. It silently discards such frames.

#### 4.5.5. Forwarding Using a Distribution Tree
  With full optimization, forwarding a multi-destinaion data frame is done as follows. References to adjacencies below do not include the adjacency on which a frame was received:
  * The RBridge RBn receives a multi-destination TRILL Data frame with inner VLAN-x and a TRILL header indicating that the selected tree is the nickname1 tree;
  * if the source from which the frame was received is not one of the adjacencies in the nickname tree for the specified ingress RBridge, the frame is dropped
  * else, if the frame is an IGMP or MLD announcement message or an MRD query message, then the encapsulated frame is forwarded onto adjacencies in the nickname1 tree that indicate there are downstream VLAN-x IPv4 or IPv6 multicast routers as appropriate
  * else, if the frame is for a Layer 2 multicast address derived from an IP multicast group, but its IP address is not the range of IP multicast address that must be treated as broadcast, the frame is forwarded onto adjacencies in the nickname1 tree that indicate there are downstream VLAN-x IP multicast routers of the corresponding type (IPv4 or IPv6), as well as adjacencies that indicate there are downstream VLAN-x receivers for that group address
  * else (the inner frame is for a Layer 2 multicast address not derived from an IP multicast group or an unknown destination or broadcast or an IP multicast address that is required to be treated as broadcast), the frame is forwarded onto an adjacency if and only if that adjacency is in the nickname ree, and mark as reaching VLAN-x links.

  For each link for which RBn is appointed forwarder, RBn additionally checks to see if it should decapsulate the frame and send it to the link in native form, or process the frame locally.

  TRILL ESADI frames will be delivered only to RBridges that are appointed forwarders for their VLAN. Such frames will be multicats throughout the campus, like other non-IP-derived multicast data frames, on the distribution tree chosen by the RBridge that created the TRILL ESADI frame, and pruned according to the Inner.VLAN ID. Thus, all the RBridge that are appointed forwarders for a link in that VLAN receive them.

### 4.6. Frame Processing Behavior
  This section describes RBridge behavior for all varieties of received frames, including how they are forwarded when appropriate. Native frames, TRILL frames, Layer 2 control frames. Processing may be organized or sequenced in a different way than described here as long as the result is the same.

  Corrupt frames, for example, frames that are not a multiple of 8 bits, are too short or long for the link protocol/hardware in use, or have a bad FCS are dicarded on receipt by an RBridge port as they are discarded on receipt at an IEEE 802.1 bridge port.

  Source address information ({VLAN, Outer.MacSA, port}) is learned by default from any frame with a unicast source address.

#### 4.6.1. Receipt of a Native Frame
  If the port is configured as disable or if end-station service is disabled on the port by configuring it as a trunk port or configuring it to use P2P Hellos, the frame is discarded.

  The ingress RBridge RB1 determines the VLAN ID for a native frame according to the same rules as IEEE [802.1Q-2005] bridges do. Once the VLAN is determined, if RB1 is not the appointed forwarder for that VLAN on the port where the frame was received or is inhibited, the frame is discarded. If it is appointed forwarder for that VLAN and is not inhibited, then the native frame is forwarded according to if it is unicast and according to if it is multicast or broadcast.

##### 4.6.1.1. Native Unicast Case
  If the destinattion MAC address of the native frame is a unicast address, the following steps are performed.

  The Layer 2 destination address and VLAN are loopked up in the ingress RBridge's database of MAC addresses and VLNAs to **find the egress RBridge RBm or the local egress port or to discover that the destination is receiving RBridge or is unknown**. one of following four cases will apply:
  * If the destination is the receiving RBridge, the frame is locally processed.
  * If the destination is known to be on the same link from which the native frame was received but is not the receiving RBridge, the RBridge silently discards the frame, sine the destination should already have received it.
  * If the destination is knwon to be on a different local link for which RBm is the appointed forwarder, then RB1 converts the native frame to a TRILL Data frame with an Outer.MacDA of the next hop RBridge towards RBm, a TRILL header with M = 0, an ingress nickname for RB1, and the egress nickname for RBm. If ingress RB1 has multiple nicknames, it should use the same nickname in the ingress nickname field whenever it encapsulates a native frame from any particular source MAC address and VLAN. This simplifies end node learning. If RBm is RB1, processing them proceeds, otherwise, the Outer.MacSA is set to the MAC address of the RB1 port on the path to the next hop RBridge towards RBm and the frame is queued for transmission out of that port.
  * If a unicast destination MAC is unknon in the frame's VLAN, RB1 handles the frame as described in xxx for a broadcast frame except that the Inner.MacDA is the original native frame's unicast destination address

##### 4.6.1.2. Native Multicast and Broadcast Frames
  If the RBridge has multiple ports attached to the same link, all but one received copy of a native multicast or broadcast is discarded to avoid duplication. All such frames that are part of the same flow must be accepted on the same port to avoid re-ordering.

  If the frame is a native IGMP [RFC3376], MLD [RFC2710], or MRD [RFC4286] frame, then RB1 should analyze it, learn any group membership or IP multicast router presence indicated, and announce that information for the appropriate VLAN in its LSP.

  For all multi-destination native frames, RB1 forwards the frame in native form to its links where it is appointed forwarder for the frame's VLAN, subject to further prunning and inhibition. In addition, it converts the native frame to a TRILL Data frame with All-RBridges multi-destination address as Outer.MacDA, a TRILL header with the multi-destination bit M = 1, the ingress nickname for RB1, and the egress nickname for the distribution tree it decide to use. It then forwards the frame on the pruned distribution tree setting the Outer.MacSA of each copy sent to the MAC address of the RB1 port on which it is sent.

  The default is for RB1 to write into the egress nickname field the nickanme for a distribution tree, from the set of distribution trees RB1 was annouced it might use, whose root is least cosr from RB1. RB1 may choose different distribution trees for different frames if RB1 has been configured to path-split multicast. In that case, RB1 must select a tree by specifying a nickname that is a distribution tree root. Also, RB1 must select a nickname that RB1 has announced to be one of those that RB1 might use. The strategy RB1 uses to select distributionn tree in multipathing multi-destination frames is beyond the scope of this document.

#### 4.6.2. Receipt of a TRILL Frame
  A TRILL frame either has the TRILL or L2-IS-IS Ethertype or has a multicast Outer.MacDA allocated to TRILL. The following tests are performed sequentially, and the first that macthes controls the handling of the frame:
  1. If the Outer.MacDA is All-IS-IS-RBridges and the Ethertype is L2-IS-IS, the frame is handled as in section 4.6.2.1
  1. If the Outer.MacDA is a multicast address allocated to TRILL other than All-RBridges, the frame is discarded
  1. If the Outer.MacDA is a unicast address other than the receiving RBridge port MAC address, the frame is discarded. (Such discarded frames are more likely addressed to another RBridge on a multi-access link and that other RBridge will handle them.)
  1. If the Ethertype is not TRILL, the frame is discarded
  1. If the Version field in the TRILL header is greater than 0, the frame is discarded
  1. If the hop count is 0, the frame is discarded
  1. If the Outer.MacDA is multicast and the M bit is zero or if the Outer.MacDA is unicast and M bit is one, the frame is discarded
  1. by default, an RBridge must not forward TRILL-encapsulated frames from a neighbor with which it does not have a TRILL IS-IS adjacency. RBridge may be configured per port to accept these frames for forwarding in cases where it is known that a non-peering device (such as an end station) is configured to originate TRILL-encapsulated frames that can be safely forwarded
  1. The Inner.MacDA is then tested. If it is the All-ESADI-RBridge multicast address and RBm implements the ESADI protocol, processing procceeds as in section 4.6.2.2 below. If it is other address or RBn does not implement ESADI, processing procceeds as in section 4.6.2.3

##### 4.6.2.1. TRILL Control Frames
  The frame is processed by the TRILL IS-IS instance on RBn and is not forwarded

##### 4.6.2.2. TRILL ESADI Frames
  If M = 0, the frame is silently discarded


  The egress nickname designate the distribution tree. The frame is forwarded as described in section 4.6.2.5. In addition, if the forwarding RBridge is an appointed forwarder for a link in the specified VLAN and implements the TRILL ESADI protcol and ESADI is enabled at the forwarding Rbridge for that VLAN, the inner frame is decapsulated and procided to that local ESADI protocol.

##### 4.6.2.3. TRILL Data Frames
  The M flag is the checked. If it is zero, processing continues as described in section 4.6.2.4, if it is one, processing continues as described in section 4.6.2.5

##### 4.6.2.4. Known Unicast TRILL Data Frames
  The egress nickname in the TRILL header is examined, and if it is unknown or reserved, the frame is discarded.

  If RBn is a transit RBridge, the hop count is decremented by one and the frame is forwarded to the next hop RBridge towards the egress RBridge. (The provision permitting RBridge to decrease the hop count by more than one under some circumstances applies only to multi-desttination frames, not to the known unicast frames considered in this subsection.)  The Inner.VLAN is not examined by a transit RBridge when it forwards a known unicast TRILL Data Frame.

  For forwarded frame, the Outer.MacSA is the MAC address of the transmitting port, the Outer.MacDA is the unicast address of the next hop RBridge, and the VLAN is the Designated VLAN on the link onto which the frame is being transmitted.

  If RBn is not a transmit RBridge, that is, if the egress RBridge indicated is the RBridge performing the processing, the Inner.MacSA and Inner.VLAN ID are, by default, learned as associated with the ingress nickname unless that nickname is unknown or reserved or the Inner.MacSA is not unicast. Then the frame being forwarded is decapsulated to native form, and the following checks are performed:
  * The Inner.MacDA is checked. If it is not unicast, the frame is discarded
  * If the Inner.MacDA corresponding to the RBridge doing the processing, the frame is locally delivered.
  * The Inner.VLAN ID is checked. If it is 0x0 or 0xFFF, the frame is discarded
  * The Inner.MacDA and Inner.VLAN ID are looked up in RBn's local address cache and the frame is then eithre sent onto the link containning the destination, if the RBridge is appointed forwarder for that link for the frame's VLAN and is not inhibited (or discarded if it is inhibited), or processed as in one of following two paragraphs

  A known unicast TRILL Data Frame can arrive at the egress RBridge only to find that the combination of Inner.MacDA and Inner.VLAN is not actually known by that Rbridge. One way this can happen is that address information may have timeout in the egress RBridge MAC address cache. In this case, the egress RBridge sends the native frame out on all links that are in the frame's VLAN for which the RBridge is appointed forwarder and has not been inhibited, except that it may refrain from sending the frame on links where it knwons there cannot be end station with the destination MAC address, for examplr, the link port is configured as a trunk.

  If due to manual configuration or learning from Layer 2 registration, the destination MAC and VLAN appear in RBn's local address cache for two or more links for which RBn is an uninhibited appointed forwarder for the frame's VLAN, RBn sends the native frame on all such links.

##### 4.6.2.5. Multi-Destination TRILL Data Frames
  The egress and ingress nicknames in the TRILL header are examined, and if either is unknown or reserved, the frame is discarded.

  The Outer.MacSA is checked and the frame disarded if it is not a tree adjacency for the tree indicated by the egress RBridge nickname on the port where the frame was received. The Reverse Path Forwarding Check is performed on the ingress and egress nicknames and the frame discarded if it fails. If there are multiple TRILL-Hello pseudonode suppressed parallel links to the previous hop RBridge, the frame is discarded if it has been received on the wrong one. If the RBridge has multiple ports connected to the link, the frame is discarded unless it was received one the right one. 

  If the Inner.VLAN ID of the frame is 0x0 or 0xFFF, the frame is discarded.

  If the RBridge is an appointed forwarder for the Inner.VLAN ID of the frame, the Inner.MacSA and Inner.VLAN ID are, by default, learned as associated with the ingress nickname unless that nickname is unknown or reserved otr the Inner.MacSA is not unicast. A copy of the frame is then decapsulated, sent in native form on those links in its VLAN for which the RBridge is appointed forwarder subject to additional pruning and inhibition as described in section 4.2.4.3, and/or locally processed as appropriate.

  The hop count is decreased (possible by more than one), and the frame is forwarded fown the tree specified by the egress RBridge nickname pruned as described in section 4.5

  For the forwarded frame, the Outer.MacSA is set to that of the port on which the frame is being transmitted, the Outer.MacDA is the All-RBridge multicast address, and the VLAN is the Designated VLAN of the link on which the frame is being transmitted.

#### 4.6.3. Receipt of a Layer 2 Control Frame
  Low-Level control frames received by an RBridge are handled within the port where they are received as described in section 4.9

  There are two types og high-level control frames, distinguished by their destination address, which are handled as described in the sections referenced below:
  
  Name|Section|Destination Address
  ----|-------|-------------------
  BPDU|4.9.3|01-80-C2-00-00-00
  VRP|4.9.4|01-80-C2-00-00-21

### 4.7. IGMP, MLD, MRD Learning
  Ingress RBridge should learn, based on native IGMP [RFC3376], MLD [RFC2710], and MRD [RFC4286] frames they received in VLANs for which they are uninhibited appointde forwarder, which IP-derived multicast messages should be forwarded onto which links. Such frames are also, in general, encapsulated as TRILL Data frames and distributed as described below and in section 4.5.

  An IGMP or MLD membership report received in native form from a link indicates a multicast group listener for that group on that link. An IGMP or MLG query or an MRD advertisement received in native form from a link indicates the presence of an IP multicast router on that link.

  IP multicast group membership reports have to be sent throughout the campus and delivered to all IP multicast routers, distinguishing IPv4 and IPv6. All IP-derived multicast traffic must also be sent to all IP multicast routers for the same version of IP.

  IP multicast data should only be sent on links where there is either an IP multicast router for that IP type (IPv4 or IPv6) or an IP multicast group listener for that IP-derived multicast MAC address, unless the IP multicast address is in the range requred to be treated as broadcast.

  RBridges do not need to annouce thenselves as listeners to be the IPv4 All-Snoopers multicast group (the group used for MRD reports), because the IPv4 multicast address for that group is in the range where all frames sent to that IP multicast address must be broadcast. However, RBridge that are performming IPv6-derived multicast optimization must announce themselves as listenr to the IPv6 All-Snoopers multicast group.

  See also "Considerations for Internet Group Management Protocol (IGMP) and Multicast Listener Discovery (MLD) Snooping Switches" [RFC4541].

### 4.8. End-Station Address Learning
  RBridges have to learn the MAC addresses and VLANs of their locally attached end stations for link/VLAN pairs for which they are the appointed forwarder. Learning this enables them to do the following:
  * Forward the native form of incoming known unicast TRILL Data frames onto the correct link
  * Decide, for an incoming native unicast frame from a link, where the RBridge is the appointed forwarder for the frame's VLAN, whether the frame is
    * Known to have been discarded for another end station on the same link, so the RBridge need do nothing, or
    * has to be converted to a TRILL Data frame and forwarded.

  RBridges need to learn the MAC addresses, VLANs, and remote RBridges of remotely attached end station for VLANs for which they and the remote Rbridge are an appointed forwarder, so they can efficiently direct native frames they receive that are unicast to those addresses and VLANs.

#### 4.8.1. Learning End-Station Addresses
  There are five independent ways an Rbridge can learn end-station addresses as follows:
  1. From the observation of VLAN-x frames received on ports where it is appointed VLAN-x forwarder, learning the {source MAC, VLAN, port} triplet of received frames
  1. The {source MAC, VLAN, ingress RBridge nickname} triplet of any native frames that it decapsulates
  1. By Layer 2 registration protocols learning the {source MAC, VLAN, port} of end stations registring at a local port
  1. By running the TRILL ESADI protocol for one or more VLANs and thereby receiving remote address information and/or transmitting local address information
  1. By management configuration

  RBridge must implement capabilities 1 and 2 above. RBridge use these capabilities unless configured, for one or more particular VLANs and/or ports, not to learn from either received frames or from decapsulating native frames to be transmitted or both.

  RBridges may implement capabilities 3 and 4 above. If capability 4 is implemented, the ESADI protocol is run only when the Rbridge is configured to do so on a per-VLAN basis.

  Rbridge should implement capability 5.

  Entries in the table of learned MAC and VLAN addresses and associated information also have a one-octet unsgined confidence level associated with each entry whose ratinale is given below. Such information learned from the observation of data has a confidence of 0x20 unless configued to have different confidence. This confidence level can be configured on a per-RBridge basis separately for information learned from local native frames and that learned from remotely originated encapsulated frames. Such information via the TRILL ESADI protocol is accompanied by a confidence level in the range 0 to 254. Such information configued by management defaults to a confidence level of 254 but may be configured to have another value.

  The table of learned MAC address includes:
  * {confidence, VLAN, MAC address, local port} for addresses learned from local native frame and local registration protocols
  * {confidence, VLAN, MAC address, egress RBridge nickname} for addresses leanred from remote encapsulated frames and ESADI link state databases
  * additional information to implement timeout of learned addresses, statically configured addresses, and the like

  When a new address and related inforamtion learn from observing data frames are to be entered into the local database, there are three possibilities:
  * If this is a new {address, VLAN} pair, the information is entered accompanished by the confidence level
  * If there is already an entry for this {address, VLAN} pair with the same accompanying delivery information, the confidence level in the local database is set to the maximum of its existing confidentce level and the confidence level with which it is being learned. In adition, if the information is being learned with the same or a higher confidence level than its existing confidence, timer information is reset
  * If there is already an entry for this {address, VLAN} pair with different information, the leanred information replaces the older information only if it is being learned with higher or equal confidence than that in that database entry. If it replaces older information, timer information is also reset.

#### 4.8.2. Learning Confidence Level Retionale
  The confidence level mechanism allows an RBridge campus manager to cause certain address learning sources to prevail over others. In default configuration, without the optional ESADI prortocol, addresses are only learned observing local native frames and the decapsulation of received TRILL Data frames. Both of these sources default to confidence level 0x20 so, since learning at an equal or high confidence ovrrides previous learning, the learning in such a deafult case mimics defualt 802.1 bridge learning.

  While RBridge campus management policies are beyond the scope of this document, here are some example types of policies that can be implemented with the confidence mechanism and the rationale for each:
  * Local Received native frames might be considered more reliable than decapsulated frames received from remote parts of the campus. To stop MAC addresses learned from such local frames from being usurped by remotely received forged frames, the confidence in locally learned addresses could be increased or that in addresses learned from remotely sourced decapsulated frames decreased.
  * MAC address information learned through a cryptographically authenticated Layer 2 registration protocol, such as 802.1X with a cryptographically based EAP method, might be considered more reliable than information learned through the mere observation of data frames. When such authenticated learned address information is transmitted via the ESADI protocol, the use of authentication in the TRILL ESADI LSP frames could make tampering with it in transit very difficult. As a result, it might be reasonable to announce such authenticated information via the ESADI protocol with a hight confidence, so it would override any alternative learning from data observation.

  Mannualy configured address information is generally considered static and so defaults to a confidence of 0xFF while no other source of such information can be configured to a confidence any higher than 0xFE. However, for other cases, such as where the mannual configuration is just a starting point that the RBridge campus manager wishes to be dynamically overridable, the confidence of such mannually configured information may be configured to a lower value.

#### 4.8.3. Forgetting End-Station Addresses
  While RBridges need to learn end-station addresses as described above, it is equally important that they be able to forget such information. Otherwise, frames form end stations that have moved to a different part of the campus could be indefinitely black-holed by RBridge with stale information as to the link to which the end station is attached.

  For end-station address information locally learned from frames received, the timeout from the last time a native frame was received or decapsulated with the information conforms to the recommendations of [802.1D]. It is refered to as the "Ageing Time" and it configurable per RBridge with a range of from 10 seconds to 1,000,000 seconds and a default value of 300 seconds.

  The situation is different for end-station address information acquired via the TRILL ESADI protocol. It is up to be originating RBridge to decide when to remove such information from its ESADI LSP (or up to ESADI protocol timeouts if the originating RBridge becomes inaccessible).

  When an Rbridge ceases to be appointed forwarder for VLAN-x on a port, it forgets all end-station address information learned from the observation of VLAN-x native frames received on that port. It also increments a per-VLAN counter of the number of times it lost appointed forwarder status on one of its ports for that VLAN.

  When, for all of its ports, RBridge RBn is no longer appointed forwarder for VLAN-x, it forgets all end-station address information learned from decapsulating VLAN-x native frames. Also, if RBn is particpating in the TRILL ESADI protocol for VLAN-x, it ceases to so participate after sending a final LSP nulling out the end-station address information for the VLAN that it has been originating. In addition, all other RBridge that are VLAN-x forwarder on at least one of their ports notice that the link state data for RBn has changed to show that it no longer has a link on VLAN-x. In response, they forget all end-station address information they have learned from decapsulating VLAN-x frames that show RBn as the ingress RBridge.

  When the appointed forwarder lost counter for RBridge RBn for VLAN-x is observed to increase via the TRILL IS-IS link state database but RBn continues to be an appointed forwarder for VLAN-x on at least one of its ports, every other RBridge that is an appointed forwadrer for VLAN-x modifiers the aging of all the addresses it has learned by decapsulating native frames in VLAN-x from ingress RBridge RBn as follows: the time remainning for each entry is adjusted to be no larger than a per-RBridge configuration parameter called "Forward Delay". This parameter is in the range of 4 to 30 seconds with a default value of 15 seconds.

#### 4.8.4. Shared VLAN Learning
  RBridge can map VLAN IDs into a smaller number of identifiers for purpoese of address learning, as [802.1Q-2005] bridges can. Then when a lookup is done in learning information, this identifier is used for matching in place of the VLAN ID. If the ID of the VLAN on which the address was learned is not retained, then there are the following consequences:
  * The Rbridge no longer has the information needed to participate in the TRILL ESADI protocol for the VLANs whose ID is not being retained
  * In cases where section 4.8.3 above requires the discarding of learned address information based on a particular VLAN, when the VLAN ID is not available for entries under a shared VLAN identifier, instead the time remaining for each entry under that shared VLAN identifier is adjusted to no longer than the RBridge's "Forward Delay"

  Although outside the scope of this specification, there are some Layer 2 features in which a set of VLANs has shared learning, where one of the VLANs is the "primary" and the other VLANs in the gorup are "secondaries". An example of this is whre traffic from different communities is separated using VLAN tags, and yet some resource (such as an IP router or DHCP router) is to be shared by all the communities. A method of implementing this feature is to give a VLAN tag, say Z, to a link containning the shared resource, and have the other VLANs, say A, C, and D, be part of the group {primary=Z, secondary=A, C, D}. An Rbridge, aware of this grouping, attahced to one of the secondary VLANs in the grroup also clains to be attached to the primary VLAN. SO an RBridge attached to A would claim to also attached to Z. An RBridge attached to the primary would claim to be attached to all the VLANs in the group.

  This document does not specify to how VLAN groups might be used. Only RBridge that participate in a VLAN group will be configured to known about VLAN group. However, to detect misconfiguration, an Rbridge configured to known about a VLAN group should report the VLAN group in its LSP.

### 4.9. RBridge Ports
  Section 4.9.1 below describes the several RBridge port configuration bits, section 4.9.2 gives a logical port structure in terms of frame processing, and section 4.9.3 and 4.9.4 describe tha handling of high-level control frames

#### 4.9.1. RBridge Port Configuration
  There are four per-port configuration bits as follows:
  * Disable port bit: When ths bit is set, all frames received or to be transmitted are disabled, with the possible exception of some Layer 2 control frames, that may be generated and transmitted or received and processed within the port. By default, ports are enabled

  * End-station service disable (trunk port) bit: When this bit is set, all native frames received on that port and all native frames that would have been sent on the port are discarded. (Note that, for this document, "native frames" does not include Layer 2 control frames.) By default, ports are not restracted to being trunk ports.

  If a port with end-station service disabled reports, in a TRILL Hello frame it sends out that port, which VLANs it provides end-station support for, it reports that there are none.

  * TRILL Traffic disable (access port) bit: If this bit is set, the goal is to avoid sending TRILL frames, except TRILL-Hello frames, on the port since it is intended only for native end-station traffic. by default, ports are not restrcted to being access ports. This bit is reported in TRILL-Hello, the DRB still appoints VLAN forwarders. However, usually no pseudonode is reported, and none of the inter-RBridge links associated with that link are reported in LSPs.

  If the DRB RB1 does not have this bit set, but neighbor RB2 on the link does have the bit set, then RB1 does not appoint RB2 as appointed forwarder for any VLAN, and none of the RBridge (including the pseudonode) report RB2 as a neighbor in LSPs.

  In some cases even though the DRB has the "access port" flag set, the DRB may choose to create a pseudonode for the access port. In this case, the other RBridge report connectivity to the pseudonode in their LSP, but the DRB sets are "overload" flag in the pseudonode LSP

  * Use P2P Hellos bit: If this bit is set, Hellos send on this port are IS-IS P2P Hellos. By default TRILL-Hellos are used. See Section 4.2.4.1 for more information on P2P links.

  The dominance relationship of these four configuration bits is as follows, where configuration bits to the left dominate those to the right. That is to say, when any pair of bits is asserted, inconsistencies in behavior they mandate are resolved in favor of behavior mendate by the bit to the left of the other bit in this list.

  Diable > P2P > Access > trunk

#### 4.9.2. RBridge Port Structure
  An Rbridge port can be modeled as having a lower-level structure similar to that of an [802.1Q-2005] bridge port as shown in Figure 11. In this figure, the double lines represent the general flow of the frames and informations while single lines represent information flow only. The dashed lines in connection with VRP (GVRP/MVRP) are to show that VRP support is optional. An actual RBridge port implementation may be structured in any way that provides the correct behavior.

  ![Detailed RBridge Port Model]()

  Low-level control frames are handled in the low-level port/linl control logic in the same way as in an [802.1Q-2005] bridge port. This can optionally include a variety of 802.1 or link specific protocols such as PAUSE (Annex 31B of [802.3]), link layer discovery [802.1AB], link aggregation [802.1AX], MAC security [802.1AE], or port-based access control [802.1X]. While handled at a low-level, These frames may effect higher-level processing. For example, a Layer 2 registration protocol may effect the confidence in learned addresses. The upper interface to this lower-level port control logic corresponds to the Internal Sublayer Services (ISS) in 802.1Q-2005.

  High-level control frames (BPDUs and, if supported, VRP frames) are not VLAN tagged. Although they extend through the ISS interface, they are not subject to port VLAN processing. Behavior on receipt of a VLAN tagged BPDU or VLAN tagged VRP frame is unspecified. If a VRP is not implemented, then all VRP frames are discarded. Handling of BPDUs is described in section 4.9.3. Handling of VRP frame is described in section 4.9.4.

  Frames other than Layer 2 control frames, that is, all TRILL and native frames, are subject to port VLAN and priority processing that is same as for an [802.1Q-2005] bridge. The upper interface to the port VLAN and priority processing corresponds to the Extended Internal Sublayer Service (EISS) in 802.1Q-2005.

  In this model, RBridge port processing below the EISS layer is identical to an [802.1Q-2005] bridge except for:
  * the handling of high-level control frames
  * that the discard of frames 
  
  have exceeded the Maximum Transit Delay is not mandatory but may be done

  As described in more detail elsewhere in this document, incoming native frames are only accepted if the RBridge is an uninhibited appointed forwarder for the frame's VLAN, after which they are normally encapsulated and forwarder; Outgoing native frames are usually obtained by decapsulation and are only output if the RBridge is an unhinbited appointed forwarder for the frame's VLAN.

  TRILL-Hellos, MTU-probes, and MTU-acks are handled per port and, like all TRILL IS-IS frames, are never forwarded. They can affect the appointed forwarder and inhibition logic as well as the RBridge's LSP.

  Except TRILL-Hellos, MTU-probes, and MTU-acks, all TRILL controls as well as TRILL data and ESADI frames are passed up to higher-level RBridge processing on receipt and passed down for transmission on creation or forwarding. Note that these frames are never blocked due to the appointed forwarder and inhibition logic, which affects only native frames, but there are additional filters on some them such as the Reverse Path Forwardomg Check.

#### 4.9.3. BPDU Handling
  If RBridge campus topology were static, RBridge would simply be end station from a bridging perspective, terminating but not otherwise interacting with spanning tree. However, there are reasons for RBridges to listen to and sometimes to transmit BPDUs as described below. Even when RBridge listen to and transmit BPDUs, this is a local RBridge port activity. The ports of a particular RBridge never interact so as to make the RBridge as a whole a spanning tree mode.

##### 4.9.3.1. Receipt of BPDUs
  RBridge must listen to spanning tree configuration BPDUs received on a port and keep track of the root bridge, if any, on that link. If MSTP is running on the link, ths is the CIST root. This information is reported per VLAN by the Rbridge in its LSP and may be used as described in Section 4.2.4.3. In addition, the receipt of spanning tree configuration BPDUs is used as indication that a link is a bridged LAN, which can affect the RBridge transmission of BPDUs.

  An RBridge must not encapsulate or forward any BPDU frame it receives.

  RBridge discard any topology change BPDUs they receive, but not section 4.9.3.3

##### 4.9.3.2. Root Bridge Changes
  A change in the root bridge seen in the soanning tree BPDUs received at an RBridge port may indicate a change in bridged LAN topology, including the possibility of the merger of two bridged LANs or the like, without any physical indication at the port. During topology transients, bridges may go into per-forwarding states that block TRILL-Hello frames. For there reasons, when an RBridge sees a root bridge change on a port for which it is appointed forwarder for one or more VLNAs, it is inhibited for a period of time between zero and 30 seconds. (An inhibited appinted forwarder discards all native frames received from or that it would otherwise have sent to the link.) this time period is configurable per port and defaults to 30 seconds.

  For example, consider two bridged LANs carrying multiple VLANs, each with variouts RBridge appointed forwarders. Should they become merged, due to a cable being plugged in or the like, those RBridges attached to the original bridged LAN with the lower priority root will see a root bridge change while those attached to the other original bridged LAN will not. Thus, all appointed forwarders in the lower priority set will be inhibited for a time period while things are sorted out by BPDUs within the merged bridged VLAN and TRILL-Hello frames between the Rbridge involved.

##### 4.9.3.3. Transmission of BPDUs
  When an RBridge ceases to be appointed forwarder for one or more VLANs out a particular port, it should, sa long as it continues to receive spanning tree BPDUs on that port, send topolgy change BPDUs until it sees the topolgy change acknownledged in a spanning tree configuration BPDU.

  Rbridge may support a capability for sending spanning tree BPDUs for the purpose of attempting to force a bridged LAN to partition as discussed in Appendix A.3.3

#### 4.9.4. Dynamic VLAN Registration
  Dynamic VLAN registration provides a means for bridges (and less commonly end station) to request that VLANs be endbaled or disabled on ports leading to the requestor. This is done by VLAN registration protocol (VRP) frames: GVRP or MVRP. RBridge may implement GVRP and/or as escribed below.

  VRP frames are never encapsulated as TRILL frames between RBridge or forwarded in native form by an Rbridge. If an RBridge does not implement a VRP, it discards any VRP frames received and sends none.

  RBridge ports may have dynamically enabled VLANs. If an RBridge supports a VRP, the actual enablement of dynamic VLANs is determined by GVRP/MVRP frames received at the port as it would be for an [802.1Q-2005]/[802.1ak] bridge.

  An Rbridge that supports a VRP sends GVRP/MVRP frames as an [802.1Q-2005]/[802.1ak] bridge would send on each port that is not configured as an RBridge trunk port or P2P port. For this purpose, it sends VRP frames to request traffic in the VLANs for which it is appointed forwarder and in the Designated VLAN, unless the Desinated VLAN is disabled on the port, and to not request traffic in any other VLAN.

