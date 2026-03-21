---
description: VxLAN. L3VNI
---

# LAB6

#### Цели

* Настроить маршрутизацию в рамках Overlay между клиентами, находящимися в разных vlan (vni).

#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_6.PNG" alt=""><figcaption></figcaption></figure>

#### Конфигурация устройств

LEAF1

```
nv overlay evpn
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001

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

interface Vlan65
  no shutdown
  vrf member COD
  ip address 11.11.11.1/24
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  vrf member COD
  no ip redirects
  ip forward

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
  description to_PC1
  switchport access vlan 65
  mtu 9216

interface loopback1
  ip address 1.1.1.1/32
  
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
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001

vlan 66
  name VLAN_66
  vn-segment 100066
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

interface Vlan66
  no shutdown
  vrf member COD
  ip address 22.22.22.1/24
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  vrf member COD
  no ip redirects
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback2
  member vni 100065
    ingress-replication protocol bgp
  member vni 100066
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
  description to_PC1
  switchport access vlan 66
  mtu 9216

interface loopback2
  ip address 2.2.2.2/32
  
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
  neighbor 10.10.10.10
    inherit peer OVERLAY
  neighbor 20.1.1.1
    inherit peer UNDERLAY
  neighbor 20.1.2.1
    inherit peer UNDERLAY
  neighbor 20.20.20.20
    inherit peer OVERLAY
evpn
  vni 100065 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 100066 l2
    rd auto
    route-target import auto
    route-target export auto
```

LEAF3

