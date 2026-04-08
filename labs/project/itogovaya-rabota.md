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
vlan 1,65,500
vlan 65
  name VLAN_65
  vn-segment 100065
vlan 500
  name L3-VNI-COD
  vn-segment 100500

route-map DIRECT-ROUTES-MAP permit 10
vrf context COD
  vni 100500
  rd 65403:1
  address-family ipv4 unicast
    route-target import 11:11 evpn
    route-target import 1:1
    route-target export 11:11 evpn
    route-target export 1:1
	
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
    address-family l2vpn evpn
      allowas-in 3
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
    address-family l2vpn evpn
      allowas-in 3
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
  advertise virtual-rmac
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
  advertise virtual-rmac
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
    advertise-pip
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

<details>

<summary>endhost-1</summary>

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
56: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:ed:27:82:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.67.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:edff:fe27:8200/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.17.67.1 dev eth0 
172.17.67.0/24 dev eth0 scope link  src 172.17.67.2 
```

</details>

<details>

<summary>endhost-2</summary>

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
60: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:00:c0:83:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.67.3/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:ff:fec0:8300/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.17.67.1 dev eth0 
172.17.67.0/24 dev eth0 scope link  src 172.17.67.3
```

</details>

<details>

<summary>endhost-3</summary>

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
57: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:86:32:42:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.67.4/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:86ff:fe32:4200/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.17.67.1 dev eth0 
172.17.67.0/24 dev eth0 scope link  src 172.17.67.4
```

</details>

<details>

<summary>endhost-4</summary>

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
58: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:b3:3e:be:00 brd ff:ff:ff:ff:ff:ff
    inet 172.16.65.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:b3ff:fe3e:be00/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.16.65.1 dev eth0 
172.16.65.0/24 dev eth0 scope link  src 172.16.65.2 
```

</details>

<details>

<summary>endhost-5</summary>

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
59: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:22:6f:26:00 brd ff:ff:ff:ff:ff:ff
    inet 172.16.66.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:22ff:fe6f:2600/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.16.66.1 dev eth0 
172.16.66.0/24 dev eth0 scope link  src 172.16.66.2 
```

</details>

<details>

<summary>endhost-6</summary>

```
interface Vlan65
 ip address 172.16.65.3 255.255.255.0
end

ip route 0.0.0.0 0.0.0.0 172.16.65.1
```

</details>

<details>

<summary>endhost-7</summary>

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
55: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:ee:8b:bc:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.67.5/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:eeff:fe8b:bc00/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.17.67.1 dev eth0 
172.17.67.0/24 dev eth0 scope link  src 172.17.67.5 
```

</details>

<details>

<summary>endhost-8</summary>

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
54: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:24:8c:3e:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.67.6/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:24ff:fe8c:3e00/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.17.67.1 dev eth0 
172.17.67.0/24 dev eth0 scope link  src 172.17.67.6 
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
BGP table version is 153, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.1        4 65400      57807      57782      153    0    0 00:55:12 24        
10.1.2.1        4 65400      57810      57780      153    0    0 00:55:06 24        
LEAF1-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0, attached
    *via 1.1.1.1, Lo1, [0/0], 2d00h, local
    *via 1.1.1.1, Lo1, [0/0], 2d00h, direct
1.1.1.2/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
1.1.1.3/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:33:56, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:33:56, bgp-65401, external, tag 65400
1.1.1.4/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:33:20, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:33:20, bgp-65401, external, tag 65400
1.2.1.1/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 2d00h, direct
10.1.1.2/32, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 2d00h, local
10.1.1.100/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 2d00h, direct
10.1.2.2/32, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 2d00h, local
10.2.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:56:13, bgp-65401, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:56:07, bgp-65401, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:56:05, bgp-65401, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:56:02, bgp-65401, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:56:06, bgp-65401, external, tag 65400
192.168.2.0/30, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:55:54, bgp-65401, external, tag 65400
    *via 10.1.2.1, [20/0], 00:55:54, bgp-65401, external, tag 65400

LEAF1-1# show ip route vrf TENANT_1
IP Route Table for VRF "TENANT_1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:21, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:19:24, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:53:16, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:19:24, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 2d00h, direct
172.17.67.1/32, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 2d00h, local
172.17.67.2/32, ubest/mbest: 1/0, attached
    *via 172.17.67.2, Vlan67, [190/0], 00:11:15, hmm
172.17.67.3/32, ubest/mbest: 1/0, attached
    *via 172.17.67.3, Vlan67, [190/0], 00:11:15, hmm
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.1.1.2%default, [20/0], 00:56:47, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1010102 encap: VXLAN
 
172.17.67.5/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:21, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.6/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:21, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:53:16, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:21, bgp-65401, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF1-2</summary>

```
LEAF1-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.2, local AS number 65402
BGP table version is 146, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
20.1.1.1        4 65400      57809      57785      146    0    0 00:55:19 24        
20.1.2.1        4 65400      57810      57783      146    0    0 00:55:14 24        
LEAF1-2# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
1.1.1.2/32, ubest/mbest: 2/0, attached
    *via 1.1.1.2, Lo1, [0/0], 2d00h, local
    *via 1.1.1.2, Lo1, [0/0], 2d00h, direct
1.1.1.3/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:34:00, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:34:00, bgp-65402, external, tag 65400
1.1.1.4/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:33:24, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:33:24, bgp-65402, external, tag 65400
1.2.1.1/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
10.1.1.100/32, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:56:17, bgp-65402, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:56:11, bgp-65402, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:56:09, bgp-65402, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:56:06, bgp-65402, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0, attached
    *via 20.1.1.2, Eth1/1, [0/0], 2d00h, direct
20.1.1.2/32, ubest/mbest: 1/0, attached
    *via 20.1.1.2, Eth1/1, [0/0], 2d00h, local
20.1.2.0/30, ubest/mbest: 1/0, attached
    *via 20.1.2.2, Eth1/2, [0/0], 2d00h, direct
20.1.2.2/32, ubest/mbest: 1/0, attached
    *via 20.1.2.2, Eth1/2, [0/0], 2d00h, local
20.2.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.1.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.2.1, [20/0], 00:56:10, bgp-65402, external, tag 65400
192.168.2.0/30, ubest/mbest: 2/0
    *via 20.1.1.1, [20/0], 00:55:58, bgp-65402, external, tag 65400
    *via 20.1.2.1, [20/0], 00:55:58, bgp-65402, external, tag 65400

LEAF1-2# show ip route vrf TENANT_1
IP Route Table for VRF "TENANT_1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:25, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:19:27, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:53:20, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:19:27, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 2d00h, direct
172.17.67.1/32, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 2d00h, local
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 00:56:51, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 00:56:51, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0, attached
    *via 172.17.67.4, Vlan67, [190/0], 00:11:20, hmm
172.17.67.5/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:25, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.6/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:25, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:53:20, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:21:25, bgp-65402, external, tag 65400, segid: 100501 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF1-3</summary>

```
LEAF1-3# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.3, local AS number 65403
BGP table version is 34, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
30.1.1.1        4 65400        678        672       34    0    0 00:33:18 24        
30.1.2.1        4 65400        673        668       34    0    0 00:33:04 24        
LEAF1-3# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
1.1.1.2/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
1.1.1.3/32, ubest/mbest: 2/0, attached
    *via 1.1.1.3, Lo1, [0/0], 00:36:03, local
    *via 1.1.1.3, Lo1, [0/0], 00:36:03, direct
1.1.1.4/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:33:28, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:33:28, bgp-65403, external, tag 65400
1.2.1.1/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.1.1.100/32, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0, attached
    *via 30.1.1.2, Eth1/1, [0/0], 00:34:24, direct
30.1.1.2/32, ubest/mbest: 1/0, attached
    *via 30.1.1.2, Eth1/1, [0/0], 00:34:24, local
30.1.2.0/30, ubest/mbest: 1/0, attached
    *via 30.1.2.2, Eth1/2, [0/0], 00:34:24, direct
30.1.2.2/32, ubest/mbest: 1/0, attached
    *via 30.1.2.2, Eth1/2, [0/0], 00:34:24, local
30.2.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
192.168.2.0/30, ubest/mbest: 2/0
    *via 30.1.1.1, [20/0], 00:34:03, bgp-65403, external, tag 65400
    *via 30.1.2.1, [20/0], 00:34:03, bgp-65403, external, tag 65400

LEAF1-3# show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:40, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.0/24, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 00:37:16, direct
172.16.65.1/32, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 00:37:16, local
172.16.65.2/32, ubest/mbest: 1/0, attached
    *via 172.16.65.2, Vlan65, [190/0], 00:35:05, hmm
172.16.65.3/32, ubest/mbest: 1/0
    *via 10.1.1.100%default, [20/0], 00:34:40, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0xa010164 encap: VXLAN
 
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.1.1.4%default, [20/0], 00:34:07, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1010104 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:40, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:19:56, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:19:56, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:19:56, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:40, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:40, bgp-65403, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF1-4</summary>

```
LEAF1-4# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.4, local AS number 65404
BGP table version is 32, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/88]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
40.1.1.1        4 65400        666        659       32    0    0 00:32:41 24        
40.1.2.1        4 65400        666        659       32    0    0 00:32:40 24        
LEAF1-4# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
1.1.1.2/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
1.1.1.3/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
1.1.1.4/32, ubest/mbest: 2/0, attached
    *via 1.1.1.4, Lo1, [0/0], 00:35:53, local
    *via 1.1.1.4, Lo1, [0/0], 00:35:53, direct
1.2.1.1/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
1.2.1.2/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
1.2.1.3/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.1.1.100/32, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.2.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.10.10.1/32, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.10.10.2/32, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.20.10.1/32, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
10.20.10.2/32, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
12.12.12.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
12.12.12.4/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
20.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
20.2.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
30.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
30.2.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
40.1.1.0/30, ubest/mbest: 1/0, attached
    *via 40.1.1.2, Eth1/1, [0/0], 00:34:06, direct
40.1.1.2/32, ubest/mbest: 1/0, attached
    *via 40.1.1.2, Eth1/1, [0/0], 00:34:06, local
40.1.2.0/30, ubest/mbest: 1/0, attached
    *via 40.1.2.2, Eth1/2, [0/0], 00:34:06, direct
40.1.2.2/32, ubest/mbest: 1/0, attached
    *via 40.1.2.2, Eth1/2, [0/0], 00:34:06, local
192.168.2.0/30, ubest/mbest: 2/0
    *via 40.1.1.1, [20/0], 00:33:32, bgp-65404, external, tag 65400
    *via 40.1.2.1, [20/0], 00:33:32, bgp-65404, external, tag 65400

LEAF1-4# show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:11, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.1.1.3%default, [20/0], 00:34:11, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1010103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0
    *via 10.1.1.100%default, [20/0], 00:34:11, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0xa010164 encap: VXLAN
 
172.16.66.0/24, ubest/mbest: 1/0, attached
    *via 172.16.66.1, Vlan66, [0/0], 00:37:05, direct
172.16.66.1/32, ubest/mbest: 1/0, attached
    *via 172.16.66.1, Vlan66, [0/0], 00:37:05, local
172.16.66.2/32, ubest/mbest: 1/0, attached
    *via 172.16.66.2, Vlan66, [190/0], 00:34:47, hmm
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:11, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:20:00, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:20:00, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:20:00, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:11, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:34:11, bgp-65404, external, tag 65400, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>SPINE1-1</summary>

```
SPINE1-1#  show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.10.10.1, local AS number 65400
BGP table version is 90, IPv4 Unicast config peers 5, capable peers 5
26 network entries and 31 paths using 8072 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/60]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.2        4 65401      57796      57798       90    0    0 00:55:50 3         
12.12.12.6      4 65500      57808      57799       90    0    0 00:55:38 13        
20.1.1.2        4 65402      57796      57798       90    0    0 00:55:50 3         
30.1.1.2        4 65403      57672      57676       90    0    0 00:33:46 3         
40.1.1.2        4 65404      57664      57667       90    0    0 00:33:01 3         
SPINE1-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 00:56:32, bgp-65400, external, tag 65401
1.1.1.2/32, ubest/mbest: 1/0
    *via 20.1.1.2, [20/0], 00:56:32, bgp-65400, external, tag 65402
