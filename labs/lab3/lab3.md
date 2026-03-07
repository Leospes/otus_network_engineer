---
description: Построение Underlay сети (IS-IS)
---

# LAB3

#### Цели

* Настроить OSPF для Underlay сети
* Убедиться в наличии IP связанности между устройствами в OSFP домене

#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:

<figure><img src="../.gitbook/assets/Топология Lab_1.PNG" alt=""><figcaption></figcaption></figure>

#### Конфигурация устройств

LEAF1

```
feature isis

interface Ethernet1/1
  description to_SPINE1
  no switchport
  medium p2p
  ip address 10.1.1.2/30
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  medium p2p
  ip address 10.1.2.2/30
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/3
  description to_PC1
  no switchport
  ip address 192.168.1.1/24
  no shutdown
  
interface loopback1
  ip address 1.1.1.1/32
  ip router isis UNDERLAY
  isis passive-interface level-1-2

router isis UNDERLAY
  net 49.0001.0010.0100.1001.00
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain otus level-2
```

LEAF2

```
feature isis
  
interface Ethernet1/1
  description to_SPINE1
  no switchport
  medium p2p
  no ip redirects
  ip address 20.1.1.2/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  medium p2p
  no ip redirects
  ip address 20.1.2.2/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/3
  description to_PC2
  no switchport
  ip address 192.168.2.1/24
  no shutdown
  
interface loopback2
  ip address 2.2.2.2/32
  ip router isis UNDERLAY
  isis passive-interface level-1-2

router isis UNDERLAY
  net 49.0001.0020.0200.2002.00
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain otus level-2
```

LEAF3

```
feature isis

interface Ethernet1/1
  description to_SPINE1
  no switchport
  medium p2p
  no ip redirects
  ip address 30.1.1.2/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description to_SPINE2
  no switchport
  medium p2p
  no ip redirects
  ip address 30.1.2.2/30
  no ipv6 redirects
  ip router isis UNDERLAY
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
  ip router isis UNDERLAY
  isis passive-interface level-1-2

router isis UNDERLAY
  net 49.0001.0030.0300.3003.00
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain otus level-2
```

SPINE1

```
feature isis

interface Ethernet1/1
  description to_LEAF1
  no switchport
  medium p2p
  no ip redirects
  ip address 10.1.1.1/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  medium p2p
  no ip redirects
  ip address 20.1.1.1/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  medium p2p
  no ip redirects
  ip address 30.1.1.1/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown
  
interface loopback10
  ip address 10.10.10.10/32
  ip router isis UNDERLAY
  isis passive-interface level-1-2

router isis UNDERLAY
  net 49.0001.0100.1001.0010.00
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain otus level-2
```

SPINE2

```
feature isis

interface Ethernet1/1
  description to_LEAF1
  no switchport
  medium p2p
  no ip redirects
  ip address 10.1.2.1/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description to_LEAF2
  no switchport
  medium p2p
  no ip redirects
  ip address 20.1.2.1/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/3
  description to_LEAF3
  no switchport
  medium p2p
  no ip redirects
  ip address 30.1.2.1/30
  no ipv6 redirects
  ip router isis UNDERLAY
  no shutdown
  
interface loopback20
  ip address 20.20.20.20/32
  ip router isis UNDERLAY
  isis passive-interface level-1-2

router isis UNDERLAY
  net 49.0001.0200.2002.0020.00
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain otus level-2
```

#### Проверка

IS-IS соседства

<pre><code><strong>LEAF1# show isis adjacency detail
</strong>IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
SPINE1          N/A             1-2    UP     00:00:25   Ethernet1/1
  Area address: 49.0001
  Up/Down transitions: 3, Last transition: 12:49:28 ago
  Circuit Type: L1-2
  IPv4 Address: 10.1.1.1
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0

SPINE2          N/A             1-2    UP     00:00:24   Ethernet1/2
  Area address: 49.0001
  Up/Down transitions: 1, Last transition: 12:49:23 ago
  Circuit Type: L1-2
  IPv4 Address: 10.1.2.1
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0
  
<strong>LEAF2# show isis adjacency detail
</strong>IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
SPINE1          N/A             1-2    UP     00:00:31   Ethernet1/1
  Area address: 49.0001
  Up/Down transitions: 2, Last transition: 12:47:11 ago
  Circuit Type: L1-2
  IPv4 Address: 20.1.1.1
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0

