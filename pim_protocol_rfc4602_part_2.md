#### PIM Register Messages
  The Designated Router (DR) on a LAN or point-to-point link encapsulates multicast packets from local sources to the RP for the **relevant group** unless it recently received a **Register-Stop message** for that (S, G) or (\*, G) from the RP.
  
  When the DR receives a Register-Stop message from the RP, it starts a **Register-Stop Timer** to maintain this state. Just before the Register-Stop timer Expires, the DR sends a **Null-Register Message** to the RP to allow the RP to refressh the Register-Stop information for the DR.
  
  If the Register-Stop timer actually expires, the DR will resume encapsulating packets from the source to the RP.
  
###### Sending Register Messages from the DR
  For the purposes of specification, we represent **the machanism to encapsulate packets** to the RP as a **Register-Tunnel**  interface, which is added to or removed from the (S, G) olist. The tunnel interface then takes part in the normal packet forwarding rules.
  
  There are four states in the DR's per-(S, G) Register state machine:
  * Join (J): The register tunnel is "joined" (the join is actually implicit, but the DR acts as if the RP has joined the DR on the tunnel interface)  
  * Prune (P): The register tunnel is "pruned" (this occurs when a Register-Stop is received)  
  * Join-Pending (JP): The register tunnel is pruned, but the DR is contemplating adding it back  
  * NoInfo (NI): No information, this is the initial state, and the state when the router is not the DR
  
  In addition, a **Register-Stop Timer** (RST) is kept if the state machine is **not in the NoInfo state**.
  
  Prev State|Register-Stop Timer expires|Could Register->True|Could Register->False|Register-Stop received|RP changed
  ----------|---------------------------|--------------------|---------------------|----------------------|----------
  NoInfo(NI)|-|->J state(add reg tunnel)|-|-|-
  Join(J)|-|-|->NI state(remove tag tunnel)|->P state(remove tag tunnel;set Register-Stop timer)|->J state(update reg tunnel)
  Join-Pending(JP)|->J state(add reg tunnel)|-|->NI state|->P state(set Register-Stop Timer)|->J state(add reg tunnel;cancel Register-Stop timer)
  Prune(P)|->JP state(set Register-Stop timer;send Null-Register)|-|->NI state|-|->J state(add reg tunnel;cancel Register-Stop Timer)
  
  Actions are defined:
  * Add Register Tunnel: A Register-Tunnel virtual interface, **VI**, is created (if it doesn't already exist) with its encapsulation target being RP(G). **DownstreamJPState(S, G, VI) is set to Join state**, causing the tunnel interface to be added to immediate_olist(S, G) and inherited_olist(S, G).
  * Remove Register Tunnel: VI is the Register-Tunnel virtual interface with encapsulation target of RP(G). **DownstreamJPState(S, G, VI) is set to NoInfo state**, causing the tunnel interface to be removed from immediate_olist(S, G) and inherited_olist(S, G). If DownstreamJPState(S, G, VI) is NoInfo for all (S, G), then VI can be deleted.
  * Update Register Tunnel: The action is accurs when RP(S) changes.
  
  ```c++
  bool CouldRegister(S, G)
  {
      return (I_am_DR(RPF_interface(S))) AND
             (KeepaliveTimer(S, G) is running) AND
             (DirectlyConnect(S) == TRUE)
  }
  ```
  * Encapsulation Data Packets in the Register Tunnel: The encapsulation DR may perform **Path MTU Discovery** to the RP to determine the effective MTU off the tunnel. Fragmentation for the smaller MTU should take both the **outer IP header** and the **PIM register header** overhead into account. In IPv6, the DR must perform Path MTU Discovery, and an ICMP Packet too big message must be sent by the encapsulating DR if it receives a packet that will not fit in the effective MTU of the tunnel. **The TTL** of a forwarded data packet is determented before it is encapsulated in the register Tunnel. The **IP ECN / Diffserv Code Point(DSCP)** bits should be copied from the original packet to the IP header of the encapsulating packet.
  * Handling Register-Stop(\*, G) messages at the DR: This was normal course of action in RFC 2362 when the **Register message** matched against (\*, G) state at the RP, and it was defined as meaning "**stop encapsulating all sources for this group**". A Register-Stop (\*, G) should be treated as a Register-Stop(S, G) for all (S, G) Register state machines that are not in the NoInfo state. A router should not apply a Register-Stop (\*, G) to source that become active after the Register-Stop(\*, G) was received.
  
###### Receiving Register Messages at the RP
  When an RP receives a Register message, the course of action is decided according to the following pseudocode:
  
  ```c++
  packet_arrives_on_rp_tunnel(rpt)
  {
      if (outer.dst is not one of my addresses){
          drop the packet silently
          // Note: this may be spoofing attempt
      }
      
      if (I_am_RP(G) AND outer.dst == RP(G)){
          sentRegisterStop = FLASE
          if (register.borderbit == TRUE){
              if (PMBR(S, G) == unknown){
                  PMBR(S, G) = outer.src
              } else if (outer.src != PMBR(S, G)){
                  send Register-Stop(S, G) to outer.src
                  drop the packet silently
              }
          }
          
          if (SPTbit(S, G) OR
              (SwitchToSptDesired(S, G) AND
               inherited_olist(S, G) == NULL)){
              send Register-Stop(S, G) to outer.src
              sendRegisterStop = TRUE
          }
          
          if (SPTbit(S, G) OR SwitchToSptDesired(S, G)){
              if (sentRegisterStop == TRUE){
                  set KeepaliveTimer(S, G) to RP_Keepalive_Period
              } else {
                  set KeepaliveTimer(S, G) to Keepalive_Period
              }
          }
      } else {
          send Register-Stop(S, G) to outer.src
          // Note (*)
      }
  }
  ```
  
#### PIM Join/Prune Messages
  A PIM Join/Prune message consists of **a list of groups** and **a list of Joined and Prune sources** for that group. When processing a received Join/Prune Message, each joined or pruned source for a group is effectively considered individually, and apply to one or more of the following state machines.
  
  When considering a Join/Prune message whose **Upstream Neighbor Address** field addresses this router, (\*, G) Joins and Prunes can affect both the (\*, G) and (S, G, rpt) downstream state machines, while (\*, \*, RP), (S, G), and (S, G, rpt) Joins and Prunes can only affect their respective downstream state machines.
  
  When considering a Join/Prune Message whose **Upstream Neighbor Address** field addresses another router, most Join or Prune messages could affect each upstream state machine.
  
  In general, a PIM Join/Prune message should only be accepted for processing if it comes from a **known PIM neighbor**. In addition, if the Hello message from a neighbor was **authenticated using IPsec AH**, then all Join/Prune messages from that neighbor must also be authenticated using IPsec AH.
  
###### Receiving (\*, \*, RP) Join/Prune Message
  The per-interface state machine:
  * NoInfo (NI): The interface has no (\*, \*, RP) Join state and no timers running
  * Join (J): The interface has (\*, \*, RP) Join state, which will cause the router to forward packet **destined for any group handled by RP** from this interface except if there is also (S, G, rpt) prune information or the router lost an assert on this interface
  * Prune-Pending (PP): The interface has received a Prune(\*, \*, RP) on this interface from a **downstream neighbor** and is waiting to see whether the prune will be overridden by another downstream router. For forwarding purposes, the Prune-Pending state functions **exactly like the Join state**.
  
  In addition, the state machine uses two timers:
  * ExpiryTimer (ET)： This timer is restarted when a **valid Join(\*, \*, RP)** is received. Expiry of the ExpiryTimer causes the interface state to revert to NoInfo for this RP.
  * Prune-Pending Timer (PPT): This timer is set when a **valid Prune(\*, \*, RP)** is received. Expiry of the ExpiryTimer causes the interface state to revert to NoInfo for this RP.
  
  Prev State|Recieve Join(\*, \*, RP)|Receive Prune (\*, \*, RP)|Prune Pending Timer Expires|Expiry Timer Expires
  ----------|------------------------|--------------------------|---------------------------|--------------------
  NoInfo(NI)|->J state(start Expiry Timer)|->NI state|-|-
  Join(J)|->J state(restart Expiry timer)|->PP state(start Prune-Pending Timer)|-|->NI state
  Prune-Pending(RP)|->J state(restart Expiry Timer)|->PP state|->NI state(Send Prune Echo(\*, \*, RP))|->NI state
  
  The transition events "Receive Join(\*, \*, RP)" and "Receive Prune(\*, \*, RP)" imply receiving a Join or Prune **targeted to this router's primary IP address on the received interface**.
  
  On unnumbered interfaces on point-to-point links, the router's address should be the same address it chose for the Hello message it sent over that interface.
  
  Transitions from NoInfo State: Receive Join(\*, \*, RP)  
  Transitions from Join State:  
  Transitions from Prune-Pending State:  
  
  **A PruneEcho(\*, \*, RP)** is simply a **Prune(\*, \*, RP) message** sent by the upstream router on a LAN with its own address in the Upstream Neighbor Address field. Its purpose is to add **additional reliability** so that if a Prune taht should have been overridden by another router is lost locally on the LAN, then the PruneEcho may be received and cause the override to happen. 
  
###### Receiving (\*, G) Join/Prune
  When a router receives a Join(\*, G), it must first check to see whether the RP in the message matches RP(G) (the router's idea of who the RP is).
  
  If a router has no RP information (e.g., has not recently received a BSR message), then it may choose to **accept Join(\*, G)** and treat the **RP in the messages as RP(G)**. Received Prune(\*, G) messages are processed even if the RP in the message does not match RP(G)
  
  There are three states:
  * NoInfo (NI): The interface has no (\*, G) Join state and no timers running
  * Join (J): The router has (\*, G) Join state, which will cause the router to forward packets destined for G from this interface except if there is also (S, G, rpt) prune information or the router lost an assert on this interface.
  * Prune-Pending(PP): The router has received a Prune(\*, G) on this interface from a downstream and is **waiting to whether the prune will be overrideen by another downstream router**. For forwarding purpose, the Prune-Pending state functions exactly like **the Join state**.
  
  In addition, the state machine uses two timers:
  * Expiry Timer (ET): This timer is **restarted when a valid Join(\*, G) is received**. Expiry of the ExpiryTimer causes the interface state to **revert to NoInfo** for this group
  * Prune-Pending Timer (PPT): This timer is set when a valid Prune(\*, G) is received. Expiry of the Prune-Pending Timer causes the interface state to revert to NoInfo for this group
  
  Prev State|Receive Join(\*, G)|Receive Prune(\*, G)|Prune-Pending Timer Expires|Expiry Timer Expires
  ----------|-------------------|--------------------|---------------------------|--------------------
  NoInfo (NI)|->J state(start Expiry Timer)|->NI state|-|-
  Join (J)|->J state(restart Expiry Timer)|->PP state(start Prune-Pending Timer)|-|->ni state
  Prune-Pending(PP)|->J state(restart Expiry Timer)|->PP state|->NI state(send Prune-Echo(\*, G))|->NI state
  
  Transitions from NoInfo state:  
  Transitions from Join state:  
  Transitions form Prune-Pending state:  
  
###### Receiving (S, G) Join/Prune Message
  The per-interface state machine:
  * NoInfo (NI)
  * Join (J)
  * Prune-Pending (PP)
  
  In Addition, there are two timers:
  * Expiry Timer (ET)
  * Prune-Pending Timer (PPT)
  
  Prev State|Receive Join(S, G)|Receive Prune(S, G)|Prune-Pending Timer Expires|Expiry Timer Expires
  ----------|------------------|-------------------|---------------------------|--------------------
  NoInfo (NI)|->J state(start Expiry Timer)|->NI state|-|-
  Join (J)|->J state(restart Expiry Timer)|->PP state(start Prune-Pending Timer)|-|->NI state
  Prune-Pending (PP)|J state(restart Expiry Timer)|->PP state|->NI state(Send Prune-Echo(S, G))|->NI state
  
###### Receiving (S, G, rpt) Join/Prune Messages
  The per-interface state machine:
  * NoInfo (NI): The interface has no (S, G, rpt) Prune state and no (S, G, rpt) timer running
  * Prune (P): The interface has (S, G, rpt) Prune state, which will cause the router not to forward packets from S destined for G from this interface even though the interface has active (\*, G) Join state.
  * Prune-Pending (PP): The router has received a Prune(S, G, rpt) on this interface from a downstream neighbor and is waiting to see whether the prune will be overridden by another downstream router. For forwarding purposes, the Prune-Pending state **functions exactly like the NoInfo state**.
  * PruneTmp (P'): This state is a **transient state** that for forwarding purposes behave exactly **like the Prune state**. A **(\*, G) Join** has been received (which may cancel the (S, G, rpt) prune). As we parse the Join/Prune message from top to bottom, we first enter this state if **the message contains a (\*, G) Join**. Later in the message, we will normally encounter an (S, G, rpt) prune to reinstate the Prune State. **However, if we reach the end of the message without encoutering such a (S, G, rpt) prune, then we will revert to NoInfo state in this state machine**.
  * Prune-Pending-Tmp (PP'): This state is a transient state that is identical to P' except that it is associate with the PP state that the P state. For forwarding purposes, PP' behaves exactly like PP state.
  
  In addition, there are two timers:
  * Expiry Timer (ET): This timer is set when a valid Prune(S, G, rpt) is received. Expiry Of Expiry Timer cause this state machine to **NoInfo state**.
  * Prune-Pending Timer (PPT): This timer is set when a valid Prune(S, G, rpt) is received. Expiry of the Prune-Pending timer cause this state machine to **Prune state**.
  
  Prev State|Receive Join(\*, G)|Receive Join(S, G, rpt)|Receive Prune(S, G, rpt)|End of Message|Prune-Pending Timer Expires|Expiry Timer Expires
  ----------|-------------------|-----------------------|------------------------|--------------|---------------------------|------------
  NoInfo (NI)|-|-|->PP state(start Prune-Pending Timer; start Expiry Timer)|-|-|-
  Prune (P)|->P' state|->NI state|->P state(restart Expiry Timer)|-|-|->NI state
  Prune-Pending (P)|->PP' state|->NI state|-|-|->P state|-
  PruneTmp (P')|-|-|->P state(restart Expiry Timer)|->NI state|-|-
  Prune-Pending-Tmp (PP')|-|-|->PP state(restart Expiry Timer)|->NI state|-|-
  
  Transitions from NoInfo State  
  Transitions from Prune-Pending State  
  Transitions from Prune State  
  Transitions from Prune-Pending-Tmp State  
  Transitions from PruneTmp State
  
###### Sending (\*, \*, RP) Join/Prune Message
  This per-interface state maichine for (\*, \*, RP) hold join state from **downstream PIM routers**. This state then determines whether a router needs to propagate a Join(\*, \*, RP) upstream towards the RP.
  
  If a router wishes to propagete a Join(\*, \*, RP) upstream, it must also **watch for message on its upstream interface from other routers on that subnet**, then these may modify its behavior.
  
  In addition, if **the MRIB changes** to indicate that the next hop towards the RP has changed, the router should prune off from the old next hop and join towards the new next hop.
  
  The upstream(\*, \*, RP) state machine contains only two states:
  * Not Joined
  * Joined
  * **JoinDesired(\*, \*, RP)**
  
  In addition, one **timer JT(\*, \*, RP)** is kept that is used to trigger the sending of a Join(\*, \*, RP) to the upstream next hop towards the RP.
  
  In Joined (J) state:  
  
  Timer Expires|See Join(\*, \*, RP) to MRIB. next hop(RP)|See Prune(\*, \*, RP) to MRIB. next hop(RP)|NBR(RPF_interface(RP), MRIB.next_hop(RP)) changes|MRIB.next_hop(RP) GenID changes
  -------------|------------------------------------------|-------------------------------------------|------|--------
  Send Join(\*, \*, RP), Send Join Timer to t_periodic|Increase Join Timer to t_joinsupress|Decrease Join Timer to t_override|send Join(\*, \*, RP) to new next hop; send Prune(\*, \*, RP) to old next hop;set Join Timer to t_periodic|Decrease Join Timer to t_override
  
  ```c++
  bool JoinDesired(*, *, RP)
  {
      if immediate_olist(*, *, RP) != NULL
          return TRUE
      else
          return FALSE
  }
  ```
  
  Transitions from NotJoined State:
  * JoinedDesired(\*, \*, RP) become true: The downstream state for (\*, \*, RP) has changed so that at least one interface is in immediate_olist(\*, \*, RP), making JoinDesired(\*, \*, RP) become True. The upstream (\*, \*, RP) state machine transitions to Joined state. Send Join(\*, \*, RP) to the **appropriate upstream neighbor**, which is NBR(RPF_interface(RP), MRIB.next_hop(RP)). Set **the Join Timer (JT)** to expire after t_periodic seconds
  
  Transitions from Joined State:
  * JoinedDesire(\*, \*, RP) become false
  * Join Timer Expires
  * See Join(\*, \*, RP) to MRIB.next_hop(RP)
  * See Prune(\*, \*, RP) to MRIB.next_hop(RP)
  * NBR(RPF_interface(RP), MRIB.next_hop(RP)) changes
  * MRIB.next_hop(RP) GenID changes
  
###### Sending (\*, G) Join/Prune Message
  The per-interface state machines for (\*, G) hold join state from downstream PIM routers. This state then determines whether a router needs to propagate a Join(\*, G) upstream towards the RP.
  
  If **a (\*, G) Assert** occurs on the upstreams interface, and this **changes this router's idea of the upstream neighbor**, it should be prepare to ensure that the Assert winner is aware of downstream routers by sending a Join(\*, G) almost immediately.
  
  If addition, if the MRIB changes to indicate that next hop towards the RP has changed, and either the upstream interface changes or there is no Assert winner on the upstream interface, the router should prune off from the old next_hop and join towards the new next hop.
  
  The upstream (\*, G) state machine only contains two state:
  * Not Joined  
  * Joined
  * **one Timer JT(\*, G)**
  
  In Joined state:
  
  Timer Expires|See Join(\*, G) to RPF'(\*, G)|See Prune(\*, G) to RPF'(\*, G)|RPF'(\*, G) changes due to an Assert|RPF'(\*, G) changes not due to an Assert|RPF'(\*, G) GenID changes
  -------------|------------------------------|-------------------------------|------------------------------------|----|----
  Send Join(\*, G); set Join Timer to t_periodic|Increase Join Timer to t_joinsupress|Descrease Join Timer to t_override|Decrease Join Timer to t_overide|Send Join(\*, G) to new next hop; send Prune(\*, G) to old next hop; set Join Timer to t_periodic|Decrease Join Timer to t_override
  
  ```c++
  bool JoinDesired(*, G)
  {
      if (immediate_olist(*, G) != NULL OR
          (JoinDesired(*, *, RP(G)) AND
           AssertWinner(*, G, RPF_interface(RP(G))) != NULL))
          return TRUE
      else
          return FALSE
  }
  ```
  
  Transitions from NotJoined State:
  * JoinDesired(\*, G) become true
  
  Transitions from Joined State:
  * JoinDesired(\*, G) become False
  * Join Timer Expires
  * See Join(\*, G) to RPF'(\*, G)
  * See Prune(\*, G) to RPF'(\*, G)
  * RPF'(\*, G) changes due to an Assert
  * RPF'(\*, G) changes not due to an Assert
  * RPF'(\*, G) GenID changes
  
###### Sending (S, G) Join/Prune Message
  The per_interface state machines for (S, G) hold join state from downstream PIM routers. This state then determines whether a router needs to propagate a Join(S, G) upstream towards the source.
  
  If a router wishes to propagate a Join(S, G) upstream, it must also watch for message on its upstream interface from other routers on that subnet, and these may modify its behavior.
  
  If a (S, G) Assert occurs on the upstream interface, and this changes the this router's idea of the upstream neighbor, it should be prepared to ensure that the Assert winner is aware of downstream routers by scheduling a Join(S, G) to be sent almost immediately.
  
  The upstream (S, G) state machine only contains two states:
  * Not Joined
  * Joined
  * **one timer JT(S, G)**
  
  In Joined State:
  
  Timer Expires|See Join(S, G) to RPF'(S, G)|See Prune(S, G) to RPF'(S, G)|See Prune(S, G, rpt) to RPF'(S, G)|See Prune(\*, G) to RPF'(S, G)|RPF'(S, G) changes not due to an Assert|RPF'(S, G) GenID changes|RPF'(S, G) changes due to an Assert
  -------------|----------------------------|-----------------------------|----------------------------------|----|----|----|----
  Send Join(S, G);set Join Timer to t_periodic|Increase Join Timer to t_joinsuppress|Decrease Join Timer to t_override|Decrease Join Timer to t_override|Decrease Join Timer to t_override|Send Join(S, G) to new next hop;send Prune(S, G) to old next hop;set join Timer to t_periodic|Decrease Join Timer to t_override|Decrease Join Timer to Override
  
  ```c++
  bool JoinDesired(S, G)
  {
      return (immediate_olist(S, G) != NULL OR
              (KeepaliveTimer(S, G) is running AND
               inherited_olist(S, G) != NULL))
  }
  ```
  
  Transitions from NotJoined State:
  * JoinDesired(S, G) become True
  
  Transitions from Joined State:
  * JoinDesired(S, G) become False
  * Join Timer Expires
  * See Join(S, G) to RPF'(S, G)
  * See Prune(S, G) to RPF'(S, G)
  * **See Prune(S, G, rpt) to RPF'(S, G)**
  * See Prune(\*, G) to RPF'(S, G)
  * RPF'(S, G) changes due to an Assert
  * RPF'(S, G) changes not due to an Assert
  * RPF'(S, G) GenID changes
  
###### (S, G, rpt) Peroidic Message
  (S, G, rpt) Joins and Prunes are **(S, G) Joins and Prunes sent on the RP tree with the RPT bit set**, either to modify the results of (\*, G) Joins, or to override the behavior of other upstream LAN peers.
  
  When a router is going to send a Join(\*, G), it should use the following pseudocode, for each (S, G) for which it has state, to decide whether to include a Prune(S, G, rpt) in the compound Join/Prune message:
  ```c++
  if (SPTbit(S, G) == TRUE){
      // Note: if receiving (S, G) on the SPT, we only prune off shared tree if the RPF neighbor differ
      if (RPF'(*, G) != RPF'(S, G)){
          add Prune(S, G, rpt) to compound message
      }
  } else if (inherited_olist(S, G, rpt) == NULL){
      // Note: all (*, G) olist interfaces received RPT prunes for (S, G)
      add Prune(S, G, rpt) to compound message
  } else if (RPF'(*, G) != RPF'(S, G, rpt)){
      // Note: we joined the shared tree, but there was an (S, G) assert and the source tree RPF neighbor is different
      add Prune(S, G, rpt) to compound message
  }   
  ```
  
  Note that **Join(S, G, rpt)** is normally sent not as a periodic message, but only as a triggered message.
  
###### State Machine for (S, G, rpt) Trigger Messages
  The state machine for (S, G, rpt) trigger message is required pre-(S, G) when there is (\*, G) or (\*, \*, RP) join state at a router and the router or any of its upstream LAN peers wishes to prune S off the RP tree.
  
  There are three states in the state machine:
  * Pruned(S, G, rpt): (\*, G) or (\*, \*, RP(G)) Joined, but (S, G, rpt) pruned
  * NotPruned(S, G, rpt): (\*, G) or (\*, \*, RP(G)) Joined, and (S, G, rpt) not pruned
  * RPTNotJoined(G): neither (\*, G) nor (\*, \*, RP(G)) has been joined
  
  In addition, there is an (S, G, rpt) override timer, OT(S, G, rpt), which is used to delay triggered Join(S, G, rpt) messages to prevent implosions of trigger messages.
  
  Prev State|PruneDesired(S, G, rpt)->True|PruneDesired(S, G, rpt)->False|RPFJoinDesired(G)->False|inherited_olist(S, G, rpt)->non-NULL
  ----------|-----------------------------|------------------------------|------------------------|------------------------------------
  RPTNotJoined(G) (NJ)|->P state|-|-|->NP state
  Pruned(S, G, rpt) (P)|-|NP state(Send Join(S, G, rpt))|NJ state|-
  NotPruned(S, G, rpt) (NP)|->P sate(Send Prune(S, G, rpt);Cancel OT)|-|->NJ state(Cancel OT)|-
  
  **In NotPruned(S, G, rpt) state**:  
  
  Override Timer expires|See Prune(S, G, rpt) to RPF'(S, G, rpt)|See Join(S, G, rpt) to RPF'(S, G, rpt)|See Prune(S, G) to RPF'(S, G, rpt)|RPF'(S, G, rpt)->RPF'(\*, G)
  ----------------------|---------------------------------------|--------------------------------------|----------|-----------------
  Send Join(S, G, rpt);Leave OT unset|OT = min(OT, t_override)|Cancel OT|OT = min(OT, t_override)|OT= min(OT, t_override)
  
  ```c++
  bool RPTJoinDesired(G)
  {
      return ((JoinDesired(*, G)) OR (JoinDesired(*, *, RP(G))))
  }
  
  bool PruneDesired(S, G, rpt)
  {
      return ((RPTJoinDesired(G)) AND 
              ((inherited_olist() == NULL) OR 
               ((SPTbit(S, G) == TRUE) AND (RPF'(*, G) != RPF'(S, G)))))
  }
  ```
  
###### Backgroud: (\*, \*, RP) and (S, G, rpt) Interaction
  We first note that reception of a join(\*, \*, RP) by itself does not cancel (S, G, rpt) prune state on the interface, whereas receiving a Join(\*, G) by itself does cancel (S, G, rpt) prune state on that interface.
  
  The upshot of this is that to prevent forwarding (S, G) on (\*, \*, RP) state, a Prune(S, G, rpt) must be used.
  
  We also note that for historical reasons there is no Assert(\*, \*, RP) message, so any forwarding contention is resolved using Assert(\*, G) messages.
  

  参考文档:
  * [PIM Protocol independent multicast 协议无关组播](http://blog.sina.com.cn/s/blog_5d16ca0c0100bzov.html)  
  * [PIM-SM Multicast Routing Protocol ](https://msdn.microsoft.com/en-us/library/bb742462.aspx)  
  * [Understanding PIM Sparse Mode](https://www.juniper.net/documentation/en_US/junos/topics/concept/multicast-pim-sparse-characteristics.html)
