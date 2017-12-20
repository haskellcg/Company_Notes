# Generic Application of Bidirectional Forwarding Detection (BFD)
  This document describes the generic application of the Bidirectional Forwarding Detection (BFD) protocol
  
## Introduction
  The BFD protocol provides a liveness detection mechanism that can be utilized by other network components for which their integral liveness mechanism are either too slow, in appropriate, or nonexistent.
  
## Overview
  The promptness of the detection of a path failure can be controlled by trading off protocol overhead and system load with detection times.
  
  BFD is not intended to directly provided control protocol liveness informartion; those protocols have their own **_means and vagaries_**. Rather, control protocols can use the services provided by BFD to inform their operation. **_BFD can be viwed as a service provided by the layer in which it is running_**.
  
  **_The service interface with BFD is straightforward. The application supplies session parameters (neighbor address, time parameters, protocol options), and BFD provides the session state, of which the most interesting transitions are to and from the Up state. The application is expected to bootstrap the BFD session, as BFD has no discovery mechanism_**
  
  If mutiple applications request different session parameters, it is a local issues as to how to resolve the parameter conflicts. BFD in turn will notify all applications bound to a session when a session state change occurs.
  
  BFD should be viewed as having an advisory role to the protocol or protocols or other network functions with which it is interacting which will then use their own mechanisms to effect any state transitions. **_The interaction is very much at arm's length, which keeps things simple and decoupled_**.
  
  It is important to remember that the interaction between BFD and its client applications has essentially no interoperability issues, because BFD is acting in an advisory nature and existing mechanisms in the client applications are used in reaction to BFD events.
  
## Basic Interaction between BFD Sessions and Clients  
  One way of modeling this interaction is to create an adaptation layer between the BFD state machine and the client application. The adaptation layer is cognizant of both the internals of the BFD implementation and the requirements of the clients
  
### Session State Hysteresis  
  Implementors may **_choose to hide rapid Up/Down/Up transitions of BFD session form its clients_**. 
  
### AdminDown State
  The AdminDown mechanism in BFD is intended to signal that the BFD session is being taken down **_for administrative purpose_**, and the session state is not indicative of the liveness of the data path.
  
  **_Therefore, a system should not indicate a connectivity failure to a client if either the local session state or the remote session state (if known) transitons to AdminDown_**, so long as that client has independent means of liveness detection (typically, control protocols)
  
  If a client does not have any independent means of liveness detection, a system should indicate a connectivity failure to a client, and assume the semantics of Down state, if either the local or remote session state transitions to AdminDown.
  
### Hitless Establishment/Reestablishment of BFD State
  It is useful to be able to configure a BFD session between a pair of systems without impacting the state of any clients that will be associate with that session. Similarly, it is useful for BFD state to be reestablished without perturbing associated clients when all BFD state is **_lost (such as in process restart situations)_**.
  
  The BFD state machine transitions that occur in the process of bringing up BFD session in such situations should not cause a connectivity failure notification to the clients.
  
## Control Protocol Interactions
  Very common client applications of BFD are control protocols, such as routing protocols. 
  
### Adjacency Establishment  
  If the session state on either the local or remote system (if known) is AdminDown, BFD has been administratively **_disabled_**, and the establishment of a control protocol adjacency must be allowed.
  
  BFD sessions are typically bootstrapped by the control protocol, using the mechanism (discovery, configuration) used by the control protocol to find neighbors. Note that it is possible in some failure scenarios for the network to be in a state such that the control protocol is capable of coming up, but the BFD session cannot be established, and, more particularly, data connot be forwarded.
  
  Therefore, the establishment of control protocol adjacencies should be blocked if both systems are willing to establish a BFD session but BFD session cannot be established. One method for determining that both systems are willing to establish a BFD session is if the control protocol carries explicit signal of this fact.
  
  If it is believed that the neighboring system does not support BFD the establishment of a control protocol adjacency should not be blocked.
  
  It is generally useful to choose the parameters resulting in the shortest Detection Time; a particular client application can always apply hystersis to the notifications from BFD if it desireds longer Detection Times.
  
  Note that many control protocols assume full connectivity between all systems on multiaccess media such as LANs. If BFD is runing on only subnet of sytems on such a network, and adjacency establishment is blocked by the absence of a BFD session, the assumptions of the control protocol may be **_violated, with unpredictable results_**.
  
