# Transparent Interconnection of Lots of Links (TRILL):MTU Negotiation

# Abstract

# 1. Introduction
  * Sz: **campus-wide** minimum acceptable inter-RBridge MTU size
  * [2-Way] => (MTU Test) => [Report]
  * Lz: recommended **link-wide** minimum inter-RBridge MTU size
  * New MTU test algorithm specified to update 4.3.2 of [RFC6325], which backward compatible with the old one
  
# 2. Link-Wide TRILL MTU Size
  Some TRILL IS-IS PDUs are exchanged only between neighbors instead of throughout the whole campus. They are confined by the link-wide **Lz** intead of **Sz**. Complete Sequence Number PDUs (CSNPs) and Partial Sequence Number PDUs (PSNPs) are examples of such PDUs.
  
  originatingSNPBufferSize = min(MTU of port, Maximum LSP size that TRILL IS-IS implementation can handle)
  
  Extended L1 Circuit Scope (E-L1CS) Flooding Scope LSP (FS-LSP)
  
  level-1 link-local PDU (such as a PSNP or CSNP), is controlled by the value of the management parameter **originatingL1SNPBufferSize**, this value determines Lz.
  
  TRILL APPsub-TLV shown in Figure 1 should be included in a TRILL GENINFO TLV [RFC7357] in a E-L1CS FS-LSP fragment zero
  ```
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        | Type = 21                     | (2 bytes)
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        | Length = 2                    | (2 bytes)
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        | originatingSNPBufferSize      | (2 bytes)
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   ```
   
## 2.1. Operations   
  If more than one originatingSNPBufferSize APPsub-TLV occurs in fragment zero, the one advertising the smallest value for originatingSNPBufferSize, but not less than 1470 bytes, is used.
  
  Even if all RBridges on the specific link have reached consensus on the value of link-wide Lz based on advertised originatingSNPBufferSize, it dose not mean that these RBridges can safely exchange PDUs between each other.
  
  **Therefore, the link MTU size should be tested.**
  
  As for Sz, RBridge continue to propagate their originatingL1LSPBufferSize across the campus through the advertisement of LSPs as defined in section 4.3.2 of [RFC6352]. The smallest value of Sz advertised by any RBridge, but not less than 1470, will be deemed as Sz. Each RBridge formats their "campus-wide" PDUs -- for example, **LSPs** -- no greater than what they determine as Sz.
  
# 3. Testing Link MTU Size  
  Event A6:
  * Event: The link MTU size has been tested
  * Condition: The link can support Sz
  
  RTT: Round-Trip Time (RTT), the default value of 5 milliseconds should be assumed

  * Step 0: RB1 sends an MTU-probe padded to the size of link-wide Lz
    * If RB1 successfully receives the MTU-ack from RB2 ...
    * Try 1470 bytes, x = (Lz + 1470) / 2
    
  * Step 1: RB1 tries to send an MTU-probe padded to the size x
  
  MTU testing is only done in the Designated VLAN [RFC7177]. Since the execution of the above algorithm can be resource consuming, it is reommended that the Designated RBridge (DRB) take the responsibility to do the testing.
  
  **Multicast MTU-probes** are used instead of unicast when multiple RBridge are desired to respond with an MTU-ack on the link.
  
  RBridges have figured out the upper bound and lower bound of the link MTU size from the execution of the above algorithm. If Sz is smaller than the lower bound or greater than the upper bound, RBridges can directly judge whether the link supports Sz without MTU-probing.
  * If lowerBound >= Sz, this link can support Sz.
  * Else if upperBound <= Sz, this link cannot support Sz.
  
  Otherwise, RBridges SHOULD test whether the link can support Sz as in item (c) below. If they do not, the only safe assumption will be that the link cannot support Sz. This assumption, without testing, might rule out the use of a link that can, in fact, handle packets up to Sz. In the worst case, this might result in unnecessary network partition.
  * lowerBound < Sz < upperBound. RBridges probe the link with MTU-probe messages padded to Sz. If an MTU-ack is received within k tries, this link can support Sz. Otherwise, this link cannot support Sz. Through this test, the lower bound and upper bound of the link MTU size can be updated accordingly.
  
# 4. Refreshing Sz  
  * Joining
    * When a new RBridge joins the campus and its originatingL1LSPBufferSize is smaller than the current Sz, reporting its originatingL1LSPBufferSize in its LSPs will cause other RBridges to decrease their Sz. Then, any LSP greater than the reduced Sz MUST be split, and/or the LSP contents in the campus MUST be otherwise redistributed so that no LSP is greater than the new Sz.
    * If the joining RBridge’s originatingL1LSPBufferSize is greater than or equal to the current Sz, reporting its originatingL1LSPBufferSize will not change Sz.
    
  * Leaving
    * From the specification of the Joining process, we know that if an RBridge’s originatingL1LSPBufferSize is smaller than Sz, this RBridge will not join this campus.
    * When an RBridge leaves the campus and its originatingL1LSPBufferSize equals Sz, its LSPs are purged from the remainder of the campus after reaching MaxAge [IS-IS]. Sz MAY be recalculated and MAY increase. In other words, while in most cases RB1 ignores link-state information for IS-IS unreachable RBridge RB2 [RFC7780], originatingL1LSPBufferSize is meaningful. Its value, even from IS-IS unreachable RBridges, is used in determining Sz. This updates [RFC7780]
    * When an RBridge leaves the campus and its originatingL1LSPBufferSize is greater than Sz, Sz will not be updated, since Sz is determined by another RBridge with a smaller originatingL1LSPBufferSize.
    
  **Frequent LSP "resizing" is harmful to the stability of the TRILL campus, so, to avoid this, upward resizing should be dampened. When an upward resizing event is noticed by an RBridge, it is recommended that a timer be set at that RBridge via a configurable parameter -- LSPresizeTime -- whose default value is 300 seconds.** Before this time expires, all subsequent upward resizing will be dampened (ignored). Of course, in a well-configured campus will all RBridges configured to have the same originatingL1LSPBufferSize, no resizing will be necessary. It does not matter if different RBridge have different dampening timers or if some RBridges resize upward more quickly than others.
  
  If the refreshed Sz is smaller than the lower bound or greater than the upper bound of the tested link MTU size, the issue of resource consumption from testing the link MTU size can be avoid according to rule (a) or (b) as specified in Section 3. Otherwise, RBridges test the link MTU size according to the rule (c).
  
# 5. Relationship betweem Port MTU, Lz, and Sz  