1.1.1.3/32, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 00:34:22, bgp-65400, external, tag 65403
1.1.1.4/32, ubest/mbest: 1/0
    *via 40.1.1.2, [20/0], 00:33:47, bgp-65400, external, tag 65404
1.2.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
1.2.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
1.2.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 2d00h, direct
10.1.1.1/32, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 2d00h, local
10.1.1.100/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 00:56:32, bgp-65400, external, tag 65401
10.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
10.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
10.10.10.1/32, ubest/mbest: 2/0, attached
    *via 10.10.10.1, Lo10, [0/0], 2d00h, local
    *via 10.10.10.1, Lo10, [0/0], 2d00h, direct
10.20.10.2/32, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:28, bgp-65400, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0, attached
    *via 12.12.12.5, Eth1/5, [0/0], 2d00h, direct
12.12.12.5/32, ubest/mbest: 1/0, attached
    *via 12.12.12.5, Eth1/5, [0/0], 2d00h, local
20.1.1.0/30, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 2d00h, direct
20.1.1.1/32, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 2d00h, local
20.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.2, [20/0], 00:56:32, bgp-65400, external, tag 65402
20.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
20.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 2d00h, direct
30.1.1.1/32, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 2d00h, local
30.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 00:34:22, bgp-65400, external, tag 65403
30.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0, attached
    *via 40.1.1.1, Eth1/4, [0/0], 2d00h, direct
40.1.1.1/32, ubest/mbest: 1/0, attached
    *via 40.1.1.1, Eth1/4, [0/0], 2d00h, local
40.1.2.0/30, ubest/mbest: 1/0
    *via 40.1.1.2, [20/0], 00:33:47, bgp-65400, external, tag 65404
192.168.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.6, [20/0], 00:56:20, bgp-65400, external, tag 65500
```

</details>

<details>

<summary>SPINE1-2</summary>

```
SPINE1-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.10.10.2, local AS number 65400
BGP table version is 90, IPv4 Unicast config peers 5, capable peers 5
26 network entries and 31 paths using 8072 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/60]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.2.2        4 65401      57796      57799       90    0    0 00:55:47 3         
12.12.12.2      4 65500      57808      57799       90    0    0 00:55:46 13        
20.1.2.2        4 65402      57796      57797       90    0    0 00:55:47 3         
30.1.2.2        4 65403      57670      57672       90    0    0 00:33:35 3         
40.1.2.2        4 65404      57665      57670       90    0    0 00:33:03 3         
SPINE1-2# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 10.1.2.2, [20/0], 00:56:36, bgp-65400, external, tag 65401
1.1.1.2/32, ubest/mbest: 1/0
    *via 20.1.2.2, [20/0], 00:56:36, bgp-65400, external, tag 65402
1.1.1.3/32, ubest/mbest: 1/0
    *via 30.1.2.2, [20/0], 00:34:26, bgp-65400, external, tag 65403
1.1.1.4/32, ubest/mbest: 1/0
    *via 40.1.2.2, [20/0], 00:33:50, bgp-65400, external, tag 65404
1.2.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
1.2.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
1.2.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.2.2, [20/0], 00:56:36, bgp-65400, external, tag 65401
10.1.1.100/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
10.1.2.0/30, ubest/mbest: 1/0, attached
    *via 10.1.2.1, Eth1/1, [0/0], 2d00h, direct
10.1.2.1/32, ubest/mbest: 1/0, attached
    *via 10.1.2.1, Eth1/1, [0/0], 2d00h, local
10.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
10.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
10.10.10.2/32, ubest/mbest: 2/0, attached
    *via 10.10.10.2, Lo10, [0/0], 2d00h, local
    *via 10.10.10.2, Lo10, [0/0], 2d00h, direct
10.20.10.1/32, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:35, bgp-65400, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0, attached
    *via 12.12.12.1, Eth1/5, [0/0], 2d00h, direct
12.12.12.1/32, ubest/mbest: 1/0, attached
    *via 12.12.12.1, Eth1/5, [0/0], 2d00h, local
20.1.1.0/30, ubest/mbest: 1/0
    *via 20.1.2.2, [20/0], 00:56:36, bgp-65400, external, tag 65402
20.1.2.0/30, ubest/mbest: 1/0, attached
    *via 20.1.2.1, Eth1/2, [0/0], 2d00h, direct
20.1.2.1/32, ubest/mbest: 1/0, attached
    *via 20.1.2.1, Eth1/2, [0/0], 2d00h, local
20.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
20.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0
    *via 30.1.2.2, [20/0], 00:34:26, bgp-65400, external, tag 65403
30.1.2.0/30, ubest/mbest: 1/0, attached
    *via 30.1.2.1, Eth1/3, [0/0], 2d00h, direct
30.1.2.1/32, ubest/mbest: 1/0, attached
    *via 30.1.2.1, Eth1/3, [0/0], 2d00h, local
30.2.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0
    *via 40.1.2.2, [20/0], 00:33:50, bgp-65400, external, tag 65404
40.1.2.0/30, ubest/mbest: 1/0, attached
    *via 40.1.2.1, Eth1/4, [0/0], 2d00h, direct
40.1.2.1/32, ubest/mbest: 1/0, attached
    *via 40.1.2.1, Eth1/4, [0/0], 2d00h, local
192.168.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.2, [20/0], 00:56:24, bgp-65400, external, tag 65500
```

</details>

<details>

<summary>LEAF2-1</summary>

```
LEAF2-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.2.1.1, local AS number 65501
BGP table version is 204, IPv4 Unicast config peers 3, capable peers 3
29 network entries and 77 paths using 13076 bytes of memory
BGP attribute entries [17/6120], BGP AS path entries [16/208]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.1.1        4 65500      57806      57788      204    0    0 00:55:27 23        
10.2.2.1        4 65500      57807      57791      204    0    0 00:55:25 23        
192.168.2.2     4 65502      57812      57782      204    0    0    2d00h 26        
LEAF2-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
1.1.1.2/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
1.1.1.3/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 00:34:11, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 00:34:11, bgp-65501, external, tag 65500
1.1.1.4/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 00:33:35, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 00:33:35, bgp-65501, external, tag 65500
1.2.1.1/32, ubest/mbest: 2/0, attached
    *via 1.2.1.1, Lo1, [0/0], 2d00h, local
    *via 1.2.1.1, Lo1, [0/0], 2d00h, direct
1.2.1.2/32, ubest/mbest: 1/0
    *via 192.168.2.2, [20/0], 2d00h, bgp-65501, external, tag 65502
1.2.1.3/32, ubest/mbest: 2/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
10.1.1.100/32, ubest/mbest: 2/0, attached
    *via 10.1.1.100, Lo1, [0/0], 2d00h, local
    *via 10.1.1.100, Lo1, [0/0], 2d00h, direct
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
10.2.1.0/30, ubest/mbest: 1/0, attached
    *via 10.2.1.2, Eth1/1, [0/0], 2d00h, direct
10.2.1.2/32, ubest/mbest: 1/0, attached
    *via 10.2.1.2, Eth1/1, [0/0], 2d00h, local
10.2.2.0/30, ubest/mbest: 1/0, attached
    *via 10.2.2.2, Eth1/2, [0/0], 2d00h, direct
10.2.2.2/32, ubest/mbest: 1/0, attached
    *via 10.2.2.2, Eth1/2, [0/0], 2d00h, local
10.10.10.1/32, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
10.10.10.2/32, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
10.20.10.1/32, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:20, bgp-65501, external, tag 65500
10.20.10.2/32, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:17, bgp-65501, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
20.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
20.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
20.2.1.0/30, ubest/mbest: 1/0
    *via 192.168.2.2, [20/0], 00:56:33, bgp-65501, external, tag 65502
20.2.2.0/30, ubest/mbest: 1/0
    *via 192.168.2.2, [20/0], 00:56:30, bgp-65501, external, tag 65502
30.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
30.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
30.2.1.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
40.1.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.1, [20/0], 00:56:09, bgp-65501, external, tag 65500
192.168.2.0/30, ubest/mbest: 1/0, attached
    *via 192.168.2.1, Vlan2, [0/0], 2d00h, direct
192.168.2.1/32, ubest/mbest: 1/0, attached
    *via 192.168.2.1, Vlan2, [0/0], 2d00h, local

LEAF2-1# show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:02, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.0/24, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 2d00h, direct
172.16.65.1/32, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 2d00h, local
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.1.1.3%default, [20/0], 00:22:10, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1010103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0, attached
    *via 172.16.65.3, Vlan65, [190/0], 00:10:54, hmm
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.1.1.4%default, [20/0], 00:22:10, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1010104 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:02, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:02, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:02, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:02, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:57:18, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:02, bgp-65501, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF2-2</summary>

