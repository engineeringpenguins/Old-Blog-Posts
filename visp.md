Virtual ISP – Update 1
======================

Project introduction
--------------------

A friend of mine wanted to create a “pocket internet” as he called it to develop some skills to use in his workplace/career. I am very interested in seeing this through and have been helping out where possible. Our goal was originally to have a virtual environment on his LAN that mimicked a GPON network with customers and ISP. After some thought the project has evolved to me being his upstream “ISP” (over VPN) and his ISP being built with his ideas (completely open source technology). Likely we will have a DMZ in between our ISP’s for things like IPAM/DCIM, Ticketing, network monitoring, etc.

I would like to eventually provide voice and video streams to his ISP. Right now I have modified my MPLS lab so that I can provide L2 and L3 VPN services. I have a pfsense firewall behind the WAN bridge that is setup to only allow LAN traffic from my double NATed router. There is a lot of network complexity here that may need to be scaled back as I have built this with static routes and OSPF neighbor. You can access the MGMT ubuntu desktop from my lan via RDP or ssh and from that machine access the rest of the lab network.

![network topology](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/visp/network-topology-150x150.png)

![pfsense](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/visp/pfsense-150x150.png)

![resources](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/visp/resources-150x150.png)

My next steps are to add services like voice, video, network monitoring, etc. I have deployed netbox in both ISP’s and down the road hope to use automated API calls to keep them in sync.
