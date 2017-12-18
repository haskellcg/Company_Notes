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




