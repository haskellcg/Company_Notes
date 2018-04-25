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
### 4.6. Frame Processing Behavior
#### 4.6.1. Receipt of a Native Frame
##### 4.6.1.1. Native Unicast Case
##### 4.6.1.2. Native Multicast and Broadcast Frames
#### 4.6.2. Receipt of a TRILL Frame
##### 4.6.2.1. TRILL Control Frames
##### 4.6.2.2. TRILL ESADI Frames
##### 4.6.2.3. TRILL Data Frames
##### 4.6.2.4. Known Unicast TRILL Data Frames
##### 4.6.2.5. Multi-Destination TRILL Data Frames
#### 4.6.3. Receipt of a Layer 2 Control Frame
### 4.7. IGMP, MLD, MRD Learning
### 4.8. End-Station Address Learning
#### 4.8.1. Learning End-Station Addresses
#### 4.8.2. Learning Confidence Level Retionale
#### 4.8.3. Forgetting End-Station Addresses
#### 4.8.4. Shared VLAN Learning
### 4.9. RBridge Ports
#### 4.9.1. RBridge Port Configuration
#### 4.9.2. RBridge Port Structure
#### 4.9.3. BPDU Handling
##### 4.9.3.1. Receipt of BPDUs
##### 4.9.3.2. Root Bridge Changes
##### 4.9.3.3. Transmission of BPDUs
#### 4.9.4. Dynamic VLAN Registration
