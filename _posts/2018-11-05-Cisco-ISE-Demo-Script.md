---
layout: post
title: Cisco ISE Demo Script
description: My Notes preparing for CCIE Security v5
comments: true
---

# Cisco ISE Demo Script

The purpose of this blog entry is to to detail in the simplest wat key ISE functionalities and lay them out so that it could be easily reviewed/demoed.

We will cover the following in the order below. Please note that the some sections could have configuration dependencies from the sections before it.

> At some point I will upload the configs based on each sections. Please feel free to reach out to me meanwhile.


- ##### Basic Wired DOT1.X Authentication (Allow access to deny completely)
- ##### Basic Wired DOT1.X Authentication with Change of VLAN/DACL
- ##### Profiling
- ##### Posture


> **NOTE** that 802.1X Works at the Layer 2 level and there is no IP communication at this level.
Good review of the process here : https://en.wikipedia.org/wiki/IEEE_802.1X


## Basic Wired DOT1.X  Authentication (Allow access to network or deny completely)

In this lab setup the switchport either authorsises the devices if the right credential is provided or completely denies access and the devices gets APIPA IP .


The `dot1.x` configuration flow consists of configuring three main sections.

1. The **Switch** to which the endpoint is connected : AAA and DOT1X related config.
2. The **ISE Server** with the details of the Switch and the end user
3. The **End Point** itself for `dot1.x` authentication


> In the topology below we will configure the **Switch** , **ISE** and the **Win** devices. Very basic connectivity is already setup as show int the topology.
![](/assets/markdown-img-paste-2018110420170354.png)

# Lets start with the Switch
---

**Different types of Host Authentication Modes**

**single-host**  - Exactly one MAC Address

**multi-host** -   One MAC Address opens the door , and rest (other VMs on the Host)  can get in easily without authentication.

**multi-domain** -  Has nothing to do with AD Domain , its about multiple VLANs like voice and data vlan.

**multi-auth**  -  Every single MAC Address has to be authenticated . Even the VMs has to be authenticated .



**The code below is commented and sequenced accordign the steps required.**


```sh
!Make sure a enable password is set
enable secret 5 $1$AGpH$kIw79LdzMFQ395d/


!Enable AAA system
aaa new-model


!Point to ISE
aaa group server radius ISE-group
 server name ISE
!
radius server ISE
 address ipv4 192.168.1.101 auth-port 1812 acct-port 1813
 key **sharedsecret_with_ISE**

!Configure shell login to use enable secret details
aaa authentication login default enable


! Use the Radius Authentication for dot1x
aaa authentication dot1x default group radius

!Authorization is for Dynamic VLANs and ACLs to be assigned
aaa authorization network default group radius

!Default method for account is  RADIUS
aaa accounting dot1x default start-stop group radius


!Include IP Address of the supplicant  IP Address of the suplicant as a part of the request.
radius-server attribute 8 include-in-access-req


!Globally enabling Dot1X Authentication
dot1x system-auth-control


!Default the port on which the endpoint is connected to reset config
default interface GigabitEthernet1/2


!Switchport configuration details
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
! Open mode for testing
 authentication open
! Authentication mode , see above for what each mode means
 authentication host-mode multi-auth
 authentication port-control auto
! Recurring authentication
 authentication periodic
! Let server decide on how often to re-athenticate
 authentication timer reauthenticate server
! Set port access entity as the  autheticator
 dot1x pae authenticator
! Supplicant retry timeout
 dot1x timeout tx-period 10

!Ensure the following for reachability from switch to ISE
interface Vlan1
 ip address 192.168.1.102 255.255.255.0
!

! Some default configs
aaa session-id common
!
```

**Copy Paste Snippet (Modify Here)**

```sh
enable secret cisco123!
aaa new-model
!
aaa group server radius ISE-group
 server name ISE
!
radius server ISE
 address ipv4 X.X.X.X auth-port 1812 acct-port 1813
 key cisco123!
!
aaa authentication login default enable
aaa authentication dot1x default group radius
aaa authorization network default group radius
aaa accounting dot1x default start-stop group radius
!
radius-server attribute 8 include-in-access-req
!
dot1x system-auth-control
!
default interface GigabitEthernetX/X
!
! # In this example we will notice , that if authentication does not
! # success , the port is not able to get into VLAN1 and hence no
! # IP Address is received .
! # show authentication port ..... ... ...
! #
! # Once the right password is supplied , the device gets into VLAN1
! # gets the  IP Address
! #
interface GigabitEthernetX/X
 switchport mode access
 switchport access vlan 1
 spanning-tree portfast
 authentication open
 authentication host-mode multi-auth
 authentication port-control auto
 authentication periodic
 authentication timer reauthenticate server
 dot1x pae authenticator
 dot1x timeout tx-period 10

! Make sure connectivity to ISE
interface Vlan1
 ip address X.X.X.X 255.255.255.0
!
!
aaa session-id common
!
```

