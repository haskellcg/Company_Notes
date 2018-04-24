### 4.3. Inter-RBridge Link MTU Size
  There are two reaons why it is important to known what size of frame each inter-RBridge link in the campus can support:
  * RBridge RB1 must known the size of link state information messages it can generate that will be guaranteed to be forwardable across all inter-RBridge links in the campus.
  * If traffic engineering tools knowwn which links support larger than minimally acceptable data packet size, paths can be computed that can support large data packets.

#### 4.3.1. Determining Campus-Wide TRILL IS-IS MTU Size
  In a stable campus, there must ultimately be agreement among all RBridge on the value of "Sz", the minimum acceptable inter-RBridge link size for the campus, for the proper operation of TRILL IS-IS. All RBridges must format their link state information messages to be in chunks of size no larger than what they believe Sz to be. Also, every RBridge RB1 should test each of its RBridge adjacencies, say, to RB2, tn ensure that the RB1-RB2 link can forward packets of at least size Sz.

  Sz has no direct on end stations and is not directly related to any end-station-to-end-station "path MTU". Methods of using Sz or any link MTU information gathered by TRILL IS-IS in the traffic engineering of routes or the determination of any oath MTU is beyond the scope of this document. Native frames that, after TRILL encapsulation, exceed the MTU of a link on which  they are sent will generally be discarded.

  Sz is determined by having each RBridge (optionally) advertise, in its LSP, its assumption of the value of the campus-wide sz. This LSP element is known in IS-IS as the originatingLSPBufferSIze, TLV #14. The default and minimum value for Sz, and the implicitly advertised value of Sz if the TLV is adsent, is 1470 octets. This length (which is also the maximum size of a TRILL-Hello) was chosen to make it extremely unlikely that a TRILL control frame, even with reasonable additional headers, tags, and/or encapsulation, would encounter MTU problems on an inter-RBridge link.

  The campus-wide value of Sz is the smallest value of Sz advertised by any RBridge.

#### 4.3.2. Testing Link MTU Size
  There are two new TRILL IS-IS message types for use between pairs of RBrudge neighbors to test the bidirectional packet size capacity of their connection. Thes messages are:
  * \-\- MTU-probe
  * \-\- MTU-ack

  Both the MTU-probe and the MTU-ack are padded to the size being tested.

  Sending of MTU-probes is optional; however, an RBridge RB2 that receives an MTU-probe from RB1 must respond with an MTU-ack padded to the same size as the MTU-probe. The MTU-probe may be multicast to All-RBridges, or unicast to a specific RBridge. The MTU-ack is normally unicast to the source of the MTU-probe wo which it responds but may be multicast to the All-RBridge.

  If RB1 fails to receive an MTU-ack to a probe of size X from RB2 after k tries (where k is a configurable parameter whose default is 3), then RB1 assumes the RB1-RB2 link cannot support isze X. If X is not greater than Sz, then RB1 sets the "failed minimum MTU test" flag for RB2 in RB1's hello. If size X succeeds, and X > Sz, then RB1 advertises the largest tested X for each adjacency in the TRILL Hellos RB1 sends on that link, and RB1 may advertise X as attributes of the link to RB2 in RB1's LSP.

### 4.4. TRILL-Hello Protocol
  The TRILL-Hello Protocol is a little different from the Layer 3 IS-IS LAN Hello Protocol and uses a new type of IS-IS messages knwon as a TRILL-Hello.

