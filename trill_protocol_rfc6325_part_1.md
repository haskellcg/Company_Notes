# Routing Bridges (RBridges): Base Protocol Specification

## Abstract

## 1. Introdcution
### 1.1. Algorithm V2, by Ray Perlner
### 1.2. Normative Content and Precedence
### 1.3. Terminology and Notation in This Document
### 1.4. Categories of Layer 2 Frames
### 1.5. Acronyms

## 2. RBridges
### 2.1. General Overview
### 2.2. End-Station Addresses
### 2.3. RBridge Encapsulation Architecture
### 2.4. Forwarding Overview
#### 2.4.1. Known-Unicast
#### 2.4.2. Multi-Destination
### 2.5. RBridges and LANs
#### 2.5.1. Link VLAN Assumptions
### 2.6. RBridges and IEEE 802.1 Bridges
#### 2.6.1. RBridge Ports and 802.1 Layering
#### 2.6.2. Incremental Deployment

## 3. Details of the TRILL Header
### 3.1. TRILL Header Formart
### 3.2. Version (V)
### 3.3. Reserved (R)
### 3.4. Multi-Destination (M)
### 3.5. Op-Length
### 3.6. Hop Count
### 3.7. RBridge Nicknames
#### 3.7.1. Egress RBridge Nickname
#### 3.7.2. Ingress RBridge Nickname
#### 3.7.3. RBridge Nickname Selection
### 3.8 TRILL Header Options

## 4. Other RBridge Design Details
### 4.1. Ethernet Data Encapsulation
#### 4.1.1. VLAN Tag Information
#### 4.1.2. Inner VLAN Tag
#### 4.1.3. Iuter VLAN Tag
#### 4.1.4. Frame Check Sequence (FCS)
### 4.2. Link State Protocol (IS-IS)
#### 4.2.1. IS-IS RBridge Identity
#### 4.2.2. IS-IS Instances
#### 4.2.3. TRILL IS-IS Frames
#### 4.2.4. TRILL Link Hellos, DRBs, and Appointed Forwarders
##### 4.2.4.1. P2P Hello Links
##### 4.2.4.2. Designated RBridge
##### 4.2.4.3. Appointed VLAN-x Forwardre
##### 4.2.4.4. TRILL LSP Information
#### 4.2.5. The TRILL ESADI Protocol
##### 4.2.5.1. TRILL ESADI Participation
##### 4.2.5.2. TRILL ESADI Information
#### 4.2.6. SPF, Forwarding, and Ambiguous Destinations
### 4.3. Inter-RBridge Link MTU Size
#### 4.3.1. Determining Campus-Wide TRILL IS-IS MTU Size
#### 4.3.2. Testing Link MTU Size
### 4.4. TRILL-Hello Protocol
#### 4.4.1. TRILL-Hello Rationale
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
