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
### 2.3. RBridge Encapsulation Architecture
### 2.4. Forwarding Overview
#### 2.4.1. Known-Unicast
#### 2.4.2. Multi-Destination
### 2.5. RBridges and LANs
#### 2.5.1. Link VLAN Assumptions
### 2.6. RBridges and IEEE 802.1 Bridges
#### 2.6.1. RBridge Ports and 802.1 Layering
#### 2.6.2. Incremental Deployment

## 3. Details of the TRILL Header
### 3.1. TRILL Header Formart
### 3.2. Version (V)
### 3.3. Reserved (R)
### 3.4. Multi-Destination (M)
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
