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
  BFD has two operating modes that may be selected, as well as an additional function that can be used in combination with the two modes.
  
  The primary mode is known as Asynchronous mode. In this mode, the system periodically sent BFD Control packets to one another, and if a number of those packets in a row are not received by the other system, the session is decalred to be down.
  
  The second mode is known as Demand mode. In this mode, it is assumed that a system has an independent way of verifying that it has connectivity to the other system. Once a BFD session is established, such a system may ask the other system to stop sending BFD Control packets, except when the system feels the need to verify connectivity explicitly, in which case a short sequence of BFD Control packets is exchanged, and then the far system quiesces. Demand mode may operate independently in each direction, or simultaneously.
  
  An adjunct to both modes is the Echo function. When the Echo function is active, a stream of BFD Echo packets is transmitted in such a way as to have the other system loop them back through its forwarding path. If a number of packets of the echoed data stream are not received, the session is declared to be down. The Echo function may be used with either Asynchronous or Demand mode. Since the Echo function is handing the task od detection, the rate of periodic transmission of Control packets may be recuded or eliminated completely.
  
  Pure Asynchronous mode is advantageous in that it requires half as many packets to achieve a particular Detection Time as does the Echo function. It is also used when the Echo function cannot be supported for some reason.
  
  The Echo function has the advantage of truly testing only the forwarding path on the remote system. This may recude round-trip jitter and thus allow more aggressive Detection Times, as well as potentially detecting some classes of failure that might not otherwise be detected.
  
  The Echo function may be enabled individually in each direction. It is enabled in a particular direction only **_when the system that loops the Echo packets back signals that it will allow it, and when the system that sends the Echo packets decides it wishes to_**.
  
  Demand mode is useful in situations where the overhead of a periodic protocol might prove onerous, such as a system with a very large number of BFD symmetrically. It is also useful when the Echo function is being used symmetrically. Demand mode has the disadvantage that Detection Times are essetially driven by the heuristics of the system implementation and are not the BFD protocol. Demand mode may not be used when the path round-trip time is greater than the desired Detection Time, or the protocol will fail to work properly.
  
## BFD Control Pakcet Format  
### Generic BFD Control Packet Format
  BFD Control pakcets are sent in an encapsulation appropriate to the environment.
  
  The BFD Control packet has a Mandatory Section and an optional Authentication Section. The format of the Authentication Section, if present, is dependent on the type of authentication in use.
  
  The Mendatory Section of a BFD Control packet has the following format:
  
  Vers|Diag|Sta|P|F|C|A|D|M|Detect Mult|Length|My Discriminator|Your Discriminator|Desired Min TX Interval|Required Min RX Interval|Required Min Echo RX Interval
  ----|----|---|-|-|-|-|-|-|-----------|------|----------------|------------------|-----------------------|--------|------
  3|5|2|1|1|1|1|1|1|8|8|32|32|32|32|32
  
  
  An optional Authentication Section May be present:
  
  Auth Type|Auth Len|Authentication Data
  ---------|--------|-------------------
  8|8|16
  
  
  * Version (Vers): The version number of the protocol. This document defines protocol version 1.
  * Diagnotic (Diag): A diagnostic code specifying the local system's reason for last change in session state (This field allows remote systems to determine the reason that the previous session failed, for example):
    * 0 -- No Diagnostic
    * 1 -- Control Detection Time Expired
    * 2 -- Echo Function Failed
    * 3 -- Neighbor Signal Session Down
    * 4 -- Forwarding Plane Reset
    * 5 -- Path Down
    * 6 -- Concatenated Path Down
    * 7 -- Administratively Down
    * 8 -- Reverse Concatenated Path Down
    * 9-31 --  Reserved for future use
  * State (Sta): The current BFD session staate as seen by the transmitting system, values are:
    * 0 -- AdminDown
    * 1 -- Down
    * 2 -- Init
    * 3 -- Up
  * Poll (P): 
    * If set, the transmitting system is requesting verification of connectivity, or of a parameter change, and is expecting a packet with the Final (F) bit in reply
    * If clear, the transimitting system is not requesting verification
  * Final (F):
    * If set, the transimitting system is responding to a receive BFD Control packet that had the Poll (P) bit set
    * If clear, the transmitting system is not responding to a Poll
  * Control Plane Independent (C):
    * If set, the transmitting system's BFD implementation does not share fate with its control plane (in other words, BFD is implemented in the forwarding plane and can continue to function through disruptions in the control plane)
    * If clear, the transmitting system's BFD implementation shares fate with its control plane
  * Authentication Present (A):
    * If set, the Authentication Section is present and the session is to be authenticated
  * Demand (D):
    * If set, Demand mode is active in the transimitting system (the system wishes to operate in Demand mode, knows that the session is Up in both directions, and is directing the remove system to cease the periodic transmission of BFD Control packets)
    * If clear, Demand mode is not active in the transmitting system
  * Multipoint (M): This bit is reserved for furture point-to-multipoint extensions to BFD. It MUST be zero on both transmit and receipt
  * Detect Mult: Detection time multiplier. The negotiate transmit interval, multiplied by this value, provides the Detection Time for the receiving system in Asynchronous mode
  * Length: Length of the BFD Control packet, **_in bytes_**
  * My Discriminator: A unique, nonzero discriminator value generated by the transmitting system, used to demultiplex multiple BFD sessions between the same pair of systems
  * Your Discriminator: Thee discriminator received from the corresponding remote system, this field reflects back the received value of My Discriminator, or is zero if that value is unknown
  * Desired Min TX Interval: This is the minimum interval, in microseconds, that the local system would like to use when transmitting BFD Control packets, less any jitter applied. The value zero is reserved
  * Required Min RX Interval: This is the minimum interval, in microseconds, between received BFD Control packets that this system is capable of supporting, less any jitter applied by the sender. If this value is zero, the transimitting system does no want the remote system to send any periodic BFD Control packets
  * Required Min Echo RX Interval: This is the minimum interval, in microseconds, between received BFD Echo packets that this system is capable of supporting, less any jitter applied by the sender. If this value is zero, the transimitting system does not support the receipt of BFD Echo packets
  
  * Auth Type: The authentication type in use, if the Authentication Present (A) bit is set:
    * 0 -- Reserved
    * 1 -- Simple Password
    * 2 -- Keyed MD5
    * 3 -- Meticulous Keyed MD5
    * 4 -- Keyed SHA1
    * 5 -- Meticulous Keyed SHA1
    * 6-255 -- Reserved for future use
  * Auth Len: The length, **_in bytes_**, of the authenticaton section, including Auth Type and Auth Len fields.
  
