---
description: VxLAN. L2 VNI
---

# LAB5

#### Цели

* Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами

#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_5.PNG" alt=""><figcaption></figcaption></figure>

#### Конфигурация устройств

LEAF1

```
nv overlay evpn
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 65
  name VLAN_65
  vn-segment 100065
  
route-map DIRECT-ROUTES-MAP permit 10

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100065
    ingress-replication protocol bgp

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
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 65
  name VLAN_65
  vn-segment 100065

route-map DIRECT-ROUTES-MAP permit 10

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback2
  member vni 100065
    ingress-replication protocol bgp

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
  switchport access vlan 65
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
```

LEAF3

```
nv overlay evpn
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 65
  name VLAN_65
  vn-segment 100065

route-map DIRECT-ROUTES-MAP permit 10

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback3
  member vni 100065
    ingress-replication protocol bgp

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
  switchport access vlan 65
  mtu 9216

interface Ethernet1/4
  description to_PC1
  switchport access vlan 65
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
```

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

l2vpn evpn BGP соседства

<pre><code><strong>LEAF1# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.1.1.1, local AS number 65501
BGP table version is 372, L2VPN EVPN config peers 2, capable peers 2
12 network entries and 17 paths using 3504 bytes of memory
BGP attribute entries [8/2880], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4 65500      22285      22243      372    0    0 00:07:11 5
20.20.20.20     4 65500      22284      22242      372    0    0 00:07:12 5

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5
10.10.10.10     I 65500 5          3          2          0          0
20.20.20.20     I 65500 5          3          2          0          0

<strong>SPINE1# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.10.10.10, local AS number 65500
BGP table version is 175, L2VPN EVPN config peers 3, capable peers 3
7 network entries and 7 paths using 2044 bytes of memory
BGP attribute entries [6/2160], BGP AS path entries [3/18]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4 65501      22268      22246      175    0    0 00:07:28 2
2.2.2.2         4 65502      22272      22237      175    0    0 00:07:19 2
3.3.3.3         4 65503      22271      22245      175    0    0 00:07:14 3

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5
1.1.1.1         I 65501 2          1          1          0          0
2.2.2.2         I 65502 2          1          1          0          0
3.3.3.3         I 65503 3          2          1          0          0
</code></pre>

Полученные параметры RD и RT на LEAF1

<pre><code><strong>LEAF1# sh bgp internal evi 100065 | i BGP|RD|RT
</strong>BGP L2VPN/EVPN RD Information for 1.1.1.1:32832
BGP Configured EVI Information:
  RD                         : 1.1.1.1:32832
  Secondary RD               : None
  Export RTs                 : 1
  Export RT list             :
  Import RTs                 : 1
  Import RT list             :
  Future RD                  : NULL
  RD                         : Yes(Auto)
  RD Autogenerated           : No
BGP VNI Information for L2-100065
  RD                         : 1.1.1.1:32832
  Secondary RD               : 0:0
  EAD/ES RTs                 : No
  Active Export RTs          : 1
  Active Export RT list      : 65501:100065
  Config Export RTs          : 1
  Config Export RT list      :
  Export RT chg/chg-pending  : 0/0
  EAD/ES RTs                 : No
  Active Import RTs          : 1
  Active Import RT list      : 65501:100065
  Config Import RTs          : 1
  Config Import RT list      :
  Import RT chg/chg-pending  : 0/0
</code></pre>

l2vpn evpn маршруты

<pre><code><strong>LEAF1# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 372, Local Router ID is 1.1.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.1.1.1:32832    (L2VNI 100065)
*>e[2]:[0]:[0]:[48]:[0242.3322.0900]:[0]:[0.0.0.0]/216
                      2.2.2.2                                        0 65500 65502 i
*>e[2]:[0]:[0]:[48]:[0242.48c5.3100]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.5cca.f100]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65500 65503 i
*>l[2]:[0]:[0]:[48]:[0242.689d.c100]:[0]:[0.0.0.0]/216
                      1.1.1.1                           100      32768 i
