# Routing Bridges (RBridges): Base Protocol Specification

## Abstract
  Routing Bridges (RBridges) provide **optimal pair-wise forwarding without configuration**, safe frowarding even during periods of temporary loops, and support for multipathing of both unicast and multicast traffic. They achieve thee goals using IS-IS routing and encapsulation of traffic with a header that includes a hop count.

  RBridges are compatible with previous IEEE 802.1 customer bridges as well as IPv4 and IPv6 routers and end nodes. They are as **invisible to current IP routers as bridges are and, like routers**, they terminate the bridge spanning tree protocol.

  The design support VLANs and the optimization of the distribution of multi-destination frames based on VLAN ID and based on IP-derived multicast groups. It also allows unicast forwarding table at transit RBridges to be sized according to the number of RBridges (rather than the number of end nodes), which allows their forwarding tables to be substantially smaller than in conventional customer bridges.

## 1. Introdcution
  In tranditional IPv4 and IPv6 networks, **each subnet has a unique prefix**. Therefore, a node in multiple subnets has multiple IP addresses, typically one per interface. This also means that when a interface moves from one subnet to another, it changes its IP address. Administration of IP networks is complicated because IP routers require per-port subnet address configuration. Careful IP address management is required to avoid creating subnets that are sparsely populated, wasting addresses.

  IEEE 802.1 bridges avoid these problems by **transparently gluing many physical links into what appears to IP to be a single LAN**. Howeverm 802.1 bridge forwarding using the spanning tree protocol has some disadvantages:
  * The spanning tree protocol works by blocking ports, limiting the number of forwarding links, and therefore creates **bottlenecks by concentrating traffic onto select links**.
  * Forwarding is not pair-wise shortest path, but is instead whatever path remains after the spanning tree eliminates redundant paths.
  * The Ethernet header does not contain a hop count (or Time to Line (TTL)) field. This is dangerous when there are temporary loops such as **when spanning tree manages are lost or components such as repeaters are added**.
  * VLANs are partition when the spanning tree configuration

  **This document presents the design for RBridges (Routing Bridges [RBridges]) that implement the TRILL protocol and are poetically summarized below**. RBridges combine the advantages of bridges and routers and, as specified in this document, are the application of link state routing to the VLAN-aware customer bridging problem.

  **With the exceptions discussed in this document, RBridges can incrementally replace IEEE [802.1Q-2005] or [802.1D] customer bridges**.

  While RBridges can be applied to a variety of link protocols, this specification focuses on IEEE [802.3] links. Use with other link type is expected to be covered in other docuements.

  The TRILL protocol, as specified herein, is desinged to be a Local Area Network protocol and not designed **with the goal of scaling beyond the size of existing briged LANs**.

### 1.1. Algorhyme V2, by Ray Perlner
  I hope that we shall one day see  
  A graph more lovely than a tree.

  A graph to boost efficiency  
  While still configuration-free.

  A network where RBridges can  
  Route packets to their target LAN.

  The paths they find, to our elation,  
  Are least cost paths to destination!

  With packet hop counts we now see,  
  The network need not be loop-free!

  RBridges work transparently,  
  Without a common spanning tree.

### 1.2. Normative Content and Precedence
  The bulk of the normative metarial in this specification appears in 1 through 4.

### 1.3. Terminology and Notation in This Document
  "TRILL/RBridge/RBridge"

  "link/simple link/port/physical port"

  "campus/RBridge campus/spanning tree[classic/rapid]"

  "IETF bit ordering"

### 1.4. Categories of Layer 2 Frames
  In this document, Layer 2 frames are divided into five categories:
  * Layer 2 control frames (such as Bridge PDUs (BPDUs))
  * native frames (non-TRILL-encapsulated data frames)
  * TRILL Data frames (TRILL-encapsulated data frames)
  * TRILL control frames
  * TRILL other frames

  The way these five types of frames are distinguished is as following:
  * Layer 2 control frames are those with **a multicast destination address in the range 01-80-C2-00-00-00 to 01-80-C2-00-00-0F or equal to 01-80-C2-00-00-21**. RBridges must not encapsulate and forward such frames, though they MAY, unless otherwise specified in this document, perform the Layer 2 function (such as MAC-level security) of the control frame. Frames with a destination address of **01-80-C2-00-00-00 (BPDU) or 01-80-C2-00-00-21 (VLAN Registration Protocol)** are called "high level control frames" in this document. All other Layer 2 control frames are called "low level control frames"
  * Native Frames are those that are not control frames and have an **Ethertype other than "TRILL"or "L2-IS-IS" and have a destination MAC address that is not one of the 16 multicast addresses reserved for TRILL**.
  * TRILL Data frames have the Ethertype "TRILL". In addition, TRILL data frames, if multicast, have the multicast destination MAC address "All-RBridges".
  * TRILL control frames have the Ethertype "L2-IS-IS". In addition, TRILL control frames, if multicast, have the multicast destination MAC addresses of "ALL-IS-IS-RBridges". (Note that **ESADI frames** look on the outside like TRILL data and are so handled but, when decapsulated, have the L2-IS-IS Ethertype.)
  * TRILL other frames are those with any of the 16 multicast destination addresses reserved for TRILL other than All-RBridges and All-IS-IS-RBridges. RBridges conformant to this specification must discard TRILL other frames.

### 1.5. Acronyms
  * AllL1ISs - All Level 1 Intermediate Systems
  * AllL2ISs - All Level 2 Intermediate Systems
  * BPDU - Bridge PDU
  * CHbH - Critical Hop-by-Hop
  * CItE - Critical Ingree-to-Egress
  * CSNP - Complete Sequence Number PDU
  * DA - Destination Address
  * DR - Destination Router
  * DRB - Destination RBridge
  * EAP - Extensible Authentication Protocol
  * ECMP - Equal Cost Multipath
  * EISS - Extended Internal Sublayer Service
  * ESADI - End-Station Address Distribution Information
  * FCS - Frame Check Sequence
  * GARP - Generic Attribute Registration Protocol
  * GVRP - GARP VLAN Registration Protocol
  * IEEE/IGMP/IP/IS-IS/LAN/P2P/PDU/PPP/RBridge
  * ISS - Internal Sublayer Service
  * LSP - Link State PDU
  * MAC - Media Access Control
  * MLD - Multicast Listener Discovery
  * MRD - Multicast Router Discovery
  * MTU - Maximum Transmission Unit
  * MVRP - Multiple VLAN Registration Protocol
  * NSAP - Network Service Access Point
  * RPF - Reverse Path Forwarding
  * SA - Source Address
  * SNMP - Simple Network Management Protocol
  * SPF - Shortest Path First
  * TLV - Type, Length, Value
  * TRILL - Transparent Interconnection of Lots of Links
  * VLAN - Virtual Local Area Network
  * VRP - VLAN Registration Protocol

## 2. RBridges
  This section provides a high-level overview of RBridges, which implement the TRILL protocol, omitting some details.

  TRILL, provides [802.1Q-2005] VLAN-aware customer bridging service. As described below, TRILL is layered above the ports of an RBridge.

  The RBridges specified by this document do not supply provider [802.1ad] or provider backbone [802.1ah] bridging or the like. The extension of TRILL to provide such provider services is left for furture work that will be separately documented. However, provider or provider backbone may be used to interconnect parts of an RBRidge campus.

