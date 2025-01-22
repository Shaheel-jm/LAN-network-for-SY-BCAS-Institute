
# LAN-network-for-SY-BCAS-Institute

 <div align="justify"> Designing a Local Area Network (LAN) for the SY-BCAS Institute requires careful consideration of devices to meet the organization’s needs efficiently. The primary objective is to establish a high-performance, reliable, and secure network infrastructure capable of supporting both current and future demands. This involves selecting devices that offer robust performance, advanced security features, scalability, and manageability. By integrating a Cisco 3725 router, HP Aruba switches, and Windows Server 2019, the proposed design ensures seamless connectivity, secure communication, and efficient resource management to support the institute’s administrative and academic activities effectively.
 </div>


## Network Design

<p align="center">
  <img src="./Network%20Design.png" alt="Image description" width="700"/>
</p>
<div align="justify">In the respective network design, there are two ISPs employeed for maintain the realiable connection between branches.</div>

## Network Segmentations
- Software Lab | VLAN - 10 | Network: 192.168.10.0/24
- Network Lab  | VLAN - 20 | Network: 192.168.20.0/24
- General Lab  | VLAN - 30 | Network: 192.168.30.0/24
- Solent Lab   | VLAN - 40 | Network: 192.168.40.0/24
- Access Lab   | VLAN - 50 | Network: 192.168.50.0/24
- Server Room  | VLAN - 60 | Network: 192.168.60.0/24

## Used Hardwares
### Routers
#### Model: Cisco 3725
- For routing purpose I have chosen Cisco 3725 model router, it stands out as the backbone of the network. Because of its advanced routing capabilities, scalability and robust security features. It has efficient data transmission while ensuring with ACL – Access control lists. Moreover with VPN feature in respective router institute can securely connect their branches. Additionally, the inclusion of redundant protocols such as HSRP – Hot Standby Router Protocol and VRRP – Virtual Router Redundancy Protocol, enhance redundancy to provide seamless failover and reliability.

### Switches
#### Model: HP Aruba Switch
- I have chosen HP Aruba switch to connect the end devices. With its efficient STP – Spanning Tree Protocol, LACP – Link Aggregation Control Protocol and VLAN segmentation enhance scalability and ensures fault tolerance by enabling redundancy and load balancing. VLAN segmentation enhance securely and resource allocation.

### Servers
#### Windows Server 2019
- I have chosen “Windows Server 2019” to host critical services like Active Directory, DHCP and DNS. It centralizes resource management, streamlines IT operations and ensures efficient administration. Robust security measures protect sensitive data. And scalability accommodates future needs and guaranteeing a robust, secure and adaptable network infrastructure.

### Network Management System : _Zabbix_
- I have chosen Zabbix as the network monitoring solution to ensure comprehensive oversight of the infrastructure. Zabbix provides real-time monitoring of network devices, servers, and applications, allowing for proactive identification and resolution of potential issues. Its robust alerting system notifies administrators of anomalies, minimizing downtime and enhancing reliability.

## Network Device Configurations
### Router Configuration
* Assigning IP Address for respective subinterface.
```bash
  Pri-Router#configuration terminal 
  Pri-Router(config)#interface FastEthernet0/0.xx
  Pri-Router(config-subif)#encapsulation dot1Q xx
  Pri-Router(config-subif)#ip address 192.168.xx.xx 255.255.255.0
```
Interface f0/0.XX(10) is variable and encapsulation dot1Q XX(10) is same as VLAN number. After that assign IP Address with preferred subnet.

* Hot standby router protocol Configuration
```bash
  Pri-Router(config)#interface FastEthernet0/0.xx
  Pri-Router(config-subif)#standby 10 ip 192.168.10.x
  Pri-Router(config-subif)#standby 10 priority 105
  Pri-Router(config-subif)#standby 10 preempt
```
Select group number for HSRP between **0 – 255**, this number should be same on all routers which are participating in HSRP, in above scenario is “10” and it is variable.

Configure the virtual IP Address for HSRP, set up priority with group number for which router gets higher priority. The here preempt used for regain its active state if it becomes available after being In standby mode.

```bash 
  Pri-Router(config)#interface FastEthernet0/0
  Pri-Router(config-if)#standby 1 track FastEthernet0/1 95
  Pri-Router(config-if)#standby 1 preempt
```
Above statement is used to track the status of other external interface, and allows HSRP to adjust the priority of the router dynamically, based on the status of tracked interface.

