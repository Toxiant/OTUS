## Lab - Implement DHCPv4<br>

>SCHEME<br>
![](EVE-SchemeDHCPv4.png)

### Addressing Table<br>

|Device|Interface|IP Address|Subnet Mask|Default Gateway|
|:-|:-|:-|:-|:-|
|R1|G0/0|10.0.0.1|255.255.255.252|N/A|
||G0/1|N/A|N/A|N/A|
||G0/1.100|192.168.0.1|255.255.255.192|10.0.0.2|
||G0/1.200|192.168.0.65|255.255.255.224|10.0.0.2|
||G0/1.1000|N/A|N/A||
|R2|G0/0|10.0.0.2|255.255.255.252|N/A|
||G0/1|192.168.0.97|255.255.255.240|10.0.0.1|
|S1|VLAN 200|192.168.0.66|255.255.255.224|192.168.0.65|
|S2|VLAN 1|192.168.0.98|255.255.255.240|192.168.0.97|
|PC-A|NIC|DHCP|DHCP|DHCP|
|PC-B|NIC|DHCP|DHCP|DHCP|

<br>
<br>

### VLAN Table<br>

|VLAN|Name|Interface Assigned|
|:-|:-|:-|
|1|N/A|S2: G0/0|
|100|Clients||S1: G0/0|	
|200|Management|S1: VLAN 200|
|999|Parking_Lot|S1: G0/2-3 G1/0-3|
|1000|Native|N/A|

## PART1: Build the Network and Configure Basic Device Settings

<spoiler title="R1#show run">
  '''
R1#show run
Building configuration...

Current configuration : 4424 bytes
!
! Last configuration change at 07:54:07 UTC Wed Jun 7 2023
!
version 15.9
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!
boot-start-marker
boot-end-marker
!

no logging console
!
no aaa new-model
!
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
ip dhcp excluded-address 192.168.0.97 192.168.0.101
!
ip dhcp pool Subnet_A
 network 192.168.0.0 255.255.255.192
 default-router 192.168.0.1 
 domain-name ccna-lab.com
 dns-server 8.8.8.8 
 lease 2 12 30
!
ip dhcp pool R2_Client_LAN
 network 192.168.0.96 255.255.255.240
 default-router 192.168.0.97 
 domain-name ccna-lab.com
 lease 2 12 30
!
no ip domain lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!
multilink bundle-name authenticated
!
redundancy
!
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:2::1/64
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 media-type rj45
!         
interface GigabitEthernet0/1.100
 description for_Clients_dhcp
 encapsulation dot1Q 100
 ip address 192.168.0.1 255.255.255.192
 ip nat inside
 ip virtual-reassembly in
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
!
interface GigabitEthernet0/1.200
 description for_Management
 encapsulation dot1Q 200
 ip address 192.168.0.65 255.255.255.224
!
interface GigabitEthernet0/1.1000
 description Native
 encapsulation dot1Q 1000 native
!
interface GigabitEthernet0/2
 ip address 10.199.199.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 shutdown 
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 no ip address
 shutdown
 duplex auto
 speed auto
 media-type rj45
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
ip nat inside source list 1 interface GigabitEthernet0/2 overload
ip route 0.0.0.0 0.0.0.0 10.199.199.1
ip route 0.0.0.0 0.0.0.0 10.0.0.2
!
ipv6 route ::/0 2001:DB8:ACAD:2::2
ipv6 ioam timestamp
!
access-list 1 permit 192.168.0.0 0.0.0.63
!
control-plane
!
banner exec ^C
banner motd ^C
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\*      Attention! Unauthorized access to this device is prohibited.      \*
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*^C
!
line con 0
line aux 0
line vty 0 4
 login
 transport input none
!
no scheduler allocate
!
end   
'''
</spoiler>

####
