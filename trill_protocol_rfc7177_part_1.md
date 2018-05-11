# Transparent Interconnection of Lots of Links (TRILL): Adjacency
# Abstract
  TODO

# 1. Introduction
  The IETF Transparent Interconnection of Lots of Links (TRILL) protocol [RFC6325] provides optimal pair-wise data frame forwarding without configuration, safe forwarding even during transient loops, and support for transmission of both unicast and multicast traffic taking advantage of multiple paths in multi-hop networks with arbitrary topology. End stations are connected to TRILL switches with Ethernet, but TRILL switches can be interconnected with arbitrary technology. TRILL accomplishes this by using [IS-IS] link state routing [RFC1195] \[RFC7176\] and a header in TRILL Data Packets that includes a hop count. The design supports data labeling by Virtual Local Area Network (VLANs) and fine-grained labels [RFC7172] as well as optimization of the distribution of multi-destination frames based on data label and multicast groups. Devices that implement TRILL are called **TRILL switches or RBridges (Routing Bridges)**.
  
  This document provides detailed specifications for five of the link local aspects of the TRILL protocol used on broadcast links (also called LAN or multi-access links) and for the three of these aspects that are also used on point-to-point (P2P) links. It includes state diagrams and implementation details where appropriate. Alternative implementation that interoperate on the wire are permitted.

  The scope of this document is limited to the following aspects of the TRILL protocol; their applicability, along with the most relevant section of this document, are as shown here:

  LAN|P2P|Section|Link-Local Aspect
  ---|---|-------|-----------------
  X|X|3|Adjacency formation and dissolution
  X||4|DRB (Designated RBridge) election
  X|X|5|MTU (Maximum Transmission Unit) matching
  X|X|6|1-hop BFD (Bidirectional Forwarding Detection) for adjacency
  X||7|Creation and use of pseudonodes [IS-IS]

  There is no DRB (also known as DIS (Designated Intermediate System)) election and no pseudonode creation on links configured as point-to-point.

  For other aspect of the TRILL base protocol, see [RFC6325] \[RFC6439\], and {ESADI]

## 1.1. Content and Precedence
  TODO

## 1.2. Terminology and Acronyms
  * BFD - Bidirectional Forwarding Detection [RFC7175]
  * SNPA - Subnetwork Point of Attachment [IS-IS]
  * TRILL switch - an alternative name for an RBridge


# 2. The TRILL Hello Environment and Purposes
  [IS-IS] has subnetwork-independent functions and subnetwork-dependent functions. Currently, Layer 3 use of IS-IS supports two types of subnetworks:
  * point-to-point link subnetworks between routers
  * general broadcast (LAN) subnetworks

  Because of the differeces between the environment of Layer 3 routers and the environment of TRILL Bridges, instead of the subnetwork-dependent functions used at Layer 3, which are specified in Section 8.2 and 8.4 of [IS-IS], the TRILL protocol uses modified subnetwork-dependent functions for point-to-point subnetworks and broadcast (LAN) subnetworks. The differences between the TRILL and Layer 3 environemnts are described through 2.1 to 2.4.

## 2.1. RBridge Interconnection by Ethernet
  TRILL supports the interconnection of RBridges by multi-access LAN links such as Ehernet. Because this includes general bridged LANs [802.1Q], the links between RBridges may contain devices or services that can restrict LAN connectivity, such as [802.1Q] bridges or carrier Ethernet services. In addition, RBridge Ethernet ports, like [802.1Q] ports, can be configured to restrict input/output on a LAN basis.

  For this reason, TRILL Data and TRILL IS-IS packets are sent on Ethernet links in a Designated VLAN that is assumed to provide connectivity between all RBridges on the link. The Designated VLAN is dictated for a LAN link by the elected Designated RBridge on that link (DRB, equivalent to the Designated Intermediate System at Layer 3). On an RBridge Ethernet port configured as point-to-point, TRILL Data and IS-IS packets are sent in that port's Desired Designated VLAN, regardless of the state of any other ports on the link. Connectivity on an Ethernet link configured as point-to-point generally depends on both ends being configued with the same Desired Designated VLAN. Because TRILL Data packets flow between RBridges on an Ethernet link only in the link's Designated VLAN, adjacency for routing calculation is based only on connectivity characteristics in the VLAN.

  (Non-Ethernet links, such as PPP [RFC6361] generally do not have any Outer.VLAN labeling, so the Designated VLAN for such links has no effect)