* DHCP Relay Configuration
```bash 
  Pri-Router(config)#interface FastEthernet0/0.10
  Pri-Router(config-subif)#ip helper-address 192.168.60.4
```
DHCP relay helps devices like end devices in different network sections to get IP Address from a central server, here according to the requirement DHCP server is placed in Server room. Here **Window Server 2019 (IP: 192.168.60.4) Work as DHCP server**.

* SNMP Configuration for Cisco Router

Simple Network Management Protocol is important for management and monitoring of network devices. It allows to track respective device performance, optimization of network resources and identifying the issues, and this facilitate proactive troubleshooting with real-time alert for network events. And this allows centralized management of network infrastructure moreover this enhance reliability, efficiency and security of network.

```bash 
  Pri-Router(config)# snmp-server community <secret> RO/RW
```
Here <secret> is typically a password for SNMP of the router.

* Site – to – site VPN Configuration
```bash 
  Pri-Router(config)# access-list 100 permit ip 192.168.0.0 0.0.255.255 172.16.0.0 0.0.255.255
  Pri-Router(config)# crypto isakmp policy 101
  Pri-Router(config-isakmp)# authentication pre-share
  Pri-Router(config-isakmp)# encryption aes 256
  Pri-Router(config-isakmp)# group 5
  Pri-Router(config-isakmp)# lifetime 120
  Pri-Router(config-isakmp)# exit
  Pri-Router(config)# crypto ipsec transform-set headTobranch esp-aes 256 esp-sha-hmac
  Pri-Router(cfg-crypto-trans)# exit
  Pri-Router(config)# crypto map BCAS-VPN 101 ipsec-isakmp
  Pri-Router(config-crypto-map)# set peer 172.13.42.234 // IP Address of interface of edge router
  Pri-Router(config-crypto-map)# set transform-set headTobranch
  Pri-Router(config-crypto-map)# pfs group5
  Pri-Router(config-crypto-map)# match address 100
  Pri-Router(config-crypto-map)# exit
  Pri-Router(config)# interface FastEthernet0/1
  Pri-Router(config-if)# crypto map BCAS-VPN  
```
### Switch Configuration
#### Spanning Tree Protocol [STP] Configuration
* Root Swtich Configuration
```bash 
  rootSwt# configuration terminal
  rootSwt(config)# vlan 10,20,30,40,50,60
  rootSwt(config-vlan-< 10,20,30,40,50,60 >)# exit
  rootSwt(config)# spanning-tree mode rpvst
  rootSwt(config)# spanning-tree trap new-root
  rootSwt(config)# spanning-tree priority 0
  rootSwt(config)# spanning-tree vlan 1,10,20,30,40,50,60
  rootSwt(config)# spanning-tree vlan 1,10,20,30,40,50,60 priority 0
  rootSwt(config)# spanning-tree 
  rootSwt(config)# interface 1/1/x
  rootSwt(config-if)# no routing
  rootSwt(config-if)# vlan trunk allowed all
  rootSwt(config-if)# spanning-tree link-type point-to-point
  rootSwt(config-if)# spanning-tree loop-guard
  rootSwt(config-if)# loop-protect
  rootSwt(config-if)# no shutdown
```
* Designated Switch Configuration
```bash 
desgXswitch# configuration terminal
desgXswitch(config)# vlan XX
desgXswitch(config-vlan-XX)# exit
desgXswitch(config)# spanning-tree mode rpvst
desgXswitch(config)# spanning-tree vlan 1,10
desgXswitch(config)# spanning-tree
```
Configuration for switch
```bash 
 desgXswitch(config)# interface 1/1/X
desgXswitch(config-if)# no routing
desgXswitch(config-if)# vlan truck allowed all 
desgXswitch(config-if)# spanning-tree link-type point-to-point
desgXswitch(config-if)# spanning-tree loop-guard
desgXswitch(config-if)# no shutdown
desgXswitch(config-if)# exit
```
Configuration for interface that connects Root and Designated Switch
```bash 
desgXswitch(config)# interface 1/1/X
desgXswitch(config-if)# no routing
desgXswitch(config-if)# vlan XX
desgXswitch(config)# exit
```
Configuration for End Devices

* Assigning IP address for Monitoring Switch
```bash 
switch(config)# interface vlan 1
switch(config-if-vlan)# ip address 192.168.1.10/24
switch(config-if-vlan)# active-gateway ip 192.168.1.1
switch(config-if-vlan)# no shutdown
switch(config-if-vlan)# exit
switch(config)# ip route 0.0.0.0/0 192.168.1.1
```
