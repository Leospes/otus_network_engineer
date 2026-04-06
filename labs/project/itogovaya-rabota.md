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

#### Проверка

ipv4 и l2vpn evpn маршруты

LEAF1



Проверка связности между endhost-1, находящимся в VRF COD, и endhost-2, endhost-3, находящимися в VRF TENANT\_1, а также Loopback интерфейсом на EDGE роутере



#### Дампы Wireshark







#### Выводы

Таким образом, нам удалось настроить ликинг между VRF COD и VRF TENANT\_1 с использованием ipv4 BGP сессий в соответствующих vrf между LEAF3 и EDGE роутером, находящимся в другом логическом сегменте.

Маршруты появились на LEAF3 как в ipv4, так и в l2vpn evpn таблицах.&#x20;

Связность между узлами сети имеется.
