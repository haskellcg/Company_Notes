## 5. RBridge Parameters
  This section lists parameters for RBridge. It is expected that the TRILL MIB will include many of the items listed in this section plus additional RBridge status and data including traffic and error counts.

  The default value and range are given for parameter added by TRILL. Where a parameter is defined as a 16-bit unsigned integer and an explicit maximum is not given, that maximum is 2\*\*16-1, for parameters imported from [802.1Q-2005], [802.1D], or IS-IS [ISO10589] \[RFC1195\], see those standards for default and range if nor given here.

### 5.1. Per RBridge
  The following parameters occur per RBridge:
  * Number of nicknames, which defaults to 1 and may be configured in range of 1 to 256
  * The desired number of distribution trees to be calculated by every Rbridge in the campus and a desired number of distribution trees for the advertising RBridge, both of which are unsgined 16-bit integers that default to 1
  * The maximum number of distribution trees the RBridge can compute. This is a 16-bit unsgined inteeger that is implementation and environment dependent and not sucject to management configuration.
  * Two list of nicknames, one designating the distribution trees to be computed and one desginating distribution trees to be used as discussed in section 4.5. By defaut, these lists are empty
  * The parameters Ageing Timer and Forward Delay with the default and range specified in [802.1Q-2005]
  * Two unsigned octets that are, respectively, the confidence in {MAC, VLAN, local port} triples learned from locally received native frams and the confidence in {MAC, VLAN, remote RBridge} triples learned from decapsulating frames. These each default to 0x20 and may each be configued to value from 0x00 to 0xFE.
  * The desired minimum acceptable inter-RBridge link MTU for the campus, that is, originatingLSPBufferSize. This is a 16-bit unsgined integr number of octets that defaults to 1470 bytes, which is the minimum valid value. Any lower value being advertised by an RBridge is ignored.
  * The number of failed MTU-probe before the RBridge conclusdes that a particular MRU is not supported, which defaults to 3 and may be configured between 1 and 255
  
  Static end-station address information and confidence in such end station information statically configured can also be configured with a default configence of 0xFF anf range of 0x00 to 0xFF. By Default, there is no such static address information. The quantity of such information that can be configured is implementation dependent.

### 5.2. Per Nickname Per RBridge
  The following is configration information per nickname at each Rbridge:
  * Priority to hold the nickname, which defaults to 0x40 if no specific value has been configured or 0xC0 if it is configured.
  * Nickname priority to be selected as distribution tree root, a 16-bit unsigned integer that defaults to 0x8000
  * Nickname value, an unsigned 16-bit quantity that defaults to the configured value if configured, elso to the last value held if the RBridge coming up after a reboot and that value is remembered, else to a random value; however, in all cases the reserved values 0x0000 and 0xFFC0 through 0xFFFF are excluded

### 5.3. Per Port Per RBridge
  An RBridge has the following per-port configuration parameters:
  * The same parameters as an [802.1Q-2005] port in terms of C-VLAN IDs. Inaddition, there is an announcing VLANs set that deafults to the enabled VLANs on the port, and ranges from the null set to the set of all legal VLAN IDs.
  * The same parameters as an [802.1Q-2005] port in terms of frame priority code point mapping
  * The inhibition time for the port when it observed a change in root bridge of an attached bridged LAN. This is in units of seconds, default to 30, and can be configured to any value from 0 to 30.
  * The desired Designated VLAN that the Rbridge will advertise in its TRILL Hellos if it is the DRB for the link via that port. This defaults to the lowest VLAN ID enabled on the port and may be configured to any valid VLAN ID that is enabled on the port.
  * Four per-port configuration bits: disable bit, disable end-station service, access port, and use P2P Hellos.
  * One bit per port such that, if the bit is set, it disables learning {MAC address, VLAN, port} triples from locally received native frames on the port. Default value is 0 == learning enabled.
  * The priority of the RBridge to be DRB and its Holding Time via the port with defaults and ranges as specific in IS-IS [RFC1195]
  * A bit that, when set, enables the receipt of TRILL-encapsulated frames from an Outer.MacSA with which the Rbridge does not have an IS-IS adjacency. Default value is 0 == disabled
  * Configuration for the optional send-BPDUs solution to the writing cloest topolgy problems as described in Appendix A.3.3. Default Bridge Address is the System ID of the Rbridge with lowest System ID. If RB1 and RB2 are part of a wiring closet topolgy, both need to be configured to know about this, and know the ID that should be used in the spanning tree protocol on the specified port.

### 5.4. Per VLAN Per RBridge
  An RBridge has the following per-VLAN configuration parameters:
  * Per-VLAN ESADI Protocol participation flag, 7-bit priority, and Holding Time. Default participation flag is 0 == not participating. Default and range of priority and Holding Time as specified in IS-IS [RFC1195]
  * One bit per VLAN that, if set, disables learning {MAC address, VLAN, remote RBridge} triples from frames decapsulated in the VLAN. Defaults to 0 == learning enabled.

## 6. Security Considerations
  TODO

### 6.1. VLAN Security Considerations
  TODO

### 6.2. BPDU/Hello Denial-of-Service Considerations
  TODO

## 7. Assignment Considerations
  TODO

### 7.1. IANA Considerations
  A new IANA registri has been created for TRILL parameter with two subregistries as below.

  This initial contents of the TRILL Nicknames subregistry are as follows:
  * 0x0000 Reserved to indicate no nickname specified
  * 0x0001 - 0xFFBF Dynamic allocated by the RBridges within each Rbridge campus
  * 0xFFC0 - 0xFFBF Available for allocation by RFC Required (single value) or IETF Review (single or multiple values)
  * 0xFFFF Permanetly reserved

  The initial contents of the TRILL multicast address subregitry are as follows:
  * 01-80-C2-00-00-40 Assigned as All-RBridge
  * 01-80-C2-00-00-41 Assigned as All-IS-IS-RBridge
  * 01-80-C2-00-00-42 Assigned as All-ESADI-RBridge
  * 01-80-C2-00-00-43 to 01-80-C2-00-00-4F Available for allocation by IETF Review

### 7.2. IEEE Registration Authority Considerations
  The Ethertype 0x22F3 is assigned by the IEEE Registration Authority to the TRILL Protocol

  The Ethertype 0x22F4 is assigned by the IEEE registration Authority for L2-IS-IS.

  The block of 16 multicast MAC addresses from 01-80-C2-00-00-40 to 01-80-C2-00-00-4F is assigned by the IEEE Registration Authority for IETF TRILL protocol use.

## 8. Normative References

## 9. Informative References

## Appendix A. Incremental Deployment Considerations
### A.1. Link Cost Determination
### A.2. Appointed Forwarders and Bridged VLANs
### A.3. Wiring Closet Topology
#### A.3.1. The RBridge Solution
#### A.3.2. The VLAN Solution
#### A.3.3. The Spanning Tree Solution
#### A.3.4. Comparison of Solutions

## Appendix B. Trunk and Access Port Configuration

## Appendix C. Multipathing

## Appendix D. Determination of VLAN and Priority

## Appendix E. Support of IEEE 802.1Q-2005 Amendments
### E.1. Complete Amendments
### E.2. In-Process Amendments

## Appendix F. Acknowledgements
