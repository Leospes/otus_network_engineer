---
description: >-
  Построение и объединение двух ЦОД на основе технологии VxLAN EVPN на
  оборудовании Cisco Nexus
---

# Проектная работа

#### **Цели**

* Реализация сетевой фабрики с использованием Overlay технологии VxLAN EVPN
* Объединение двух ЦОД-ов (POD) на основе Multi-Pod топологии
* Протестировать связанность между клиентами внутри VNI, между VNI внутри одного VRF, а также между VNI в разных VRF.

#### **Задачи**

* Распределение адресного пространства
* Настройка Underlay и Overlay сети на основе eBGP
* Настройка EVPN/VXLAN с использованием L2VNI и L3VNI
* Настройка подключения одного или нескольких клиентов с использованием VPC для повышения отказоустойчивости
* Настройка связности и eBGP пиринга между площадками с использованием Multi-Pod топологии
* Создание нескольких VRF и размещение в них клиентов
* Реализация ликинга маршрутов между VRF через внешний роутер
* Проверка связности между клиентами одной площадки, а также между площадками в рамках одного vlan/vni, между разными vni, а также между разными VRF.

#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_9.PNG" alt=""><figcaption></figcaption></figure>

#### IP адресация

<table><thead><tr><th width="133.5712890625">Network</th><th width="351.5714111328125">Octet</th><th>Description</th></tr></thead><tbody><tr><td>x0.y.z.0/30</td><td><p>x - номер LEAF, </p><p>y - номер ЦОД/POD, </p><p>z - номер SPINE</p></td><td>p2p сеть LEAFx&#x3C;-->SPINEz</td></tr><tr><td>1.x.y.z/32</td><td><p>x - номер ЦОД, </p><p>y - номер POD внутри ЦОД, </p><p>z - номер LEAF</p></td><td>loopback интерфейс LEAFz</td></tr><tr><td>10.x0.y0.z/32</td><td><p>x - номер ЦОД, </p><p>y - номер POD внутри ЦОД, </p><p>z - номер SPINE</p></td><td>loopback интерфейс SPINEz</td></tr><tr><td>172.16.0.0/12</td><td></td><td>клиентские подсети</td></tr><tr><td>12.12.12.0/24</td><td><p>12.12.12.0/30 - p2p SPINE1-1&#x3C;-->SPINE2-2</p><p>12.12.12.4/30 - p2p SPINE1-2&#x3C;-->SPINE2-1</p></td><td>пул для Data Center Interconnect (DCI) интерфейсов</td></tr><tr><td>192.168.0.0/16</td><td><p>192.168.1.0/30 - VPC keepalive</p><p>192.168.2.0/30 - VPC peer-link eBGP</p></td><td>служебные подсети</td></tr><tr><td>10.1.1.100</td><td></td><td>VPC shared, Anycast VTEP IP для VXLAN пиринга</td></tr></tbody></table>

COD1

| Device    | Interface       | IP Address  | Subnet Mask     |
| --------- | --------------- | ----------- | --------------- |
| SPINE1-1  | lo10            | 10.10.10.1  | 255.255.255.255 |
| SPINE1-1  | e1/1            | 10.1.1.1    | 255.255.255.252 |
| SPINE1-1  | e1/2            | 20.1.1.1    | 255.255.255.252 |
| SPINE1-1  | e1/3            | 30.1.1.1    | 255.255.255.252 |
| SPINE1-1  | e1/4            | 40.1.1.1    | 255.255.255.252 |
| SPINE1-1  | e1/5            | 12.12.12.5  | 255.255.255.252 |
| SPINE1-2  | lo10            | 10.10.10.2  | 255.255.255.255 |
| SPINE1-2  | e1/1            | 10.1.2.1    | 255.255.255.252 |
| SPINE1-2  | e1/2            | 20.1.2.1    | 255.255.255.252 |
| SPINE1-2  | e1/3            | 30.1.2.1    | 255.255.255.252 |
| SPINE1-2  | e1/4            | 40.1.2.1    | 255.255.255.252 |
| SPINE1-2  | e1/5            | 12.12.12.1  | 255.255.255.252 |
| SPINE2-1  | lo10            | 10.20.10.1  | 255.255.255.255 |
| SPINE2-1  | e1/1            | 10.2.1.1    | 255.255.255.252 |
| SPINE2-1  | e1/2            | 20.2.1.1    | 255.255.255.252 |
| SPINE2-1  | e1/3            | 30.2.1.1    | 255.255.255.252 |
| SPINE2-1  | e1/4            | 12.12.12.2  | 255.255.255.252 |
| SPINE2-2  | lo10            | 10.20.10.2  | 255.255.255.255 |
| SPINE2-2  | e1/1            | 10.2.2.1    | 255.255.255.252 |
| SPINE2-2  | e1/2            | 20.2.2.1    | 255.255.255.252 |
| SPINE2-2  | e1/3            | 30.2.2.1    | 255.255.255.252 |
| SPINE2-2  | e1/4            | 12.12.12.6  | 255.255.255.252 |
| LEAF1-1   | lo1             | 1.1.1.1     | 255.255.255.255 |
| LEAF1-1   | e1/1            | 10.1.1.2    | 255.255.255.252 |
| LEAF1-1   | e1/2            | 10.1.2.2    | 255.255.255.252 |
| LEAF1-1   | vlan67          | 172.17.67.1 | 255.255.255.0   |
| LEAF1-2   | lo1             | 1.1.1.2     | 255.255.255.255 |
| LEAF1-2   | e1/1            | 20.1.1.2    | 255.255.255.252 |
| LEAF1-2   | e1/2            | 20.1.2.2    | 255.255.255.252 |
| LEAF1-2   | vlan67          | 172.17.67.1 | 255.255.255.0   |
| LEAF1-3   | lo1             | 1.1.1.3     | 255.255.255.255 |
| LEAF1-3   | e1/1            | 30.1.1.2    | 255.255.255.252 |
| LEAF1-3   | e1/2            | 30.1.2.2    | 255.255.255.252 |
| LEAF1-3   | vlan65          | 172.16.65.1 | 255.255.255.0   |
| LEAF1-4   | lo1             | 1.1.1.4     | 255.255.255.255 |
| LEAF1-4   | e1/1            | 40.1.1.2    | 255.255.255.252 |
| LEAF1-4   | e1/2            | 40.1.2.2    | 255.255.255.252 |
| LEAF1-4   | vlan66          | 172.16.66.1 | 255.255.255.0   |
| LEAF2-1   | lo1 (main)      | 1.2.1.1     | 255.255.255.255 |
| LEAF2-1   | lo1 (secondary) | 10.1.1.100  | 255.255.255.255 |
| LEAF2-1   | e1/1            | 10.2.1.2    | 255.255.255.252 |
| LEAF2-1   | e1/2            | 10.2.2.2    | 255.255.255.252 |
| LEAF2-1   | vlan65          | 172.16.65.1 | 255.255.255.0   |
| LEAF2-1   | vlan2           | 192.168.2.1 | 255.255.255.252 |
| LEAF2-1   | mgmt0           | 192.168.1.1 | 255.255.255.0   |
| LEAF2-2   | lo1 (main)      | 1.2.1.2     | 255.255.255.255 |
| LEAF2-2   | lo1 (secondary) | 10.1.1.100  | 255.255.255.255 |
| LEAF2-2   | e1/1            | 20.2.1.2    | 255.255.255.252 |
| LEAF2-2   | e1/2            | 20.2.2.2    | 255.255.255.252 |
| LEAF2-2   | vlan65          | 172.16.65.1 | 255.255.255.0   |
| LEAF2-2   | vlan2           | 192.168.2.2 | 255.255.255.252 |
| LEAF2-2   | mgmt0           | 192.168.1.2 | 255.255.255.0   |
| endhost-1 | eth0            | 172.17.67.2 | 255.255.255.0   |
| endhost-2 | eth0            | 172.17.67.3 | 255.255.255.0   |
| endhost-3 | eth0            | 172.17.67.4 | 255.255.255.0   |
| endhost-4 | eth0            | 172.16.65.2 | 255.255.255.0   |
| endhost-5 | eth0            | 172.16.66.2 | 255.255.255.0   |
| endhost-6 | eth0            | 172.16.65.3 | 255.255.255.0   |
| endhost-7 | eth0            | 172.17.67.5 | 255.255.255.0   |
| endhost-8 | eth0            | 172.17.67.6 | 255.255.255.0   |

