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
  
  * IP/UDP BFD Encapsulation (BFD with IP/UDP Headers): In the first method, the VCCV encapsulation of BFD includes the IP/UDP headers as defined in Section 4 of [RFC5881]. BFD Control packets are therefore transmitted in UDP with destination port 3784 and source port within the range 49152 through 65535. The IP Protocol Number and UDP Port numbers discirminate among the possible VCCV payload (i.e., )
  
  
  
  
  
## My References
  * [Pseudowire](https://baike.baidu.com/item/Pseudowire/3780134?fr=aladdin)
