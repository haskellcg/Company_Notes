# Generic Application of Bidirectional Forwarding Detection (BFD)
  This document describes the generic application of the Bidirectional Forwarding Detection (BFD) protocol
  
## Introduction
  The BFD protocol provides a liveness detection mechanism that can be utilized by other network components for which their integral liveness mechanism are either too slow, in appropriate, or nonexistent.
  
## Overview
  The promptness of the detection of a path failure can be controlled by trading off protocol overhead and system load with detection times.
  
  BFD is not intended to directly provided control protocol liveness informartion; those protcols have their own **_means and vagaries_**. Rather, control protocols can use the services provided by BFD to inform their operation. **_BFD can be viwed as a service provided by the layer in which it is running_**.
  
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
  Note that this should not be interpreted as BFD replacing the control protcol liveness mechanism, if any, as the control protocol may rely on machanisms not verified bt BFD (multicast, for instance) so BFD most liekly cannot detect all failures that would impact the control protocol. **_However, a control protcol may choose to use BFD session state information to more rapidlt detect an impending control protocol failure, particularly if the control protocol operates in-band (over the data protocol)_**.
  
  If the control protocol has an explicit mechanism rather than impacting the connectivity of the control protocol, particularly if the control protocol operates out-of-band from the failed data protocol.
  
#### Control Protocol with Multiple Data Protocols
  Slightly different mechanisms are used if the control protocol supports the routing of multiple data protocols, **_depending on whether the control protocol supports separate topologies for each data_**.
  
##### Shared Topologies  
  
  
  


