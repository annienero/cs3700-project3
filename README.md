Our high level approach began by more or less implementing features of TCP to improve our protocol.
We decided to organize the receiver's received packets by creating a sequential list of ordered packets
and a dictionary of packets to track the ones we can't add to the list yet.
We did this by keeping track of the sequence number of the last packet in the ordered list.
This allowed us to successfully transmit out of order data and data with duplicates.
We then had to figure out when to retransmit a packet we suspect was dropped.
We did this by setting an RTT max and comparing it to every packet currently "in flight".
If a packet hasn't been acked for more than RTT, we retransmit it.
At this point, we were passing most of the tests aside from the
500ms latency and 50% drop rate ones.
To combat this problem, we decided to dynamically set the RTT based off of the
in flight times of other acked packets.
This allowed us to pass the 500ms latency test, but not the 50% drop consistently.

