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

#### Проверка

eBGP соседства

<pre><code><strong>LEAF1# show ip bgp summary
</strong>BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.1, local AS number 65501
BGP table version is 18, IPv4 Unicast config peers 2, capable peers 2
15 network entries and 26 paths using 5436 bytes of memory
BGP attribute entries [4/1440], BGP AS path entries [3/26]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.1        4 65500         27         27       18    0    0 00:01:00 11
10.1.2.1        4 65500         29         27       18    0    0 00:01:02 11

LEAF1#show bgp ipv4 unicast neighbors
BGP neighbor is 10.1.1.1, remote AS 65500, ebgp link, Peer index 3
  Inherits peer configuration from peer-template UNDERLAY
  BGP version 4, remote router ID 10.10.10.10
  Neighbor previous state = OpenConfirm
  BGP state = Established, up for 00:03:35
  Neighbor vrf: default
  Peer is directly attached, interface Ethernet1/1
  Enable logging neighbor events
  Last read 0.901969, hold time = 9, keepalive interval is 3 seconds
  Last written 0.890489, keepalive timer expiry due 00:00:02
  Received 79 messages, 0 notifications, 0 bytes in queue
  Sent 79 messages, 0 notifications, 0(0) bytes in queue
  Enhanced error processing: On
    0 discarded attributes
  Connections established 1, dropped 0
  Last update recd 00:03:30, Last update sent  = 00:03:30
   Last reset by us never, due to No error
  Last error length sent: 0
  Reset error value sent: 0
  Reset error sent major: 0 minor: 0
  Notification data sent:
  Last reset by peer never, due to No error
  Last error length received: 0
  Reset error value received 0
  Reset error received major: 0 minor: 0
  Notification data received:

  Neighbor capabilities:
  Dynamic capability: advertised (mp, refresh, gr) received (mp, refresh, gr)
  Dynamic capability (old): advertised received
  Route refresh capability (new): advertised received
  Route refresh capability (old): advertised received
  4-Byte AS capability: advertised received
  Address family IPv4 Unicast: advertised received
  Graceful Restart capability: advertised received

  Graceful Restart Parameters:
  Address families advertised to peer:
    IPv4 Unicast
  Address families received from peer:
    IPv4 Unicast
  Forwarding state preserved by peer for:
  Restart time advertised to peer: 120 seconds
  Stale time for routes advertised by peer: 300 seconds
  Restart time advertised by peer: 120 seconds
  Extended Next Hop Encoding Capability: advertised received
  Receive IPv6 next hop encoding Capability for AF:
    IPv4 Unicast  VPNv4 Unicast

  Message statistics:
                              Sent               Rcvd
  Opens:                         1                  1
  Notifications:                 0                  0
  Updates:                       3                  4
  Keepalives:                   75                 73
  Route Refresh:                 0                  0
  Capability:                    1                  1
  Total:                        79                 79
  Total bytes:                1570               1638
  Bytes in queue:                0                  0

  For address family: IPv4 Unicast
  BGP table version 18, neighbor version 18
  11 accepted prefixes (11 paths), consuming 3168 bytes of memory
  0 received prefixes treated as withdrawn
  4 sent prefixes (4 paths)
  Last End-of-RIB received 00:00:05 after session start
  Last End-of-RIB sent 00:00:05 after session start
  First convergence 00:00:05 after session start with 4 routes sent

  Local host: 10.1.1.2, Local port: 29180
  Foreign host: 10.1.1.1, Foreign port: 179
  fd = 83

