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
## 1.2. Terminology and Acronyms

# 2. The TRILL Hello Environment and Purposes
## 2.1. RBridge Interconnection by Ethernet
## 2.2. Handling Native Frames
## 2.3. Zero or Minimal Configuration
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