```
nv overlay evpn
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001

vlan 67
  name VLAN_67
  vn-segment 100067
vlan 500
  name L3-VNI-COD
  vn-segment 100500

route-map DIRECT-ROUTES-MAP permit 10
vrf context COD
  vni 100500
  rd 65503:1
  address-family ipv4 unicast
    route-target import 11:11 evpn
    route-target import 1:1
    route-target export 11:11 evpn
    route-target export 1:1

interface Vlan67
  no shutdown
  vrf member COD
  ip address 33.33.33.1/24
  fabric forwarding mode anycast-gateway

interface Vlan500
  description L3-VNI-COD
  no shutdown
  vrf member COD
  no ip redirects
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback3
  member vni 100065
    ingress-replication protocol bgp
  member vni 100067
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
  description to_PC1
  switchport access vlan 67
  mtu 9216

interface Ethernet1/4
  description to_PC1
  switchport access vlan 67
  mtu 9216

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

Конфигурация спайнов без изменений

SPINE1

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
  ip address 10.1.1.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  mtu 9216
  ip address 20.1.1.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  mtu 9216
  ip address 30.1.1.1/30
  no shutdown

interface loopback10
  ip address 10.10.10.10/32
  
router bgp 65500
  router-id 10.10.10.10
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
    remote-as 65501
  neighbor 2.2.2.2
    inherit peer OVERLAY
    remote-as 65502
  neighbor 3.3.3.3
    inherit peer OVERLAY
    remote-as 65503
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
  ip address 10.1.2.1/30
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  mtu 9216
  ip address 20.1.2.1/30
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  mtu 9216
  ip address 30.1.2.1/30
  no shutdown

interface loopback20
  ip address 20.20.20.20/32

router bgp 65500
  router-id 20.20.20.20
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map DIRECT-ROUTES-MAP
  address-family l2vpn evpn
    retain route-target all
  template peer OVERLAY
    update-source loopback20
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
    remote-as 65501
  neighbor 2.2.2.2
    inherit peer OVERLAY
    remote-as 65502
  neighbor 3.3.3.3
    inherit peer OVERLAY
    remote-as 65503
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

#### Проверка

Видно, что появились l2vpn evpn маршруты уровня L3

<pre><code><strong>LEAF1# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 114, Local Router ID is 1.1.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.1.1.1:32832    (L2VNI 100065)
*>l[2]:[0]:[0]:[48]:[0242.a2c4.5b00]:[0]:[0.0.0.0]/216
                      1.1.1.1                           100      32768 i
*>l[2]:[0]:[0]:[48]:[0242.a2c4.5b00]:[32]:[11.11.11.2]/272
                      1.1.1.1                           100      32768 i
*>l[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                           100      32768 i

Route Distinguisher: 2.2.2.2:32833
* e[2]:[0]:[0]:[48]:[0242.4410.cb00]:[32]:[22.22.22.2]/272
                      2.2.2.2                                        0 65500 65502 i
*>e                   2.2.2.2                                        0 65500 65502 i

Route Distinguisher: 3.3.3.3:32834
* e[2]:[0]:[0]:[48]:[0242.0bca.8900]:[32]:[33.33.33.2]/272
                      3.3.3.3                                        0 65500 65503 i
*>e                   3.3.3.3                                        0 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                                        0 65500 65503 i
*>e                   3.3.3.3                                        0 65500 65503 i

<strong>Route Distinguisher: 65501:1    (L3VNI 100500)
</strong><strong>*>e[2]:[0]:[0]:[48]:[0242.0bca.8900]:[32]:[33.33.33.2]/272
</strong><strong>                      3.3.3.3                                        0 65500 65503 i
</strong><strong>*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
</strong><strong>                      3.3.3.3                                        0 65500 65503 i
</strong><strong>*>e[2]:[0]:[0]:[48]:[0242.4410.cb00]:[32]:[22.22.22.2]/272
</strong><strong>                      2.2.2.2                                        0 65500 65502 i
</strong>					  
<strong>LEAF1# show bgp l2vpn evpn 22.22.22.2
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
Route Distinguisher: 2.2.2.2:32833
BGP routing table entry for [2]:[0]:[0]:[48]:[0242.4410.cb00]:[32]:[22.22.22.2]/272, version 46
Paths: (2 available, best #2)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
Multipath: eBGP

  Path type: external, path is valid, not best reason: newer EBGP path, no labeled nexthop
  AS-Path: 65500 65502 , path sourced external to AS
    2.2.2.2 (metric 0) from 20.20.20.20 (20.20.20.20)
      Origin IGP, MED not set, localpref 100, weight 0
<strong>      Received label 100066 100500
</strong><strong>      Extcommunity: RT:11:11 RT:65501:100066 ENCAP:8 Router MAC:0cd2.0000.1b08
</strong>
  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported to 2 destination(s)
             Imported paths list: COD L3-100500
  AS-Path: 65500 65502 , path sourced external to AS
    2.2.2.2 (metric 0) from 10.10.10.10 (10.10.10.10)
      Origin IGP, MED not set, localpref 100, weight 0
<strong>      Received label 100066 100500
</strong><strong>      Extcommunity: RT:11:11 RT:65501:100066 ENCAP:8 Router MAC:0cd2.0000.1b08
</strong>
  Path-id 1 not advertised to any peer

Route Distinguisher: 65501:1    (L3VNI 100500)
BGP routing table entry for [2]:[0]:[0]:[48]:[0242.4410.cb00]:[32]:[22.22.22.2]/272, version 45
Paths: (1 available, best #1)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
Multipath: eBGP

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported from 2.2.2.2:32833:[2]:[0]:[0]:[48]:[0242.4410.cb00]:[32]:[22.22.22.2]/272
  AS-Path: 65500 65502 , path sourced external to AS
    2.2.2.2 (metric 0) from 10.10.10.10 (10.10.10.10)
      Origin IGP, MED not set, localpref 100, weight 0
<strong>      Received label 100066 100500
</strong><strong>      Extcommunity: RT:11:11 RT:65501:100066 ENCAP:8 Router MAC:0cd2.0000.1b08
</strong>
  Path-id 1 not advertised to any peer
  
<strong>LEAF2# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 297, Local Router ID is 2.2.2.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.1.1.1:32832
* e[2]:[0]:[0]:[48]:[0242.a2c4.5b00]:[32]:[11.11.11.2]/272
                      1.1.1.1                                        0 65500 65501 i
*>e                   1.1.1.1                                        0 65500 65501 i

Route Distinguisher: 2.2.2.2:32833    (L2VNI 100066)
*>l[2]:[0]:[0]:[48]:[0242.4410.cb00]:[0]:[0.0.0.0]/216
                      2.2.2.2                           100      32768 i
*>l[2]:[0]:[0]:[48]:[0242.4410.cb00]:[32]:[22.22.22.2]/272
                      2.2.2.2                           100      32768 i
*>l[3]:[0]:[32]:[2.2.2.2]/88
                      2.2.2.2                           100      32768 i

Route Distinguisher: 3.3.3.3:32834
* e[2]:[0]:[0]:[48]:[0242.0bca.8900]:[32]:[33.33.33.2]/272
                      3.3.3.3                                        0 65500 65503 i
*>e                   3.3.3.3                                        0 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                                        0 65500 65503 i
*>e                   3.3.3.3                                        0 65500 65503 i

<strong>Route Distinguisher: 65502:1    (L3VNI 100500)
</strong><strong>*>e[2]:[0]:[0]:[48]:[0242.0bca.8900]:[32]:[33.33.33.2]/272
</strong><strong>                      3.3.3.3                                        0 65500 65503 i
</strong><strong>*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
</strong><strong>                      3.3.3.3                                        0 65500 65503 i
</strong><strong>*>e[2]:[0]:[0]:[48]:[0242.a2c4.5b00]:[32]:[11.11.11.2]/272
</strong><strong>                      1.1.1.1                                        0 65500 65501 i
</strong>
<strong>LEAF2# show bgp l2vpn evpn 11.11.11.2
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
Route Distinguisher: 1.1.1.1:32832
BGP routing table entry for [2]:[0]:[0]:[48]:[0242.a2c4.5b00]:[32]:[11.11.11.2]/272, version 278
Paths: (2 available, best #2)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
Multipath: eBGP

  Path type: external, path is valid, not best reason: newer EBGP path, no labeled nexthop
  AS-Path: 65500 65501 , path sourced external to AS
    1.1.1.1 (metric 0) from 10.10.10.10 (10.10.10.10)
      Origin IGP, MED not set, localpref 100, weight 0
<strong>      Received label 100065 100500
</strong><strong>      Extcommunity: RT:11:11 RT:65502:100065 ENCAP:8 Router MAC:0ce9.0000.1b08
</strong>
  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported to 2 destination(s)
             Imported paths list: COD L3-100500
  AS-Path: 65500 65501 , path sourced external to AS
    1.1.1.1 (metric 0) from 20.20.20.20 (20.20.20.20)
      Origin IGP, MED not set, localpref 100, weight 0
<strong>      Received label 100065 100500
</strong><strong>      Extcommunity: RT:11:11 RT:65502:100065 ENCAP:8 Router MAC:0ce9.0000.1b08
</strong>
  Path-id 1 not advertised to any peer

Route Distinguisher: 65502:1    (L3VNI 100500)
BGP routing table entry for [2]:[0]:[0]:[48]:[0242.a2c4.5b00]:[32]:[11.11.11.2]/272, version 276
Paths: (1 available, best #1)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
Multipath: eBGP

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported from 1.1.1.1:32832:[2]:[0]:[0]:[48]:[0242.a2c4.5b00]:[32]:[11.11.11.2]/272
  AS-Path: 65500 65501 , path sourced external to AS
    1.1.1.1 (metric 0) from 20.20.20.20 (20.20.20.20)
      Origin IGP, MED not set, localpref 100, weight 0
<strong>      Received label 100065 100500
</strong><strong>      Extcommunity: RT:11:11 RT:65502:100065 ENCAP:8 Router MAC:0ce9.0000.1b08
</strong>
  Path-id 1 not advertised to any peer
</code></pre>

Также видим, что nve интерфейс теперь принадлежит новому vni 100500. Кроме того, появились маршруты, связанные с ним и соответствующим вланом 500.

<pre><code><strong>LEAF1# show nve vni
</strong>Codes: CP - Control Plane        DP - Data Plane
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

<strong>LEAF1# show l2route mac all
</strong>
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
65          0242.a2c4.5b00 Local  L,                 0          Eth1/3
500         0cd2.0000.1b08 VXLAN  Rmac               0          2.2.2.2
500         0cde.0000.1b08 VXLAN  Rmac               0          3.3.3.3

<strong>LEAF2# show nve vni
</strong>Codes: CP - Control Plane        DP - Data Plane
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

<strong>LEAF2# show l2route mac all
</strong>
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
66          0242.4410.cb00 Local  L,                 0          Eth1/3
500         0cde.0000.1b08 VXLAN  Rmac               0          3.3.3.3
500         0ce9.0000.1b08 VXLAN  Rmac               0          1.1.1.1
</code></pre>

Проверка связности между endhost-1, endhost-2, endhost-3 и endhost-4

```
#PC1#
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
24: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:a2:c4:5b:00 brd ff:ff:ff:ff:ff:ff
    inet 11.11.11.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a2ff:fec4:5b00/64 scope link
       valid_lft forever preferred_lft forever
	   
