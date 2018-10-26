Our high level approach began by more or less implementing features of TCP to improve our protocol.
We decided to organize the receiver's received packets by creating a sequential list of ordered packets
and a dictionary of packets to track the ones we can't add to the list yet.
We did this by keeping track of the sequence number of the last packet in the ordered list.
This allowed us to successfully transmit out of order data and data with duplicates.
We then had to figure out when to retransmit a packet we suspect was dropped.
We did this by setting an RT0 and comparing it to every packet currently "in flight".
If a packet hasn't been acked for more than RTO, we retransmit it.
At this point, we were passing most of the tests aside from the
500ms latency and 50% drop rate ones.
To combat this problem, we decided to dynamically set the RTO based off of the
in flight times of other acked packets. (we ignored when a retransmitted packet was acked
to avoid having extremely short samples from packets we had erroneously assumed were dropped,
when in reality their acks were just late)
This allowed us to pass the 500ms latency test, but not the 50% drop consistently.
We also implemented a form of fast retransmit, which retransmits any unacked packets between a newly acked
section of 3 packets in a row, and the last previously acked packet. This was slightly helpful but I think
it ack'tually didn't make much difference.
In order to handle the 500ms latency case in a speedy manner, we needed a way to slowly scale up the RTO on
retransmissions, because often in the 500ms latency case, between fast retransmissions and the long wait for
acks, all of our packets would get retransmitted, and thus we would treat every sample of RTT as an
invalid sample, and never dynamically update our RTO. To combat this, we implemented a feature
that incremented the RTO whenever a packet was retransmitted, so that it would eventually approach a suitable
timeout that would allow for a packet to be acked without a retransmission.
This caused problems for the 50% drop test, which also ended up doing a lot of retransmissions
But unlike the latency test, the drop test really wanted there to be a low RTO. Our solution to this was
to implement a slow-start-like scaling function for incrementing RTO. While RTO is small, we scale up exponentially
and once we reach a predefined threshold switch to linear scaling. This allowed the drop test to still stay
relatively low on the RTO since it didn't do as many retransmissions as the latency test, and it allowed
the latency test to get to the high RTO it needed, since it would hit the really steep part of the exponential curve.
We tested using the given scripts, usually running ./test as a general test and using netsim and ./run with the --live
tag to debug specific tests that were failing. Our approach to design and work was generally to pick a particular
test that we wanted to pass and then implement the features to pass that test, going along like this one test
at a time, until we had all tests passing.
