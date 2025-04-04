# How to configure connections between networks

## Table of contents
1. **[LAN Configuration](#how-to-step-by-step-configure-a-lan)**
2. **[Static Routing](#static-routing)**
3. **[OSPF and RIP Configuration](#how-to-configure-ospf-and-rip-on-a-router)**
4. **[SSH Configuration](#how-to-configure-ssh-on-a-router-switch)**
5. **[DHCP Config](#setup-dhcp)**
6. **[NAT Static](#how-to-configure-nat-on-a-static-config)**
7. **[NAT Dyamic](#how-to-configure-nat-on-a-dynamic-config)**
8. **[Port Forwarding](#port-forwarding)**

<hr>

### Let's say we have 3 LANs

### LAN 1 10.0.8.0/22
- VLAN 10 name TEN size 128, has PC0 and PC1
- VLAN 20 name TWENTY size 512, has PC2 and PC3
### LAN 2 192.168.10.0/25
- VLAN 1 size 128, has PC0 and PC4
### LAN 3 192.168.20.0/24
- VLAN 100 size 256, has PC0 and PC4

# How to step by step configure a LAN

### 1. Set IP adresses to all computers

### 2. Add VLANs to your switches and add the PSs to the VLANS

type into your switch for example in LAN 1 for PC0:

    vlan 10

        name TEN

    int fa0/1

        switchport mode access

        switchport access vlan 10

### 3. Set trunk between your switch and router

type into your switch for example in LAN 1:

    int gi0/0

        switchport mode trunk

        switchport access vlan 10,20

### 4. Set subports for a port from your <b>router</b>
type into your router for example in LAN 1 for vlan 20:

    int gi0/0.20

        encapsulation dot1Q

        ip address 192.168.8.1

        no shutdown

### 5. Set your default gateway in your computer. For example PC0 in VLAN 10 in LAN 1 will have a default gateway of <b>10.0.10.1</b>

### 6. Set network adresses to ports in your router
- Let's say that between LAN 1 and LAN 2 we choose a connection with an ip of 100.100.100.100/30
- Router of LAN 1 will have an ip adress of 100.100.100.101/30 and the router of LAN 2 will have an ip of 100.100.100.102/30

Set the gi0/1 to be the port that is connected to the remote router.

    int gi0/1

        ip address 100.100.100.101 255.255.255.252

And in the router on the other side we set the same thing but the other ip address

    int gi0/1

        ip address 100.100.100.102 255.255.255.252

# Static Routing

1. ### Add networks to the static table
```
ip route [remote_network_ip] [rem_net_mask] [net_connection_ip]
```
So if we take that and use it in our example, on router 1 its gonna be:
```
ip route 192.168.10.0 255.255.255.128 100.100.100.102
```

### <p style="color: #ef4b45">8. Do the previous steps for all LANs</p>

### 2. Now you can try pinging between networks
For example from PC0 in LAN 1, ping PC4 in LAN 3
```
ping 192.168.20.10
```

<hr>

# How to configure OSPF and RIP on a router
## RIP:
### 1. Set RIP on the router
```
router rip
```
### 2. Configure local network
```
network 192.168.10.1
```
### 3. Configure remote network
```
network 10.0.10.0
```
## OSPF:
### 1. Set OSPF on the router
> prolly do this on all your routers
```
router ospf [pid]
```
### 2. Set interface to the switch
```
passive-interface gi0/0
```
### 3. Configure local network(s) and use inverted mask
```
network 192.168.10.1 0.0.0.255 area [area_num]
```
### 4. Configure remote network (the connection IP) and use inverted mask
```
network 200.200.200.0 0.0.0.3 area [area_num]
```

<hr>

# How to configure SSH on a router/switch

### 1. Set ip address to vlan on a switch and to a port on a router
Switch:
```
int vlan [number]
    ip address 192.168.10.1 255.255.255.0
    no sh
```
Router:
```
int gi0/0
    ip address 192.168.10.1 255.255.255.0
    no sh
```
### 2. Set hostnames
```
hostname admin
ip domain name admin
```

### 3. Generate crypto key
```
crypto key generate rsa
```

### 4. Set a password and username
Enable password:
```
enable password admin
```
Local password:
```
username admin password admin
```

### 5. Set ssh version
```
connect ssh version 2
```

### 6. Enable VTY line for communication
```
line vty 0 15
transport input ssh
login local
```

### 7. Connect to router/switch via computer
```
ssh -l admin 192.168.10.1
```

# Setup DHCP
### exclude adresses for servers and routers
```
ip dhcp excluded-adress [from_ip] [to_ip]
```
```
ip dhcp pool [vlan]
```
```
network [vlan_ip] [mask]
```
```
default-router [router_ip]
```
### set helper address on second router
#### go into sub port
```
int g0/X.XX
```
```
ip helper-address [ip]
```

# How to configure NAT on a STATIC config
## 1. DO NOT SET A DEFAULT GATEWAY TO YOUR INTERNET SERVER
### 2. Add a **default route** to all external routers
```
ip route 0.0.0.0 0.0.0.0 [next_hop]
```

### 3. Set a NAT direction to the border router (the one connected to the internet)
```
int g0/0
    ip nat [inside/outside]
```

### 4. Turn on the NAT service for translating local adresses
- #### Set the [number] to set the group number that can access the internet
- #### Set g0/x as the port connected to the internet
```
ip nat inside source list [number] interface g0/x overloaded
```

### Set up the access-list (use the [number] from the previous step)
- Basic - 1-99
- Advanced - 100-199
### To DENY the access to internet for a specific address
```
access-list [number] deny [ip_address] [wild_mask]
```
#### example:
```
access-list 33 deny 192.168.0.0 0.0.1.255
```
### To ALLOW the access to internet for the rest of the network
```
access-list [number] permit any
``` 

### Add the internet port to the access group to out
```
int gig0/[internet_port]
    access-group [ACL num] out
``` 

### Add the inside ports to the access group to in
```
int gig0/[neighbor ports]
    access-group [ACL num] in
``` 


# How to configure NAT on a DYNAMIC config
## 1. DO NOT SET A DEFAULT GATEWAY TO YOUR INTERNET SERVER
### 2. Add a **default route** ONLY ON THE BORDER ROUTER
```
ip route 0.0.0.0 0.0.0.0 [Internet_ip]
```

### 3. Set a NAT direction on the border router to all ports (the one connected to the internet)
```
int g0/X
    ip nat [inside/outside]
```


### 4. Send the information to the other routers
```
router ospf [ospf_pid]
```
```
default-information originate
```

### 5. Turn on the NAT service for translating local adresses ON THE BORDER ROUTER
- #### Set the [number] to set the group number that can access the internet
- #### Set g0/x as the port connected to the internet
```
ip nat inside source list [number] interface g0/[internet port num] overloaded
```

### Set up the access-list (use the [number] from the previous step)
- Basic - 1-99
- Advanced - 100-199
### To DENY the access to internet for a specific address
```
access-list [number] deny [ip_address] [wild_mask]
```
#### example:
```
access-list 33 deny 192.168.0.0 0.0.1.255
```
### To ALLOW the access to internet for the rest of the network
```
access-list [number] permit any
```

### Add the internet port to the access group to out
```
int gig0/[internet_port]
    access-group [ACL num] out
``` 

### Add the inside ports to the access group to in
```
int gig0/[neighbor ports]
    access-group [ACL num] in
``` 

# Port forwarding
## We will Allow TPD / UDP ports on the border router to access the e-shop server from the internet
> Step 1., 2., 3. FROM NAT HAVE to be done to do this
## 1. Open a port on the border router that is pointing to the e-shop server
```
ip nat inside source static tcp [source_local_ip] [port] [global_ip]
```
For example for HTTPS:
```
ip nat inside source static tcp 192.168.2.100 443 213.200.13.200 443
```
And for example for HTTP:
```
ip nat inside source static tcp 192.168.2.100 80 213.200.13.200 80
```
#### Useful ports
- HTTPS: 443
- HTTP: 80
- SSH: 22
- FTP: 20 & 21

# Access-list
### A list of rules, that allows and denies network communication
> Useful website: https://samuraj-cz.com/clanek/cisco-ios-8-acl-access-control-list/

### There's already some info on ACL in **[NAT Static](#how-to-configure-nat-on-a-static-config)**, but il prolly repeat it

### Show cmd: ```show ip access-list```

### Rule handling : 
- from top to bottom
- Allow something, deny the rest (or the other way around)

### Where can i apply:
- VLANs
- physical ports
- SSH (```access class <num> in```)
- NAT (```ip nat inside source list <num>```, also **[HERE](#how-to-configure-nat-on-a-static-config)**)
- port-forwarding (**[HERE](#port-forwarding)**)

> info: its ISO 2., 3., 4. layer

### What number to choose:
- 1-99 - basic ACL (limited by source addresses)
- 100-199 - extended ACL (it watches the __source, destination address__ and the __soft port__)

### VLAN (not that important, just do this on the port)
in an L3 switch:
```
vlan access-map NOT-TO-SERVER 10
```

### Physical port ACL
int:
```
int gig0/X
    ip access-group <num> in/out
```
or also in subint:
```
int gig0/X.X
    ip access-group <num> in/out
```

### Port-forwarding ACL
- It's already in the main command there's already a __source, destination address defined__ and also the **port** you're forwarding
