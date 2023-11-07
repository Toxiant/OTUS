>Scheme<br>
![](AS_BGP.png)<br>

### 1.Настроите iBGP в офисe Москва между маршрутизаторами R14 и R15.<br>

#### Москва BGP AS 1001<br>

|Device|Interface|IP address/net|network|neighbor ip/AS|
|:-|:-|:-|:-|:-|
|R14|lo0|10.0.0.14/32|14.14.14.0/24|10.0.0.15 1001|
|R14|lo1|14.14.14.1/24|14.14.14.0/24||
|R15|lo0|10.0.0.15/32|15.15.15.0/24|10.0.0.14 1001|
|R15|lo1|15.15.15.1/24|15.15.15.0/24||

 Соседство для iBGP устанавливаем с lo0 маршрутизаторов R14 и R15. Маршруты для сетей интерфейса lo0<br>
внутри нашей AS 1001 анонсируются по протоколу OSPF.  

R14 config BGP Moscow
<details>
  <summary>click for see config</summary>
router bgp 1001<br>
 bgp log-neighbor-changes<br>
 neighbor 10.0.0.15 remote-as 1001<br>
 neighbor 10.0.0.15 update-source Loopback0<br>
 neighbor 100.200.0.2 remote-as 101<br>
 !<br>
 address-family ipv4<br>
  network 14.14.14.0 mask 255.255.255.0<br>
  neighbor 10.0.0.15 activate<br>
  neighbor 10.0.0.15 next-hop-self<br>
  neighbor 100.200.0.2 activate<br>
 exit-address-family<br>
</details>

R15 config BGP Moscow
<details>
  <summary>click for see config</summary>
router bgp 1001<br>
 bgp log-neighbor-changes<br>
 neighbor 10.0.0.14 remote-as 1001<br>
 neighbor 10.0.0.14 update-source Loopback0<br>
 neighbor 100.200.100.2 remote-as 301<br>
 !<br>
 address-family ipv4<br>
  network 15.15.15.0 mask 255.255.255.0<br>
  neighbor 10.0.0.14 activate<br>
  neighbor 10.0.0.14 next-hop-self<br>
  neighbor 100.200.100.2 activate<br>
 exit-address-family<br>
</details>

>R14#show ip bgp summ<br>
![](R14_show_bgp_summ.png)<br>

### 2.Настроите iBGP в провайдере Триада, с использованием RR.<br>

#### Триада AS 520<br>

|Device|Interface|IP address/net|network|neighbor ip/AS|
|:-|:-|:-|:-|:-|
|R23|lo0|10.0.0.23/32|23.23.23.0/24|10.0.0.24 520|
||lo1||23.23.23.1/24||
|R24|lo0|10.0.0.24/32|24.24.24.0/24|10.0.0.23 520|
||lo1|24.24.24.1/24||10.0.0.25 520|
|||||10.0.0.26 520|
|R25|lo0|10.0.0.25/32|25.25.25.0/24|10.0.0.24 520|
||lo1|25.25.25.1/24|100.200.150.28/30||
||||100.200.150.24/30||
|R26|lo0|10.0.0.26/32|26.26.26.0/24|10.0.0.24 520|
||lo1|26.26.26.1/24|100.200.150.20/30||

Соседство для iBGP устанавливаем с lo0 маршрутизаторов R23,R24,R25,R26. Маршруты для сетей интерфейса lo0<br>
внутри нашей AS 520 анонсируются по протоколу IS-IS, R24 назначена роль RouteReflector.

R24 config BGP Triada
<details>
  <summary>click for see config</summary>