### Reaction to BFD Session State Changes  
  If a BFD session transitions from Up state to AdminDown, or the session transitions from Up to Down because the remote system is indicating that the session is in state AdminDown, **_clients should not take any control protocol action_**.
  
  For other transitions from Up to Down state, the mechanism by which the protocol reacts to a path failure signaled by BFD depends on the capabilities of the protcol.
  
#### Control Protocol with a Single Data Protocol  
  Note that this should not be interpreted as BFD replacing the control protcol liveness mechanism, if any, as the control protocol may rely on machanisms not verified bt BFD (multicast, for instance) so BFD most likely cannot detect all failures that would impact the control protocol. **_However, a control protcol may choose to use BFD session state information to more rapidly detect an impending control protocol failure, particularly if the control protocol operates in-band (over the data protocol)_**.
  
  If the control protocol has an explicit mechanism rather than impacting the connectivity of the control protocol, particularly if the control protocol operates out-of-band from the failed data protocol.
  
#### Control Protocol with Multiple Data Protocols
  Slightly different mechanisms are used if the control protocol supports the routing of multiple data protocols, **_depending on whether the control protocol supports separate topologies for each data_**.
  
##### Shared Topologies  
  With a share topology, if one of the data protocols fails (as signaled by the associated BFD session), it is necessary to consider the path to have failed for all data protcols.
  
  Therefore, when a BFD session transitions from Up to Down, action should be taken in the control protocol to signal the lack of connectivity for the path in the topology corresponding to the BFD session. **_If this cannot be signaled otherwise, a control protocol timeout should be emulated for the associate neighbor_**.
  
##### Independent Topologies
  With individual routing topologies for each data protocol, only the failed data protocol needs to be rerouted around the failed path.
  
  Generally, this can be done without impacting the connectivity of other topologies (since otherwise it is very difficult to support separate topologies  for multiple data protocols).
  
### Interactions with Graceful Restart Mechanisms  
  A number of control protocol support Graceful Restart mechanisms, including IS-IS, OSPF, BGP. These mechanisms are desinged to allow a control protocol to restart without perturbing network connectivity state (lest it appear that the system and/or all of its links had failed). They are predicated on the existence of a separate forwarding plane that does not necessarily share fate with the control plane in which the routing protocols operate. **_In particular, the assumption is that the forwarding plane can continue to function while the protocols restart and sort things out_**.
   
  **_BFD implementations annouce via the Control Plane Independent "C" bit whether or not BFD shares fate with the control plane. This information is used to determine the actions to be taken in conjuntion with Graceful Restart_**.
   
  If BFD does not share its fate with the control plane on either system, it can be used to determine whether **_Graceful Restart in a control protocol is not viable (the forwarding plane is not operating)_**.
   
  If the control protocol has a Graceful Restart mechanism, BFD may be used in conjunction with this mechanism. The interaction between BFD and the control protocol depends on the capabilities of the control protocol and whether or not BFD shares fate with the control plane. **_In particular, it may be desirable for a BFD session failure to abort the Graceful Restart process and allow the failure to be visible to network_**.
  
#### BFD Fate Independent of the Control Plane  
  If BFD is implemented in the forwarding plane and does not share fate with the control plane on either system (the "C" bit is set in the BFD Control packets in both directions), **_control protocol restarts should not affect the BFD session_**. In this case, a BFD session failure implies that data can no longer be forwarded, so any Graceful Restart in progress at that time of the BFD session failure should be aborted in order to avoid black holes, and a topology change should be signaled in the control protocol.
   
#### BFD Shares Fate with the Control Plane   
  If BFD shares fate with the control plane on either system (the "C" bit is clear in either direction), a BFD session failure connot be disentangled from other events taking place in the control plane. In many cases, the BFD session will fail as a side effect of the restart taking place. As such, it would be best to avoid aborting any Graceful Restart taking place, if possible (since otherwise BFD and Graceful Restart cannot coexist)
  
##### Control Protocols with Planned Restart Signaling
  Some control protocols can signal a planned restart prior to the restart taking place. In this case, if a BFD session failure occurs during the restart, such a planned restart should not be aborted and the session failure should not result in a topology change being signaled in the control protocol.
  
