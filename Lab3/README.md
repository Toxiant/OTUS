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


## PART1: Build the Network and Configure Basic Device Settings<br>

1. R1 config
<details>
  <summary>click for sohw config</summary>
Building configuration...<br>
!<br>
ip dhcp excluded-address 192.168.0.1 192.168.0.5<br>
ip dhcp excluded-address 192.168.0.97 192.168.0.101<br>
!<br>
ip dhcp pool Subnet_A<br>
 network 192.168.0.0 255.255.255.192<br>
 default-router 192.168.0.1 <br>
 domain-name ccna-lab.com<br>
 dns-server 8.8.8.8 <br>
 lease 2 12 30<br>
!<br>
ip dhcp pool R2_Client_LAN<br>
 network 192.168.0.96 255.255.255.240<br>
 default-router 192.168.0.97 <br>
 domain-name ccna-lab.com<br>
 lease 2 12 30<br>
!<br>
no ip domain lookup<br>
ip cef<br>
ipv6 unicast-routing<br>
ipv6 cef<br>
!<br>
interface GigabitEthernet0/0<br>
 ip address 10.0.0.1 255.255.255.252<br>
 duplex auto<br>
 speed auto<br>
 media-type rj45<br>
 ipv6 address FE80::1 link-local<br>
 ipv6 address 2001:DB8:ACAD:2::1/64<br>
!<br>
interface GigabitEthernet0/1<br>
 no ip address<br>
 duplex auto<br>
 speed auto<br>
 media-type rj45<br>
!         <br>
interface GigabitEthernet0/1.100<br>
 description for_Clients_dhcp<br>
 encapsulation dot1Q 100<br>
 ip address 192.168.0.1 255.255.255.192<br>
 ip virtual-reassembly in<br>
 ipv6 address FE80::1 link-local<br>
 ipv6 address 2001:DB8:ACAD:1::1/64<br>
!<br>
interface GigabitEthernet0/1.200<br>
 description for_Management<br>
 encapsulation dot1Q 200<br>
 ip address 192.168.0.65 255.255.255.224<br>
!<br>
interface GigabitEthernet0/1.1000<br>
 description Native<br>
 encapsulation dot1Q 1000 native<br>
!<br>
ip route 0.0.0.0 0.0.0.0 10.0.0.2<br>
!<br>
ipv6 route ::/0 2001:DB8:ACAD:2::2<br>
ipv6 ioam timestamp<br>
!<br>
access-list 1 permit 192.168.0.0 0.0.0.63<br>
</details>

2. R2 config
<details>
  <summary>click for see config</summary>
ipv6 unicast-routing<br>
ipv6 cef<br>
!<br>
interface GigabitEthernet0/0<br>
 ip address 10.0.0.2 255.255.255.252<br>
 duplex auto<br>
 speed auto<br>
 media-type rj45<br>
 ipv6 address FE80::2 link-local<br>
 ipv6 address 2001:DB8:ACAD:2::2/64<br>
!<br>
interface GigabitEthernet0/1<br>
 description for_Clients<br>
 ip address 192.168.0.97 255.255.255.240<br>
 ip helper-address 10.0.0.1<br>
 duplex auto<br>
 speed auto<br>
 media-type rj45<br>
 ipv6 address FE80::1 link-local<br>
 ipv6 address 2001:DB8:ACAD:3::1/64<br>
!<br>
ip route 0.0.0.0 0.0.0.0 10.0.0.1<br>
!<br>
ipv6 route ::/0 2001:DB8:ACAD:2::1<br>
ipv6 ioam timestamp<br>
end<br>
</details>

3. S1 config
<details>
  <summary>click for see config</summary>
hostname S1<br>
!<br>
interface GigabitEthernet0/0<br>
 switchport access vlan 100<br>
 switchport mode access<br>
 negotiation auto<br>
!<br>
interface GigabitEthernet0/1<br>
 switchport trunk allowed vlan 100,200,1000<br>
 switchport trunk encapsulation dot1q<br>
 switchport trunk native vlan 1000<br>
 switchport mode trunk<br>
 negotiation auto<br>
!<br>
interface GigabitEthernet0/2<br>
 switchport access vlan 999<br>
 switchport mode access<br>
 shutdown<br>
 negotiation auto         <br>
!<br>
interface Vlan200<br>
 ip address 192.168.0.66 255.255.255.224<br>
!<br>
ip route 0.0.0.0 0.0.0.0 192.168.0.65<br>
</details>

4. S2 config
<details>
  <summary>click for see config</summary>
hostname S2<br>
!<br>
interface Vlan1<br>
 ip address 192.168.0.98 255.255.255.240<br>
!<br>
ip route 0.0.0.0 0.0.0.0 192.168.0.97<br>
</details>

#### Verify the DHCPv4 Server configuration<br>

>Issue the command ***show ip dhcp pool*** to examine the pool details.<br>
![](show_ip_dhcp_pool.png)<br>

