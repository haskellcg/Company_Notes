# Bidirectional Forwarding Detection (BFD) for the Pseudowire Virtual Circuit Connectivity Verification (VCCV)
  This document describes Connectivity Verification (CV) Types using BFD with Virtual Circuit Connectivity Verification (VCCV). VCCV provides a control channel that is associated with a pseudowire (PW), as well as the corresponding operations and management functions such as connectivity verification to be used over that control channel.

## Introduction
  BFD is used over the VCCV control channel primarily as a pseudowire fault detection mechanism, for detecting data-plane failures. Some BFD CV Types can additionally carry fault status between the endpoints of the pseudowire. Furthermore, this information can then be translated into the native Operations, Administration, and Maintenance (OAM) status codes by the native acess technologies, such as ATM, Frame Relay, or Ethernet. The specific details of such status interworking are out of the scope of this document, and are only noted here to illustrate the utility of BFD over VCCV for such purposes. Those details can be found in [OAM-MSG-MAP].
  
  The new BFD CV Types are PW demultiplexer-agnostic, and hence applicable for both MPLS and Layer Two Tunneling Protocol version 3 (L2TPv3) pseudowire demultiplexers. This document concerns itself with the BFD VCCV operation over single-segment pseudowires (SS-PWs). This specification describes procedures only for BFD asynchronous mode.
  
## BFD CV
  VCCV can support several Connectivity Verification Types. This section describes new CV Types for use when BFD is used as the VCCV payload.
  
  4 CV Types are defined for BFD:
  
  -|Fault Detection Only|Fault Detection and Status Signaling
  -|--------------------|------------------------------------
  BFD, IP/UDP Encapsulation (with IP/UDP Headers)|0x04|0x08
  BFD, PW-ACH Encapsulation (without IP/UDP Headers)|0x10|0x20
  
### BFD CV Type Operation
  When headrt-beat indication is necessary for one or more PWs, the BFD provides a means of continuous monitoring of the PW data path and , in some operational modes, propagation of PW receive and transmit defect state indications.
  
  In order to use BFD, both ends of the PW connection need to agree on the BFD CV Type to use:
  * For statically provisioned pseudowires, both ends need to be statically configured to use the same BFD CV Type (in addition to being statically configured for VCCV with the same CC Type).  
  * For dynamically established pseudowires, both ends of the PW must have signaed the exsitence of a control channel and the ability to run BFD on it
  
  Once a node has selected a valid BFD CV Type to use (either statically provisioned or selected dynamically after the node has both signaled and received signaling from its peer of these capabilities), it begins sending BFD Control packets:
  * BFD Control packets are sent on the VCCV control channel. The use of the VCCV control channel provides the context reuiqred to bind and bootstrap the BFD session, since discriminator values are not exchanged; the pseudowire demultiplexer field (e.g., MPLS PW Label or L2TPv3 Session ID) provides the context to demultiplex the first BFD Control packet, and thux single-hop BFD initialization procedures are allowed
  * A single BFD session exists per pseudowire. Both PW endpoints take the Active role sending initial BFD Control packets with a Your Discriminator field of zero, and BFD Control packet received with a Your Discriminator field of zero are associated to the BFD session bound to the PW.
  * BFD must be run in asynchronous mode
  
  The operation of BFD VCCV for PWs is therefore symmetrical. Both endpoints of the bidirectional pseudowire must sent BFD messages on the VCCV control channel.
  
  The following scenario exemlifies the operation: when **_downstream PE (D-PE) does not recieve BFD Control messages from its upstream peer PE (U-PE) during a certain number of transmission intervals (a number provisioned by the operator as "Detect Mult" or detection time multiplier [RFC5880])_**, D-PE declares that the PW in its receive direction is down. In other words, D-PE enters the "PW receive detect" state for this PW. After this calculated Detection Time, D-PE declares the session Down, and signals this to the remote end via the State (Sta) with Disgnostic code 1 (Control Detection Time Expired). 
  
  In turn, U-PE declares the PW is down in its transmit direction, setting the State to Down with Diagnostic code 3 (Neighbor signaled session down) in its control messages to D-PE. U-PE enters the "PW transmit defect" state for this PW.
  