#### 4.4.1. TRILL-Hello Rationale
  The reason for definninf this new type of link in TRILL is that in Layer 3 IS-IS, the LAN Hello protocol may elect multiple Desinated Routers (DRs) since, when choosing a DR, routers ignore other routers with whom they do not have 2-way connectivity. Also, Layer 3 IS-IS LAN hellos are padded, to avoid forming adjacencies between neighbors that can't speak the maximum-sized packet to each other. This means, in Layer 3 IS-IS, the neighbors that have connectivity to each other, but with an MTU on that connection less than what they percieve as maximum sized packets, will not see each other's hellos. The result is that routers might from cliques, resulting in the link turnning into multiple pseudonodes.

  This behavior is fine for Layer 3, but not for Layer 2, where loops may form if there are multiple DRBs. Thereform, the TRILL-Hello protocol is a little different from Layer 3 IS-IS's LAN Hello protocol.

  One other issue with TRILL-Hellos is to ensure that subnets of the information can appear in any single message, and be processable, in the spirit of IS-IS LSPs and CSNPs. TRILL-Hello frames, even though they are not padded, can become very large. An example where this might be the case is when some sort of backbone technology interconnects hundreds of TRILL sites over what would appear to TRILL to be a giant Ethernet, where the RBridge connected to that cloud will perceive that backbone to be a single link with hundreds of neighbors.

  In TRILL (Unlike in Layer 3 IS-IS), the DRB is selected based solely on priority and MAC address. In other words, if RB2 receives a TRILL-Hello from RB1 with higher (priority, MAC), RB2 defers to RB1 as DRB, regardless of whether RB1 lists RB2 in RB1's TRILL-Hello.

  Although the neighbor list in a TRILL-Hello does not influence the DRB election, it does determine what is announced in LSPs. RB1 only reports links to Rbridges with which it has two way connectivity. If RB1 is the DRB on a link, and for whatever reason (MTU mismatch, or one-way connectivity) RB1 and RB2 to not have two-way connectivity, then RB2 does not report a link to RB1 (or the psedunode), and RB1 (or RB1 on behalf of the pseudonode) does not report a link to RB2.

#### 4.4.2. TRILL-Hello Contents and Timing
  The TRILL-Hello has a new IS-IS message type. It starts with the same fixed header as an IS-IS LAN Hello, which includes the 7-bit priority for the issuing RBridge to be DRB on that link. TRILL-Hellos are sent with the same timming as IS-IS LAN Hellos.

  TRILL-Hello messages, including their Outer.MacDA and Outer.MacSA, but excluding any other Outer.VLAN or other tags, must not exceed 1470 octets in length and should be padded. The folloing information appear in every TRILL-Hello. References to "TLV" may actually be a "sub-TLV" as specified in separate documents [RFC6165] \[RFC6326\].

  1. The VLAN ID of the Designated VLAN for the link.
  1. A copy of the Outer.VLAN ID with which the Hello was tagged on sending.
  1. A 16-bit port ID assigned by the sending RBridge to the port the TRILL-Hello is sent on such that no two ports of that RBridge have the same port ID.
  1. A nickname of the sending RBridge
  1. Two flags as follows:
      1. A flag that, if set, indicates that the sender has detected VLAN mapping on the link, within the past 2 of its Holding Times.
      1. A flag that, if set, indicates that the sender believes it is appointed forwarder for the VLANand port on which the TRILL Hello was sent.

  The Following information may appear:
  1. The set of VLANs for which end-station service is enabled on the port.
  1. Several flags as follows:
      1. A flag that, if set, indicates that the sender's port was configured as an access port
      1. A flag that, if set, indicates that the sender's port was configured as a trunk port
      1. A bypass pseudonode flag, as described below in this section
  1. If the sender is the DRB, the RBridges (excluding itself) that it appoints as forwarder for link and the VLANs for which it appoints them. As describe below, this TLV is designed so that not all the appointment information need be included in each Hello. Its absence means that appointed forwarders should continues as previously assigned.
  1. The TRILL neighbor list. This is a new TLV, not the same as the IS-IS Neighbor TLV, in order to accomodate fragmentation and reporting MTU on the link

  The Appointed Forwarders TLV specifies a range of VLANs and, within that range, sepcifies within RBridge, if any, other than the DRB, is appointed forwarder for the VLANs in that range [RFC6326]. Appointing an RBridge as forwarder on a port for a VLAN that is not enable on that port has no effect.

  It is anticipated that many links between RBridges will be point-to-point, in which case using a pseudonode merely adds to the complexity. If the DRB specifie the bypass pseudonode bit in its TRILL-Hellos, the RBridges on the link just report their adjacencies as point-to-point. This has no effect on how LSPs are flooded on a link. It only effects what LSPs are generated.

  For example, if RB1 and RB2 are the only RBridges on the link and RB1 is the DRB, then if RB1 creates a pseudonode that is used, there are 3 LSPs: for, say, RB1.25 (the pseudonode), RB1, and RB2, where RB1.25 reports connectivity to RB1 and RB2, and Rb1 and RB2 each just say they are connected to RB1.25. Whereas if DRB RB1 sets the bypass pseudonode bit in its Hellos, then there will be only 2 LSPs: RB1 and RB2 each reporting connectivity to each other.

  A DRB should set the bypass pseudonode bit for its link unless, for a particular link, it has seen at least two simultaneous adjacencies on the link at somee point since it last rebooted.

