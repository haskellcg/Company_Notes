#### PIM Bootstrap and RP Discovery
  For correct operation, every PIM router within a PIM domain must be able to **map a particular multicast group address to the same RP**. A notable exception to this is where a PIM domain is broken up into **multiple administrative scope regions**; these are regions where a border has been configured so that a range of multicast groups will not be forwarded across that border.
  
  The modified criteria for admin-scoped regions are that the region is convex with respect to forwarding based on the MRIB, and that all PIM routers within the scope region map scoped groups to the same RP within that region.
  
  This specification does not mandate the use of a single machanism to provide routers with the information to perform the group-to-RP mapping. Currently four machanism are possible, and all four have associated problems:
  * Static Configuration: A PIM routers must support a static configuration a group-to-RP mappings.
  * Embedded-RP: Emdedded-RP defines an address allocation policy in which the address of the Rendezvous Point (RP) is encoded in an IPv6 multicast group address.
  * Cisco's Auto-RP: Auto-RP uses a PIM Dense-Mode multicast group to annouce group-to-RP mapping from a central location. This mechanism is not useful if PIM Dense-Mode is not being run in parallel with PIM Sparse-Mode, and was only intended for use with PIM SM Version 1.
  * BootStrap Router (BSR): Any router in the domain that is configured to be a possible RP reports its candidacy to the BSR, and then a domain-wide flooding mechanism distributes the BSR's chosen set of RPs throughout the domain.

  As far as PIM-SM is concerned, the only important requirement is that **all routers in the domain (or admin scope sone for scoped regions) receive the same set of group-range-to-RP mappings**.
  
###### Group-to-RP Mapping  
  Using one of the mechanisms described above, a PIM router receives one or more possible group-range-to-RP mappings. Each mapping specifies a range of multicast groups should be mapped. Each mapping may also have an asssociate priority.
  
  The algorithm for performing the **group-to-RP mapping** is as follows:
  1. Perform longest match on group-range to obtain a list of RPs.
  1. From this list of matching RPs, find the one with highest priority. Eliminate any RPs from the list that have lower priorities.
  1. If only one RP remains in the list, use that RP
  1. If multiple RPs are in the list, us ethe PIM hash function to choose one.
  
  Thus, if two or more group-range-to-RP mapping cover a particular group, the one with **the longest mask** is the mapping to use. {the highest priority, hash function}.
  
  The algorithms is invoked by DR when it needs to determine an RP for a given group, e.g., upon reception of packet or IGMP/MLD membership indication for a group for which the DR does not know the RP. It is invoked by any router that has (\*, \*, RP) state when a packet is received for which there is no corresponding (S, G) or (\*, G) entry. Furthermore, the mapping function is invoked by all routers upon receiving a (\*, G) or (\*, \*, RP) Join/Prune message.
  
  Note that if the set of possible group-range-to-RP mapping changes, each router will need to check whether any existing groups are affected. This may, for example, cause a DR or acting DR to re-join a group, or cause it to restart register encapsulation to the new RP.
  
###### Hash Function
  The hash function is used by all routers within a domain, to map a group to one of the RPs from the matching set of group-range-to-RP mappings (this set all have the same longest mask length and same highest priority).
  
  The algorrithms takes as input the group address, and the addresses of the candidate RPs from the mappings, and **gives as output one RP address to be used**.
  
  The protocol requires that **all routers hash to the same RP within a domain (except fro transients)**. The following hash function must be used in each router:  
  * For RP addresses in the macthing group-range-to-RP mappings, compute a value:
  ```
  Value(G, M, C(i)) = (1103515245 * ((1103515245 * (G & M) + 12345) XOR C(i)) + 12345) mod 2^31
  
  C(i): the RP address
  M: hash-mask, hash-mask allows a small number of consecutive groups to always hash to the sam RP
  ```
  For instance, hierarchically-encoded data can be sent on consecutive group addresses to get the same delay and fate-sharing characteristics.
  
  * The candidate RP with the highest resulting hash value is then the RP chosen by this Hash Function. If more than one RP has the same highest hash value, the RP with the highest IP address is chosen.
  
  