BGP neighbor is 10.1.2.1, remote AS 65500, ebgp link, Peer index 4
  Inherits peer configuration from peer-template UNDERLAY
  BGP version 4, remote router ID 20.20.20.20
  Neighbor previous state = OpenConfirm
  BGP state = Established, up for 00:03:38
  Neighbor vrf: default
  Peer is directly attached, interface Ethernet1/2
  Enable logging neighbor events
  Last read 0.311138, hold time = 9, keepalive interval is 3 seconds
  Last written 0.888292, keepalive timer expiry due 00:00:02
  Received 81 messages, 0 notifications, 0 bytes in queue
  Sent 79 messages, 0 notifications, 0(0) bytes in queue
  Enhanced error processing: On
    0 discarded attributes
  Connections established 1, dropped 0
  Last update recd 00:03:33, Last update sent  = 00:03:30
   Last reset by us never, due to No error
  Last error length sent: 0
  Reset error value sent: 0
  Reset error sent major: 0 minor: 0
  Notification data sent:
  Last reset by peer never, due to No error
  Last error length received: 0
  Reset error value received 0
  Reset error received major: 0 minor: 0
  Notification data received:

  Neighbor capabilities:
  Dynamic capability: advertised (mp, refresh, gr) received (mp, refresh, gr)
  Dynamic capability (old): advertised received
  Route refresh capability (new): advertised received
  Route refresh capability (old): advertised received
  4-Byte AS capability: advertised received
  Address family IPv4 Unicast: advertised received
  Graceful Restart capability: advertised received

  Graceful Restart Parameters:
  Address families advertised to peer:
    IPv4 Unicast
  Address families received from peer:
    IPv4 Unicast
  Forwarding state preserved by peer for:
  Restart time advertised to peer: 120 seconds
  Stale time for routes advertised by peer: 300 seconds
  Restart time advertised by peer: 120 seconds
  Extended Next Hop Encoding Capability: advertised received
  Receive IPv6 next hop encoding Capability for AF:
    IPv4 Unicast  VPNv4 Unicast

  Message statistics:
                              Sent               Rcvd
  Opens:                         1                  1
  Notifications:                 0                  0
  Updates:                       3                  4
  Keepalives:                   75                 75
  Route Refresh:                 0                  0
  Capability:                    1                  1
  Total:                        79                 81
  Total bytes:                1570               1676
  Bytes in queue:                0                  0

  For address family: IPv4 Unicast
  BGP table version 18, neighbor version 18
  11 accepted prefixes (11 paths), consuming 3168 bytes of memory
  0 received prefixes treated as withdrawn
  4 sent prefixes (4 paths)
  Last End-of-RIB received 00:00:05 after session start
  Last End-of-RIB sent 00:00:07 after session start
  First convergence 00:00:07 after session start with 4 routes sent

  Local host: 10.1.2.2, Local port: 51910
  Foreign host: 10.1.2.1, Foreign port: 179
  fd = 82
  
  SPINE1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.10.10.10, local AS number 65500
BGP table version is 29, IPv4 Unicast config peers 3, capable peers 3
14 network entries and 17 paths using 4376 bytes of memory
BGP attribute entries [4/1440], BGP AS path entries [3/18]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.2        4 65501      19461      19458       29    0    0 00:05:27 4
20.1.1.2        4 65502      19609      19607       29    0    0 16:20:15 4
30.1.1.2        4 65503      19609      19609       29    0    0 16:20:16 5