/ # ip route
default via 11.11.11.1 dev eth0
11.11.11.0/24 dev eth0 scope link  src 11.11.11.2

/ # ping 22.22.22.2
PING 22.22.22.2 (22.22.22.2) 56(84) bytes of data.
64 bytes from 22.22.22.2: icmp_seq=1 ttl=62 time=28.8 ms
64 bytes from 22.22.22.2: icmp_seq=2 ttl=62 time=17.5 ms
64 bytes from 22.22.22.2: icmp_seq=3 ttl=62 time=13.2 ms
64 bytes from 22.22.22.2: icmp_seq=4 ttl=62 time=16.8 ms
64 bytes from 22.22.22.2: icmp_seq=5 ttl=62 time=16.2 ms
^C
--- 22.22.22.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 13.176/18.493/28.776/5.347 ms

/ # ping 33.33.33.2
PING 33.33.33.2 (33.33.33.2) 56(84) bytes of data.
64 bytes from 33.33.33.2: icmp_seq=1 ttl=62 time=23.4 ms
64 bytes from 33.33.33.2: icmp_seq=2 ttl=62 time=15.1 ms
64 bytes from 33.33.33.2: icmp_seq=3 ttl=62 time=17.4 ms
64 bytes from 33.33.33.2: icmp_seq=4 ttl=62 time=16.2 ms
64 bytes from 33.33.33.2: icmp_seq=5 ttl=62 time=14.1 ms
^C
--- 33.33.33.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 14.140/17.242/23.405/3.269 ms

