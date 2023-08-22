### Москва OSPF

|Device|Interface|IP address/net|Type|Area|
|:-|:-|:-|:-|:-|
|R12|Ethernet0/0.100|172.16.0.1/29|nssa|10|
||lo0|10.0.0.12/32|nssa|10|
||Ethernet0/0.200|192.168.0.1/24|nssa|10|
||Ethernet0/2|172.16.0.9/30|nssa|10|
||Ethernet0/3|172.16.0.13/30|nssa|10|
||Ethernet1/0|172.16.0.37/30|nssa|10|
|R13|Ethernet0/0.100|172.16.0.2/29|nssa|10|
||lo0|10.0.0.13/32|nssa|10|
||Ethernet0/2|172.16.0.17/30|nssa|10|
||Ethernet0/3|172.16.0.21/30|nssa|10|
||Ethernet1/0|172.16.0.38/30|nssa|10|
|R14|Ethernet0/0|172.16.0.10/30|nssa|10|
||lo0|10.0.0.14/32|Standard|0|
||Ethernet0/1|172.16.0.22/30|nssa|10|
||Ethernet0/2|100.200.0.1/30|-|-|
||Ethernet0/3|172.16.0.25/30|Totally-Stub|101|
||Ethernet1/0|172.16.0.33/30|Standard|0|
|R15|Ethernet0/0|172.16.0.18/30|nssa|10|
||lo0|10.0.0.15/32|Standard|0|
||Ethernet0/1|172.16.0.14/30|nssa|10|
||Ethernet0/2|100.200.100.1/30|-|-|
||Ethernet0/3|172.16.0.29/30|Standard|102|
||Ethernet1/0|172.16.0.34/30|Standard|0|
|R19|Ethernet0/0|172.16.0.26/30|Totally-Stub|101|
||lo0|10.0.0.19/32|Totally-stub|101|
|R20|Ethernet0/0|172.16.0.30/30|Standard|102|
||lo0|10.0.0.20/32|Standard|102|
<br>
<br>
#### Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.<br>
Создал процесс ospf 1, интерфейсам l0 и eth1/0 маршрутизаторов R14 и R15 назначили созданный процесс ospf c указанием заны 0.<br>
```
R14#
interface Loopback0
 ip address 10.0.0.14 255.255.255.255
 ip ospf 1 area 0
interface Ethernet1/0
 description to R15 eth1/0
 ip address 172.16.0.33 255.255.255.252
 ip ospf 1 area 0
R15#
interface Loopback0
 ip address 10.0.0.15 255.255.255.255
 ip ospf 1 area 0
interface Ethernet1/0
 description to R14 eth1/0
 ip address 172.16.0.34 255.255.255.252
 ip ospf 1 area 0
```
<br>

#### Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.<br>
<br>
На маршрутизаторах R12-R13 все интерфейсы поместили в зону 10 процесса ospf, в процессе ospf маршрутзаторов указали тип nssa<br> 
для тупиковой сети с возможностью приёма маршрута по умолчанию от маршрутизаторов типа ASBR:R14,R15 - для зоны 10.<br>
На маршрутизаторах R14 и R15 прописали маршруты по умолчанию. В процессе OSPF указали трансляцию маршрутов по умолчанию в зону 10.<br> 
Интерфейсы eth0/0 и eth0/1 маршрутизаторов R14-15, поместили в зону 10.<br>