<strong>SPINE1# show bgp ipv4 unicast neighbors
</strong>BGP neighbor is 10.1.1.2, remote AS 65501, ebgp link, Peer index 3
  Inherits peer configuration from peer-template UNDERLAY
  BGP version 4, remote router ID 1.1.1.1
  Neighbor previous state = OpenConfirm
  BGP state = Established, up for 00:06:03
  Neighbor vrf: default
  Peer is directly attached, interface Ethernet1/1
  Enable logging neighbor events
  Last read 00:00:01, hold time = 9, keepalive interval is 3 seconds
  Last written 00:00:01, keepalive timer expiry due 00:00:01
  Received 19473 messages, 0 notifications, 0 bytes in queue
  Sent 19470 messages, 1 notifications, 0(0) bytes in queue
  Enhanced error processing: On
    0 discarded attributes
  Connections established 2, dropped 1
  Last update recd 00:05:58, Last update sent  = 00:05:58
   Last reset by us 00:15:21, due to holdtimer expired error
  Last error length sent: 0
  Reset error value sent: 0
  Reset error sent major: 4 minor: 0
  Notification data sent:
  Last reset by peer never, due to No error
  Last error length received: 0
  Reset error value received 0
  Reset error received major: 0 minor: 0
  Notification data received:

  Neighbor capabilities:
  Dynamic capability: advertised (mp, refresh, gr) received (mp, refresh, gr)
  Dynamic capability (old): advertised received
  Route refresh capability (new): advertised received
  Route refresh capability (old): advertised received
  4-Byte AS capability: advertised received
  Address family IPv4 Unicast: advertised received
  Graceful Restart capability: advertised received

  Graceful Restart Parameters:
  Address families advertised to peer:
    IPv4 Unicast
  Address families received from peer:
    IPv4 Unicast
  Forwarding state preserved by peer for:
  Restart time advertised to peer: 120 seconds
  Stale time for routes advertised by peer: 300 seconds
  Restart time advertised by peer: 120 seconds
  Extended Next Hop Encoding Capability: advertised received
  Receive IPv6 next hop encoding Capability for AF:
    IPv4 Unicast  VPNv4 Unicast

  Message statistics:
                              Sent               Rcvd
  Opens:                         2                  2
  Notifications:                 1                  0
  Updates:                      11                  5
  Keepalives:                19462              19463
  Route Refresh:                 0                  0
  Capability:                    3                  3
  Total:                     19470              19473
  Total bytes:              370504             370085
  Bytes in queue:                0                  0

  For address family: IPv4 Unicast
  BGP table version 29, neighbor version 29
  4 accepted prefixes (4 paths), consuming 1152 bytes of memory
  0 received prefixes treated as withdrawn
  11 sent prefixes (11 paths)
  Last End-of-RIB received 00:00:05 after session start
  Last End-of-RIB sent 00:00:05 after session start
  First convergence 00:00:05 after session start with 11 routes sent

  Local host: 10.1.1.1, Local port: 179
  Foreign host: 10.1.1.2, Foreign port: 29180
  fd = 82

BGP neighbor is 20.1.1.2, remote AS 65502, ebgp link, Peer index 4
  Inherits peer configuration from peer-template UNDERLAY
  BGP version 4, remote router ID 2.2.2.2
  Neighbor previous state = OpenConfirm
  BGP state = Established, up for 16:20:51
  Neighbor vrf: default
  Peer is directly attached, interface Ethernet1/2
  Enable logging neighbor events
  Last read 00:00:02, hold time = 9, keepalive interval is 3 seconds
  Last written 00:00:01, keepalive timer expiry due 00:00:01
  Received 19621 messages, 0 notifications, 0 bytes in queue
  Sent 19619 messages, 0 notifications, 0(0) bytes in queue
  Enhanced error processing: On
    0 discarded attributes
  Connections established 1, dropped 0
  Last update recd 16:14:04, Last update sent  = 00:05:58
   Last reset by us never, due to No error
  Last error length sent: 0
  Reset error value sent: 0
  Reset error sent major: 0 minor: 0
  Notification data sent:
  Last reset by peer never, due to No error
  Last error length received: 0
  Reset error value received 0
  Reset error received major: 0 minor: 0
  Notification data received:

  Neighbor capabilities:
  Dynamic capability: advertised (mp, refresh, gr) received (mp, refresh, gr)
  Dynamic capability (old): advertised received
  Route refresh capability (new): advertised received
  Route refresh capability (old): advertised received
  4-Byte AS capability: advertised received
  Address family IPv4 Unicast: advertised received
  Graceful Restart capability: advertised received

  Graceful Restart Parameters:
  Address families advertised to peer:
    IPv4 Unicast
  Address families received from peer:
    IPv4 Unicast
  Forwarding state preserved by peer for:
  Restart time advertised to peer: 120 seconds
  Stale time for routes advertised by peer: 300 seconds
  Restart time advertised by peer: 120 seconds
  Extended Next Hop Encoding Capability: advertised received
  Receive IPv6 next hop encoding Capability for AF:
    IPv4 Unicast  VPNv4 Unicast

  Message statistics:
                              Sent               Rcvd
  Opens:                         1                  1
  Notifications:                 0                  0
  Updates:                       8                  2
  Keepalives:                19614              19616
  Route Refresh:                 0                  0
  Capability:                    2                  2
  Total:                     19619              19621
  Total bytes:              373178             372847
  Bytes in queue:                0                  0

  For address family: IPv4 Unicast
  BGP table version 29, neighbor version 29
  4 accepted prefixes (4 paths), consuming 1152 bytes of memory
  0 received prefixes treated as withdrawn
  11 sent prefixes (11 paths)
  Last End-of-RIB received 00:00:05 after session start
  Last End-of-RIB sent 00:00:05 after session start
  First convergence 00:00:05 after session start with 4 routes sent

  Local host: 20.1.1.1, Local port: 51750
  Foreign host: 20.1.1.2, Foreign port: 179
  fd = 84