SPINE2          N/A             1-2    UP     00:00:27   Ethernet1/2
  Area address: 49.0001
  Up/Down transitions: 2, Last transition: 12:47:16 ago
  Circuit Type: L1-2
  IPv4 Address: 20.1.2.1
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0
  
<strong>LEAF3# show isis adjacency detail
</strong>IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
SPINE1          N/A             1-2    UP     00:00:26   Ethernet1/1
  Area address: 49.0001
  Up/Down transitions: 3, Last transition: 12:49:36 ago
  Circuit Type: L1-2
  IPv4 Address: 30.1.1.1
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0

SPINE2          N/A             1-2    UP     00:00:24   Ethernet1/2
  Area address: 49.0001
  Up/Down transitions: 3, Last transition: 12:49:33 ago
  Circuit Type: L1-2
  IPv4 Address: 30.1.2.1
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0
  
<strong>SPINE1# show isis adjacency detail
</strong>IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
LEAF1           N/A             1-2    UP     00:00:27   Ethernet1/1
  Area address: 49.0001
  Up/Down transitions: 2, Last transition: 12:50:09 ago
  Circuit Type: L1-2
  IPv4 Address: 10.1.1.2
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0

LEAF2           N/A             1-2    UP     00:00:23   Ethernet1/2
  Area address: 49.0001
  Up/Down transitions: 1, Last transition: 12:49:56 ago
  Circuit Type: L1-2
  IPv4 Address: 20.1.1.2
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0

LEAF3           N/A             1-2    UP     00:00:28   Ethernet1/3
  Area address: 49.0001
  Up/Down transitions: 2, Last transition: 12:49:56 ago
  Circuit Type: L1-2
  IPv4 Address: 30.1.1.2
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0
  
<strong>SPINE2# show isis adjacency detail
</strong>IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
LEAF1           N/A             1-2    UP     00:00:26   Ethernet1/1
  Area address: 49.0001
  Up/Down transitions: 2, Last transition: 12:50:27 ago
  Circuit Type: L1-2
  IPv4 Address: 10.1.2.2
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0

LEAF2           N/A             1-2    UP     00:00:31   Ethernet1/2
  Area address: 49.0001
  Up/Down transitions: 3, Last transition: 12:50:24 ago
  Circuit Type: L1-2
  IPv4 Address: 20.1.2.2
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0

LEAF3           N/A             1-2    UP     00:00:28   Ethernet1/3
  Area address: 49.0001
  Up/Down transitions: 2, Last transition: 12:50:17 ago
  Circuit Type: L1-2
  IPv4 Address: 30.1.2.2
  IPv6 Address: 0::
  BFD session for IPv4 not requested
  BFD session for IPv6 not requested
   MT-0 AF IPv4: BFD OFF, topo_nlpid_bfd_required N, topo_nlpid_state Y
   MT-0: topo_bfd_required N, topo_usable Y
   isis_bfd_required N, isis_neighbor_usable Y
  Restart capable: 1; ack 0;
  Restart mode: 0; seen(ra 0; csnp(0; l1 0; l2 0)); suppress 0
</code></pre>

Маршруты на LEAF1

Видно, что до подсетей удалённых хостов имеются равноценные маршруты через интерфейсы в сторону спайнов, то есть имеется ECMP (Equal-cost multi-path routing). Дефолтная стоимость линка на nexus равна 40 для Ethernet портов и 1 для loopback:

