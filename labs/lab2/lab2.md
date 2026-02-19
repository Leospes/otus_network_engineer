---
description: Построение Underlay сети (OSPF)
---

# LAB2

#### Цели

* Настроить OSPF для Underlay сети
* Убедиться в наличии IP связанности между устройствами в OSFP домене

#### Используемая схема сети на основе коммутаров Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_1.PNG" alt=""><figcaption></figcaption></figure>

#### Конфигурация устройств

LEAF1

```
feature ospf

router ospf 10
  router-id 1.1.1.1
  passive-interface default

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 10.1.1.2/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 10.1.2.2/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/3
  description to_PC1
  no switchport
  ip address 192.168.1.1/24
  ip router ospf 10 area 0.0.0.10
  no shutdown
```

LEAF2

```
feature ospf

router ospf 10
  router-id 2.2.2.2
  passive-interface default

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 20.1.1.2/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 20.1.2.2/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/3
  description to_PC2
  no switchport
  ip address 192.168.2.1/24
  ip router ospf 10 area 0.0.0.10
  no shutdown
```

LEAF3

```
feature ospf

router ospf 10
  router-id 3.3.3.3
  passive-interface default

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 30.1.1.2/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 30.1.2.2/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/3
  description to_PC3
  no switchport
  ip address 192.168.3.1/24
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/4
  description to_PC4
  no switchport
  ip address 192.168.4.1/24
  ip router ospf 10 area 0.0.0.10
  no shutdown
```

SPINE1

```
feature ospf

router ospf 10
  router-id 10.10.10.10
  passive-interface default

interface Ethernet1/1
  description to_LEAF1
  no switchport
  ip address 10.1.1.1/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  ip address 20.1.1.1/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  ip address 30.1.1.1/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown
```

SPINE2

```
feature ospf

router ospf 10
  router-id 20.20.20.20
  passive-interface default

interface Ethernet1/1
  description to_LEAF1
  no switchport
  ip address 10.1.2.1/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  ip address 20.1.2.1/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  ip address 30.1.2.1/30
  ip ospf authentication
  ip ospf authentication-key 3 4258f34a25410d21
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 10 area 0.0.0.10
  no shutdown
```

#### Проверка

OSPF соседства

```
LEAF1# show ip ospf neighbors
 OSPF Process ID 10 VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.10.10.10       1 FULL/ -          00:02:57 10.1.1.1        Eth1/1
 20.20.20.20       1 FULL/ -          00:02:51 10.1.2.1        Eth1/2
 
LEAF2# show ip ospf neighbors
 OSPF Process ID 10 VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.10.10.10       1 FULL/ -          00:01:07 20.1.1.1        Eth1/1
 20.20.20.20       1 FULL/ -          00:01:01 20.1.2.1        Eth1/2

LEAF3# show ip ospf neighbors
 OSPF Process ID 10 VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.10.10.10       1 FULL/ -          00:01:35 30.1.1.1        Eth1/1
 20.20.20.20       1 FULL/ -          00:01:28 30.1.2.1        Eth1/2

SPINE1# show ip ospf neighbors
 OSPF Process ID 10 VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 1.1.1.1           1 FULL/ -          00:01:55 10.1.1.2        Eth1/1
 2.2.2.2           1 FULL/ -          00:01:56 20.1.1.2        Eth1/2
 3.3.3.3           1 FULL/ -          00:01:55 30.1.1.2        Eth1/3

SPINE2# show ip ospf neighbors
 OSPF Process ID 10 VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 1.1.1.1           1 FULL/ -          00:02:03 10.1.2.2        Eth1/1
 2.2.2.2           1 FULL/ -          00:02:03 20.1.2.2        Eth1/2
 3.3.3.3           1 FULL/ -          00:02:02 30.1.2.2        Eth1/3
```

Маршруты на LEAF1

Видно, что до подсетей удалённых хостов имеются равноценные маршруты через интерфейсы в сторону спайнов, то есть имеется ECMP (Equal-cost multi-path routing). Метрика маршрутов при этом одинакова и имеет значение 120, так как дефолтная стоимость линка на nexus равна 40 (40 Gbit reference / 1 Gbit of interface).

