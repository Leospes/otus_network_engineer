---
description: VxLAN. Routing
---

# LAB8

#### Цели

* Реализовать передачу суммарных префиксов через EVPN route-type 5.
* Настроить маршрутизацию/ликинг между клиентами/VRF через внешнее устройство - маршрутизатор/firewall и др.
* Протестировать связанность между клиентами в разных VRF.

#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_8.PNG" alt=""><figcaption></figcaption></figure>

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

LEAF3

Заметим, что в BGP под настройками VRF address-family ipv4 unicast применяем команду advertise l2vpn evpn, чтобы маршруты попавшие или находящиеся в VRF анонсировались в BGP EVPN. На новых версиях ПО данной команды не требуется, так как это дефолтное поведение, но для наглядности мы её применили.

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
  ip address 33.33.33.1/24
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
  ip address 30.1.1.2/30
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  mtu 9216
  ip address 30.1.2.2/30
  no shutdown

interface Ethernet1/3
  description to_endhost-2
  switchport access vlan 67
  mtu 9216

interface Ethernet1/4
  description to_endhost-3
  switchport access vlan 67
  mtu 9216

interface Ethernet1/5
  no switchport
  mac-address 0ce8.0000.1b09
  vrf member COD
  ip address 10.2.2.1/30
  no shutdown

interface Ethernet1/6
  no switchport
  mac-address 0ce8.0000.1b10
  vrf member TENANT_1
  ip address 10.3.3.1/30
  no shutdown

interface loopback3
  ip address 3.3.3.3/32
  
router bgp 65503
  router-id 3.3.3.3
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
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  template peer UNDERLAY
    remote-as 65500
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.10.10.10
    inherit peer OVERLAY
  neighbor 20.20.20.20
    inherit peer OVERLAY
  neighbor 30.1.1.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  neighbor 30.1.2.1
    inherit peer UNDERLAY
    address-family ipv4 unicast
  vrf COD
    address-family ipv4 unicast
      advertise l2vpn evpn
      redistribute direct route-map DIRECT-ROUTES-MAP
      maximum-paths 2
    neighbor 10.2.2.2
      remote-as 65600
      address-family ipv4 unicast
  vrf TENANT_1
    address-family ipv4 unicast
      advertise l2vpn evpn
      redistribute direct route-map DIRECT-ROUTES-MAP
      maximum-paths 2
    neighbor 10.3.3.2
      remote-as 65600
      address-family ipv4 unicast
evpn
  vni 100067 l2
    rd auto
    route-target import auto
    route-target export auto
```

EDGE

Стоит обратить внимание, что стандартное поведение Cisco Nexus отличается от, например, классического Cisco IOS или Arista и для того, чтобы Cisco Nexus отправлял BGP анонсы маршрутов обратно пирам, от которых их получил, требуется использование специальной команды disable-peer-as-check.

Также применяем настройку as-override, чтобы EDGE роутер подменял в as-path автономную систему LEAF3 на свою. Если этого не сделать, то LEAF3 не примет обратные анонсы из-за срабатывания BGP loop prevention.

```
#EDGE#
route-map DIRECT-ROUTES-MAP permit 10

interface Ethernet1/1
  no switchport
  ip address 10.2.2.2/30
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.3.3.2/30
  no shutdown
  
interface loopback0
  ip address 4.4.4.4/32
  
router bgp 65600
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
    maximum-paths 2
  neighbor 10.2.2.1
    remote-as 65503
    address-family ipv4 unicast
      as-override
      disable-peer-as-check
  neighbor 10.3.3.1
    remote-as 65503
    address-family ipv4 unicast
      as-override
      disable-peer-as-check
