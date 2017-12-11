# Bidirectional Forwarding Detection (BFD)
  It intends to detect faults in the bidirectional path between two forwarding engines, including interfaces, data link(s), and to the extent possible the forwarding engines themselves, with potentially very low latency. It operates independently of media, data protocols, and routing protocols.
  
## Introduction
  An increasingly impotant feature of networking equipment is the rapid detection of communication failures between adjacent systems, in order to more quickly establish alternative paths. Detection can some fairly quickly in certian circumstances when data link hardware comes into play (such as Synchronous Optical Network (SONET) alarms). However, there are media that do not provide this kind of signaling (such as Ethernet), and some media may not detect certain kinds of failures in the path, for example, failing interfaces or forwarding engine components.
  
  Networks use raletively slow "Hello" mechanisms, usaully in routing protocols. Furthermore, routing protocol Hellos are of no help when those routing protocols are not in use, and the semantics of detection are subtly different -- they detect a failure in the path between the two routing protocol engines.

  The goal of Bidirectional Forwarding Detection (BFD) is to provide low-overhead, short-duration detection of failure in the path between adjacent forwarding engines, including the interfaces, data links, and to the extent possible, the forwarding engines thenselves.
  
  An additional goal is to provide a single mechanism that can be used for liveness detection over any media, at any protocol layer, with a wide range of Detection Times and overhead, to avoid a proliferation of different methods.
  
## Design  
  **_BFD is designed to detect failures in communication with a forwarding plane next hop. It is intended to be implemented in some component of the forwarding engine of a system_**, in cases where the forwarding and control engines are separated. This not only binds the protocol more to the forwarding plane, but decouples the protocol from the fate of the routing protocol engine, making it useful in concern with various "graceful restart" mechanisms for thoes protocols. BFD may also be implemented in the control engine, though doing so may preclude the detecttion of some kinds of failures.
  
  **_BFD operates on the top of any data protocol (network layer, link layer, tunnels, etc.) being forwarded between two systems_**. It is always run in a unicast, point-to-point mode. **_BFD packets are carried as the payload of whatever encapsulating protocol is appropriate for the medium and network_**. BFD maybe be running at multiple layers in a system. The context of the operation of any parrticular BFD session is bound to its encapsulation.
  
  BFD can provide failure detecttion on any kind of path between systems, including direct physical links, virtual circuits, tunnels, MPLS Label Switched Path (LSPs), multihop routed paths, and undirectional links (so long as there is some return path, of course). **_Multiple BFD sessions can be established between the same pair of system when multiple paths between them are present in at least one direction, even if a lesser number of paths are available in the other direction (multiple parallel undirectional links or MPLS LSPs, for example)_**.
  
  The BFS state machine implements a **_three-way handshake_**, both when establishing a BFD session and when tearing it down for any reason, to ensure that both systems are aware of the state change.
  
  BFS can be abstracted as a simple service. The service primitives provided by BFD are to create, destroy, and modify a session, given the destination address and other parameters. BFD in return provides a signal to its clients indicating when the BFD session goes up or down.
  
## Protocol Overview  
  BFD is a simple Hello protocol that, in many respects, is similar to the detection coponents of well-known routing protocols. A pair of systems transmit BFD packets periodically over each path between the two systems. Under some conditions, systems may negotiate not to send periodic BFD packets in order to reduce overhead.
  
  The path is only declared to be operational when two way communication has been establishedd between systems, though this does not preclude the use of unidirectional links.
  
  A separate BFD session is created for each communications path and data protocol in use between two systems.
  
  Each system estimates how quickly it can send and receive BFD packets in order to come to an agreement with its neighbor about how rapidly detection of failure will take place. These estimates can be modified in real time in order to adapte to unusual situations. This desgin also allows for fast systems on shared medium with a slow system to be able to more rapidly detect failures between the fast systems while allowing the slow system to participate to the best of its ability.
  
### Addressing and Session Establishment  
  A BFD session is established based on the needs of the application that will be making use of it. It is up to the application to determine the need for BFD, and the addresses to use -- there is no discovery mechanism in BFD. For example, an OSPF implementation may request a BFD session to be established to be a neighbor discovered using the OSPF Hello protocol.
  
### Operating Modes  
  BFD has two operating modes that may be selected, as well as an additional function





























## Issues:
  * The UDP port is 3784, how to generate multiple BFD session on a single port?



**_page 5_**