<pre><code>LEAF1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%&#x3C;string>' in via output denotes VRF &#x3C;string>

1.1.1.1/32, ubest/mbest: 2/0, attached
    *via 1.1.1.1, Lo1, [0/0], 1d00h, local
    *via 1.1.1.1, Lo1, [0/0], 1d00h, direct
10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 23:07:47, direct
10.1.1.2/32, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 23:07:47, local
10.1.2.0/30, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 23:07:46, direct
10.1.2.2/32, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 23:07:46, local
20.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, Eth1/1, [110/80], 23:07:08, ospf-10, intra
20.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, Eth1/2, [110/80], 23:07:03, ospf-10, intra
30.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, Eth1/1, [110/80], 23:07:08, ospf-10, intra
30.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, Eth1/2, [110/80], 23:07:03, ospf-10, intra
192.168.1.0/24, ubest/mbest: 1/0, attached
    *via 192.168.1.1, Eth1/3, [0/0], 1d00h, direct
192.168.1.1/32, ubest/mbest: 1/0, attached
    *via 192.168.1.1, Eth1/3, [0/0], 1d00h, local
<strong>192.168.2.0/24, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [110/120], 22:55:31, ospf-10, intra
</strong><strong>    *via 10.1.2.1, Eth1/2, [110/120], 22:55:31, ospf-10, intra
</strong><strong>192.168.3.0/24, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [110/120], 22:55:25, ospf-10, intra
</strong><strong>    *via 10.1.2.1, Eth1/2, [110/120], 22:55:25, ospf-10, intra
</strong><strong>192.168.4.0/24, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [110/120], 22:54:25, ospf-10, intra
</strong><strong>    *via 10.1.2.1, Eth1/2, [110/120], 22:54:25, ospf-10, intra
</strong>
LEAF1# show ip route 192.168.4.0 detail
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%&#x3C;string>' in via output denotes VRF &#x3C;string>

<strong>192.168.4.0/24, ubest/mbest: 2/0
</strong><strong>    Extended Community: 0x08 18 00 00 00 00 00 00 00 00 80 00 00 00 00 0a 00 00 00 00 00 00 00 00
</strong><strong>    *via 10.1.1.1, Eth1/1, [110/120], 22:55:12, ospf-10, intra
</strong><strong>    *via 10.1.2.1, Eth1/2, [110/120], 22:55:12, ospf-10, intra
</strong>
LEAF1# show ip ospf route
 OSPF Process ID 10 VRF default, Routing Table
  (D) denotes route is directly attached      (R) denotes route is in RIB
  (L) denotes route label is in ULIB          (NHR) denotes next-hop is in RIB
10.1.1.0/30 (intra)(D) area 0.0.0.10
     via 10.1.1.0/Eth1/1*  , cost 40 distance 110 (NHR)
10.1.2.0/30 (intra)(D) area 0.0.0.10
     via 10.1.2.0/Eth1/2*  , cost 40 distance 110 (NHR)
20.1.1.0/30 (intra)(R) area 0.0.0.10
     via 10.1.1.1/Eth1/1  , cost 80 distance 110 (NHR)
20.1.2.0/30 (intra)(R) area 0.0.0.10
     via 10.1.2.1/Eth1/2  , cost 80 distance 110 (NHR)
30.1.1.0/30 (intra)(R) area 0.0.0.10
     via 10.1.1.1/Eth1/1  , cost 80 distance 110 (NHR)
30.1.2.0/30 (intra)(R) area 0.0.0.10
     via 10.1.2.1/Eth1/2  , cost 80 distance 110 (NHR)
192.168.1.0/24 (intra)(D) area 0.0.0.10
     via 192.168.1.0/Eth1/3*  , cost 40 distance 110 (NHR)
