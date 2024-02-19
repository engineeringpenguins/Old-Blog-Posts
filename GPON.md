GPON Fundamentals
=================

GPON Introduction
-----------------

Gigabit Passive Optical Network’s (GPON) are a relatively economical infrastructure for an Internet Service Provider (ISP) to implement. The “passive” aspect is what separates GPON from a traditional ethernet or wireless deployment. In a Passive Optical Network (PON) there is no electrical equipment between the ISP’s distribution building and their customer’s building. At a very basic level you are running a strand of fiber from one end to the other without regenerating the signal or doing any kind of switching/routing in between.

Physical Equipment
------------------

The goal with any ISP is to get data from your network to a customers router. The specific types of technology may differ but GPON networks will always have the following functionality at some point in their topology.

Router

The router and ISP core/backbone infrastructure are technically a part of the GPON network despite not being passive or using GPON standards to communicate from any devices upstream of router.

Uplink circuits, DHCP, DNS, etc. happen here.

This is also where layer 2 GPON frames come in and are addressed with IP information.

OLT

Optical Line Terminal (OLT): Typically converts Ethernet into GPON traffic. Handles all negotiation, authentication, and distribution with fiber modems.

The different frequencies of light being multiplexed down a single strand of fiber occur here. The most common frequencies in my career have been 1310nm for upload and 1490nm for download. I used to think of these as “red” and “blue” light to distinguish them.

Downstream traffic from the OLT is actually a broadcast using “Time Division Multiplexing” (TDM) which has to do with the nature of the light pulses. This traffic is encrypted with AES and only the modem with the private key is able to decode the data. With no overhead for switching to specific clients TDM is actually has a throughput of over 2GB.

Upstream traffic from the modem to the OLT is p2p using “Time Division Multiple Access” TDMA has much less potential throughput due to having a “time slot” in which it is able to send data to the OLT (similar to a token ethernet topology). The timeslot assigned to the modem is done based on the distance away the OLT has calculated itself from the modem. As this traffic is directed straight at the OLT and no other modems this traffic is unencrypted.

The OLT will authenticate the modems during the initial negotiation process. If duplicate MAC’s are ever detected both modems will be blocked from communicating. This is invaluable for preventing man in the middle attacks as the OLT can detect duplicate MAC addresses in addition to different distances from OLT (different TDMA value).

Splitter

Fiber splitter: essentially uses mirrors to divide one beam of light into many smaller beams of light, similar to a rectangular prism.

Many ISP’s use a 1:32 splitter with some using 1:64. The risk of data loss gets higher the more complex of a fiber splitter is in use. An alternative to using a splitter is to create “splits” in the transport fiber and splice strands off of the main fiber. This introduces significantly more loss than a traditional fiber splitter if not done perfectly.

ONT

Optical Network Terminal (ONT): a fiber modem that decrypts downstream traffic from OLT and converts data into ethernet allowing a subscriber to use the copper ports found on the modem. Many ISP’s provide customers “integrated modems” which means it will work as a home router. The ONT at minimum has one fiber ingress port and one copper egress port to convert GPON to Ethernet and vice versa. Any wifi capability or routing functionality is an addition to the base concept of an ONT.

There are other physical characteristics and technology/devices of the GPON network (like a distribution box or a fiber vault) but the high level design will always have layer 1 and 2 fiber communication with routing and copper devices as bookends.