router bgp 520<br>
 bgp log-neighbor-changes<br>
 neighbor AS520 peer-group<br>
 neighbor AS520 remote-as 520<br>
 neighbor AS520 update-source Loopback0<br>
 neighbor 10.0.0.23 peer-group AS520<br>
 neighbor 10.0.0.25 peer-group AS520<br>
 neighbor 10.0.0.26 peer-group AS520<br>
 neighbor 100.200.150.14 remote-as 2042<br>
 neighbor 100.200.200.6 remote-as 301<br>
 !<br>
 address-family ipv4<br>
  network 24.24.24.0 mask 255.255.255.0<br>
  neighbor AS520 route-reflector-client<br>
  neighbor AS520 next-hop-self<br>
  neighbor 10.0.0.23 activate<br>
  neighbor 10.0.0.25 activate<br>
  neighbor 10.0.0.26 activate<br>
  neighbor 100.200.150.14 activate<br>
  neighbor 100.200.200.6 activate<br>
 exit-address-family<br>
</details>

R24#show ip bgp sum<br>
![](R24_sh_bgp_sum.png)<br>

### 3.Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.<br>

На R15 назначили через route-map LocalPref 200 с провайдером Ламас для исходящего трафика.<br>
На входящий трафик повлияли путём увеличения as-path для провайдера Китрон - настроили community.<br>

R15 config BGP Moscow
<details>
  <summary>click for see config</summary>
router bgp 1001<br>
 bgp log-neighbor-changes<br>
 neighbor 10.0.0.14 remote-as 1001<br>
 neighbor 10.0.0.14 update-source Loopback0<br>
 neighbor 100.200.100.2 remote-as 301<br>
 !<br>
 address-family ipv4<br>
  network 15.15.15.0 mask 255.255.255.0<br>
  neighbor 10.0.0.14 activate<br>
  neighbor 10.0.0.14 next-hop-self<br>
  neighbor 100.200.100.2 activate<br>
  neighbor 100.200.100.2 route-map MAIN in<br>
 exit-address-family<br>
 !<br>
route-map MAIN permit 10<br>
 set local-preference 200<br>
</details>

R14 config BGP Moscow
<details>
  <summary>click for see config</summary>
router bgp 1001<br>
 bgp log-neighbor-changes<br>
 neighbor 10.0.0.15 remote-as 1001<br>
 neighbor 10.0.0.15 update-source Loopback0<br>
 neighbor 100.200.0.2 remote-as 101<br>
 !<br>
 address-family ipv4<br>
  network 14.14.14.0 mask 255.255.255.0<br>
  neighbor 10.0.0.15 activate<br>
  neighbor 10.0.0.15 next-hop-self<br>
  neighbor 100.200.0.2 activate<br>
  neighbor 100.200.0.2 send-community both<br>
  neighbor 100.200.0.2 route-map COMtoR22 out<br>
 exit-address-family<br>
 !<br>
route-map COMtoR22 permit 10<br>
 set community 1001:103<br>
</details>
 
R22 config BGP Kitorn
<details>
  <summary>click for see config</summary>
router bgp 101<br>
 bgp log-neighbor-changes<br>
 neighbor 100.200.0.1 remote-as 1001<br>
 neighbor 100.200.0.6 remote-as 520<br>
 neighbor 100.200.200.2 remote-as 301<br>
 !<br>
 address-family ipv4<br>
  network 22.22.22.0 mask 255.255.255.0<br>
  neighbor 100.200.0.1 activate<br>
  neighbor 100.200.0.1 route-map COMtoR14 in<br>
  neighbor 100.200.0.6 activate<br>
  neighbor 100.200.200.2 activate<br>
 exit-address-family<br>
 !<br>
ip community-list 1 permit 1001:103<br>
 !<br>
route-map COMtoR14 permit 10<br>
 match community 1<br>
 set as-path prepend last-as 3<br>
</details>

R23#traceroute 14.14.14.1 so 23.23.23.1<br>
![](R23_trace-R14.png)<br>

R14#show ip bgp<br>
![](R14_sh_ip_ngp.png)<br>

### 4.Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.<br>

R18 config BGP SPB
<details>
  <summary>click for see config</summary>
router bgp 2042<br>
 bgp log-neighbor-changes<br>
 network 18.18.18.0 mask 255.255.255.0<br>
 neighbor 100.200.150.13 remote-as 520<br>
 neighbor 100.200.150.17 remote-as 520<br>
 maximum-paths 2<br>
</details>

R18#show ip route bgp<br>
![](R18_sh_ip_ro_bgp.png)<br>


