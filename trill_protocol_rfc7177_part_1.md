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
## 2.5. Purpose of the TRILL Hello Protocol

# 3. Adjacency State Machinery
## 3.1. TRILL Hellos, Ports, and VLANs
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