```
LEAF2-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.2.1.2, local AS number 65502
BGP table version is 229, IPv4 Unicast config peers 3, capable peers 3
29 network entries and 77 paths using 13076 bytes of memory
BGP attribute entries [17/6120], BGP AS path entries [16/208]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
20.2.1.1        4 65500      57804      57785      229    0    0 00:55:35 23        
20.2.2.1        4 65500      57806      57786      229    0    0 00:55:31 23        
192.168.2.1     4 65501      57821      57779      229    0    0    2d00h 26        
LEAF2-2# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
1.1.1.2/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
1.1.1.3/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 00:34:13, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 00:34:13, bgp-65502, external, tag 65500
1.1.1.4/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 00:33:38, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 00:33:38, bgp-65502, external, tag 65500
1.2.1.1/32, ubest/mbest: 1/0
    *via 192.168.2.1, [20/0], 2d00h, bgp-65502, external, tag 65501
1.2.1.2/32, ubest/mbest: 2/0, attached
    *via 1.2.1.2, Lo2, [0/0], 2d00h, local
    *via 1.2.1.2, Lo2, [0/0], 2d00h, direct
1.2.1.3/32, ubest/mbest: 2/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
10.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
10.1.1.100/32, ubest/mbest: 2/0, attached
    *via 10.1.1.100, Lo2, [0/0], 2d00h, local
    *via 10.1.1.100, Lo2, [0/0], 2d00h, direct
10.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
10.2.1.0/30, ubest/mbest: 1/0
    *via 192.168.2.1, [20/0], 2d00h, bgp-65502, external, tag 65501
10.2.2.0/30, ubest/mbest: 1/0
    *via 192.168.2.1, [20/0], 2d00h, bgp-65502, external, tag 65501
10.10.10.1/32, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
10.10.10.2/32, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
10.20.10.1/32, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:22, bgp-65502, external, tag 65500
10.20.10.2/32, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:20, bgp-65502, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
20.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
20.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
20.2.1.0/30, ubest/mbest: 1/0, attached
    *via 20.2.1.2, Eth1/1, [0/0], 2d00h, direct
20.2.1.2/32, ubest/mbest: 1/0, attached
    *via 20.2.1.2, Eth1/1, [0/0], 2d00h, local
20.2.2.0/30, ubest/mbest: 1/0, attached
    *via 20.2.2.2, Eth1/2, [0/0], 2d00h, direct
20.2.2.2/32, ubest/mbest: 1/0, attached
    *via 20.2.2.2, Eth1/2, [0/0], 2d00h, local
30.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
30.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
30.2.1.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
30.2.2.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
40.1.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
40.1.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.1, [20/0], 00:56:12, bgp-65502, external, tag 65500
192.168.2.0/30, ubest/mbest: 1/0, attached
    *via 192.168.2.2, Vlan2, [0/0], 2d00h, direct
192.168.2.2/32, ubest/mbest: 1/0, attached
    *via 192.168.2.2, Vlan2, [0/0], 2d00h, local

LEAF2-2# show ip route vrf COD
IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:05, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.16.65.0/24, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 2d00h, direct
172.16.65.1/32, ubest/mbest: 1/0, attached
    *via 172.16.65.1, Vlan65, [0/0], 2d00h, local
172.16.65.2/32, ubest/mbest: 1/0
    *via 1.1.1.3%default, [20/0], 00:22:14, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1010103 encap: VXLAN
 
172.16.65.3/32, ubest/mbest: 1/0, attached
    *via 172.16.65.3, Vlan65, [190/0], 00:10:58, hmm
172.16.66.2/32, ubest/mbest: 1/0
    *via 1.1.1.4%default, [20/0], 00:22:14, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1010104 encap: VXLAN
 
172.17.67.0/24, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:05, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:05, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:05, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:05, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.0/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:57:26, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
 
192.168.3.4/30, ubest/mbest: 1/0
    *via 1.2.1.3%default, [20/0], 00:54:05, bgp-65502, external, tag 65500, segid: 100500 tunnelid: 0x1020103 encap: VXLAN
```

</details>

<details>

<summary>LEAF2-3</summary>

```
LEAF2-3#  show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.2.1.3, local AS number 65503
BGP table version is 143, IPv4 Unicast config peers 2, capable peers 2
29 network entries and 51 paths using 10580 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [8/92]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
30.2.1.1        4 65500      57817      57792      143    0    0 00:55:38 24        
30.2.2.1        4 65500      57817      57792      143    0    0 00:55:34 24        
LEAF2-3# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
1.1.1.2/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
1.1.1.3/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:34:18, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:34:18, bgp-65503, external, tag 65500
1.1.1.4/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:33:42, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:33:42, bgp-65503, external, tag 65500
1.2.1.1/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
1.2.1.2/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
1.2.1.3/32, ubest/mbest: 2/0, attached
    *via 1.2.1.3, Lo3, [0/0], 2d00h, local
    *via 1.2.1.3, Lo3, [0/0], 2d00h, direct
10.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
10.1.1.100/32, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
10.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
10.2.1.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
10.2.2.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
10.10.10.1/32, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
10.10.10.2/32, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
10.20.10.1/32, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:27, bgp-65503, external, tag 65500
10.20.10.2/32, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:24, bgp-65503, external, tag 65500
12.12.12.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
12.12.12.4/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
20.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
20.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
20.2.1.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
20.2.2.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
30.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
30.2.1.0/30, ubest/mbest: 1/0, attached
    *via 30.2.1.2, Eth1/1, [0/0], 2d00h, direct
30.2.1.2/32, ubest/mbest: 1/0, attached
    *via 30.2.1.2, Eth1/1, [0/0], 2d00h, local
30.2.2.0/30, ubest/mbest: 1/0, attached
    *via 30.2.2.2, Eth1/2, [0/0], 2d00h, direct
30.2.2.2/32, ubest/mbest: 1/0, attached
    *via 30.2.2.2, Eth1/2, [0/0], 2d00h, local
40.1.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
40.1.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
192.168.2.0/30, ubest/mbest: 2/0
    *via 30.2.1.1, [20/0], 00:56:16, bgp-65503, external, tag 65500
    *via 30.2.2.1, [20/0], 00:56:16, bgp-65503, external, tag 65500

LEAF2-3# show ip route vrf TENANT_1
IP Route Table for VRF "TENANT_1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 00:53:33, bgp-65503, external, tag 65600
172.16.65.2/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 00:34:24, bgp-65503, external, tag 65600
172.16.65.3/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 00:53:33, bgp-65503, external, tag 65600
172.16.66.2/32, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 00:33:51, bgp-65503, external, tag 65600
172.17.67.0/24, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 2d00h, direct
172.17.67.1/32, ubest/mbest: 1/0, attached
    *via 172.17.67.1, Vlan67, [0/0], 2d00h, local
172.17.67.2/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 00:21:41, bgp-65503, external, tag 65500, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.3/32, ubest/mbest: 1/0
    *via 1.1.1.1%default, [20/0], 00:21:41, bgp-65503, external, tag 65500, segid: 100501 tunnelid: 0x1010101 encap: VXLAN
 
172.17.67.4/32, ubest/mbest: 1/0
    *via 1.1.1.2%default, [20/0], 00:21:41, bgp-65503, external, tag 65500, segid: 100501 tunnelid: 0x1010102 encap: VXLAN
 
172.17.67.5/32, ubest/mbest: 1/0, attached
    *via 172.17.67.5, Vlan67, [190/0], 00:11:28, hmm
172.17.67.6/32, ubest/mbest: 1/0, attached
    *via 172.17.67.6, Vlan67, [190/0], 00:11:28, hmm
192.168.3.0/30, ubest/mbest: 1/0
    *via 192.168.3.6, [20/0], 00:53:33, bgp-65503, external, tag 65600
192.168.3.4/30, ubest/mbest: 1/0, attached
    *via 192.168.3.5, Eth1/6, [0/0], 2d00h, direct
192.168.3.5/32, ubest/mbest: 1/0, attached
    *via 192.168.3.5, Eth1/6, [0/0], 2d00h, local
```

</details>

<details>

<summary>SPINE2-1</summary>

```
SPINE2-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.20.10.1, local AS number 65500
BGP table version is 74, IPv4 Unicast config peers 4, capable peers 4
26 network entries and 38 paths using 8744 bytes of memory
BGP attribute entries [11/3960], BGP AS path entries [10/84]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.1.2        4 65501      57805      57797       74    0    0 00:55:53 8         
12.12.12.1      4 65400      57815      57800       74    0    0 00:55:55 14        
20.2.1.2        4 65502      57808      57792       74    0    0 00:55:54 8         
30.2.1.2        4 65503      57799      57802       74    0    0 00:55:54 3         
SPINE2-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
1.1.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
1.1.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:34:29, bgp-65500, external, tag 65400
1.1.1.4/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:33:54, bgp-65500, external, tag 65400
1.2.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65501
1.2.1.2/32, ubest/mbest: 1/0
    *via 20.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65502
1.2.1.3/32, ubest/mbest: 1/0
    *via 30.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65503
10.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
10.1.1.100/32, ubest/mbest: 1/0
    *via 20.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65502
10.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 2d00h, direct
10.2.1.1/32, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 2d00h, local
10.2.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65501
10.10.10.2/32, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
10.20.10.1/32, ubest/mbest: 2/0, attached
    *via 10.20.10.1, Lo10, [0/0], 2d00h, local
    *via 10.20.10.1, Lo10, [0/0], 2d00h, direct
12.12.12.0/30, ubest/mbest: 1/0, attached
    *via 12.12.12.2, Eth1/4, [0/0], 2d00h, direct
12.12.12.2/32, ubest/mbest: 1/0, attached
    *via 12.12.12.2, Eth1/4, [0/0], 2d00h, local
20.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0, attached
    *via 20.2.1.1, Eth1/2, [0/0], 2d00h, direct
20.2.1.1/32, ubest/mbest: 1/0, attached
    *via 20.2.1.1, Eth1/2, [0/0], 2d00h, local
20.2.2.0/30, ubest/mbest: 1/0
    *via 20.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65502
30.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:34:29, bgp-65500, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0, attached
    *via 30.2.1.1, Eth1/3, [0/0], 2d00h, direct
30.2.1.1/32, ubest/mbest: 1/0, attached
    *via 30.2.1.1, Eth1/3, [0/0], 2d00h, local
30.2.2.0/30, ubest/mbest: 1/0
    *via 30.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65503
40.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:33:54, bgp-65500, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.1, [20/0], 00:56:27, bgp-65500, external, tag 65400
192.168.2.0/30, ubest/mbest: 1/0
    *via 10.2.1.2, [20/0], 00:56:27, bgp-65500, external, tag 65501
```

</details>

<details>

<summary>SPINE2-2</summary>

