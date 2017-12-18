# Bidirectional Forwarding Detection (BFD) for IPv4 and IPv6 (Single Hop)
  This document describes the use of the Bidirectional Forwarding Detection (BFD) protocol over IPv4 and IPv6 for single IP hops.
  
## Introduction
  One very desirable application for Bidirectional Forwarding Detection (BFD) is to **_track IPv4 and IPv6 connectivity between directly connected systems_**. This could be used to supplement the detection meachanisms in routing protocols or to monitor router-host connectivity, among other applications.
  
## Applications and Limitations
  This application of BFD can be used by any pair of systems communicating via IPv4 and/or IPv6 across a single IP hop that is asscociated with an incoming interface. This includes, but is not limited yo, physical media, virtual circuits, and tunnels.
  
  If the BFD Echo function is used, transmitted packets are immediately routed back towards the sender on the interface over which they were sent. This may interact with other mechanisms that are used on the two systems that employ BFD. In particular, ingress filtering is imcompatible with the way Echo packets need to be sent. **_Implementations that support the Echo function must ensure that ingress filtering is not used on an interface that employs the Echo function or make an exception for ingress filtering Echo packets_**.
  
  An implementation of the Echo function also requires APIs that may not exist on all systems. A system implementing the Echo function must be capable of sending packets to its own address, which will typically require bypassing the normal forwarding lookup. This typically requires access to APIs that bypass IP-layer functionality.
  
  Please note that BFD is intended as an Operations, Administration and Maintenance (OAM) mechanism for connectivity check and connection verification. It is applicable for network-based services (e.g., router-to-router, subscriber-to-gateway, LSP/circuit endpoints, and service appliance failure detection). In these scenatios it is required that the operator correctly provision the rates at which BFD is transmitted to avoid congestion and false failure detection. **_It is not applicable for application-to-application failure detection across the Internet because it does not have sufficient capability to do necessary congestion detection and avoidance and therefore cannot prevent congestion collaps_**. 
  
  Host-to-host or application-to-application deployment across the Internet will require the encapsulation of BFD within a transport that provides "TCP-friendly" [TFRC] behavior.
  
## Initialization and Demultiplexing 
  In this application, there will be only a single BFD session between 2 systems over a given interface (logical or physical) for a particular protocol. The BFD session must be bound to this interface.
  
  As such, both sides of a session must take the active role, and any BFD packet from the remote machine with a zero value of Your Discriminator must be associated with the session to the remote system, interface, and the protocol.
  
## Encapsulation
  BFD Control packets must be transmitted in UDP packets with **_destination port 3784, within an IPv4 or IPv6 packet_**. The source port must be in range **_49152 through 65535_**. The same UDP source port must be used for all BFD Control packets associated with a particular session. The source port should be unique among all BFD sessions on the system.
  
  If more than 16384 BFD sessions are simultaneously active, UDP source port numbers may be reused on multiple session, but the number of distinct uses of the same UDP source port number should be minimized. An implementation may use the UDP port source port number to aid in demultiplexing incoming BFD Control packets, but ultimately the mechanisms in BFD must be used to demultiplex incoming packets to the proper session.
  
  BFD Echo packets must be transmitted in UDP packets with destination UDP port **_3785 in an IPv4 or IPv6 packets_**. The setting of the UDP source port is outside the scope of this specification. The destination address must be chosen in such a way as to preclude the remote system from generating ICMP or Neighbor Discovery Redirect messages.
  
  In particular, the source address should not be part of the subnet bound to the interface over which the BFD Echo packet is being transmitted, and it should not be an IPv6 link-local address, unless it is known by other means that the remote system will not send Redirects.
  
  The above requirements may require the bypassing of some common IP layer functionality, particularly in host implementations.
  
## TTL/Hop Limit Issues
  If BFD authentication is not in use on a session, all BFD Control packets for the session must be sent with a Time to Live (TTL) or Hop Limit value of 255. All received BFD Control packets that are demultiplexed to the session must be **_discard if the received TTL or Hop Limit is not euqal to 255_**.  A discussion of this mechanism can be founf in GTSM.
  
  If BFD authentication is in use on a session, all BFD Control packets must be sent with a TTL or Hop Limit value of 255. All received BFD Control packets that are demultiplexed to the session may be discard if the received TTL or Hop Limit is not euqal to 255. If the TTL/Hop Limit check is made, it may be done before any cryptographic authentication takes place if this will avoid unnecessary calculation that would be detrimental to the receiving system.
  
  "authentication in use" means that the system is sending BFD Control packets with the Authentication bit set and with the Authentication Section included and that all unauthenticated packets demultiplexed to the session are discarded, per the BFD base specification.
  
## Addressing Issues  
  Implementations must ensure that all BFD Control packets are transmitted over the one-hop path being protected by BFD.
  
  On a multiaccess network, BFD Control pakcets must be transmitted with source and destination addresses that are part of the subnet (addresses from and to interfaces on the subnet).
  
  On a point-to-point link, the source address of a BFD Control packet must not be used to identify the session. This means that the initial BFD packets must be accepted with any source address, and that subsequent BFD packets must be demultiplexed solely by the Your Discriminator field (as is always the case). 
  
  This allows the source address to change if necessary. 
  
  If the received source address changes, the local system must not use that address as the destination in outgoing BFD Control packets; rather, **_it must continue to use the address configured at session creatation_**. An implementation may notify the application that the neighbor's source address has changed, so that the application might choose to change the destination address ot take some other action. 
  
  Note that the TTL/Hop Limit Check described in section 5 (or the use of authentication) precludes the BFD packets from having come from any source other than the immediate neighbor.
  
## BFD for Use with Tunnels
  A number of mechanism are available to tunnel IPv4 and IPv6 over arbitrary topologies. If the tunnel mechanism does not decrement the TTL or Hop Limit of the network protocol carried within, the mechanism described in this document may be used to provide liveness detection for the tunnel. The BFD authentication mechanism should be used and is strongly encouraged.
  
## IANA Considerations
  Ports **_3784 and 3875_** were assigned by IANA for use with the BFD Control and BFD Echo protocols, respectively.
  
## Security Considerations
  In this application, the use of TTL=255 on transmit and receive, coupled with an association to an incoming interface, is viewed as supplying equivalent security characteristic to other protocols used in the infrastructure, as it is not trivially spoofable
  
  The use of the TTL=255 check simultaneously with BFD authentication provides a low overhead mechanism for discarding a class of unauthorized packets and may be useful in implementations in which cryptographic checksum use is susceptible to denial-of-service attacks. The use or non-use of this mechanism does not impact interoperability.
  
## References  

## My References
  * [TFRC](https://baike.baidu.com/item/TFRC/6527816?fr=aladdin)
  * [GTSM](https://tools.ietf.org/pdf/rfc5082.pdf)