BGP neighbor is 30.1.1.2, remote AS 65503, ebgp link, Peer index 5
  Inherits peer configuration from peer-template UNDERLAY
  BGP version 4, remote router ID 3.3.3.3
  Neighbor previous state = OpenConfirm
  BGP state = Established, up for 16:20:52
  Neighbor vrf: default
  Peer is directly attached, interface Ethernet1/3
  Enable logging neighbor events
  Last read 0.452312, hold time = 9, keepalive interval is 3 seconds
  Last written 00:00:01, keepalive timer expiry due 00:00:01
  Received 19621 messages, 0 notifications, 0 bytes in queue
  Sent 19621 messages, 0 notifications, 0(0) bytes in queue
  Enhanced error processing: On
    0 discarded attributes
  Connections established 1, dropped 0
  Last update recd 16:13:56, Last update sent  = 00:05:58
   Last reset by us never, due to No error
  Last error length sent: 0
  Reset error value sent: 0
  Reset error sent major: 0 minor: 0
  Notification data sent:
  Last reset by peer never, due to No error
  Last error length received: 0
  Reset error value received 0
  Reset error received major: 0 minor: 0
  Notification data received:

  Neighbor capabilities:
  Dynamic capability: advertised (mp, refresh, gr) received (mp, refresh, gr)
  Dynamic capability (old): advertised received
  Route refresh capability (new): advertised received
  Route refresh capability (old): advertised received
  4-Byte AS capability: advertised received
  Address family IPv4 Unicast: advertised received
  Graceful Restart capability: advertised received

  Graceful Restart Parameters:
  Address families advertised to peer:
    IPv4 Unicast
  Address families received from peer:
    IPv4 Unicast
  Forwarding state preserved by peer for:
  Restart time advertised to peer: 120 seconds
  Stale time for routes advertised by peer: 300 seconds
  Restart time advertised by peer: 120 seconds
  Extended Next Hop Encoding Capability: advertised received
  Receive IPv6 next hop encoding Capability for AF:
    IPv4 Unicast  VPNv4 Unicast

  Message statistics:
                              Sent               Rcvd
  Opens:                         1                  1
  Notifications:                 0                  0
  Updates:                       8                  2
  Keepalives:                19616              19616
  Route Refresh:                 0                  0
  Capability:                    2                  2
  Total:                     19621              19621
  Total bytes:              373212             372851
  Bytes in queue:                0                  0

  For address family: IPv4 Unicast
  BGP table version 29, neighbor version 29
  5 accepted prefixes (5 paths), consuming 1440 bytes of memory
  0 received prefixes treated as withdrawn
  10 sent prefixes (10 paths)
  Last End-of-RIB received 00:00:05 after session start
  Last End-of-RIB sent 00:00:05 after session start
  First convergence 00:00:05 after session start with 4 routes sent

  Local host: 30.1.1.1, Local port: 42566
  Foreign host: 30.1.1.2, Foreign port: 179
  fd = 83
