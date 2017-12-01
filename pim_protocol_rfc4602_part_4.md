#### PIM Bootstrap and RP Discovery
  For correct operation, every PIM router within a PIM domain must be able to **map a particular multicast group address to the same RP**. A notable exception to this is where a PIM domain is broken up into **multiple administrative scope regions**; these are regions where a border has been configured so that a range of multicast groups will not be forwarded across that border.
  
  The modified criteria for admin-scoped regions are that the region is convex with respect to forwarding based on the MRIB, and that all PIM routers within the scope region map scoped groups to the same RP within that region.
  
  This specification does not mandate the use of a single machanism to provide routers with the information to perform the group-to-RP mapping. Currently four machanism are possible, and all four have associated problems:
  * Static Configuration: A PIM routers must support a static configuration a group-to-RP mappings.
  * Embedded-RP: Emdedded-RP defines an address allocation policy in which the address of the Rendezvous Point (RP) is encoded in an IPv6 multicast group address.
  * Cisco's Auto-RP: Auto-RP uses a PIM Dense-Mode multicast group to annouce group-to-RP mapping from a central location. This mechanism is not useful if PIM Dense-Mode is not being run in parallel with PIM Sparse-Mode, and was only intended for use with PIM SM Version 1.
  * BootStrap Router (BSR): Any router in the domain that is configured to be a possible RP reports its candidacy to the BSR, and then a domain-wide flooding mechanism distributes the BSR's chosen set of RPs throughout the domain.