#### Source-Specific Multicast  
  The Source-Specific Multicast (SSM) service model can be implemented with a strict subset of the PIM-SM protocol machanisms. **Both regular IP multicast and SSM semantics can coexist on a single router, and both can be implemented using the PIM-SM protocol**.
  
  A range of multicast addresses, currently 232.0.0.0/8 in IPv4 and FF3x::/32 for IPv6, is reserved for SSM, and the choice of semantics is determined by the multicast group address in both data packets and PIM messages.
  
###### Protocol Modifications for SSM Destination Addresses
  The following rules override the normal PIM-SM behavior for a multicast address G in the SSM range:
  * A router must not send a (\*, G) Join/Prune message for any reason
  * A router must not send an (S, G, rpt) Join/Prune message for any reason
  * A router must not send a Register message for any packet that is destined to an SSM address
  * A router must not forward packets based on (\*, G) or (S, G, rpt) state
  * A router acting as an RP must not forward any Register-encapsulated packet that has an SSM destination address
  
  Addinationally:
  * A router MAY be configured to advertise itself as a Candidate RP for an SSM address. If so, it should respond with a Register-Stop message to any Register message containning a packet destined for an SSM address
  * A router MAY optiomize out the creation and maintenance of (S, G, rpt) and (\*, G) state for SSM destination addresses -- this state is not needed for SSM packets
  
###### PIM-SSM-Only Routers
  An implementer may choose to implement only the subset of PIM Sparse-Mode that provides SSM forwarding semantics.
  
  A PIM-SSM-Only router MUST implement the following portions of this specifications:
  * Upstream (S, G) state machine
  * Downstream (S, G) state machine
  * (S, G) Assert state machine
  * Hello messages, neighbor discovery, and DR election
  * Packet forwarding rules
  
  A PIM-SSM-Only routers does not need to implement the following protocol elements:
  * (\*, G), (S, G, rpt), (\*, \*, RP) Downstream state machine
  * (\*, G), (S, G, rpt), (\*, \*, RP) Upstream state machine
  * (\*, G) Assert state machine
  * Bootstrap RP election
  * Keepalive Timer
  * SPTbit
  
  The Keepalive Timer should be treated as always running, and SPTbit should be treated as always being set for an SSM address.
  
  The Packet forwarding rules can be simplified in a PIM-SSM-Only router:
  ```c++
  if (iif == RPF_interface(S) AND UpstreamJPState(S, G) == Joined){
      oiflist = inherited_olist(S, G)
  } else if (iif is in inherited_olist(S, G)){
      send Assert(S, G) on iif
  }
  
  oiflist = oiflist (-) iif
  forward packet on all interfaces in oiflist
  ```
  
#### PIM Packet Formats  
  **All PIM protocol messages have IP protocol number 103**.
  
  PIM messages are either unicast (e.g., Register and Register-Stop) or multicast with TTL 1 to the 'ALL-PIM-ROUTERS' group (e.g., Join/Prune, Assert, etc.). 
  
  **The source address used for unicast message is a domain-wide reachable address**. 
  
  **The source address used for multicast messages is the link-local address of the interface on which the message is being sent**.
  
  **The IPv4 'ALL-PIM-ROUTERS' group is '224.0.0.13'. The IPv6 'ALL-PIM-ROUTERS' group is 'FF02::d'.**
  
  The PIM header common to all PIM message is:
  
  PIM Ver|Type|Reserved|Checksum
  -------|----|--------|--------
  4|4|8|16
  
  * PIM Ver: PIM Version number is 2
  * Type: Types for specific PIM messages are:
    * 0: Hello, Multicast to ALL-PIM-ROUTERS
    * 1: Register, Unicast to RP
    * 2: Register-Stop, Unicast to source of Register packet
    * 3: Join/Prune, Multicast to ALL-PIM-ROUTERS
    * 4: Bootstrap, Multicast to ALL-PIM-ROUTERS    
    * 5: Assert, Multicast to ALL-PIM-ROUTERS
    * 6: Graft (used in PIM-DM only), Unicast to RPF'(S)
    * 7: Graft-Ack (used in PIM-DM only), Unicast to source of Graft packet
    * 8: Candidate-RP-Advertisement, Unicast to Domain's BSR
  * Reserved: Set to zero on transmission, Ignored upon receipt
  * Checksum: The checksum is a standard IP checksum, the 16-bit one's complement of the one's complement sum of the entire PIM message, excluding the "Multicast data packet" section of the Register message. **For computing the check sum, the checksum field is zeroed. If the packet's length is not an integral number of 16-bit words, the packet is padded with a trailing byte of zero before performing the checksum.**
  
