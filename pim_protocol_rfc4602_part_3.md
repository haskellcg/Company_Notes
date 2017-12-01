#### PIM Assert Message
  Where multiple PIM routers peer over a shared LAN, it is possible for more than one upstream router to have valid forwarding state for a packet, **which can lead to packet duplication**. PIM does not attempt to prevent this from occurring. Instead, **it detects when this has happened and elects a single forwarder amongst the upstream routers to prevent further duplication**.
   
  This election is performed using PIM Assert Message. Assert messages are also received by downstream routers on the LAN, and these cause subsequent Join/Prune messages to be sent to the upstream router that won the Assert.

  In general, a PIM Assert messages should only be accepted for processing if it comes from a known PIM neighbor. In addition, if the Hello message from a neighbor was authenticated using the IPsec Authentication Header (AH), then all Assert messsages from that neighbor MUST also be authenticated using IPsec AH.
  
###### (S, G) Assert Message State Machine
  There are three states:
  * NoInfo (NI)
  * I am Assert Winner (W)
  * I am Assert Loser (L)
  * Assert Timer (AT)

###### (\*, G) Assert Message State Machine
  There are three states:
  * NoInfo (NI)
  * I am Assert Winner (W)
  * I am Assert Loser (L)
  * Assert Timer (AT)

###### Assert Metrics
  Assert metrics are defined as:
  ```c++
  struct assert_metric{
      rpt_bit_flag;
      metric_preference;
      route_metric;
      ip_address
  };
  ```
  
  An assert metric for (S, G) to include in (or compare against) an Assert message sent on interface I should be computed using the following pseudocode:
  ```c++
  assert_metric my_assert_metric(S, G, I)
  {
      if (CouldAssert(S, G, I) == TRUE){
          return spt_assert_metric(S, I)
      } else if (CouldAssert(*, G, I) == TRUE){
          return rpt_assert_metric(S, I)
      } else {
          return infinite_assert_metric()
      }
  }
  
  assert_metric spt_assert_metric(S, I)
  {
      return {0, MRIB.pref(S), MRIB.metric(S), my_ip_address(I)}
  }
  
  assert_metric rpt_assert_metric(G, I)
  {
      return {1, MRIB.pref(RP(G)), MRIB.metric(RP(G)), my_ip_address(I)}
  }
  
  assert_metric inifinite_assert_metric()
  {
      return {1, infinity, infinity, 0}
  }
  ```
  
###### AssertCancel Messages  
  An AssertCancel message is simply **an RPT Assert message but with infinite metric**. It is sent by the assert winner when it deletes the forwarding state that had caused the assert to occur. It will cause any other router that has forwarding state to send its own assert, and to take over forwarding.
  
  An AssertCancel(S, G) is an infinite metric assert with the RPT bit set that names S as the source.
   
  An AssertCancel(\*, G) is an infinite metric assert with the RPT bit set that the source set to zero.
   
###### Assert State Macros   
  ```c++
  bool last_assert(S, G, rpt, I)
  {
      if (RPF_interface(RP(G)) == I OR
          (RPF_interface(S) == I AND SPTbit(S, G) == TRUE)){
          return FALSE
      } else {
          return ((AssertWinner(S, G, I) != NULL) AND
                  (AssertWinner(S, G, I) != me))
      }
  }
  
  bool lost_assert(S, G, I)
  {
      if (RPF_interface(S) == I){
          return FALSE
      } else {
          return (AssertWinner(S, G, I) != NULL AND
                  AssertWinner(S, G, I) != me AND
                  (AssertWinnerMetrics(S, G, I) is better than spt_assert_metric(S, I)))
      }
  }
  
  bool lost_assert(*, G, I);
  ```
  
  Summary of Assert Rules and Rationale
   