</code></pre>

Маршруты на LEAF1

Видно, что до подсетей удалённых хостов имеются равноценные маршруты через интерфейсы в сторону спайнов, то есть имеется ECMP (Equal-cost multi-path routing). Данного результата удалось достичь благодаря настройке maximum-paths 2 на лифах:

```
LEAF1# show bgp all
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 18, Local Router ID is 1.1.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>r1.1.1.1/32         0.0.0.0                  0        100      32768 ?
*>e2.2.2.2/32         10.1.1.1                                       0 65500 65502 ?
*|e                   10.1.2.1                                       0 65500 65502 ?
*>e3.3.3.3/32         10.1.1.1                                       0 65500 65503 ?
*|e                   10.1.2.1                                       0 65500 65503 ?
*>r10.1.1.0/30        0.0.0.0                  0        100      32768 ?
* e                   10.1.1.1                 0                     0 65500 ?
*>r10.1.2.0/30        0.0.0.0                  0        100      32768 ?
* e                   10.1.2.1                 0                     0 65500 ?
*>e10.10.10.10/32     10.1.1.1                 0                     0 65500 ?
*>e20.1.1.0/30        10.1.1.1                 0                     0 65500 ?
* e                   10.1.2.1                                       0 65500 65502 ?
* e20.1.2.0/30        10.1.1.1                                       0 65500 65502 ?
*>e                   10.1.2.1                 0                     0 65500 ?
*>e20.20.20.20/32     10.1.2.1                 0                     0 65500 ?
*>e30.1.1.0/30        10.1.1.1                 0                     0 65500 ?
* e                   10.1.2.1                                       0 65500 65503 ?
* e30.1.2.0/30        10.1.1.1                                       0 65500 65503 ?
*>e                   10.1.2.1                 0                     0 65500 ?
*>r192.168.1.0/24     0.0.0.0                  0        100      32768 ?
*>e192.168.2.0/24     10.1.1.1                                       0 65500 65502 ?
*|e                   10.1.2.1                                       0 65500 65502 ?
*>e192.168.3.0/24     10.1.1.1                                       0 65500 65503 ?
*|e                   10.1.2.1                                       0 65500 65503 ?
*>e192.168.4.0/24     10.1.1.1                                       0 65500 65503 ?
*|e                   10.1.2.1                                       0 65500 65503 ?

LEAF1# show ip route bgp-65501
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

2.2.2.2/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
3.3.3.3/32, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
10.10.10.10/32, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
20.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
20.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
20.20.20.20/32, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
30.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
30.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
192.168.2.0/24, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
192.168.3.0/24, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
192.168.4.0/24, ubest/mbest: 2/0
    *via 10.1.1.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
    *via 10.1.2.1, [20/0], 00:01:50, bgp-65501, external, tag 65500
```

Маршруты на SPINE1