#### Конфигурация устройств

<details>

<summary>LEAF1-1</summary>

```
nv overlay evpn
feature bgp
feature pim
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature nv overlay

fabric forwarding anycast-gateway-mac 0011.0011.0011
vlan 1,25,67,501
vlan 25
  name VLAN_25
  vn-segment 100025
vlan 67
  name VLAN_67
  vn-segment 100067
vlan 501
  name L3-VNI-TENANT_1
  vn-segment 100501

route-map DIRECT-ROUTES-MAP permit 10
vrf context TENANT_1
  vni 100501
  rd 65401:2
  address-family ipv4 unicast
    route-target import 22:22 evpn
    route-target import 2:2
    route-target export 22:22 evpn
    route-target export 2:2
	
interface Vlan67
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip address 172.17.67.1/24
  fabric forwarding mode anycast-gateway

interface Vlan501
  description L3-VNI-TENANT_1
  no shutdown
  mtu 9216
  vrf member TENANT_1
  no ip redirects
  ip forward
  no ipv6 redirects

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100067
    ingress-replication protocol bgp
  member vni 100501 associate-vrf

interface Ethernet1/1
  description to_SPINE1
  no switchport
  mtu 9216
  ip address 10.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 10.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-1
  switchport access vlan 67
  mtu 9216

interface Ethernet1/4
  description to_endhost-2
  switchport access vlan 67
  mtu 9216
  
interface loopback1
  ip address 1.1.1.1/32
  
router bgp 65401
  router-id 1.1.1.1
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
  template peer OVERLAY
    remote-as 65400
    update-source loopback1
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65400
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 10.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 10.10.10.1
    inherit peer OVERLAY
  neighbor 10.10.10.2
    inherit peer OVERLAY
evpn
  vni 100067 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>LEAF1-2</summary>

```
nv overlay evpn
feature bgp
feature pim
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature nv overlay

fabric forwarding anycast-gateway-mac 0011.0011.0011
vlan 1,67,501
vlan 67
  name VLAN_67
  vn-segment 100067
vlan 501
  name L3-VNI-TENANT_1
  vn-segment 100501

route-map DIRECT-ROUTES-MAP permit 10
vrf context TENANT_1
  vni 100501
  rd 65402:2
  address-family ipv4 unicast
    route-target import 22:22 evpn
    route-target import 2:2
    route-target export 22:22 evpn
    route-target export 2:2
	
interface Vlan67
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip address 172.17.67.1/24
  fabric forwarding mode anycast-gateway

interface Vlan501
  description L3-VNI-TENANT_1
  no shutdown
  mtu 9216
  vrf member TENANT_1
  no ip redirects
  ip forward
  no ipv6 redirects

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100067
    ingress-replication protocol bgp
  member vni 100501 associate-vrf

interface Ethernet1/1
  description to_SPINE1
  no switchport
  mtu 9216
  ip address 20.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 20.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-3
  switchport access vlan 67
  mtu 9216
  
interface loopback1
  ip address 1.1.1.2/32
  
router bgp 65402
  router-id 1.1.1.2
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
  template peer OVERLAY
    remote-as 65400
    update-source loopback1
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65400
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.10.10.1
    inherit peer OVERLAY
  neighbor 10.10.10.2
    inherit peer OVERLAY
  neighbor 20.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 20.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
evpn
  vni 100067 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>LEAF1-3</summary>

```
nv overlay evpn
feature bgp
feature pim
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature nv overlay

fabric forwarding anycast-gateway-mac 0011.0011.0011
vlan 1,65,67,500-501
vlan 65
  name VLAN_65
  vn-segment 100065
vlan 67
  name VLAN_67
  vn-segment 100067
vlan 500
  name L3-VNI-COD
  vn-segment 100500
vlan 501
  name L3-VNI-TENANT_1
  vn-segment 100501

route-map DIRECT-ROUTES-MAP permit 10
vrf context COD
  vni 100500
  rd 65403:1
  address-family ipv4 unicast
    route-target import 11:11 evpn
    route-target import 1:1
    route-target export 11:11 evpn
    route-target export 1:1
vrf context TENANT_1
  vni 100501
  rd 65403:2
  address-family ipv4 unicast
    route-target import 22:22 evpn
    route-target import 2:2
    route-target export 22:22 evpn
    route-target export 2:2
	
interface Vlan65
  no shutdown
  vrf member COD
  no ip redirects
  ip address 172.16.65.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan67
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip address 172.17.67.1/24
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  vrf member COD
  no ip redirects
  ip forward
  no ipv6 redirects

interface Vlan501
  description L3-VNI-TENANT_1
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100065
    ingress-replication protocol bgp
  member vni 100067
    ingress-replication protocol bgp
  member vni 100500 associate-vrf
  member vni 100501 associate-vrf

interface Ethernet1/1
  description to_SPINE1
  no switchport
  mtu 9216
  ip address 30.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 30.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-4
  switchport access vlan 65
  mtu 9216
  
interface loopback1
  ip address 1.1.1.3/32
  
router bgp 65403
  router-id 1.1.1.3
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
  template peer OVERLAY
    remote-as 65400
    update-source loopback1
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65400
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.10.10.1
    inherit peer OVERLAY
  neighbor 10.10.10.2
    inherit peer OVERLAY
  neighbor 30.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 30.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
evpn
  vni 100065 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 100067 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>LEAF1-4</summary>

```
nv overlay evpn
feature bgp
feature pim
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature nv overlay

fabric forwarding anycast-gateway-mac 0011.0011.0011
vlan 1,66-67,500-501
vlan 66
  name VLAN_66
  vn-segment 100066
vlan 67
  name VLAN_67
  vn-segment 100067
vlan 500
  name L3-VNI-COD
  vn-segment 100500
vlan 501
  name L3-VNI-TENANT_1
  vn-segment 100501

route-map DIRECT-ROUTES-MAP permit 10
vrf context COD
  vni 100500
  rd 65403:1
  address-family ipv4 unicast
    route-target import 11:11 evpn
    route-target import 1:1
    route-target export 11:11 evpn
    route-target export 1:1
vrf context TENANT_1
  vni 100501
  rd 65404:2
  address-family ipv4 unicast
    route-target import 22:22 evpn
    route-target import 2:2
    route-target export 22:22 evpn
    route-target export 2:2
	
interface Vlan66
  no shutdown
  vrf member COD
  no ip redirects
  ip address 172.16.66.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan67
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip address 172.17.67.1/24
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  vrf member COD
  no ip redirects
  ip forward
  no ipv6 redirects

interface Vlan501
  description L3-VNI-TENANT_1
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100066
    ingress-replication protocol bgp
  member vni 100067
    ingress-replication protocol bgp
  member vni 100500 associate-vrf
  member vni 100501 associate-vrf

interface Ethernet1/1
  description to_SPINE1
  no switchport
  mtu 9216
  ip address 40.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 40.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-5
  switchport access vlan 66
  mtu 9216
  
interface loopback1
  ip address 1.1.1.4/32
  
router bgp 65404
  router-id 1.1.1.4
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
    nexthop route-map UNCHANGED
    advertise-pip
  template peer OVERLAY
    remote-as 65400
    update-source loopback1
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65400
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.10.10.1
    inherit peer OVERLAY
  neighbor 10.10.10.2
    inherit peer OVERLAY
  neighbor 40.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 40.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
evpn
  vni 100066 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 100067 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>SPINE1-1</summary>

