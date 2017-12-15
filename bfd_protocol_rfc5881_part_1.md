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
  BFD Control packets must be transmitted in UDP packets
  
  
  
  
  
  
  
  
  
  
  
  
  
  
## My References
  * [TFRC](https://baike.baidu.com/item/TFRC/6527816?fr=aladdin)