###### Encoded Source And Group Address Formats  
  **Encoded-Unicast Address:**
  
  Addr Family|Encoding Type|Unicast Address
  -----------|-------------|---------------
  8|8|16
  
  * Addr Family: The PIM address family of the 'Unicast Address' field of this address. Values 0-127 are as assigned by the IANA for Internet Address Families. Values 128-250 are reserved to be assigned by the IANA for PIM-specific Address Families. Values 251-255 are desinated for private use.
  * Encoding Type: The type of encoding used within a specific Address Family. The value '0' is reserved for this field and represents the native encoding of the Address Family.
  * Unicast Address: The unicast address as represented by the given Address Family and Encoding Type.
  
  **Encoded-Group Address:**
  
  Addr Family|Encoding Type|B|Reserved|Z|Mask Len|Group multicast Address
  -----------|-------------|-|--------|-|--------|-----------------------
  8|8|1|6|1|8|32
  
  * Addr Family
  * Encoding Type
  * [B]idirectional PIM: Indicates the group range should use Bidirectional PIM. For PIM-SM defined in this specification, this bit MUST be zero
  * Reserved: Transmitted as zero. Ignore upon receipt.
  * Admin Scope [Z]one: Indicates the group range is an admin scope zone. This is used in the Bootstrap Router Mechanism only. For othr purposes, this bit is set to zero and ignored on receipt.
  * Mask Len: The Mask Length fiels is 8 bits. The value is the number of contiguous one bits that are left justified and used as a mask; when combined with the group address, it describes a range of groups.
  * Group multicast Address: Contains the group address
  
  **Encoded-Source Address**
  
  Addr Family|Encoding Type|Reserved|S|W|R|Mask Len|Source Address
  -----------|-------------|--------|-|-|-|--------|--------------
  8|8|5|1|1|1|8|32
  
  * Addr Family
  * Encoding Type
  * Reserved: Transimitted as zero, ignored on receipt
  * S: The Sparse bit is a 1-bit value, set to 1 for PIM-SM. It is used for PIM Version 1 compatibility.
  * W: The WC (or WildCard) bit is a 1-bit value for use with PIM Join/Prune messages
  * R: The RPT (or Rendezvous Point Tree) bit is a 1-bit value for use with PIM Join/Prune messages. If the WC bit is 1, the RPT bit MUST be 1.
  * Mask Len: The mask length field is 8 bits. The value is the number of contiguous one bits left justified used as a mask which, combined with the Source Address, describes a source subnet.
  * Source Address
  
###### Hello Message Format  
  It is sent periodically by routers on all interfaces:
  
  PIM Ver|Type|Reserved|Checksum|Option Type|Option Length|Option Value|....|Option Type|Option Length|Option Value
  -------|----|--------|--------|----------|------------|-----------|----|----------|------------|-----------
  4|4|8|16|16|16|32|....|16|16|32
  
  * PIM Ver, Type, Reserved, Checksum
  * Option Type: The type of the option given in the following OptionValue field
  * Option Length: The length of the OptionValue field in bytes
  * Option Value: A variable length field, carrying the value of the option
  
  **The option fields may contain the following values**:
  
  Option Type|Option Length|Option Value
  -----------|-------------|------------
  1|2|HoldTime
  2|4|T[1] + Propagation_Delay[15] + Override_interval[16]
  19|4|DR Priority
  20|4|Generation ID
  24|Variable|Secondary Address 1-N[Encoded-Unicast format][32]
  
  * Option Type 3-16: reserved to be defined in future version of this document
  * Option Type 18: deprecated and should not be used
  * Option Type 17-65000 are assigned by the IANA
  * Option Type 65001-65535 are reserved for Private Use
  