```
nv overlay evpn
feature bgp
feature vn-segment-vlan-based
feature nv overlay

route-map DIRECT-ROUTES-MAP permit 10
route-map UNCHANGED permit 10
  set ip next-hop unchanged
  
interface Ethernet1/1
  description to_LEAF1-1
  no switchport
  mtu 9216
  ip address 10.1.1.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF1-2
  no switchport
  mtu 9216
  ip address 20.1.1.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF1-3
  no switchport
  mtu 9216
  ip address 30.1.1.1/30
  no shutdown

interface Ethernet1/4
  description to_LEAF1-4
  no switchport
  mtu 9216
  ip address 40.1.1.1/30
  no shutdown

interface Ethernet1/5
  description to_COD2
  no switchport
  no ip redirects
  ip address 12.12.12.5/30
  no shutdown
  
interface loopback10
  ip address 10.10.10.1/32
  
router bgp 65400
  router-id 10.10.10.1
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
  address-family l2vpn evpn
    retain route-target all
  template peer OVERLAY
    update-source loopback10
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map UNCHANGED out
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    timers 3 9
    address-family ipv4 unicast
  neighbor 1.1.1.1
    inherit peer OVERLAY
    remote-as 65401
  neighbor 1.1.1.2
    inherit peer OVERLAY
    remote-as 65402
  neighbor 1.1.1.3
    inherit peer OVERLAY
    remote-as 65403
  neighbor 1.1.1.4
    inherit peer OVERLAY
    remote-as 65404
  neighbor 10.1.1.2
    inherit peer UNDERLAY
    remote-as 65401
    address-family ipv4 unicast
  neighbor 10.20.10.2
    inherit peer OVERLAY
    remote-as 65500
  neighbor 12.12.12.6
    inherit peer UNDERLAY
    remote-as 65500
  neighbor 20.1.1.2
    inherit peer UNDERLAY
    remote-as 65402
    address-family ipv4 unicast
  neighbor 30.1.1.2
    inherit peer UNDERLAY
    remote-as 65403
    address-family ipv4 unicast
  neighbor 40.1.1.2
    inherit peer UNDERLAY
    remote-as 65404
    address-family ipv4 unicast
```

</details>

<details>

<summary>SPINE1-2</summary>

```
nv overlay evpn
feature bgp
feature vn-segment-vlan-based
feature nv overlay

route-map DIRECT-ROUTES-MAP permit 10
route-map UNCHANGED permit 10
  set ip next-hop unchanged
  
interface Ethernet1/1
  description to_LEAF1-1
  no switchport
  mtu 9216
  ip address 10.1.2.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF1-2
  no switchport
  mtu 9216
  ip address 20.1.2.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF1-3
  no switchport
  mtu 9216
  ip address 30.1.2.1/30
  no shutdown

interface Ethernet1/4
  description to_LEAF1-4
  no switchport
  mtu 9216
  ip address 40.1.2.1/30
  no shutdown

interface Ethernet1/5
  description to_COD2
  no switchport
  no ip redirects
  ip address 12.12.12.1/30
  no shutdown
  
interface loopback10
  ip address 10.10.10.2/32
  
router bgp 65400
  router-id 10.10.10.2
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
  address-family l2vpn evpn
    retain route-target all
  template peer OVERLAY
    update-source loopback10
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map UNCHANGED out
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    timers 3 9
    address-family ipv4 unicast
  neighbor 1.1.1.1
    inherit peer OVERLAY
    remote-as 65401
  neighbor 1.1.1.2
    inherit peer OVERLAY
    remote-as 65402
  neighbor 1.1.1.3
    inherit peer OVERLAY
    remote-as 65403
  neighbor 1.1.1.4
    inherit peer OVERLAY
    remote-as 65404
  neighbor 10.1.2.2
    inherit peer UNDERLAY
    remote-as 65401
    address-family ipv4 unicast
  neighbor 10.20.10.1
    inherit peer OVERLAY
    remote-as 65500
  neighbor 12.12.12.2
    inherit peer UNDERLAY
    remote-as 65500
  neighbor 20.1.2.2
    inherit peer UNDERLAY
    remote-as 65402
    address-family ipv4 unicast
  neighbor 30.1.2.2
    inherit peer UNDERLAY
    remote-as 65403
    address-family ipv4 unicast
  neighbor 40.1.2.2
    inherit peer UNDERLAY
    remote-as 65404
    address-family ipv4 unicast
```

</details>

<details>

<summary>LEAF2-1</summary>

```
nv overlay evpn
feature bgp
feature pim
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001
vlan 1-2,65,500
vlan 2
  name for_VPC_redundancy
vlan 65
  name VLAN_65
  vn-segment 100065
vlan 500
  name L3-VNI-COD
  vn-segment 100500

route-map DIRECT-ROUTES-MAP permit 10
vrf context COD
  vni 100500
  rd 65501:1
  address-family ipv4 unicast
    route-target import 11:11 evpn
    route-target import 1:1
    route-target export 11:11 evpn
    route-target export 1:1
	
vpc domain 1
  peer-switch
  peer-keepalive destination 192.168.1.2
  peer-gateway
  ip arp synchronize
  
interface Vlan2
  description special_svi_over_peer-link_for_redundancy
  no shutdown
  no ip redirects
  ip address 192.168.2.1/30
  no ipv6 redirects
  ip pim sparse-mode

interface Vlan65
  no shutdown
  vrf member COD
  no ip redirects
  ip address 172.16.65.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  mtu 9216
  vrf member COD
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  spanning-tree port type network
  vpc peer-link

interface port-channel3
  description to_endhost-1
  switchport mode trunk
  switchport trunk allowed vlan 65
  vpc 3

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100065
    ingress-replication protocol bgp
  member vni 100500 associate-vrf

interface Ethernet1/1
  description to_SPINE1
  no switchport
  mtu 9216
  ip address 10.2.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 10.2.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-6
  switchport mode trunk
  switchport trunk allowed vlan 65
  channel-group 3 mode active

interface Ethernet1/4
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active

interface Ethernet1/5
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active
  
interface mgmt0
  vrf member management
  ip address 192.168.1.1/24

interface loopback1
  ip address 1.2.1.1/32
  ip address 10.1.1.100/32 secondary
  
router bgp 65501
  router-id 1.2.1.1
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
    nexthop route-map UNCHANGED
    advertise-pip
  template peer OVERLAY
    remote-as 65500
    update-source loopback1
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65500
    timers 3 9
    address-family ipv4 unicast
  template peer VPC
    remote-as 65502
    update-source Vlan2
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.2.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 10.2.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 10.20.10.1
    inherit peer OVERLAY
  neighbor 10.20.10.2
    inherit peer OVERLAY
  neighbor 192.168.2.2
    inherit peer VPC
evpn
  vni 100065 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>LEAF2-2</summary>

```
nv overlay evpn
feature bgp
feature pim
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001
vlan 1-2,65,500
vlan 2
  name for_VPC_redundancy
vlan 65
  name VLAN_65
  vn-segment 100065
vlan 500
  name L3-VNI-COD
  vn-segment 100500

route-map DIRECT-ROUTES-MAP permit 10
vrf context COD
  vni 100500
  rd 65502:1
  address-family ipv4 unicast
    route-target import 11:11 evpn
    route-target import 1:1
    route-target export 11:11 evpn
    route-target export 1:1
	
vpc domain 1
  peer-switch
  peer-keepalive destination 192.168.1.1
  peer-gateway
  ip arp synchronize
  
interface Vlan2
  description special_svi_over_peer-link_for_redundancy
  no shutdown
  no ip redirects
  ip address 192.168.2.2/30
  no ipv6 redirects
  ip pim sparse-mode

interface Vlan65
  no shutdown
  vrf member COD
  no ip redirects
  ip address 172.16.65.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  vrf member COD
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  spanning-tree port type network
  vpc peer-link

interface port-channel3
  description to_endhost-1
  switchport mode trunk
  switchport trunk allowed vlan 65
  vpc 3

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback2
  member vni 100065
    ingress-replication protocol bgp
  member vni 100500 associate-vrf

interface Ethernet1/1
  description to_SPINE1
  no switchport
  mtu 9216
  ip address 20.2.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 20.2.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-6
  switchport mode trunk
  switchport trunk allowed vlan 65
  channel-group 3 mode active

interface Ethernet1/4
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active

interface Ethernet1/5
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active
  
interface mgmt0
  vrf member management
  ip address 192.168.1.2/24

interface loopback2
  ip address 1.2.1.2/32
  ip address 10.1.1.100/32 secondary
  
router bgp 65502
  router-id 1.2.1.2
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
  template peer OVERLAY
    remote-as 65500
    update-source loopback2
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65500
    timers 3 9
    address-family ipv4 unicast
  template peer VPC
    remote-as 65501
    update-source Vlan2
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.20.10.1
    inherit peer OVERLAY
  neighbor 10.20.10.2
    inherit peer OVERLAY
  neighbor 20.2.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 20.2.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 192.168.2.1
    inherit peer VPC
