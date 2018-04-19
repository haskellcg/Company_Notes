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

  While Rbridges can be applied to a variety of link protocols, this specification focuses on IEEE [802.3] links. Use with other link type is expected to be covered in other docuements.

  The TRILL protocol, as specified herein, is desinged to be a Local Area Network protocol and not designed **with the goal of scaling beyond the size of existing briged LANs**.

### 1.1. Algorhyme V2, by Ray Perlner
  I hope that we shall one day see  
  A graph more lovely than a tree.

  A graph to boost efficiency  
  While still configuration-free.

  A network where Rbridges can  
  Route packets to their target LAN.

  The paths they find, to our elation,  
  Are least cost paths to destination!

  With packet hop counts we now see,  
  The network need not be loop-free!

  Rbridges work transparently,  
  Without a common spanning tree.

### 1.2. Normative Content and Precedence
  The bulk of the normative metarial in this specification appears in 1 through 4.

### 1.3. Terminology and Notation in This Document
  "TRILL/RBridge/Rbridge"

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
  * TRILL other frames are those with any of the 16 multicast destination addresses reserved for TRILL other than All-RBridges and All-IS-IS-Rbridges. RBridges conformant to this specification must discard TRILL other frames.

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
  Because Rbridge are compatible with IEEE [802.1Q-2005] customer bridges, except as discussed in this document, a bridge LAN can be upgraded by incrementally replcing such bridges with RBridge. Bridges that have not yet been replaced are transparent to RBridge traffic. The physical links directly interconnected by such bridges, together with the bridges themselves, constitute bridged LANs. These bridged LANs apear to RBridges to be multi-access links.

  If the bridges replaced by RBridges were default configuration bridges, then theri RBridge replacements will not require configuration.

  Because RBridges, as described in this document, only provide customer services, they cannot replace provider bridges or provider backbone bridges, just as a customer bridge can't replace a provider bridge. However, **such provider devices can be part of the bridged LAN between RBridges**. Extension of TRILL to support provider services is left for future work and will be separately documented.

  Of course, if the bridges replaced had any port level protocols enabled, such as port-based access control [802.1X] or MAC security [802.1AE], replacement RBridge would **need the same port level protocols enabled and similarly configured**. In addition, the replacement RBridges would have to support the same link type and link level protocols as the replaced bridges.

  An RBridge campus will work best if all IEEE [802.1D] and [802.1Q-2005] bridges are replaced with RBridges, assuming the Rbridges have the same speed and capacity as the bridges. However, there may be intermediate states, where only some bridges have been replaced by RBridges, with inferior performence.


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
  * M = 0 (False) - The egress Rbridge nickname contains a nickname of the egress RBridge for a knwon unicast MAC address
  * M = 1 (True) - The egress RBridge nickname field contains a nickname that specifies a distribution tree. This nickname is selected by the ingress RBridge for a TRILL Data frame or by the source RBridge for a TRILL ESADI frame.

### 3.5. Op-Length
### 3.6. Hop Count
### 3.7. RBridge Nicknames
#### 3.7.1. Egress RBridge Nickname
#### 3.7.2. Ingress RBridge Nickname
#### 3.7.3. RBridge Nickname Selection
### 3.8 TRILL Header Options

## 4. Other RBridge Design Details
### 4.1. Ethernet Data Encapsulation
#### 4.1.1. VLAN Tag Information
#### 4.1.2. Inner VLAN Tag
#### 4.1.3. Iuter VLAN Tag
#### 4.1.4. Frame Check Sequence (FCS)
### 4.2. Link State Protocol (IS-IS)
#### 4.2.1. IS-IS RBridge Identity
#### 4.2.2. IS-IS Instances
#### 4.2.3. TRILL IS-IS Frames
#### 4.2.4. TRILL Link Hellos, DRBs, and Appointed Forwarders
##### 4.2.4.1. P2P Hello Links
##### 4.2.4.2. Designated RBridge
##### 4.2.4.3. Appointed VLAN-x Forwardre
##### 4.2.4.4. TRILL LSP Information
#### 4.2.5. The TRILL ESADI Protocol
##### 4.2.5.1. TRILL ESADI Participation
##### 4.2.5.2. TRILL ESADI Information
#### 4.2.6. SPF, Forwarding, and Ambiguous Destinations
### 4.3. Inter-RBridge Link MTU Size
#### 4.3.1. Determining Campus-Wide TRILL IS-IS MTU Size
#### 4.3.2. Testing Link MTU Size
### 4.4. TRILL-Hello Protocol
#### 4.4.1. TRILL-Hello Rationale
#### 4.4.2. TRILL-Hello Contents and Timing
##### 4.4.2.1. TRILL Neighbor List
#### 4.4.3. TRILL MTU-Probe and TRILL Hello VLAN Tagging
#### 4.4.4. Multiple Ports on the Same Link
#### 4.4.5. VLAN Mapping within a Link
### 4.5. Distribution Trees
#### 4.5.1. Distribution Tree Calculation
#### 4.5.2. Multi-Destination Frame Checks
#### 4.5.3. Pruning the Distribution Tree
#### 4.5.4. Tree Distribution Optimization
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

## 5. RBridge Parameters
### 5.1. Per RBridge
### 5.2. Per Nickname Per RBridge
### 5.3. Per Port Per RBridge
### 5.4. Per VLAN Per RBridge

## 6. Security Considerations
### 6.1. VLAN Security Considerations
### 6.2. BPDU/Hello Denial-of-Service Considerations

## 7. Assignment Considerations
### 7.1. IANA Considerations
### 7.2. IEEE Registration Authority Considerations

## 8. Normative References

## 9. Informative References

## Appendix A. Incremental Deployment Considerations
### A.1. Link Cost Determination
### A.2. Appointed Forwarders and Bridged VLANs
### A.3. Wiring Closet Topology
#### A.3.1. The RBridge Solution
#### A.3.2. The VLAN Solution
#### A.3.3. The Spanning Tree Solution
#### A.3.4. Comparison of Solutions

## Appendix B. Trunk and Access Port Configuration

## Appendix C. Multipathing

## Appendix D. Determination of VLAN and Priority

## Appendix E. Support of IEEE 802.1Q-2005 Amendments
### E.1. Complete Amendments
### E.2. In-Process Amendments

## Appendix F. Acknowledgements