```
SPINE2-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.20.10.2, local AS number 65500
BGP table version is 90, IPv4 Unicast config peers 4, capable peers 4
26 network entries and 38 paths using 8744 bytes of memory
BGP attribute entries [11/3960], BGP AS path entries [10/84]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.2.2        4 65501      57816      57799       90    0    0 00:55:53 8         
12.12.12.5      4 65400      57816      57802       90    0    0 00:55:54 14        
20.2.2.2        4 65502      57810      57795       90    0    0 00:55:53 8         
30.2.2.2        4 65503      57800      57803       90    0    0 00:55:54 3         
SPINE2-2# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
1.1.1.2/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
1.1.1.3/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:34:33, bgp-65500, external, tag 65400
1.1.1.4/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:33:57, bgp-65500, external, tag 65400
1.2.1.1/32, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65501
1.2.1.2/32, ubest/mbest: 1/0
    *via 20.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65502
1.2.1.3/32, ubest/mbest: 1/0
    *via 30.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65503
10.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
10.1.1.100/32, ubest/mbest: 1/0
    *via 20.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65502
10.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
10.2.1.0/30, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65501
10.2.2.0/30, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/1, [0/0], 2d00h, direct
10.2.2.1/32, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/1, [0/0], 2d00h, local
10.10.10.1/32, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
10.20.10.2/32, ubest/mbest: 2/0, attached
    *via 10.20.10.2, Lo20, [0/0], 2d00h, local
    *via 10.20.10.2, Lo20, [0/0], 2d00h, direct
12.12.12.4/30, ubest/mbest: 1/0, attached
    *via 12.12.12.6, Eth1/4, [0/0], 2d00h, direct
12.12.12.6/32, ubest/mbest: 1/0, attached
    *via 12.12.12.6, Eth1/4, [0/0], 2d00h, local
20.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
20.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
20.2.1.0/30, ubest/mbest: 1/0
    *via 20.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65502
20.2.2.0/30, ubest/mbest: 1/0, attached
    *via 20.2.2.1, Eth1/2, [0/0], 2d00h, direct
20.2.2.1/32, ubest/mbest: 1/0, attached
    *via 20.2.2.1, Eth1/2, [0/0], 2d00h, local
30.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
30.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:34:33, bgp-65500, external, tag 65400
30.2.1.0/30, ubest/mbest: 1/0
    *via 30.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65503
30.2.2.0/30, ubest/mbest: 1/0, attached
    *via 30.2.2.1, Eth1/3, [0/0], 2d00h, direct
30.2.2.1/32, ubest/mbest: 1/0, attached
    *via 30.2.2.1, Eth1/3, [0/0], 2d00h, local
40.1.1.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:56:31, bgp-65500, external, tag 65400
40.1.2.0/30, ubest/mbest: 1/0
    *via 12.12.12.5, [20/0], 00:33:57, bgp-65500, external, tag 65400
192.168.2.0/30, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 00:56:31, bgp-65500, external, tag 65501
```

</details>

<details>

<summary>EDGE</summary>

```
EDGE# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 4.4.4.4, local AS number 65600
BGP table version is 46, IPv4 Unicast config peers 2, capable peers 2
10 network entries and 12 paths using 3112 bytes of memory
BGP attribute entries [7/2520], BGP AS path entries [6/92]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.3.1     4 65503       2924       2903       46    0    0 00:54:43 4         
192.168.3.5     4 65503       2913       2906       46    0    0 00:54:42 5         
EDGE# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

4.4.4.4/32, ubest/mbest: 2/0, attached
    *via 4.4.4.4, Lo1, [0/0], 2d00h, local
    *via 4.4.4.4, Lo1, [0/0], 2d00h, direct
172.16.65.2/32, ubest/mbest: 1/0
    *via 192.168.3.1, [20/0], 00:35:33, bgp-65600, external, tag 65503
172.16.65.3/32, ubest/mbest: 1/0
    *via 192.168.3.1, [20/0], 00:54:41, bgp-65600, external, tag 65503
172.16.66.2/32, ubest/mbest: 1/0
    *via 192.168.3.1, [20/0], 00:35:00, bgp-65600, external, tag 65503
172.17.67.0/24, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 00:54:41, bgp-65600, external, tag 65503
172.17.67.2/32, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 00:54:41, bgp-65600, external, tag 65503
172.17.67.3/32, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 00:54:41, bgp-65600, external, tag 65503
172.17.67.4/32, ubest/mbest: 1/0
    *via 192.168.3.5, [20/0], 00:54:41, bgp-65600, external, tag 65503
192.168.3.0/30, ubest/mbest: 1/0, attached
    *via 192.168.3.2, Eth1/1, [0/0], 2d00h, direct
192.168.3.2/32, ubest/mbest: 1/0, attached
    *via 192.168.3.2, Eth1/1, [0/0], 2d00h, local
192.168.3.4/30, ubest/mbest: 1/0, attached
    *via 192.168.3.6, Eth1/2, [0/0], 2d00h, direct
192.168.3.6/32, ubest/mbest: 1/0, attached
    *via 192.168.3.6, Eth1/2, [0/0], 2d00h, local
```

</details>

l2vpn evpn маршруты

<details>

<summary>LEAF1-1</summary>

```
LEAF1-1# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.1.1.1, local AS number 65401
BGP table version is 1772, L2VPN EVPN config peers 2, capable peers 2
38 network entries and 53 paths using 10808 bytes of memory
BGP attribute entries [25/9000], BGP AS path entries [6/140]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.1      4 65400      87230      86624     1772    0    0    1d00h 15        
10.10.10.2      4 65400      87112      86619     1772    0    0    1d00h 15        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.1      I 65400 15         6          2          0          7         
10.10.10.2      I 65400 15         6          2          0          7         
LEAF1-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 1772, Local Router ID is 1.1.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65503:2
* e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 ?
* e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65403 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65403 i
* e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65502 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65502 i
* e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65404 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65404 i

Route Distinguisher: 1.1.1.1:32834    (L2VNI 100067)
*>l[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65400 65402 i
*>l[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>l[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65400 65402 i
*>l[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>l[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                           100      32768 i
*>e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65400 65402 i
*>e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65400 65500 65503 i

Route Distinguisher: 1.1.1.2:32834
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65400 65402 i
* e                   1.1.1.2                                        0 65400 65402 i
* e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65400 65402 i
*>e                   1.1.1.2                                        0 65400 65402 i
* e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65400 65402 i
*>e                   1.1.1.2                                        0 65400 65402 i

Route Distinguisher: 1.2.1.3:32834
* e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i

Route Distinguisher: 65401:2    (L3VNI 100501)
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65400 65402 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65403 i
*>e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65502 i
*>e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65404 i
```

</details>

<details>

<summary>LEAF1-2</summary>

```
LEAF1-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.1.1.2, local AS number 65402
BGP table version is 2070, L2VPN EVPN config peers 2, capable peers 2
41 network entries and 58 paths using 11588 bytes of memory
BGP attribute entries [25/9000], BGP AS path entries [6/140]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.1      4 65400      87250      86624     2070    0    0    1d00h 17        
10.10.10.2      4 65400      87149      86622     2070    0    0    1d00h 17        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.1      I 65400 17         8          2          0          7         
10.10.10.2      I 65400 17         8          2          0          7         
LEAF1-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 2070, Local Router ID is 1.1.1.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65503:2
* e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 ?
* e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65403 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65403 i
* e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65502 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65502 i
* e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65404 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65404 i

Route Distinguisher: 1.1.1.1:32834
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
* e                   1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
* e                   1.1.1.1                                        0 65400 65401 i
* e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65400 65401 i
*>e                   1.1.1.1                                        0 65400 65401 i
* e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65400 65401 i
*>e                   1.1.1.1                                        0 65400 65401 i
* e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65400 65401 i
*>e                   1.1.1.1                                        0 65400 65401 i

Route Distinguisher: 1.1.1.2:32834    (L2VNI 100067)
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>l[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>l[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65400 65401 i
*>l[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                           100      32768 i
*>e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65400 65500 65503 i

Route Distinguisher: 1.2.1.3:32834
* e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i
* e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65400 65500 65503 i
*>e                   1.2.1.3                                        0 65400 65500 65503 i

Route Distinguisher: 65402:2    (L3VNI 100501)
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65400 65500 65503 i
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65403 i
*>e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65502 i
*>e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65404 i
```

</details>

<details>

<summary>LEAF1-3</summary>

```
LEAF1-3# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.1.1.3, local AS number 65403
BGP table version is 1163, L2VPN EVPN config peers 2, capable peers 2
35 network entries and 56 paths using 10412 bytes of memory
BGP attribute entries [33/11880], BGP AS path entries [8/160]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.1      4 65400      29760      29499     1163    0    0    1d00h 17        
10.10.10.2      4 65400      29645      29499     1163    0    0    1d00h 17        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.1      I 65400 17         8          2          0          7         
10.10.10.2      I 65400 17         8          2          0          7         
LEAF1-3# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 1163, Local Router ID is 1.1.1.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
* e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65400 65404 i
*>e                   1.1.1.4                                        0 65400 65404 i

Route Distinguisher: 65501:1
* e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65501 i

Route Distinguisher: 65502:1
* e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65502 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i

Route Distinguisher: 65503:1
* e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 ?
* e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 ?
* e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65402 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 1.1.1.3:32832    (L2VNI 100065)
*>l[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                           100      32768 i
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i
*>l[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                           100      32768 i
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i
*>l[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                           100      32768 i
* e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i

Route Distinguisher: 1.1.1.4:32833
* e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65400 65404 i
*>e                   1.1.1.4                                        0 65400 65404 i

Route Distinguisher: 1.2.1.1:32832
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65501 i
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65501 i
* e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65501 i

Route Distinguisher: 1.2.1.2:32832
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65502 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65400 65500 65502 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i
* e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65400 65500 65502 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i

Route Distinguisher: 65403:1    (L3VNI 100500)
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65502 i
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65501 i
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65400 65404 i
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65402 i
```

</details>

<details>

<summary>LEAF1-4</summary>

```
LEAF1-4#  show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.1.1.4, local AS number 65404
BGP table version is 923, L2VPN EVPN config peers 2, capable peers 2
27 network entries and 40 paths using 7980 bytes of memory
BGP attribute entries [24/8640], BGP AS path entries [8/160]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.1      4 65400      29736      29488      923    0    0    1d00h 12        
10.10.10.2      4 65400      29626      29481      923    0    0    1d00h 12        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.1      I 65400 12         5          0          0          7         
10.10.10.2      I 65400 12         5          0          0          7         
LEAF1-4#  show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 923, Local Router ID is 1.1.1.4
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65501:1
* e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65501 i

Route Distinguisher: 65502:1
* e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65502 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i

Route Distinguisher: 65503:1
* e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 ?
* e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 ?
* e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65402 i
*>e                   1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 1.1.1.3:32832
* e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65400 65403 i
*>e                   1.1.1.3                                        0 65400 65403 i

Route Distinguisher: 1.1.1.4:32833    (L2VNI 100066)
*>l[2]:[0]:[0]:[48]:[0242.226f.2600]:[0]:[0.0.0.0]/216
                      1.1.1.4                           100      32768 i
*>l[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                           100      32768 i
*>l[3]:[0]:[32]:[1.1.1.4]/88
                      1.1.1.4                           100      32768 i

Route Distinguisher: 1.2.1.1:32832
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65400 65500 65501 i
*>e                   10.1.1.100                                     0 65400 65500 65501 i

Route Distinguisher: 1.2.1.2:32832
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65400 65500 65502 i
*>e                   10.1.1.100                                     0 65400 65500 65502 i

Route Distinguisher: 65403:1    (L3VNI 100500)
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65502 i
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65400 65500 65501 i
*>l[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65400 65403 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65400 65500 65501 i
* e                   10.1.1.100                                     0 65400 65500 65502 i
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65400 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65400 65500 65503 65600 65600 65503 65400 65402 i
```

