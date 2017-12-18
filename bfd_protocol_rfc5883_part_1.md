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
  It maybe desired to use BFD for liveness detection over paths for which no part of the route is known (or if knwon, may not be stable). A straightforward approach to this problem is to **_limit BFD deployment to a single session between a source/destination address pair_**. Multiple session between the same pair of systems must have at least one endpoint address distinct from one another.
  
  In this scenario, the initial packet is demultiplexed to the appropriate BFD session based on the source/destination aaddress pair when Your Discriminator is set to zero.
  
  This approach is appropriate for general connectivity detection between systems over routed paths and **_is also useful for OSPF Virtual Links_**.
  
### Out-of-Band Discriminator Signaling
  Another approach to the demultiplexing problems is to signal the discriminator values in each direction through an out-of-band mechanism prior to establishing the BFD session.
  
  Once learned, the discriminator are sent as usual in the BFD Control packets; no packets with Your Discriminator set to zero are ever sent. This method is used by **_the BFD MPLS specification [BFD-MPLS]_**.
  
  This approach is advantageous because it allows BFD to be directed by other system components that have knowledge of the path in use, and from the perspective of BFD implementation it is very simple.
  
  The disadvantage is that it requires at least some level of BFD-specific knowledge in parts of the system outside of BFD.
  
### Undirectional Links
  Unidirectional links are classified as multihop paths because the return path (which should exist at some level in order to make the link useful) may be arbitrary, and the return paths for BFD sessions protecting parallel unidirectional links may be overlap or even be identical. (If two unidirectional links, one in each direction, are to carry a single BFD session, this can be done using the single-hop approach).
  
  Either of the two method outlined earlier may be used in the unidirectional link case, but a more general solution can be found strictly within BFD and without addressing limitations.
  
  The approach is simmilar to the one-hop sepcification, since the unidirectional link is a single hop. Lets define the two systems as the Unidirectional Sender and Unidirectional Reciever. In this approach, the Unidirectional Sender must operate in the Active role, and the Unidirectional Recieved must operat in the Passive role.
  
  In the Passive role, by definition, the Unidirectional Receiver does not transmit any BFD Control Packets until it learns the discriminator value in use by the other system (upon receipt of the first BFD Control packet).
  
  The Unidirectional Receiver demutiplexes the first packet to the proper BFD session base on the physical or logical link over which it was received. This allows the receiver to learn the remote discriminator value, which it then echoes back to the sender in its own BFD Control packet, after which time all packet are demultiplexed solely by discriminator.
  
## Encapsulation
  The encapsulation of BFD Control packets for multihop appliaction in IPv4 and IPv6 is identical to that define in BFD-1HOP, **_except that the UDP destination port must have a value 4784_**. This can aid in the demultiplexing and internal routing of incoming BFD packets.
  
## Authentication
  By their nature, multihop paths expose BFD to spoofing. As the 
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
## My References
  * [OAM](https://baike.baidu.com/item/OAM/2260538?fr=aladdin)
  * [out-of-band](https://baike.baidu.com/item/out-of-band/15801641?fr=aladdin)
  

  
