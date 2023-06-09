## Part:1 Build the Network and Configure Basic Device Settings

>SHEME<br>
![](EVE-Sheme.png)

#### c. Assign a device name to the router.
R1(config)#hostname R1<br>

#### d. Disable DNS lookup to prevent the router from attempting to translate incorrectly entered commands as though they were host names.
R1(config)#no ip domain-lookup<br>

#### e. Assign class as the privileged EXEC encrypted password.
R1(config)#enable secret class<br>

#### f. Assign cisco as the console password and enable login.
R1(config)#line console ?<br>
  <0-0>  First Line number<br>
R1(config)#line console 0<br>
R1(config-line)#password cisco<br>
R1(config-line)#login<br>
  
#### g. Assign cisco as the VTY password and enable login.
R1(config)#line vty 0 4<br>
R1(config-line)#password cisco<br>
R1(config-line)#login<br>

#### h. Encrypt the plaintext passwords.
R1#show run | beg line<br>
line con 0<br>
 password cisco<br>
 login<br>
line aux 0<br>
line vty 0 4<br>
 password cisco<br>
 login<br>
 transport input none<br>
!<br>
no scheduler allocate<br>
!<br>
end<br>

R1(config)#service password-encryption<br>
R1(config)#do show run | beg line<br>
line con 0<br>
 password 7 14141B180F0B<br>
 login<br>
line aux 0<br>
line vty 0 4<br>
 password 7 094F471A1A0A<br>
 login<br>
 transport input none<br>
!<br>
no scheduler allocate<br>
!<br>
end<br>

#### i. Create a banner that warns anyone accessing the device that unauthorized access is prohibited.
R1(config)#banner login $ <br>
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*<br>
\*      Attention! Unauthorized access to this device is prohibited.      \*<br>
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*$

#### j. Save the running configuration to the startup configuration file.
R1#wr<br>

#### k. Set the clock on the router.
R1(config)#clock timezone Moscow +3 0<br>
*Apr  8 20:20:30.623: %SYS-6-CLOCKUPDATE: System clock has been updated from 23:40:30 Moscow Sat Apr 8 2023 to 23:20:30 Moscow Sat Apr 8 2023, configured from console by console.<br>

Completed the same commands for S1 and S2<br>

## Part:2 Create VLANs and Assign Switch Ports

### Addressing Table
|Device|Interface|IP Address|Subnet Mask|Default Gateway|
|:----:|:-------:|:--------:|:---------:|:-------------:|
|R1|G0/0.3|192.168.3.1|255.255.255.0|N/A|
||G0/0.4|192.168.4.1|255.255.255.0|N/A|
||G0/0.8|N/A|N/A|N/A|
|S1|VLAN 3|192.168.3.11|255.255.255.0|192.168.3.1|
|S2|VLAN 3|192.168.3.12|255.255.255.0|192.168.3.1|
|PC-A|NIC|192.168.3.3|255.255.255.0|192.168.3.1|
|PC-B|NIC|192.168.4.3|255.255.255.0|192.168.4.1|

R1#show run<br>
!<br>
interface GigabitEthernet0/0.3<br>
 encapsulation dot1Q 3<br>
 ip address 192.168.3.1 255.255.255.0<br>
!<br>
interface GigabitEthernet0/0.4<br>
 encapsulation dot1Q 4<br>
 ip address 192.168.4.1 255.255.255.0<br>
!<br>
interface GigabitEthernet0/0.8<br>
 encapsulation dot1Q 8<br>

Switch1#show run<br>
!<br>
interface Vlan3<br>
 ip address 192.168.3.11 255.255.255.0<br>
!<br>
ip route 0.0.0.0 0.0.0.0 192.168.3.1<br>

Switch2#show run<br>
!<br>
interface Vlan3<br>
 ip address 192.168.3.12 255.255.255.0<br>
!<br>
ip route 0.0.0.0 0.0.0.0 192.168.3.1<br>


### VLAN Table
|VLAN|Name|Interface Assigned|
|:-|:-|:-|
|3|Management|S1: VLAN 3<br> S2: VLAN 3<br>S1: Gi0/2|
|4|Operations|S2: Gi0/1|
|7|ParkingLot|S1: Gi0/3, Gi1/0, Gi1/1, Gi1/2, Gi1/3<br> S2: Gi0/2, Gi0/3, Gi1/0, Gi1/1, Gi1/2, Gi1/3|
|:8|Native|N/A|

Switch1#show run<br>
!<br>
!<br>
interface GigabitEthernet0/0<br>
 switchport trunk allowed vlan 3,4,8<br>
 switchport trunk encapsulation dot1q<br>
 switchport trunk native vlan 8<br>
 switchport mode trunk<br>
 negotiation auto<br>
!<br>
interface GigabitEthernet0/1<br>
 switchport trunk allowed vlan 3,4,8<br>
 switchport trunk encapsulation dot1q<br>
 switchport trunk native vlan 8<br>
 switchport mode trunk<br>
 negotiation auto<br>
!<br>
interface GigabitEthernet0/2<br>
 switchport access vlan 3<br>
 switchport mode access<br>
 negotiation auto<br>
!<br>
interface GigabitEthernet0/3<br>
 switchport access vlan 7<br>
 switchport mode access<br>
 shutdown<br>
 negotiation auto<br>
!<br>

Switch2#show run<br>
!<br>
interface GigabitEthernet0/0<br>
 switchport trunk allowed vlan 3,4,8<br>
 switchport trunk encapsulation dot1q<br>
 switchport trunk native vlan 8<br>
 switchport mode trunk<br>
 negotiation auto<br>
!<br>
interface GigabitEthernet0/1<br>
 switchport access vlan 4<br>
 switchport mode access<br>
 negotiation auto<br>
!<br>
interface GigabitEthernet0/2<br>
 switchport access vlan 7<br>
 switchport mode access<br>
 shutdown<br>
 negotiation auto<br>

####  Assign VLANs to the correct switch interfaces.
>show vlan<br>
![](show_vlan.png)

## Part:3 Configure an 802.1Q Trunk Between the Switches
>show interfaces trunk<br>
![](show_interfaces_trunk.png)

## Part:4 Configure Inter-VLAN Routing on the Router
>show ip interface brief<br> 
![](show_ip_interface_brief.png)

## Part:4 Verify Inter-VLAN Routing is Working
>Ping from PC-A to its default gateway<br>
![](Ping_from_PC-A_to_its_default_gateway.png)

>Ping from PC-A to PC-B<br>
![](Ping_from_PC-A_to_PC-B.png)

>Ping from PC-A to S2<br>
![](Ping_from_PC-A_to_S2.png)

>From the command prompt on PC-B, issue the tracert command to the address of PC-A.<br>
![](trace_from_PC-B_to_PC-A.png)

What intermediate IP addresses are shown in the results?<br>

The result showed us intermediate IP our gateway.