**Finally**

```sh
! # Here we actually lock the port down. with "no auth open"
interface GigabitEthernet0/0
 no authentication open
 authentication host-mode single-host
```

# Lets configure ISE now
---

**OPTIONAL Set the ISE Password to be less restrictive**
![](/assets/markdown-img-paste-20181104205150657.png)

**Add the User "bob" with password "cisco"**
![](/assets/markdown-img-paste-20181104205619430.png)

**Add the Switch SW1 with RADIUS "cisco123!"**
This basically lest the switch to communication via RADIUS to ISE
![](/assets/markdown-img-paste-20181104205545490.png)

**Test Authentication**

```sh
debug aaa
test aaa group ISE-group bob cisco new-code
```

# Finally lets enable the PC to do DOT1.X
---

![](/assets/markdown-img-paste-20181104220926256.png)
![](/assets/markdown-img-paste-20181104221039835.png)

# **Troubleshooting Commands**

**PC Troubleshooting**

`services.msc --> Wired Auto Config --> Start`

**Switch Troubleshooting**
```sh
do debug radius authentication
show dot1x all

debug aaa
test aaa group ISE-group bob cisco new-code

show authentication sessions interface gi1/2

```


## Basic Wired DOT1.X  Authentication with Change of VLAN

> **No Additional switch configuration is required for the change of VLAN/DACL**


This builds upon the lab above. Instead of blatenlty denying network access to the user , we put in on a separate VLAN.

1. **Make sure the user is moved under a user group**
![](/assets/markdown-img-paste-20181105230403658.png)


2. **Create and Authorization Profile to set the VLAN to 30 or whatever**
![](/assets/markdown-img-paste-20181105230500743.png)

3. **Set the Policy set (Authorization Policy)**
![](/assets/markdown-img-paste-2018110523072321.png)


### **And that is it!**
Now let the user `bob` login and the switchport vlan should change to the desired VLAN.

## Configuring DACL is pretty much the same as the VLAN change above.

1. **Make the DACL**
![](/assets/markdown-img-paste-20181105232047938.png)
2. **Add the DACL to the Authorization to the Profiles**
![](/assets/markdown-img-paste-20181105231114865.png)
3. **Now re-authenticate the PC and you should see new ACLs on the switch (show ip access-list)**
> Notice the deny to 8.8.8.8
![](/assets/markdown-img-paste-20181105232236124.png)
```sh
Switch# sh authentication sessions interface gigabitEthernet 2/0/1 details

            Interface:  GigabitEthernet2/0/1
          MAC Address:  9457.a5b0.0ade
         IPv6 Address:  Unknown
         IPv4 Address:  Unknown
            User-Name:  94-57-A5-B0-0A-DE
               Status:  Authorized
               Domain:  DATA
       Oper host mode:  multi-auth
     Oper control dir:  both
      Session timeout:  N/A
    Common Session ID:  0A0000640000012B75977C34
      Acct Session ID:  0x00000039
               Handle:  0xE7000033
       Current Policy:  POLICY_Gi2/0/1

Local Policies:

        Service Template: DEFAULT_LINKSEC_POLICY_SHOULD_SECURE (priority 150)

Server Policies:

        ACS ACL:  xACSACLx-IP-PERMIT_ALL_TRAFFIC-57452910

Method status list:

       Method           State
       dot1x            Stopped
       mab              Authc Success

```

4. **Now check the ping front the PC and you should not be able to ping 8.8.8.8 anymore**
![](/assets/markdown-img-paste-20181105232306747.png)


# **To be continued...**


> SNMP Trap
```sh
snmp-server community snmp_ro RO
snmp-server trap-source Vlan10
snmp-server source-interface informs Vlan10
snmp-server enable traps snmp authentication linkdown linkup coldstart warmstart
snmp-server enable traps
snmp-server host 10.10.10.4 version 2c snmp_ro
```