### 2.1. General Overview
  RBridges run a link state protocol amongest themselves. This give then enough information to compute pair-wise optimal paths for unicast, and calculate distribution trees for delivery of frames either to destinations whose location is unknown or to multicast/broadcast groups.

  To mitigate temporary loop issues, RBridges forward based on a header with a hop count. RBridges also specify the next hop RBridge as the frame destination when forwarding unicast frames across a shared-media link, **which avoids spawning additional copies of frames during a temporary loop**. **A Reverse Path Forwarding Check** and other checks are performed on multi-destination frames to further control potentially looping traffic.

  The first RBridge that a unicast frame encounters in a campus, RB1, **encapsulates the received frame with a TRILL header that specifies the last RBridge, RB2**, where the frame is decaosulated. RB1 is known as the "ingress RBridge" and RB2 is known as the "rgress RBridge"

  To save toom in the TRILL header and simplify forwarding loopups, **a dynamic nickname acquisition protocol is run among the RBridges to select 2-octet nicknames for RBridges**, unique within the, which are an abbreviation for the IS-IS ID of the RBridge. The 2-octet nicknames are used to specify the ingree and egress RBridges in the TRILL header.

  Multipathing of multi-destination frames through alternative distribution trees and ECMP (Equal Cost MultiPath) of unicast frames are supported.

  Networks with a more mesh-like structure will benefit to a greater extent from the multipathing and optimal paths provided by TRILL than will more tree-like networks.

  RBridges run a protocol on a link to elect a "Designate RBridge" (DRB). The TRILL-IS-IS election protocol on a link is a little different from the layer 3 IS-IS election protocol, because in TRILL it is essential that only one RBridge be elected DRB, whereas in layer 3 IS-IS it is possible for multiple routers to elected Designated Router (also known as Designated Intermediate System). **As with an IS-IS router, the DRB may give a pseudonode name to the link, issue an LSP (Link State PDU)** on  behalf of the pseudonode, and issue CSNPs (Complete Sequence Number PDUs) on the link. Additionally, the DRB has some TRILL-specific duties, **including specifying which VLAN will be the Designate VLAN used for communication between RBridges on the link**.

  The DRB either encapsulates/decapsulates all data traffic to/from the link, or, for load splitting, delegates this responsibility, for one or more VLANs, to other RBridges on the link. There must at all times be at most one RBridge on the link that encapsulates/decapsulates traffic for a particular VLAN. We will refer to the RBridge appointed to forward VLAN-x traffic on behalf of the link as the "appointed VLAN-x forwarder".

  RBridges should support **SNMPv3**. The RBridge MIB will be specified in the separate document. If IP service is available to an RBridge, it should support SNMPv3 over UDP over IPv4 and IPv6, however, management can be used, within a campus, even for an RBridge that lacks an IP or other Layer 3 transpot stack or which does not have a Layer 3 address, by **transporting SNMP with Ethernet**.

### 2.2. End-Station Addresses
  An RBridge, RB1, that is the VLNA-x forwarder on any of its links must learn the location of VLAN-x end nodes, both on the links for which it is VLAN-x forwarder and on other links in the campus. RB1 learns the port, VLAN, and layer 2 (MAC) addresses of end nodes on links for which it is VLAN-x forwarder from the source address of frames received, as bridges do, or through configuration or a layer 2 explicit registration protocol such as IEEE 802.11 association and authentication. RB1 learns the VLAN and layer 2 address of distant VLAN-x end nodes, and corresponding RBridge to which they are attached, by looking at the RBridge nickname in the TRILL header and the VLAN and source MAC address of the inner frame of TRILL data frames that it decapsulates.

  Additionally, an RBridge that is the appointed VLAN-x forwarder on one or more links may use  the End-Station Address Distribution Information (ESADI) protocol to announce some or all of the attached VLAN-x end nodes on those links.

  The ESADI protocol could be used to announce end nodes that have been explicitely enrolled. Such information might be more authoritative than that learned from data frames being decapsulated on the link. Also the addresses enrolled and distributed in this way can be more secure for two reasons:
  * the enrollment might be authenticated (for example, by cryptographically based EAP methods via [802.1X])
  * the ESADI protocol also supports cryptographic authentication of its messages \[RFC5304\] [RFC5310] for more secure transmission

  If and end station is unpluged from one RBridge and plugged into another, then, depending on circumstances, **frames addressed to that end syayion cna be black-holed**. That is, they can be sent just to the older RBridge that the end station used to be connected to until cached address information at some remote RBridge(s) times out, possibly for a number of minutes or longer. With the ESADI protocol, the link interruption from the unplugging can cause an immediate update to be sent.

  Even if the ESADI protocol is used to announce or learn attached end nodes, RBridges must strill learn from received native frames and decapsulated TRILL Data Frames unless configured not to do so. Advertising end nodes using ESADI is optional, as is learning from these announcements.

### 2.3. RBridge Encapsulation Architecture
  The Layer 2 technology used to connect RBridges may be either IEEE [802.3] or some other link technology such as PPP [RFC1661]. This is possible since the RBridge relay function is layered on top of the Layer 2 technologies.

                      ------------
                     /            \
        +-----+     /  Ethernet    \    +-----+
        | RB1 |----<                >---| RB2 |
        +-----+     \   Cloud      /    +-----+
                     \            /
                      ------------

  Figure 1 shows two RBridges, RB1 and RB2, interconnected through an Ethernet cloud. The Ethernet cloud may include hubs, point-to-point or shared media, IEEE 802.1D bridges, or 802.1Q bridges.

        +--------------------------------+
        | Outer Ethernet Header          |
        +--------------------------------+
        | TRILL Header                   |
        +--------------------------------+
        | Inner Ethernet Header          |
        +--------------------------------+
        | Ethernet Payload               |
        +--------------------------------+
        | Ethernet FCS                   |
        +--------------------------------+

  Figure 2 shows the format of a TRILL data or ESADI frame traveling through the Ethernet cloud between RB1 and RB2.

        +--------------------------------+
        | PPP Header                     |
        +--------------------------------+
        | TRILL Header                   |
        +--------------------------------+
        | Inner Ethernet Header          |
        +--------------------------------+
        | Ethernet Payload               |
        +--------------------------------+
        | PPP FCS                        |
        +--------------------------------+

  In the case of media different from Ethernet, the header specific to that media replaces the oter Ethernet header. For example, Figure 3 shows a TRILL encapsulation over PPP.

  The outer header is link-specific and, although this document specifics only [802.3] link, other links are allowed.

  In both cases, the inner Ethernet header and the Ethernet Payload come from the original frame and are encapsulated with a TRILL header as they travel between RBridges. Use of a TRILL header offers the following benefits:
  * loop nitigation through use of a hop count field
  * elimination of the need for end-station VLAN amd MAC address learning in transit RBridges
  * direction of unicast frames towards the egress RBridges (this enables unicast forwarding tables of transit RBridges to be sized with the number of RBridges rather than the total number of nodes); and
  * provision of a separate VLAN tag for forwarding traffic between RBridges, independent of the VLAN of the native frame

  When forwarding unicast frames beween RBridges, the outer header has the MAC destination address of the next hop RBridge, to avoid frame duplication if the inter-RBridge link is multi-access. This also enables multipathing of unicast, since the transmitting RBridge can specify the next hop. Having the outer header specify the transmitting RBridge as the source address ensures that any bridges inside the Ethernet cloud will not get confused, as they might be if multipathing is in use and they were to see the original source or ingress RBridge in the outer header.

### 2.4. Forwarding Overview
  RBridge are ture routers in the sense that, in the forwarding of a frame by a transit RBridge, **the outer Layer 2 header is replaced at each hop with an appropriate Layer 2 header for the next hop, and hop count decreased**. Despite these modifications of the outer Layer 2 header and the hop count in the TRILL header, the original encapsulated frame is preserved, including the original frame's VLAN tag.

  From a forwarding standpoint, transit frames may be classified into two categories: known-unicast and multi-destination. **Layer 2 control frames and TRILL control and TRILL other frames ate not transit frames**, are not forwarded by RBridges, and are not included in these categories.