##### Control Protocols without Planned Restart Signaling  
  Control protocols that cannot signal a planned restart depend on the recently restarted system to signal the Graceful Restart prior to the control protocol adjacency timeout. In most cases, whether the restart is planned or unplanned, it is likely that the BFD session will time out prior to the onset of Graceful Restart, in which case a topology change should be signaled in the control protocol.
  
  However, if the restart is in fact planned, an implementation may adjust the BFD session timing parameters prior to restarting in such a way that the Detection Time in each direction is longer than the restart period of the control protocol, providing the restarting system the same opportunity to enter Graceful Restart as it would have without BFD. 
  
  The restarting system should not send any BFD Control packets until there is a high likelihood that its neighbors know a Graceful Restart is taking place, as the first BFD Control packet will cause the BFD session to fail.
  
### Interaction with Multiple Control Protocols
  If multiple control protocols wish to establish BFD sessions with the same remote system for the same data protocol, all must share a single BFD session.
  
  If hierarchical or dependent layers of control protocols are in use (say, OSPF and Internet BGP (IBGP)), it may not be useful for more than one of them to interact with BFD. In this example, because IBGP is dependent on OSPF for its routing information, the faster failure detection replayed to IBGP may actually be deterimental. **_The cost of a peer state transition is high in BFD, and OSPF will naturally heal the path through the network if it were to receive the failure detection_**.
  
  In general, it is best for the protocol at the lowest point in the hierarchy to interact with BFD, and then to use existing interactions between the control protocols to effect changes as necessary. This will provide the fastest possible failure detection and recovery in a network.
  
## Interaction with Non-Protocol Functions
  BFD session status may be used to affect other system functions that are not protocol based (for example, **_static routes_**). If the path to a remote system fails, it may be desirable to avoid passing traffic to that remote system, so the local system may wish to take internal measures to accomplish this (such as **_withdrawing a static route and withdrawing that route from routing protocols_**).
  
  If it is known, or presumed, that the remote system is BFD capable and the BFD session is not in Up state, appropriate action should be taken (**_such as withdrawing a static route_**).
  
  If it is known, or presumed, that the remote system does not support BFD, action such as withdrawing a static route should not be taken.
  
  Bootstrapping of the BFD session in the non-protocol case is likely to be derived from configuration information.
  
  There is no need to exchange endpoints or discriminator values via any mechanism other than configuration (via Operational Support Systems or any other means) as the endpoints must be knwon and configured by the same means.
  
## Data Protocols and Demultiplexing  
  BFD is intended to **_protect a single "data protocol" and is encapsulated within that protocol_**.
   
  The BFD Control packets must be marked in the same way as the data packets, partly to ensure as much fate sharing as possible between BFD and data traffic, and also to demultiplex the initial packet if the discriminator values have not been changed.
   
## Multiple Link Subnetworks   
  A number of technologies exist for aggregating multiple parallel links at layer N-1 and treating them as a single link at layer N.
  
### Complete Decoupling  
  The simplest approach is to simply run BFD over the layer N path, with no interaction with the layer N-1 mechanisms.
  
### Layer N-1 Hints  
  This approach is to have the layer N-1 mechanism inform the layer N BFD when the aggregated link is no longer viable. In this case, the BFD need not wait for the session to time out. This is analogous to triggering a session failure based on the hardware-detected failure of a single link.
   
### Aggregating BFD Sessions   
  This approach would be to use BFD on each layer N-1 link and to aggregate the state of the multiple sessions into a single indication to the layer N clients. This approach has the advantage that it is independent of the layer N-1 technology.
  
  However, this approach only works if the layer N neighbor is the same as the layer N-1 neighbor (a single hop at layer N-1)
  
### Combinations of Scenarios
  Combinations of more than one of the scenarios listed above may be useful in some cases.
   