</details>

<details>

<summary>SPINE1-1</summary>

```
SPINE1-1#  show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.10.10.1, local AS number 65400
BGP table version is 1462, L2VPN EVPN config peers 5, capable peers 5
42 network entries and 42 paths using 12264 bytes of memory
BGP attribute entries [34/12240], BGP AS path entries [14/232]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4 65401      86724      86710     1462    0    0    1d00h 5         
1.1.1.2         4 65402      86711      86717     1462    0    0    1d00h 3         
1.1.1.3         4 65403      86568      86612     1462    0    0    1d00h 3         
1.1.1.4         4 65404      86589      86612     1462    0    0    1d00h 4         
10.20.10.2      4 65500      86982      86628     1462    0    0    1d00h 27        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
1.1.1.1         I 65401 5          4          1          0          0         
1.1.1.2         I 65402 3          2          1          0          0         
1.1.1.3         I 65403 3          2          1          0          0         
1.1.1.4         I 65404 4          3          1          0          0         
10.20.10.2      I 65500 27         10         3          0          14        
SPINE1-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 1462, Local Router ID is 10.10.10.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65404 i

Route Distinguisher: 65501:1
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65501 i

Route Distinguisher: 65502:1
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65502 i

Route Distinguisher: 65503:1
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 65503:2
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65403 i
*>e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65502 i
*>e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65404 i

Route Distinguisher: 1.1.1.1:32834
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65401 i
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65401 i
*>e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65401 i

Route Distinguisher: 1.1.1.2:32834
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65402 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65402 i
*>e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65402 i

Route Distinguisher: 1.1.1.3:32832
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65403 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65403 i
*>e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65403 i

Route Distinguisher: 1.1.1.4:32833
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65404 i
*>e[3]:[0]:[32]:[1.1.1.4]/88
                      1.1.1.4                                        0 65404 i

Route Distinguisher: 1.2.1.1:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65501 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65500 65501 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65500 65501 i

Route Distinguisher: 1.2.1.2:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65502 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65500 65502 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65500 65502 i

Route Distinguisher: 1.2.1.3:32834
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65500 65503 i
*>e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65500 65503 i
```

</details>

<details>

<summary>SPINE1-2</summary>

```
SPINE1-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.10.10.2, local AS number 65400
BGP table version is 1315, L2VPN EVPN config peers 5, capable peers 5
42 network entries and 42 paths using 12264 bytes of memory
BGP attribute entries [34/12240], BGP AS path entries [14/232]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4 65401      86721      86673     1315    0    0    1d00h 5         
1.1.1.2         4 65402      86711      86697     1315    0    0    1d00h 3         
1.1.1.3         4 65403      86573      86584     1315    0    0    1d00h 3         
1.1.1.4         4 65404      86577      86578     1315    0    0    1d00h 4         
10.20.10.1      4 65500      57803      58702     1315    0    0 00:07:39 27        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
1.1.1.1         I 65401 5          4          1          0          0         
1.1.1.2         I 65402 3          2          1          0          0         
1.1.1.3         I 65403 3          2          1          0          0         
1.1.1.4         I 65404 4          3          1          0          0         
10.20.10.1      I 65500 27         10         3          0          14        
SPINE1-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 1315, Local Router ID is 10.10.10.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65404 i

Route Distinguisher: 65501:1
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65501 i

Route Distinguisher: 65502:1
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65502 i

Route Distinguisher: 65503:1
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 65503:2
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65403 i
*>e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65502 i
*>e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65404 i

Route Distinguisher: 1.1.1.1:32834
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65401 i
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65401 i
*>e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65401 i

Route Distinguisher: 1.1.1.2:32834
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65402 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65402 i
*>e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65402 i

Route Distinguisher: 1.1.1.3:32832
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65403 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65403 i
*>e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65403 i

Route Distinguisher: 1.1.1.4:32833
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65404 i
*>e[3]:[0]:[32]:[1.1.1.4]/88
                      1.1.1.4                                        0 65404 i

Route Distinguisher: 1.2.1.1:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65501 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65500 65501 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65500 65501 i

Route Distinguisher: 1.2.1.2:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65502 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65500 65502 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65500 65502 i

Route Distinguisher: 1.2.1.3:32834
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65500 65503 i
*>e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65500 65503 i
```

</details>

<details>

<summary>LEAF2-1</summary>

```
LEAF2-1# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.2.1.1, local AS number 65501
BGP table version is 891, L2VPN EVPN config peers 2, capable peers 2
29 network entries and 41 paths using 8372 bytes of memory
BGP attribute entries [24/8640], BGP AS path entries [7/130]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.20.10.1      4 65500      87059      86561      891    0    0 00:18:02 12        
10.20.10.2      4 65500      87184      86559      891    0    0 00:18:01 12        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.20.10.1      I 65500 12         4          1          0          7         
10.20.10.2      I 65500 12         4          1          0          7         
LEAF2-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 891, Local Router ID is 1.2.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
* e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65500 65400 65404 i
*>e                   1.1.1.4                                        0 65500 65400 65404 i

Route Distinguisher: 65503:1
* e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 ?
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 ?
* e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e                   1.2.1.3                                        0 65500 65503 ?
* e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 1.1.1.3:32832
* e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65500 65400 65403 i
*>e                   1.1.1.3                                        0 65500 65400 65403 i
* e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>e                   1.1.1.3                                        0 65500 65400 65403 i
* e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65500 65400 65403 i
*>e                   1.1.1.3                                        0 65500 65400 65403 i

Route Distinguisher: 1.1.1.4:32833
* e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65500 65400 65404 i
*>e                   1.1.1.4                                        0 65500 65400 65404 i

Route Distinguisher: 1.2.1.1:32832    (L2VNI 100065)
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65500 65400 65403 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                        100      32768 i
*>e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65500 65400 65403 i
*>l[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                        100      32768 i

Route Distinguisher: 65501:1    (L3VNI 100500)
*>l[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65500 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65500 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i
```

</details>

<details>

<summary>LEAF2-2</summary>

```
LEAF2-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.2.1.2, local AS number 65502
BGP table version is 805, L2VPN EVPN config peers 2, capable peers 2
29 network entries and 41 paths using 8372 bytes of memory
BGP attribute entries [24/8640], BGP AS path entries [7/130]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.20.10.1      4 65500      87080      86575      805    0    0 00:18:07 12        
10.20.10.2      4 65500      87190      86572      805    0    0 00:18:06 12        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.20.10.1      I 65500 12         4          1          0          7         
10.20.10.2      I 65500 12         4          1          0          7         
LEAF2-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 805, Local Router ID is 1.2.1.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
* e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65500 65400 65404 i
*>e                   1.1.1.4                                        0 65500 65400 65404 i

Route Distinguisher: 65503:1
* e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 ?
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 ?
* e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e                   1.2.1.3                                        0 65500 65503 ?
* e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e                   1.2.1.3                                        0 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
* e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i
*>e                   1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 1.1.1.3:32832
* e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65500 65400 65403 i
*>e                   1.1.1.3                                        0 65500 65400 65403 i
* e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>e                   1.1.1.3                                        0 65500 65400 65403 i
* e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65500 65400 65403 i
*>e                   1.1.1.3                                        0 65500 65400 65403 i

Route Distinguisher: 1.1.1.4:32833
* e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65500 65400 65404 i
*>e                   1.1.1.4                                        0 65500 65400 65404 i

Route Distinguisher: 1.2.1.2:32832    (L2VNI 100065)
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65500 65400 65403 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                        100      32768 i
*>e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65500 65400 65403 i
*>l[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                        100      32768 i

Route Distinguisher: 65502:1    (L3VNI 100500)
*>l[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65500 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65500 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65500 65503 65600 65600 65503 65400 65402 i
```

</details>

<details>

<summary>LEAF2-3</summary>

```
LEAF2-3# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.2.1.3, local AS number 65503
BGP table version is 1868, L2VPN EVPN config peers 2, capable peers 2
51 network entries and 67 paths using 14700 bytes of memory
BGP attribute entries [39/14040], BGP AS path entries [13/198]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.20.10.1      4 65500      87131      86623     1868    0    0    1d00h 15        
10.20.10.2      4 65500      87213      86623     1868    0    0    1d00h 15        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.20.10.1      I 65500 15         13         2          0          0         
10.20.10.2      I 65500 15         13         2          0          0         
LEAF2-3# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 1868, Local Router ID is 1.2.1.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
* e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65500 65400 65404 i
*>e                   1.1.1.4                                        0 65500 65400 65404 i

Route Distinguisher: 65501:1
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65501 i
* e                   10.1.1.100                                     0 65500 65501 i

Route Distinguisher: 65502:1
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65502 i
* e                   10.1.1.100                                     0 65500 65502 i

Route Distinguisher: 1.1.1.1:32834
* e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65500 65400 65401 i
*>e                   1.1.1.1                                        0 65500 65400 65401 i
* e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65500 65400 65401 i
*>e                   1.1.1.1                                        0 65500 65400 65401 i
* e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65500 65400 65401 i
*>e                   1.1.1.1                                        0 65500 65400 65401 i
* e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65500 65400 65401 i
*>e                   1.1.1.1                                        0 65500 65400 65401 i
* e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65500 65400 65401 i
*>e                   1.1.1.1                                        0 65500 65400 65401 i

Route Distinguisher: 1.1.1.2:32834
* e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65500 65400 65402 i
*>e                   1.1.1.2                                        0 65500 65400 65402 i
* e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65500 65400 65402 i
*>e                   1.1.1.2                                        0 65500 65400 65402 i
* e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65500 65400 65402 i
*>e                   1.1.1.2                                        0 65500 65400 65402 i

Route Distinguisher: 1.1.1.3:32832
* e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>e                   1.1.1.3                                        0 65500 65400 65403 i

Route Distinguisher: 1.1.1.4:32833
* e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65500 65400 65404 i
*>e                   1.1.1.4                                        0 65500 65400 65404 i

Route Distinguisher: 1.2.1.1:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65500 65501 i
* e                   10.1.1.100                                     0 65500 65501 i

Route Distinguisher: 1.2.1.2:32832
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65500 65502 i
*>e                   10.1.1.100                                     0 65500 65502 i

Route Distinguisher: 1.2.1.3:32834    (L2VNI 100067)
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65500 65400 65401 i
*>l[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65500 65400 65402 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65500 65400 65401 i
*>l[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65500 65400 65401 i
*>l[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                           100      32768 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65500 65400 65402 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65500 65400 65401 i
*>l[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                           100      32768 i
*>e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65500 65400 65401 i
*>e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65500 65400 65402 i
*>l[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                           100      32768 i

Route Distinguisher: 65503:1    (L3VNI 100500)
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65502 i
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65500 65501 i
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65500 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65500 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65500 65400 65403 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65500 65502 i
* e                   10.1.1.100                                     0 65500 65501 i
*>l[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65600 65600 ?
*>l[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                  0        100      32768 ?
*>l[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                  0                     0 65600 ?
*>l[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                  0                     0 65600 ?
*>l[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65600 65600 65500 65400 65401 i
*>l[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65600 65600 65500 65400 65401 i
*>l[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65600 65600 65500 65400 65402 i

Route Distinguisher: 65503:2    (L3VNI 100501)
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65500 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65500 65400 65402 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65500 65400 65401 i
*>l[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                  0        100      32768 ?
*>l[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                  0                     0 65600 ?
*>l[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                  0        100      32768 ?
*>l[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                  0                     0 65600 ?
*>l[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65600 65600 65500 65400 65403 i
*>l[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65600 65600 65500 65502 i
*>l[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65600 65600 65500 65400 65404 i
```