#### 2.4.1. Known-Unicast
  These frames have a unicast inner MAC destination address (Inner.MacDA) and are those for which the ingress RBridge knows the egress RBridge for the destination MAC address in the frame's VLAN.

  Such frames are forwarded RBridge hop by RBridge hop to their egress RBridge.

#### 2.4.2. Multi-Destination
  These are frames that must be delivered to multiple destinations.

  Multi-destination frames include the following:
  * unicast frames for which the location of the destination is unknow: the Inner.MacDA is unicast, but the ingress RBridge does not know its location in the frame's VLAN
  * multicast frames for which the Layer 2 destination address is derived from an IP multicast address: the Inner.MacDA is multicast, from the set of Layer 2 multicast addresses derived from IPv4 [RFC1112] or IPv6 [RFC2464] multicast addresses. These frames are handled somewhat differently in different subcases:
    * IGMP [RFC3376] and MLD [RFC2710] multicast group membership reports
    * IGMP [RFC3376] and MLD [RFC2710] quries and MRD [RFC4286] announcement messages
    * other IP-derived Layer 2 multicast frames
  * multicast frames for which the Layer 2 destnation address is not derived from an IP multicast address: the Inner.MacDA is multicast, and not from the set of Layer 2 multicast addresses derived from IPv4 or IPv6 multicast addresses
  * broadcast frames: the Inner.MacDA is broadcast (FF-FF-FF-FF-FF-FF)

  RBridges build **distribution trees and use these trees for forwarding multi-destination frames**. Each distribution tree reaches all RBridges in the campus, is shared across all VLANs, and maybe used for the distribution of a native frame that is in any VLAN. However, the distribution of any particular frame on a distribution tree is pruned in different ways for different cases to avoid unnecessary propagation of the frame.

### 2.5. RBridges and LANs
  A VLAN is a way to partition end nodes in a campus into different Layer 2 communities [802.1-2005]. Use of VLANs requires configuration. By default, the port of receipt determines the VLAN of a frame sent by an end station. End station can also explicitly insert this information in a frame.

  IEEE [802.1Q-2005] bridges can be configured to support multiple customer VLANs over a single simple link by inserting/removing a VLAN tag in the frame. VLAN tags used by TRILL have the same format as VLAN tags defined in IEEE [802.1Q-2005]. As shown in Figure 2, there are two places where such tags may be present in a TRILL-encapsulated frame sent over an IEEE [802.3] link: one in the outer header (Outer.VLAN) and one in inner header (Inner.VLAN).

  RBridges enforce delivery of a native frame originating in a particular VLAN only to other links in the same VLAN; however, there are a few differences in the handling of VLANs between an RBridge campus and an 802.1 bridged LAN as described below.

#### 2.5.1. Link VLAN Assumptions
  Certain configurations of bridges may cause partitions of a VLAN on a link. For such configurations, a frame sent by one RBridge to a neighbor on that link might not arrive, if tagged with a VLAN that is partitioned due to bridge configuration.

  TRILL requires at least one VLAN per link that gives full connectivity to all the RBridge on that link. The default VLAN is 1, though RBridges may be configured to use a different VLAN. The DRB dictates to the other RBridges which VLAN to use.

  Since there will be only one appinted forwarder for any VLAN, say, VLAN-x, on a link, if bridges are configured to cause VLAN-x to be partitioned on a link, some VLAN0-x end nodes on that link may be orphaned (unable to communicate with the rest of the campus).

  It is possible for bridge and port configuration to cause **VLAN mapping on a link** (where a VLAN-x frame turns into a VLAN-y frame). TRILL detects this by inserting a copy of the outer VLAN into TRILL Hello messages and checking it on receipt. If detected, it takes steps to ensure that there is at most a single appointed forwarder on the link, to avoid possible frame duplication or loops.

  TRILL behaves as conservatively as possible, avoiding loops rather than avoiding partial connectivity. As a result, lack of connectivity may result from bridge or port misconfiguration.

### 2.6. RBridges and IEEE 802.1 Bridges
  RBridges ports are, except as describe below, layered on top of IEEE [802.1Q-2005] port facilities.

#### 2.6.1. RBridge Ports and 802.1 Layering
  RBridges ports make use of [802.1Q-2005] port VLAN and priority processing. In addition, they may implement other lower-level 802.1 prrotocols as well as protocols for the link in use, such as PAUSE (Annex 31B of [802.3]), port-based access control [802.1X], MAC security [802.1AE], or link aggregation [802.11AX].

  However, RBridges do not use spanning tree and do not block ports as spanning tree does. Figure 4 shows a high-level disgram of an RBridge with one port connected to an IEEE 802.3 link. Single lines represent the flow of control information, double lines the flow of both frames and control information.

  The upper interface to the lower-level port/link control logic corresponds to the Internal Sublayer Service (ISS) in [802.1Q-2005]. In RBridges, high-level control frames are processed above the ISS interface.

  The upper interface to the port VLAN and priority processing corresponds to the Extended Internal Sublayer Service (EISS) in [802.1Q-2005]. In RBridges, native and TRILL frames are processed above the EISS interface and are subject to port VLAN and priority processing.

#### 2.6.2. Incremental Deployment
  Because RBridge are compatible with IEEE [802.1Q-2005] customer bridges, except as discussed in this document, a bridge LAN can be upgraded by incrementally replcing such bridges with RBridge. Bridges that have not yet been replaced are transparent to RBridge traffic. The physical links directly interconnected by such bridges, together with the bridges themselves, constitute bridged LANs. These bridged LANs apear to RBridges to be multi-access links.

  If the bridges replaced by RBridges were default configuration bridges, then theri RBridge replacements will not require configuration.

  Because RBridges, as described in this document, only provide customer services, they cannot replace provider bridges or provider backbone bridges, just as a customer bridge can't replace a provider bridge. However, **such provider devices can be part of the bridged LAN between RBridges**. Extension of TRILL to support provider services is left for future work and will be separately documented.

  Of course, if the bridges replaced had any port level protocols enabled, such as port-based access control [802.1X] or MAC security [802.1AE], replacement RBridge would **need the same port level protocols enabled and similarly configured**. In addition, the replacement RBridges would have to support the same link type and link level protocols as the replaced bridges.

  An RBridge campus will work best if all IEEE [802.1D] and [802.1Q-2005] bridges are replaced with RBridges, assuming the RBridges have the same speed and capacity as the bridges. However, there may be intermediate states, where only some bridges have been replaced by RBridges, with inferior performence.


## 3. Details of the TRILL Header
  This section specifies the TRILL header. Section 4 below provides other RBridge design details.

### 3.1. TRILL Header Formart
  The TRILL header is shown in Figure 5 and is independent of the data link layer used. When that layer is IEEE [802.3], it is prefixed with the 16-bit TRILL Ethertype [RFC5342], making it 64 bit aligned. If Op-Length is a multiple of 64 bits, then 64-bit alignment is normally maintained for the content of an encapsulated frame.

                                  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                  | V | R |M|Op-Length| Hop Count |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        | Egress RBridge Nickname | Ingress RBridge Nickname      |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        | Options...
        +-+-+-+-+-+-+-+-+-+-+-+-

  * V (Version): 2-bit unsigned integer
  * R (Reserved): 2 bits
  * M (Multi-destination): 1 bit
  * Op-Length (Options Length): 5-bit unsigned integer
  * Hop Count: 6-bit unsigned integer
  * Egress RBridge Nickname: 16-bit identifier
  * Ingress RBridge Nickname: 16-bit identifier
  * Options: present if Op-Length is non-zero

