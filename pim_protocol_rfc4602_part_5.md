#### PIM Timers
  PIM-SM maintians the following timers, they are countdown timers.
  
  * Per interface (I):
    * Hello Timer: HT(I)
    * Per Neighbor (N):
      * Neignbor Liveness Timer: NLT(N, I)
    * Per Active RP (RP):
      * (\*, \*, RP) Join Expiry Timer: ET(\*, \*, RP, I)
      * (\*, \*, RP) Prune-Pending Timer: PPT(\*, \*, RP, I)
    * Per Group (G):
      * (\*, G) Join Expiry Timer: ET(\*, G, I)
      * (\*, G) Prune-Pending Timer: PPT(\*, G, I)
      * (\*, G) Assert Timer: AT(\*, G, I)
    * Per Source (S):
      * (S, G) Join Expiry Timer: ET(S, G, I)
      * (S, G) Prune-Pending Timer: PPT(S, G, I)
      * (S, G) Assert Timer: AT(S, G, I)
      * (S, G, rpt) Prune Expiry Timer: ET(S, G, rpt, I)
      * (S, G, rpt) Prune-Pending Timer: PPT(S, G, rpt, I)
  * Per Active RP (RP):
    * (\*, \*, RP) Upstream Join Timer: JT(\*, \*, RP)
  * Per Group (G):
    * (\*, G) Upstream Join Timer: JT(\*, G)
    * Per Source (S):
      * (S, G) Upstream Join Timer: JT(S, G)
      * (S, G) Keepalive Timer: KAT(S, G)
      * (S, G, rpt) Upstream Override Timer: OT(S, G, rpt)
  * At the DRs or relevant Assert Winners Only:
    * Per Source, Group pair (S, G):
      * Register-Stop Timer: RST(S, G)
  
#### Timer Values  
  The default value for these are given below:
  
  Value Name|Value|Explanation
  ----------|-----|-----------
  Propagation_delay_default|0.5 secs|Expected propagtion delay over the local link
  t_override_default|2.5 secs|Default delay interval over which to randomize when scheduling a delay Join message
  Hello_Period|30 secs|Periodic interval for Hello messages
  Triggered_Hello_Delay|5 secs|Randomize interval for initial Hello message on bootup or triggered Hello message to a rebooting neighbor
  Default_Hello_Holdtime|3.5 * Hello_Period|Default holdtime to keep neighbor state alive
  Hello_Holdtime|from message|Holdtime from Hello Message Holdtime option
  J/P_HoldTime|from message|Holdtime from Join/Prune Message
  J/P_Override_Interval(I)|Default: Effective_Propagation_Delay(I) + EffectiveOverride_Interval(I)|Short period after a join or prune to allow other routers on the LAN to override the join or prune
  Assert_Override_Interval|Default: 3 secs|Short interval before an assert times out where the assert winner resents an Assert message
  Assert_Time|Deafult: 180 secs|Period after last assert before assert state is timed out
  t_periodic|Default: 60 secs|Period between Join/Prune Message
  t_suppressed|rand(1.1 * t_periodic, 1.4 * t_periodic) when Suppression_Enable(I) is true, 0 otherwise|Supression period when someone else sends a J/P messages so we don't need to do so
  t_override|rand(0, Effective_Override_Interval(I))|Randomized delay to prevent respones implosion when sending a join message to override someone else's Prune message
  t_override (Upstream)|see Upstream Join Timer|See Upstream Join Timer
  Keepalive_Period|Default: 210 secs|Period after last (S, G) data packet during which (S, G) Join state will be maintained even in the absence of (S, G) Join messages
  RP_Keepalive_Period|(3 * Register_Suppression_Time) + Register_Probe_Time|As Keepalive_Period but at the RP when a Register-Stop is Sent
  Register_Suppression_Time|Deafult: 60 secs|Period during which a DR stops sending Register-encapsulated data to the RP after receiving a Register-Stop message
  Register_Probe_Time|Default: 5 secs|Timer before RST expiries when a DR may send a NULL-Register to the RP to cause it to resend a Register-Stop message
  
  t_periodic may be set to take into account such things as configured bandwidth and expected average number of multicast route entries for the attached network or link (e.g., the period would be longer for lower-speed links, or for routers in the center of the network that expect to have a larger number of entries).
  
  If the Join/Prune-Period is modified during operation, these changes should be made relatively infrequently, and the router should continue to refresh at its previous Join/Prune-Period for at least Join/Prune-Holdtime, in order to allow the upstream router to adapt.
  
  
## IANA Considerations
###### PIM Address Family




























  **_128_**
