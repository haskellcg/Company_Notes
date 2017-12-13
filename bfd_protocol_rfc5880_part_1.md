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
  8|8|8|8|32|128
  
  * Auth Type: 2/3
  * Auth Len: The length of the Authentication Section, in bytes. For keyed MD5 and Meticulous Keyed MD5 authentication, the length is 24 (=4 + 4 + 16)
  * Auth Key ID: The authentication key ID is use for this packet. This allows mutiple keys to be active simultaneously
  * Reserved: This byte MUST be set to zero on transmit, and ignore on receipt
  * Sequence Number: The sequence number for this packet. **_For Keyed MD5 Authentication, this value is incremented occasionally. For meticulous Keyed MD5 Authentication, this value is incremented for each successive packet transmitted for a session_**. This provides protection against replay attacks
  * Auth Key/Digest: This field carries the 16-byte MD5 digest for the packet. When the digest is calsulated, the shared MD5 key is stored in this field, **_padded to 16 bytes with trailing zero bytes if needed_**.
  
### Keyd SHA1 and Meticulous keyed SHA1 Authentication Section Format  
  If the Authentication Present (A) bit is set in the header, and the Authentication Type field contains 4 (Keyed SHA1) or 5 (Meticulous keyed SHA1), the Authentication Section has the following format:
  
  Auth Type|Auth Len|Auth Key ID|Reserved|Sequence Number|Auth Key/Hash
  ---------|--------|-----------|--------|---------------|---------------
  8|8|8|8|32|160
  
  * Auth Type: 4/5
  * Auth Len: For Keyed SHA1 and Meticulous Keyed SHA1 authenrication, the length is 28 (= 4 + 4 + 20)
  * Auth Key ID: The authentication key ID in use for this packet. This allow mutiple keys to be active simultaneously.
  * Reserved: This byte MUST be set to zero on transmit and ignore on receipt
  * Sequence Number: The sequence number for this packet. **_For Keyed SHA1 Authentication, this value is incremented occasionally. For Meticulous Keyed SHA1 Authentication, this value is incremented for each successive packet transmitted for a session_**. This provides protection against replay attacks
  * Auth Key/Hash: This field carries the 20-byte SHA1 hash for the packet. When the hash is calculated, the shared SHA1 key is stored in this field, padded to a length of 20 bytes with trailing zero bytes if needed.
  
## BFD Echo Packet Format  
  **_BFD Echo packets are sent in an encapsulation appropriate to the environment_**. See the appropriate application documents for the specifics of particular environments.
  
  The payload of a BFD Echo packet is a local matter, since only the sending system ever processes the content. The only requirement is that sufficient information is included to demultiplex the received packet to the correct BFD session after it is looped back to the sender.
  
  Some form of authentication SHOULD be included, since Echo packets may be spoofed.
  
## Elements of Procedure  
  This section discusses the normative requirements of the protocol in order to achieve interoperability. **_It is important for implementor to enforce only the requirement specified in this section, as misguided pedantry has been proven by experience to affect interoperability adversely_**.
  
### Overview  
  A system may take either an Active role or a Passive role **_in session initialization_**.
  
  **_At least one system must take the Active role (possibly both)_**. The role that a system takes is specific to the application of BFD, and is outside the scope of this specification.
  
  **_A session begins with the periodic, slow transmission of BFD Control packets_**. When bidirectional communication is achived, the BFD session becomes Up.
  
  Once the BFD session is Up, a system can choose to start the Echo function if it desires and the other system signals that it will allow it. **_The rate of transmission of Control packet is typically kept low when the Echo function is active_**.
  
  If the Echo function is not active, the transmission rate of Control packets may be increased to a level necessary to achive the Detection Time requirements for the session.
  
  **_Once the session is Up, a system may signal that it has entered Demand mode, and the transmission of BFD Control packets by remote system ceases_**. Other means of implying connectivity are used to keep the session alive. If either system wishes to verify bidirectional connectivity, it can initiate a short exchange of BFD Control packets.
  
  If Demand mode is not active, and no Control packets are received in the calculated Detection Time, the session is declared Down. This is signaled to the remote end via the state (Sta) field in outgoing packets.
  
  If **_sufficient_** Echo packets are lost, the session is declared Down in the same manner.
  
  If Demand mode is active and no approptiate Control packets are received in response to a **_Poll Sequence_**, the session is declared Down in the same manner.
  
  If the session goes Down, the transmission of Echo packets (if any) ceases, and the transmission of Control packets goes back to the **_slow rate_**.
  
  Once a session has been declared Down, it cannot come back up until the remote end first signals that it is down (by leaving the Up state), thus implementing a three-way handshake.
  
  **_A session may be kept administratively down by entering the AdminDown state and sending an explanatory diagostic code inthe Diagnostic field_**.
  