<pre><code>LEAF1# show isis interface
IS-IS process: UNDERLAY VRF: default
loopback1, Interface status: protocol-up/link-up/admin-up
  IP address: 1.1.1.1, IP subnet: 1.1.1.1/32
  IPv6 routing is disabled
  Level1
    No auth type and keychain
    Auth check set
  Level2
    No auth type and keychain
    Auth check set
  Index: 0x0003, Local Circuit ID: 0x01, Circuit Type: L1-2
  Prefix suppress is disabled globally on interface loopback1
  Level 1
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  Level 2
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  BFD IPv4 is locally disabled for Interface loopback1
  BFD does not support AF IPv4
  BFD IPv6 is locally disabled for Interface loopback1
  BFD does not support AF IPv6
  MTR is disabled
  Level      Metric
  1               1
  2               1
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
<strong>    1  0        1       no   UP    UP       yes      DN       no
</strong><strong>    2  0        1       no   UP    UP       yes      DN       no
</strong>
Ethernet1/1, Interface status: protocol-up/link-up/admin-up
  IP address: 10.1.1.2, IP subnet: 10.1.1.0/30
  IPv6 routing is disabled
    No auth type and keychain
    Auth check set
  Index: 0x0001, Local Circuit ID: 0x01, Circuit Type: L1-2
  Prefix suppress is disabled globally on interface Ethernet1/1
  Level 1
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  Level 2
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  BFD IPv4 is locally disabled for Interface Ethernet1/1
  BFD IPv6 is locally disabled for Interface Ethernet1/1
  MTR is disabled
  Extended Local Circuit ID: 0x1A000000, P2P Circuit ID: 0000.0000.0000.00
  Retx interval: 5, Retx throttle interval: 66 ms
  LSP interval: 33 ms, MTU: 1500
  MTU check OFF on P2P interface
  P2P Adjs: 1, AdjsUp: 1, Priority 64
  Hello Interval: 10, Multi: 3, Next IIH: 00:00:01
  MT    Adjs   AdjsUp  Metric   CSNP  Next CSNP  Last LSP ID
  1          1        1      40     10  00:00:02   ffff.ffff.ffff.ff-ff
  2          1        1      40     10  00:00:05   ffff.ffff.ffff.ff-ff
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
<strong>    1  0        40      no   UP    UP       yes      DN       no
</strong><strong>    2  0        40      no   UP    UP       yes      DN       no
</strong>
Ethernet1/2, Interface status: protocol-up/link-up/admin-up
  IP address: 10.1.2.2, IP subnet: 10.1.2.0/30
  IPv6 routing is disabled
    No auth type and keychain
    Auth check set
  Index: 0x0002, Local Circuit ID: 0x01, Circuit Type: L1-2
  Prefix suppress is disabled globally on interface Ethernet1/2
  Level 1
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  Level 2
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  BFD IPv4 is locally disabled for Interface Ethernet1/2
  BFD IPv6 is locally disabled for Interface Ethernet1/2
  MTR is disabled
  Extended Local Circuit ID: 0x1A000200, P2P Circuit ID: 0000.0000.0000.00
  Retx interval: 5, Retx throttle interval: 66 ms
  LSP interval: 33 ms, MTU: 1500
  MTU check OFF on P2P interface
  P2P Adjs: 1, AdjsUp: 1, Priority 64
  Hello Interval: 10, Multi: 3, Next IIH: 00:00:07
  MT    Adjs   AdjsUp  Metric   CSNP  Next CSNP  Last LSP ID
  1          1        1      40     10  00:00:04   ffff.ffff.ffff.ff-ff
  2          1        1      40     10  00:00:04   ffff.ffff.ffff.ff-ff
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
<strong>    1  0        40      no   UP    UP       yes      DN       no
</strong><strong>    2  0        40      no   UP    UP       yes      DN       no
</strong>
Ethernet1/3, Interface status: protocol-up/link-up/admin-up
  IP address: 192.168.1.1, IP subnet: 192.168.1.0/24
  IPv6 routing is disabled
  Level1
    No auth type and keychain
    Auth check set
  Level2
    No auth type and keychain
    Auth check set
  Index: 0x0004, Local Circuit ID: 0x01, Circuit Type: L1-2
  Prefix suppress is disabled globally on interface Ethernet1/3
  Level 1
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  Level 2
    Address-family IPv4 Advertise Passive only is disabled
    Address-family IPv6 Advertise Passive only is disabled
  BFD IPv4 is locally disabled for Interface Ethernet1/3
  BFD IPv6 is locally disabled for Interface Ethernet1/3
  MTR is disabled
  Passive level: level-1-2
  LSP interval: 33 ms, MTU: 1500
  MTU check OFF on LAN interface level-1
  MTU check OFF on LAN interface level-2
  Level   Metric-0   Metric-2   CSNP  Next CSNP  Hello   Multi   Next IIH
  1              40      0     10 Inactive      10   3       Inactive
  2              40      0     10 Inactive      10   3       Inactive
  Level  Adjs   AdjsUp Pri  Circuit ID         Since
  1         0        0  64  0000.0000.0000.00  never
  2         0        0  64  0000.0000.0000.00  never
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
<strong>    1  0        40      no   UP    DN       yes      DN       no
</strong><strong>    2  0        40      no   UP    DN       yes      DN       no
</strong></code></pre>