###### Register Message Format  
  A Register message is sent by **the DR or a PMBR to the RP when a multicast packet needs to be transmitted on the RP-tree**. The IP source address is set to the address of DR, the destination address to the RP's address. The IP TTL of the PIM packet is the system's normal unicast TTL.
  
  PIM Ver|Type|Reserved|Checksum|B|N|Reserved2|Multicast Data Packet
  -------|----|--------|--------|-|-|---------|---------------------
  4|4|8|16|1|1|30|32
  
  * PIM Ver, Type, Reserved, Checksum: Note that in order to **reduce encapsulation overhead**, the checksum for Register is done only on the first 8 bytes of the packet, including the PIM header and the next 4 bytes, excluding the data packet portion.
  * B: The Border bit. If the router is a DR for a source that it is directly connected to, it sets the B bit to 0. If the router is a PMBR for a source in a directly connected cloud, it sets the B bit to 1.
  * N: The Null-Register bit. Set to 1 by the DR that is probing the RP before expiring its local Register-Suppression Timer. Set to 0 otherwise.
  * Reserved2: Transmitted as zero, ignored on receipt.
  * Multicast Data Packet: The original packet sent by the source. Note that the TTL of the original packet is decremented before encapsulation, just like any other packet that is forwarded.
  
###### Register-Stop Message Format  
  A Register-Stop is unicast from the RP to the sender of the Register message. The IP source address is the address to which the register was addressed. The IP destination address is the source address of the register message.
  
  PIM Ver|Type|Reserved|Checksum|Group Address (Encoded-Group Format)|Source Address (Encoded-Unicast Format)
  -------|----|--------|--------|------------------------------------|---------------------------------------
  4|4|8|16|32|32
  
  * PIM Ver, Type, Reserved, Checksum
  * Group Address: The group address from the multicast data packet in the Register. Note that for Register-Stops the Mask Len field contains the full address length * 8, if the message is sent for a single group
  * Source Address: The host address of the source from the multicast data packet in the register. A special wild card value consisting of an address field of all zeros can be used to indicate any source.
  
###### Join/Prune Message Format  
   A Join/Prune Message is sent by routers towards upstream sources and RPs. **Joins are sent to build shared trees (RP trees) or source trees (SPT)**. Prunes are sent to prune source trees when members leave groups as well as sources that do not use the shared tree.
   
   PIM Ver|Type|Reserved|Checksum|Upstream Neighbor Address|Reserved|Num Groups|Holdtime
   -------|----|--------|--------|-------------------------|--------|----------|--------
   4|4|8|16|32|8|8|16
   
   Multicast Group Address 1|Number of Joined Sources|Number of Prune Sources|Joined Source Address 1|...|Joined Source Address n|Prune Source Address 1|...|Prune Address n|......|Multicast Group Address n...
   -------------------------|------------------------|-----------------------|-----------------------|---|-|-|-|-|-|-
   32|16|16|32|...|32|32|...|32|......|32...
   
   * PIM Ver, Type, Reserved, Checksum
   * Unicast Upstream Neighbor Address: The address of the upstream neighbor that is the target of the message. The format for this address is given in the Encoded-Unicast Address. For IPv6 the source address used for multicast message is the link-local address of the interface on which the message is being sent. For IPv4, the source address is the primary address associate with that interface.
   * Reserved: Transmitted as zero, ignored on receipt
   * Holdtime: The amount of time a receiver must keep the Join/Prune state alive, in seconds. If the Holdtime is set to '0xffff', the receiver of this message should hold the state until canceled by the appropriate canceling Join/Prune message, or timed out according to local policy. **Note that the HoldTime must be larger than the J/P_Override_Interval(I)**.
   * Number of Groups: The number of multicast group sets contained in the message.
   * Multicast group address
   * Number of Joined Sources
   * Joined Source Address 1 .. n: This list contains the sources for a given group that the sending router will forward multicast datagrams from if received on the interface on which the Join/Prune message is sent.
   * Number of Pruned Sources
   * Pruned Source Address 1 .. n: This list contains the sources for a given group that the sending router dose not want to forward multicast datagrams from when received on the interface on which the Join/Prune message is sent.
  
  **There are two valid group set types**:
  * Wildcard Group Set: The beginning of the multicast address range in the group address field and the prefix length of the multicast address range in the mask length field of the Multicast Group Address (i.e., '224.0.0.0/4' for IPv4 or 'ff00::/8' for IPv6). **Each Join/Prune message should contain at most one wildcard group set**. [(\*, \*, RP)]
  * Group-Specific Set: A Group-Specific Set is represented by a valid IP multicast address in the group address field and the full length of the IP address in the mask length field of the Multicast Group Address. [(\*, G), (S, G, rpt), (S, G)]
  
  There are a number of combinations that have a valid interpretation but that are not generated by the protocol as described in this specification:
  * Combining a (\*, G) Join and a (S, G, rpt) Join/Prune entry
  * As Join/Prune messages are targeted to a single PIM neighbor, including both a (S, G) Join and (S, G, rpt) Prune in the same message, **The (S, G) Join informs the neighbor that the sender wishes to receive the particular source on the shortest path tree. It is therefore unnecessary for the router to say that it no longer wishes to receive it on the shared tree.** However, there is a valid interpretation for this combination of entries. A downstream router my have to instruct its upstream only to start forwarding a specific source once it has started receiving the source on the shotest path tree  
  * The combination of a (S, G) Prune and a (S, G, rpt) Join could possibly be used by router to switch from recieving a particular source on the shortest-path tree back to receiving it on the shared tree.
  
  -|Join(\*, G)|Prune(\*, G)|Join(S, G, rpt)|Prune(S, G, rpt)|Join(S, G)|Prune(S, G)
  -------------|-----------|------------|---------------|----------------|----------|-----------
  Join(\*, G)|-|no|?|yes|yes|yes
  Prune(\*, G)|no|-|?|?|yes|yes
  Join(S, G, rpt)|?|?|-|no|yes|?
  Prune(S, G, rpt)|yes|?|no|-|yes|?
  Join(S, G)|yes|yes|yes|yes|-|no
  Prune(S, G)|yes|yes|?|?|no|-
  
  -|Join(\*, \*, RP)|Prune(\*, \*, RP)|all others
  -------------|----------------|-----------------|----------
  Join(\*, \*, RP)|-|no|yes
  Prune(\*, \*, RP)|no|-|yes
  all others|yes|yes|see above
  
  **Group Set Fragmentation**
  
  When building a Join/Prune for a particular neighbor, a router should try to include in the message as much of the information it needs to convey to the neighbot as possible.
  
  There is an exeption with group sets that contain a (\*, G) Joined source list entry. The group set expresses the router's interest in receiving all traffic for the specified group on the shared tree, and it MUST include an (S, G, rpt) Prune source list entry for every source that the router does not wish to receive. This list of (S, G, rpt) Prune source-list entries MUST not be split in multiple messages.
  
