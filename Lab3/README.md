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

####
