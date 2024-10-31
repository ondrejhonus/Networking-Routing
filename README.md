# How to configure connections between networks

Let's say we have 3 LANs
### LAN 1 10.0.8.0/22
- VLAN 10 name TEN size 128, has PC0 and PC1
- VLAN 20 name TWENTY size 512, has PC2 and PC3
### LAN 2 192.168.10.0/25
- VLAN 1 size 128, has PC0 and PC4
### LAN 3 192.168.20.0/24
- VLAN 100 size 256, has PC0 and PC4

## How to step by step configure a LAN

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

### 7. Set network addresses to static table

the command for setting a network address is:
```
ip route [remote_network_ip] [rem_net_mask] [net_connection_ip]
```
So if we take that and use it in our example, on router 1 its gonna be:
```
ip route 192.168.10.0 255.255.255.128 100.100.100.102
```

### <p style="color: #ef4b45">8. Do the previous steps for all LANs</p>

### 9. Now you can try pinging between networks
For example from PC0 in LAN 1, ping PC4 in LAN 3
```
ping 192.168.20.10
```