</details>

<details>

<summary>SPINE2-1</summary>

```
SPINE2-1# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.20.10.1, local AS number 65500
BGP table version is 1302, L2VPN EVPN config peers 4, capable peers 4
42 network entries and 42 paths using 12264 bytes of memory
BGP attribute entries [34/12240], BGP AS path entries [14/208]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.2.1.1         4 65501      86659      86591     1302    0    0 00:18:28 4         
1.2.1.2         4 65502      86669      86608     1302    0    0 00:18:27 4         
1.2.1.3         4 65503      86748      86702     1302    0    0    1d00h 19        
10.10.10.2      4 65400        175        159     1302    0    0 00:07:44 15        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
1.2.1.1         I 65501 4          3          1          0          0         
1.2.1.2         I 65502 4          3          1          0          0         
1.2.1.3         I 65503 19         4          1          0          14        
10.10.10.2      I 65400 15         11         4          0          0         
SPINE2-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 1302, Local Router ID is 10.20.10.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65400 65404 i

Route Distinguisher: 65501:1
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65501 i

Route Distinguisher: 65502:1
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65502 i

Route Distinguisher: 65503:1
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                  0                     0 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 65503:2
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                  0                     0 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                  0                     0 65503 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65403 i
*>e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65502 i
*>e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65404 i

Route Distinguisher: 1.1.1.1:32834
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65400 65401 i

Route Distinguisher: 1.1.1.2:32834
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65400 65402 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65400 65402 i
*>e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65400 65402 i

Route Distinguisher: 1.1.1.3:32832
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65400 65403 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65400 65403 i
*>e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65400 65403 i

Route Distinguisher: 1.1.1.4:32833
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65400 65404 i
*>e[3]:[0]:[32]:[1.1.1.4]/88
                      1.1.1.4                                        0 65400 65404 i

Route Distinguisher: 1.2.1.1:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65501 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65501 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65501 i

Route Distinguisher: 1.2.1.2:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65502 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65502 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65502 i

Route Distinguisher: 1.2.1.3:32834
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65503 i
*>e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65503 i
```

</details>

<details>

<summary>SPINE2-2</summary>

```
SPINE2-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.20.10.2, local AS number 65500
BGP table version is 1533, L2VPN EVPN config peers 4, capable peers 4
42 network entries and 42 paths using 12264 bytes of memory
BGP attribute entries [34/12240], BGP AS path entries [14/208]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.2.1.1         4 65501      86659      86608     1533    0    0 00:18:32 4         
1.2.1.2         4 65502      86668      86625     1533    0    0 00:18:32 4         
1.2.1.3         4 65503      86751      86707     1533    0    0    1d00h 19        
10.10.10.1      4 65400      86967      86669     1533    0    0    1d00h 15        

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
1.2.1.1         I 65501 4          3          1          0          0         
1.2.1.2         I 65502 4          3          1          0          0         
1.2.1.3         I 65503 19         4          1          0          14        
10.10.10.1      I 65400 15         11         4          0          0         
SPINE2-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 1533, Local Router ID is 10.20.10.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65403:1
*>e[2]:[0]:[0]:[48]:[0cec.0000.1b08]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65400 65404 i

Route Distinguisher: 65501:1
*>e[2]:[0]:[0]:[48]:[0ce9.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65501 i

Route Distinguisher: 65502:1
*>e[2]:[0]:[0]:[48]:[0cd2.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65502 i

Route Distinguisher: 65503:1
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                                        0 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                  0                     0 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.17.67.2]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.3]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65401 i
*>e[5]:[0]:[0]:[32]:[172.17.67.4]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65402 i

Route Distinguisher: 65503:2
*>e[5]:[0]:[0]:[24]:[172.17.67.0]/224
                      1.2.1.3                  0                     0 65503 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.0]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[30]:[192.168.3.4]/224
                      1.2.1.3                  0                     0 65503 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      1.2.1.3                  0                     0 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[172.16.65.2]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65403 i
*>e[5]:[0]:[0]:[32]:[172.16.65.3]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65502 i
*>e[5]:[0]:[0]:[32]:[172.16.66.2]/224
                      1.2.1.3                                        0 65503 65600 65600 65503 65400 65404 i

Route Distinguisher: 1.1.1.1:32834
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.00c0.8300]:[32]:[172.17.67.3]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[2]:[0]:[0]:[48]:[0242.ed27.8200]:[32]:[172.17.67.2]/272
                      1.1.1.1                                        0 65400 65401 i
*>e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65400 65401 i

Route Distinguisher: 1.1.1.2:32834
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[0]:[0.0.0.0]/216
                      1.1.1.2                                        0 65400 65402 i
*>e[2]:[0]:[0]:[48]:[0242.8632.4200]:[32]:[172.17.67.4]/272
                      1.1.1.2                                        0 65400 65402 i
*>e[3]:[0]:[32]:[1.1.1.2]/88
                      1.1.1.2                                        0 65400 65402 i

Route Distinguisher: 1.1.1.3:32832
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[0]:[0.0.0.0]/216
                      1.1.1.3                                        0 65400 65403 i
*>e[2]:[0]:[0]:[48]:[0242.b33e.be00]:[32]:[172.16.65.2]/272
                      1.1.1.3                                        0 65400 65403 i
*>e[3]:[0]:[32]:[1.1.1.3]/88
                      1.1.1.3                                        0 65400 65403 i

Route Distinguisher: 1.1.1.4:32833
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[0]:[0.0.0.0]/216
                      1.1.1.4                                        0 65400 65404 i
*>e[2]:[0]:[0]:[48]:[0242.226f.2600]:[32]:[172.16.66.2]/272
                      1.1.1.4                                        0 65400 65404 i
*>e[3]:[0]:[32]:[1.1.1.4]/88
                      1.1.1.4                                        0 65400 65404 i

Route Distinguisher: 1.2.1.1:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65501 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65501 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65501 i

Route Distinguisher: 1.2.1.2:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65502 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[172.16.65.3]/272
                      10.1.1.100                                     0 65502 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65502 i

Route Distinguisher: 1.2.1.3:32834
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      1.2.1.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[172.17.67.6]/272
                      1.2.1.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[172.17.67.5]/272
                      1.2.1.3                                        0 65503 i
*>e[3]:[0]:[32]:[1.2.1.3]/88
                      1.2.1.3                                        0 65503 i
```

</details>

VXLAN туннели

<details>

<summary>LEAF1-1</summary>

```
LEAF1-1# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac       
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      1.1.1.2                                 Up    CP        1d01h    0c63.0000.1b08   
nve1      1.2.1.3                                 Up    CP        1d01h    0cde.0000.1b08  

LEAF1-1# show nve interface
Interface: nve1, State: Up, encapsulation: VXLAN
 VPC Capability: VPC-VIP-Only [not-notified]
 Local Router MAC: 0c9a.0000.1b08
 Host Learning Mode: Control-Plane
 Source-Interface: loopback1 (primary: 1.1.1.1, secondary: 0.0.0.0)

LEAF1-1# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 640 pkts, 58792 bytes - mcast: 38 pkts, 1756 bytes
  TX
    ucast: 1013 pkts, 139418 bytes - mcast: 0 pkts, 0 bytes

LEAF1-1# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100067   UnicastBGP        Up    CP   L2 [67]                 
nve1      100501   n/a               Up    CP   L3 [TENANT_1]           

LEAF1-1# show nve vni ingress-replication 
Interface VNI      Replication List  Source  Up Time      
--------- -------- ----------------- ------- -------      

nve1      100067   1.2.1.3           BGP-IMET 1d01h       
                   1.1.1.2           BGP-IMET 1d01h  
```

</details>

<details>

<summary>LEAF1-2</summary>

```
LEAF1-2# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac       
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      1.1.1.1                                 Up    CP        1d01h    0c9a.0000.1b08   
nve1      1.2.1.3                                 Up    CP        1d01h    0cde.0000.1b08 

LEAF1-2# show nve interface
Interface: nve1, State: Up, encapsulation: VXLAN
 VPC Capability: VPC-VIP-Only [not-notified]
 Local Router MAC: 0c63.0000.1b08
 Host Learning Mode: Control-Plane
 Source-Interface: loopback1 (primary: 1.1.1.2, secondary: 0.0.0.0)

LEAF1-2# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 1515 pkts, 143934 bytes - mcast: 46 pkts, 2316 bytes
  TX
    ucast: 394 pkts, 52252 bytes - mcast: 0 pkts, 0 bytes

LEAF1-2# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100067   UnicastBGP        Up    CP   L2 [67]                 
nve1      100501   n/a               Up    CP   L3 [TENANT_1]           

LEAF1-2# show nve vni ingress-replication
Interface VNI      Replication List  Source  Up Time      
--------- -------- ----------------- ------- -------      

nve1      100067   1.2.1.3           BGP-IMET 1d01h       
                   1.1.1.1           BGP-IMET 1d01h   
```

