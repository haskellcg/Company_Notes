# Transparent Interconnection of Lots of Links (TRILL): Problem and Applicability Statement

## Abstract
  Current IEEE 802.1 LANs use spanning tree protocols that have a number of chanllenges. These protocols need to strickly avoid loops. even temporary ones, during route propagation, because of the lack of header loop detection support. Routing tend not to take full advantage of alternate paths, or even non-overlapping pairwise paths (in the case of spanning trees).

## 1. Introduction
  Conventional Ethernet networks -- known in the Internet as Ethernet link subnet -- have a number of attractive features, allowing hosts and routers to relocate within the subnet without requiring renumbering, and support automatic configuration. **The basis of simplicity of these subnets is the spanning tree, which although simple and elegant, can have substantial limitations**.

  With spanning trees, the bandwidth across the subnet is limited because traffic flows over a subnet of links forming a single tree -- or, with the latest version of the protocol and significant additional configuration, over a small number of superimposed trees.

  The alternative to an Ethernet link subnet is often a network subnet. Network subnets can use link-state routing protocols that allow traffic to traverse least-cost paths rather than being aggregated on a spanning tree backbone, providing higher aggregate capacity and more resistance to link failures.

  Unfortunately, IP -- the dominant network layer technology -- requires that hosts be renumbered when relocated in different network subnets, interrupting network and transport associations that are in progress during the transition.

## 2. The TRILL Problem
  Ethernet subnets have evolved from "thicknet" to "thinnet" to twisted pair with hubs to twisted pair with switches, becoming increasingly simple to wire and manage. Each level has corresponding toponogy restrictions; thicknet is inherently linear, whereas thinnet and hub-connected twisted pair have to be wired as a tree. Switches, added in IEEE 802.1D, allow network managers to avoid thinking in trees, where the spanning tree protocol finds a valid tree automatically; unfortunately, this additional simplicity comes with a number of associated penalties.

  The spanning tree often results in inefficient use of the link topology; traffic is concertrated on the spanning tree path. The addition in IEEE 802.1Q of support for multiple spanning trees helps a little, but the use of multiple spanning trees requires additional configuration, the number of trees is limited, and these defects apply within each tree regardless.

  The spanning tree protocol reacts to certain small topology changes with large effects on the reconfiguration of links in use. Each of these aspects of the spanning tree protocol can cause problems for current link-layer deployments.

### 2.1. Inefficient Paths
### 2.2. Multipath Forwarding
### 2.3. Convergence and Safety
### 2.4. Stability of IP Multicast Optimization
### 2.5. IEEE 802.1 Bridging Protocols
### 2.6. Problems Not Addressed

## 3. Desired Properties of Solutions to TRILL
### 3.1. No CHange to Link Capabilities
### 3.2. Zero Configuration and Zero Assumption
### 3.3. Forwarding Loop Mitigation
### 3.4. Spanning Tree Management
### 3.5. Multiple Attachments
### 3.6. VLAN Issues
### 3.7. Operational Equivalence
### 3.8. Optimizations
### 3.9. Internet Architecture Issues

## 4. Applicability

## 5. Security Considerations

## 6. Acknowledgments

## 7. Informative References