evpn
  vni 100065 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>LEAF2-3</summary>

```
nv overlay evpn
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001
vlan 1,67,101-102,500-501
vlan 67
  name VLAN_67
  vn-segment 100067
vlan 500
  name L3-VNI-COD
  vn-segment 100500
vlan 501
  name L3-VNI-TENANT_1
  vn-segment 100501

route-map DIRECT-ROUTES-MAP permit 10
vrf context COD
  vni 100500
  rd 65503:1
  address-family ipv4 unicast
    route-target import 11:11 evpn
    route-target import 1:1
    route-target export 11:11 evpn
    route-target export 1:1
vrf context TENANT_1
  vni 100501
  rd 65503:2
  address-family ipv4 unicast
    route-target import 22:22 evpn
    route-target import 2:2
    route-target export 22:22 evpn
    route-target export 2:2
	
interface Vlan67
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip address 172.17.67.1/24
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  vrf member COD
  no ip redirects
  ip forward

interface Vlan501
  description L3-VNI-TENANT_1
  no shutdown
  vrf member TENANT_1
  no ip redirects
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback3
  member vni 100067
    ingress-replication protocol bgp
  member vni 100500 associate-vrf
  member vni 100501 associate-vrf

interface Ethernet1/1
  description to_SPINE1
  no switchport
  mtu 9216
  ip address 30.2.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 30.2.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-7
  switchport access vlan 67
  mtu 9216

interface Ethernet1/4
  description to_endhost-8
  switchport access vlan 67
  mtu 9216

interface Ethernet1/5
  description to_EDGE
  no switchport
  mac-address 0ce8.0000.1b09
  vrf member COD
  ip address 192.168.3.1/30
  no shutdown

interface Ethernet1/6
  description to_EDGE
  no switchport
  mac-address 0ce8.0000.1b10
  vrf member TENANT_1
  ip address 192.168.3.5/30
  no shutdown
  
interface loopback3
  ip address 1.2.1.3/32
  
router bgp 65503
  router-id 1.2.1.3
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
  template peer OVERLAY
    remote-as 65500
    update-source loopback3
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      as-override
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65500
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.20.10.1
    inherit peer OVERLAY
  neighbor 10.20.10.2
    inherit peer OVERLAY
  neighbor 30.2.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 30.2.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  vrf COD
    address-family ipv4 unicast
      advertise l2vpn evpn
      redistribute direct route-map DIRECT-ROUTES-MAP
      maximum-paths 2
    neighbor 192.168.3.2
      remote-as 65600
      address-family ipv4 unicast
  vrf TENANT_1
    address-family ipv4 unicast
      advertise l2vpn evpn
      redistribute direct route-map DIRECT-ROUTES-MAP
      maximum-paths 2
    neighbor 192.168.3.6
      remote-as 65600
      address-family ipv4 unicast
evpn
  vni 100067 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>SPINE2-1</summary>

```
nv overlay evpn
feature bgp
feature vn-segment-vlan-based
feature nv overlay

route-map DIRECT-ROUTES-MAP permit 10
route-map UNCHANGED permit 10
  set ip next-hop unchanged
  
interface Ethernet1/1
  description to_LEAF1
  no switchport
  mtu 9216
  ip address 10.2.1.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  mtu 9216
  ip address 20.2.1.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  mtu 9216
  ip address 30.2.1.1/30
  no shutdown

interface Ethernet1/4
  description to_COD1
  no switchport
  mac-address 0c75.0000.1b11
  no ip redirects
  ip address 12.12.12.2/30
  no shutdown
  
interface loopback10
  ip address 10.20.10.1/32
  
router bgp 65500
  router-id 10.20.10.1
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
  address-family l2vpn evpn
    retain route-target all
  template peer OVERLAY
    update-source loopback10
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map UNCHANGED out
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    timers 3 9
    address-family ipv4 unicast
  neighbor 1.2.1.1
    inherit peer OVERLAY
    remote-as 65501
  neighbor 1.2.1.2
    inherit peer OVERLAY
    remote-as 65502
  neighbor 1.2.1.3
    inherit peer OVERLAY
    remote-as 65503
  neighbor 10.2.1.2
    inherit peer UNDERLAY
    remote-as 65501
    address-family ipv4 unicast
  neighbor 10.10.10.2
    inherit peer OVERLAY
    remote-as 65400
  neighbor 12.12.12.1
    inherit peer UNDERLAY
    remote-as 65400
  neighbor 20.2.1.2
    inherit peer UNDERLAY
    remote-as 65502
    address-family ipv4 unicast
  neighbor 30.2.1.2
    inherit peer UNDERLAY
    remote-as 65503
    address-family ipv4 unicast
```

</details>

<details>

<summary>SPINE2-2</summary>

```
nv overlay evpn
feature bgp
feature vn-segment-vlan-based
feature nv overlay

route-map DIRECT-ROUTES-MAP permit 10
route-map UNCHANGED permit 10
  set ip next-hop unchanged
  
interface Ethernet1/1
  description to_LEAF1
  no switchport
  mtu 9216
  ip address 10.2.2.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  mtu 9216
  ip address 20.2.2.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  mtu 9216
  ip address 30.2.2.1/30
  no shutdown

interface Ethernet1/4
  description to_COD1
  no switchport
  mac-address 0c10.0000.1b12
  no ip redirects
  ip address 12.12.12.6/30
  no shutdown
  
interface loopback10
  ip address 10.20.10.2/32
  
router bgp 65500
  router-id 10.20.10.2
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
  address-family l2vpn evpn
    retain route-target all
  template peer OVERLAY
    update-source loopback10
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map UNCHANGED out
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    timers 3 9
    address-family ipv4 unicast
  neighbor 1.2.1.1
    inherit peer OVERLAY
    remote-as 65501
  neighbor 1.2.1.2
    inherit peer OVERLAY
    remote-as 65502
  neighbor 1.2.1.3
    inherit peer OVERLAY
    remote-as 65503
  neighbor 10.2.2.2
    inherit peer UNDERLAY
    remote-as 65501
    address-family ipv4 unicast
  neighbor 10.10.10.1
    inherit peer OVERLAY
    remote-as 65400
  neighbor 12.12.12.5
    inherit peer UNDERLAY
    remote-as 65400
  neighbor 20.2.2.2
    inherit peer UNDERLAY
    remote-as 65502
    address-family ipv4 unicast
  neighbor 30.2.2.2
    inherit peer UNDERLAY
    remote-as 65503
    address-family ipv4 unicast
```

</details>

<details>

<summary>EDGE</summary>

```
feature bgp

route-map DIRECT-ROUTES-MAP permit 10

interface Ethernet1/1
  no switchport
  ip address 192.168.3.2/30
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 192.168.3.6/30
  no shutdown
  
interface loopback1
  ip address 4.4.4.4/32
  
router bgp 65600
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  neighbor 192.168.3.1
    remote-as 65503
    address-family ipv4 unicast
      as-override
      disable-peer-as-check
  neighbor 192.168.3.5
    remote-as 65503
    address-family ipv4 unicast
      as-override
      disable-peer-as-check
```

</details>

#### Проверка

ipv4 маршруты

<details>

<summary>LEAF1-1</summary>

```
LEAF1-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.1, local AS number 65401
BGP table version is 49, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.1        4 65400      28306      28294       49    0    0 23:34:38 24        
10.1.2.1        4 65400      28306      28292       49    0    0 23:34:34 24        
LEAF1-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0, attached
    *via 1.1.1.1, Lo1, [0/0], 23:37:16, local
    *via 1.1.1.1, Lo1, [0/0], 23:37:16, direct
1.1.1.2/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:35:15, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:35:11, bgp-65401, external, tag 65400
1.1.1.3/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:35:09, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:35:07, bgp-65401, external, tag 65400
1.1.1.4/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:35:15, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:35:11, bgp-65401, external, tag 65400
1.2.1.1/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:32:15, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:32:15, bgp-65401, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:32:15, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:32:15, bgp-65401, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:34:55, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:34:52, bgp-65401, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 23:35:31, direct
10.1.1.2/32, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 23:35:31, local
10.1.1.100/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:32:15, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:32:15, bgp-65401, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 23:35:31, direct
10.1.2.2/32, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 23:35:31, local
10.2.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:34:52, bgp-65401, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:34:55, bgp-65401, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:35:15, bgp-65401, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:35:11, bgp-65401, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:34:52, bgp-65401, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:34:55, bgp-65401, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:35:11, bgp-65401, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:35:15, bgp-65401, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:35:15, bgp-65401, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:35:11, bgp-65401, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:34:52, bgp-65401, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:34:55, bgp-65401, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:35:15, bgp-65401, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:35:11, bgp-65401, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:34:52, bgp-65401, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:34:55, bgp-65401, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 23:35:15, bgp-65401, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 23:35:11, bgp-65401, external, tag 65400
192.168.2.0/30, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 23:34:54, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 23:34:52, bgp-65401, external, tag 65400

