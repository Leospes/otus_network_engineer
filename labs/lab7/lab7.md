---
description: VXLAN. Multihoming
---

# LAB7

#### Цели

* Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming.
* Протестировать отказоустойчивость - убедиться, что связанность не теряется при отключении одного из линков.

#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_7 (1).PNG" alt=""><figcaption></figcaption></figure>

#### Конфигурация устройств

LEAF1

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
vrf context management
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
  ip address 11.11.11.1/24
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
  source-interface loopback1
  member vni 100065
    ingress-replication protocol bgp
  member vni 100500 associate-vrf

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
  switchport mode trunk
  switchport trunk allowed vlan 65
  channel-group 3 mode active

interface Ethernet1/4

interface Ethernet1/5
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active

interface Ethernet1/6
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active

interface mgmt0
  vrf member management
  ip address 192.168.1.1/24

interface loopback1
  ip address 1.1.1.1/32
  ip address 10.1.1.100/32 secondary

router bgp 65501
  router-id 1.1.1.1
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  address-family l2vpn evpn
    maximum-paths 2
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
  neighbor 10.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 10.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 10.10.10.10
    inherit peer OVERLAY
  neighbor 20.20.20.20
    inherit peer OVERLAY
  neighbor 192.168.2.2
    inherit peer VPC
evpn
  vni 100065 l2
    rd auto
    route-target import auto
    route-target export auto
```

LEAF2

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
vrf context management
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
  ip address 11.11.11.1/24
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
  ip address 20.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 20.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-1
  switchport mode trunk
  switchport trunk allowed vlan 65
  channel-group 3 mode active

interface Ethernet1/4

interface Ethernet1/5
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active

interface Ethernet1/6
  description PEER-LINK
  switchport mode trunk
  switchport trunk allowed vlan 2,65,500
  channel-group 1 mode active

interface mgmt0
  vrf member management
  ip address 192.168.1.2/24

interface loopback2
  ip address 2.2.2.2/32
  ip address 10.1.1.100/32 secondary
  
router bgp 65502
  router-id 2.2.2.2
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
  neighbor 10.10.10.10
    inherit peer OVERLAY
  neighbor 20.1.1.1
    inherit peer UNDERLAY
  neighbor 20.1.2.1
    inherit peer UNDERLAY
  neighbor 20.20.20.20
    inherit peer OVERLAY
  neighbor 192.168.2.1
    inherit peer VPC
evpn
  vni 100065 l2
    rd auto
    route-target import auto
    route-target export auto
```

Конфигурации LEAF3, SPINE1, SPINE2 без изменений.

#### Проверка

#### Выводы

Таким образом, нам удалось настроить L3 маршрутизацию между разными VNI с использованием дополнительного VNI 100500 (vlan 500) и vrf COD. Также была настроен anycast gateway.&#x20;

Маршруты l2vpn evpn уровня L3 успешно изучаются, а VTEPs осуществляют замену маков на уровне Ethernet и таким образом осуществляют маршрутизацию.

Связность между узлами сети имеется.