## 2.2. Handling Native Frames
  This section discusses the handling of "native" frames as defined in Section 1.4 of [RFC6325]. As such, this section is not applicable to point-to-point links between TRILL switches or any link where all the TRILL sswitch ports on the link have been configured as "trunk ports" by setting the end-station service disable bit for the port.

  Layer 3 data packets, such as IP packets, are already "tamed" when they are originated by the end station: they include a hop count and Layer 3 source and destination address fields. Furthermore, for ordinary data packets, there is no requirement to preserve any outer Layer 2 addressing, and if the packets are unicast, they are explicitly addressed to their first-hop router.

  In contrast, TRILL switches must accept, transport, and deliver "untamed" native frames: native frames that lack of hop count field usable by TRILL and have Layer 2 MAC (Media Access Control) addresses that indicate their source and destination. These Layer 2 addresses must be preserved for delivery to the native frame's Layer 2 destination. One resulting difference is that RBridge ports providing native frame service must receive in promiscuous MAC address mode, while Layer 3 router ports typically receive in a selective MAC address mode.

  TRILL handles these requirements by having, on the link where an end station originates a native frame, one RBridge "ingress" such a locally originated native frame by adding a TRILL Header that includes a hop count, thus converting it to a TRILL Data packet. This augmented packet is then routed to one RBridge on the link having the destination end station for the frame (or one Rbridge on each such link if it is a multi-destination frame). Such final Rbridge perform an "egress" function, removing the TRILL Header and delivering the original frame to its destinations. (For the purpose of TRILL, a Layer 3 router is an end-station.)

  **Care must be taken to avoid a loop that would involve egressing a native frame and then re-ingress it** because, while it is in native form, it would not be protected by a hop count and could loop forever. Such a loop could, for multi-destination frames, even involve multiplication of the number of frames each time around and would likely sature all links involved within miliseconds. For TRILL, safety against suvh loop for a link is more important than ransient loss of data connectivity on that link.

  The primary TRILL defense mechanism against such loops, which is mandatory, is to assure that, as far as practically possible, there is only a single RBridge that is in charge of ingressing and egressing native frames from and to a link where TRILL is offering end-station service. This is a Designated RBridge and Appointed forwarder mechanism initially specified in the TRILL base potocol, discussed in section 2.5 below, and further specified in both section 4 below and [RFC6439].

## 2.3. Zero or Minimal Configuration
  TRILL provides connectivity and least-cost paths with zero configuration. For additional services, it strives to require only minimal configuration; however, services that require configuration when offered by [802.1Q] bridges, such as non-default VLANs or priority, will require configuration. This differs from Layer 3 routing where routers typically need to be configured as to the subnetworks connected to each port, etc, to provide services.

## 2.4. MRU Robustness
  TRILL IS-IS needs to be robust against links with reasonably restricted MTUs, including links that accommodate only the classic Ethernet frame size, despite the addition of reasonable headers such as VLAN tags. Such robustness is particularly required for TRILL Hellos to assure correct adjacency and the election of a unique DRB on LAN links.

  TRILL will also be used inside data centers where it is common for all or most of the links and switches to support frames substantially larger than the classic Ethernet maximum size. For example, they may have an MTU adequate to confortably handle Fiber Channel over Ethernet frames, for which T11 recommends a 2500-byte MTU [FCoE], or even 9K byte jumbo frames. It would be beneficial for a TRILL campus with such a large MTU to be able to safely make use of it.

  These needs are met by a mandatory maximum on the size of TRILL Hellos and by the optional use of MTU testing as described below.

