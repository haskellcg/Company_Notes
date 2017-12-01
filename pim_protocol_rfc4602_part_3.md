#### PIM Assert Message
   Where multiple PIM routers peer over a shared LAN, it is possible for more than one upstream router to have valid forwarding state for a packet, **which can lead to packet duplication**. PIM does not attempt to prevent this from occurring. Instead, **it detects when this has happened and elects a single forwarder amongst the upstream routers to prevent further duplication**.
   
   This election is performed using PIM Assert Message. Assert messages are also received by downstream routers on the LAN, and these cause subsequent Join/Prune messages to be sent to the upstream router that won the Assert.
