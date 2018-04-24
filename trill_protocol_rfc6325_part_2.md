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
##### 4.4.2.1. TRILL Neighbor List
#### 4.4.3. TRILL MTU-Probe and TRILL Hello VLAN Tagging
#### 4.4.4. Multiple Ports on the Same Link
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
