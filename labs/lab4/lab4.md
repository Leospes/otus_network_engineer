---
description: Построение Underlay сети (BGP)
---

# LAB4

#### Цели

* Настроить eBGP для Underlay сети
* Убедиться в наличии IP связанности между устройствами в разных AS

#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_4.PNG" alt=""><figcaption></figcaption></figure>

#### Конфигурация устройств

LEAF1

```
feature bgp

route-map DIRECT-ROUTES-MAP permit 10

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 10.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 10.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_PC1
  no switchport
  ip address 192.168.1.1/24
  no shutdown

interface loopback1
  ip address 1.1.1.1/32
  
router bgp 65501
  router-id 1.1.1.1
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  template peer UNDERLAY
    remote-as 65500
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 10.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
```

LEAF2

```
feature bgp

route-map DIRECT-ROUTES-MAP permit 10

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 20.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 20.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_PC2
  no switchport
  ip address 192.168.2.1/24
  no shutdown
  
interface loopback2
  ip address 2.2.2.2/32

router bgp 65502
  router-id 2.2.2.2
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  template peer UNDERLAY
    remote-as 65500
    timers 3 9
    address-family ipv4 unicast
  neighbor 20.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 20.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
```

LEAF3

```
feature bgp

route-map DIRECT-ROUTES-MAP permit 10

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 30.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 30.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_PC3
  no switchport
  ip address 192.168.3.1/24
  no shutdown

interface Ethernet1/4
  description to_PC4
  no switchport
  ip address 192.168.4.1/24
  no shutdown

interface loopback3
  ip address 3.3.3.3/32
  
router bgp 65503
  router-id 3.3.3.3
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  template peer UNDERLAY
    remote-as 65500
    timers 3 9
    address-family ipv4 unicast
  neighbor 30.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 30.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
```

SPINE1

```
feature bgp

route-map DIRECT-ROUTES-MAP permit 10

interface Ethernet1/1
  description to_LEAF1
  no switchport
  ip address 10.1.1.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  ip address 20.1.1.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  ip address 30.1.1.1/30
  no shutdown

interface loopback10
  ip address 10.10.10.10/32
  
router bgp 65500
  router-id 10.10.10.10
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
  template peer UNDERLAY
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.1.2
    inherit peer UNDERLAY
    remote-as 65501
    address-family ipv4 unicast
  neighbor 20.1.1.2
    inherit peer UNDERLAY
    remote-as 65502
    address-family ipv4 unicast
  neighbor 30.1.1.2
    inherit peer UNDERLAY
    remote-as 65503
    address-family ipv4 unicast
```

SPINE2

```
feature bgp

route-map DIRECT-ROUTES-MAP permit 10

interface Ethernet1/1
  description to_LEAF1
  no switchport
  ip address 10.1.2.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  ip address 20.1.2.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  ip address 30.1.2.1/30
  no shutdown

interface loopback20
  ip address 20.20.20.20/32
  
router bgp 65500
  router-id 20.20.20.20
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
  template peer UNDERLAY
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.2.2
    inherit peer UNDERLAY
    remote-as 65501
    address-family ipv4 unicast
  neighbor 20.1.2.2
    inherit peer UNDERLAY
    remote-as 65502
    address-family ipv4 unicast
  neighbor 30.1.2.2
    inherit peer UNDERLAY
    remote-as 65503
    address-family ipv4 unicast
```