### 3.2. Version (V)
  Version (V) is a 2-bit field. Version zero of TRILL is specified in this document. An RBridge RB1 must check the V field in a received TRILL-encapsulate frame. If the V field has a value not recognized by RB1, then RB1 must silently discard the frame. The allocation of new TRILL Version numbers requires an IETF Standards Action.

### 3.3. Reserved (R)
  The two R bits are reserved for future use in extensions to this version zero of the TRILL protocol. They must be set to zero when the TRILL header is added by an ingress RBridge, transparently copied but otherwise ignored by transit RBridges, and ignored by egress RBridges. The allocation of reserved TRILL header bits requires an IETF Standards Action.

### 3.4. Multi-Destination (M)
  The Multi-destination bit indicates that the frame is to be delivered to a class of destination end station via a distribution tree and that the egress RBridge nickname field specifies this tree. In particular:
  * M = 0 (False) - The egress RBridge nickname contains a nickname of the egress RBridge for a knwon unicast MAC address
  * M = 1 (True) - The egress RBridge nickname field contains a nickname that specifies a distribution tree. This nickname is selected by the ingress RBridge for a TRILL Data frame or by the source RBridge for a TRILL ESADI frame.

### 3.5. Op-Length
  There are provisions to express in the TRILL header that a frame is using an optional capapbility and to encode information into header in connection with that capapbility.

  The Op-Length header field gives the length of the TRILL header options in units of 4 octets, which allows up to **124 octets of option area**. If Op-Length is zero, there are no options present. If options are present, they follow immediately after the Ingress RBridge Nickname field.

### 3.6. Hop Count
  The Hop Count field is a 6-bit unsigned integer. An RBridge drops frames received with a hop count of zero, otherwise it decrements the hop count. (This behavior is different from IPv4 and IPv6 in order to support the later addition of a traceroute-like facility that would be able to get a hop execeeded from an egress RBridge.)

  For known unicast frames, the ingress RBridge should set the Hop Count in excess of the number of RBridge hops it expects to the egress RBridge to allow for alternate routing later in the path.

  For multi-destination frames, the Hop Count should be set by the ingress RBridge (or source RBridge for a TRILL ESADI frame) to at least the expected number of hops to the most distant RBridge. To accomplish this, RBridge RBn calculates, for each branch from RBn of the specified distribution tree rooted at RB1, the maximum number of hops in that branch.

  Multi-destination frames are of particular danger because **a loop involving one or more distribution tree forks could result in the rapid generation of mutliple copies of the frame, even with the normal hop count mechanism**. It is for this reason that multi-destination frames are subject to **a stringent Reverse Path Forwarding Check and other checks as described**. As an optional additional traffic control measure, when forwarding a multi-destination frame onto a distribution tree branch, **transit RBridge RBm May decrease the Hop count by more than 1**, unless decreasing the hop count by more than 1 would result in a hop count insufficent to reach all destinations in that branch of the tree rooted at RBi.

  Using a hop count close or equal to the minimum needed on multi-destination frames provides additional protection against problems with temporary loops when forwarding.

  Although the RBridge may decrease the hop count of multi-destination frames by more than 1, under the circumstances above, the RBridge forwarding a frame must decrease the hop count by at least 1, and discard the frame if cannot do so because the hop count is 0. The option to decrease the hop count by more than  1 under the circumstances described above applies only to multi-destination frames, not to known unicast frames.

### 3.7. RBridge Nicknames
  Nicknames are 16-bit dynamically assigned quantities that act as abbreviations for RBridges' IS-IS IDs to achieve a more compact encoding and can be used to specify potentially different trees with the same root. This assignment allows specifying up to 2\*\*16 RBridges; However, the value **0x0000** is reserved to indicate that a nickname is not specified, the value **0xFFC0 through 0xFFFE** are reserved for future specification, and the value **0xFFFE** are permanently reserved. RBridges piggyback a nickname acquisition protocol on the link state protocol to acquire one or more nicknames unique within the campus.

#### 3.7.1. Egress RBridge Nickname
  There are two cases for the contents of the egress RBridge nickname field, depending on the M bit. The nickname is filled in **by the ingress RBridge for TRILL Data frames** and **by the source RBridge for TRILL ESADI frames**.
  * For known unicast TRILL Data frames, M == 0 and the egress RBridge nickname field specifies the egress RBridge; that is, it specifies the RBridge that needs to remove the TRILL encapsulation and forward the native frame. Once the egress nickname field is set, it must not be changed by any subsequent transit RBridge.
  * For multi-destination TRILL Data frames and for TRILL ESADI frames, M == 1. The egress RBridge nickname field contains a nickname specifying the distribution tree selected to be used to forward the frame. This root nickname must not be changed by transit RBridge.

#### 3.7.2. Ingress RBridge Nickname
  The ingress RBridge nickname is set to a nickname of the ingress RBridge for TRILL Data frames and to a nickname of the source RBridge for TRILL ESADI frames. If the RBridge setting the ingress nickname has multiple nicknames, it should use the same nickname in the ingress field whenever it encapsulates a frame with any particular Inner.MacSA and Inner.VLAN value. Thsi simplifies end node learning.

#### 3.7.3. RBridge Nickname Selection
  The nickname selection protocols is piggybacked on TRILL IS-IS as follows:
  * The nickname or nicknames beding used by an RBridge are carried in an IS-IS TLV (type-length-value data element) along with a priority of use value [RFC6326]. Each RBridge chooses it own nickname or nicknames.
  * Nickname values may be configured. An RBridge that has been configured with one or more nickname values will has priority for those nickname values over all RBridges with non-configured nicknames.
  * The nickname value 0x0000 and the value from 0xFFC0 through 0xFFFF are reserved and must not be selected by or configued for an RBridge. The value 0x0000 is used to indicate that a nickname is not known.
  * The priority of use field reported with a nickname is an unsgined 8-bit value, where was significant bit 0x80 indicate that the nickname value was configured. The bottom 7 bits have the default value 0x40, but may be configured to be some other value. Additionally, an RBridge may increase its priority after holding a nickname for some amount of time. However, the most significant bit of the priority must not be set unless the nickname value was configured.
  * Once an RBridge has successfully acquired a nickname, it should attempt to reuse it in the case of a reboot
  * Each RBridge is responsible for ensuring that its nickname or each of its nicknames is unique. If RB1 chooses nickname x, and RB1 discovers, through receipt of an LSP for RB2 at any later time, that RB2 has chosen x, then the RBridge or pseudonode with the numerically higher IS-IS ID (LAN ID) keeps the nickname, or if there is a tie in priority, the RBridge with the numerically higher IS-IS System ID keeps the nickname, and ther other RBridge must select a new nickname. This can require an RBridge with a configured nickname to select a replacement nickname.
  * To minimize the probability of nickname collisions, an RBridge selects a nickname randomly from the apparently available nicknames, based on its copy of the link state. This random selection can be by the RBridge hashing some of its parameters, e.g., SystemID, time and date, and other entropy sources, such as those given in [RFC4086], each time or by the RBridge using such hashing to create a seed and making any selections based on pseudo-random numbers generated from that seed [RFC4086]. The random numbers or seed and the algorithm used should make uniformly distributed selections over the available nicknames. Convergence to a nickname-collision-free is accelerated by selecting new nicknames only from those that appear to be available and by having the highest priority nickname involved in a nickname conflict retain its value. There is no reason for all RBridges to use the same algorithm for selecting nickname.
  * If two RBridge campus merge, then transient nickname collisions are possible. As soon as each RBridge receives the LSPs from the other RBridges, the RBridges that need to change nickname select new nicknames that do not, to the best of their knwoledge, collide with any existing nicknames. Some RBridges may need to change nicknames more than once before the situation is resolved.
  * To minimize the probability of a new RBridge usurping a nickname already in use, an RBridge should wait to acquire the libk state database from a neighbor before it announces any nicknames that were not configured
  * Any RBridge by default has only a single nickname but may be configured to request multiple nicknames. Each sub nicknames would specify a shortest path tree with the RBridge as root but, since the tree number is used in tiebreaking when there are multiple equal cost paths, the trees for the different nicknames will likely utilize different links. Because of the potential tree computation load it impose, this capability to request multiple nikcnames for an RBridge should be used spartingly. For example, it should be used at a few RBridge that, because of campus technology, are particularly good places from which to calculate multiple different shortest path distribution trees. Such trees need separate nicknames to traffic can be multipathed across them.
  * If it is desired for a pseudonode to be a tree root, the DRB may request one or more nickname in the pseudonode LSP.

  Every nickname in use in a campus identifies an RBridge (or pseudonode) and  every nickname designates a distribution tree rooted at the RBridge (or pseudonode) it identifies. However, only a limited number of these potential distribution trees are actually computed by all the RBridge in a compus.