<pre><code>LEAF1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%&#x3C;string>' in via output denotes VRF &#x3C;string>

1.1.1.1/32, ubest/mbest: 2/0, attached
    *via 1.1.1.1, Lo1, [0/0], 16:22:04, local
    *via 1.1.1.1, Lo1, [0/0], 16:22:04, direct
<strong>2.2.2.2/32, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [115/81], 13:09:17, isis-UNDERLAY, L1
</strong><strong>    *via 10.1.2.1, Eth1/2, [115/81], 13:09:21, isis-UNDERLAY, L1
</strong><strong>3.3.3.3/32, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [115/81], 13:09:17, isis-UNDERLAY, L1
</strong><strong>    *via 10.1.2.1, Eth1/2, [115/81], 13:09:13, isis-UNDERLAY, L1
</strong>10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 15:55:49, direct
10.1.1.2/32, ubest/mbest: 1/0, attached
    *via 10.1.1.2, Eth1/1, [0/0], 15:55:49, local
10.1.2.0/30, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 15:55:13, direct
10.1.2.2/32, ubest/mbest: 1/0, attached
    *via 10.1.2.2, Eth1/2, [0/0], 15:55:13, local
10.10.10.10/32, ubest/mbest: 1/0
    *via 10.1.1.1, Eth1/1, [115/41], 13:09:30, isis-UNDERLAY, L1
20.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, Eth1/1, [115/80], 13:09:30, isis-UNDERLAY, L1
20.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, Eth1/2, [115/80], 13:09:24, isis-UNDERLAY, L1
20.20.20.20/32, ubest/mbest: 1/0
    *via 10.1.2.1, Eth1/2, [115/41], 13:09:24, isis-UNDERLAY, L1
30.1.1.0/30, ubest/mbest: 1/0
    *via 10.1.1.1, Eth1/1, [115/80], 13:09:30, isis-UNDERLAY, L1
30.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.2.1, Eth1/2, [115/80], 13:09:24, isis-UNDERLAY, L1
192.168.1.0/24, ubest/mbest: 1/0, attached
    *via 192.168.1.1, Eth1/3, [0/0], 16:20:35, direct
192.168.1.1/32, ubest/mbest: 1/0, attached
    *via 192.168.1.1, Eth1/3, [0/0], 16:20:35, local
<strong>192.168.2.0/24, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [115/120], 00:05:52, isis-UNDERLAY, L1
</strong><strong>    *via 10.1.2.1, Eth1/2, [115/120], 00:05:52, isis-UNDERLAY, L1
</strong><strong>192.168.3.0/24, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [115/120], 00:05:46, isis-UNDERLAY, L1
</strong><strong>    *via 10.1.2.1, Eth1/2, [115/120], 00:05:46, isis-UNDERLAY, L1
</strong><strong>192.168.4.0/24, ubest/mbest: 2/0
</strong><strong>    *via 10.1.1.1, Eth1/1, [115/120], 00:05:28, isis-UNDERLAY, L1
</strong><strong>    *via 10.1.2.1, Eth1/2, [115/120], 00:05:28, isis-UNDERLAY, L1
</strong>
LEAF1# show isis route
IS-IS process: UNDERLAY VRF: default
IS-IS IPv4 routing table

1.1.1.1/32, L1, direct
   *via loopback1, metric 1, L1, direct
    via loopback1, metric 1, L2, direct
<strong>2.2.2.2/32, L1
</strong><strong>   *via 10.1.1.1, Ethernet1/1, metric 81, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>   *via 10.1.2.1, Ethernet1/2, metric 81, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>3.3.3.3/32, L1
</strong><strong>   *via 10.1.1.1, Ethernet1/1, metric 81, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>   *via 10.1.2.1, Ethernet1/2, metric 81, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong>10.1.1.0/30, L1, direct
   *via Ethernet1/1, metric 40, L1, direct
    via Ethernet1/1, metric 40, L2, direct