## 2.5. Purpose of the TRILL Hello Protocol
  There are three purpose for the TRILL Hello Protocol. They are listed below, along with a reference to be the section of this document in which each is discussed:
  * To determined which RBridge neighbors have acceptable connectivity to be reported as part of the topology
  * To elect a unique Designated RBridge on broadcast (LAN) links 
  * To determine the MTU with which it is possible to safelt communicate with each RBridge neighbor

  In Layer 3 IS-IS, all three of these functions are combined. Hellos may be padded to the maximum length so that a router neighbor is not discovered if it is impossible to communicate with it using maximum-sized Layer 3 IS-IS packets. Also, even if Hellos from a neighbor R2 are received by R1, if connectivity to R2 is not 2-way (i.e., R2 does not list R1 in R2's Hello), then R1 doesnot consider R2 as a Desginated Intermediate System (Designated Router) candidate. Because of this logic, it is possible at Layer 3 for multiple Designated Routers to be elected on a LAN, with each representing the LAN as a pseudonode. It appears to the topology as if the LAN is now two or more separate LANs. Although this is surprising, this does not cause problems for Layer 3.
  
  In contrast, this behavior is not acceptable for TRILL, since in TRILL it is important that all RBridges on a link known about each other, and on broadcast (LAN) links that they choose a single RBridge to be the DRB to control the native frame ingress and egress. Otherwise, multiple RBridge might ingress/egress the same native frame, forming loops that are not protected by the hop count in the TRILL Header as discussed above.

  The TRILL Hello Protocol is best understood by focusing separately on each of these functions listed above, which we do in section 3, 4, and 5.

  One other issue with TRILL LAN Hellos is to ensure that subnets of the information can appear in any single message, and be processable, in the spirit of IS-IS Link State PDUs (LSPs) and complete Sequence Number PDUs (CSNPs). LAN TRILL Hello packets, even though they are not padded, can become very large. An example where this might be the case is **When some sort of backbone technology interconnects hundreds of TRILL sites over what would appear to TRILL to be a giant Ethernet, Where the RBridge connected to that cloud will perceive that backbone to be a single link with hundreds of neighbors**. Thus, the TRILL LAN Hello uses a different Neighbor TLV [RFC7176] that lists neighbors seen for a range of MAC (SNPA) addresses.


# 3. Adjacency State Machinery
  Each RBridge port has associate with it a port state, as discuessed in section 4, and a table of zero or more adjacencies (if the port is configured as point-to-point, zero, or one) as discussed in this section. The states such adjacencies can have, the events that cause adjacency state change, the actions associated with those state changes, a state table, and a state disgram are given below.

## 3.1. TRILL Hellos, Ports, and VLANs
  The determination of adjacencies on links is made using TRILL Hellos (see section 8), an optional MTU test (see section 5), and, optionally, BFD (see section 6) and/or other connectivity tests. If the MAC (SNPA) addresses of more than one RBridge port on a broadcast link are the same, all but one of such ports are put in the suspended state and do not partivipate in the link, execpt to monitor whether they should stay suspended. If the two ports on a point-to-point link have MAC (SNPA) addresses, it does not affect TRILL operation if they are the same. (PPP ports, for example, do not have MAC addresses [RFC6361])

  The following items must be the same for all TRILL Hellos issued by an RBridge on a particular Ethernet portm regardless of the VLAN in which the Hello is sent:
  * Source MAC address
  * Priority to be the DRB
  * Desired Designated VLAN
  * Port ID, and 
  * if included, BFD-Enabled TLV [RFC6213] and PORT-TRILL-VER sub-TLV [RFC7176]

  Of course, the priority, Desired Designated VLAN, and possible the inclusion or value of the PORT-TRILL-VER sub-TLV, and/or BFD-Enabled TLV can change on occasion, but then the new value(s) must similarly be used in all TRILL Hellos on the LAN port, regardless of VLAN.

  * On broadcast links:
    * Because bridges acting as glue on an Ethernet broadcast link might be configured in such way that some VLANs are partitioned, it is necessary for RBridges to transmit Hellos on Ethernet links with multiple VLAN tags. The conceptually simplest solution may have been to have RBridge transmit up to 4,094 times as many Hellos, one with each legal VLAN ID enabled at each port, but this would obviously have deleterious performance implications. So, the TRILL protocol specifies that if RB1 knowns it is not the DRB, **it transmit its Hellos on only a limited set of VLANs**. Only an RBridge that believes itself to be the DRB on a broadcast Ethernet link "spray" its TRILL Hellos on all of its enabled VLANs at the port. An in both cases, an RBridge can be configured to send Hellos on only s subset of those VLANs. **The details are given in [RFC6325], section 4.4.3**.
  * On Point-to-point-links:
    * If the link technology is VLAN secsitive, such as Ethernet, an RBridge sends TRILL Hellos only in the Desired Designated VLAN for which it is configured.

## 3.2. Adjacency Table Entries and States
## 3.3. Adjacency and Hello Events
## 3.4. Adjacency State Diagram and Table
## 3.5. Multiple Parallel Links
## 3.6. Insufficient Space in Adjacency Table

# 4. LAN Ports and DRB State
## 4.1. Port Table Entries and DRB Election State
## 4.2. DRB Election Events
### 4.2.1. DRB Election Details
### 4.2.2. Change in DRB
### 4.2.3. Change in Designated VLAN
## 4.3. Port State Table and Diagram

# 5. MTU Matching

# 6. BFD-Enabled TLV and BFD Session Bootstrapping

# 7. Pseudonodes

# 8. More TRILL Hello Details
## 8.1. Contents of TRILL Hellos
## 8.2. Transmitting TRILL Hellos
### 8.2.1. TRILL Neighbor TLVs
## 8.3. Receiving TRILL Hellos

# 9. Multiple Ports on the same Broadcast link

# 10. IANA Considerations

# 11. Security Considerations

# Appendix A. Changes from RFC 6327

# Appendix B. Changes from RFC 6325

# Normative References

# Informative References

# Acknonwledgements