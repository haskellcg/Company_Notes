# Bidirectional Forwarding Detection (BFD) for MPLS Label Switched Paths (LSPs)
  One desirable applicaton of Bidirectional Forwarding Detection (BFD) is to detect a Multiprotovcol Label Switching (MPLS) Label Switched Path (LSP) data plane failure.
  
  **_LSP Ping is existing mechanism for detecting MPLS data plane failures and for verifying the MPLS LSP data plane against the control plane_**. BFD can be used for the former, but not for the latter. 
  
  However, the control plane processing required for BFD Control packets is relatively smaller than the processing required for LSP Ping messages. A combination of LSP ping and BFD can be used to provide faster data plane failure detecttion and/or make it possible to provide such detecttion on a greater number of LSPs.
  
## Introduction
  In particular, BFD can be used to detect a data plane faailure in the forwarding path of an MPLS LSP. 
  
## Applicability
  In the event of an MPLS LSP failing to deliver data traffic, it may not always be possible to detect the failure using the MPLS control plane.
  
  For instance, the control plane of the MPLS LSP may be functional while the data plane may be mis-forwarding or dropping data.
    
  Hence, there is a need for a mechanism to detect a data plane failure in the MPLS LSP path.
  
### BFD for MPLS LSPs: Motivation
  LSP Ping described in RFC4379 is an existing mechanism for detecting an MPLS LSP data plane failure. In addition, LSP Ping also provides a mechanism for verifying the MPLS control plane agianst the data plane. This is done by ensuring that the LSP is mapped to the same Forwarding Equivalence Class (FEC), at the egree, as the ingree.
  
  The LSP may be associated with any of the following FECs:
  * Resource Reservation Protocol (RSVP) LSP_Tunnel IPv4/IPv6 Session [RFC3209]
  * Label Distribution Protocol (LDP) IPv4/IPv6 prefix [RFC5036]
  * Virtual Private Network (VPN) IPv4/IPv6 prefix [RFC4364]
  * Layer 2 VPN [L2-VPN]
  * Pseudowise based on PWid FEC and Generalized PWid FEC [RFC4447]
  * Border Gateway Protocol (BGP) labeled prefixes [RFC3107]
  
  LSP Ping includes extensive control plane verification. BFD, on the other hand, was designed as a lightweight means of testing only the data plane.
  
  As result, LSP Ping is computational more expensive than BFD for detecting MPLS LSP data plane faults. BFD is also more suitable for being implemented in hardware or firmware due to its fixed packet format. Thus, the use of BFD for detecting MPLS LSP data plane faults has the following advantages:
  * Support for fault detecttion for great number of LSPs
  * Fast detection. Detection with sub-second granularty is considered as fast detection. LSP Ping is intended to be used in an environment where fault detection messages are exchanged, either for diagnostic purpose or for infrequent periodic fault detection, in the order of tens of seconds or minutes. Hence, it is not propriate for fast detection. BFD, on the other hand, is designed for sub-second fault detection intervals:
    * In the case of a bypass LSP based for a facility-based link or node protection
    * MPLS Pseudowises (PWs)
    
### Using BFD in Conjunction with LSP Ping   
  LSP Ping performs the following functions that are outside the scope of BFD:
  * Association of an LSP Ping Echo request message with a FEC.
  * Euqla Cost Multi-Path (ECMP) considerations.
  * Traceroute. LSP Ping supports traceroute for a FEC and it can be used for fault isolation
  
  Hence, BFD is used in conjunction with LSP Ping for MPLS LSP fault detection:
  * LSP Ping is used for bootstraping the BFD session as described later in this document
  * BFD is ued to exchange fault detection packets at the required detection interval
  * LSP Ping is used to periodically verify the control plane against the data plane by ensuring that the LSP is maped to the same FEC, at the egree, as the ingree
  
## Theory of Operation
  BFE Control packets must be sent along the same data path as the LSP being verified and are processed by the BFD processing module of the egress LSR. If the LSP is associated with multiple FECs, a BFD session should be established for each FEC.
  
  For instance, this may happen in the case of next-hop label allocation. Hence, the operation is conceptually similar to the data plane fault detection procedures of LSP Ping.
  
  If MPLS fast-reroute is being used for the MPLS LSP, the use of BFD for fault detection can result in false fault detections if the BFD fault detection interval is less than the MPLS fast-reroute switchover time. When MPLS fate-reroute is triggered because of a link or node failure, BFD Control packets will be dropped until traffic is switched exceeds the BFD fault detection interval, a fault will be declared even though the MPLS LSP is being locally repaired. 
  
  To avoid this, the BFD fault detection interval should be greater than the fast-reroute switchover time. An implementation should provide configuration options to control the BFD fault detection interval.
  
  If there are multiple alternate paths from an ingress LSR to an egress LSR for an LDP IP FEC, LSP Ping traceroute may be used to determine each of these alternate paths. A BFD session should be established for each alternate path that is discovered.
  
  Periodic LSP Ping Echo request messages should be sent by the ingress LSR to egress LSR along the same data as the LSP. This is to periodically verify the control plane against the data by ensuring that the LSP is mapped to the same FEC, at the egress, as the ingress. The rate of generation of these LSP Ping Echo request messages should be significantly less than the rate of the generation of BFD Control packets. **_An implementation may provide configuration options to control the rate of generation of the periodic LSP Ping Echo request messages_**.
  
  To enable fault detection procedures psecified in this document, for particular MPLS LSP, this document requires the ingress and egress **_LSR to be configured_**. This includes configuration for supporting BFD and LSP Ping as specified in this document. It also includes **_condifuration that enables the ingress LSR to determine the method used by the egress LSR to identity Operations, Administration, and Maintenance (OAM) packets, e.g._**, whether the Time to Live (TTL) of the innermost MPLS label needs to be set to 1 to enable the egress LSP to identify the OAM packet.
  
  For fault detection for MPLS PWs, this document assume that the PW control channel type is configured and the support of LSP Ping is also configured.
  
  
  
  