10.1.2.0/30, L1, direct
   *via Ethernet1/2, metric 40, L1, direct
    via Ethernet1/2, metric 40, L2, direct
10.10.10.10/32, L1
   *via 10.1.1.1, Ethernet1/1, metric 41, L1 (I,U), table-map-deny no-pol, admin-dist 115
20.1.1.0/30, L1
   *via 10.1.1.1, Ethernet1/1, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
20.1.2.0/30, L1
   *via 10.1.2.1, Ethernet1/2, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
20.20.20.20/32, L1
   *via 10.1.2.1, Ethernet1/2, metric 41, L1 (I,U), table-map-deny no-pol, admin-dist 115
30.1.1.0/30, L1
   *via 10.1.1.1, Ethernet1/1, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
30.1.2.0/30, L1
   *via 10.1.2.1, Ethernet1/2, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
192.168.1.0/24, L1, direct
   *via Ethernet1/3, metric 40, L1, direct
    via Ethernet1/3, metric 40, L2, direct
<strong>192.168.2.0/24, L1
</strong><strong>   *via 10.1.1.1, Ethernet1/1, metric 120, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>   *via 10.1.2.1, Ethernet1/2, metric 120, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>192.168.3.0/24, L1
</strong><strong>   *via 10.1.1.1, Ethernet1/1, metric 120, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>   *via 10.1.2.1, Ethernet1/2, metric 120, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>192.168.4.0/24, L1
</strong><strong>   *via 10.1.1.1, Ethernet1/1, metric 120, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>   *via 10.1.2.1, Ethernet1/2, metric 120, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong></code></pre>

Маршруты на SPINE1

<pre><code>SPINE1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%&#x3C;string>' in via output denotes VRF &#x3C;string>

1.1.1.1/32, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [115/41], 13:11:10, isis-UNDERLAY, L1
2.2.2.2/32, ubest/mbest: 1/0
    *via 20.1.1.2, Eth1/2, [115/41], 13:10:56, isis-UNDERLAY, L1
3.3.3.3/32, ubest/mbest: 1/0
    *via 30.1.1.2, Eth1/3, [115/41], 13:10:56, isis-UNDERLAY, L1
10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 15:38:31, direct
10.1.1.1/32, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 15:38:31, local
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [115/80], 13:11:10, isis-UNDERLAY, L1
10.10.10.10/32, ubest/mbest: 2/0, attached
    *via 10.10.10.10, Lo10, [0/0], 16:23:45, local
    *via 10.10.10.10, Lo10, [0/0], 16:23:45, direct
20.1.1.0/30, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 15:38:30, direct
20.1.1.1/32, ubest/mbest: 1/0, attached
    *via 20.1.1.1, Eth1/2, [0/0], 15:38:30, local
20.1.2.0/30, ubest/mbest: 1/0
    *via 20.1.1.2, Eth1/2, [115/80], 13:10:56, isis-UNDERLAY, L1
<strong>20.20.20.20/32, ubest/mbest: 3/0
</strong><strong>    *via 10.1.1.2, Eth1/1, [115/81], 13:11:04, isis-UNDERLAY, L1
</strong><strong>    *via 20.1.1.2, Eth1/2, [115/81], 13:10:56, isis-UNDERLAY, L1
</strong><strong>    *via 30.1.1.2, Eth1/3, [115/81], 13:10:51, isis-UNDERLAY, L1
</strong>30.1.1.0/30, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 15:38:30, direct
30.1.1.1/32, ubest/mbest: 1/0, attached
    *via 30.1.1.1, Eth1/3, [0/0], 15:38:30, local
30.1.2.0/30, ubest/mbest: 1/0
    *via 30.1.1.2, Eth1/3, [115/80], 13:10:56, isis-UNDERLAY, L1
192.168.1.0/24, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [115/80], 00:07:40, isis-UNDERLAY, L1
192.168.2.0/24, ubest/mbest: 1/0
    *via 20.1.1.2, Eth1/2, [115/80], 00:07:32, isis-UNDERLAY, L1
192.168.3.0/24, ubest/mbest: 1/0
    *via 30.1.1.2, Eth1/3, [115/80], 00:07:27, isis-UNDERLAY, L1
