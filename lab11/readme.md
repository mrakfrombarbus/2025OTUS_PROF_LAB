# VPN. GRE. DmVPN

## Цель:
- Настроить GRE между офисами Москва и С.-Петербург
- Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги.


## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

- Настроите GRE между офисами Москва и С.-Петербург.
- Настроите DMVMN между Москва и Чокурдах, Лабытнанги.
- Все узлы в офисах в лабораторной работе должны иметь IP связность.
- План работы и изменения зафиксированы в документации.

## Топология

![](Topology9.png)


## Настроите GRE между офисами Москва и С.-Петербург.

R15
```
interface Tunnel0
 ip address 10.15.18.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel destination 111.111.111.2

```

R18
```
!
interface Tunnel0
 ip address 10.15.18.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel destination 200.0.0.2

```
Проверка
```
R18(config-if)#do ping 10.15.18.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R18(config-if)#do ping 10.15.18.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 14/15/16 ms

```

## Настроите DMVMN между Москва и Чокурдах, Лабытнанги.
R15 HUB

```
!
interface Tunnel13
 ip address 10.66.6.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map multicast dynamic
 ip nhrp network-id 13
 ip nhrp registration no-unique
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel mode gre multipoint

```
R27 Spoke
```
!
interface Tunnel13
 ip address 10.66.6.2 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map 10.66.6.1 200.0.0.2
 ip nhrp map multicast 200.0.0.2
 ip nhrp network-id 13
 ip nhrp nhs 10.66.6.1
 ip nhrp registration no-unique
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```
R28 Spoke
```
!
interface Tunnel13
 ip address 10.66.6.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map 10.66.6.1 200.0.0.2
 ip nhrp map multicast 200.0.0.2
 ip nhrp network-id 13
 ip nhrp nhs 10.66.6.1
 ip nhrp registration no-unique
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
!

```
Проверяем
```
R15#sh dm
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel13, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 111.1.1.2             10.66.6.2    UP 00:01:15     D
     1 16.16.16.2            10.66.6.3    UP 00:01:04     D

R15#
```
А теперь поднимаем EIGRP и доводим все до второй фазы
```
R15#sh run | s eigrp
router eigrp 100
 network 10.66.6.0 0.0.0.255
 network 192.168.10.0
 network 192.168.20.0
R15#


R27#sh run | s eigrp
router eigrp 100
 network 1.1.1.27 0.0.0.0
 network 10.66.6.0 0.0.0.255
R27#


R28#sh run | s eigrp
router eigrp 100
 network 1.1.1.28 0.0.0.0
 network 10.66.6.0 0.0.0.255
 network 192.168.50.0
 network 192.168.60.0
R28#
```
Проверяем маршруты и доступность

```
R15#sh ip eigrp neighbors
EIGRP-IPv4 Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.66.6.3               Tu13                     14 00:00:57   42  1398  0  17
0   10.66.6.2               Tu13                     14 00:00:57   24  1398  0  15
R15#

R15#sh ip route eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 200.0.0.1 to network 0.0.0.0

      1.0.0.0/32 is subnetted, 9 subnets
D        1.1.1.27 [90/27008000] via 10.66.6.2, 00:03:35, Tunnel13
D        1.1.1.28 [90/27008000] via 10.66.6.3, 00:03:35, Tunnel13
D     192.168.50.0/24 [90/26905600] via 10.66.6.3, 00:03:35, Tunnel13
D     192.168.60.0/24 [90/26905600] via 10.66.6.3, 00:03:35, Tunnel13
R15#
R15#ping 1.1.1.27
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.27, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
R15#ping 1.1.1.28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms
R15#trace 1.1.1.27
Type escape sequence to abort.
Tracing the route to 1.1.1.27
VRF info: (vrf in name/id, vrf out name/id)
  1 10.66.6.2 [AS 301] 5 msec 14 msec *
R15#



R27#ping 1.1.1.28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R27#


R28#
R28#ping 1.1.1.27
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.27, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/12 ms
R28#

```
