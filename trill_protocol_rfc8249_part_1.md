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