### BFD Encapsulation  
  The VCCV message comprise a BFD Control packet encapsulated as specific by the CV Type. There two ways in which a BFD connectivity verification packet may be encapsulated over the VCCV control channel.
  
  This document defines 4 BFD CV Types, which can be grouped into two pairs of BFD CV Types from an encapsulation point of view.
  
  * IP/UDP BFD Encapsulation (BFD with IP/UDP Headers): In the first method, the VCCV encapsulation of BFD includes the IP/UDP headers as defined in Section 4 of [RFC5881]. BFD Control packets are therefore transmitted in UDP with destination port 3784 and source port within the range 49152 through 65535. The IP Protocol Number and UDP Port numbers discirminate among the possible VCCV payload (i.e., differentiate among ICMP Ping and LSP Ping defined in [RFC5085] and BFD). The IP version must match the IP version used for signaling for dynamically established pseudowires or must be configured for statically provisioned pseudowire. The source IP address is an address of the sender. The destination IP address is a (randomly chosen) IPv4 address from the range 127/8 or IPv6 address from the range 0:0:0:0:0:FFFF:127.0.0.0/104. The Time to Live/Hop Limit and Generalized TTL Security Mechanism (GTSM) procedures from Section 5 of [RFC5881] apply to this encapsulation, and hence the TTL/Hop Limit is set to 25. **_If the PW is established by signaling, then the BFD CV Type used for this encapsulation is either 0x04 or 0x08_**.
  
  * PW-ACH BFD Encapsulation (BFD without IP/UDP Headers): In the second method, a BFD Control packet is encapsulated directly in the VCCV control channel and the IP/UDP headers are omitted from the BFD encapsution. Therefore, to utilize this encapsulation, a pseudowire must use the PW Associated Channel Header (PW-ACH) Control Word format for its Control Word (CW) or L2-Specific Sublayer. In this capsulation, a "raw" BFD Control packet (i.e., a BFD COntrol packet as defined without IP/UDP headers) follows directly the PW-ACH. The PW-ACH Channel Type indicates that the Associated Channel carries "raw" BFD. The PW Associated Channel (PWAC) is defined in section 5 of RFC4385, and its Channel Type field is used to discriminate the VCCV pauload types. When VCCV carries PW-ACH-encapsulated BFD (i.e., "raw" BFD), the PW-ACH (pseudowire CW's or L2SS') Channel Type must be set to 0x0007 to include "BFD Control, PW-ACH-Encapsulated" (i.e., BFD without IP/UDP headers). This is to allow the identification of the encased BFD payload when demultiplexing the VCCV control channel. **_If the pw is established by signaling, then the BFD CV Type used for this encapsulation is either 0x10 or 0x20_**.
  
  In summary, for the IP/UDP encapsulation of BFD (BFD with IP/UDP headers), if a PW Associated Channel Header is used, the Channel Type must indicate either IPv4 (0x0021) or IPv6 (0x0057). For the PW-ACH encapsulation of BFD (BFD without IP/UDP headers), the PW Associated Channel Header must be used and the Channel Type must indicate BFD Control packet (0x0007).
  
