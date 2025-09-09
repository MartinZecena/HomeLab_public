# Network
## Network Architecture
- IP Protocol: IPv4 with static addresses, only wifi will use DHCP
- coax cable connected to the ISP router, it will be set to bridge mode so it acts only as a modem
in order to avoid double Firewall and NAT 
- the modem is connected to ETH1 in the router with OPNsense
- the switch is connected to the router using ETH0
- the switch will use 802.1Q Vlan, all ports will be tagged except home network
- OpenVPN server will be available for remote connection
- all the devices described in hardware section are then connected to the switch in the following manner:

```
        port 1: NAS       
        port 2: pc1
        port 3: mirror port 2
        port 4: pc2
        port 5: RaspberryPi
        port 6: Laptop
        port 7: AP
        port 8: router
```


## VLAN and subnets
each vlan has its own subnet for full isolation  
Home:192.168.1.0/24 //(**NAS, internet access, AP)
VLAN10:192.168.10.0/24 //labVLAN (laptop*, pi)[management network]
VLAN20:192.168.20.0/24 //vulnerableVLAN (pc1, mirror)[no internet access]   
VLAN30:192.168.30.0/24//monitoringVLAN 
VLAN40:192.168.40.0/24 //piVLAN(red/blueTeam pi5)[no internet access]


**NAS will also be here, since my old NAS doesnt support VLAN security measures were taken:
deactivation of ssh and firewall rule to block internet trafic

*laptop will have access to two vlans, vlan1 for network management and vlan20 for lab work

![vlan_Configuration](img/vlans.png)



## Network Diagram
```
[Modem/ISP Router: 192.168.0.1] 
[VPN Tunnel: 192.168.5.0/24]
  |
[Router/Firewall(OPNsense):192.168.1.1]<--(trunk port)-->[Managed Switch: 192.168.1.2]
(Vlan Gateways)                                               |
192.168.1.1                                                   |
192.168.10.1                                                  |
192.168.20.1                                                  |
192.168.30.1                                                  |
192.168.40.1                                                  |
  |                                                           |--(Home)--> [NAS: 192.168.1.4]
  |                                                           |            [WiFi AP: 192.168.1.3]
  |                                                           |             DHCP: 192.168.1.100-150
  |                                                           |
  |                                                           |--(VLAN 10)--> [Laptop: 192.168.10.2]
  |                                                           |--(VLAN 20)--> [PC1: 192.168.20.2:8008]
  |                                                           |--(VLAN 30)--> [PC2: 192.168.30.2]
  |                                                           |--(VLAN 40)--> [RaspPi: 192.168.40.2]
  
 ```


 ## Network Configuration
 For the Network setup, multiple steps are necessary:
  - create and setup VLANs on OPNsense
  - create and setup VLANs on switch
  - configure VLANs on each device
  - configure routing tables on each device
  - configure firewall rules


- OPNsense network configuration checklist:
  - Create VLANs in Interfaces -> Devices -> VLANS
  - Assign new interface in interfaces -> assignments (LAN is parent interface)
  - Configure the interfaces in interfaces -> <interface>
  - configure the gateways in system -> gateways -> configuration (disable gateway monitoring)
  - Configure DHCP in Services -> ISC DHCPv4
  - (optional recommended) create aliases for devices and networks
 


- upload the file with scp to proxmox server
  - create vlan with Network Manager
  ```
  sudo nmcli connection add type vlan \
    con-name "<vlan>" \
    ifname <dev>.<vlanId> \
    dev <dev> \
    id <vlanId> \
    ipv4.method manual \
    ipv4.addresses <ip>/24 \
    ipv4.gateway <gateway> \  
    \
  ```


  - create vlan with networking
  ```
  sudo nano /etc/network/interfaces

   iface eth0 inet static
	 address <ip>
	 netmask <ip>
	 gateway <ip>
	 vlan-raw-device
   ```

- start vlan
```
sudo nmcli connection up "<vlan>"
```
- check routing tables
```
netstat -rn
```
- modify routing tables with Network Manager
```
reset default tables:
nmcli conn modify <dev> ipv4.routes “”

add a new static route:
nmcli conn modify <dev> +ipv4.routes “<destination> <gateway>”

```
- modify routing tables with networking

```
  sudo nano /etc/network/interfaces

  after the vlan definition
  post-up ip route add <destination>/<netmask> via <gateway> dev <dev>

```
## Firewall Configuration
In order to allow the communication between the subnets/vlans we need to create firewall rules
for each subnet.
- VLAN10 needs a rule for internet access and DNS.
- All VLANs(except vlan 20) need restricted access to Home Vlan for NAS and internet access.
- VLAN20 has the following rules:
![firewall_rules.png](./img/firewall_rules.png)
  lab -> NAS
  lab -> vulnerable
  lab -> monitoring
  lab -> WAN
  AP  -> WAN

these are the only rules that i have since i dont want the devices in any other VLAN to start communication
with VLAN20 for security reasons.
NOTE: for proper communication between the vlans is necessary to set reply-to: disable
this way the package will take the correct route

## VPN Configuration
OpenVPN configuration checklist:
- create a new user for vpn in system -> access -> users
- create a CA(Certification Authority) System -> Trust -> Authorities
- create client certificate in System -> Trust -> Certificates (common name: username)
- create a server certificate 
- generate an auth. static key in Openvpn -> Instances
- create a server instance
  - protocol: UDP(IPv4)
  - port: 1194

- create a new interface for OpenVPN
- create a Firewall rule to allow access to WAN Address on port 1194 
- create a Firewall rule to allow access from vpn network to Lab network
- create .ovpn file in VPN -> Openvpn -> client export (export type: file only)

## AP Configuration
- make sure DHCP is disabled in OPNsense or that values dont colide with AP DHCP values
- set the DNS to 1.1.1.1 and 8.8.8.8
- create firewall rule to block access to any other critical device on the network

## Security measures taken
- VLANS subnets and IPs where chosen taking functionality and security in account, some limitations in current system (no bridge mode possible, old NAS is not possible to be placed in any VLAN)
- firewall rules only allow traffic:
  lab -> NAS
  lab -> vulnerable
  lab -> monitoring
  lab -> WAN
  AP  -> WAN

  WAN is saddly still connected to the own ISP router/firewall causing double NAT  

- all the passwords created were different using at least 12 characters mixing upper lower case letters numbers and special characters a usb with a password manager was used

- ssh connection was disabled for all devices in lab
- all external communication from NAS was disabled, only through laptop and AP accessible
- only essencial external communication services are enabled on laptop and RaspberryPi
- VPN was disabled