```

Конфигурации SPINE1, SPINE2 и конечных хостов без изменений.

#### Проверка

ipv4 и l2vpn evpn маршруты

LEAF1

<pre><code><strong>LEAF1# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.1.1.1, local AS number 65501
BGP table version is 22, L2VPN EVPN config peers 2, capable peers 2
11 network entries and 15 paths using 3212 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [3/42]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4 65500      24215      24183       22    0    0 20:09:02 4         
20.20.20.20     4 65500      24216      24183       22    0    0 20:09:02 4         

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.10     I 65500 4          0          0          0          4         
20.20.20.20     I 65500 4          0          0          0          4 
        
<strong>LEAF1# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 22, Local Router ID is 1.1.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65503:1
* e[5]:[0]:[0]:[24]:[33.33.33.0]/224
                      3.3.3.3                                        0 65500 65503 65600 65600 ?
*>e                   3.3.3.3                                        0 65500 65503 65600 65600 ?
* e[5]:[0]:[0]:[30]:[10.2.2.0]/224
                      3.3.3.3                                        0 65500 65503 ?
*>e                   3.3.3.3                                        0 65500 65503 ?
* e[5]:[0]:[0]:[30]:[10.3.3.0]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
*>e                   3.3.3.3                                        0 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
*>e                   3.3.3.3                                        0 65500 65503 65600 ?

Route Distinguisher: 1.1.1.1:32832    (L2VNI 100065)
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                        100      32768 i
*>l[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                        100      32768 i

Route Distinguisher: 65501:1    (L3VNI 100500)
*>e[5]:[0]:[0]:[24]:[33.33.33.0]/224
                      3.3.3.3                                        0 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[10.2.2.0]/224
                      3.3.3.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[10.3.3.0]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
					  
<strong>LEAF1# show bgp l2vpn evpn 33.33.33.0
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
Route Distinguisher: 65503:1
BGP routing table entry for [5]:[0]:[0]:[24]:[33.33.33.0]/224, version 18
Paths: (2 available, best #2)
Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
Multipath: eBGP

  Path type: external, path is valid, not best reason: newer EBGP path, no labeled nexthop
  Gateway IP: 0.0.0.0
  AS-Path: 65500 65503 65600 65600 , path sourced external to AS
    3.3.3.3 (metric 0) from 10.10.10.10 (10.10.10.10)
      Origin incomplete, MED not set, localpref 100, weight 0
      Received label 100500
      Extcommunity: RT:11:11 ENCAP:8 Router MAC:0cde.0000.1b08

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported to 2 destination(s)
             Imported paths list: COD L3-100500
  Gateway IP: 0.0.0.0
  AS-Path: 65500 65503 65600 65600 , path sourced external to AS
    3.3.3.3 (metric 0) from 20.20.20.20 (20.20.20.20)
      Origin incomplete, MED not set, localpref 100, weight 0
      Received label 100500
      Extcommunity: RT:11:11 ENCAP:8 Router MAC:0cde.0000.1b08

  Path-id 1 not advertised to any peer

Route Distinguisher: 65501:1    (L3VNI 100500)
BGP routing table entry for [5]:[0]:[0]:[24]:[33.33.33.0]/224, version 14
Paths: (1 available, best #1)
Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
Multipath: eBGP

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported from 65503:1:[5]:[0]:[0]:[24]:[33.33.33.0]/224 
  Gateway IP: 0.0.0.0
  AS-Path: 65500 65503 65600 65600 , path sourced external to AS
    3.3.3.3 (metric 0) from 20.20.20.20 (20.20.20.20)
      Origin incomplete, MED not set, localpref 100, weight 0
      Received label 100500
      Extcommunity: RT:11:11 ENCAP:8 Router MAC:0cde.0000.1b08

  Path-id 1 not advertised to any peer
</code></pre>

LEAF2

<pre><code><strong>LEAF2# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 2.2.2.2, local AS number 65502
BGP table version is 22, L2VPN EVPN config peers 2, capable peers 2
11 network entries and 15 paths using 3212 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [3/42]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4 65500      24228      24195       22    0    0 20:09:40 4         
20.20.20.20     4 65500      24228      24194       22    0    0 20:09:41 4         

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.10     I 65500 4          0          0          0          4         
20.20.20.20     I 65500 4          0          0          0          4 
        
<strong>LEAF2# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 22, Local Router ID is 2.2.2.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65503:1
* e[5]:[0]:[0]:[24]:[33.33.33.0]/224
                      3.3.3.3                                        0 65500 65503 65600 65600 ?
*>e                   3.3.3.3                                        0 65500 65503 65600 65600 ?
* e[5]:[0]:[0]:[30]:[10.2.2.0]/224
                      3.3.3.3                                        0 65500 65503 ?
*>e                   3.3.3.3                                        0 65500 65503 ?
* e[5]:[0]:[0]:[30]:[10.3.3.0]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
*>e                   3.3.3.3                                        0 65500 65503 65600 ?
* e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
*>e                   3.3.3.3                                        0 65500 65503 65600 ?

Route Distinguisher: 2.2.2.2:32832    (L2VNI 100065)
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                        100      32768 i
*>l[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                        100      32768 i

Route Distinguisher: 65502:1    (L3VNI 100500)
*>e[5]:[0]:[0]:[24]:[33.33.33.0]/224
                      3.3.3.3                                        0 65500 65503 65600 65600 ?
*>e[5]:[0]:[0]:[30]:[10.2.2.0]/224
                      3.3.3.3                                        0 65500 65503 ?
*>e[5]:[0]:[0]:[30]:[10.3.3.0]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
*>e[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      3.3.3.3                                        0 65500 65503 65600 ?
</code></pre>

LEAF3

<pre><code><strong>LEAF3# show ip bgp summary vrf COD
</strong>BGP summary information for VRF COD, address family IPv4 Unicast
BGP router identifier 10.2.2.1, local AS number 65503
BGP table version is 52, IPv4 Unicast config peers 1, capable peers 1
5 network entries and 7 paths using 1460 bytes of memory
BGP attribute entries [5/1800], BGP AS path entries [4/36]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.2.2        4 65600       1754       1748       52    0    0    1d04h 4    
     
<strong>LEAF3# show ip bgp summary vrf TENANT_1
</strong>BGP summary information for VRF TENANT_1, address family IPv4 Unicast
BGP router identifier 10.3.3.1, local AS number 65503
BGP table version is 24, IPv4 Unicast config peers 1, capable peers 1
5 network entries and 6 paths using 1556 bytes of memory
BGP attribute entries [3/1080], BGP AS path entries [2/24]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.3.3.2        4 65600       1750       1746       24    0    0    1d04h 4  

<strong>LEAF3# show ip route vrf COD
</strong>IP Route Table for VRF "COD"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%&#x3C;string>' in via output denotes VRF &#x3C;string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 1d04h, bgp-65503, external, tag 65600
10.2.2.0/30, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/5, [0/0], 1d11h, direct
10.2.2.1/32, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/5, [0/0], 1d11h, local
10.3.3.0/30, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 1d04h, bgp-65503, external, tag 65600
11.11.11.2/32, ubest/mbest: 1/0
    *via 10.1.1.100%default, [20/0], 20:09:13, bgp-65503, external, tag 65500, segid: 100500 tunnelid: 0xa010164 encap: VXLAN
 
33.33.33.0/24, ubest/mbest: 1/0
    *via 10.2.2.2, [20/0], 1d00h, bgp-65503, external, tag 65600

<strong>LEAF3# show ip route vrf TENANT_1
</strong>IP Route Table for VRF "TENANT_1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%&#x3C;string>' in via output denotes VRF &#x3C;string>

4.4.4.4/32, ubest/mbest: 1/0
    *via 10.3.3.2, [20/0], 1d04h, bgp-65503, external, tag 65600
10.2.2.0/30, ubest/mbest: 1/0
    *via 10.3.3.2, [20/0], 1d04h, bgp-65503, external, tag 65600
10.3.3.0/30, ubest/mbest: 1/0, attached
    *via 10.3.3.1, Eth1/6, [0/0], 1d11h, direct
10.3.3.1/32, ubest/mbest: 1/0, attached
    *via 10.3.3.1, Eth1/6, [0/0], 1d11h, local
11.11.11.2/32, ubest/mbest: 1/0
    *via 10.3.3.2, [20/0], 20:09:20, bgp-65503, external, tag 65600
33.33.33.0/24, ubest/mbest: 1/0, attached
    *via 33.33.33.1, Vlan67, [0/0], 1d04h, direct
33.33.33.1/32, ubest/mbest: 1/0, attached
    *via 33.33.33.1, Vlan67, [0/0], 1d04h, local
33.33.33.2/32, ubest/mbest: 1/0, attached
    *via 33.33.33.2, Vlan67, [190/0], 00:30:27, hmm
33.33.33.3/32, ubest/mbest: 1/0, attached
    *via 33.33.33.3, Vlan67, [190/0], 00:30:27, hmm   
	
<strong>LEAF3# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 3.3.3.3, local AS number 65503
BGP table version is 328, L2VPN EVPN config peers 2, capable peers 2
17 network entries and 20 paths using 5060 bytes of memory
BGP attribute entries [13/4680], BGP AS path entries [5/54]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4 65500      58310      58150      328    0    0    1d04h 2         
20.20.20.20     4 65500      58306      58146      328    0    0    1d04h 2         

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
10.10.10.10     I 65500 2          2          0          0          0         
20.20.20.20     I 65500 2          2          0          0          0  
       
<strong>LEAF3# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 328, Local Router ID is 3.3.3.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.1.1.1:32832
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                                     0 65500 65501 i
*>e                   10.1.1.100                                     0 65500 65501 i

Route Distinguisher: 2.2.2.2:32832
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                                     0 65500 65502 i
*>e                   10.1.1.100                                     0 65500 65502 i

Route Distinguisher: 3.3.3.3:32834    (L2VNI 100067)
*>l[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      3.3.3.3                           100      32768 i
*>l[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      3.3.3.3                           100      32768 i
*>l[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                           100      32768 i
*>l[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[33.33.33.2]/272
                      3.3.3.3                           100      32768 i
*>l[3]:[0]:[32]:[3.3.3.3]/88
                      3.3.3.3                           100      32768 i

Route Distinguisher: 65503:1    (L3VNI 100500)
* e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                                     0 65500 65501 i
*>e                   10.1.1.100                                     0 65500 65502 i
*>l[5]:[0]:[0]:[24]:[33.33.33.0]/224
                      3.3.3.3                                        0 65600 65600 ?
*>l[5]:[0]:[0]:[30]:[10.2.2.0]/224
                      3.3.3.3                  0        100      32768 ?
*>l[5]:[0]:[0]:[30]:[10.3.3.0]/224
                      3.3.3.3                  0                     0 65600 ?
*>l[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      3.3.3.3                  0                     0 65600 ?

Route Distinguisher: 65503:2    (L3VNI 100501)
*>l[5]:[0]:[0]:[24]:[33.33.33.0]/224
                      3.3.3.3                  0        100      32768 ?
*>l[5]:[0]:[0]:[30]:[10.2.2.0]/224
                      3.3.3.3                  0                     0 65600 ?
*>l[5]:[0]:[0]:[30]:[10.3.3.0]/224
                      3.3.3.3                  0        100      32768 ?
*>l[5]:[0]:[0]:[32]:[4.4.4.4]/224
                      3.3.3.3                  0                     0 65600 ?
*>l[5]:[0]:[0]:[32]:[11.11.11.2]/224
                      3.3.3.3                                        0 65600 65600 65500 65502 i
</code></pre>

Проверка связности между endhost-1, находящимся в VRF COD, и endhost-2, endhost-3, находящимися в VRF TENANT\_1, а также Loopback интерфейсом на EDGE роутере

<pre><code><strong>endhost-1#show run int vlan 65
</strong>Building configuration...

Current configuration : 61 bytes
!
interface Vlan65
 ip address 11.11.11.2 255.255.255.0
end

<strong>endhost-1#show ip route 
</strong>Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 11.11.11.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 11.11.11.1
      11.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        11.11.11.0/24 is directly connected, Vlan65
L        11.11.11.2/32 is directly connected, Vlan65

<strong>endhost-2/ # ip a
</strong>1: lo: &#x3C;LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
29: eth0: &#x3C;BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:ee:8b:bc:00 brd ff:ff:ff:ff:ff:ff
    inet 33.33.33.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:eeff:fe8b:bc00/64 scope link 
       valid_lft forever preferred_lft forever
	   
<strong>endhost-2/ # ip route
</strong>default via 33.33.33.1 dev eth0 
33.33.33.0/24 dev eth0 scope link  src 33.33.33.2

<strong>endhost-3/ # ip a
</strong>1: lo: &#x3C;LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
28: eth0: &#x3C;BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:24:8c:3e:00 brd ff:ff:ff:ff:ff:ff
    inet 33.33.33.3/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:24ff:fe8c:3e00/64 scope link 
       valid_lft forever preferred_lft forever
	   
<strong>endhost-3/ # ip route
</strong>default via 33.33.33.1 dev eth0 
33.33.33.0/24 dev eth0 scope link  src 33.33.33.3

<strong>endhost-1#ping 33.33.33.2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 33.33.33.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 21/25/34 ms

<strong>endhost-1#ping 33.33.33.3
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 33.33.33.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 14/20/26 ms

<strong>endhost-1#ping 4.4.4.4   
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 4.4.4.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/15/23 ms

<strong>endhost-2/ # ping 11.11.11.2
</strong>PING 11.11.11.2 (11.11.11.2) 56(84) bytes of data.
64 bytes from 11.11.11.2: icmp_seq=1 ttl=251 time=20.3 ms
64 bytes from 11.11.11.2: icmp_seq=2 ttl=251 time=28.1 ms
64 bytes from 11.11.11.2: icmp_seq=3 ttl=251 time=23.4 ms
64 bytes from 11.11.11.2: icmp_seq=4 ttl=251 time=24.1 ms
64 bytes from 11.11.11.2: icmp_seq=5 ttl=251 time=26.8 ms
64 bytes from 11.11.11.2: icmp_seq=6 ttl=251 time=23.5 ms
^C
--- 11.11.11.2 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5007ms
rtt min/avg/max/mdev = 20.292/24.354/28.084/2.514 ms

<strong>endhost-2/ # ping 4.4.4.4
</strong>PING 4.4.4.4 (4.4.4.4) 56(84) bytes of data.
64 bytes from 4.4.4.4: icmp_seq=1 ttl=254 time=12.9 ms
64 bytes from 4.4.4.4: icmp_seq=2 ttl=254 time=5.93 ms
64 bytes from 4.4.4.4: icmp_seq=3 ttl=254 time=4.72 ms
64 bytes from 4.4.4.4: icmp_seq=4 ttl=254 time=8.55 ms
64 bytes from 4.4.4.4: icmp_seq=5 ttl=254 time=10.2 ms
^C
--- 4.4.4.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 4.721/8.470/12.905/2.943 ms

<strong>endhost-3/ # ping 11.11.11.2
</strong>PING 11.11.11.2 (11.11.11.2) 56(84) bytes of data.
64 bytes from 11.11.11.2: icmp_seq=1 ttl=251 time=29.5 ms
64 bytes from 11.11.11.2: icmp_seq=2 ttl=251 time=22.4 ms
64 bytes from 11.11.11.2: icmp_seq=3 ttl=251 time=28.7 ms
64 bytes from 11.11.11.2: icmp_seq=4 ttl=251 time=23.0 ms
64 bytes from 11.11.11.2: icmp_seq=5 ttl=251 time=20.0 ms
^C
--- 11.11.11.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 20.027/24.723/29.537/3.724 ms

<strong>endhost-3/ # ping 4.4.4.4
</strong>PING 4.4.4.4 (4.4.4.4) 56(84) bytes of data.
64 bytes from 4.4.4.4: icmp_seq=1 ttl=254 time=10.8 ms
64 bytes from 4.4.4.4: icmp_seq=2 ttl=254 time=4.42 ms
64 bytes from 4.4.4.4: icmp_seq=3 ttl=254 time=3.96 ms
64 bytes from 4.4.4.4: icmp_seq=4 ttl=254 time=7.23 ms
64 bytes from 4.4.4.4: icmp_seq=5 ttl=254 time=8.37 ms
^C
--- 4.4.4.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 3.961/6.963/10.844/2.552 ms
</code></pre>

#### Дампы Wireshark

Видно, что происходит маршрутизация icmp пакетов через линки между LEAF3 и EDGE роутером в соответствующих VRF.

Далее между LEAF3 и SPINE1, SPINE2 обмен пакетов происходит по ECMP через L3VNI 100500, принадлежащий VRF COD, поскольку благодаря выполненному ликингу через внешний роутер происходит перенаправление пакета в нужны VRF.

LEAF3 <<-->> EDGE

<figure><img src="../.gitbook/assets/Dump Lab_8_3.PNG" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Dump Lab_8_4.PNG" alt=""><figcaption></figcaption></figure>

LEAF3 <<-->> SPINE1

<figure><img src="../.gitbook/assets/Dump Lab_8_1.PNG" alt=""><figcaption></figcaption></figure>

LEAF3 <<-->> SPINE2

<figure><img src="../.gitbook/assets/Dump Lab_8_2.PNG" alt=""><figcaption></figcaption></figure>

#### Выводы

Таким образом, нам удалось настроить ликинг между VRF COD и VRF TENANT\_1 с использованием ipv4 BGP сессий в соответствующих vrf между LEAF3 и EDGE роутером, находящимся в другом логическом сегменте.

Маршруты появились на LEAF3 как в ipv4, так и в l2vpn evpn таблицах.&#x20;

Связность между узлами сети имеется.
