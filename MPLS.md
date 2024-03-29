MPLS Implementation
===================

MPLS Lab
--------

Multi Protocol Label Switching  is a fun network architecture to learn about, I would reccomend anyone to immerse themselves in some kind of lab. It is different than traditional networking because it does not rely on IP routing inside the network but instead adds a “label” (called a shim) in front of the packet that acts as a routing marker. MPLS routers do not get bogged down with the overhead of checking IP routing tables, instead they are incredibly efficient operating at near line speed akin to a layer 2 switch.

The MPLS Shim is located between the Layer 2 header and the layer 3 header. Routers only have to read a fraction of the data to get to the next hop. Every MPLS router has a “forwarding table” that is used to “push, pop, or swap” labels.

/\*! elementor - v3.19.0 - 07-02-2024 \*/ .elementor-widget-image{text-align:center}.elementor-widget-image a{display:inline-block}.elementor-widget-image a img\[src$=".svg"\]{width:48px}.elementor-widget-image img{vertical-align:middle;display:inline-block} ![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/mpls.png)

MPLS can work with layer 2 or layer 3 traffic, can be implemented on copper and fiber networks and is protocol independent  (Ethernet, ATM, Frame Relay, TCP/IP). When the Label Edge Router (LER) gets a ingress packet it will perform a routing lookup. Inside of the MPLS network the LER will create a “Label Switched Path” (LSP) to the destination “Label Switch Router” (LSR) or LER. 

In this lab I have a MPLS network with two FRR Customer Edge Routers connected to Cisco CSR1000’s for the Label Edge Routers which then connect into the MikroTik LSR core.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/pyramid-1024x441.png)

Cisco (IOS XE) configuration:

MikroTik configuration:

Interface Configuration

> (config) interface GigabitEthernet1
> 
> ip address 10.0.1.x 255.255.255.252
> 
> no shutdown
> 
> interface Lo0
> 
> ip address 10.10.10.x 255.255.255.255

Verify

> show ip interface brief
> 
> ping other side of p2p link
> 
> if needed: show running-config

OSPF Configuration

> (config) router ospf 1
> 
> network 10.0.1.x 255.255.255.252 area 0
> 
> network 10.10.10.x 255.255.255.255 area 0

Verify

> \# Show ip route ospf
> 
> show ip ospf neighbor

MPLS Configuration

> (config) mpls ip
> 
> int Gx
> 
> mpls ip

Verify

> show mpls forwarding-table 0
> 
> traceroute will show mpls hops inside the network

Interface Configuration

> /ip address add address 10.0.1.x/30 interface=etherx
> 
> /interface bridge add name=Lo0
> 
> /ip address add address 10.10.10.x/32 interface=Lo0

Verify

> /interface print
> 
> /ip address print
> 
> ping other side of P2P link
> 
> if needed: export

OSPF Configuration

> /router ospf instance add router-id=10.10.10.x
> 
> /router ospf network add area=backbone network=10.0.1.x/30
> 
> /router ospf network add area=backbone network=10.10.10.x/32

Verify

> /router ospf instance print
> 
> /router ospf network print
> 
> /router ospf neighbor print

MPLS Configuration

> /mpls ldp set enable=yes lsr-id=10.10.10.x transport-ip=10.10.10.x
> 
> /mpls ldp interface add interface=etherx

Verify

> /mpls ldp neighbor print
> 
> /mpls ldp interface print
> 
> /mpls ldp print


![MikroTik - Verify Interfaces/Address](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/minterfacec.png)

MikroTik - Verify Interfaces/Address

![Cisco - Verify Interface/Address](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/show-ip-int-br.png)

Cisco - Verify Interface/Address

![Mikrotik - Verify OSPF](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/mospf.png)

Mikrotik - Verify OSPF

![Cisco - Verify OSPF](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/iproute.png)

Cisco - Verify OSPF

![Mikrotik - Verify MPLS](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/ldp-print.png)

Mikrotik - Verify MPLS

![Cisco - Verify MPLS](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/forwarding-table.png)

Cisco - Verify MPLS

![Result: Customer edge](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/certraceroute-1-1024x561.png)

Result: Customer edge

![Result: Provider edge](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/certraceroute-1024x507.png)

Result: Provider edge

At this point MPLS has been enabled via LDP and knows where all other routers are via OSPF. You should be able to traceroute from inside the network and see the MPLS labels but not see them from an external traceroute (see above). From here depending on the customers need you will likely deploy a Layer 2 VPN or a Layer 3 VPN.

