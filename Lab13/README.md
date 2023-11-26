### 1. Настроите GRE между офисами Москва и С.-Петербург.<br>

 Создаём tun интерфейсы на R15 и R18. На R18 инициируем eigrp с редистрибьюцией маршрутов из уже существующего eigrp,<br> 
на R15 инициируем eigrp процесс с редистрибьюцией маршрутов из ospf. Запускаем eigrp на сети для туннельного интерфейса.<br>

R15 gre
```
!
interface Tunnel1
 ip address 192.168.100.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 5
 tunnel source 15.15.15.1
 tunnel destination 18.18.18.1
!
router eigrp 1
 network 192.168.100.0 0.0.0.3
redistribute ospf 1 metric 10 20 100 200 1400
```
<br>

R18 gre
```
router eigrp 1
 network 192.168.100.0 0.0.0.3
 redistribute eigrp 2042 metric 20 30 100 200 1400
!
interface Tunnel1
 ip address 192.168.100.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 5
 tunnel source 18.18.18.1
 tunnel destination 15.15.15.1
```
<br>

### 2.Настроите DMVMN между Москва и Чокурдах, Лабытнанги.<br>