### CV Types for BFD
  The CV Type is defined as a bitmark field used to indicate the specific CV Type or Types of VCCV packets that may be sent on the VCCV control channel.
  
  The defined values for the different BFD CV Types for MPLS and L2TPv3 PWs are:
  
  Bit (Value)|Description
  -----------|-----------
  Bit 2 (0x04)|BFD IP/UDP-encapsulated, for PW Fault Detection only
  Bit 3 (0x08)|BFD IP/UDP-encapsulated, for PW Fault Detection and AC/PW Fault Status Signaling
  Bit 4 (0x10)|BFD PW-ACH-encapsulated, for PW Fault Detection only
  Bit 5 (0x20)|BFD PW-ACH-encapsulated, for PW Fault Detection and AC/PW Fault Status Signaling
  
  It should be noted that four BFD CV Types have been defined by combination two types of encapsulation with two types of functionality.
  
  Given the bidirectional nature of BFD, before selecting a given BFD CV Type capability to be used in dynamically established pseudowires, there must be common CV Types in the VCCV capability advertised and recieved. That is, only BFD CV Types that were both advertised and received are available to be selected. Additionally, only one BFD CV Type can be used (selecting a BFD CV Type excludes all the remaining BFD CV Types).
  
  The following list enumerates rules, restrictions, and clarifications on the usage of BFD CV Types:
  * BFD CV Types used for fault detection and status signaling (i.e., CV Types 0x08 and 0x20) should not be used when a control protocol such as LDP or L2TPv3 is available that can signal the AC/PW srarys to the remote endpoint of the PW
  
  * BFD CV Types used for fault detection only (i.e., CV Types 0x04 and 0x010) can be used whether or not protocol that can signal AC/PW status is available. This includes both statically provisioned and dynamic signaled pseudowire:
    * In this case, BFD is used exclusively to delect faults on the PW; if it is desired to convey AC/PW fault status, some means other than BFD are to be used. Examples include using LDP status messages when using MPLS as a transport, and the Circuit Status Attribute Value Pair (AVP) in an L2TPv3 SLI messsage for L2TPv3
    
  * Pseudowire that do not use a CW or L2SS using the PW Associated Channel Header must not use the BFD CV Types 0x10 or 0x20 (i.e., PW-ACH encapsulation of BFD, without IP/UDP headers)
    * PWs that use a PW-ACH include CC Type 1 and MPLS CC Type 2 and 3 when using a COntrol Word. This restriction stems from the fact that the encapsulation uses the Channel Type in the PW-ACH
    * PWs that do not use PW-ACH can use the VCCV BFD encapsulation with IP/UDP headers, as the only VCCV BFD encapsulation supported. Using the IP/UDP encapsulated BFD CV Types allows for the concurrent use of other VCCV CV Types that use an encapsulation with IP headers
    
  * Only a single BFD CV Type can be selected and used. All BFD CV Types are mutually exclusive. After selecting a BFD CV Type, a node must not use any of the other three BFD CV Types
  
  * Once a PE has chosen a single BFD CV Type to use, it must continue using it until when the PW is re-signaled. In order to change the negotiated and selected BFD CV Type, the PW must be torn down and re-established
  
## Capability Selection  
  The precedence rules for selection of various CC and CV Types is clearly outlined in Section 7 in RFC5085. This section augments there rules when the BFD CV Type to use out of the four available CV Types defined is tied to multiple factors, as described in Section 3.3. 
  
  Given that BFD is bidirectional in nature, only CV Types that are both received and sent in VCCV capability signaling advertisement can be selected.
  
  When multiple BFD CV Types are advertised, and after applying the rules in Section 3.3, the set that both ends of the pesudowire have in common is determined. If the two ends have more than one BFD CV Type in common, the following list of BFD CV Types is considered in the order of the lowest list number CV Type to the highest list number CV Type, and the CV Type wioth the lowest list number is used:
  * 0x20 - BFD PW-ACH-encapsulated (without IP/UDP headers), for PW Fault Detection and AC/PW Fault Status Signaling
  * 0x10 - BFD PW-ACH-encapsulated (without IP/UDP headers), for PW Fault Detection only
  * 0x08 - BFD IP/UDP-encapsulated, for PW Fault Detection and AC/PW Fault Signaling
  * 0x04 - BFD IP/UDP-encapsulated, for PW Fault Detection only
  