```
SPINE1# show bgp all
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 29, Local Router ID is 10.10.10.10
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>e1.1.1.1/32         10.1.1.2                 0                     0 65501 ?
*>e2.2.2.2/32         20.1.1.2                 0                     0 65502 ?
*>e3.3.3.3/32         30.1.1.2                 0                     0 65503 ?
*>r10.1.1.0/30        0.0.0.0                  0        100      32768 ?
* e                   10.1.1.2                 0                     0 65501 ?
*>e10.1.2.0/30        10.1.1.2                 0                     0 65501 ?
*>r10.10.10.10/32     0.0.0.0                  0        100      32768 ?
* e20.1.1.0/30        20.1.1.2                 0                     0 65502 ?
*>r                   0.0.0.0                  0        100      32768 ?
*>e20.1.2.0/30        20.1.1.2                 0                     0 65502 ?
* e30.1.1.0/30        30.1.1.2                 0                     0 65503 ?
*>r                   0.0.0.0                  0        100      32768 ?
*>e30.1.2.0/30        30.1.1.2                 0                     0 65503 ?
*>e192.168.1.0/24     10.1.1.2                 0                     0 65501 ?
*>e192.168.2.0/24     20.1.1.2                 0                     0 65502 ?
*>e192.168.3.0/24     30.1.1.2                 0                     0 65503 ?
*>e192.168.4.0/24     30.1.1.2                 0                     0 65503 ?

SPINE1# show ip route bgp-65500
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 00:06:38, bgp-65500, external, tag 65501
2.2.2.2/32, ubest/mbest: 1/0
    *via 20.1.1.2, [20/0], 16:14:43, bgp-65500, external, tag 65502
3.3.3.3/32, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 16:14:36, bgp-65500, external, tag 65503
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 00:06:38, bgp-65500, external, tag 65501
20.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.2, [20/0], 16:14:43, bgp-65500, external, tag 65502
30.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 16:14:36, bgp-65500, external, tag 65503
192.168.1.0/24, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 00:06:38, bgp-65500, external, tag 65501
192.168.2.0/24, ubest/mbest: 1/0
    *via 20.1.1.2, [20/0], 16:14:43, bgp-65500, external, tag 65502
192.168.3.0/24, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 16:14:36, bgp-65500, external, tag 65503
192.168.4.0/24, ubest/mbest: 1/0
    *via 30.1.1.2, [20/0], 16:14:36, bgp-65500, external, tag 65503
```

Проверка связности между PC2 и PC1, PC3, PC4

```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 192.168.1.2/24
GATEWAY     : 192.168.1.1
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10558
RHOST:PORT  : 127.0.0.1:10559
MTU         : 1500

PC1> ping 192.168.2.2

84 bytes from 192.168.2.2 icmp_seq=1 ttl=61 time=13.076 ms
84 bytes from 192.168.2.2 icmp_seq=2 ttl=61 time=16.503 ms
84 bytes from 192.168.2.2 icmp_seq=3 ttl=61 time=18.584 ms
84 bytes from 192.168.2.2 icmp_seq=4 ttl=61 time=10.666 ms
84 bytes from 192.168.2.2 icmp_seq=5 ttl=61 time=10.937 ms

PC1> ping 192.168.3.2

84 bytes from 192.168.3.2 icmp_seq=1 ttl=61 time=15.824 ms
84 bytes from 192.168.3.2 icmp_seq=2 ttl=61 time=16.818 ms
84 bytes from 192.168.3.2 icmp_seq=3 ttl=61 time=15.963 ms
84 bytes from 192.168.3.2 icmp_seq=4 ttl=61 time=16.681 ms
84 bytes from 192.168.3.2 icmp_seq=5 ttl=61 time=14.616 ms

PC1> ping 192.168.4.2

84 bytes from 192.168.4.2 icmp_seq=1 ttl=61 time=16.882 ms
84 bytes from 192.168.4.2 icmp_seq=2 ttl=61 time=16.114 ms
84 bytes from 192.168.4.2 icmp_seq=3 ttl=61 time=17.152 ms
84 bytes from 192.168.4.2 icmp_seq=4 ttl=61 time=12.365 ms
84 bytes from 192.168.4.2 icmp_seq=5 ttl=61 time=13.597 ms
```

#### Выводы

Таким образом, нам удалось настроить Underlay с использованием eBGP и обеспечить этим связность между подсетями фабрики.

Спайны были помещены в одну AS 65500, так как это считается best practice вариантом для избежания неоптимальной маршрутизации при&#x20;
