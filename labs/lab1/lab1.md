# LAB1

## Проектирование адресного пространства

#### Цели

* Собрать топологию CLOS на лабораторном стенде GNS3 с использованием образов Nexus 9000/9300 Series.
* Распределить адресное пространство для Underlay сети
* Настроить адресацию на сетевых узлах

#### Реализованная топология CLOS

<figure><img src="../.gitbook/assets/Топология Lab_1.PNG" alt=""><figcaption></figcaption></figure>

#### Таблица адресации

| Device | Interface | IP Address  | Subnet Mask     |
| ------ | --------- | ----------- | --------------- |
| SPINE1 | lo10      | 10.10.10.10 | 255.255.255.255 |
| SPINE1 | e1/1      | 10.1.1.1    | 255.255.255.252 |
| SPINE1 | e1/2      | 20.1.1.1    | 255.255.255.252 |
| SPINE1 | e1/3      | 30.1.1.1    | 255.255.255.252 |
| SPINE2 | lo20      | 20.20.20.20 | 255.255.255.255 |
| SPINE2 | e1/1      | 10.1.2.1    | 255.255.255.252 |
| SPINE2 | e1/2      | 20.1.2.1    | 255.255.255.252 |
| SPINE2 | e1/3      | 30.1.2.1    | 255.255.255.252 |
| LEAF1  | lo1       | 1.1.1.1     | 255.255.255.255 |
| LEAF1  | e1/1      | 10.1.1.2    | 255.255.255.252 |
| LEAF1  | e1/2      | 10.1.2.2    | 255.255.255.252 |
| LEAF1  | e1/3      | 192.168.1.1 | 255.255.255.0   |
| LEAF2  | lo2       | 2.2.2.2     | 255.255.255.255 |
| LEAF2  | e1/1      | 20.1.1.2    | 255.255.255.252 |
| LEAF2  | e1/2      | 20.1.2.2    | 255.255.255.252 |
| LEAF2  | e1/3      | 192.168.2.1 | 255.255.255.0   |
| LEAF3  | lo3       | 3.3.3.3     | 255.255.255.255 |
| LEAF3  | e1/1      | 30.1.1.2    | 255.255.255.252 |
| LEAF3  | e1/2      | 30.1.2.2    | 255.255.255.252 |
| LEAF3  | e1/3      | 192.168.3.1 | 255.255.255.0   |
| LEAF3  | e1/4      | 192.168.4.1 | 255.255.255.0   |
| PC1    | e0        | 192.168.1.2 | 255.255.255.0   |
| PC2    | e0        | 192.168.2.2 | 255.255.255.0   |
| PC3    | e0        | 192.168.3.2 | 255.255.255.0   |
| PC4    | e0        | 192.168.4.2 | 255.255.255.0   |

#### Конфигурация устройств

LEAF1

```
interface loopback1
  ip address 1.1.1.1/32

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 10.1.1.2/30

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 10.1.2.2/30

interface Ethernet1/3
  description to_PC1
  no switchport
  ip address 192.168.1.1/24
  no shutdown
```

LEAF2

```
interface loopback2
  ip address 2.2.2.2/32

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 20.1.1.2/30

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 20.1.2.2/30

interface Ethernet1/3
  description to_PC2
  no switchport
  ip address 192.168.2.1/24
```

LEAF3

```
interface loopback3
  ip address 3.3.3.3/32

interface Ethernet1/1
  description to_SPINE1
  no switchport
  ip address 30.1.1.2/30

interface Ethernet1/2
  description to_SPINE2
  no switchport
  ip address 30.1.2.2/30

interface Ethernet1/3
  description to_PC3
  no switchport
  ip address 192.168.3.1/24

interface Ethernet1/4
  description to_PC4
  no switchport
  ip address 192.168.4.1/24
```

SPINE1

```
interface loopback10
  ip address 10.10.10.10/32

interface Ethernet1/1
  description to_LEAF1
  no switchport
  ip address 10.1.1.1/30

interface Ethernet1/2
  description to_LEAF2
  no switchport
  ip address 20.1.1.1/30

interface Ethernet1/3
  description to_LEAF3
  no switchport
  ip address 30.1.1.1/30
```

SPINE2

```
interface loopback20
  ip address 20.20.20.20/32

interface Ethernet1/1
  description to_LEAF1
  no switchport
  ip address 10.1.2.1/30

interface Ethernet1/2
  description to_LEAF2
  no switchport
  ip address 20.1.2.1/30

interface Ethernet1/3
  description to_LEAF3
  no switchport
  ip address 30.1.2.1/30
```

PC1

```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 192.168.1.2/24
GATEWAY     : 192.168.1.1
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10120
RHOST:PORT  : 127.0.0.1:10121
MTU         : 1500
```

PC2

```
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 0.0.0.0/0
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 10122
RHOST:PORT  : 127.0.0.1:10123
MTU         : 1500
```

PC3

```
PC3> show ip

NAME        : PC3[1]
IP/MASK     : 192.168.3.2/24
GATEWAY     : 192.168.3.1
DNS         :
MAC         : 00:50:79:66:68:02
LPORT       : 10124
RHOST:PORT  : 127.0.0.1:10125
MTU         : 1500
```

PC4

```
PC4> show ip

NAME        : PC4[1]
IP/MASK     : 192.168.4.2/24
GATEWAY     : 192.168.4.1
DNS         :
MAC         : 00:50:79:66:68:03
LPORT       : 10126
RHOST:PORT  : 127.0.0.1:10127
MTU         : 1500
```
