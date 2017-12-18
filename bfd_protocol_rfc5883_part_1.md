# Bidirectional Forwarding Detection (BFD) for Multihop Paths
  This document describes the use of the BFD potocol over multihop paths, including unidirectional links.
  
## Introduction  
  The BFD one-hop specification describes how to use BFD across single hops in IPv4 and Ipv6.
  
  BFD can also be useful on arbitrary paths between systems, which may span multiple network hops and follow unpredictable paths. Furthermore, a pair of systems may have multiple paths between them that may overlap. 
  
## Applicability
  Please note that BFD is intended as an Operations, Administration, and Maintenance (OAM) mechanism for connectivity check and connection verification. It is applicable for network-based services (e.r. router-to-router, subscriber-to-gateway, LSP/circuit endpoints, and service applicance failure detection). 
  
  In these scenarios it is required that the operator correctly provision the raates at which BFD is transmitted to avoid congestion (e.g link, I/O, CPU) and false failure detection. 
  
  It is not applicable for application-to-application failure detection across the Internet because it does not have sufficient capability to do necessary congestion detection and avoidance and therefore cannot prevent congestion collaps.
  
  Host-to-host or application-to-application deployment across the Internet will required the encapsulation of BFD within a transport that **_provides "TCP-friendly" [TFRC] behavior_**.
  
## Issues
  There are three primary issues in use of BFD for multiple paths:
  
  * security and spoofing. BFD-1HOP describes a lightweight method of avoiding spoofing by requiring a Time to Live (TTL)/HOP Limit of 255 on both transmit and receive, but this obviously does not work across multiple hops. **_The untilization of BFD authentication addresses this issue_**.
  
  * Demultiplexing multiple BFD sessions between the same pair of systems to the proper BFD session. In particular, the first BFD packet received for a session may carry a Your Discriminator value of zero, resulting in ambiguity as to which session the packet should be associated. Once the discriminator values have been exchanged, **_all further packets are demultiplexed to the proper BFD session solely by the contents of the Your Discriminator field_**. BFD-1HOP addresses this by requiring that multiplt sessions traverse independent physical or logical links -- the first packet is demultiplexed base on the link over which it was received. In the more general case, this scheme cannot work, as two paths over which BFD is running may overlap to an arbitrary degree (including the first and/or last hop)
  
  * Echo function must not be used over multiple hops. Intermediate hops would route the packets back to the sender, and connectivity through the entire path would not be possible to verify.
  
## Demultiplexing Packets
  There are a number of possibilities for addressing the demultiplexing issue that may be used, depending on the appliction.
  
### Totally Arbitrary Paths
  It maybe desired to use BFD for liveness detection over paths for which no part 
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
