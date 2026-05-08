# ASDN-legacy-network


<img width="996" height="543" alt="image" src="https://github.com/user-attachments/assets/d355d4b9-221c-46ab-bdad-df7625787e44" />

In this lab, we have established the connectivity between Headquarter network and Branch network over GRE VPN tunnel along with the HSRP for gateway redundancy, Etherchannel for Link aggregation and VTP for VLAN management.

Starting with VTP which will automatically cascade/sync Vlan accross the other switches.
```cisco
1. VTP
Device: Core-Switch-1
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

Devices: Core Switch-2, Access Switch-1 and Access Switch-1
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

interface Vlan20
 ip address 192.168.20.2 255.255.255.0
 standby 20 ip 192.168.20.1
 standby 20 priority 150
 standby 20 preempt

interface Vlan100
 ip address 192.168.100.4 255.255.255.0

interface Vlan200
 ip address 192.168.200.2 255.255.255.0
 standby 200 ip 192.168.200.1
 standby 200 priority 150
 standby 200 preempt

Device: Core-Switch-2
interface Vlan10
 ip address 192.168.10.3 255.255.255.0
 standby 10 ip 192.168.10.1

interface Vlan20
 ip address 192.168.20.3 255.255.255.0
 standby 20 ip 192.168.20.1

interface Vlan100
 ip address 192.168.100.5 255.255.255.0

interface Vlan200
 ip address 192.168.200.3 255.255.255.0
 standby 200 ip 192.168.200.1

4. OSPF routing
Devices: HQ-Rtr-1
router ospf 1
router-id 1.1.1.1
network 172.16.1.0 0.0.0.3 area 0 (Advertisment of VPN Tunnel Network)
network 192.168.100.0 0.0.0.255 area 0 (Advertisment of LAN Network)
redistribute static subnets (Sharing Static Routes to reach VLANs 10,20,200)
exit

Device: HQ-Rtr-2
router ospf 1
router-id 2.2.2.2
network 172.16.2.0 0.0.0.3 area 0
network 192.168.100.0 0.0.0.255 area 0
redistribute static subnets
exit

Device: Branch Router
router ospf 1
router-id 3.3.3.3
network 192.168.30.0 0.0.0.255 area 0 (Advertisment of LAN Network)
network 172.16.1.0 0.0.0.3 area 0 (Advertisment of VPN Tunnel(Pri) Network)
network 172.16.2.0 0.0.0.3 area 0 (Advertisment of VPN Tunnel(Sec) Network)
exit

5 VPN (GRE over IPsec)
Device: HQ-Rtr-1 and HQ-Rtr-2
crypto isakmp policy 10
encr aes 256
authentication pre-share
group 5

crypto isakmp key CISCO123 address 200.2.2.2
crypto ipsec transform-set MY_SET esp-aes 256 esp-sha-hmac

crypto map MY_MAP 10 ipsec-isakmp
set peer 200.2.2.2
set transform-set MY_SET
match address 120

Device: HQ-Rtr-1
access-list 120 permit gre host 100.1.1.1 host 200.2.2.2
crypto map MY_MAP 10 ipsec-isakmp
set peer 200.2.2.2
set transform-set MY_SET
match address 120

interface Tunnel0
ip address 172.16.1.1 255.255.255.252
mtu 1476
tunnel source GigabitEthernet0/0/0
tunnel destination 200.2.2.2

interface GigabitEthernet0/0/0
crypto map MY_MAP


Device: HQ-Rtr-2
access-list 120 permit gre host 100.1.2.1 host 200.2.2.2
crypto map MY_MAP 10 ipsec-isakmp
set peer 200.2.2.2
set transform-set MY_SET
match address 120

interface Tunnel0
ip address 172.16.2.1 255.255.255.252
mtu 1476
tunnel source GigabitEthernet0/0/0
tunnel destination 200.2.2.2

interface GigabitEthernet0/0/0
crypto map MY_MAP


Device: Branch Rtr
crypto isakmp policy 10
encr aes 256
authentication pre-share
group 5

crypto isakmp key CISCO123 address 100.1.1.1
crypto isakmp key CISCO123 address 100.1.2.1

crypto ipsec transform-set MY_SET esp-aes 256 esp-sha-hmac

crypto map MY_MAP 10 ipsec-isakmp 
set peer 100.1.1.1
set transform-set MY_SET 
match address 120

crypto map MY_MAP 20 ipsec-isakmp 
set peer 100.1.2.1
set transform-set MY_SET 
match address 121

access-list 120 permit gre host 200.2.2.2 host 100.1.1.1
access-list 121 permit gre host 200.2.2.2 host 100.1.2.1

interface Tunnel1
ip address 172.16.1.2 255.255.255.252
mtu 1476
tunnel source GigabitEthernet0/0/0
tunnel destination 100.1.1.1

interface Tunnel2
ip address 172.16.2.2 255.255.255.252
mtu 1476
tunnel source GigabitEthernet0/0/0
tunnel destination 100.1.2.1

interface GigabitEthernet0/0/0
crypto map MY_MAP
