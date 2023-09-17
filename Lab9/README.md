>Scheme<br>
![](AS_BGP.png)<br>

### Москва BGP AS 1001<br>

|Device|Interface|IP address/net|network|neighbor ip/AS|
|:-|:-|:-|:-|:-|
|R14|Ethernet0/2|100.200.0.1/30|14.14.14.0/24|100.200.0.2 101|
|R15|Ethernet0/2|100.200.100.1/30|15.15.15.0/24|100.200.100.2 301|

### Киторн  AS 101<br>

|Device|Interface|IP address/net|network|neighbor ip/AS|
|:-|:-|:-|:-|:-|
|R22|Ethernet0/0|100.200.0.2/30|22.22.22.0/24|100.200.0.1 1001|
||||100.200.0.0/24||
||Ethernet0/1|100.200.200.1/30|22.22.22.0/24|100.200.200.2 301|
||||100.200.0.0/24||

### Ламас AS 301<br>

|Device|Interface|IP address/net|network|neighbor ip/AS|
|:-|:-|:-|:-|:-|
|R21|Ethernet0/0|100.200.100.1/30|21.21.21.0/24|100.200.100.1 1001|
||||100.200.100.0/24||
||||100.200.200.0/24||
||Ethernet0/1|100.200.200.2/30|21.21.21.0/24|100.200.200.1 101|
||||100.200.100.0/24||
||||100.200.200.0/24||
||Ethernet0/2|100.200.200.6/30|21.21.21.0/24|100.200.200.5 520|
||||100.200.100.0/24||
||||100.200.200.0/24||

### Триада AS 520<br>

|Device|Interface|IP address/net|network|neighbor ip/AS|
|:-|:-|:-|:-|:-|
|R24|Ethernet0/0|100.200.200.5/30|24.24.24.0/24|100.200.200.6 301|
||||100.200.150.12/30||
||Ethernet0/3|100.200.150.13/30|24.24.24.0/24|100.200.150.14 2042|
||||100.200.150.12/30||

### Санкт-Петербург AS 2042<br>

|Device|Interface|IP address/net|network|neighbor ip/AS|
|:-|:-|:-|:-|:-|
|R18|Ethernet0/2|100.200.150.14/30|18.18.18.0/24|100.200.150.13 520|

##### Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.<br>

На маршрутизаторах R14,R15,R21,R22 инициируем процесс bgp с указанием соответвующих AS,<br>
предварительно добавиv статический маршрут в Null0 на всех роутерах для сетей которые<br> 
хотим анонсировать по bgp<br>

R14 config BGP Moscow
<details>
  <summary>click for see config</summary>
router bgp 1001<br>
 bgp log-neighbor-changes<br>
 neighbor 100.200.0.2 remote-as 101<br>
 !<br>
 address-family ipv4<br>
  network 14.14.14.0 mask 255.255.255.0<br>
  neighbor 100.200.0.2 activate<br>
 exit-address-family<br>
</details>

R15 config BGP Moscow
<details>
  <summary>click for see config</summary>
router bgp 1001<br>
 bgp log-neighbor-changes<br>
 neighbor 100.200.100.2 remote-as 301<br>
 !<br>
 address-family ipv4<br>
  network 15.15.15.0 mask 255.255.255.0<br>
  neighbor 100.200.100.2 activate<br>
 exit-address-family<br>
</details>


##### Настроите eBGP между провайдерами Киторн и Ламас.<br>

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
  network 100.200.0.0 mask 255.255.255.0<br>
  neighbor 100.200.0.1 activate<br>
  neighbor 100.200.0.6 activate<br>
  neighbor 100.200.200.2 activate<br>
 exit-address-family<br>
</details>

R21 config BGP Lamas
<details>
  <summary>click for see config</summary>
router bgp 301<br>
 bgp log-neighbor-changes<br>
 neighbor 100.200.100.1 remote-as 1001<br>
 neighbor 100.200.200.1 remote-as 101<br>
 neighbor 100.200.200.5 remote-as 520<br>
 !<br>
 address-family ipv4<br>
  network 21.21.21.0 mask 255.255.255.0<br>
  network 100.200.100.0 mask 255.255.255.0<br>
  network 100.200.200.0 mask 255.255.255.0<br>
  neighbor 100.200.100.1 activate<br>
  neighbor 100.200.200.1 activate<br>
  neighbor 100.200.200.5 activate<br>
 exit-address-family<br>
</details>

##### Настроите eBGP между Ламас и Триада.<br>

R24 config BGP Triada
<details>
  <summary>click for see config</summary>
router bgp 520<br>
 bgp log-neighbor-changes<br>
 neighbor 100.200.150.14 remote-as 2042<br>
 neighbor 100.200.200.6 remote-as 301<br>
 !<br>
 address-family ipv4<br>
  network 24.24.24.0 mask 255.255.255.0<br>
  network 100.200.150.12 mask 255.255.255.252<br>
  neighbor 100.200.150.14 activate<br>
  neighbor 100.200.200.6 activate<br>
 exit-address-family<br>
</details> 

##### Настроите eBGP между офисом С.-Петербург и провайдером Триада.<br>

R18 config BGP SPB
<details>
  <summary>click for see config</summary>
router bgp 2042<br>
 bgp log-neighbor-changes<br>
 network 18.18.18.0 mask 255.255.255.0<br>
 neighbor 100.200.150.13 remote-as 520<br>
<details>

##### Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.<br>

После настройки и запуска процесса bgp на роутерах:R14,R15,R18,R21,R22,R24 - появилась ip связанность<br>
между роутерами офиса Москвы и С.-Петербурга.<br>

![](traceroute.png)<br>

На скриншоте выше при проверке доступности ip шлюза Москвы из С.-петербурга и обратно в выводе маршрута<br>
отображены не только транзитные адреса шлюзов но и номера автономных систем протокола eBGP.<br>

