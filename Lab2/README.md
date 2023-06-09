## Лабораторная работа. Развертывание коммутируемой сети с резервными каналами<br>

>Топология<br>
![](EVE-ShemeSTP.png)

### Таблица адресации<br>

|Устройство|Интерфейс|IP-адрес|Маска подсети|
|:-|:-|:-|:-|
|S1|VLAN 1|192.168.1.1|255.255.255.0|
|S2|VLAN 1|192.168.1.2|255.255.255.0|
|S3|VLAN 1|192.168.1.3|255.255.255.0|


### ЧАСТЬ 1: Создание сети и настройка основных параметров устройства<br>

#### Настройте базовые параметры каждого коммутатора.<br>

hostname S1(S2,S3)<br>
no ip domain-lookup<br>
enable secret class<br>
line console 0<br>
password cisco<br>
login<br>
line vty 0 4<br>
password cisco<br>
login<br>
service password-encryption<br>
banner motd $<br>
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*<br>
\*      Attention! Unauthorized access to this device is prohibited.      \*<br>
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*$<br>

clock timezone Moscow +3 0<br>
interface vlan 1<br>
address 192.168.1.1 255.255.255.0<br>
no sh<br>
end<br>
wr<br>

#### Проверьте связь.<br>

>Проверка связи.<br>
![](ping_S1_to_S2_S3.png)<br>
![](ping_S2_to_S3.png)<br>

### ЧАСТЬ 2: Определение корневого моста<br>

#### Отключите все порты на коммутаторах. Настройте подключенные порты в качестве транковых.<br>

(config)#interface range gigabitEthernet 0/0-3<br>
(config-if-range)#shutdown<br>
(config-if-range)#switchport trunk encapsulation dot1q<br>
(config-if-range)#switchport mode trunk<br>
(config-if-range)#switchport trunk allowed vlan all<br>

#### Включите порты Gi0/1 и Gi0/3 на всех коммутаторах.<br>

(config)int gi 0/1<br>
(config-if)#no shu<br>
(config-if)#int gi 0/3<br>
(config-if)#no shu<br>

#### Отобразите данные протокола spanning-tree.<br>

>S1#show spanning-tree<br>
![](S1_sh_spanning-tree.png)<br>

>S2#show spanning-tree<br>
![](S2_sh_spanning-tree.png)<br>

>S3#show spanning-tree<br>
![](S3_sh_spanning-tree.png)<br>

>SHEME
![](SHEME_STP_port.png)<br>

##### Какой коммутатор является корневым мостом?<br>
Коммутатор S1<br>

##### Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?<br>
Все коммутаторы находятся в одном vlan и имеют одинаковый BID 32769.<br>
Выбор корневого осуществлён по наименьшему MAC приндалежащему коммутатор S1 c MAC: 5000.0001.0000<br>

##### Какие порты на коммутаторе являются корневыми портами?<br>
Транковые порты свитчей смотрящие в сторону корневого коммутатора являются корневыми: <br>
S2 - Gi0/1 Root<br>
S3 - Gi0/3 Root<br>

##### Какие порты на коммутаторе являются назначенными портами?<br>
Транковые порты на свитчах направленные от корневого коммутатора в сторону остальных коммутаторов участников spannig-tree процесса:<br>
S1 - Gi0/1 Desg<br>
S1 - Gi0/3 Desg<br>
S2 - Gi0/3 Desg<br>
 
##### Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?<br>
В собранной схеме только один порт свитча S3 является альтернативным и заблокированным: S3 - Gi0/1 Altn BLK  

##### Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?<br>
Коммутатор S2 и S3 имеют одинаковую стоимость до корневого коммутатора S1. Коммутатор S2 имеет меньшее значение поля mac в BID поэтому его порты станут designated а порт на коммутаторе S3 ALT.


### ЧАСТЬ 3: Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов<br>

#### ШАГ 1: Определите коммутатор с заблокированным портом.<br>

>S3#show spanning-tree<br>
![](S3_sh_spanning-tree.png)<br>

>S2#show spanning-tree<br>
![](S2_sh_spanning-tree.png)<br>

#### ШАГ 2: Измените стоимость порта.<br>

S3(config-if)#int gi0/3<br>
S3(config-if)#spanning-tree cost 3

#### ШАГ 3: Просмотрите изменения протокола spanning-tree.<br>

>S3#show spanning-tree<br>
![](S3_sh_spanning-tree_cost.png)<br>

>S2#show spanning-tree<br>
![](S2_sh_spanning-tree_cost.png)<br>

#### ШАГ 4: Удалите изменения стоимости порта.<br>

S3(config-if)#int gi0/3<br>
S3(config-if)#no spanning-tree cost<br>

>S2#show spanning-tree<br>
![](S2_sh_spanning-tree.png)<br>

>S3#show spanning-tree<br>
![](S3_sh_spanning-tree.png)<br>

 
### ЧАСТЬ 4: Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов<br>

##### Включите порты Gi0/0 и Gi0/2 на всех коммутаторах.<br>

(config)int gi 0/0<br>
(config-if)#no shu<br>
(config-if)#int gi 0/2<br>
(config-if)#no shu<br> 

>SHEME
![](SHEME_STP_port_all.png)<br>

>S2#show spanning-tree<br>
![](S2_sh_spanning-tree_all.png)<br>

>S3#show spanning-tree<br>
![](S3_sh_spanning-tree_all.png)<br>

##### Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?<br> Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?<br>
Выбран порт на котором PortID соседа имеет наименьшее значение:<br> 
S3 Gi0/2 root cost=4 pri=128.3<br>

#### Вопросы для повторения.<br>

##### Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?<br>
Значение "root path cost" в поле BPDU, выбирается порт от которого маршрут до корневого моста имеет наименьшую стоимость.<br>

##### Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?<br>
Будет выбран порт в сторону коммутатора с наименьшим BID(Bridge ID) состосящий из: Bridge priority и MAC Address коммутатора.<br>

##### Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?<br>
Будет выбрано самое низкое значение порта отправителя port ID(port priority).<br>