#### Assert Message Format  
  The Assert message is used to resolve forwarder conflicts between routers on a link. It is sent when a router receives a multicast data packet on an interface on which the router would normally have forwarded that packet.
  
  PIM Ver|Type|Reserved|Checksum|Group Address|Source Address|R|Metric Preference|Metric
  -------|----|--------|--------|-------------|--------------|-|-----------------|------
  4|4|8|16|32|32|1|31|32
  
  * PIM Version, Type, Reserved, Checksum
  * Group Address: The group address for which the router wishes to resolve the forwarding conflict.
  * Source Address: Source address for which the router wishes to resolve the forwarding conflict. The source address May be set to zero for (\*, G) asserts.
  * R: RPT-bit is a 1-bit value. **The RPT-bit is set to 1 for Assert(\*, G) messages and 0 for Assert(S, G) messages**.
  * Metric Preference: Preference value assigned to the unicast routing protocol that provided the route to the multicast source or Rendezvous-Point.
  * Metric: The unicast routing table metric associate with the route used to reach the multicast source or Rendezvous-Point. The metric is in units applicable to the unicast routing protocol used.

  Assert messages:
  * Assert(S, G): Source-specific asserts are sent by routers forwarding a specific source on the shortest-path tree (SPTbit is TRUE). (S, G) Asserts have the Group-Address field set to the group G and the Source-Address field set to the source S. **The RPTbit is set to 0, the Metric_Preference is set to MRIB.pref(S) and the Metric is set to MRIB.metric(S)**.   
  * Assert(\*, G): For data-triggered Asserts, the Source-Address field MAY be set to the IP source address of the data packet that triggered the Assert and is set to zero otherwise. The RPT-bit is set to 1, the Metric-Preference is set to MRIB.pref(RP(G)), and the Metric is set to MRIB.metric(RP(G))