*>l[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                           100      32768 i
*>e[3]:[0]:[32]:[2.2.2.2]/88
                      2.2.2.2                                        0 65500 65502 i
*>e[3]:[0]:[32]:[3.3.3.3]/88
                      3.3.3.3                                        0 65500 65503 i

Route Distinguisher: 2.2.2.2:32832
*>e[2]:[0]:[0]:[48]:[0242.3322.0900]:[0]:[0.0.0.0]/216
                      2.2.2.2                                        0 65500 65502 i
* e                   2.2.2.2                                        0 65500 65502 i
*>e[3]:[0]:[32]:[2.2.2.2]/88
                      2.2.2.2                                        0 65500 65502 i
* e                   2.2.2.2                                        0 65500 65502 i

Route Distinguisher: 3.3.3.3:32832
* e[2]:[0]:[0]:[48]:[0242.48c5.3100]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65500 65503 i
*>e                   3.3.3.3                                        0 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.5cca.f100]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65500 65503 i
*>e                   3.3.3.3                                        0 65500 65503 i
*>e[3]:[0]:[32]:[3.3.3.3]/88
                      3.3.3.3                                        0 65500 65503 i
* e                   3.3.3.3                                        0 65500 65503 i

<strong>LEAF1# show bgp l2vpn evpn 0242.3322.0900
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
Route Distinguisher: 1.1.1.1:32832    (L2VNI 100065)
BGP routing table entry for [2]:[0]:[0]:[48]:[0242.3322.0900]:[0]:[0.0.0.0]/216, version 361
Paths: (1 available, best #1)
Flags: (0x000212) (high32 00000000) on xmit-list, is in l2rib/evpn, is not in HW
Multipath: eBGP

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop, in rib
             Imported from 2.2.2.2:32832:[2]:[0]:[0]:[48]:[0242.3322.0900]:[0]:[0.0.0.0]/216
  AS-Path: 65500 65502 , path sourced external to AS
    2.2.2.2 (metric 0) from 20.20.20.20 (20.20.20.20)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 100065
      Extcommunity: RT:65501:100065 ENCAP:8

  Path-id 1 not advertised to any peer

Route Distinguisher: 2.2.2.2:32832
BGP routing table entry for [2]:[0]:[0]:[48]:[0242.3322.0900]:[0]:[0.0.0.0]/216, version 363
Paths: (2 available, best #1)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
Multipath: eBGP

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported to 1 destination(s)
             Imported paths list: L2-100065
  AS-Path: 65500 65502 , path sourced external to AS
    2.2.2.2 (metric 0) from 20.20.20.20 (20.20.20.20)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 100065
      Extcommunity: RT:65501:100065 ENCAP:8

  Path type: external, path is valid, not best reason: newer EBGP path, no labeled nexthop
  AS-Path: 65500 65502 , path sourced external to AS
    2.2.2.2 (metric 0) from 10.10.10.10 (10.10.10.10)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 100065
      Extcommunity: RT:65501:100065 ENCAP:8

  Path-id 1 not advertised to any peer
  
<strong>LEAF1# show l2route fl all
</strong>Topology ID Peer-id     Flood List                              Label(VNI) Service Node
----------- ----------- -------------------------------------- ------------ ------------
65          1           2.2.2.2                                 100065       no
65          2           3.3.3.3                                 100065       no

<strong>LEAF1# sh l2route mac all
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
65          0242.3322.0900 BGP    Rcv                0          2.2.2.2 (Label: 100065)
65          0242.48c5.3100 BGP    Rcv                0          3.3.3.3 (Label: 100065)
65          0242.5cca.f100 BGP    Rcv                0          3.3.3.3 (Label: 100065)
65          0242.689d.c100 Local  L,                 0          Eth1/3
</code></pre>

<pre><code><strong>SPINE1# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 175, Local Router ID is 10.10.10.10
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.1.1.1:32832
*>e[2]:[0]:[0]:[48]:[0242.689d.c100]:[0]:[0.0.0.0]/216
                      1.1.1.1                                        0 65501 i
*>e[3]:[0]:[32]:[1.1.1.1]/88
                      1.1.1.1                                        0 65501 i

Route Distinguisher: 2.2.2.2:32832
*>e[2]:[0]:[0]:[48]:[0242.3322.0900]:[0]:[0.0.0.0]/216
                      2.2.2.2                                        0 65502 i
*>e[3]:[0]:[32]:[2.2.2.2]/88
                      2.2.2.2                                        0 65502 i

Route Distinguisher: 3.3.3.3:32832
*>e[2]:[0]:[0]:[48]:[0242.48c5.3100]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.5cca.f100]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65503 i
*>e[3]:[0]:[32]:[3.3.3.3]/88
                      3.3.3.3                                        0 65503 i                         0 65503 i

<strong>SPINE1# show bgp l2vpn evpn 0242.3322.0900
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
Route Distinguisher: 2.2.2.2:32832
BGP routing table entry for [2]:[0]:[0]:[48]:[0242.3322.0900]:[0]:[0.0.0.0]/216, version 168
Paths: (1 available, best #1)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
  AS-Path: 65502 , path sourced external to AS
    2.2.2.2 (metric 0) from 2.2.2.2 (2.2.2.2)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 100065
      Extcommunity: RT:65500:100065 ENCAP:8

  Path-id 1 advertised to peers:
    1.1.1.1            3.3.3.3
</code></pre>

Проверка связности между endhost-1, endhost-2, endhost-3 и endhost-4&#x20;

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
14: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 02:42:68:9d:c1:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.1/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:68ff:fe9d:c100/64 scope link
       valid_lft forever preferred_lft forever

/ # ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=8.31 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=11.6 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=7.90 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=13.6 ms
64 bytes from 192.168.1.2: icmp_seq=5 ttl=64 time=12.6 ms
^C
--- 192.168.1.2 ping statistics ---
58 packets transmitted, 58 received, 0% packet loss, time 57071ms
rtt min/avg/max/mdev = 7.309/13.937/21.076/2.518 ms

/ # ping 192.168.1.3
PING 192.168.1.3 (192.168.1.3) 56(84) bytes of data.
64 bytes from 192.168.1.3: icmp_seq=1 ttl=64 time=30.4 ms
64 bytes from 192.168.1.3: icmp_seq=2 ttl=64 time=17.7 ms
64 bytes from 192.168.1.3: icmp_seq=3 ttl=64 time=15.0 ms
64 bytes from 192.168.1.3: icmp_seq=4 ttl=64 time=16.9 ms
64 bytes from 192.168.1.3: icmp_seq=5 ttl=64 time=15.2 ms
^C
--- 192.168.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 15.038/19.042/30.381/5.757 ms

/ # ping 192.168.1.4
PING 192.168.1.4 (192.168.1.4) 56(84) bytes of data.
64 bytes from 192.168.1.4: icmp_seq=1 ttl=64 time=17.5 ms
64 bytes from 192.168.1.4: icmp_seq=2 ttl=64 time=7.36 ms
64 bytes from 192.168.1.4: icmp_seq=3 ttl=64 time=14.5 ms
64 bytes from 192.168.1.4: icmp_seq=4 ttl=64 time=14.7 ms
64 bytes from 192.168.1.4: icmp_seq=5 ttl=64 time=21.9 ms
^C
--- 192.168.1.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 7.359/15.184/21.879/4.737 ms
```

#### Wireshark dump

Захват трафика на линках между LEAF1 и SPINE1, а также LEAF1 и SPINE2 показывает, что у нас работает ECMP, потому что пакеты ICMP, инкапсулированные в VXLAN видны в обоих случаях.

<figure><img src="../.gitbook/assets/Dump Lab_5.PNG" alt=""><figcaption></figcaption></figure>

#### Выводы

Таким образом, нам удалось настроить VXLAN туннели, использующие EVPN в качестве Control Plain протокола, распространяющего маршрутную информацию о mac адресах устройств.&#x20;

Соседства l2vpn evpn bgp успешно поднялись и маршрутная информация на уровне L2 получена всеми устройствами.

Связность между узлами сети имеется.&#x20;