/ # ping 33.33.33.3
PING 33.33.33.3 (33.33.33.3) 56(84) bytes of data.
64 bytes from 33.33.33.3: icmp_seq=1 ttl=62 time=18.4 ms
64 bytes from 33.33.33.3: icmp_seq=2 ttl=62 time=15.0 ms
64 bytes from 33.33.33.3: icmp_seq=3 ttl=62 time=15.5 ms
64 bytes from 33.33.33.3: icmp_seq=4 ttl=62 time=20.3 ms
64 bytes from 33.33.33.3: icmp_seq=5 ttl=62 time=15.0 ms
^C
--- 33.33.33.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 15.003/16.828/20.268/2.128 ms
```

#### Wireshark dump

Захват трафика на линках показывает, что трафик инкапсулируется в L3 VNI 100500

<figure><img src="../.gitbook/assets/Dump Lab_6.PNG" alt=""><figcaption></figcaption></figure>

#### Выводы

Таким образом, нам удалось настроить L3 маршрутизацию между разными VNI с использованием дополнительного VNI 100500 (vlan 500) и vrf COD. Также была настроен anycast gateway.&#x20;

Маршруты l2vpn evpn уровня L3 успешно изучаются, а VTEPs осуществляют замену маков на уровне Ethernet и таким образом осуществляют маршрутизацию.

Связность между узлами сети имеется.