</details>

<details>

<summary>LEAF1-3</summary>

```
LEAF1-3# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac       
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      1.1.1.4                                 Up    CP        1d00h    0cec.0000.1b08   
nve1      1.2.1.3                                 Up    CP        1d00h    0cde.0000.1b08   
nve1      10.1.1.100                              Up    CP        00:22:33 0200.0a01.0164 

LEAF1-3# show nve interface
Interface: nve1, State: Up, encapsulation: VXLAN
 VPC Capability: VPC-VIP-Only [not-notified]
 Local Router MAC: 0c6e.0000.1b08
 Host Learning Mode: Control-Plane
 Source-Interface: loopback1 (primary: 1.1.1.3, secondary: 0.0.0.0)

LEAF1-3# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 372 pkts, 33952 bytes - mcast: 0 pkts, 0 bytes
  TX
    ucast: 253 pkts, 36866 bytes - mcast: 0 pkts, 0 bytes

LEAF1-3# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100065   UnicastBGP        Up    CP   L2 [65]                 
nve1      100500   n/a               Up    CP   L3 [COD]                

LEAF1-3# show nve vni ingress-replication
Interface VNI      Replication List  Source  Up Time      
--------- -------- ----------------- ------- -------      

nve1      100065   10.1.1.100        BGP-IMET 00:29:43 
```

</details>

<details>

<summary>LEAF1-4</summary>

```
LEAF1-4# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac       
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      1.1.1.3                                 Up    CP        1d00h    0c6e.0000.1b08   
nve1      1.2.1.3                                 Up    CP        1d00h    0cde.0000.1b08   
nve1      10.1.1.100                              Up    CP        00:22:46 0200.0a01.0164 

LEAF1-4# show nve interface
Interface: nve1, State: Up, encapsulation: VXLAN
 VPC Capability: VPC-VIP-Only [not-notified]
 Local Router MAC: 0cec.0000.1b08
 Host Learning Mode: Control-Plane
 Source-Interface: loopback1 (primary: 1.1.1.4, secondary: 0.0.0.0)

LEAF1-4# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 135 pkts, 11536 bytes - mcast: 0 pkts, 0 bytes
  TX
    ucast: 14 pkts, 2072 bytes - mcast: 0 pkts, 0 bytes

LEAF1-4# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100066   UnicastBGP        Up    CP   L2 [66]                 
nve1      100500   n/a               Up    CP   L3 [COD]                

LEAF1-4# show nve vni ingress-replication
Interface VNI      Replication List  Source  Up Time      
--------- -------- ----------------- ------- -------      

nve1      100066   
```

</details>

<details>

<summary>LEAF2-1</summary>

```
LEAF2-1# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac       
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      1.1.1.3                                 Up    CP        00:50:07 0c6e.0000.1b08   
nve1      1.1.1.4                                 Up    CP        00:50:07 0cec.0000.1b08   
nve1      1.2.1.3                                 Up    CP        00:50:12 0cde.0000.1b08 

LEAF2-1# show nve interface
Interface: nve1, State: Up, encapsulation: VXLAN
 VPC Capability: VPC-VIP-Only [notified]
 Local Router MAC: 0ce9.0000.1b08
 Host Learning Mode: Control-Plane
 Source-Interface: loopback1 (primary: 1.2.1.1, secondary: 10.1.1.100)

LEAF2-1# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 75 pkts, 6342 bytes - mcast: 6 pkts, 284 bytes
  TX
    ucast: 92 pkts, 12654 bytes - mcast: 0 pkts, 0 bytes

LEAF2-1# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100065   UnicastBGP        Up    CP   L2 [65]                 
nve1      100500   n/a               Up    CP   L3 [COD]                

LEAF2-1# show nve vni ingress-replication
Interface VNI      Replication List  Source  Up Time      
--------- -------- ----------------- ------- -------      

nve1      100065   1.1.1.3           BGP-IMET 00:57:20    
```

</details>

<details>

<summary>LEAF2-2</summary>

```
LEAF2-2# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac       
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      1.1.1.3                                 Up    CP        00:50:33 0c6e.0000.1b08   
nve1      1.1.1.4                                 Up    CP        00:50:33 0cec.0000.1b08   
nve1      1.2.1.3                                 Up    CP        00:50:33 0cde.0000.1b08 

LEAF2-2# show nve interface
Interface: nve1, State: Up, encapsulation: VXLAN
 VPC Capability: VPC-VIP-Only [notified]
 Local Router MAC: 0cd2.0000.1b08
 Host Learning Mode: Control-Plane
 Source-Interface: loopback2 (primary: 1.2.1.2, secondary: 10.1.1.100)

LEAF2-2# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 134 pkts, 12308 bytes - mcast: 6 pkts, 284 bytes
  TX
    ucast: 157 pkts, 21574 bytes - mcast: 0 pkts, 0 bytes

LEAF2-2# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100065   UnicastBGP        Up    CP   L2 [65]                 
nve1      100500   n/a               Up    CP   L3 [COD]                

LEAF2-2# show nve vni ingress-replication
Interface VNI      Replication List  Source  Up Time      
--------- -------- ----------------- ------- -------      

nve1      100065   1.1.1.3           BGP-IMET 00:57:41   
```

</details>

<details>

<summary>LEAF2-3</summary>

```
LEAF2-3# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac       
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      1.1.1.1                                 Up    CP        1d01h    0c9a.0000.1b08   
nve1      1.1.1.2                                 Up    CP        1d01h    0c63.0000.1b08   
nve1      1.1.1.3                                 Up    CP        1d00h    0c6e.0000.1b08   
nve1      1.1.1.4                                 Up    CP        1d00h    0cec.0000.1b08   
nve1      10.1.1.100                              Up    CP        00:22:57 0200.0a01.0164 

LEAF2-3# show nve interface
Interface: nve1, State: Up, encapsulation: VXLAN
 VPC Capability: VPC-VIP-Only [not-notified]
 Local Router MAC: 0cde.0000.1b08
 Host Learning Mode: Control-Plane
 Source-Interface: loopback3 (primary: 1.2.1.3, secondary: 0.0.0.0)

LEAF2-3# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 2020 pkts, 195478 bytes - mcast: 42 pkts, 1988 bytes
  TX
    ucast: 2661 pkts, 381514 bytes - mcast: 0 pkts, 0 bytes

LEAF2-3# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100067   UnicastBGP        Up    CP   L2 [67]                 
nve1      100500   n/a               Up    CP   L3 [COD]                
nve1      100501   n/a               Up    CP   L3 [TENANT_1]           

LEAF2-3# show nve vni ingress-replication
Interface VNI      Replication List  Source  Up Time      
--------- -------- ----------------- ------- -------      

nve1      100067   1.1.1.1           BGP-IMET 1d01h       
                   1.1.1.2           BGP-IMET 1d01h 
```

</details>

Содержимое mac-VRF

<details>

<summary>LEAF1-1</summary>

```
LEAF1-1# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops                              
----------- -------------- ------ ------------------- ---------- ---------------------------------------------------------
67          0242.00c0.8300 Local  L,                 0          Eth1/4                                                   
67          0242.248c.3e00 BGP    SplRcv             0          1.2.1.3 (Label: 100067)                                  
67          0242.8632.4200 BGP    SplRcv             0          1.1.1.2 (Label: 100067)                                  
67          0242.ed27.8200 Local  L,                 0          Eth1/3                                                   
67          0242.ee8b.bc00 BGP    SplRcv             0          1.2.1.3 (Label: 100067)                                  
501         0c63.0000.1b08 VXLAN  Rmac               0          1.1.1.2                                                  
501         0cde.0000.1b08 VXLAN  Rmac               0          1.2.1.3                                                  
LEAF1-1# show l2route evpn mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan (Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Piporp): Directly connected Orphan to PIP based vPC BGW 
(Pipporp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Host IP                                 Prod   Flags              Seq No     Next-Hops                              
----------- -------------- --------------------------------------- ------ ----------------- ---------- ---------------------------------------------------------
67          0242.ed27.8200 172.17.67.2                             HMM    L,                 0         Local                                  
67          0242.00c0.8300 172.17.67.3                             HMM    L,                 0         Local                                  
67          0242.8632.4200 172.17.67.4                             BGP    --                 0         1.1.1.2 (Label: 100067)                                  
67          0242.ee8b.bc00 172.17.67.5                             BGP    --                 0         1.2.1.3 (Label: 100067)                                  
67          0242.248c.3e00 172.17.67.6                             BGP    --                 0         1.2.1.3 (Label: 100067)  
```

</details>

<details>

<summary>LEAF1-2</summary>

```
LEAF1-2# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops                              
----------- -------------- ------ ------------------- ---------- ---------------------------------------------------------
67          0242.00c0.8300 BGP    SplRcv             0          1.1.1.1 (Label: 100067)                                  
67          0242.248c.3e00 BGP    SplRcv             0          1.2.1.3 (Label: 100067)                                  
67          0242.8632.4200 Local  L,                 0          Eth1/3                                                   
67          0242.ed27.8200 BGP    SplRcv             0          1.1.1.1 (Label: 100067)                                  
67          0242.ee8b.bc00 BGP    SplRcv             0          1.2.1.3 (Label: 100067)                                  
501         0c9a.0000.1b08 VXLAN  Rmac               0          1.1.1.1                                                  
501         0cde.0000.1b08 VXLAN  Rmac               0          1.2.1.3                                                  
LEAF1-2# show l2route evpn mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan (Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Piporp): Directly connected Orphan to PIP based vPC BGW 
(Pipporp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Host IP                                 Prod   Flags              Seq No     Next-Hops                              
----------- -------------- --------------------------------------- ------ ----------------- ---------- ---------------------------------------------------------
67          0242.ed27.8200 172.17.67.2                             BGP    --                 0         1.1.1.1 (Label: 100067)                                  
67          0242.00c0.8300 172.17.67.3                             BGP    --                 0         1.1.1.1 (Label: 100067)                                  
67          0242.8632.4200 172.17.67.4                             HMM    L,                 0         Local                                  
67          0242.ee8b.bc00 172.17.67.5                             BGP    --                 0         1.2.1.3 (Label: 100067)                                  
67          0242.248c.3e00 172.17.67.6                             BGP    --                 0         1.2.1.3 (Label: 100067)    
```

</details>

<details>

<summary>LEAF1-3</summary>

