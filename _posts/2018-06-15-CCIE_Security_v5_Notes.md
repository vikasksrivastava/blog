---
layout: post
title: CCIE Security v5 Notes
description: My Notes preparing for CCIE Security v5
comments: true
---


   - [VPN (Policy based )](#vpn-policy-based-)
			- [Key Exchange Protocol](#key-exchange-protocol)
		- [GRE Tunnel](#gre-tunnel)
				- [Step 1](#step-1)
				- [Step 2](#step-2)
		- [GRE over IPSec - Tunnel Mode](#gre-over-ipsec-tunnel-mode)
		- [GRE / IPSec - Transport Mode](#gre-ipsec-transport-mode)
			- [Configuration](#configuration)
		- [Native IPSec Tunnel [S-VTI]](#native-ipsec-tunnel-s-vti)
		- [MGRE (Multipoint GRE)](#mgre-multipoint-gre)
			- [A Multipoint GRE Full Configuration Snippet](#a-multipoint-gre-full-configuration-snippet)
		- [DMVPN (Dynamic Multipoint VPN)](#dmvpn-dynamic-multipoint-vpn)
		- [DMVPN - EIGRP - Phases [I,II,III]](#dmvpn-eigrp-phases-iiiiii)
		- [Redundancy [Dual-Hub DMVPN Setup]](#redundancy-dual-hub-dmvpn-setup)
		- [Encrypting the Tunnel using IPSEC](#encrypting-the-tunnel-using-ipsec)
		- [GETVPN](#getvpn)
		- [VRF](#vrf)
		- [VRF - Aware VPN )Site-to-Site](#vrf-aware-vpn-site-to-site)
		- [VRF Aware [Get VPN]](#vrf-aware-get-vpn)
		- [Routers as a CA Server](#routers-as-a-ca-server)
		- [CA Based VPNs](#ca-based-vpns)
		- [IKEv2 VPNS](#ikev2-vpns)
			- [Troubleshooting Commands and Outputs](#troubleshooting-commands-and-outputs)
			- [Error Messages and Resolution](#error-messages-and-resolution)


### VPN (Policy based )

#### Key Exchange Protocol


For two sides to encrypt or decrypt the traffic , a key needs to be shared between two endpoints.

You need the following to secure a Tunnel :

    - Key
    - Encryption
    - Hashing

> **Diffie Hellman** is the algorith that generates a `KEY` . Lifetime of a DH key is 3600 secs (1hr).

There are two tunnels :
1. `PHASE 1` The **first tunnel** is to exchange the KEY . `ISAKMP` Internet Security Association and Key Managment Protocol is used here .
2. `PHASE 2` The **second tunnel** is for Data transfer. `ESP` Encapsulation Security Payload is used in this phase.

<img src="/assets/markdown-img-paste-20180619143018242.png" alt="Drawing" style="width: 600px;"/>

> Though its not recommened , you can manually setup the `Phase 2` tunnel to use a manual key skipping the `Phase 1` negotiation (without using `ISAKMP`).


**Configuration example**

**1.  Configure the Parameters for `Phase 1`**
```sh
crypto isakmp policy 10
 auth pre-share   //KEY
 encryption 3des  //ENCRYTPION
 hash md5         //HASH
 group 2 // The group command actually generates the hey to be used in the second phase .

crytpto isakmp policy 20
 auth pre-share
 encrytption 3des
 hash sha
 group 2

crypto isakmp key cisco111 address 1.1.1.1
crypto isakmp key cisco111 address 2.2.2.2
```

**2.  Configure the Parameters for `Phase 2` (only encryption and hash , as we have already got the key from phase 1)**

```sh
crypto ipsec tranform-set TSET esp-3des esp-md5

```

**3.  Define traffic that will be encrypted over the Tunnel**

```sh
access-list 101 permit 10.1.1.0 0.0.0.255 10.2.2.0 0.0.0.255
```

**4. Finally Create a Crypto MAP to tie all of the above together**

```sh
crypto map CMAP 5 ipsec-isakmp ! (5 is sequence number , and isakmp means  (get the key from Diffie-Hellman))
match address 101  //access-list
set peer 192.168.1.10
set transform-set TSET // Configured above
```

**5. Apply on the interface**

int fa0/0
 crpypt map CMAP


Always ping with ping source field
show crypto ipsec sa
show crpto isakmp sa


THe tunnel will stay up for :

Phase 1  86400 sec 24 hrs
Phase 2 3600 sec 1 hr

![](/assets/markdown-img-paste-20180619173443458.png)

> In the above VPN Configuration , the interesting traffic is define by an `ACL`. Such VPNS are called Policy based VPN.


### GRE Tunnel

GRE Tunnel basically creates a virtual point to point link between two routers which traditionally were establishing VPN based on interesting traffic define by ACLs . Which was a tedious process.

![](/assets/markdown-img-paste-20180622062800407.png)

Here is the sample configuration of a GRE Tunnel. It is basically a two step process
1. Creat a virtual link between the two routers , in this example R1 and R2

2. Now since they are "virtually" directly connected , you can run a routing protocol to exchange routing information directly within them.

##### Step 1
R1

```sh
interface tun0
 ip add 192.168.1.1 255.255.255.0
 tunnel source 192.1.10.2
 tunnel destination 192.1.20.2

```

R2

```sh
interface tun0
 ip add 192.168.1.2 255.255.255.0
 tunnel source 192.1.20.2
 tunnel destination 192.1.10.2
```


##### Step 2
Now you can run a routing protocol of choice to make them talk

```sh
!
router eigrp 10
 network 10.0.0.0
 network 192.168.1.0
 no auto-summary
!
```

### GRE over IPSec - Tunnel Mode

![](/assets/markdown-img-paste-20180622070147560.png)

In this mode once the GRE tunnel is up , we basically apply the Crypto Map configuration as a `profile` to the tunnel interface (in this e.g IPROF )

> Notice that there is no need to define `match` for interesting traffic or `set-peer` as this is all taken care by the `tunnel0` interface by default as every traffic via the tunnel interface is interesting and the peer (set peer) is known because of GRE. .


```sh
! R1
! Configure Phase I

crypto isakmp policy 10
 auth pre-share
 encryption 3des
 hash md5
 group 2

crypto isakmp key cisco123 address 192.1.20.2


! Configure Phase II

crypto ipsec transform-set TSET esp-3des esp-md5

! Configure an IPSec Profile and attach the transform-set to it

crypto ipsec profile IPROF
 set transform-set TSET

! Assign the IPSec Profile to the Tunnel interface

interface tunnel0
  tunnel protection ipsec profile IPROF

```

```sh
! R2
! Configure Phase I

crypto isakmp policy 10
 auth pre-share
 encryption 3des
 hash md5
 group 2

crypto isakmp key cisco123 address 192.1.10.2


! Configure Phase II

crypto ipsec transform-set TSET esp-3des esp-md5

! Configure an IPSec Profile and attach the transform-set to it

crypto ipsec profile IPROF
 set transform-set TSET

! Assign the IPSec Profile to the Tunnel interface

interface tunnel0
  tunnel protection ipsec profile IPROF

```


Now notice that in the plain simple GRE the packet looked like this , which totaled  **88 bytes**:

```sh
+----+------------+------------+-------+-------------+------------+------+
|GRE | 192.1.10.1 | 192.1.20.3 | EIGRP | 192.168.1.1 | 224.0.0.10 | Data |
+----+------------+------------+-------+-------------+------------+------+
```

With IPSec enable on it (which we did in the config example above) the packet size increase to **140 bytes** because of the ESP header and now looks like this :

```sh
+----------------+-----------------+------------+------------+-------+-------------+------------+------+
|ESP| 192.1.10.X | 192.1.20.X| GRE | 192.1.10.1 | 192.1.20.3 | EIGRP | 192.168.1.1 | 224.0.0.10 | Data |
+----------------+-----------------+------------+------------+-------+-------------+------------+------+
```

> In the above packet , the entire packet including GRE is just a `data` packet (encapsulated in the ESP).

So the **overhead** is that you have increased the size of the packet by 52 bytes (140 - 88 = 52 ) .

`Tunnel Mode` is the default mode for IPSec .

Notice the `tunnel ` in output
```sh
show crypto ipsec sa

outbound esp sas:
 spi: 0x99D47D47(2580839751)
   transform: esp-3des esp-md5-hmac ,
   in use settings ={Tunnel, }
```


Now if you GRE and ESP header are the same , you can run IPSec in `Tranport Mode`

In transport mode , it will remove the duplication of ESP and GRE header and make the packet **smaller** .

---

### GRE / IPSec - Transport Mode

> An efficient mode over Tunnel Mode and saves some overhead on encryption

So with above theory

`ESP| 192.1.10.X | 192.1.20.X| GRE | 192.1.10.1 | 192.1.20.3 | EIGRP | 192.168.1.1 | 224.0.0.10 | Data |`

Changes to

`ESP| 192.1.10.X | 192.1.20.X| GRE | EIGRP | 192.168.1.1 | 224.0.0.10 | Data |`

Above changes saves **16** bytes for  you , so now your packet size becomes 140 - 16 = **124**

#### Configuration

The configuration of the transport mode is similar to the tunnel mode , the only change is highlighted below (`mode transport`)

```sh

crypto ipsec transform-set TSET esp-3des esp-md5
 mode transport

clear crypto sa
```

### Native IPSec Tunnel [S-VTI]

> Also know as Static Virtual TUnnel Interface

Now continuing on to the above flow , is GRE required for the IPSec ? Cant we run the IPSec over the tunnel interface without GRE ?  The answer to that is yes and we basically


```sh
interface tunnel0
 tunnel mode ipsec ipv4
```

So what benefit did the above change provide ?

The packet changed from
`ESP| 192.1.10.X | 192.1.20.X| GRE | EIGRP | 192.168.1.1 | 224.0.0.10 | Data |`

to  (remove of GRE header , 8 bytes)

`ESP| 192.1.10.X | 192.1.20.X| EIGRP | 192.168.1.1 | 224.0.0.10 | Data |`


Above changes saves **8** bytes for  you , so now your packet size becomes 124 - 8 = **116**


### MGRE (Multipoint GRE)

The point to point configuration setting above does not scale with a lot of sites , that is where MGRE comes to rescue.

Now since in GRE we had to define the tunnel destination address: `tunnel destination X.X.X.X`

Since in this case , their could be multiple destinations we need to use a mapping table which maps destination to be reached to the next hop address to be used . This system is called NHRP (Next Hop Resolution Protocol)


`ip nhrp map 192.168.1.1 192.1.10.1`

The above command means that if I want to go to `192.168.1.1` the public IP address for the same is `192.1.10.1`

```sh
interface tunnel0
 ip address 192.168.1.1
 tunnel source 192.168.20.2
 tunnel mode gre multipoint
 ip nhrp network-id 1
 ip nhrp map 192.168.1.1 192.1.10.1
 ip nhrp map 192.168.1.2 192.1.20.2
 ip nhrp map 192.168.1.3 192.1.30.3
```

#### A Multipoint GRE Full Configuration Snippet

![](/assets/markdown-img-paste-20180622171449545.png)

```sh
! R1

interface tunnel0
 ip address 192.168.1.1 255.255.255.0
 tunnel source 192.1.10.2
 tunnel mode gre multipoint
 ip nhrp network-id 1
 ip nhrp map 192.168.1.2 192.1.20.2
 ip nhrp map 192.168.1.3 192.1.30.2
 ip nhrp map 192.168.1.4 192.1.40.2


!R2

interface tunnel0
 ip address 192.168.1.2 255.255.255.0
 tunnel source 192.1.20.2
 tunnel mode gre multipoint
 ip nhrp network-id 1
 ip nhrp map 192.168.1.1  192.1.10.2
 ip nhrp map 192.168.1.3  192.1.30.2
 ip nhrp map 192.168.1.4  192.1.40.2


! R3

interface tunnel0
  ip address 192.168.1.3 255.255.255.0
  tunnel source 192.1.30.2
  tunnel mode gre multipoint
  ip nhrp network-id 1
  ip nhrp map 192.168.1.1 192.1.10.2
  ip nhrp map 192.168.1.2 192.1.20.2
  ip nhrp map 192.168.1.4 192.1.40.2


! R4

interface tunnel0
  ip address 192.168.1.4 255.255.255.0
  tunnel source 192.1.40.2
  tunnel mode gre multipoint
  ip nhrp network-id 1
  ip nhrp map 192.168.1.1 192.1.10.2
  ip nhrp map 192.168.1.2 192.1.20.2
  ip nhrp map 192.168.1.3 192.1.30.2

```

```sh
R1#show ip nhrp
192.168.1.2/32 via 192.168.1.2, Tunnel0 created 00:08:03, never expire
  Type: static, Flags: used
  NBMA address: 192.1.20.2
192.168.1.3/32 via 192.168.1.3, Tunnel0 created 00:08:03, never expire
  Type: static, Flags: used
  NBMA address: 192.1.30.2
192.168.1.4/32 via 192.168.1.4, Tunnel0 created 00:08:03, never expire
  Type: static, Flags: used
  NBMA address: 192.1.40.2
R1#
```


**DISADVANTAGES** In this type of NHRP Based name resolution you need to **statically** define the mapping which is not scalable and **also requires all sites to have static IP Addressing.**

Now this disadvantage was eliminiated by the use of a **Next Hop Server** like a DNS naming server.

**How does the `Next Hop Server` works ?**

Under the inteface configuration on the end routers , we define the Next Hop Servers IP Address. When these interfaces come up they register their information to the Next Hop Server notifying :

"Hey my Tunnel address is `X.X.X.X` and my Public address is `Y.Y.Y.Y` "


**Now the NHS Server has all the mapping of Tunnel IP and the Public IP .**



### DMVPN (Dynamic Multipoint VPN)

**How does the `Next Hop Server` works ?**

Under the inteface configuration on the end routers , we define the Next Hop Servers IP Address. When these interfaces come up they register their information to the Next Hop Server.

![](/assets/markdown-img-paste-20180622171449545.png)

**Step 1.** Enabling the Next Hop Server

```sh
! R1

interface tunnel0
 ip address 192.168.1.1 255.255.255.0
 tunnel source f0/0
 tunnel mode gre multipoint
 ip nhrp network-id 1
 ip nhrp map multicast dynamic // For sending the reply for EIGRP packets received from clients , the NHS Server knows who to send the reply by looking at the dynamic NHRP table.

router eigrp 100
 no auto
 network 10.0.0.0
 network 192.168.1.0
 network 172.16.0.0

```

That's all for configuring a NHS!

**Step 2.** Configuring the Next Hop Client

```sh
! R2

interface tunnel0
 ip address 192.168.1.2 255.255.255.0
 tunnel source f0/1
 tunnel mode gre multipoint
 ip nhrp network-id 1
 ip nhrp nhs 192.168.1.1
 ip nhrp map 192.168.1.1  192.1.10.2 // How do I reach the NHS ? This line basically points every client to the Next Hop Server.
 ip nhrp map multicast 192.1.10.2 // For the EIGRP Packets

 router eigrp 100
  no auto
  network 10.0.0.0
  network 192.168.1.0
  network 172.16.0.0
```

Repeat the above configuration for other Clients on the DMVPN.

```sh
R1#sh ip nhrp
192.168.1.2/32 via 192.168.1.2, Tunnel0 created 00:00:12, expire 01:59:47
  Type: dynamic, Flags: unique registered used
  NBMA address: 192.1.20.2
```

**This completes your DMVP Configuration.**



### DMVPN - EIGRP - Phases [I,II,III]

**Phase I** - With the default configuration of DMVPN only the neighborship is formed between `Hub` and `Spoke` routers but not between `Spoke` and `Spoke` directly.

To resolve this we to turn off split-horizon on the hub.

> **Split Horizon**, dont send the update back on the same interface you learned the route on.


```sh
interface tunnel0
 no ip split-horizon eigrp 100
```

**BEFORE** (without the split-horizon configured)

```sh
R2#sh ip route eigrp
     172.16.0.0/24 is subnetted, 2 subnets
D       172.16.1.0 [90/297372416] via 192.168.1.1, 00:10:13, Tunnel0
     10.0.0.0/24 is subnetted, 2 subnets
D       10.1.1.0 [90/297372416] via 192.168.1.1, 00:10:13, Tunnel0
```

**AFTER** (after the split-horizon configured)

```sh
R2#sh ip route eigrp
     172.16.0.0/24 is subnetted, 4 subnets
D       172.16.4.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
D       172.16.1.0 [90/297372416] via 192.168.1.1, 00:18:44, Tunnel0
D       172.16.3.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
     10.0.0.0/24 is subnetted, 4 subnets
D       10.4.4.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
D       10.3.3.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
D       10.1.1.0 [90/297372416] via 192.168.1.1, 00:18:44, Tunnel0
```
---
**Phase II** (Traffic from Spoke to Spoke goes direct) - Second you would like the traffic to be point to point and not hoping thorough the NHS Router (look at the putput above where all traffic is hopping through 192.168.1.1).

Notice the traffic is going via 192.168.1.1

```sh
R3#traceroute 10.2.2.1

Type escape sequence to abort.
Tracing the route to 10.2.2.1

  1 192.168.1.1 20 msec 20 msec 20 msec
  2 192.168.1.2 32 msec *  48 msec
R3#
```

The resolution of this issue is to ensure that the `next-hop` isnt changed. In the above example

1. `R2` told `R1` about its routes first.
2. Then R1 tells `R3` about its routes , here it changes the `next-hop` for addresss learned from `R2` to itself .


```sh
interface tunnel0
 no ip next-hop-self eigrp 100
```

**BEFORE**  (All traffic going via 192.168.1.1)

```sh
R2#sh ip route eigrp
     172.16.0.0/24 is subnetted, 4 subnets
D       172.16.4.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
D       172.16.1.0 [90/297372416] via 192.168.1.1, 00:18:44, Tunnel0
D       172.16.3.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
     10.0.0.0/24 is subnetted, 4 subnets
D       10.4.4.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
D       10.3.3.0 [90/310172416] via 192.168.1.1, 00:00:08, Tunnel0
D       10.1.1.0 [90/297372416] via 192.168.1.1, 00:18:44, Tunnel0
```

**AFTER**  ( Now , all site specific traffic goes to the specific sites router **without** hopping over 192.168.1.1)

```sh
R2#sh ip route eigrp
     172.16.0.0/24 is subnetted, 4 subnets
D       172.16.4.0 [90/310172416] via 192.168.1.4, 00:00:30, Tunnel0
D       172.16.1.0 [90/297372416] via 192.168.1.1, 00:00:30, Tunnel0
D       172.16.3.0 [90/310172416] via 192.168.1.3, 00:00:30, Tunnel0
     10.0.0.0/24 is subnetted, 4 subnets
D       10.4.4.0 [90/310172416] via 192.168.1.4, 00:00:30, Tunnel0
D       10.3.3.0 [90/310172416] via 192.168.1.3, 00:00:30, Tunnel0
D       10.1.1.0 [90/297372416] via 192.168.1.1, 00:00:30, Tunnel0
```

---
**Phase III** -

In this phase NHRP basically redirects Spokes to reach directly out to other spokes its tryign to reach.

The NHS Server points the spokes to where they are trying to reach and installs a NHRP Cache entry pointing towards that remote spoke in the requesting spoke.

```sh
! HUB
ip nhrp redirect
```

```sh
! SPOKES
ip nhrp shortcut
```

So with this , only the first NHRP resolution request is sent to `R1` and the remaining data flow happens directly

```sh
R4#trace 10.3.3.1

Type escape sequence to abort.
Tracing the route to 10.3.3.1

  1 192.168.1.1 28 msec 40 msec
    192.168.1.3 24 msec
R4#trace 10.3.3.1

Type escape sequence to abort.
Tracing the route to 10.3.3.1

  1 192.168.1.3 32 msec *  40 msec
```


### Redundancy [Dual-Hub DMVPN Setup]

In this case you basically copy the configuration of the existing NHS  , make a new NHS . Point each other NHS with a MAP command and run routing protocol on both .

After this you point your spokes to the additional NHS . Pretty basic stuff .


### Encrypting the Tunnel using IPSEC

```sh
!1. Phase 1

crypto isakmp policy 10
 auth pre-share
 hash md5
 encryption 3des
 crypto isakmp key cisco123 address 0.0.0.0

 !2. Phase 2

 crypto ipsec transform-set TSET esp-3des esp-md5
  mode transport
  exit

!3. IPSec Profile

 crypto ipsec profile PROF
  set transform-set TSET
  exit

!4. Apply the profile to the interface

 interface tunnel 0
  tunnel protection ipsec profile PROF

```

### GETVPN

> GetVPN is a Cisco only solution

GETVPNS are used in a MPLS Private WAN type deployments as the GETVPNs packets cannot be routed over the internet.

Why do we need GETVPNs when we have DMVPN : THe purpose to ket full encryption capabilities while the routing is already setup.

GetVPN copies the inner header onto the outer header.
It only works on fully routed networks. It can potentially work on the Internet but only if you are using Public addresses on the inside .

A multisite IPSec VPN uses a single session key for multiple sites . It has two entities ; `Key Server` and `Group Member`

![](/assets/markdown-img-paste-20180623141229194.png)

> Session key is exchanged in `Phase 1` and used in `Phase 2`

In a normal Phase 1 (ISA) , the only thing exchanges happens is the session key.

**`ISAKMP`** This protocol only echanges `Session Key` . Run on UDP/500
**`GDOI` (Group Domain of Interpretation)** : This Exchanges `Session Key` , `Interesting Traffic ACL`, `Transform Set` and is specially made for GETVPNs. Runs on UDP/848 . This protocol is like an extension for ISAKMP.

**In GETVPNs ,**

**`PHASE 1`** is setup between `Group Member` and `Key Server` .
**`PHASE 2`** is setup between `Group Member` and `Group Member`


> **Prerquisite for GETVPN** : Make sure all networks are able to reach all networks

#### Configuration of a GETVPN

**Step 1.** Configure the Key Server

```sh
! KEY SERVER

! 1. Phase I

crypto isakmp policy 10
 auth pre-share
 hash md5
 enc 3des
 group 2

crypto isakmp key cisco123 address 192.1.10.2
crypto isakmp key cisco123 address 192.1.20.2
crypto isakmp key cisco123 address 192.1.30.2

! 2. Phase II

crypto ipsec transform-set TSET esp-3des esp-sha-hmac

! 3. Configure IPSEC Profile

crypto ipsec profile IPROF
 set transform-set TSET

! 4. Configure Interesting traffic ACL

access-list 101 permit ip 10.1.0.0 0.0.255.255 10.1.0.0 0.0.255.255

! 5. Configure the GDOI Group Configuration

crypto gdoi group SALES
 identity number 111 ! Should match on the Group members
 server local ! I am the key server , so local
  sa ipsec 10
   profile IPROF
   match address ipv4 101
address ipv4 192.1.40.2
```

**Step 1.** Configure the Group Members

```sh
!R1

! 1. Phase I

crypto isakmp policy 10
 auth pre-share
 hash md5
 enc 3des
 group 2

crypto isakmp key cisco123 address 192.1.40.2

! 2. Configure GDOI to point to the Key Server

crypto gdoi group GRP-R1
 identity number 111 ! Should match on the Group members
 server address ipv4 192.1.40.2 ! Point to the key server

! 3. Configure a Crypto MAP

crypto map CMAP 10 gdoi
 set group GRP-R1

! 4. Apply Crypto MAP to the outgoing interface
int f0/0
 crypto map CMAP ! As soon as you do this , the key is downloaded.
```

Once you enter the above configuration on the Group member you will see the following output on the member :

```sh
*Mar  1 01:12:10.919: %CRYPTO-5-GM_REGSTER: Start registration to KS 192.1.40.2 for group GRP-R1 using address 192.1.10.2
*Mar  1 01:12:10.947: %CRYPTO-6-GDOI_ON_OFF: GDOI is ON
*Mar  1 01:12:11.711: %GDOI-5-GM_REGS_COMPL: Registration to KS 192.1.40.2 complete for group GRP-R1 using address 192.1.10.2
```

**Verification Commands**

```sh
R4-KEYSERVER#show crypto gdoi ks members

Group Member Information :

Number of rekeys sent for group SALES : 0

Group Member ID   : 192.1.10.2
Group ID          : 111
Group Name        : SALES
Key Server ID     : 192.1.40.2

Group Member ID   : 192.1.20.2
Group ID          : 111
Group Name        : SALES
Key Server ID     : 192.1.40.2

Group Member ID   : 192.1.30.2
Group ID          : 111
Group Name        : SALES
Key Server ID     : 192.1.40.2

R4-KEYSERVER#show crypto gdoi
GROUP INFORMATION

    Group Name               : SALES (Multicast)
    Group Identity           : 111
    Group Members            : 0
    IPSec SA Direction       : Both
    Active Group Server      : Local
    Group Rekey Lifetime     : 86400 secs
    Rekey Retransmit Period  : 10 secs
    Rekey Retransmit Attempts: 2

      IPSec SA Number        : 10
      IPSec SA Rekey Lifetime: 3600 secs
      Profile Name           : IPROF
      Replay method          : Count Based
      Replay Window Size     : 64
      ACL Configured         : access-list 101

    Group Server list        : Local

```

**Re-keyeing :**

The lifetime of a key is 3600s , the lifetime counter starts when the first group member registers and a key is handed over.

Let's say when a Group Member G1 registers to Key Server KS , the Key K1 is handed over with a count down timer started.

*After 5 minutes (300sec) another Group Member G2 registers , it will be handed over the same key as G1 but the time remaining on it will be 3600-300=3300 secs.*

**OR**

*You can configure RE-KEYING , which basically sends a new key to everyone when a new Group Members join . This can be done over UNICAST or MULTICAST.*


**Re-Keyeing Configuration** (Only done on Key Server)

**Step 1.** Generate a RSA Keypair on the Key Server

```sh
crypto key generate rsa modulus 1024 label GETVPN-KEY
```

**Step 2.** Configre the GDOI group for re-keyeing

```sh
crypto gdoi group SALES
 server local
 rekey transport unicast
 rekey authentication mypubkey rsa GETVPN-KEY
 rekey algorithm 3des-cbc
 rekey lifetime seconds 3600
```

### VRF - A Quick Introduction




### VRF - Aware VPN )Site-to-Site
### VRF Aware [Get VPN]

----
###  Routers as a CA Server
### CA Based VPNs
### IKEv2 VPNS
 ### Using Legacy Menthods
 ### Using S-VTIs

![](assets/markdown-img-paste-20180623213245289.png)
























<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>













<br>


#### Troubleshooting Commands and Outputs


```sh
R1#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id slot status
192.1.20.2      192.1.10.2      QM_IDLE           1001    0 ACTIVE

```

```sh
R1#show crypto ipsec sa

interface: FastEthernet0/0
    Crypto map tag: CMAP, local addr 192.1.10.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (10.1.1.0/255.255.255.0/0/0)
   remote ident (addr/mask/prot/port): (10.2.2.0/255.255.255.0/0/0)
   current_peer 192.1.20.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 5, #pkts encrypt: 5, #pkts digest: 5
    #pkts decaps: 5, #pkts decrypt: 5, #pkts verify: 5
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 5, #recv errors 0

     local crypto endpt.: 192.1.10.2, remote crypto endpt.: 192.1.20.2
     path mtu 1500, ip mtu 1500, ip mtu idb FastEthernet0/0
     current outbound spi: 0x99D47D47(2580839751)

     inbound esp sas:
      spi: 0x890A3034(2299146292)
        transform: esp-3des esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 1, flow_id: SW:1, crypto map: CMAP
        sa timing: remaining key lifetime (k/sec): (4415180/3433)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x99D47D47(2580839751)
        transform: esp-3des esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 2, flow_id: SW:2, crypto map: CMAP
        sa timing: remaining key lifetime (k/sec): (4415180/3433)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE

     outbound ah sas:

     outbound pcp sas:
```

#### Error Messages and Resolution

```sh
*Mar  1 00:42:09.431: %CRYPTO-4-IKMP_BAD_MESSAGE: IKE message from 192.1.20.2 failed its sanity check or is malformed
```
The above warrants a key mismatch .

---

```sh
*Mar  1 01:34:18.575: %DUAL-5-NBRCHANGE: IP-EIGRP(0) 100: Neighbor 192.168.1.3 (Tunnel0) is down: retry limit exceeded
*Mar  1 01:34:21.279: %DUAL-5-NBRCHANGE: IP-EIGRP(0) 100: Neighbor 192.168.1.3 (Tunnel0) is up: new adjacency
*Mar  1 01:34:22.259: %DUAL-5-NBRCHANGE: IP-EIGRP(0) 100: Neighbor 192.168.1.2 (Tunnel0) is down: retry limit exceeded
```

This was caused due to the following commands missing from the DMVPN Clients


```sh
ip nhrp map multicast 192.1.10.2
```
---