### 3.8 TRILL Header Options
  All RBridges must be able to skip the number of 4-octet chunks indicated by the Op-Length field in order to find the inner frame, since RBridge must be able to find the destination MAC address and VLAN tag in the inner frame. (Transit RBridges need such information to filter VLANs, IP multicast, and the like. Egress RBridge need to find the inner header to correctly decapsulate and handle the inner frame.)

  To ensure backward-compatible safe operation, when Op-Length is non-zero indicating that option are present, the top two bits of the first octet of the options area are specified as follows:

            +------+------+----+----+----+----+----+----+
            | CHbH | CItE |          Reserved           |
            +------+------+----+----+----+----+----+----+

  If the CHbH (Critical Hop-by-Hop) bit is one, one or more critical hop-by-hop options are present. Transit RBridge that do not support all of the critical hop-by-hop option present, for example, an RBridge that supported no options must drop the frame. If the CHbH bit is zero, the frame is safe, from the point of view of options processing, for a transit RBridge to forward, regardless of what options that RBridge does or does nor support. A transit RBridge that supports none of the options present must transparently forward the options area when it forward a frame.

  It the CItE (Critical Ingress-to-Egress) bit is one, one or more critical ingress-or-egress options are present. If it is zero, no such options are present. If either CHbH or CItE is non-zero, egress RBridge that dont support all critical options present, for example, an RBridge that supports no options, must drop the frame. If both CHbH and CItE are zero, the frame is safe, from the point of view of options, for any egress RBridge to process, regardless of what options that RBridge does or does not support.

  Options, including the meaning of the bits labeled as Reserved in Figure 6, will be further sepcified in other documents and expected to include provisions for hop-by-hop and ingress-to-egress options as well as critical and non-critical options.

  **Note:** Most RBridge implementations are expected to be optimized for the simplest and most common cases of frame forwarding and processing. The inclusion of options may, and the inclusion of complex or lenghy options likely will, cause frame processing using a "slow path" with inferior performance to "fast path" processing. Limited slow path throughput may cause such frame to be discarded.


## 4. Other RBridge Design Details
  This section specifies other RBridge design details.

### 4.1. Ethernet Data Encapsulation
  TRILL data and ESADI frames in transit on Ethernet links are encapsulated with an outer Ethernet header. This outer header looks, to a bridge on the path between two RBridges, like the header of a regular Ethernet frame; therefore, bridges forward the frame as they normally would. To enable RBridge to distinguish such TRILL Data frames, **a new TRILL Ethertype** is used in the outer header.

  Figure 7 details a TRILL Data frame with an outer VLAN tag traveling on an Ethernet link as shown at the top of the figure, that is, between transit RBridges RB3 and RB4. The native frame originate at end station ESa, was encapsulated by ingress RBridge RB1, and will ultimately be decapsulated by egress RBridge RB2 and delivered to destination end station ESb. The encapsulation shown has the advantage, if TRILL options are absent or the length of such options is a multiple of 64 bits, of aligning the original Ethernet frame at a 64-bit boundary.

  When a TRILL Data frame is carried over an Ethernet cloud, it has three pairs of addresses:
  * Outer Ethernet Header: Outer Destination MAC address (Outer.MacDA) and Outer Source MAC Address (Outer.MacSA), these addresses are used to specify the next hop RBridge and the transmitting RBridge respectively.
  * TRILL Header: Egress Nickname and Ingress Nickname. These specify nicknames of the egress and ingress RBridges, respectively, unless the frame is multi-destination, in which case the Egress Nickname specifies the distribution tree on which the frame is being sent.
  * Inner Ethernet Header: Inner Destination MAC Address (Inner.MacDA) and Inner Source MAC Address (Inner.MacSA), these addresses are as transmitted by the original end station, specifying, respectively the destination and source of the inner frame.

  A TRILL Data frame also potentially has two VLAN tags, that can carry two different VLAN Identifiers and specify priority.

  Flow:

            +-----+  +-------+   +-------+       +-------+   +-------+  +----+
            | ESa +--+  RB1  +---+  RB3  +-------+  RB4  +---+  RB2  +--+ESb |
            +-----+  |ingress|   |transit|   ^   |transit|   |egress |  +----+
                     +-------+   +-------+   |   +-------+   +-------+

  Outer Ethernet Header:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           Outer Destination MAC Address (RB4)                 |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | Outer Destination MAC Address | Outer Source MAC Address      |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           Outer Source MAC Address (RB3)                      |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |Ethertype = C-Tag [802.1Q-2005]| Outer.VLAN Tag Information    |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  TRILL Header:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | Ethertype = TRILL             | V | R |M|Op-Length| Hop Count |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | Egress (RB2) Nickname         | Ingress (RB1) Nickname        |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Inner Ethernet Header:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |         Inner Destination MAC Address (ESb)                   |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | Inner Destination MAC Address | Inner Source MAC Address      |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |         Inner Source MAC Address (ESa)                        |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |Ethertype = C-Tag [802.1Q-2005]| Inner.VLAN Tag Information    |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Payload:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | Ethertype of Original Payload |                               |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
            |                   Original Ethernet Payload                   |
            |                                                               |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Frame Check Sequence:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           New FCS (Frame Check Sequence)                      |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