```
LEAF1-3# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops                              
----------- -------------- ------ ------------------- ---------- ---------------------------------------------------------
65          0242.b33e.be00 Local  L,                 0          Eth1/3                                                   
65          0c8c.4346.8041 BGP    SplRcv             1          10.1.1.100 (Label: 100065)                               
500         0200.0a01.0164 VXLAN  Rmac               0          10.1.1.100                                               
500         0cde.0000.1b08 VXLAN  Rmac               0          1.2.1.3                                                  
500         0cec.0000.1b08 VXLAN  Rmac               0          1.1.1.4                                                  
LEAF1-3# show l2route evpn mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan (Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Piporp): Directly connected Orphan to PIP based vPC BGW 
(Pipporp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Host IP                                 Prod   Flags              Seq No     Next-Hops                              
----------- -------------- --------------------------------------- ------ ----------------- ---------- ---------------------------------------------------------
65          0242.b33e.be00 172.16.65.2                             HMM    L,                 0         Local                                  
65          0c8c.4346.8041 172.16.65.3                             BGP    --                 1         10.1.1.100 (Label: 100065)
```

</details>

<details>

<summary>LEAF1-4</summary>

```
LEAF1-4# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops                              
----------- -------------- ------ ------------------- ---------- ---------------------------------------------------------
66          0242.226f.2600 Local  L,                 0          Eth1/3                                                   
500         0200.0a01.0164 VXLAN  Rmac               0          10.1.1.100                                               
500         0c6e.0000.1b08 VXLAN  Rmac               0          1.1.1.3                                                  
500         0cde.0000.1b08 VXLAN  Rmac               0          1.2.1.3                                                  
LEAF1-4# show l2route evpn mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan (Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Piporp): Directly connected Orphan to PIP based vPC BGW 
(Pipporp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Host IP                                 Prod   Flags              Seq No     Next-Hops                              
----------- -------------- --------------------------------------- ------ ----------------- ---------- ---------------------------------------------------------
66          0242.226f.2600 172.16.66.2                             HMM    L,                 0         Local 
```

</details>

<details>

<summary>LEAF2-1</summary>

```
LEAF2-1# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops                              
----------- -------------- ------ ------------------- ---------- ---------------------------------------------------------
65          0242.b33e.be00 BGP    SplRcv             0          1.1.1.3 (Label: 100065)                                  
65          0c8c.4346.8041 Local  L,                 1          Po3                                                      
500         0c6e.0000.1b08 VXLAN  Rmac               0          1.1.1.3                                                  
500         0cde.0000.1b08 VXLAN  Rmac               0          1.2.1.3                                                  
500         0cec.0000.1b08 VXLAN  Rmac               0          1.1.1.4                                                  
LEAF2-1# show l2route evpn mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan (Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Piporp): Directly connected Orphan to PIP based vPC BGW 
(Pipporp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Host IP                                 Prod   Flags              Seq No     Next-Hops                              
----------- -------------- --------------------------------------- ------ ----------------- ---------- ---------------------------------------------------------
65          0242.b33e.be00 172.16.65.2                             BGP    --                 0         1.1.1.3 (Label: 100065)                                  
65          0c8c.4346.8041 172.16.65.3                             HMM    L,                 1         Local 
```

</details>

<details>

<summary>LEAF2-2</summary>

```
LEAF2-2# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops                              
----------- -------------- ------ ------------------- ---------- ---------------------------------------------------------
65          0242.b33e.be00 BGP    SplRcv             0          1.1.1.3 (Label: 100065)                                  
65          0c8c.4346.8041 Local  L,                 1          Po3                                                      
500         0c6e.0000.1b08 VXLAN  Rmac               0          1.1.1.3                                                  
500         0cde.0000.1b08 VXLAN  Rmac               0          1.2.1.3                                                  
500         0cec.0000.1b08 VXLAN  Rmac               0          1.1.1.4                                                  
LEAF2-2# show l2route evpn mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan (Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Piporp): Directly connected Orphan to PIP based vPC BGW 
(Pipporp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Host IP                                 Prod   Flags              Seq No     Next-Hops                              
----------- -------------- --------------------------------------- ------ ----------------- ---------- ---------------------------------------------------------
65          0242.b33e.be00 172.16.65.2                             BGP    --                 0         1.1.1.3 (Label: 100065)                                  
65          0c8c.4346.8041 172.16.65.3                             HMM    L,                 1         Local       
```

</details>

<details>

<summary>LEAF2-3</summary>

```
LEAF2-3# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops                              
----------- -------------- ------ ------------------- ---------- ---------------------------------------------------------
67          0242.00c0.8300 BGP    SplRcv             0          1.1.1.1 (Label: 100067)                                  
67          0242.248c.3e00 Local  L,                 0          Eth1/4                                                   
67          0242.8632.4200 BGP    SplRcv             0          1.1.1.2 (Label: 100067)                                  
67          0242.ed27.8200 BGP    SplRcv             0          1.1.1.1 (Label: 100067)                                  
67          0242.ee8b.bc00 Local  L,                 0          Eth1/3                                                   
500         0200.0a01.0164 VXLAN  Rmac               0          10.1.1.100                                               
500         0c6e.0000.1b08 VXLAN  Rmac               0          1.1.1.3                                                  
500         0cec.0000.1b08 VXLAN  Rmac               0          1.1.1.4                                                  
501         0c63.0000.1b08 VXLAN  Rmac               0          1.1.1.2                                                  
501         0c9a.0000.1b08 VXLAN  Rmac               0          1.1.1.1                                                  
LEAF2-3# show l2route evpn mac-ip all
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote 
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan (Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Piporp): Directly connected Orphan to PIP based vPC BGW 
(Pipporp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Host IP                                 Prod   Flags              Seq No     Next-Hops                              
----------- -------------- --------------------------------------- ------ ----------------- ---------- ---------------------------------------------------------
67          0242.ed27.8200 172.17.67.2                             BGP    --                 0         1.1.1.1 (Label: 100067)                                  
67          0242.00c0.8300 172.17.67.3                             BGP    --                 0         1.1.1.1 (Label: 100067)                                  
67          0242.8632.4200 172.17.67.4                             BGP    --                 0         1.1.1.2 (Label: 100067)                                  
67          0242.ee8b.bc00 172.17.67.5                             HMM    L,                 0         Local                                  
67          0242.248c.3e00 172.17.67.6                             HMM    L,                 0         Local 
```

</details>

Проверка связности между клиентами

<details>

<summary>endhost-1</summary>

```
 / # ping 172.17.67.3
PING 172.17.67.3 (172.17.67.3) 56(84) bytes of data.
64 bytes from 172.17.67.3: icmp_seq=1 ttl=64 time=2.27 ms
64 bytes from 172.17.67.3: icmp_seq=2 ttl=64 time=2.32 ms
64 bytes from 172.17.67.3: icmp_seq=3 ttl=64 time=1.83 ms
^C
--- 172.17.67.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 1.832/2.139/2.321/0.218 ms
/ # ping 172.17.67.4
PING 172.17.67.4 (172.17.67.4) 56(84) bytes of data.
64 bytes from 172.17.67.4: icmp_seq=1 ttl=64 time=16.4 ms
64 bytes from 172.17.67.4: icmp_seq=2 ttl=64 time=15.6 ms
64 bytes from 172.17.67.4: icmp_seq=3 ttl=64 time=17.3 ms
^C
--- 172.17.67.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 15.553/16.444/17.349/0.733 ms
/ # ping 172.17.67.5
PING 172.17.67.5 (172.17.67.5) 56(84) bytes of data.
64 bytes from 172.17.67.5: icmp_seq=1 ttl=64 time=20.0 ms
64 bytes from 172.17.67.5: icmp_seq=2 ttl=64 time=12.3 ms
64 bytes from 172.17.67.5: icmp_seq=3 ttl=64 time=14.1 ms
^C
--- 172.17.67.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 12.280/15.456/20.034/3.317 ms
/ # ping 172.17.67.6
PING 172.17.67.6 (172.17.67.6) 56(84) bytes of data.
64 bytes from 172.17.67.6: icmp_seq=1 ttl=64 time=12.4 ms
64 bytes from 172.17.67.6: icmp_seq=2 ttl=64 time=14.4 ms
64 bytes from 172.17.67.6: icmp_seq=3 ttl=64 time=7.89 ms
^C
--- 172.17.67.6 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 7.894/11.549/14.381/2.711 ms
/ # ping 172.16.65.2
PING 172.16.65.2 (172.16.65.2) 56(84) bytes of data.
64 bytes from 172.16.65.2: icmp_seq=1 ttl=59 time=50.5 ms
64 bytes from 172.16.65.2: icmp_seq=2 ttl=59 time=48.1 ms
64 bytes from 172.16.65.2: icmp_seq=3 ttl=59 time=30.3 ms
^C
--- 172.16.65.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 30.253/42.933/50.450/9.017 ms
/ # ping 172.16.65.3
PING 172.16.65.3 (172.16.65.3) 56(84) bytes of data.
64 bytes from 172.16.65.3: icmp_seq=1 ttl=250 time=46.2 ms
64 bytes from 172.16.65.3: icmp_seq=2 ttl=250 time=39.0 ms
64 bytes from 172.16.65.3: icmp_seq=3 ttl=250 time=49.3 ms
^C
--- 172.16.65.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 38.998/44.823/49.280/4.307 ms
/ # ping 172.16.66.2
PING 172.16.66.2 (172.16.66.2) 56(84) bytes of data.
64 bytes from 172.16.66.2: icmp_seq=1 ttl=59 time=44.0 ms
64 bytes from 172.16.66.2: icmp_seq=2 ttl=59 time=26.3 ms
64 bytes from 172.16.66.2: icmp_seq=3 ttl=59 time=38.1 ms
^C
--- 172.16.66.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 26.342/36.146/43.956/7.327 ms
```

</details>

<details>

<summary>endhost-4</summary>

```
/ # ping 172.16.66.2
PING 172.16.66.2 (172.16.66.2) 56(84) bytes of data.
64 bytes from 172.16.66.2: icmp_seq=1 ttl=62 time=18.4 ms
64 bytes from 172.16.66.2: icmp_seq=2 ttl=62 time=7.87 ms
64 bytes from 172.16.66.2: icmp_seq=3 ttl=62 time=9.91 ms
^C
--- 172.16.66.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 7.872/12.051/18.374/4.547 ms
```

</details>

#### Дампы Wireshark

<figure><img src="../.gitbook/assets/Dump Lab_9_1.PNG" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/Dump Lab_9_2.PNG" alt=""><figcaption></figcaption></figure>



#### Выводы

Таким образом, нам удалось настроить&#x20;

ликинг между VRF COD и VRF TENANT\_1 с использованием ipv4 BGP сессий в соответствующих vrf между LEAF3 и EDGE роутером, находящимся в другом логическом сегменте.

Маршруты появились на LEAF3 как в ipv4, так и в l2vpn evpn таблицах.&#x20;

Связность между узлами сети имеется.