>Issue the command ***show ip dhcp bindings*** to examine established DHCP address assignments.<br>
![](show_ip_dhcp_bindings.png)<br>

>Issue the command ***show ip dhcp server statistics*** to examine DHCP messages.<br>
![](ip_dhcp_server_statistics.png)<br>

>Attempt to acquire an IP address from DHCP on PC-A<br>
![](ping_R1.png)<br>

>Attempt to acquire an IP address from DHCP on PC-B<br>
![](ping_R1_from_PC-B.png)<br>

>Issue the ***show ip dhcp server statistics*** on R2 to verify DHCP messages.<br>
![](show_ip_dhcp_server_statistics_R2.png)<br>

## Lab - Configure DHCPv6

### Addressing table

|Device|Interface|IPv6 Address|
|:-|:-|:-|
|R1|Gi0/0|2001:db8:acad:2::1/64|
|||fe80::1|
|R1|Gi0/1|2001:db8:acad:1::1/64|
|||fe80::1|
|R2|Gi0/0|2001:db8:acad:2::2/64|
|||fe80::2|
||Gi0/1|2001:db8:acad:3::1 /64|
|||fe80::1|
|PC-A|NIC|DHCP|
|PC-B|NIC|DHCP|

## PART2: Verify SLAAC Address Assignment from R1

>Verifing that Host PC-A receives an IPv6 address using the SLAAC method.<br>
![](ipconfig_PC-A.png)<br>

*Where did the host-id portion of the address come from?*<br>
 - Our PC using to random method for generate IPv6 address on default.<br>
First part our address's  comes from <details> <summary>RA mesage (field - Prefix: 2001:db8:acad:1:: ) and second part generete random.</summary> 
Frame 166: 118 bytes on wire (944 bits), 118 bytes captured (944 bits) on interface -, id 0<br>
Ethernet II, Src: 50:00:00:04:00:01 (50:00:00:04:00:01), Dst: IPv6mcast_01 (33:33:00:00:00:01)<br>
Internet Protocol Version 6, Src: fe80::1, Dst: ff02::1<br>
Internet Control Message Protocol v6<br>
    Type: Router Advertisement (134)<br>
    Code: 0<br>
    Checksum: 0xf336 [correct]<br>
    [Checksum Status: Good]<br>
    Cur hop limit: 64<br>
    Flags: 0x00, Prf (Default Router Preference): Medium<br>
    Router lifetime (s): 1800<br>
    Reachable time (ms): 0<br>
    Retrans timer (ms): 0<br>
    ICMPv6 Option (Source link-layer address : 50:00:00:04:00:01)<br>
        Type: Source link-layer address (1)<br>
        Length: 1 (8 bytes)<br>
        Link-layer address: 50:00:00:04:00:01 (50:00:00:04:00:01)<br>
    ICMPv6 Option (MTU : 1500)<br>
        Type: MTU (5)<br>
        Length: 1 (8 bytes)<br>
        Reserved<br>
        MTU: 1500<br>
    ICMPv6 Option (Prefix information : 2001:db8:acad:1::/64)<br>
        Type: Prefix information (3)<br>
        Length: 4 (32 bytes)<br>
        Prefix Length: 64<br>
        Flag: 0xc0, On-link flag(L), Autonomous address-configuration flag(A)<br>
            1... .... = On-link flag(L): Set<br>
            .1.. .... = Autonomous address-configuration flag(A): Set<br>
            ..0. .... = Router address flag(R): Not set<br>
            ...0 0000 = Reserved: 0<br>
        Valid Lifetime: 2592000<br>
        Preferred Lifetime: 604800<br>
        Reserved<br>
        Prefix: 2001:db8:acad:1:: <br>
</details>

## Part 3: Configure and Verify a DHCPv6 server on R1<br>

>Examine the output of *ipconfig /all* and notice the changes.<br>
![](PC-A_DHCPv6.png)<rb>

>Test connectivity by pinging R2’s G0/0/1 interface IP address.<br>
![](pinging_R2.png)<br>

## Part 4-5: Configure a stateful DHCPv6 server on R1 and verify DHCPv6 relay on R2.<br>

>Statefull DHCP ipv6 on PC-B<br>
![](statfull-PC-B.png)

## Lab - Configure DHCPv6

### Addressing table

|Device|Interface|IPv6 Address|
|:-|:-|:-|
|R1|Gi0/0|2001:db8:acad:2::1/64|
|||fe80::1|
|R1|Gi0/1|2001:db8:acad:1::1/64|
|||fe80::1|
|R2|Gi0/0|2001:db8:acad:2::2/64|
|||fe80::2|
||Gi0/1|2001:db8:acad:3::1 /64|
|||fe80::1|
|PC-A|NIC|DHCP|
|PC-B|NIC|DHCP|

>Test connectivity by pinging R1’s G0/0/1 interface IP address.<br>
![](ping-R1.png)<br>