#### 4.1.1. VLAN Tag Information
  A "VLAN Tag" (formerly known as a Q-tag), also known as a "C-tag" for customer tag, includes a VLAN ID and a priority field as shown in Figure 8. The "VLAN ID" may be zero, indicating that no VLAN is specified, just a priority, although such frames are called "priority tagged" rather than "VLAN tagged" [802.1Q-2005].

  Use of [802.1ad] S-tag, also known as service tags, and use of stacked tagsm are beyond the scope of this document.

            +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
            | Priority  | C |                   VLAN ID                     |
            +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

  As recommended in 802.1Q-2005, RBridge should be implemented so as to allow use of the full range of VLAN IDs from 0x001 through 0xFFE. RBridges may support a smaller number of simultaneously active VLAN IDs. VLAN ID zero is the null VLAN identifier and indicates that no VLAN is specified while VLAN ID 0xFFF is reserved.

  **The VLAN ID 0xFFF must not be used.** RBridge must discard any frame they received with an Outer.VLAN ID of 0xFFF. RBridge must discard any frame for which they examine the Inner.VLAN ID and find it to be 0xFFF; such examination is required at all egress RBridges that decapsulate a frame.

  The "C" bit shown in Figure 8 is **not used in the Inner.VLAN in TRILL**. It must be set to zero there by ingress RBridges, transparently forwarded by transit RBridges, and is ignored by egress RBridge.

  As specified in [802.1Q-2005], the priority field contains an unsigned value from 0 through 7 where 1 indicates the lowest priority, 7 the highest priority, and the default priority zero is considered to be higher than priority 1 but lower than priority 2. The [802.1ad] amendment to [802.1Q-2005] permits mapping some adjacent pairs of priority levels into a single priority level with and without drop eligibility. Ongoing work in IEEE 802.1 (802.1az, Appendix E) suggests the ability to configure "Priority groups" that hav a certain guaranteed bandwidth. RBridge ports may also implement such options. RBridge are not required to implement any particular number of distinct priority levels but may treat one or more adjacent priority levels in the same fashion.

  Frames with the same source address, destination address, VLAN, and priority that are received on the same port as each other and are transmitted on the same port must be transmitted in the order received unless the RBridge classifies the frames into more find-gained flows, in which case this ordering requirement applies to each such flow. Frames in the same VLAN with the same priority and received on the same port may be sent out different ports if multipathing is in effect.

#### 4.1.2. Inner VLAN Tag
  The "Inner VLAN tag Information" (Inner.VLAN) field contains the VLAN tag information associated with the native frame when it was ingressed or the VLAN tag information associated with a TRILL ESADI frame when that frame was created. When a TRILL frame passes through a transit RBridge, the Inner.VLAN must not be changed except when VLAN mapping is being intentionally performed within that RBridge.

  **When a native frame arrives at an RBridge, the associated VLAN ID and priority are determined as specified in [802.1Q-2005]**. If the RBridge is an appointed forwarder for that VLAN and the delivery of the frame requires transmission to one or more other links, this ingress RBridge froms a TRILL Data frame with the associated VLAN ID and priority placed in the Inner.VLAN information.

  The VLAN ID is required at the ingress RBridge as one element in determining the approriate egress RBridge for a known unicast frame and is needed at the ingress and every transit RBridge for multi-destination frames to correctly prune the distribution tree.

#### 4.1.3. Outer VLAN Tag
  TRILL frames sent by an RBridge, except for some TRILL-Hello frames, use an Outer.VLAN ID specified by the Designate RBridge (DRB) for the link onto which they are being sent, referred to as the Designate VLAN. For TRILL data and ESADI frames, the priority in the Outer.VLAN tag should be set to the priority in the Inner.VLAN tag.

  TRILL frames forwarded by a transit RBridge use the priority present in the Inner.VLAN of the frame as received. TRILL Data farmes are sent with the priority associated with the corresponding native frame when received. **TRILL IS-IS frames should be sent with priority 7.**

  Whether an Outer.VLAN tag actually appears on the wire when a TRILL frame is sent depends on the configuration of the RBridge port through which it is sent in the same way as the appearance of a VLAN tag on a frame sent by an [802.1Q-2005] bridge depends on the configuration of the bridge port.

#### 4.1.4. Frame Check Sequence (FCS)
  Each Ethernet frame has a single Frame Check Sequence (FCS) that is computed to cover that entire frame, for detecting frame corruption due to bit errors on a link. Thus, when a frame is encapsulated, the original FCS is not included but discarded. Any received frame for which the FCS check fails should be discarded (this may not be possible in the case of cur through forwarding). The FCS normally changes on encapsulation, decapsulation, and every TRILL hop due to changes in the outer destiantion and source addresses, the decrementing of the hop count, etc.

  Although the FCS is normally calculated just before transmission, it is desirable, when pratical, for an FCS to accompany a frame within an RBridge after receipt. That FCS could then be dynamically updated to account for changes to the frame during RBridge processing and used for transmission or checked against the FCS calculated for frame transmission. This optional, more continuous use of an FCS would be helpful in detecting some internal RBridge failures such as memory errors.

### 4.2. Link State Protocol (IS-IS)
  TRILL uses an extension of IS-IS [ISO10589] \[RFC1195\] as its routing protocol. IS-IS has folowing advantages:
  * It runs directly over Layer 2, so therefore it may be run without configuration (no IP addresses need to be assigned).
  * It is easy to extend by defining new TLV (type-length-value) data elements and sub-elements for carrying TRILL information
  
  This section describes TRILL use of IS-IS, except for the TRILL-Hello protocol, which is described below, and the MTU-probe and MTU-ack messages.

#### 4.2.1. IS-IS RBridge Identity
  Each RBridge has a unique 48-bit (6-octet) IS-IS System ID. This ID may be derived from any of the RBridge's unique MAC addresses.
  
  A pseudonode is assigned a 7-octet ID by the DRB that create it, by taking a 6-octet ID owned by the DRB, and appending another octet. The 6-octet ID used to form a pseudonode ID should be the DRB's ID unless the DRB has to create IDs for pseudonodes for more than 255 links. The only constraint for correct operation is that the 7 octet ID be unique within the campus, and that the 7th octet be nonzero. An RBridge has a 7-octet ID consisting of its 6-octet system ID concatenated with a zero octet.

  In this document, we use the term "IS-IS ID" to refer to the 7-octet quantity that can be either the ID of an RBridge or a pseudonode.

#### 4.2.2. IS-IS Instances
  TRILL implements a separate IS-IS instance from any used by Layer 3, that is, different from the one used by routers. Layer 3 IS-IS frames must be distinguished from TRILL IS-IS frames even when those Layer 3 IS-IS frames are transiting an RBridge campus.

  Layer 3 IS-IS native frames have special multicast destination addresses specified for that purpose, such as AllL1ISs or AllL2ISs. When they are TRILL encapsulated, these multicast addresses appear as the Inner.MacDA and the Outer.MacDA will be the All-RBridges multicast address.

  Within TRILL, there is an IS-IS instance across all RBridges in the campus. This instance uses TRILL IS-IS frames that are distinguished by having a different Ethertype "L2-IS-IS". Additionally, for TRILL IS-IS frames that are multicast, there is a distinct multicast destination address of All-IS-IS-RBridge. TRILL IS-IS frames do not have a TRILL header.

  ESADI is a separate protocol from the IS-IS instance implemented by all the RBridge. There is a separate ESADI instance for each VLAN, and ESADI frames are encapsulated just like TRILL Data frame. After the TRILL header, the ESADI frame has an inner Ethernet header with the Inner.MacDA of "All-ESADI-RBridges" and the "LS-IS-IS" Ethertype followed by the ESADI frame.

#### 4.2.3. TRILL IS-IS Frames
  All RBridges must participate in the TRILL IS-IS instance, which constitutes a single Level 1 IS-IS area using the fixed area address zero. TRILL IS-IS frames are never forwarded by an RBridge but are locally prcessed on receipt. (Such processing may cause the RBridge to send additional TRILL IS-IS frame.)

  A TRILL IS-IS frame on an 802.3 link is structured as show below. All such frames are Ehthertype encoded. The RBridge port out of which such a frame is sent will strip the outer VLAN tag if configured to do so.

  Outer Ehthernet Header:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           All-IS-IS-RBridges Multicast Address                |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | All-IS-IS-RBridges continued  |  Source RBridge MAC Address   |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           Source RBridge MAC Address continued                |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |Ethertype = C-Tag [802.1Q-2005]|  Outer.VLAN Tag Information   |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | L2-IS-IS Ethertype            |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  IS-IS Payload:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | IS-IS Common Header, IS-IS PDU Specific Fields, IS-IS TLVs    |

  Frame Check Sequence:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           FCS (Frame Check Sequence)                          |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  The VLAN specified in the Outer.VLAN informarion will be the Designated VLAN for the link on which the frame is sent, except in the case of some TRILL Hellos.