LEAF1-1# show ip route vrf TENANT_1
IP Route Table for VRF "TENANT_1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:36:47, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:49:33, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 23:40:06, direct
172.17.67.1/32, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 23:40:06, local
172.17.67.2/32, ubest/mbest: 1/0, attached
    *via 172.17.67.2, Vlan67, [190/0], 00:38:31, hmm
172.17.67.3/32, ubest/mbest: 1/0, attached
    *via 172.17.67.3, Vlan67, [190/0], 00:38:31, hmm
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.1.1.2%default, [20/0], 23:24:47, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1010102 encap: VXLAN
 
172.17.67.5/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:47:22, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.6/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:47:28, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:36:47, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:36:47, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF1-2</summary>

```
LEAF1-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.2, local AS number 65402
BGP table version is 49, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
20.1.1.1        4 65400      28309      28297       49    0    0 23:34:47 24        
20.1.2.1        4 65400      28307      28295       49    0    0 23:34:42 24        
LEAF1-2# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:35:21, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:35:14, bgp-65402, external, tag 65400
1.1.1.2/32, ubest/mbest: 2/0, attached
    *via 1.1.1.2, Lo1, [0/0], 23:37:24, local
    *via 1.1.1.2, Lo1, [0/0], 23:37:24, direct
1.1.1.3/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:35:15, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:35:13, bgp-65402, external, tag 65400
1.1.1.4/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:35:21, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:35:14, bgp-65402, external, tag 65400
1.2.1.1/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:32:21, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:32:21, bgp-65402, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:32:21, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:32:21, bgp-65402, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:35:01, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:34:58, bgp-65402, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:21, bgp-65402, external, tag 65400
10.1.1.100/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:32:21, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:32:21, bgp-65402, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:35:14, bgp-65402, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:34:58, bgp-65402, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:01, bgp-65402, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:21, bgp-65402, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:35:14, bgp-65402, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:34:58, bgp-65402, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:01, bgp-65402, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:35:14, bgp-65402, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:21, bgp-65402, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0, attached
    *via 20.1.1.2, Eth1/1, [0/0], 23:35:39, direct
20.1.1.2/32, ubest/mbest: 1/0, attached
    *via 20.1.1.2, Eth1/1, [0/0], 23:35:39, local
20.1.2.0/30, ubest/mbest: 1/0, attached
    *via 20.1.2.2, Eth1/2, [0/0], 23:35:38, direct
20.1.2.2/32, ubest/mbest: 1/0, attached
    *via 20.1.2.2, Eth1/2, [0/0], 23:35:38, local
20.2.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:34:58, bgp-65402, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:01, bgp-65402, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:21, bgp-65402, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:35:14, bgp-65402, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:34:58, bgp-65402, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:01, bgp-65402, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 23:35:21, bgp-65402, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 23:35:14, bgp-65402, external, tag 65400
192.168.2.0/30, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 23:35:00, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 23:34:58, bgp-65402, external, tag 65400

LEAF1-2# show ip route vrf TENANT_1
IP Route Table for VRF "TENANT_1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:36:52, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:49:38, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 23:40:13, direct
172.17.67.1/32, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 23:40:13, local
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 23:25:17, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 23:24:57, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0, attached
    *via 172.17.67.4, Vlan67, [190/0], 00:38:37, hmm
172.17.67.5/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:47:26, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.6/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:47:32, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:36:52, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:36:52, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF1-3</summary>

```
LEAF1-3(config-if)# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.3, local AS number 65403
BGP table version is 49, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
30.1.1.1        4 65400      28305      28292       49    0    0 23:34:34 24        
30.1.2.1        4 65400      28304      28293       49    0    0 23:34:35 24        
LEAF1-3(config-if)# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
1.1.1.2/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
1.1.1.3/32, ubest/mbest: 2/0, attached
    *via 1.1.1.3, Lo1, [0/0], 23:37:12, local
    *via 1.1.1.3, Lo1, [0/0], 23:37:12, direct
1.1.1.4/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
1.2.1.1/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:32:24, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:32:24, bgp-65403, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:32:24, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:32:24, bgp-65403, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:35:05, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:35:02, bgp-65403, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
10.1.1.100/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:32:24, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:32:24, bgp-65403, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:02, bgp-65403, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:05, bgp-65403, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:02, bgp-65403, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:05, bgp-65403, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:02, bgp-65403, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:05, bgp-65403, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0, attached
    *via 30.1.1.2, Eth1/1, [0/0], 23:35:31, direct
30.1.1.2/32, ubest/mbest: 1/0, attached
    *via 30.1.1.2, Eth1/1, [0/0], 23:35:31, local
30.1.2.0/30, ubest/mbest: 1/0, attached
    *via 30.1.2.2, Eth1/2, [0/0], 23:35:30, direct
30.1.2.2/32, ubest/mbest: 1/0, attached
    *via 30.1.2.2, Eth1/2, [0/0], 23:35:30, local
30.2.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:02, bgp-65403, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:05, bgp-65403, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 23:35:12, bgp-65403, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 23:35:20, bgp-65403, external, tag 65400
192.168.2.0/30, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 23:35:04, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 23:35:02, bgp-65403, external, tag 65400

LEAF1-3(config-if)#  show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:24, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.0/24, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 23:40:30, direct
172.16.65.1/32, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 23:40:30, local
172.16.65.2/32, ubest/mbest: 1/0, attached
    *via 172.16.65.2, Vlan65, [190/0], 00:08:59, hmm
172.16.65.3/32, ubest/mbest: 1/0
    *via 10.1.1.100%default, [20/0], 00:50:11, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0xa010164 encap: VXLAN
 
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.1.1.4%default, [20/0], 23:24:55, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1010104 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:24, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:24, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:24, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF1-4</summary>

```
LEAF1-4(config)# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.4, local AS number 65404
BGP table version is 49, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
40.1.1.1        4 65400      28306      28294       49    0    0 23:34:42 24        
40.1.2.1        4 65400      28306      28294       49    0    0 23:34:41 24        
LEAF1-4(config)# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:35:27, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
1.1.1.2/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:35:27, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
1.1.1.3/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:35:20, bgp-65404, external, tag 65400
1.1.1.4/32, ubest/mbest: 2/0, attached
    *via 1.1.1.4, Lo1, [0/0], 23:37:24, local
    *via 1.1.1.4, Lo1, [0/0], 23:37:24, direct
1.2.1.1/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:32:28, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:32:28, bgp-65404, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:32:28, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:32:28, bgp-65404, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:35:08, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:35:05, bgp-65404, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:27, bgp-65404, external, tag 65400
10.1.1.100/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:32:28, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:32:28, bgp-65404, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:05, bgp-65404, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:08, bgp-65404, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:27, bgp-65404, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:05, bgp-65404, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:08, bgp-65404, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:27, bgp-65404, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:27, bgp-65404, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:05, bgp-65404, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:08, bgp-65404, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:27, bgp-65404, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:21, bgp-65404, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 23:35:05, bgp-65404, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 23:35:08, bgp-65404, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0, attached
    *via 40.1.1.2, Eth1/1, [0/0], 23:35:40, direct
40.1.1.2/32, ubest/mbest: 1/0, attached
    *via 40.1.1.2, Eth1/1, [0/0], 23:35:40, local
40.1.2.0/30, ubest/mbest: 1/0, attached
    *via 40.1.2.2, Eth1/2, [0/0], 23:35:40, direct
40.1.2.2/32, ubest/mbest: 1/0, attached
    *via 40.1.2.2, Eth1/2, [0/0], 23:35:40, local
192.168.2.0/30, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 23:35:07, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 23:35:05, bgp-65404, external, tag 65400

LEAF1-4(config)# show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:28, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.1.1.3%default, [20/0], 00:49:24, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1010103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0
    *via 10.1.1.100%default, [20/0], 00:50:14, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0xa010164 encap: VXLAN
 
172.16.66.0/24, ubest/mbest: 1/0, attached
    *via 172.16.66.1, Vlan66, [0/0], 23:40:42, direct
172.16.66.1/32, ubest/mbest: 1/0, attached
    *via 172.16.66.1, Vlan66, [0/0], 23:40:42, local
172.16.66.2/32, ubest/mbest: 1/0, attached
    *via 172.16.66.2, Vlan66, [190/0], 00:39:06, hmm
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:28, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:28, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:37:28, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>SPINE1-1</summary>

```
SPINE1-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.10.10.1, local AS number 65400
BGP table version is 32, IPv4 Unicast config peers 5, capable peers 5
26 network entries and 31 paths using 8072 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/60]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.2        4 65401      28303      28304       32    0    0 23:35:02 3         
12.12.12.6      4 65500      28309      28305       32    0    0 23:35:01 13        
20.1.1.2        4 65402      28303      28304       32    0    0 23:35:03 3         
30.1.1.2        4 65403      28298      28299       32    0    0 23:34:47 3         
40.1.1.2        4 65404      28299      28300       32    0    0 23:34:53 3         
SPINE1-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 23:35:40, bgp-65400, external, tag 65401
1.1.1.2/32, ubest/mbest: 1/0
    *via 20.1.1.2, [20/0], 23:35:40, bgp-65400, external, tag 65402
