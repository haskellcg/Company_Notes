# Transparent Interconnection of Lots of Links (TRILL) Use of IS-IS
# Abstract
  TODO

# 1. Introduction
  The IETF Transparent Interconnection of Lots of Links (TRILL) prrotocol [RFC6325] provides transparent forwarding in multi-hop networks with arbitrary topology and link technologies using a header witth a hop count and link-state routing. TRILL provides optimal pair-wise forwarding without configuration, safe forwarding even during periods of temporary loops, and support for multipathing of both unicast and multicast traffic. Intermediate System (ISs) implementing TRILL are called Routing Bridges (RBridges) or TRILL Switches.

  This document, in conjunction with [RFC6165], specifics the data formats and code points for the IS-IS [RFC1195] extensions to support TRILL. These data formats and code points may also be used by technologies other than TRILL.

  This document obsoletes [RFC6326], which generally corresponded to the base TRILL protocol [RFC6325]. There has been substantial development of TRILL  since the publication of those documents. The main changes from [RFC6326] are summarized below, and a full list is given in section 7.
  1. Added multicast group announcements by IPv4 and IPv6 addres
  1. Added facilities for Announcing capapbilities supported
  1. Added a tree affinity sub-TLV whereby ISs can request distribution tree association
  1. Added multi-topology support
  1. Added control-plane support for TRILL Data frame find-gained labels. This support is independent of the data-plane representation
  1. Fixed the verified erratum [Err2869] in [RFC6326]
## 1.1 Convertions Used in This Document
  * BVL - Bit Vector Length
  * BVO - Bit Vector Offset
  * IHH - IS-IS Hello
  * IS - Intermediate System. Forr this document, all relevant intermediate systems are RBridge [RFC6325]
  * MAC - Media Access Control
  * MT - Multi-Topology
  * NLPID - Network Layer Protocol Identifier
  * SNPA - Subnetwork Point of Attachment (MAC address)

# 2. TLV and Sub-TLV Extensions to IS-IS for TRILL
## 2.1 Group Address TLV
### 2.1.1. Group MAC Address Sub-TLV
### 2.1.2. Group IPv4 Address Sub-TLV
### 2.1.3. Group IPv6 Address Sub-TLV
### 2.1.4. Group Labeled MAC Address Sub-TLV
### 2.1.5. Group Labeled IPv4 Address Sub-TLV
### 2.1.6. Group Labeled IPv6 Address Sub-TLV
## 2.2. Multi-Topology-Aware Port Capability Sub-TLVs
### 2.2.1. Special VLANs and Flags Sub-TLV
### 2.2.2. Enabled-VLANs Sub-TLV
### 2.2.3. Appointed Forwarders Sub-TLV
### 2.2.4. Port TRILL Version Sub-TLV
### 2.2.5. VLANs Appointed Sub-TLV
## 2.3. Sub-TLVs of the Router Capability and MT-Capability TLVs
### 2.3.1. TRILL Version Sub-TLV
### 2.3.2. Nickname Sub-TLV
### 2.3.3. Trees Sub-TLV
### 2.3.4. Tree Identifiers Sub-TLV
### 2.3.5. Trees Used identifiers Sub-TLV
### 2.3.6. Interested VLANs and Spanning Tree Roots Sub-TLV
### 2.3.7. VLAN Group Sub-TLV
### 2.3.8. Interested Labels and Spanning Tree Roots Sub-TLV
### 2.3.9. RBridge Channel Protocols Sub-TLV
### 2.3.10 Affinity Sub-TLV
### 2.3.11 Label Group Sub-TLV
## 2.4. MTU Sub-TLV for Extended Readability and MT-ISN TLVs
## 2.5. TRILL Neighbor TLV

# 3. MTU PDUs

# 4. Use of Existing PDUs and TLVs
## 4.1. TRILL IIH PDUs
## 4.2. Area Address
## 4.3. Protocols Supported
## 4.4. Link State PDUs (LSPs)
## 4.5. Originating LSP Buffer Size

# 5. IANA Consideration
## 5.1. TLVs
## 5.2. Sub-TLVs
## 5.3. PDUs
## 5.4. Reserved and Capability Bits
## 5.5. TRILL Neighbor Record Flags

# 6. Security Consideration

# 7. Changes from RFC 6326

# 8. References
## 8.1. Normative References
## 8.2. Informative References

# 9. Acknowledgement