#### 4.2.4. TRILL Link Hellos, DRBs, and Appointed Forwarders
  RBridges default to using TRILL Hellos unless, on a per-port basis, they are configured to use P2P Hellos. TRILL-Hello frames are specified in 4.4.

  RBridges are normally configured to use P2P Hellos only when there are exactly two of them on a link. However, it can occur that RBridge are misconfigured as to which type of hello to use. This is safe but may cause lack of RBridge-to-RBridge connectivity. An RBridge port configured to use P2P Hellos ignores TRILL Hellos, and an RBridge port configured to use TRILL Hellos ignores P2P Hellos.

  If any of the RBridge ports on a link is configured to use TRILL Hellos, one of such RBridge orts using TRILL Hellos is **elected DRB (Desinated RBridge)** for the link. This election is based on configured priority (most significant field), and source MAC address as communicated by TRILL-Hello frames. The DRB, designates the VLAN to be used on the link for inter-RBridge communication by the non-P2P RBridge ports and appoints itself or other RBridges on the link appointed forwarder for VLANs on the link.

##### 4.2.4.1. P2P Hello Links
  RBridges ports can be configured to use IS-IS P2P Hellos. This implies that the port is a point-to-point link to another RBridge. An RBridge must not provide any end-station (native frame) service on a port configured to use P2P Hellos.

  As with Layer 3 IS-IS. such P2P ports do not participate in a DRB election. They send all frames VLAN tagged as being in the Deisred Designated VLAN configured for the port, although this tag may be stripped if the port is so configured. Since all traffic through the port should be TRILL frames or Layer 2 control frames, such a port cannot be an appointed forwarder. RBridge P2P ports must use **the IS-IS three-way handshake [RFC5303]** so that extended circuit IDs are associated with the link for tie breaking purposes.

  Even if all simple links in a network are physically point-to-point, if some of the nodes are bridges, the bridged LANs that include those bridges appear to be multi-access links to attached RBridges. This would necessitate using TRILL Hellos for proper operation in many cases.

  While it is safe to erroneously configure ports as P2P, this may result in lack of connectivity.

##### 4.2.4.2. Designated RBridge
  TRILL IS-IS elects one RBridge for each LAN link to be the Designated RBridge (DRB), that is, to have special duties. The Designated RBridge:
  * Choose, for the link, and announces in its TRILL Hellos, the Designated VLAN ID to be used for inter-RBridge communication. This VLAN ID to be used for inter-RBridge communication. This VLAN is used for all TRILL-encapsulated data and ESADI frames and TRILL IS-IS frames except some TRILL-Hello frames.
  * If the link is represented in the IS-IS topology as a pseudonode, chooses a pseudonode ID and announces that in its TRILL Hellos and issues an LSP on behalf of the pseudonode.
  * Issues CSNPs.
  * For each VLAN-x apearing on the link, chooses an RBridge on the link to be the appointed VLAN-x forwrder (the DRB may choose itself to be the appointed VLAN-x forwarder for all or some of the VLANs).
  * Before appointing a VLAN-x forwarder (including appointing itself), wait as least its holding time (to ensure it is the DRB).
  * If configuted to send TRILL-Hello frames, continues to send them on all its enabled VLANs that have been configured in the Announcing VLANs set of the DRB, which defaults to all enabled VLANs.

##### 4.2.4.3. Appointed VLAN-x Forwardre
  The appointed VLAN-x forwarder for a link is responsible for the following points. In connection with the loop avoidance points, when an appointed forwarder for a port is "inhibited", it drops any native frames it receives and does not transmit but instead drops any native frames it decapsulates, in the VLAN for which it is appointed.
  * Loop avoidance
    * Inhibiting itself for a time, configurable per port from zero to 30 seconds, which defaults to 30 seconds, after it sees a root bridge change on the link
    * Inhibiting itself for VLAN-x, if it has received a Hello in which the sender asserts that it is appointed forwarder and that is either
      * received on VLAN-x (has VLAN-x as its Outer.VLAN) or
      * was originally sent on VLAN-x as indicated inside the body of the Hello
    * Optionally, not decapsulating a frame from ingress RBridge RBm, unless it has RBm's LSP, and the root bridge on the link it is about to forward onto is not listed in RBm's list of root bridges for VLAN-x. This is known as the "decapsulation check" or "root bridge collision check".
  * Unless inhibited, receiving VLAN-x native traffic from the link and forwarding it as appropriate.
  * Recieving VLAN-x traffic for the link and, unless inhibited, transmitting it in native from after decapsulating it as appropriate.
  * Learning the MAC address of local VLAN-x nodes by looking at the source address of VLAN-x frames from the link.
  * Optionally learning the port of local VLAN-x nodes based on any sort of Layer 2 registration protocols, such as IEEE 802.11 association and authentication.
  * Keeping track of the {egress RBridge, VLAN, MAC address} of distant VLAN-x end nodes, learned by looking at the fields {ingress RBridge, Inner.VLAN ID, Inner.MacSA} from VLAN-x frames being received for decapsulation onto the link.
  * Optionally observe native IGMP [RFC3376], MLD [RFC2710], and MRD [RFC4286] frames to learn the present of local multicast listeners and multicast routers,
  * Optionally listening to TRILL ESADI messages for VLAN-x to learn {egress RBridge, VLAN-x, MAC address} triplets and the confidence level of such expliityly advertised end nodes.
  * Optionally advertising VLAN-x end nodes, on link for which it is appointed VLAN-x forwarderm in ESADI messages.
  * Sending TRILL-Hello frames on VLAN-x unless the Announcing VLANs set for the port has been configured to disable them.
  * Listensing to BPDUs on a common spanning tree tree to learn the root bridge, if any, for that link and to report in its LSP the complete set of root bridges seen on any of its links for which it is appointed forwarder for VLAN-x.

  When an appointed forwarder observer that the DRB on a link has changed, it no longer considers itself appointed for that link until appointed by the new DRB.