### BFD State Machine  
  This allow a three-way handshakes for both session establishment and session teardown (**_assuring that both systems are aware of all session state changes_**). A fouth state (AdminDown) exists so that a session can be administratively put down indefinitely.
  
  **_Echo system communicates its session state in the State field in the BFD Control packet, and that received state, in combination with the lodal session state, drives the state machine_**.
  
  Down state means that **_the session is down (or has just been created)_**. A session remains in Down state until the remote system indicates that it agrees that the session is down by sending a BFD Control packet with State field set to anything other than Up. **_If the packet signals Down state_**, the session advances to Init state; **_if that packet signals Init state_**, the session advances to Up state. Semantically, Down state indicates that the forwarding path is unavailable, and that appropriate actions should be taken by the applications monitoring the state of the BFD session. A system may hold a session in Down state indefinitely (by simply refusing to advance the session state). **_This may be done for operational or administrative reasons, among others_**.
  
  Init state means that **_remote system is communicating_** , and the local system desires to bring the session up, but the remote system does not yet realize it. A session will remian in Init state until either a BFD Control Packet is received that is signaling Init or Up state (**_in which case the session advances to Up state_**) or the Detection Time expires, meaning that communicating with the remote system has been lost (**_in which case the session advances to Down state_**).
  
  Up state means that the BFD session has successfully been established, and implies that connectivity between the system is working. The session will remain in the Up state **_until either connectivity fails or the session is taken down administratively_**. If either the remote system **_signals Down state or the Detection Time expires_**, the session advances to Down state.
  
  AdminDown state means that **_the session is being held administratively down_**. This causes the remote system to enter Down state, and remain there until the local system exits AdminDown state. AdminDown state has no semantic implications for the availability of the forwarding path.
  
  The following diagram provides an overview of the state machine:
  
  ![BFD state machine](https://github.com/haskellcg/Blog_Pictures/blob/master/bfd_protocol_rfc5880_part_1_1.PNG)
  
### Demultiplexing and the Discriminator Fields
  Since multiple BFD sessions may be running between two systems, there needs to be a machanism for demultiplexing received BFD packets to the peoper session.
  
  **_Each system must choose an opaque discriminator value that identifies each session, and which must be unique among all BFD sessions on the system_**. The local discriminator is sent in the My Discriminator field in the BFD Control packet, and is **_echoed back in the Your Discriminator field of packets sent from the remote end_**.
  
  Once the remote end echoes back the local discriminator, all further received packets are demultiplexed based on the Your Discriminiator field only (**_which means that, among other things, the source address field can change or the interface over which the packets are received change, but the packets will still be associated with the proper session_**).
  
  The method of demultiplexing the initial packets (in which Your Discriminator is zero) is **_application dependent_**.
  
  Note that it is permissible for a system to change its discriminator during a session without affecting the session state, since only **_that system uses its discriminator for demultiplexing purpose (by having the other system reflect it back)_**.
  
### The Echo Function and Asymmetry  
  The Echo function can be run independently in each direction between a pair of systems. For whatever reason, a system may advertise that it is willing to receive (and loop back) Echo packets, but may not wish to ever send any. **_The fact that a system is sending Echo packets is not directly signaled to the system looping them back_**.
  
  When a system is using the Echo function, it is advantageous to choose a sedate reception rate for Control packets, since liveness detection is being handle by the Echo packets. This can be controlled by manipulating the Required Min RX Interval field.
  
  If the Echo function is only being run in one direction, the system not running the Echo function will more liekly wish to receive fairly rapid Control packets in order to achieve its desired Detection Time. Since BFD allows independent transmission rates in each direction, this is easily accomplished.
  
  A system should otherwise advertise the lowest value of Required Min RX interval and Required Min Echo RX Interval that it can under the circumstances, to give the other system more freedom in choosing its transmission rate. Note that a system is committing to be able to receive both streams of packets at the rate it advertises, so this should be take into acount when choosing the value to advertise
  
### The Poll Sequence  
  A Poll Sequence is an exchange of BFD Control packets that is used in some circumstances to **_ensure that the remote system is aware of parameter changes_**. It is also used in Demand mode to validate bidirectional connctivity.
  
  A Poll Sequence consists of a system sending periodic BFD Control packets with the Poll (P) bit set. When the other system receives a Poll, it immediately transmits a BFD Control packet with the Final (F) bit set, independent of any periodic BFD Control packets it may be sending. When the system sending the Poll sequence receives a packet wih Final, the Poll sequence is terminated, and any subsequent BFD Control packets are sent with the Poll bit cleared. **_A BFD Control packet must not have both the Poll (P) and Final (F) bits set_**.
  
  If periodic BFD Control packets are already being sent (the remote system is not Demand mode), the Poll Sequence must be performed by setting the Poll (P) bit on those scheduled periodic transmissions, additional packets must not be sent.
  
  After a Poll Sequence is terminated, the system requesting the Poll Sequence will cease the periodic transimission of BFD Control packets if the remote end is **_in Demand mode_**; otherwise, it will return to the periodic transmission of BFD Control packets with the Poll (P) bit clear.
  
  Typically, the entire sequence consists of a single packet in each direction, though packet losses or relatively long packet latencies may result in multiple Poll packets to be sent before the sequence terminates.
    
### Demand Mode
  Demand mode is requested independently in each direction by virtue of a system setting the Demand (D) bit in its BFD Control packets. The system receiving the Demand bit cease the periodic transimission of BFD Control packets. If both systems are oprating in Demand mode, no periodic BFD Control packets will flow in either direction.
  
  Demand mode requires that some mechanism is used to imply continuing connectivity between the two systems. **_The mechanism used does not have to be the same in both directions_**. One possible mechanism is the receipt of traffic from the remote system; another is the use of the Echo function.
  
  When a system in Demand mode wishes to verify bidirectional connectivity, **_it initiates a Poll Sequence_**. If no response is received to a Poll, the Poll is repeated until the Detection Time expires, at which point the session is declared to be Down. Note that **_if Demand mode is operating only on the local system_**, the Poll Sequence is performed by simply setting the Poll (P) bit in regular periodic BFD Control packets.
  
  **_The Detection Time in Demand mode_** is calculated differently than in Asynchronous mode; it is base on the transmit rate of the local system, rather than the transmit rate of the remote system.
  
  Note that the Poll mechanism will always fail unless the negotiated Detection Time is greater than the round-trip time between the two systems.
  
  Demand mode MAY be enabled or disabled at any time, independently in each direction, by setting or clearing the Demand (D) bit  in the BFD Control packet, without affecting the BFD session state. **_Note that the Demand bit must not be set unless both systems pereive the session to be Up_**.
  
  When the transimitted value of the Demand (D) bit is to be changed, the transmitting system must initiate a Poll Sequence in conjunction with changing the bit in order to ensure that both system are aware of the change.
  
  If Denmand mode is active on either or bother systems, a Poll Sequence must be intiate whenever the contents of the next BFD Control packet to be sent would be different than the contents of the previous packet, with the exception of the Poll (P) and Final (F) bits. **_This ensure that parameter changes are transmitted to the remote system and that the remote system acknowledges these changes_**.
  
  **_The total Detection Time for a particular system is the sum of the time prior to the initiation of the Poll Sequence, plus the calculated Detection Time_**.
  
  Note that if Demand mode is enabled in only one direction, continuous bidirectional connectivity verification is lost (only connectivity in the direction from the system in Demand mode to the other system will be verified).
  
### Authentication  
  Implementations supporting authentication must support both types of SHA1 authentication. Other forms of authentication are optional.

#### Enabling and Disabling Authentication
  In a simple implementation, a BFD session will fail when authentication is either turned on or turned off, because the packet acceptance rules essentially require the local and remote machines to do so in a more or less synchronized fashion (within the Detection Time) -- a packet with authentication will only be accept if authentication is "in use".
  
#### Simple Password Authentication  
  The receiving system accepts he packet if the Password and Key ID macthes one of the Password/ID pairs configured in that system.
  
  **_The Auth Len field must be set to the proper length (4 to 19 bytes = 1-16 + 3)_**.
  
  For interoperability, the management interface by which the password is configured must accept ASCII strings, and should also allow the configuration of any arbitrary binary string in hexadecimal form.
  
#### Keyed MD5 and Meticulous Keyed MD5 Authentication  
  In these methods of authentication, one or more secret keys (with corresponding key IDs) are configured in each system. One of the keys is included in an MD5 digest calculated over the outgoing BFD Control packet, but the Key itself is not carried in the packet. To help avoid replay attacks, a sequence number is also carried in each packet.
  
  The receiving system accepts the packet **_if the key ID macthes one of the configured Keys, an MD5 digest including the selected key matches that carried in the packet, and the sequence number is greater than or equal to the last sequence number received (for Keyed MD5), or strictly greater than the last sequence number received (for Meticulous Keyed MD5)_**.
  
  **_An MD5 digest must be calculated over the entire BFD Control packet_**. The resulting digest must be stored in the Auth Key/Digest field prior to transmission.

  For Keyed MD5, **_bfd.XmitAuthSeq_** may be incremented in a criculae fashion (when treated as an unsigned 32-bit value). bfd.XmitAuthSeq should be incremented when the session state changes, or when the transmitted BFD Control packet carries different contents than the previously transmitted packet.
  
  If **_bfd.AuthSeqKnown_** is 1, examine the Sequence Number field. For Keyed MD5, if the sequence number lies outside of the range of **_bfd.RcvAuthSeq to bfd.RcvAuthSeq+(3*Detect Mult)_** inclusive, the receivd packet must be discarded. For Meticulous Keyed MD5, if the sequence number lies outside of the range of **_bfd.RcvAuthSeq+1 to bfd.RcvAuthSeq+(3*Detect Mult)_** inclusive the received packet must be discarded.
  
  Otherwise, **_bfd.AuthSeq_** must be set to 1, and **_bfd.RcvAuthSeq_** must be set to the value of the received Sequence Number field.

#### Keyed SHA1 and Meticulous Keyed SHA1 Authentication
  Same as MD5 part.
   
### Functional Specifics   
  **_the Echo function active_**
  
  **_local Demand mode active: bfd.DemandMode_**
  
  **_remote Demand mode active: bfd.RemoteDemandMode_**
  
#### State Variables 
  A minimum amount of information about a session needs to be tracked in order to achieve the elements of procedure described here. The following is a set of state variables that are helpful in describing the mechanisms of BFD. Any means of tracking this state may be used so long as the protocol behaves as described.
  
  When the text refers to initializing a state variable, this takes place only at the time that the session (and the corresponding state variables) is created. The state variable are subsequently manipulated by the state machine and are never reinitialized, even if the session fails and is reestablished.
  
  Once session state is created, and at least one BFD Control packet is received from the remote end, it must be preserved for at least one Detection Time subsequent to the receipt of the last BFD Control packet, regardless of the session state.
  
  * bfd.SessionState: The perceived state of the session (Init, Up, Down, AdminDown). It is expected that this state change (particularly, to and from Up state) is reported to other components of the system. **_This varibale must be initialized to Down_**.
  
  * bfd.RemoteSessionState: The session state last reported by the remote system in the State (Sta) field of the BFD Control Control packet. **_This variable must be initialized to Down_**.
  
  * bfd.LocalDiscr: The local discriminator for this BFD session, used to uniquely identify it. It must be unique across all BFD sessions on this systems, and nonzero. **_It should be set to random (but still unique) value to improve security_**.
  
  * bfd.RemoteDiscr: The remote discriminator for this BFD session. This is the discriminator chosen by the remote system, and is totally opaque to local system. **_This must be initialized to zero_**. If a period of a Detection Time passes without the receipt of a valid, authenticated BFD packet from the remote system, this variable must be set to zero.
  
  * bfd.LocalDiag: the diagnostic code specifying the reason for the most recent change in the local session state. **_This must be initialized to zero (No Diagnostic)_**.
  
  * bfd.DesiredMinTxInterval: The minimum interval, in microseconds, between transmitted BFD Control packets that this system would like to use at the current time, less any jitter applied. The actual interval is negotiated between the two systems. This must be initialized to a value of **_at least one second (1,000,000 microseconds)_**.
  
  * bfd.RequiredMinRxInterval: The minimum interval, in microseconds, between received BFD Control packets that this system requires, less any jitter applied by the sender. **_A value of zero means that this system does not want to receive any periodic BFD control packets_**.
  
  * bfd.RemoteMinRxInterval: The last value of Required Min RX Interval received from the remote system in a BFD Control packet. **_This variable must be initialized to 1_**.
  
  * bfd.DemandMode: Set to 1 if the local system wishes to use Demand mode, or 0 if not.
  
  * bfd.RemoteDemandMode: Set to 1 if the remote system wishes to use Demand mode, or 0 if not. This is the value of the Demand (D) bit in the last received BFD Control packet. **_This variable must be intialized to zero_**.
  
  * bfd.DetectMult: The desired Detection Time multiplier for BFD Control packets on the local system. The negoriated Control packet transmission interval, multiplied by this variable, will be the Detection Time for this session. **_This variable must be a nonzero integer_**.
  
  * bfd.AuthType: The authentication type in use for this session, or zero if no authentication is in use
  
  * bfd.RcvAuthSeq: A 32-bit unsigned integer containing the last sequence number for Keyed MD5 or SHA1 Authentication that was received. **_The Initial value is unimportant_**.
  
  * bfd.XmitAuthSeq: A 32-bit unsigned integer containing the next sequence number for Keyed MD5 or SHA1 Authentication to be transmitted. **_This variable must be intialized to a random 32-bit value_**.
  
  * bfd.AuthSeqKnown: Set to 1 if the next sequence number for Keyed MD5 or SHA1 authentication expected to be received is known, or 0 if it is not known. **_This variable must be initialized to zero_**. This variable must set to be zero after no packets have been received on this session for at least twice the Detection Time. **_This ensures that the sequence number can be resynchronized if the remote system restarts_**.
  
#### Timer Negotiation  
  The time values used to determine BFD packet transmission intervals and the session Detection Time are continuously negotiated, and thus may be changed at any time. **_The negotiation and time value are independent in each direction for each session_**.
  
  Each system reports in the BFD Control packet how rapidly it would like to transmit BFD packets, as well as how rapidly it is prepare to receive them. This allow either **_system to unilaterally determine the maximum packet rate in both directions_**.
  
#### Timer Manipulation
  The time values used to determine BFD packet transimission intervals and the session Detection Time may be modified at any time without affecting the state of the session.
  
  If either bfd.DesiredMinTxInterval is changed or bfd.RequiredMinRxInterval is changed, a Poll Sequence must be intiated. **_If the timming is such that a system receiving a Poll sequence wishes to change the paramaters described in this paragraph, the new parameter vaues may be carried in packets with the Final (F) bit set, even if the Poll sequence has not yet been sent_**.

  If bfd.DesiredMinTxInterval is increased and bfd.SessionState is Up, the autual transimission interval must not change until the Poll Sequence described above has terminated. **_This is to ensure that the remote system updates its Detection Time before the transimission interval increases_**.
  
  If bfd.RequiredMinRxInterval is reduced and bfd.SessionState is Up, the previous value of bfd.RequiredMinRxInterval must be used when calculating the Detection Time for the remote system until the Poll Sequence described above has terminated. **_This is to ensure that the remote system is transmitting packets at the higher rate (and those packets are being received) prior to the Detection Time being reduced_**.
  
  When bfd.SessionState is not Up, the system must set bfd.DesiredMinTxInterval to a value of not less than one second. **_This is intended to ensure that the bandwidth consumed by BFD sessions that are not Up is negligible, particularly in the case where a neighbor may bot be running BFD_**.
  
  If the local system reduces its transmit interval due to bfd.RemoteMinRxInterval being reduced (the remote system has advertised a reduced value in Required Min Rx Interval), and the remote system is not in Demand mode, the local system must honor the new interval immediately. In other words, the local system cannot wait longer than the new interval between the previous packet transmission and the next one.

  When the Echo function is active, a system should set bfd.RequiredMinRxInterval to a value of not less than one second. **_This is intend to keep received BFD Contrpl traffic at a negligible level, since the actual detection function is being performed using BFD Echo packets_**.
  
  **_In any case than those explicitly called out abve, timing parameter changes must be effected immediately (changing the transmission rate and/or the Detection Time)_**.
  
  Note that the Poll Sequence mechanism is ambiguous if more than one parameter change is made that would required its use, and those multiple changes are spread across multiple packets (**_since the semantics of the returning Final are unclear_**). Therefore, if multiple changes are mode that require the use of a Poll Sequence, there are 3 choinces:
  * They must have communicated in a single BFD Control packet (so the semantics of the Final replay are clear)
  * completed to disambiguate the situation (at least a round trip time since the last Poll was transmitted) prior to the initiation of another Poll Sequence
  * an addition BFD Control packet with the Final (F) bit clear must be received after the Poll Sequence has completed prior to the initiation of another Poll Sequence (this option is not available when Demand mode is active)
  
#### Calculating the Detection Time  
  **_The Detection Time (the period of time without receiving BFD packets after which the session is determined to have failed) is not carried explicitly in the protocol_**.
  
  **_Note that there may be different Detection Times in each direction_**.
  
  **_The calculation of the Detection Time is slightly different when in Demand mode versus Asynchronous mode_**.
  
  In Asynchronous mode (**_The Detect Mult value is the number of packets that have to be missed in a row to declare the session to be down_**):
  ```
  local.Detection_Time = remote.Detect_Mult * remote.Agreed_Trasmit_Interval
  remote.Agreed_Trasmit_Interval = max(bfd.RequiredMinRxInterval, remote.DesiredMinTxInterval)
  ```
  
  If Demand mode is not active, and a period of time equal to the Detection Time passed without receiving a BFD Control packet from the remote system, and bfd.SessionState is Init or Up, **_the session has gone down -- the local system must set bfd.SessionState to Down and bfd.LocalDiag to 1 (Control Detection Time expired)_**.
  
  In Demand mode (**_bfd.DetectMult is the number of packets that have to be missed in a row to delcared the session to be down_**):
  ```
  local.Detection_Time = bfd.DetectMult * local.Agreed_Transmit_Interval
  local.Agreed_Transmit_Interval = max(bfd.DesiredMinTxInterval, bfd.RemoteMinRxInterval)
  ```
  
  If Demand mode is active, and a period of time equal to the Detection Time passed after the initiation of a Poll Sequence (the transmission of the first BFD Control packet with the Poll bit set), the session has gone down -- **_the local system must set bfd.SessionState to Down and bfd.LocalDiag to 1 (Control Detection Time expired)_**.

#### Detecting Failures with the Echo Function

#### Receiption of BFD Control Packets

#### Transmitting BFD Control Packets

#### Receipion of BFD Echo Packets

#### Transmitting of BFD Echo Packets

#### Min Rx Interval Change
  When it is desired to change the rate at which BFD Control packets arrive from the remote system, **_bfd.RequiredMinRxInterval_** can be changed at any time to any value. The new value will be transimitted in the next outgoing Control packet, and the remote system will adjust accordingly.
  
#### Min Tx Interval Change  
  When it is desired to change the rate at which BFD Control packets are transmitted to the remote system (subject to the requirements of the neighboring system), **_bfd.DesiredMinTxInterval_** can be changed at any time to any value.
  
#### Detect Multiplier Change  
  When it is desired to change the detect multiplier, the value of bfd.DetectMult can be changed to any nonzero value. The new value will be transmitted with the next BFD Control packet, **_and the use a Poll Sequence is not necessary_**.
  
#### Enabling or Disabling The Echo Function
  If it is 























## Issues:
  * The UDP port is 3784, how to generate multiple BFD session on a single port?



**_page 31_**