### Simple Password Authentiation Section Format  
  If the Authentication Present (A) bit is set in the header, and the Authentication Type field contains 1 (Simple Password), the Authentication Section has the following format:
  
  Auth Type|Auth Len|Auth Key ID|Password
  ---------|--------|-----------|--------
  8|8|8|8
  
  * Auth Type: 1
  * Auth Len: 3 + len(Password)
  * Auth Key ID: The authentication key ID in use for this packet. This allow **_multiple keys to be active simultaneously_**.
  * Password: The simple password in use on this session. The password is a binary string, and MUST be from 1 to 16 bytes in length. The password MUST be encoded and configured.
  
### Keyed MD5 and Meticulous Keyed MD5 Authentication Section Format
  **_The use of MD5-base authentication is strongly discouraged_**. However, it is documented here for compatibility with existing emplementation.
  
  If the Authentication Present (A) bit is set in the header, and the Authentication Type field contains 2 (Keyed MD5) or 3 (Meticulous Keyed MD5), the Authentication Section has the following format:
  
  Auth Type|Auth Len|Auth Key ID|Reserved|Sequence Number|Auth Key/Digest
  ---------|--------|-----------|--------|---------------|---------------
  8|8|8|8|32|32
  
  * Auth Type: 2/3
  * Auth Len: The length of the Authentication Section, in bytes. For keyed MD5 and Meticulous Keyed MD5 authentication, the length is 24 (=4 + 4 + 16)
  * Auth Key ID: The authentication key ID is use for this packet. This allows mutiple keys to be active simultaneously
  * Reserved: This byte MUST be set to zero on transmit, and ignore on receipt
  * Sequence Number: The sequence number for this packet. **_For Keyed MD5 Authentication, this value is incremented occasionally. For meticulous Keyed MD5 Authentication, this value is incremented for each successive packet transmitted for a session_**. This provides protection against replay attacks
  * Auth Key/Digest: This field carries the 16-byte MD5 digest for the packet. When the digest is calsulated, the shared MD5 key is stored in this field, **_padded to 16 bytes with trailing zero bytes if needed_**.
  
### Keyd SHA1 and Meticulous keyed SHA1 Authentication Section Format  
  
  
  
  
  
  
  
  
    
  
    





























## Issues:
  * The UDP port is 3784, how to generate multiple BFD session on a single port?



**_page 11_**