## Other Application Issues   
  Running BFD inside the tunnel is recommended, as it exercises more aspects of the path. One way to accommodate this is to address BFD packets based on the tunnel endpoints, assuming that they are numbered.
   
  If a planned outage is to take place on a path over which BFD is run, it is prefreable to take down the BFD session by going to AdiminDown state prior to the outage. The system asserting AdminDown should do so for at least one Detection Time in order to ensure that the remote system is aware of it.
  
  Similar, if BFD is to be deconfigured from a system, it is desirable not to trigger any client application action. Simply ceasing the transmission of BFD Control packets will cause the remote system to detect a session failure. In order to avoid this, the **_system on which BFD is being deconfigured should put the session into AdminDown state and maintain this state for a Detection Time to ensure that the remote system is aware of it_**.
  
## Interoperability Issues  
  Asynchronous mode is mandatory and is always available, and other modes and functions are negotiated at run time.
  
  The interaction between BFD and other protocols and control functions is very loosely coupled. The action taken are based on existing mechanisms in those protocols and functions, so interoperability problems are very unlikely unless BFD is applied in contradictory ways (**_such as a BFD session failure causing one implementation to go down and another implementation to come up_**).
  
## Specific Protocol Interactions (Non-Normative)  
  Since the interactions do not affect interoperability, they are non-normative
  
### BFD Interactions with OSPFv2, OSPFv3, and IS-IS
#### Session Establishment
  The most obvious choice for triggering BFD session establishment with these protocols would be to use the discovery mechanism inherent in the Hello protocols in OSPF and IS-IS to bootstrap the establishment of the BFD session. Any BFD sessions established to support OSPF and IS-IS across a single IP hop must operate in accordance with BFD-1HOP.
  
#### Reactione to BFD State Changes  
  At this time, OSPFv2 and OSPFv3 carry routing information for a single data protocol (IPV4 and IPv6, respectively) so when it is desired to signal a topology change after a BFD session failure, this should be done by tearing down the corresponding OSPF neighbor.
  
  IS-IS may be used to support only one data protocol, or multiple data protocols. [ISIS] specifies a common topology for multiple data protocols, but work is under way to support multiple data protocols. If multiple topologies are used to support multiple data protocols (or multiple classes of service of the same data protocol), the topology-specific path associate with a failing BFD session should no longer be advertised in IS-IS Label Switched Paths (LSPs) in order to **_signal a lack of connectibity_**. Otherwise, a failing BFD session should be signaled by simulating an IS-IS adjacency failure.
  
  OSPF has a planned restart signaling mechanism, whereas IS-IS does not.
   
#### OSPF Virtual Links
  If it is desired to use BFD for failure detection of OSPF Virtual Links, the mechanism described in [BFD-MULTI] must be used, since OSPF Virtual Links may traverse an arbitrary number of hops. BFD authentication should be used and is strongly encouraged.
  
### Interaction with BGP
  BFD maybe useful with External Border Gateway Protocol (EBGP) session [BGP] in order to **_more rapidly trigger topology changes in the face of path failure_**. It is generally unwise for IBGP sessions to interact with BFD if the underlying IGP is already doing so.
  
  EBGP sessions being advised by BFD establish either a one-hop [BFD-1HOP] or a multihop [BFD-MULTI] session, depending on whether or not neighbot is immediately adjacent. The BFD session should be established to the BGP neighbor (as opposed to any other Next Hop advertised in BGP). BFD authentication should be used and is strongly encouraged.
  
  [BGP-GRACE] describes a Graceful Restart mechanism for BPG. If Graceful Restart is not taking place on an EBGP session, and the corresponding BFD session fails, the EBGP session should be torn down. If Graceful Restart is taking place. BGP Graceful Restart does not signal planned restarts. 
  
### Interactions with RIP
  The Routing Information Protocol (RIP) is somewhat unique in that, at least as specified, neighbor adjacency state is not stored per se. Rather, installed routes contain a next hop address, which in most cases is the address of the advertising neighbor (but may not be).
  
  In the case of RIP, when the BFD session associate with a neighbor fails, an expiration of the "timeout" timer for each route installed from the neighbor (for which the neighbor is the next hop) should be simulated.
  
  Note that if a BFD session fails, and a route is received from that neighbor with a next hop address that is not the address of the neighbor itself, the router will linger until it natually times out (after 180 seconds). However, if an implementation keep track of all of the routes received from each neighbor, all of the routes from the beighbor corresponding to the failed BFD session should be timed out, regardless of the next hop specified therein, and thereby avoiding the lingering route problem.
  
## Security Considerations  
   
## References   
 