Layer 2 VPN
-----------

If you want two remote sites to be on the same LAN you would use a layer 2 VPN. Customer Edge Router 1 will appear to be connected to a network switch/directly connected to CER2.

In our original topology image we have two customer sites connected to an MPLS cloud. The CER’s and LER’s are connected via a layer 2 bridge that carries through the MPLS core. The LAN side of CER1 can utilize any local resources (including pulling an ip via DHCP) on the LAN side of  CER2.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/MPLS/mpls2.0-1024x475.png)

I re organized my existing topology and added redundancy in the LSR core. Our LER’s now have CustB CER’s (Cisco) attached and are forming a L2 VPN.

VPNs at this level are only affected by PER/LER configuration, there is no need to change the LSR’s from here on out. I am going to configure Virtual Private Lan Service on the LER’s. To do this you will have the customer routers tag all outgoing traffic on VLAN 10. Provider routers will create a bridge domain with each other and forward VLAN 10 traffic tagged with CustB’s VPN ID. The end result is that CustB’s routers are on the same network isolated by a VPN multipoint tunnel.

Cisco (IOS XE) Configuration

CER Interface Configuration

> (config) interface GigabitEthernet 1
> 
> no shutdown
> 
> encapsulation dot1Q 10
> 
> interface GigabitEthernet 1.1
> 
> ip address 10.1.1.x 255.255.255.0
> 
> no shut
> 
> interface Loopback0
> 
> ip address 10.10.10.x 255.255.255.0

Verify

> show ip interface brief

CER OSPF Configuration

> (config) router ospf 1
> 
> network 10.1.1.x 0.0.0.255 area 0
> 
> network 10.10.10.x 0.0.0.255 area 0

Verify

The L2 vpn is not set up currently so the CER’s will not be able to share OSPF routes with each other yet.

PER configuration

> (config) interface GigabitEthernetX
> 
> no ip address
> 
> no shutdown
> 
> service instance 1 ethernet
> 
> encapsulation dot1q 10
> 
> bridge-domain 1
> 
> l2 vfi CustB manual
> 
> vpnid 123
> 
> bridge-domain 1
> 
> neighbor 10.10.10.x encapsulation MPLS

Verify

> ping 10.1.1.x
> 
> show ip ospf neighbor

Layer 3 VPN
-----------

If CustA did not want their remote sites to be on the same LAN (security, address management, etc.) they may want a layer 3 solution. With a L3 VPN both sites will be on different IP networks but from their edge router’s perspective the sites will be directly connected (despite having multiple routers in between them). 

I have not been able to find a IGP other than BGP able to support layer 3 VPN tags. The core idea in this lab is that our two Provider Edge Routers will have their customer facing ports peering with each other with MP-BGP (which I think of as “extended/enhanced functionality BGP”). To isolate the traffic we use VRF’s that travel over the vpnv4 address family in MP-BGP. Once we redistribute our OSPF routes into the MP-BGP link and the MP-BGP routes (ospf from both sides) into ospf for the customers, a layer 3 VPN has been established.

Label Edge Router configuration (Cisco IOS XE)

Configure VRF (PER 1&2)

> (config) ip vrf CustA
> 
> rd 1:1
> 
> route-target both 1:1
> 
> interface GigabitEthernet1
> 
> ip vrf forwarding CustA
> 
> ip address 10.0.1.x 255.255.255.252

Verify

> \# show ip vrf int
> 
> ping vrf custA 10.0.1.x

Configure MP-BGP

> (config) router bgp 1738
> 
> neighbor 10.10.10.x remote-as 1738
> 
> neighbor 10.10.10.x update-source lo0
> 
> address-family vpnv4
> 
> neighbor 10.10.10.x activate

Verify

> \# Show BGP vpnv4 unicast all summary

Configure OSPF w/ VRF

> (config) router ospf 2 vrf CustA
> 
> network 10.0.1.x 0.0.0.3

Verify

> \# show ip route vrf CustA

Redistribute OSPF into BGP

> (conf) router bgp 1738 
> 
> address family ipv4 vrf CustA
> 
> redistribute ospf 2

Verify

> \# Show ip bgp vpnv4 vrf CustA

Redistribute BGP into OPSF

> (config) router ospf 2
> 
> redistribute bgp 1738 subnets

Verify

> \# show ip route vrf CustA
> 
> show ip bgp vpnv4 vrf CustA