<strong>192.168.2.0/24 (intra)(R) area 0.0.0.10
</strong><strong>     via 10.1.1.1/Eth1/1  , cost 120 distance 110 (NHR)
</strong><strong>     via 10.1.2.1/Eth1/2  , cost 120 distance 110 (NHR)
</strong><strong>192.168.3.0/24 (intra)(R) area 0.0.0.10
</strong><strong>     via 10.1.1.1/Eth1/1  , cost 120 distance 110 (NHR)
</strong><strong>     via 10.1.2.1/Eth1/2  , cost 120 distance 110 (NHR)
</strong><strong>192.168.4.0/24 (intra)(R) area 0.0.0.10
</strong><strong>     via 10.1.1.1/Eth1/1  , cost 120 distance 110 (NHR)
</strong><strong>     via 10.1.2.1/Eth1/2  , cost 120 distance 110 (NHR)
</strong>	 
LEAF1# show ip ospf route 192.168.4.0/24
 OSPF Process ID 10 VRF default, Routing Table for prefixes 192.168.4.0/24
  (D) denotes route is directly attached      (R) denotes route is in RIB
  (L) denotes route label is in ULIB          (NHR) denotes next-hop is in RIB
<strong>192.168.4.0/24 (intra)(R) area 0.0.0.10
</strong><strong>     via 10.1.1.1/Eth1/1  , cost 120 distance 110 (NHR)
</strong><strong>     via 10.1.2.1/Eth1/2  , cost 120 distance 110 (NHR)
</strong></code></pre>

Маршруты на SPINE1

```
SPINE1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 23:10:23, direct
10.1.1.1/32, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 23:10:23, local
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [110/80], 23:10:10, ospf-10, intra
10.10.10.10/32, ubest/mbest: 2/0, attached
    *via 10.10.10.10, Lo10, [0/0], 23:25:28, local
    *via 10.10.10.10, Lo10, [0/0], 23:25:28, direct
20.1.1.0/30, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 23:10:23, direct
20.1.1.1/32, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 23:10:23, local
20.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.2, Eth1/2, [110/80], 23:10:11, ospf-10, intra
30.1.1.0/30, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 23:10:22, direct
30.1.1.1/32, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 23:10:22, local
30.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.2, Eth1/3, [110/80], 23:10:10, ospf-10, intra
192.168.1.0/24, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [110/80], 23:01:07, ospf-10, intra
192.168.2.0/24, ubest/mbest: 1/0
    *via 20.1.1.2, Eth1/2, [110/80], 22:58:32, ospf-10, intra
192.168.3.0/24, ubest/mbest: 1/0
    *via 30.1.1.2, Eth1/3, [110/80], 22:58:26, ospf-10, intra
192.168.4.0/24, ubest/mbest: 1/0
    *via 30.1.1.2, Eth1/3, [110/80], 22:57:26, ospf-10, intra
```

Проверка связности между PC1 и PC4

```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 192.168.1.2/24
GATEWAY     : 192.168.1.1
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10100
RHOST:PORT  : 127.0.0.1:10101
MTU         : 1500

PC4> show ip

NAME        : PC4[1]
IP/MASK     : 192.168.4.2/24
GATEWAY     : 192.168.4.1
DNS         :
MAC         : 00:50:79:66:68:03
LPORT       : 10126
RHOST:PORT  : 127.0.0.1:10127
MTU         : 1500

PC1> ping 192.168.4.2

84 bytes from 192.168.4.2 icmp_seq=1 ttl=61 time=23.687 ms
84 bytes from 192.168.4.2 icmp_seq=2 ttl=61 time=14.829 ms
84 bytes from 192.168.4.2 icmp_seq=3 ttl=61 time=15.178 ms
84 bytes from 192.168.4.2 icmp_seq=4 ttl=61 time=18.660 ms
84 bytes from 192.168.4.2 icmp_seq=5 ttl=61 time=16.199 ms
```

#### Выводы

Таким образом, нам удалось настроить единый домен OSPF с номером area 10.

Соседства успешно поднялись и маршрутная информация получена всеми устройствами.

Связность между узлами сети имеется.