1.1.1.3/32, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 23:35:34, bgp-65400, external, tag 65403
1.1.1.4/32, ubest/mbest: 1/0
    *via 40.1.1.2, [20/0], 23:35:40, bgp-65400, external, tag 65404
1.2.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:32:40, bgp-65400, external, tag 65500
1.2.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:32:40, bgp-65400, external, tag 65500
1.2.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:20, bgp-65400, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 23:36:00, direct
10.1.1.1/32, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 23:36:00, local
10.1.1.100/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:32:40, bgp-65400, external, tag 65500
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 23:35:40, bgp-65400, external, tag 65401
10.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:19, bgp-65400, external, tag 65500
10.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:20, bgp-65400, external, tag 65500
10.10.10.1/32, ubest/mbest: 2/0, attached
    *via 10.10.10.1, Lo10, [0/0], 23:37:42, local
    *via 10.10.10.1, Lo10, [0/0], 23:37:42, direct
10.20.10.2/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:20, bgp-65400, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0, attached
    *via 12.12.12.5, Eth1/5, [0/0], 23:35:59, direct
12.12.12.5/32, ubest/mbest: 1/0, attached
    *via 12.12.12.5, Eth1/5, [0/0], 23:35:59, local
20.1.1.0/30, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 23:36:00, direct
20.1.1.1/32, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 23:36:00, local
20.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.2, [20/0], 23:35:40, bgp-65400, external, tag 65402
20.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:11, bgp-65400, external, tag 65500
20.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:20, bgp-65400, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 23:36:00, direct
30.1.1.1/32, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 23:36:00, local
30.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 23:35:34, bgp-65400, external, tag 65403
30.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:20, bgp-65400, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:20, bgp-65400, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0, attached
    *via 40.1.1.1, Eth1/4, [0/0], 23:36:00, direct
40.1.1.1/32, ubest/mbest: 1/0, attached
    *via 40.1.1.1, Eth1/4, [0/0], 23:36:00, local
40.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.2, [20/0], 23:35:40, bgp-65400, external, tag 65404
192.168.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 23:35:19, bgp-65400, external, tag 65500
```

</details>

<details>

<summary>SPINE1-2</summary>

```
SPINE1-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.10.10.2, local AS number 65400
BGP table version is 32, IPv4 Unicast config peers 5, capable peers 5
26 network entries and 31 paths using 8072 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/60]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.2.2        4 65401      28302      28305       32    0    0 23:35:00 3         
12.12.12.2      4 65500      28310      28305       32    0    0 23:35:00 13        
20.1.2.2        4 65402      28302      28303       32    0    0 23:35:00 3         
30.1.2.2        4 65403      28299      28300       32    0    0 23:34:51 3         
40.1.2.2        4 65404      28300      28301       32    0    0 23:34:55 3         
SPINE1-2# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 10.1.2.2, [20/0], 23:35:45, bgp-65400, external, tag 65401
1.1.1.2/32, ubest/mbest: 1/0
    *via 20.1.2.2, [20/0], 23:35:45, bgp-65400, external, tag 65402
1.1.1.3/32, ubest/mbest: 1/0
    *via 30.1.2.2, [20/0], 23:35:36, bgp-65400, external, tag 65403
1.1.1.4/32, ubest/mbest: 1/0
    *via 40.1.2.2, [20/0], 23:35:44, bgp-65400, external, tag 65404
1.2.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:32:44, bgp-65400, external, tag 65500
1.2.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:32:44, bgp-65400, external, tag 65500
1.2.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.2, [20/0], 23:35:45, bgp-65400, external, tag 65401
10.1.1.100/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:32:44, bgp-65400, external, tag 65500
10.1.2.0/30, ubest/mbest: 1/0, attached
    *via 10.1.2.1, Eth1/1, [0/0], 23:35:56, direct
10.1.2.1/32, ubest/mbest: 1/0, attached
    *via 10.1.2.1, Eth1/1, [0/0], 23:35:56, local
10.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
10.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
10.10.10.2/32, ubest/mbest: 2/0, attached
    *via 10.10.10.2, Lo10, [0/0], 23:37:38, local
    *via 10.10.10.2, Lo10, [0/0], 23:37:38, direct
10.20.10.1/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0, attached
    *via 12.12.12.1, Eth1/5, [0/0], 23:35:54, direct
12.12.12.1/32, ubest/mbest: 1/0, attached
    *via 12.12.12.1, Eth1/5, [0/0], 23:35:54, local
20.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.2, [20/0], 23:35:45, bgp-65400, external, tag 65402
20.1.2.0/30, ubest/mbest: 1/0, attached
    *via 20.1.2.1, Eth1/2, [0/0], 23:35:55, direct
20.1.2.1/32, ubest/mbest: 1/0, attached
    *via 20.1.2.1, Eth1/2, [0/0], 23:35:55, local
20.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
20.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:15, bgp-65400, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.2, [20/0], 23:35:36, bgp-65400, external, tag 65403
30.1.2.0/30, ubest/mbest: 1/0, attached
    *via 30.1.2.1, Eth1/3, [0/0], 23:35:55, direct
30.1.2.1/32, ubest/mbest: 1/0, attached
    *via 30.1.2.1, Eth1/3, [0/0], 23:35:55, local
30.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.2, [20/0], 23:35:44, bgp-65400, external, tag 65404
40.1.2.0/30, ubest/mbest: 1/0, attached
    *via 40.1.2.1, Eth1/4, [0/0], 23:35:55, direct
40.1.2.1/32, ubest/mbest: 1/0, attached
    *via 40.1.2.1, Eth1/4, [0/0], 23:35:55, local
192.168.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 23:35:21, bgp-65400, external, tag 65500
```

</details>

<details>

<summary>LEAF2-1</summary>

```
LEAF2-1(config-if)# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.2.1.1, local AS number 65501
BGP table version is 76, IPv4 Unicast config peers 3, capable peers 3
29 network entries and 77 paths using 13076 bytes of memory
BGP attribute entries [17/6120], BGP AS path entries [16/208]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.1.1        4 65500      28302      28293       76    0    0 23:34:28 23        
10.2.2.1        4 65500      28302      28294       76    0    0 23:34:29 23        
192.168.2.2     4 65502      28298      28291       76    0    0 23:34:17 26        
LEAF2-1(config-if)# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
1.1.1.2/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
1.1.1.3/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
1.1.1.4/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
1.2.1.1/32, ubest/mbest: 2/0, attached
    *via 1.2.1.1, Lo1, [0/0], 23:32:31, local
    *via 1.2.1.1, Lo1, [0/0], 23:32:31, direct
1.2.1.2/32, ubest/mbest: 1/0
    *via 192.168.2.2, [20/0], 23:32:31, bgp-65501, external, tag 65502