## IANA Considerations  
### MPLS CV Types for the VCCV Interface Paramters Sub-TLV 
  IANA has augmented the "VCCV Connectivity Verification (CV) Types" registry in the Pseudowire Name Spaces reachable from [IANA]. These are bit field values.
  
  MPL Connectivity Verification (CV) Types:
  
  Bit (Value)|Description
  -----------|-----------
  Bit 2 (0x04)|BFD IP/UDP-encapsulated, for PW Fault Detection only
  Bit 3 (0x08)|BFD IP/UDP-encapsulated, for PW Fault Detection and AC/PW Fault Status Signaling
  Bit 4 (0x10)|BFD PW-ACH-encapsulated, for PW Fault Detection only
  Bit 5 (0x20)|BFD PW-ACH-encapsulated, for PW Fault Detection and AC/PW Fault Status Signaling
  
### PW Associated Channel Type
  The PW Associated Channel Types used by VCCV rely on repviously allocated numbers from the Pseudowire Associated Channel Types Registry [RFC4385] in the Pseudowire Name Spaces reachable from [IANA].
  
  Registry:
  
  Value|Description|TLV Follows|Reference
  -----|-----------|-----------|---------
  0x0007|BFD Control, PW-ACH encapsulation (without IP/UDP Headers)|No|[This document]
  
### L2TPv3 CV Types for the VCCV Capability AVP
  IANA has reserved the following L2TPv3 Connectivity Verification (CV) Types in the VCCV Capability AVP (Values) registry:
  
  VCCV Capability AVP (Attribute Type 96) Values:
  
  Bit (Value)|Description
  -----------|-----------
  Bit 2 (0x04)|BFD IP/UDP-encapsulated, for PW Fault Detection only
  Bit 3 (0x08)|BFD IP/UDP-encapsulated, for PW Fault Detection and AC/PW Fault Status Signaling
  Bit 4 (0x10)|BFD PW-ACH-encapsulated, for PW Fault Detection only
  Bit 5 (0x20)|BFD PW-ACH-encapsulated, for PW Fault Detection and AC/PW Fault Status Signaling
  
## Congestion Considerations  
  The congestion considerations that apply to RFC5085 apply to this mode of operation as well. 
  
  BFD as a VCCV application is required to provide details on congestion and bandwidth considerations. BFD provides with a desired minimum transmit interval and a required minimum receive interval, negotiates the transmission interval using these configurable fields, and has a packet of fixed size (setting the transmission rate).
  
  Therefore, it results in a configuration limit bandwidth utilization. As state in RFC5805, this is sufficient protection agianst congestion as long as BFD's configured maximum bit-rate is minimal compared to the bit-rate of the pseudowire the VCCV channel is associated with.
  
  If the pseudowire bit-rate can't be guaranteed to be minimal, like potentially for highly variable bit-rate and/or congestion responsive pseudowires, BFD will be required to operate using an daptive congestion control mechanism (for example, including a throttled transmission rate on "congestion detected" situations, and a slow-start after shutdown due to congestion and until basic connectivity is verified).
  
  Since the bandwidth utilized by BFD is configuration-limited, the VCCV channel must not be rate-limited below this maximum configurable bandwidth or BFD will not operate correctly.
  
  The VCCV channel could provide rate-limiting above the maximum BFD rate, to protect from a misbehaving BFD application, so that it does not conflict and can coexist. Additionally, the VCCV channel should not use any additional congestion control loop that would interface or negatively interact with that of BFD.
  
## Security Considerations
  Routers that important the additional CV Types defined herein are subject to the same security considerations as defined in RFC5085, RFC5880, RFC5881.
  
  This specification does not raise any additional security issues beyong these. The IP/UDP-encapsulated BFD makes use of the TTL/Hop Limit procedures described in Section 5 of RFC5881, including the use of the Generalized TTL Security Mechanism (GTSM) as a security mechanism.
  
## Acknowledgements

## References
  

## My References
  * [Pseudowire](https://baike.baidu.com/item/Pseudowire/3780134?fr=aladdin)