##### 4.2.4.4. TRILL LSP Information
  The information items in the TRILL IS-IS LSP that are mentioned elsewhere in this document are list below. Unless an item is stated in the list below to be optional, it must be included. Other items may be included unless thier inclusion is prohibited elsewhere in this document. The catual encoding of this information and the IS-IS Type of sub-Type values for any news IS-IS TLV or sub-TLV data elements are specified in separate documents [RFC6165] \[RFC6326\].
  1. The IS-IS IDs of neighbors (pseudonodes as well as RBridges) of RBridge RBn, and the cost of the link to each of those neighbors. RBridges must use the Extended IS Reachability TLV (#22, also known as "wide metric" [RFC5305]) and must not use the IS Reachability TLV (#2, also known as "narrow metric"). To facilitate efficient operation without configuration and consitent with [802.1D], RBridge should, by defauly, set the cost of a link to the integer part of twenty trillion dividid by the RBridge port's bit rate but not more than 2\*\*24-2; for example, the cost of for a link accessed by a 1Gbps port would default to 20,000. (Note that 2\*\*24-1 has a special meanning in IS-IS and would exclude the linm from SPF routes.) However, the link cose may, by default, be decrease for aggregated links and or increased to nor more than 2\*\*24-2 if the link appears to be a bridged LAN. The tested MTU for the link may be included via a sub-TLV.
  1. The following information in connection with the nickname or each of the nicknames of RBridge RBn:
      1. The nickname value (2 octets)
      1. The unsigned 8-bit priority for RBn to have that nickname
      1. The 16-bit unsigned priority of that nickname to becoming a distribution tree root
  1. The maximum TRILL Header Version supported by RBridge RBn
  1. The Following information, in addition to the per-nickname tree root priority, in connection with distribution tree determination and announment.
      1. An unsigned 16-bit number that is the number of trees all RBridges in the campus calculate if RBn has the highest priority tree root
      1. A seconf unsigned 16-bit TODO
      1. TODO
      1. TODO
      1. TODO
  1. The list of VLAN IDs of VLANs directly connected to RBn for links on which RBn is the appointed forwarder for that VLAN. TODO
  1. Optionally, the largest TRILL IS-IS frame that the RBridge can handle using the originatingLSPBufferSize TLV #14.
  1. Optionally, a list of VLAN groups where address learning is shared across that VLAN group. Each VLAN group is a list of VLAN IDs, where the first VLAN ID listed in a group, if present, id the "primary" and the others are "secondary". This is to detect misconfiguration of features outside the scope of this document. RBridge that do not support featires such as "shared VLAN learning" ignore this field.
  1. Optionally, the Authentication TLV #10.

#### 4.2.5. The TRILL ESADI Protocol
  RBridges that are the appointed VLAN-x forwarder for a link may participate in the TRILL ESADI protocol for that VLAN. But all transit RBridge must properly forward TRILL ESADI frames as if they were mutlicast TRILL Data frames. TRILL ESADI frames are structured like IS-IS frames but are always TRILL encapsulated on the wire as if they are TRILL Data frames.

  Because of this forwarding, it appears to the ESADI protocol at an RBridge that it is directly conencted by a shared virtual link to all other RBridge in the campus running ESADI for that VLAN. RBridges that do not implement the ESADI protocol or are not appointed forwarder for that VLAN do not decapsulated or locally process any TRILL ESADI frames they received for that VLAN. In other words, these frames are transparently tunneled through transit RBridge. Such transit RBridges treat them exactly as multicast TRILL Data frames and no special processing is invoked due to such forwarding.

  TRILL ESADI frames sent on an IEEE 802.3 link are strauctured as shown below. The outer VLAN tag will not be present if it was stripped by the port out of which the frame was sent.

  Outer Ethernet Header:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |               Next Hop Destination Address                    |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | Next Hop Destination Address  |   Sending RBridge MAC Address |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |               Sending RBridge Port MAC Address                |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |Ethertype = C-Tag [802.1Q-2005]| Outer.VLAN Tag Information    |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  TRILL Header:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |       Ethertype = TRILL       | V | R |M|Op-Length| Hop Count |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | Egress (Dist. Tree) Nickname  |   Ingress (Origin) Nickname   |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Inner Ethernet Header:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           All-ESADI-RBridges Multicast Address                |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            | All-ESADI-RBridges continued  |   Origin RBridge MAC Address  |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |           Origin RBridge MAC Address continued                |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |Ethertype = C-Tag [802.1Q-2005]| Inner.VLAN Tag Information    |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |       Ethertype = L2-IS-IS    |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  ESADI Payload (formmated as IS-IS):

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |   IS-IS Common Header, IS-IS PDU Specific Fields, IS-IS TLVs  |

  Frame Check Sequence:

            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |               FCS (Frame Check Sequence)                      |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  The Next Hop Destination Address or Outer.MacDA is the All-RBridges multicast address. The VLAN specified in the Outer.VLAN information will always be the Designated VLAN for the link on which the frame is sent. The V and R fields will be zero while the M field will be one. The VLAN specified in the Inner.VLAN information will be the VLAN to which the ESADI frame applies. The origin RBridge MAC Address or Inner.MacSA must be a globally unique MAC address owned by the RBridge originating the ESADI frame, for example, any of its port MAC addresses, and each RBridge must use the same Inner.MacSA for all of the ESADI frames that RBridge originates.

##### 4.2.5.1. TRILL ESADI Participation
  An RBridge does not send any Hellos because of participation in the ESADI protocol. The information available the in TRILL IS-IS link state database is sufficient to determine the ESADI DRB on the virtual link for the ESADI protcol for each VLAN. In particular, the link state database information for each RBridge includes the VLANs, if any, for which the RBridge is participating in the ESADI protocol, its priority for being selected as DRB for the ESADI protocol foreach of those VLANs, its holding time, and its IS-IS system ID for breaking ties priority.

  An RBridge need not perform any routing calculating because of participation in the ESADI protocol. Since all RBridges participating in ESADI for a particular VLAN appear to be connected to the same single virutal link, there are no routing decision to be made. A participating RBridge merely transmit the ESADI frames it originates on this virtual link.

  The ESADI DRB sends TRILL-ESADI-CSNP frames on the ESADI virtual link. For robustness, a participaing RBridge that determines that some other RBridge should be ESADI DRB on such virtual link but has not received or sent a TRILL-ESADI-CSNP in at least the ESADI DRB holding time may also send a TRILL-ESADI-CSNP on the virtual link. A participating RBridge that determines that no other RBridges are participating in the ESADI protocol for a particular VLAN should not send ESADI information or TRILL-ESADI-CSNPs on the virtual link for the VLAN.

##### 4.2.5.2. TRILL ESADI Information
  The information distributed with the ESADI protocol is the list of local end-station MAC addresses knwon to be the originating RBridge and, for each such address, a one-octet unsgined "confident" rating in the range 0-254.

  It is intended to optionally provide for VLAN ID translation within RBridges, as specified in [VLAN-MAPPING]. This includes translating TRILL ESADI frames. If TRILL ESADI frames could contain VLAN IDs in arbitrary internal locations, such translation would be impractical. Thus, TRILL ESADI frames must not contain the VLAN ID of the VLAN to which they apply in the body of the frame after the Inner.VLAN tag.

#### 4.2.6. SPF, Forwarding, and Ambiguous Destinations
  This section describes the logical result desired. Alternative implementation methods may be used as long as they produce the same forwarding behavior.

  When building a forwarding table, an RBridge RB1 calculate shortest paths from itself as desrcribed in Appendix C.1 of [RFC1195]. Nicknames are added into the shortest path calculation as a final step, just as with an end node. If multiple RBridge, say, RBa and RBb, claim the same nickname, this is a transitory condition and one of RBa or RBb will defer and choose a new nickname. However, RB1 simply adds that nickname as if it were attached to both RB1 and RBb, and uses its standard shortest path calculation to choose the next hop.

  An ingress RBridge RB2 maps a native frame's known unicast destination MAC address and VLAN into an egress RBridge nickname. If RB2 learns addresses only from the observation of received and decapsulated frames, then such MAC addresses cannot be duplicated within a VLAN in RB2 tables because more recent learned information, if of a higher or equal confidence, overwrites previous information and, if of a lower confidence, if ignored. However, duplicates of the same MAC within a VLAN can appear in ESADI data and between ESADI data and addresses learned from observation of received and decapsulated frames, entered by manual configuration, or learned through Layer 2 registration protocols. If duplicate MAC addresses occur within a VLAN, RB2 sends frames to the MAC with the highest confidence. If confidence are also tied between the duplicates, for consistency it is suggested that RB2 direct all such frames (or all such frames in the same ECMP flow) toward the same egress RBridge; however, the use of other policies will not cause a network problem since transit RBridge do not exanmine the Inner.MacDA for known unicast frames.