1.2.1.3/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
10.1.1.100/32, ubest/mbest: 2/0, attached
    *via 10.1.1.100, Lo1, [0/0], 23:32:31, local
    *via 10.1.1.100, Lo1, [0/0], 23:32:31, direct
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
10.2.1.0/30, ubest/mbest: 1/0, attached
    *via 10.2.1.2, Eth1/1, [0/0], 23:35:43, direct
10.2.1.2/32, ubest/mbest: 1/0, attached
    *via 10.2.1.2, Eth1/1, [0/0], 23:35:43, local
10.2.2.0/30, ubest/mbest: 1/0, attached
    *via 10.2.2.2, Eth1/2, [0/0], 23:35:43, direct
10.2.2.2/32, ubest/mbest: 1/0, attached
    *via 10.2.2.2, Eth1/2, [0/0], 23:35:43, local
10.10.10.1/32, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
10.10.10.2/32, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
10.20.10.1/32, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
10.20.10.2/32, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
20.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
20.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
20.2.1.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
20.2.2.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
30.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
30.2.1.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 23:35:10, bgp-65501, external, tag 65500
40.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 23:35:08, bgp-65501, external, tag 65500
192.168.2.0/30, ubest/mbest: 1/0, attached
    *via 192.168.2.1, Vlan2, [0/0], 23:35:29, direct
192.168.2.1/32, ubest/mbest: 1/0, attached
    *via 192.168.2.1, Vlan2, [0/0], 23:35:29, local

LEAF2-1(config-if)#  show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:55, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.0/24, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 23:38:55, direct
172.16.65.1/32, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 23:38:55, local
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.1.1.3%default, [20/0], 00:49:44, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1010103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0, attached
    *via 172.16.65.3, Vlan65, [190/0], 00:08:25, hmm
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.1.1.4%default, [20/0], 23:25:18, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1010104 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:55, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:26:13, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:25:53, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:25:47, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:55, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:55, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF2-2</summary>

```
LEAF2-2(config-if)# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.2.1.2, local AS number 65502
BGP table version is 78, IPv4 Unicast config peers 3, capable peers 3
29 network entries and 79 paths using 13268 bytes of memory
BGP attribute entries [17/6120], BGP AS path entries [16/208]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
20.2.1.1        4 65500      28298      28288       78    0    0 23:34:18 23        
20.2.2.1        4 65500      28299      28289       78    0    0 23:34:21 23        
192.168.2.1     4 65501      28302      28286       78    0    0 23:34:19 28        
LEAF2-2(config-if)# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
1.1.1.2/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
1.1.1.3/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
1.1.1.4/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
1.2.1.1/32, ubest/mbest: 1/0
    *via 192.168.2.1, [20/0], 23:32:34, bgp-65502, external, tag 65501
1.2.1.2/32, ubest/mbest: 2/0, attached
    *via 1.2.1.2, Lo2, [0/0], 23:32:34, local
    *via 1.2.1.2, Lo2, [0/0], 23:32:34, direct
1.2.1.3/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
10.1.1.100/32, ubest/mbest: 2/0, attached
    *via 10.1.1.100, Lo2, [0/0], 23:32:34, local
    *via 10.1.1.100, Lo2, [0/0], 23:32:34, direct
10.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
10.2.1.0/30, ubest/mbest: 1/0
    *via 192.168.2.1, [20/0], 23:35:08, bgp-65502, external, tag 65501
10.2.2.0/30, ubest/mbest: 1/0
    *via 192.168.2.1, [20/0], 23:35:08, bgp-65502, external, tag 65501
10.10.10.1/32, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
10.10.10.2/32, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
10.20.10.1/32, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
10.20.10.2/32, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
20.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
20.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
20.2.1.0/30, ubest/mbest: 1/0, attached
    *via 20.2.1.2, Eth1/1, [0/0], 23:35:43, direct
20.2.1.2/32, ubest/mbest: 1/0, attached
    *via 20.2.1.2, Eth1/1, [0/0], 23:35:43, local
20.2.2.0/30, ubest/mbest: 1/0, attached
    *via 20.2.2.2, Eth1/2, [0/0], 23:35:43, direct
20.2.2.2/32, ubest/mbest: 1/0, attached
    *via 20.2.2.2, Eth1/2, [0/0], 23:35:43, local
30.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
30.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
30.2.1.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 23:35:03, bgp-65502, external, tag 65500
40.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 23:35:06, bgp-65502, external, tag 65500
192.168.2.0/30, ubest/mbest: 1/0, attached
    *via 192.168.2.2, Vlan2, [0/0], 23:35:33, direct
192.168.2.2/32, ubest/mbest: 1/0, attached
    *via 192.168.2.2, Vlan2, [0/0], 23:35:33, local

LEAF2-2(config-if)#  show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:59, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.0/24, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 23:38:59, direct
172.16.65.1/32, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 23:38:59, local
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.1.1.3%default, [20/0], 00:49:48, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1010103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0, attached
    *via 172.16.65.3, Vlan65, [190/0], 00:08:29, hmm
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.1.1.4%default, [20/0], 23:25:22, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1010104 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:59, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:26:17, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:25:57, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:25:51, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:59, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 23:35:59, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF2-3</summary>

```
LEAF2-3(config-if)# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.2.1.3, local AS number 65503
BGP table version is 49, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/92]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
30.2.1.1        4 65500      28312      28300       49    0    0 23:34:56 24        
30.2.2.1        4 65500      28311      28300       49    0    0 23:34:56 24        
LEAF2-3(config-if)# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
1.1.1.2/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
1.1.1.3/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
1.1.1.4/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
1.2.1.1/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:32:37, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:32:37, bgp-65503, external, tag 65500
1.2.1.2/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:32:37, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:32:37, bgp-65503, external, tag 65500
1.2.1.3/32, ubest/mbest: 2/0, attached
    *via 1.2.1.3, Lo3, [0/0], 23:37:29, local
    *via 1.2.1.3, Lo3, [0/0], 23:37:29, direct
10.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
10.1.1.100/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:32:37, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:32:37, bgp-65503, external, tag 65500
10.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
10.2.1.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
10.2.2.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
10.10.10.1/32, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
10.10.10.2/32, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
10.20.10.1/32, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
10.20.10.2/32, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
20.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
20.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
20.2.1.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
20.2.2.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
30.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
30.2.1.0/30, ubest/mbest: 1/0, attached
    *via 30.2.1.2, Eth1/1, [0/0], 23:35:50, direct
30.2.1.2/32, ubest/mbest: 1/0, attached
    *via 30.2.1.2, Eth1/1, [0/0], 23:35:50, local
30.2.2.0/30, ubest/mbest: 1/0, attached
    *via 30.2.2.2, Eth1/2, [0/0], 23:35:50, direct
30.2.2.2/32, ubest/mbest: 1/0, attached
    *via 30.2.2.2, Eth1/2, [0/0], 23:35:50, local
40.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 23:35:17, bgp-65503, external, tag 65500
40.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
192.168.2.0/30, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 23:35:14, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 23:35:16, bgp-65503, external, tag 65500

LEAF2-3(config-if)# show ip route vrf TENANT_1
IP Route Table for VRF "TENANT_1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 23:37:57, bgp-65503, external, tag 65600
172.16.65.2/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 00:49:08, bgp-65503, external, tag 65600
172.16.65.3/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 00:49:58, bgp-65503, external, tag 65600
172.16.66.2/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 23:24:42, bgp-65503, external, tag 65600
172.17.67.0/24, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 23:40:23, direct
172.17.67.1/32, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 23:40:23, local
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 23:25:37, bgp-65503, external, tag 65500, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 23:25:17, bgp-65503, external, tag 65500, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.1.1.2%default, [20/0], 23:25:12, bgp-65503, external, tag 65500, segid: 100501 tunnelid: 0x1010102 encap: VXLAN
 
172.17.67.5/32, ubest/mbest: 1/0, attached
    *via 172.17.67.5, Vlan67, [190/0], 00:38:51, hmm
172.17.67.6/32, ubest/mbest: 1/0, attached
    *via 172.17.67.6, Vlan67, [190/0], 00:38:51, hmm
192.168.3.0/30, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 23:37:57, bgp-65503, external, tag 65600
192.168.3.4/30, ubest/mbest: 1/0, attached
    *via 192.168.3.5, Eth1/6, [0/0], 23:38:43, direct