192.168.4.0/24, ubest/mbest: 1/0
    *via 30.1.1.2, Eth1/3, [115/80], 00:07:08, isis-UNDERLAY, L1

SPINE1# show isis route
IS-IS process: UNDERLAY VRF: default
IS-IS IPv4 routing table

1.1.1.1/32, L1
   *via 10.1.1.2, Ethernet1/1, metric 41, L1 (I,U), table-map-deny no-pol, admin-dist 115
2.2.2.2/32, L1
   *via 20.1.1.2, Ethernet1/2, metric 41, L1 (I,U), table-map-deny no-pol, admin-dist 115
3.3.3.3/32, L1
   *via 30.1.1.2, Ethernet1/3, metric 41, L1 (I,U), table-map-deny no-pol, admin-dist 115
10.1.1.0/30, L1, direct
   *via Ethernet1/1, metric 40, L1, direct
    via Ethernet1/1, metric 40, L2, direct
10.1.2.0/30, L1
   *via 10.1.1.2, Ethernet1/1, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
10.10.10.10/32, L1, direct
   *via loopback10, metric 1, L1, direct
    via loopback10, metric 1, L2, direct
20.1.1.0/30, L1, direct
   *via Ethernet1/2, metric 40, L1, direct
    via Ethernet1/2, metric 40, L2, direct
20.1.2.0/30, L1
   *via 20.1.1.2, Ethernet1/2, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
<strong>20.20.20.20/32, L1
</strong><strong>   *via 10.1.1.2, Ethernet1/1, metric 81, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>   *via 20.1.1.2, Ethernet1/2, metric 81, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong><strong>   *via 30.1.1.2, Ethernet1/3, metric 81, L1 (I,U), table-map-deny no-pol, admin-dist 115
</strong>30.1.1.0/30, L1, direct
   *via Ethernet1/3, metric 40, L1, direct
    via Ethernet1/3, metric 40, L2, direct
30.1.2.0/30, L1
   *via 30.1.1.2, Ethernet1/3, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
192.168.1.0/24, L1
   *via 10.1.1.2, Ethernet1/1, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
192.168.2.0/24, L1
   *via 20.1.1.2, Ethernet1/2, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
192.168.3.0/24, L1
   *via 30.1.1.2, Ethernet1/3, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
192.168.4.0/24, L1
   *via 30.1.1.2, Ethernet1/3, metric 80, L1 (I,U), table-map-deny no-pol, admin-dist 115
</code></pre>

Проверка связности между PC2 и PC1, PC3, PC4

```
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 192.168.2.2/24
GATEWAY     : 192.168.2.1
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 10250
RHOST:PORT  : 127.0.0.1:10251
MTU         : 1500

PC2> ping 192.168.1.2

192.168.1.2 icmp_seq=1 timeout
84 bytes from 192.168.1.2 icmp_seq=2 ttl=61 time=15.338 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=61 time=15.751 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=61 time=14.827 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=61 time=14.971 ms

PC2> ping 192.168.3.2

192.168.3.2 icmp_seq=1 timeout
84 bytes from 192.168.3.2 icmp_seq=2 ttl=61 time=15.845 ms
84 bytes from 192.168.3.2 icmp_seq=3 ttl=61 time=14.636 ms
84 bytes from 192.168.3.2 icmp_seq=4 ttl=61 time=14.872 ms
84 bytes from 192.168.3.2 icmp_seq=5 ttl=61 time=14.120 ms

PC2> ping 192.168.4.2

192.168.4.2 icmp_seq=1 timeout
84 bytes from 192.168.4.2 icmp_seq=2 ttl=61 time=16.650 ms
84 bytes from 192.168.4.2 icmp_seq=3 ttl=61 time=15.284 ms
84 bytes from 192.168.4.2 icmp_seq=4 ttl=61 time=15.247 ms
84 bytes from 192.168.4.2 icmp_seq=5 ttl=61 time=15.307 ms
```

#### Выводы

Таким образом, нам удалось настроить единый домен IS-IS с номером area 0001 на уровнях L1-L2.

Соседства успешно поднялись и маршрутная информация получена всеми устройствами.

Связность между узлами сети имеется.
