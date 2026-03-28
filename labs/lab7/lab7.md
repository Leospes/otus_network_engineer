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

Обратите внимание, что дополнительно был создан Vlan 2 и соответствующий ему SVI интерфейс с адресацией из подсети 192.168.2.0/30 на обоих лифах. Данный линк нужен, чтобы правильно работала настройка peer-gateway в домене VPC, которая помогает обрабатывать пакет клиента, даже если они адресованы MAC адресу своего VPC пира (соседа).

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

В качестве конечного клиентского устройства, подключённого через VPC был использован виртуальный Cisco IOS коммутатор:&#x20;

```
interface Port-channel3
 description to_VPC
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no negotiation auto
 channel-group 3 mode active
!
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no negotiation auto
 channel-group 3 mode active

interface Vlan65
 ip address 11.11.11.2 255.255.255.0
 
ip route 0.0.0.0 0.0.0.0 11.11.11.1
```

#### Проверка

Состояние VPC

LEAF1

<pre><code><strong>LEAF1# show vpc
</strong>Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : secondary, operational primary
Number of vPCs configured         : 1
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Delay-restore Orphan-port status  : Timer is off.(timeout = 0s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     65

vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
3     Po3           up     success     success               65

<strong>LEAF1# show vpc peer-keepalive
</strong>
vPC keep-alive status             : peer is alive
--Peer is alive for             : (25051) seconds, (127) msec
--Send status                   : Success
--Last send at                  : 2026.03.28 18:19:51 965 ms
--Sent on interface             : mgmt0
--Receive status                : Success
--Last receive at               : 2026.03.28 18:19:51 966 ms
--Received on interface         : mgmt0
--Last update from peer         : (0) seconds, (814) msec

vPC Keep-alive parameters
--Destination                   : 192.168.1.2
--Keepalive interval            : 1000 msec
--Keepalive timeout             : 5 seconds
--Keepalive hold timeout        : 3 seconds
--Keepalive vrf                 : management
--Keepalive udp port            : 3200
--Keepalive tos                 : 192

<strong>LEAF1# show vpc statistics peer-keepalive
</strong>

vPC keep-alive statistics
----------------------------------------------------
peer-keepalive tx count:          256756
peer-keepalive rx count:          256260
average interval for peer rx:     166
Count of peer state changes:      1

<strong>LEAF1#  show vpc role
</strong>
vPC Role status
----------------------------------------------------
vPC role                        : secondary, operational primary
Dual Active Detection Status    : 0
vPC system-mac                  : 00:23:04:ee:be:01
vPC system-priority             : 32667
vPC local system-mac            : 0c:e9:00:00:1b:08
vPC local role-priority         : 32667
vPC local config role-priority  : 32667
vPC peer system-mac             : 0c:d2:00:00:1b:08
vPC peer role-priority          : 32667
vPC peer config role-priority   : 32667

<strong>LEAF1# show vpc statistics peer-link
</strong>port-channel1 is up
admin state is up,
  Hardware: Port-Channel, address: 0ce9.0000.0105 (bia 0ce9.0000.0105)
  Description: PEER-LINK
  MTU 9216 bytes, BW 2000000 Kbit , DLY 10 usec
  reliability 54994/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, medium is broadcast
  Port mode is trunk
  full-duplex, 1000 Mb/s
  Input flow-control is off, output flow-control is off
  Auto-mdix is turned off
  Switchport monitor is off
  EtherType is 0x8100
  Members in this channel: Eth1/5, Eth1/6
  Last clearing of "show interface" counters never
  3 interface resets
  Load-Interval #1: 30 seconds
    30 seconds input rate 1456 bits/sec, 2 packets/sec
    30 seconds output rate 3840 bits/sec, 2 packets/sec
    input rate 1.46 Kbps, 2 pps; output rate 3.84 Kbps, 2 pps
  Load-Interval #2: 5 minute (300 seconds)
    300 seconds input rate 1200 bits/sec, 1 packets/sec
    300 seconds output rate 2592 bits/sec, 1 packets/sec
    input rate 1.20 Kbps, 1 pps; output rate 2.59 Kbps, 1 pps
  RX
    919 unicast packets  8981933 multicast packets  12 broadcast packets
    8982864 input packets  701771398 bytes
    279172874241 jumbo packets  0 storm suppression packets
    0 runts  0 giants  0 CRC  0 no buffer
    1443712566365450552 input error  0 short frame  0 overrun   0 underrun  0 ignored
    0 watchdog  0 bad etype drop  0 bad proto drop  0 if down drop
    0 input with dribble  0 input discard
    0 Rx pause
    0 Stomped CRC
  TX
    959 unicast packets  4707854 multicast packets  6 broadcast packets
    4708819 output packets  347207222 bytes
    0 jumbo packets
    12764042493264397566 output error  0 collision  0 deferred  0 late collision
    0 lost carrier  0 no carrier  0 babble  0 output discard
    0 Tx pause

<strong>LEAF1# show vpc statistics vpc 3
</strong>port-channel3 is up
admin state is up,
 vPC Status: Up, vPC number: 3
  Hardware: Port-Channel, address: 0ce9.0000.0103 (bia 0ce9.0000.0103)
  Description: to_endhost-1
  MTU 1500 bytes, BW 1000000 Kbit , DLY 10 usec
  reliability 228/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, medium is broadcast
  Port mode is trunk
  full-duplex, 1000 Mb/s
  Input flow-control is off, output flow-control is off
  Auto-mdix is turned off
  Switchport monitor is off
  EtherType is 0x8100
  Members in this channel: Eth1/3
  Last clearing of "show interface" counters never
  3 interface resets
  Load-Interval #1: 30 seconds
    30 seconds input rate 936 bits/sec, 1 packets/sec
    30 seconds output rate 368 bits/sec, 0 packets/sec
    input rate 936 bps, 1 pps; output rate 368 bps, 0 pps
  Load-Interval #2: 5 minute (300 seconds)
    300 seconds input rate 48 bits/sec, 0 packets/sec
    300 seconds output rate 336 bits/sec, 0 packets/sec
    input rate 48 bps, 0 pps; output rate 336 bps, 0 pps
  RX
    57 unicast packets  5994 multicast packets  2 broadcast packets
    6053 input packets  672823 bytes
    927444127693810 jumbo packets  0 storm suppression packets
    0 runts  0 giants  0 CRC  0 no buffer
    1789807035106197506 input error  0 short frame  0 overrun   0 underrun  0 ignored
    0 watchdog  0 bad etype drop  0 bad proto drop  0 if down drop
    0 input with dribble  0 input discard
    0 Rx pause
    0 Stomped CRC
  TX
    682074694035215 unicast packets  4481390963 multicast packets  630204661625782276 broadcast packets
    630886740801208454 output packets  4682080349 bytes
    4294967296 jumbo packets
    302626440807972864 output error  0 collision  0 deferred  0 late collision
    0 lost carrier  0 no carrier  0 babble  0 output discard
    0 Tx pause
</code></pre>

LEAF2

<pre><code><strong>LEAF2# show vpc
</strong>Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary, operational secondary
Number of vPCs configured         : 1
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Delay-restore Orphan-port status  : Timer is off.(timeout = 0s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     2,65,500

vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
3     Po3           up     success     success               65

<strong>LEAF2# show vpc peer-keepalive
</strong>
vPC keep-alive status             : peer is alive
--Peer is alive for             : (31255) seconds, (797) msec
--Send status                   : Success
--Last send at                  : 2026.03.28 20:16:18 967 ms
--Sent on interface             : mgmt0
--Receive status                : Success
--Last receive at               : 2026.03.28 20:16:18 971 ms
--Received on interface         : mgmt0
--Last update from peer         : (0) seconds, (90) msec

vPC Keep-alive parameters
--Destination                   : 192.168.1.1
--Keepalive interval            : 1000 msec
--Keepalive timeout             : 5 seconds
--Keepalive hold timeout        : 3 seconds
--Keepalive vrf                 : management
--Keepalive udp port            : 3200
--Keepalive tos                 : 192

<strong>LEAF2# show vpc statistics peer-keepalive
</strong>

vPC keep-alive statistics
----------------------------------------------------
peer-keepalive tx count:          31339
peer-keepalive rx count:          31333
average interval for peer rx:     882
Count of peer state changes:      0
LEAF2# show vpc role

vPC Role status
----------------------------------------------------
vPC role                        : primary, operational secondary
Dual Active Detection Status    : 0
vPC system-mac                  : 00:23:04:ee:be:01
vPC system-priority             : 32667
vPC local system-mac            : 0c:d2:00:00:1b:08
vPC local role-priority         : 32667
vPC local config role-priority  : 32667
vPC peer system-mac             : 0c:e9:00:00:1b:08
vPC peer role-priority          : 32667
vPC peer config role-priority   : 32667

<strong>LEAF2# show vpc statistics peer-link
</strong>port-channel1 is up
admin state is up,
  Hardware: Port-Channel, address: 0cd2.0000.0105 (bia 0cd2.0000.0105)
  Description: PEER-LINK
  MTU 9216 bytes, BW 2000000 Kbit , DLY 10 usec
  reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, medium is broadcast
  Port mode is trunk
  full-duplex, 1000 Mb/s
  Input flow-control is off, output flow-control is off
  Auto-mdix is turned off
  Switchport monitor is off
  EtherType is 0x8100
  Members in this channel: Eth1/5, Eth1/6
  Last clearing of "show interface" counters never
  1 interface resets
  Load-Interval #1: 30 seconds
    30 seconds input rate 1064 bits/sec, 1 packets/sec
    30 seconds output rate 896 bits/sec, 1 packets/sec
    input rate 1.06 Kbps, 1 pps; output rate 896 bps, 1 pps
  Load-Interval #2: 5 minute (300 seconds)
    300 seconds input rate 2208 bits/sec, 1 packets/sec
    300 seconds output rate 1528 bits/sec, 1 packets/sec
    input rate 2.21 Kbps, 1 pps; output rate 1.53 Kbps, 1 pps
  RX
    4856 unicast packets  4188596 multicast packets  7 broadcast packets
    4193459 input packets  279376312 bytes
    0 jumbo packets  0 storm suppression packets
    0 runts  0 giants  0 CRC  0 no buffer
    0 input error  0 short frame  0 overrun   0 underrun  0 ignored
    0 watchdog  0 bad etype drop  0 bad proto drop  0 if down drop
    0 input with dribble  0 input discard
    0 Rx pause
    0 Stomped CRC
  TX
    3057 unicast packets  4188509 multicast packets  11 broadcast packets
    4191577 output packets  272206743 bytes
    0 jumbo packets
    0 output error  0 collision  0 deferred  0 late collision
    0 lost carrier  0 no carrier  0 babble  0 output discard
    0 Tx pause

<strong>LEAF2# show vpc statistics vpc 3
</strong>port-channel3 is up
admin state is up,
 vPC Status: Up, vPC number: 3
  Hardware: Port-Channel, address: 0cd2.0000.0103 (bia 0cd2.0000.0103)
  Description: to_endhost-1
  MTU 1500 bytes, BW 1000000 Kbit , DLY 10 usec
  reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, medium is broadcast
  Port mode is trunk
  full-duplex, 1000 Mb/s
  Input flow-control is off, output flow-control is off
  Auto-mdix is turned off
  Switchport monitor is off
  EtherType is 0x8100
  Members in this channel: Eth1/3
  Last clearing of "show interface" counters never
  1 interface resets
  Load-Interval #1: 30 seconds
    30 seconds input rate 208 bits/sec, 0 packets/sec
    30 seconds output rate 328 bits/sec, 0 packets/sec
    input rate 208 bps, 0 pps; output rate 328 bps, 0 pps
  Load-Interval #2: 5 minute (300 seconds)
    300 seconds input rate 120 bits/sec, 0 packets/sec
    300 seconds output rate 336 bits/sec, 0 packets/sec
    input rate 120 bps, 0 pps; output rate 336 bps, 0 pps
  RX
    755 unicast packets  2769 multicast packets  0 broadcast packets
    3524 input packets  542327 bytes
    0 jumbo packets  0 storm suppression packets
    0 runts  0 giants  0 CRC  0 no buffer
    0 input error  0 short frame  0 overrun   0 underrun  0 ignored
    0 watchdog  0 bad etype drop  0 bad proto drop  0 if down drop
    0 input with dribble  0 input discard
    0 Rx pause
    0 Stomped CRC
  TX
    43 unicast packets  55533 multicast packets  6 broadcast packets
    55582 output packets  3784711 bytes
    0 jumbo packets
    0 output error  0 collision  0 deferred  0 late collision
    0 lost carrier  0 no carrier  0 babble  0 output discard
    0 Tx pause
</code></pre>

EVPN маршруты

<pre><code><strong>LEAF1# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.1.1.1, local AS number 65501
BGP table version is 36, L2VPN EVPN config peers 2, capable peers 2
7 network entries and 9 paths using 2044 bytes of memory
BGP attribute entries [5/1800], BGP AS path entries [1/10]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4 65500      10651      10614       36    0    0 08:48:42 2
20.20.20.20     4 65500      10641      10609       36    0    0 08:48:48 2

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5
10.10.10.10     I 65500 2          2          0          0          0
20.20.20.20     I 65500 2          2          0          0          0

<strong>LEAF1# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 36, Local Router ID is 1.1.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.1.1.1:32832    (L2VNI 100065)
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                        100      32768 i
*>l[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                        100      32768 i

Route Distinguisher: 3.3.3.3:32834
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                                        0 65500 65503 i
* e                   3.3.3.3                                        0 65500 65503 i
* e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[33.33.33.2]/272
                      3.3.3.3                                        0 65500 65503 i
*>e                   3.3.3.3                                        0 65500 65503 i

Route Distinguisher: 65501:1    (L3VNI 100500)
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[33.33.33.2]/272
                      3.3.3.3                                        0 65500 65503 i

<strong>LEAF2# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 2.2.2.2, local AS number 65502
BGP table version is 73, L2VPN EVPN config peers 2, capable peers 2
7 network entries and 9 paths using 2044 bytes of memory
BGP attribute entries [5/1800], BGP AS path entries [1/10]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4 65500      10364      10323       73    0    0 02:15:12 2
20.20.20.20     4 65500      10382      10344       73    0    0 02:15:10 2

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5
10.10.10.10     I 65500 2          2          0          0          0
20.20.20.20     I 65500 2          2          0          0          0

<strong>LEAF2# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 73, Local Router ID is 2.2.2.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 2.2.2.2:32832    (L2VNI 100065)
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                        100      32768 i
*>l[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                        100      32768 i

Route Distinguisher: 3.3.3.3:32834
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                                        0 65500 65503 i
* e                   3.3.3.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[33.33.33.2]/272
                      3.3.3.3                                        0 65500 65503 i
* e                   3.3.3.3                                        0 65500 65503 i

Route Distinguisher: 65502:1    (L3VNI 100500)
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                                        0 65500 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[33.33.33.2]/272
                      3.3.3.3                                        0 65500 65503 i
                      
<strong>LEAF3# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 3.3.3.3, local AS number 65503
BGP table version is 319, L2VPN EVPN config peers 2, capable peers 2
8 network entries and 11 paths using 2432 bytes of memory
BGP attribute entries [7/2520], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.10     4 65500      88800      88748      319    0    0    3d01h 2
20.20.20.20     4 65500      88790      88739      319    0    0    3d01h 2

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5
10.10.10.10     I 65500 2          2          0          0          0
20.20.20.20     I 65500 2          2          0          0          0

<strong>LEAF3# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 319, Local Router ID is 3.3.3.3
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
                      10.1.1.100                                     0 65500 65502 i
*>e                   10.1.1.100                                     0 65500 65501 i

<strong>SPINE1# show bgp l2vpn evpn summary
</strong>BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.10.10.10, local AS number 65500
BGP table version is 376, L2VPN EVPN config peers 3, capable peers 3
11 network entries and 11 paths using 3212 bytes of memory
BGP attribute entries [9/3240], BGP AS path entries [3/18]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4 65501      12695      12686      376    0    0 10:32:23 3
2.2.2.2         4 65502      87500      87487      376    0    0 02:17:33 3
3.3.3.3         4 65503      88862      88729      376    0    0    3d01h 5

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5
1.1.1.1         I 65501 3          2          1          0          0
2.2.2.2         I 65502 3          2          1          0          0
3.3.3.3         I 65503 5          4          1          0          0

<strong>SPINE1# show bgp l2vpn evpn
</strong>BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 376, Local Router ID is 10.10.10.10
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, &#x26; - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.1.1.1:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65501 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                                     0 65501 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65501 i

Route Distinguisher: 2.2.2.2:32832
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[0]:[0.0.0.0]/216
                      10.1.1.100                                     0 65502 i
*>e[2]:[0]:[0]:[48]:[0c8c.4346.8041]:[32]:[11.11.11.2]/272
                      10.1.1.100                                     0 65502 i
*>e[3]:[0]:[32]:[10.1.1.100]/88
                      10.1.1.100                                     0 65502 i

Route Distinguisher: 3.3.3.3:32834
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[0]:[0.0.0.0]/216
                      3.3.3.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.248c.3e00]:[32]:[33.33.33.3]/272
                      3.3.3.3                                        0 65503 i
*>e[2]:[0]:[0]:[48]:[0242.ee8b.bc00]:[32]:[33.33.33.2]/272
                      3.3.3.3                                        0 65503 i
*>e[3]:[0]:[32]:[3.3.3.3]/88
                      3.3.3.3                                        0 65503 i
</code></pre>

VXLAN туннели

<pre><code><strong>LEAF1# show nve peers
</strong>Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      3.3.3.3                                 Up    CP        06:04:51 0cde.0000.1b08

<strong>LEAF2# show nve peers
</strong>Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      3.3.3.3                                 Up    CP        02:15:28 0cde.0000.1b08

<strong>LEAF3# show nve peers
</strong>Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      10.1.1.100                              Up    CP        08:54:04 0ce9.0000.1b08
</code></pre>

ipv4 BGP сессии

Стоит заметить, что для повышения отказоустойчивости была настроена дополнительная ipv4 BGP сессия между лифами на ранее созданном линке interface Vlan 2, которая поможет лифу, потерявшему аплинки до спайнов построить с ними новые сессии, обменяться маршрутами evpn и таким образом продолжить передавать пакеты.

<pre><code><strong>LEAF1# show ip bgp summary
</strong>BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 1.1.1.1, local AS number 65501
BGP table version is 189, IPv4 Unicast config peers 3, capable peers 3
13 network entries and 33 paths using 5716 bytes of memory
BGP attribute entries [7/2520], BGP AS path entries [6/56]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.1        4 65500      88994      88979      189    0    0    3d02h 8
10.1.2.1        4 65500      88993      88980      189    0    0    3d02h 8
<strong>192.168.2.2     4 65502       2478       2470      189    0    0 01:53:22 12
</strong>
<strong>LEAF2# show ip bgp summary
</strong>BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 2.2.2.2, local AS number 65502
BGP table version is 178, IPv4 Unicast config peers 3, capable peers 3
13 network entries and 37 paths using 6100 bytes of memory
BGP attribute entries [7/2520], BGP AS path entries [6/56]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
20.1.1.1        4 65500      10696      10690      178    0    0 02:29:11 10
20.1.2.1        4 65500      10698      10692      178    0    0 02:29:09 10
<strong>192.168.2.1     4 65501       2482       2472      178    0    0 01:53:26 12
</strong></code></pre>

#### Тесты отказоустойчивости

При проверке связности между хостами видно, что icmp пакеты распределяются в оба линка собранного VPC:

<figure><img src="../.gitbook/assets/Dump Lab_7_1.PNG" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Dump Lab_7_2.PNG" alt=""><figcaption></figcaption></figure>

Во время отключения аплинков на одном из peer LEAF, участвующем в VPC, связность между хостами endhost-3 и endhost-1 теряется на короткий промежуток времени или не теряется вовсе.

<pre><code>/ # ip a
1: lo: &#x3C;LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
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
	   
/ # ip route
default via 33.33.33.1 dev eth0
33.33.33.0/24 dev eth0 scope link  src 33.33.33.3

/ # ping 11.11.11.2
PING 11.11.11.2 (11.11.11.2) 56(84) bytes of data.
64 bytes from 11.11.11.2: icmp_seq=95 ttl=253 time=13.6 ms
64 bytes from 11.11.11.2: icmp_seq=96 ttl=253 time=14.9 ms
64 bytes from 11.11.11.2: icmp_seq=97 ttl=253 time=14.5 ms
....
64 bytes from 11.11.11.2: icmp_seq=554 ttl=253 time=18.4 ms
64 bytes from 11.11.11.2: icmp_seq=555 ttl=253 time=10.7 ms
64 bytes from 11.11.11.2: icmp_seq=556 ttl=253 time=14.4 ms
^C
--- 11.11.11.2 ping statistics ---
<strong>556 packets transmitted, 462 received, 16.9065% packet loss, time 557793ms
</strong>rtt min/avg/max/mdev = 8.674/14.731/57.056/3.686 ms
</code></pre>

На LEAF, который теряет аплинки происходит переустановление BGP сессий со спайнами через VPC peer-link, так как на нём у нас поднята ipv4 BGP сессия для получения маршрутов до loopback интерфейсов спайнов.

#### Выводы

Таким образом, нам удалось настроить VPC между LEAF1 и LEAF2 и проверить отказоустойчивость схемы.

Коммутаторы VPC домена используют secondary ip адрес на loopback интерфейсах для анонса маршрутной evpn информации.

Обмен маршрутами происходит успешно, связность между хостами имеется и сохраняется при отказе одного из плечей.
