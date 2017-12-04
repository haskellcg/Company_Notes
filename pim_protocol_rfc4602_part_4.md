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
  
######  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  **_106_**
