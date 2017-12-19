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
  
## Initialization and Demultiplexing  
  A BFD session may be established for a FEC associated with an MPLS LSP. As described above, in the case of PHP or when the egress LSR distributes an explicit nul label to the penultimate hop router, or next-hop label allocation, the BFD Control packet received by the egress LSR does not contain sufficient information to associate it with a BFD session.
  
  Hence, the demultiplexing must be done using the remote discriminator field in the received BFD Control packet.
  
## Sesison Establishment  
  A BFD session is bootstrapped using LSP Ping. The initiation of fault detection for a particular <MPLS LSP, FEC> combination results in the exchange of LSP Ping Echo request and Echo reply packets, in the ping mode, between the ingress and egress LSRs for that <MPLS LSP, FEC>.
  
  To establish a BFD session, an LSP Ping Echo request message must carry the local discriminator assigned by the ingress LSR for the BFD session. This must subsequently be used as the My Discriminator field in the BFD session packets sent by the ingress LSR.
  
  On the receipt of the LSP Ping Echo request message, the egress LSR must send a BFD Control packet to the ingress LSR, if the validation of the FEC in the LSP Ping Echo request message succeeds. This BFD Control packet must be set the Your Discriminator field to the discriminator received from the ingress LSR in the LSP Ping Echo request message. The egress LSR may respond with an LSP Ping Echo reply message that carries the local discriminator assigned by it for the BFD session. 
  
  The local discriminator assigned by the egress LSR must be used as the MY Discriminator field in the BFD session packets sent by the egress LSR.
  
  The ingress LSR follows the procedures in BFD to sent BFD Control packets to the egress LSR in response to the BFD Control packets received fron the egress LSR. The BFD Control packets from the ingress to the egress LSR must set the local discriminator of the egress LSR, in the Your Discriminator field.
  
  The egress LSR demultiplexes the BFD session based on the received Your Discriminator field. As mentioned above, the gress LSR must sent Control packets to the ingress LSR with the Your discriminator field set to the local discriminator of the ingress LSR. **_The ingress LSR uses this to demultiplex the BFD session_**.
  
### BFD Discriminator TLV in LSP Ping
  LSP Ping Echo request and Echo reply messages carry a BFD discriminator TLV for the purpose of session establishment as described above.
  
  IANA has assigned **_a type value of 15 to this TLV_**. This TLV has a length 4. The value contains the 4-byte local discriminator that the LSR, sending the LSP Ping message, associates with the BFD session.
  
  If the BFD session is not in Up state, the periodic LSP Ping Echo request messages must include the BFD Discriminator TLV.
  
## Encapculation
  BFD Control packets sent by the ingress LSR must be encapsulated in the MPLS label stack that corresponding to the FEC for which detection is being performed. If the label stack has a depth greater than one, the TTL of the inner MPLS label may be set to 1. This may be necessary for certain FECs to enable the egress LSR's control plane to receive the packet. For MPLS PWs, alternatively, the presence of a fault detection message may be indicated by setting a bit in the control word.
  
  The BFD Control packet sent by the ingress LSR must be a UDP packet with a well-known destination port 3784 and a source port assigned by the sender as per the procedures in BFD-IP. The source IP address is a routable address of the sender.
  
  The destination IP address must be randomly chosen from the 127/8 range for IPv4 and from the 0:0:0:0:0:FFFF:7F00/104 range for IPv6 with the following exception. If the FEC is an LDP IP FEC, the ingress LSR may discover multiple alternate paths to the egress LSR for this FEC using LSP Ping traceroute.
  
  In this case, the destination IP address, used in a BFD session established for one such alternate path, is the address in the 127/8 range for IPv4 or 0:0:0:0:0:FFFF:7F00/104 range for IPv6 discovered by LSP Ping traceroute to exercise that particular alternate path.
  
  The IP TTL or Hop limit must be set to 1.
  
  BFD Control packet sent by the egress LSR are UDP packets. The source IP address is a routable address of the replier.
  
  The BFD Control packet sent by the egress LSR to the ingress LSR may be routed based on the destination IP address as per the procedures in BFD-MHOP. If this is the case, the destination IP address must be set to the source IP address of the LSP Ping Echo request message , received by the egress LSR from the ingress LSR.
  
  Or the BFD Control packet sent by the egress LSR to the ingress LSR may be encapsulated in an MPLS label stack. In this case, the presence of the fault detection message is indicated as described above. This may be the case if the FEC for which the fault detection is being performed corresponds to a bidirectional LSP from the egress LSR to the ingress LSR. In this case, the destination IP address must be randomly chosen from the 127/8 range for IPv4 and from the 0:0:0:0:0:FFFF:7F00/104 range for IPv6.
  
  The BFD Control packet sent by the egress LSR must have a well-known destination port 4784, if it is routed BFD-MHOP, or it must have a wll-known destination port 3784 BFD-IP if it is encapsulated in a MPLS label stack. The source port must be assigned by the egress LSR as per the procedures in BFD-IP.
  
  Note that once that BFD sesison for the MPLS LSP is Up, either end of the BFD session must not change the source IP address and the local discriminator values of the BFD Control packets it generates, unless it first brings down the session. This implies that an LSR must ignore BFD packets for a given session, demultiplexed using the Your Discriminator field, if the session is in Up state and if the My Disriminator or the Source IP address fields of the received packet do not match the valuies associated with the session.
  
## Security Considerations
  For BFD Control packets sent by the ingress LSR or when the BFD Control packet sent by the egress LSR are encapsulated in an MPLS label stack, MPLS security considerations apply.
  
  When BFD Control packets sent by the egress LSR are routed, the authentication considerations discuessed in BFD-MHOP should be followed.
  
## IANA Considerations
  This document introduces a BFD discriminator TLV in LSP Ping. The BFD Discriminator has been assgined a value of 15 from the LSP Ping TLVs and sub-TLVs registry maintained by IANA.
  
## Acknowledgments

## References
  
## My References
  * [TLV]()
  * [FEC]()
  * [egress, ingress]()
  * [traceroute]()
  
  
