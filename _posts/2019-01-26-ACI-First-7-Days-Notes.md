---
layout: post
title: First 7 Days ACI
description: First 7 Days ACI
comments: true
---

# Notes on "Your First Seven Days Of ACI" - BRKACI-1001 Ciscolive! Session


---

# Day 1 - Why ACI

 ![](assets/markdown-img-paste-20200126092949307.png)

>**ACI solves the above challenges by providing a simple Leaf/Spine Topology, ECMP which removes the dependency on STP and so on an so forth .**

![](assets/markdown-img-paste-20200126093011812.png)

### ACI Fabric Discovery

![](assets/markdown-img-paste-20200126093627913.png)

All the internal communication betweek the Spines,Leafs and the APIC happens on the Infra IP Address denoted by red T in the picture above. The reachability provided by this Infra Network is then used to deploy the required L2/L2 config wherever needed on the leafs.


![](assets/markdown-img-paste-20200126094243775.png)

`ISIS` : This enables IP reachability between TEPs , the APIC assigns the TEP address. ISIS is automatically established and requires no configuration.

`MP-BGP` : This is L3 Out Configuration . The routes learned via the WAN (L3 Out in the above pic) needs to be "reflected" / "learnt" by other Leafs. Hence the MP-BGP config.  Note that the no manual config is required except for the the assignment of RR (Route Reflector)

---

**Basic Details about the Ports on the APIC and the the Leafs/Spine**

![](assets/markdown-img-paste-20200126094750556.png)

![](assets/markdown-img-paste-20200126094915966.png)

**Each device needs their individual mgmt IP rechability for AAA and NTP**

![](assets/markdown-img-paste-20200126095656484.png)

**Capability to compare configuration**

![](assets/markdown-img-paste-20200126101615741.png)

![](assets/markdown-img-paste-20200126101638584.png)

**Fabric Policies vs Tenant Policies**

`Fabric Policies` : More about Interfaces , Speeds , LLDP
`Tenant Policies` : Its is more about configuration related to EPG/BD/VRF etc

![](assets/markdown-img-paste-20200126101806428.png)


![](assets/markdown-img-paste-20200126102954828.png)

---
## Endpoint
---
**What is an endpoint ?**

It's a combination of MAC address and IP Address.

We can see it on the APIC
![](assets/markdown-img-paste-2020012611072006.png)

We can find more details about the same on the specific Leaf


![](assets/markdown-img-paste-20200126110818755.png)


MAC and /32 IP Address are stored in the `Endpoint Table`
Exception is L3 Out , If we use the same mechanism of learnign all the IP Address , on the MAC address on Nexus router we would have thousand of /32.  Other IP Information is used int he Ip Table just as the normal routing. That is the reason why we use arp table for L3 out.

**ACI Leafs Can Learn via ARP via 2 Methods**

ACI Learns the :

- Source MAC and Source IP during ARP
- A routed frame triggers a Source IP and MAC Address Learning


**Pervasive Gateway**

Pervasive gateway means a local gateway residing on every switch for each subnet on that switch.

![](assets/markdown-img-paste-20200126174516134.png)

**Proxy Routing**

Every endpoint learn by a Leaf is informed to every spine using multicast. This way every SPINE knows about every endpoint in the network.

When Leafs do not know a path to a remote endpoint , they can query the Spine for the same.

![](assets/markdown-img-paste-20200126174645703.png)

---


---

# Day 2 - Infrastructure and Policies

---

---

# Day 3 - Forwarding Overview

---


---

# Day 4 - Network Centric Migrations

---


---

# Day 5 - Multi Location Deployments

---

---

# Day 6 - Troubleshooting Tools

---


---

# Day 7 - Additional Resources

---