##### 4.4.2.1. TRILL Neighbor List
  The new TRILL Neighbor TLV includes following information for each neighbor is lists:
  1. The neighbor's MAC address.
  1. MTU size to this neighbor as a 2-octet unsigned integer in units of 4-octets chunks. The value zero indicates that the MTU is untested.
  1. A flag for "failed minimum MTU test".

  To allow partial reporting of neighbors, the neighbor IDs must be sorted by ID. If a set of neighbors {X1, X2, X3, ..., Xn} is reported in RB1's Hello, then X1 < X2 < X3, ..., Xn. If RBridge RB2's ID is between X1 and Xn, and does not in RB1's Hello, then RB2 knowns that RB1 has not heard RB2's Hello.

  Additonally there are two overall TRILL Neighbor List TLV flags: "the smallest ID I reported in this hello is the smallest ID of any neighbor" and "the largest ID I reported in this hello is the largest ID of any neighbor". If all the neighbors fit in RB1's TRILL-Hello, both flags will be set.

  If RB1 reports {X1, ..., Xn} in its Hello, with the "smallest" flag set, and RB2's ID is smaller than X1, then RB2 knowns that RB1 was not heard RB2's Hello. Similarly, if RB2's ID is larger than Xn and the "largest" flag is set, then RB2 knwons that RB1 has not heard RB2's Hello.

  To Ensurn that any Rbridge RB2 can definitively determine whether RB1 can hear RB2, RB1's neighbor list must eventually cover every possible range of IDs, that is, within a period that depends on RB1's policy and not necessarily within any specific period such as the holding time. In other words, if X1 is the smallest ID reported in one of RB1's neighbor lists, and the "smallest" flag is not set, then X1 must also appears as the largest ID reported in a different TRILL-Hello Neighbor list. Or, fragments may overlap, as long as there is no gap, such that some range, say between Xi and Xi, never appears in any fragment.

#### 4.4.3. TRILL MTU-Probe and TRILL Hello VLAN Tagging
  The MTU-probe mechanism is designed to determine the MTU for transmissions between RBridges. MTU-probe and probe acknownledgements are only sent on the Designated VLAN.

  An RBridge RBn maintains for each port the same VLAN information as a customer IEEE [802.1Q-2005] bridge, including the set of VLANs enabled for output through that port. In addition, RBn maintains the following TRILL-specific VLAN parameters per port:
  * Desired Designated VLAN: TODO
  * Designate VLAN: TODO
  * Announcing VLANs set. TODO
  * Forwarding VLANs set: TODO

  On each of its ports that is not configured to use P2P Hellos, an RBridge sends TRILL-Hellos Outer.VLAN tagged with each VLAN in a set of VLANs. This set depends on the RBridge's DRB status and the above VLAN parameters. Rbridges send TRILL Hellos Outer.VLAN tagged with the Designated VLAN, unless that VLAN is not enabled on that port. In addition, the DRB sends TRILL Hellos Outer.VLAN tagged with  each enabled VLAN in ite announcing VLAN set. All non-DRB RBridge send TRILL-Hellos Outer.VLAN tagged with all enabled VLANs that are in the intersection of their Forwarding VLANs set and their Announcing VLANs set. More symbolically, TRILL-Hello frames, when sent, are sent as follows:
  ```
  If sender is DRB:
    intersection (Enabled VLANs,
                  union(Designated VLAN, Announcing VLANs))
  If sender is not DRB:
    intersection (Enabled VLANs,
                  union(Designated VLAN,
                        intersection(Forwarding VLANs, Announcing VLANs)))
  ```

  Configuring the Announcing VLANs set to be null minimizes the number of TRILL-Hellos. In that case, TRILL Hellos are only tagged with the Designated VLAN. Greate care should be taken in configuring an RBridge to not send TRILL Hellos on any VLAN where that RBridge is appointed forwarder as, under some circumstances, failure to send such Hellos can lead to loops.

  The number of TRILL-Hellos is maximized, within this specification, by configuring the Annoucning VLANs set to be the set of all enabled VLAN IDs, which is the default. In that case, the DRB will send TRILL-Hello frames tagged with all its enabled VLAN tags; in addition, any non-DRB RBridge RBn will send TRILL-Hello frames tagged with the Designated VLAN, if enabled, and tagged with all VLANs for which RBn is an appointed forwarder. (it is possible to send even more TRILL-Hellos. In particular, non-DRB RBridges could send TRILL-Hellos on enabled VLANs for which they are not an appointed forwarder and which are not the Designated VLAN. This would cause no harm other than a further communications and processing burden.)

  When an RBridge port comes up, until it has heard a TRILL-Hello from a Higher priority Rbridge, it considers itself to be DRB on that port and sends TRILL-Hellos on that basis. Similarly, even if it has at some time recognized some other Rbridge one the link as DRB, if it receives no TRILL-Hellos on that port from an Rbridge with highwe priority as DRB for a long enough time, as specified by IS-IS, it will revert to believing itself DRB.

