# ASDN-legacy-network

<img width="1052" height="540" alt="image" src="https://github.com/user-attachments/assets/b45a35f1-68ef-41e4-8293-c8517a706b88" />

Starting with VTP which will automatically cascade/sync Vlan accross the other switches.
1. VTP
First we create the VTP
Device: Core-Switch-1
configure terminal
vtp domain Bank
vtp password Cisco
vtp version 2
vtp mode server
vlan 10
name FACULTY
vlan 20
name STUDENT
vlan 100
name EDGE_ROUTERS
vlan 200
name MANAGEMENT

Devices: Core Switch-2, Access Switches-1 and Access Switche-1
configure terminal
vtp domain Bank
vtp password Cisco
vtp version 2
vtp mode client

2. EtherChannel
Devices:  Core Switch-l and Core Switch-2
interface range FastEthernet0/23 - 24
switchport trunk encapsulation dot1q
switchport mode trunk
channel-group 1 mode active
exit
interface Port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 1,10,20,100,200
exit

3. HSRP (First Hop Redundancy Protocol)
Device: Core-Switch-1
interface Vlan10
ip address 192.168.10.2 255.255.255.0
standby 10 ip 192.168.10.1
standby 10 priority 150
standby 10 preempt
exit
interface Vlan20
ip address 192.168.20.2 255.255.255.0
standby 20 ip 192.168.20.1
standby 20 priority 150
standby 20 preempt
exit

Device: Core-Switch-2
interface Vlan10
ip address 192.168.10.3 255.255.255.0
standby 10 ip 192.168.10.1
standby 10 priority 100
exit
interface Vlan20
ip address 192.168.20.3 255.255.255.0
standby 20 ip 192.168.20.1
standby 20 priority 100
exit

4. OSPF routing
Devices: HQ-Rtr-1
router ospf 1
router-id 1.1.1.1
network 172.16.1.0 0.0.0.3 area 0 (Advertisment of VPN Tunnel Network)
network 192.168.100.0 0.0.0.255 area 0 (Advertisment of LAN Network)
redistribute static subnets (Sharing Static Routes to reach VLANs 10,20,200)
exit

Devices: HQ-Rtr-2
router ospf 1
router-id 2.2.2.2
network 172.16.1.0 0.0.0.3 area 0
network 192.168.100.0 0.0.0.255 area 0
redistribute static subnets
exit

Device: Branch-Router
router ospf 1
router-id 3.3.3.3
network 192.168.30.0 0.0.0.255 area 0 (Advertisment of LAN Network)
network 172.16.1.0 0.0.0.3 area 0 (Advertisment of VPN Tunnel(Pri) Network)
network 172.16.2.0 0.0.0.3 area 0 (Advertisment of VPN Tunnel(Sec) Network)
exit
