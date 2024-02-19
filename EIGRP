EIGRP Fundamentals
==================


It wasnt until I took a CCNA practice exam that I realized how many EIGRP questions are on the exam. To better understand some of the basics of Enhanced Interior Gateway Routing Protocol (EIGRP) I wanted to create a simple topology and observe how it behaved. I am using Cisco CSR1000 (IOS XE) to implement EIGRP. Through packet captures and online research I was able to learn more about how this protocol is built.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/eigrp/eigrp-topology-1024x464.png)


Cisco Configuration:


Configure Interfaces

> (config) interface GigabitEthernet1
> 
> ip address 10.0.0.x 255.255.255.252
> 
> no shutdown

Verify

> \# Show ip interface brief

Configure EIGRP

> Configure Terminal
> 
> router eigrp 1738
> 
> network 10.0.0.X

router EIGRP ASN, I have used a random ASN of 1738

Verify

\# show routing protocol

show ip route eigrp


Routing Protocol Type
---------------------

In my line of thinking I have to relate EIGRP to OSPF to get started. OSPF is a Link-State dynamic routing protocol meaning that it relies on having a complete map of the entire network and running calculations and updates so that any one router knows exactly how to get to another router. EIGRP is a Distance-Vector protocol (Cisco calls it a hybrid protocol but it seems to be based on distance vector principles) which means that each router sees itself as part of a relay race and only knows to send traffic to the next hop. What separates EIGRP from an antiquated distance vector routing protocol like RIP is the ability to qualify different metrics for different links and have active backups in the event of link failure.

**Distance-Vector**

**Link-State**

*   Less bandwidth and CPU overhead
*   Relies on Neighbors for information
*   Slower convergence
*   Bad routes can “count to infinity”

*   More bandwidth and CPU calculations
*   Knowledge is universal
*   Faster convergence
*   Bad routes removed before loops occur

While OSPF uses the “Cost” of a link to determine the best route EIGRP uses a “Composite Metric” to determine the optimal path to an endpoint. The Composite Metric is a formula that uses “K” values to calculate the prefered route. By default all values other than Bandwidth (Kbps) and Delay(microseconds) are equal to 0  and dont influence the path determination.

K1 is Bandwidth

K2 is Load

K3 is Delay

K4 and K5 are reliability

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/eigrp/EIGRP-calculation.png)

EIGRP packets: Hello, Update, Query, Reply, Update
--------------------------------------------------

Both OSPF and EIGRP establish neighborship via Hello packets. EIGRP sends all of the composite metrics (K values) in the Hello messages. Instead of using LSA’s to exchange routes EIGRP routers will exchange “full” routes via “Update” packets (unicast). Any future Update packets will be “Partial” routing updates (multicast). If a route times out from the routing table a router will broadcast “Query” packets to all neighbors asking if they know the route. All neighbors will then send “Reply” packets with either a metric of the route (if found) or an “Infinite metric” of the route (if not found). “Acknowledgement” packets are sent by any router that recieves a Update, Query, or Reply to ensure successful transmit/recieve.

Route Calculation
-----------------

EIGRP uses “Feasible Distance” (FD) to determine what would be the “Cost” in OSPF. The FD is the final composite metric or by default the sum of the delay on all Ingress interfaces and the lowest bandwidth interface in the path. In my topology all links are gigabit interfaces so the BW is 1000000Kbps but if I had a 10000Kbps interface then every router after it would report their composite metric as being 10000Kbps.

As each router passes on routing information it sends the “Advertised Distance” (AD) or “Reported Distance” which is the composite metric BEFORE factoring in the router that received it. The ingress router will have to add the AD to the cost of its own ingress interface to create the FD.

You can change the Delay on an interface by going into interface configuration mode and typing:

> delay x

x being the number times 100 that you would like to set. To adjust the bandwidt from interface configuration mode:

> bandwidth x

x being the number in Kbps you want to set the physical interface to.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/eigrp/topology.png)

The second entry (10.0.0.20/30) has an entry in parentheticals: (3072/2816). The number on the left is the Feasible Distance and the number on the right is the Advertised Distance.

EIGRP Routing Table
-------------------

After EIGRP calculates the FD of all possible routes to a destination address the route with the lowest FD will be placed in the routing table. This routing table entry is called a “Successor” as it won the election.

>  # show ip route eigrp

If two routes have equal FD they will both be Successors in the routing table. EIGRP will also have backup routes called “Feasible successors” (FS) that exist in the topology table. Having backup routes available makes EIGRP unique in that even as a Distance Vector protocol it actually has a faster convergence than OSPF because it does not need to run SPF again.

> \# show ip eigrp topology

If you want FS to be listed in the routing table you can adjust the “Variance” by allowing a discrepancy of double triple, etc. of the current FD to be listed. To allow any FD with a value less than double the Successor:

> config router eigrp ASN
> 
> variance 2

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/eigrp/variance.png)

EIGRP has an routing Administrative Distance of 90 which makes it one of the lower/more preferred dynamic routing protocols (OSPF has an Administrative Distance of 110). Higher cost protocols like iBGP (200) can still be preferred over EIGRP if the destination has a longer match on the route in the table. 

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/eigrp/eigrpcost.png)

A route can only be a Feasible Successor if the AD is lower than the FD of the successor.

To explain this another way If Router A wants to get to router D and he is currently going through Router B the route through RouterC would be considered a FS because 50+50=100 is less than 25+100=125. If the link between Router A and C had an AD of 100 then it would not be a FS and removed from the topology table 50+100=150.

EIGRP Advertisement Configuration
---------------------------------

The syntax to advertise a network is very similar to OSPF. 

> (config) router EIGRP ASN
> 
> network A.B.C.D wildcardmask

The difference is you do not specify an instance ID but the Autonamous System Number. However, with EIGRP you could advertise 10.0.1.1/30 in the following ways:

1.  network 10.0.1.1 255.0.0.0
2.  network 10.0.1.1 
3.  network 10.0.1.1 0.0.0.3
4.  network 0.0.0.0

In OSPF you are required to type the reverse mask but in EIGRP you can advertise your network with no mask at all and it will detect the class of address (Class A). You can also advertise it with a bigger mask and it will automatically advertise any networks in that space. The 0.0.0.0 wildcard network that you can use in OSPF is also relevant in EIGRP.

Hello and Dead Timers
---------------------

EIGRP has a Hello timer similar to OSPF where a Hello packet is routinely broadcast in the network. While OSPF has “Dead Timers” EIGRP uses “Hold Timers” which are essentially OSPF’s Hello and Dead timers combined. EIGRP routers will broadcast Hold messages to all routers with a TTL for the origin router. If the neighboring routers do not get another HOLD messages before the TTL expires the route is removed. This is in contrast to OSPF where if the time specified in the dead timer is exceeded by the frequency of Hello messages the route is considered down.

Future Goals
------------

EIGRP is capable of weighting interfaces for load balancing. For instance you can have a ratio of packets per different interfaces (Every 4th packet hits G2 instead of G1).

How EIGRP calculates unequal cost load balancing/Traffic Shaping.