#### 4.4.4. Multiple Ports on the Same Link
  It is possible for an Rbridge RB1 to have multiple ports to the same link. It is important for RB1 to recognize which of its ports are on the same link, so, for instance, if RB1 is appointed  forwarder for VLAN A, RB1 knowns that only one of its ports acts as appointed forwarder for VLAN A on that link.

  RB1 detects this condition based on receiving TRILL-Hello messages with the same IS-IS pseudonode ID on multiple ports. RB1 might have one set of ports, say, {p1, p2, p3} on one link, and another set of ports {p4, p5} on a second link, and yet other ports, say, p6, p7, p8, that are each on distinct link. Let us call a set of ports on the same link a "port group".

  If RB1 detects that a set of ports, say, {p1, p2, p3}, is a port group on a link, then RB1 must ensure that **it does not cause loops** when it encapsulates and decapsulates traffic from/to that link. If RB1 is appointed forwarder for VLAN A on that Ethernet link, RB1 must encapsulate/decapsulate VLAN A on only one of the ports. However, if RB1 is appointed forwrader for more than one VLAN, RB1 may choose to load split among its ports, using one port for some set of VLANs, and another port for a disjoint set of VLANs.

  If RB1 detects VLAN mapping occuring, then RB1 must not load split as appointed forwarder, and instead must act as appointd VLAN forwarder on that link on only one of its ports in the port group.

  When forwarding TRILL-encapsulated multi-destination frames to/from a link on which RB1 has a port group, RB1 may choose to load split among its ports, provided that it does not duplicate frames, and procided that it keeps frames for the same flow on the same port. If RB1's neighbor on that link, RB2, accepts multi-destination frames on that tree on that link from RB1, RB2 must accept the frame from any of RB2's adjancencies to RB1 on that link.

  If an RBridge has more than one port connected to a link and those ports have the same MAC address, they can be distinguished by the port ID contained in TRILL-Hellos.

#### 4.4.5. VLAN Mapping within a Link
### 4.5. Distribution Trees
#### 4.5.1. Distribution Tree Calculation
#### 4.5.2. Multi-Destination Frame Checks
#### 4.5.3. Pruning the Distribution Tree
#### 4.5.4. Tree Distribution Optimization
#### 4.5.5. Forwarding Using a Distribution Tree
### 4.6. Frame Processing Behavior
#### 4.6.1. Receipt of a Native Frame
##### 4.6.1.1. Native Unicast Case
##### 4.6.1.2. Native Multicast and Broadcast Frames
#### 4.6.2. Receipt of a TRILL Frame
##### 4.6.2.1. TRILL Control Frames
##### 4.6.2.2. TRILL ESADI Frames
##### 4.6.2.3. TRILL Data Frames
##### 4.6.2.4. Known Unicast TRILL Data Frames
##### 4.6.2.5. Multi-Destination TRILL Data Frames
#### 4.6.3. Receipt of a Layer 2 Control Frame
### 4.7. IGMP, MLD, MRD Learning
### 4.8. End-Station Address Learning
#### 4.8.1. Learning End-Station Addresses
#### 4.8.2. Learning Confidence Level Retionale
#### 4.8.3. Forgetting End-Station Addresses
#### 4.8.4. Shared VLAN Learning
### 4.9. RBridge Ports
#### 4.9.1. RBridge Port Configuration
#### 4.9.2. RBridge Port Structure
#### 4.9.3. BPDU Handling
##### 4.9.3.1. Receipt of BPDUs
##### 4.9.3.2. Root Bridge Changes
##### 4.9.3.3. Transmission of BPDUs
#### 4.9.4. Dynamic VLAN Registration