192.168.3.5/32, ubest/mbest: 1/0, attached
    *via 192.168.3.5, Eth1/6, [0/0], 23:38:43, local
```

</details>

<details>

<summary>SPINE2-1</summary>

```
SPINE2-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.20.10.1, local AS number 65500
BGP table version is 36, IPv4 Unicast config peers 4, capable peers 4
26 network entries and 36 paths using 8552 bytes of memory
BGP attribute entries [11/3960], BGP AS path entries [10/84]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.1.2        4 65501      28302      28300       36    0    0 23:34:44 6         
12.12.12.1      4 65400      28311      28305       36    0    0 23:35:05 14        
20.2.1.2        4 65502      28305      28295       36    0    0 23:34:32 8         
30.2.1.2        4 65503      28304      28305       36    0    0 23:35:07 3         
SPINE2-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
1.1.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
1.1.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
1.1.1.4/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
1.2.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.2, [20/0], 23:32:47, bgp-65500, external, tag 65501
1.2.1.2/32, ubest/mbest: 1/0
    *via 20.2.1.2, [20/0], 23:32:47, bgp-65500, external, tag 65502
1.2.1.3/32, ubest/mbest: 1/0
    *via 30.2.1.2, [20/0], 23:35:24, bgp-65500, external, tag 65503
10.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
10.1.1.100/32, ubest/mbest: 1/0
    *via 20.2.1.2, [20/0], 23:32:47, bgp-65500, external, tag 65502
10.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 23:36:04, direct
10.2.1.1/32, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 23:36:04, local
10.2.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.2, [20/0], 23:35:24, bgp-65500, external, tag 65501
10.10.10.2/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
10.20.10.1/32, ubest/mbest: 2/0, attached
    *via 10.20.10.1, Lo10, [0/0], 23:37:42, local
    *via 10.20.10.1, Lo10, [0/0], 23:37:42, direct
12.12.12.0/30, ubest/mbest: 1/0, attached
    *via 12.12.12.2, Eth1/4, [0/0], 23:36:03, direct
12.12.12.2/32, ubest/mbest: 1/0, attached
    *via 12.12.12.2, Eth1/4, [0/0], 23:36:03, local
20.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0, attached
    *via 20.2.1.1, Eth1/2, [0/0], 23:36:04, direct
20.2.1.1/32, ubest/mbest: 1/0, attached
    *via 20.2.1.1, Eth1/2, [0/0], 23:36:04, local
20.2.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.2, [20/0], 23:35:18, bgp-65500, external, tag 65502
30.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0, attached
    *via 30.2.1.1, Eth1/3, [0/0], 23:36:04, direct
30.2.1.1/32, ubest/mbest: 1/0, attached
    *via 30.2.1.1, Eth1/3, [0/0], 23:36:04, local
30.2.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.2, [20/0], 23:35:24, bgp-65500, external, tag 65503
40.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 23:35:24, bgp-65500, external, tag 65400
192.168.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.2, [20/0], 23:35:24, bgp-65500, external, tag 65501
```

</details>

<details>

<summary>SPINE2-2</summary>

```
SPINE2-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.20.10.2, local AS number 65500
BGP table version is 37, IPv4 Unicast config peers 4, capable peers 4
26 network entries and 36 paths using 8552 bytes of memory
BGP attribute entries [11/3960], BGP AS path entries [10/84]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.2.2        4 65501      28305      28302       37    0    0 23:34:49 6         
12.12.12.5      4 65400      28314      28307       37    0    0 23:35:12 14        
20.2.2.2        4 65502      28308      28298       37    0    0 23:34:39 8         
30.2.2.2        4 65503      28306      28306       37    0    0 23:35:11 3         
SPINE2-2# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
1.1.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
1.1.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
1.1.1.4/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
1.2.1.1/32, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 23:32:51, bgp-65500, external, tag 65501
1.2.1.2/32, ubest/mbest: 1/0
    *via 20.2.2.2, [20/0], 23:32:51, bgp-65500, external, tag 65502
1.2.1.3/32, ubest/mbest: 1/0
    *via 30.2.2.2, [20/0], 23:35:31, bgp-65500, external, tag 65503
10.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
10.1.1.100/32, ubest/mbest: 1/0
    *via 20.2.2.2, [20/0], 23:32:51, bgp-65500, external, tag 65502
10.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 23:35:30, bgp-65500, external, tag 65501
10.2.2.0/30, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/1, [0/0], 23:36:08, direct
10.2.2.1/32, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/1, [0/0], 23:36:08, local
10.10.10.1/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
10.20.10.2/32, ubest/mbest: 2/0, attached
    *via 10.20.10.2, Lo20, [0/0], 23:37:48, local
    *via 10.20.10.2, Lo20, [0/0], 23:37:48, direct
12.12.12.4/30, ubest/mbest: 1/0, attached
    *via 12.12.12.6, Eth1/4, [0/0], 23:36:08, direct
12.12.12.6/32, ubest/mbest: 1/0, attached
    *via 12.12.12.6, Eth1/4, [0/0], 23:36:08, local
20.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.2, [20/0], 23:35:22, bgp-65500, external, tag 65502
20.2.2.0/30, ubest/mbest: 1/0, attached
    *via 20.2.2.1, Eth1/2, [0/0], 23:36:08, direct
20.2.2.1/32, ubest/mbest: 1/0, attached
    *via 20.2.2.1, Eth1/2, [0/0], 23:36:08, local
30.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.2, [20/0], 23:35:31, bgp-65500, external, tag 65503
30.2.2.0/30, ubest/mbest: 1/0, attached
    *via 30.2.2.1, Eth1/3, [0/0], 23:36:08, direct
30.2.2.1/32, ubest/mbest: 1/0, attached
    *via 30.2.2.1, Eth1/3, [0/0], 23:36:08, local
40.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 23:35:31, bgp-65500, external, tag 65400
192.168.2.0/30, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 23:35:30, bgp-65500, external, tag 65501
```

</details>

<details>

<summary>EDGE</summary>

```
EDGE# show ip route 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 2/0, attached
    *via 4.4.4.4, Lo1, [0/0], 23:49:10, local
    *via 4.4.4.4, Lo1, [0/0], 23:49:10, direct
172.16.65.2/32, ubest/mbest: 1/0
    *via 192.168.3.1, [20/0], 00:57:59, bgp-65600, external, tag 65503
172.16.65.3/32, ubest/mbest: 1/0
    *via 192.168.3.1, [20/0], 00:58:49, bgp-65600, external, tag 65503
172.16.66.2/32, ubest/mbest: 1/0
    *via 192.168.3.1, [20/0], 23:33:34, bgp-65600, external, tag 65503
172.17.67.0/24, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 23:46:48, bgp-65600, external, tag 65503
172.17.67.2/32, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 23:34:28, bgp-65600, external, tag 65503
172.17.67.3/32, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 23:34:08, bgp-65600, external, tag 65503
172.17.67.4/32, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 23:34:03, bgp-65600, external, tag 65503
192.168.3.0/30, ubest/mbest: 1/0, attached
    *via 192.168.3.2, Eth1/1, [0/0], 23:47:29, direct
192.168.3.2/32, ubest/mbest: 1/0, attached
    *via 192.168.3.2, Eth1/1, [0/0], 23:47:29, local
192.168.3.4/30, ubest/mbest: 1/0, attached
    *via 192.168.3.6, Eth1/2, [0/0], 23:47:29, direct
192.168.3.6/32, ubest/mbest: 1/0, attached
    *via 192.168.3.6, Eth1/2, [0/0], 23:47:29, local
```

</details>



l2vpn evpn маршруты

<details>

<summary>LEAF1-1</summary>

```
```

</details>





Проверка связности между endhost-1, находящимся в VRF COD, и endhost-2, endhost-3, находящимися в VRF TENANT\_1, а также Loopback интерфейсом на EDGE роутере



#### Дампы Wireshark







#### Выводы

Таким образом, нам удалось настроить ликинг между VRF COD и VRF TENANT\_1 с использованием ipv4 BGP сессий в соответствующих vrf между LEAF3 и EDGE роутером, находящимся в другом логическом сегменте.

Маршруты появились на LEAF3 как в ipv4, так и в l2vpn evpn таблицах.&#x20;

Связность между узлами сети имеется.
