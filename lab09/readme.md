# BGP для маршрутизации IPv6 unicast

## Цель:
- Настроить eBGP/iBGP IPv6 unicast для всех сегментов сети по аналогичной логике с настройкой eBGP/iBGP IPv4 unicast.

## Описание/Пошаговая инструкция выполнения домашнего задания
- Настроите eBGP IPv6 unicast между офисом Москва и двумя провайдерами - Киторн и Ламас.
- Настроите eBGP IPv6 unicast между провайдерами Киторн и Ламас.
- Настроите eBGP IPv6 unicast между Ламас и Триада.
- Настроите eBGP IPv6 unicast между офисом С.-Петербург и провайдером Триада.
- Организуете IPv6 unicast связность между пограничными роутерами офисов Москва и С.-Петербург.
- Настроите iBGP IPv6 unicast в офисе Москва между маршрутизаторами R14 и R15.
- Настроите iBGP IPv6 unicast в провайдере Триада, с использованием RR.
- Все сети IPv6 в лабораторной работе должны иметь связность между собой.

## Топология
![](Topology9.png)

Для LLA будем использовать адресацию по принципу fe80::DeviceNumber+InterfaceNumber. Так, для R22 интерфейс e0/1 - fe80::221   
ДЛя лупбэков - fc00::DevN/128    
Для GUA - 2000:AS1:AS2::IPv4. Пример - 2000:101:301::11:11:11:1/64      
LLA для iBGP, GUA для eBGP
## Настроите eBGP IPv6 unicast между офисом Москва и двумя провайдерами - Киторн и Ламас.
<details>
<summary>R14</summary>
<pre><code>
R14#sh run | sec int
mmi polling-interval 60
interface Loopback0
 ip address 1.1.1.14 255.255.255.255
 ipv6 address FC00::14/128
interface Ethernet0/0
 ip address 10.10.10.10 255.255.255.252
 ip ospf 1 area 0
interface Ethernet0/1
 ip address 10.10.10.30 255.255.255.252
 ip ospf 1 area 0
interface Ethernet0/2
 ip address 100.0.0.2 255.255.255.252
 ipv6 address 2000:101:1001:0:100::2/64
interface Ethernet0/3
 ip address 10.10.10.33 255.255.255.252
 ip ospf 1 area 101

R14#sh run | sec bgp
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 neighbor 1.1.1.15 remote-as 1001
 neighbor 1.1.1.15 update-source Loopback0
 neighbor 2000:101:1001:0:100::1 remote-as 101
 neighbor 100.0.0.1 remote-as 101
 !
 address-family ipv4
  network 1.1.1.14 mask 255.255.255.255
  redistribute ospf 1
  neighbor 1.1.1.15 activate
  neighbor 1.1.1.15 soft-reconfiguration inbound
  no neighbor 2000:101:1001:0:100::1 activate
  neighbor 100.0.0.1 activate
  neighbor 100.0.0.1 soft-reconfiguration inbound
  neighbor 100.0.0.1 route-map R14_AS_PREPEND out
  neighbor 100.0.0.1 filter-list 1 out
 exit-address-family
 !
 address-family ipv6
  network FC00::14/128
  neighbor 2000:101:1001:0:100::1 activate
 exit-address-family
R14#



</code></pre>
</details>
<details>
<summary>R15</summary>
<pre><code>
R15#sh run | sec int
mmi polling-interval 60
interface Loopback0
 ip address 1.1.1.15 255.255.255.255
 ipv6 address FC00::15/128
interface Ethernet0/0
 ip address 10.10.10.26 255.255.255.252
 ip ospf 1 area 0
interface Ethernet0/1
 ip address 10.10.10.14 255.255.255.252
 ip ospf 1 area 0
interface Ethernet0/2
 ip address 200.0.0.2 255.255.255.252
 ipv6 address FE80::152 link-local
 ipv6 address 2000:301:1001:0:200::2/64
interface Ethernet0/3
 ip address 10.10.10.37 255.255.255.252
 ip ospf 1 area 102
R15#sh run | sec bgp
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 neighbor 1.1.1.14 remote-as 1001
 neighbor 1.1.1.14 update-source Loopback0
 neighbor 2000:301:1001:0:200::1 remote-as 301
 neighbor 200.0.0.1 remote-as 301
 !
 address-family ipv4
  network 1.1.1.15 mask 255.255.255.255
  redistribute ospf 1
  neighbor 1.1.1.14 activate
  no neighbor 2000:301:1001:0:200::1 activate
  neighbor 200.0.0.1 activate
  neighbor 200.0.0.1 soft-reconfiguration inbound
  neighbor 200.0.0.1 route-map R15_LOC_PREF in
  neighbor 200.0.0.1 filter-list 1 out
 exit-address-family
 !
 address-family ipv6
  network FC00::15/128
  neighbor 2000:301:1001:0:200::1 activate
 exit-address-family


</code></pre>
</details>
<details>
<summary>R21</summary>
<pre><code>
R21#sh run | sec int
mmi polling-interval 60
interface Loopback0
 ip address 1.1.1.21 255.255.255.255
 ipv6 address FC00::21/128
interface Ethernet0/0
 ip address 200.0.0.1 255.255.255.252
 ipv6 address 2000:301:1001:0:200::1/64
interface Ethernet0/1
 ip address 11.11.11.2 255.255.255.252
interface Ethernet0/2
 ip address 13.13.13.1 255.255.255.252
R21#sh run | sec bgp
router bgp 301
 bgp router-id 21.21.21.21
 bgp log-neighbor-changes
 neighbor 11.11.11.1 remote-as 101
 neighbor 13.13.13.2 remote-as 520
 neighbor 2000:301:1001:0:200::2 remote-as 1001
 neighbor 200.0.0.2 remote-as 1001
 !
 address-family ipv4
  network 1.1.1.21 mask 255.255.255.255
  neighbor 11.11.11.1 activate
  neighbor 13.13.13.2 activate
  no neighbor 2000:301:1001:0:200::2 activate
  neighbor 200.0.0.2 activate
  neighbor 200.0.0.2 default-originate
  neighbor 200.0.0.2 soft-reconfiguration inbound
  neighbor 200.0.0.2 route-map SPBTOMSK out
 exit-address-family
 !
 address-family ipv6
  network FC00::21/128
  neighbor 2000:301:1001:0:200::2 activate
 exit-address-family
R21#


</code></pre>
</details>
<details>
<summary>R22</summary>
<pre><code>

R22#sh run | sec int
mmi polling-interval 60
interface Loopback0
 ip address 1.1.1.22 255.255.255.255
 ipv6 address FC00::22/128
interface Ethernet0/0
 ip address 100.0.0.1 255.255.255.252
 ipv6 address 2000:101:1001:0:100::1/64
interface Ethernet0/1
 ip address 11.11.11.1 255.255.255.252
interface Ethernet0/2
 ip address 12.12.12.1 255.255.255.252
R22#sh run | sec bgp
router bgp 101
 bgp router-id 22.22.22.22
 bgp log-neighbor-changes
 neighbor 11.11.11.2 remote-as 301
 neighbor 12.12.12.2 remote-as 520
 neighbor 2000:101:1001:0:100::2 remote-as 1001
 neighbor 100.0.0.2 remote-as 1001
 !
 address-family ipv4
  network 1.1.1.22 mask 255.255.255.255
  neighbor 11.11.11.2 activate
  neighbor 12.12.12.2 activate
  no neighbor 2000:101:1001:0:100::2 activate
  neighbor 100.0.0.2 activate
  neighbor 100.0.0.2 default-originate
  neighbor 100.0.0.2 soft-reconfiguration inbound
  neighbor 100.0.0.2 route-map REJECTALL out
 exit-address-family
 !
 address-family ipv6
  network FC00::22/128
  neighbor 2000:101:1001:0:100::2 activate
 exit-address-family
R22#

</code></pre>
</details>

Результат:
R14
```
R14#show ip bgp ipv6 unicast
BGP table version is 3, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::14/128     ::                       0         32768 i
 *>  FC00::22/128     2000:101:1001:0:100::1
                                                0             0 101 i
R14#sh ipv6 route bgp
IPv6 Routing Table - default - 5 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   FC00::22/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2


```
R15
```
R15#show ip bgp ipv6 unicast
BGP table version is 3, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::15/128     ::                       0         32768 i
 *>  FC00::21/128     2000:301:1001:0:200::1
                                                0             0 301 i
R15#sh ipv6 route bgp
IPv6 Routing Table - default - 5 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   FC00::21/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2

R15#

```
R21
```
R21#show ip bgp ipv6 unicast
BGP table version is 3, local router ID is 21.21.21.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::15/128     2000:301:1001:0:200::2
                                                0             0 1001 i
 *>  FC00::21/128     ::                       0         32768 i
R21#

```

R22
```
R22# show ip bgp ipv6 unicast
BGP table version is 3, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::14/128     2000:101:1001:0:100::2
                                                0             0 1001 i
 *>  FC00::22/128     ::                       0         32768 i
R22#

```
Соседство установлено, маршруты до лупбэков имеются.


##  Настроите eBGP IPv6 unicast между провайдерами Киторн и Ламас.
R21
```
R21(config)#int e0/1
R21(config-if)#ipv6 add 2000:101:301::11:11:11:2/64
R21(config)#router bgp 301
R21(config-router)#neighbor 2000:101:301::11:11:11:1 remote-as 101
R21(config-router)#address-family ipv6
R21(config-router-af)#neighbor 2000:101:301::11:11:11:1 activate
R21(config-router-af)#
*Jan 24 15:59:40.237: %BGP-5-ADJCHANGE: neighbor 2000:101:301:0:11:11:11:1 Up
R21(config-router-af)#
```
R22
```
R22(config)#int e0/1
R22(config-if)#ipv6 add 2000:101:301::11:11:11:1/64
R22(config-if)#router bgp 101
R22(config-router)#neighbor 2000:101:301::11:11:11:2 remote-as 301
R22(config-router)#address-family ipv6
R22(config-router-af)#neighbor 2000:101:301::11:11:11:2 activate
R22(config-router-af)#
R22#
*Jan 24 15:59:15.426: %SYS-5-CONFIG_I: Configured from console by console
R22#
*Jan 24 15:59:40.237: %BGP-5-ADJCHANGE: neighbor 2000:101:301:0:11:11:11:2 Up
R22#
R22#sh ip bgp ipv6 unicast
BGP table version is 5, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::14/128     2000:101:1001:0:100::2
                                                0             0 1001 i
 *>  FC00::15/128     2000:101:301:0:11:11:11:2
                                                              0 301 1001 i
 *>  FC00::21/128     2000:101:301:0:11:11:11:2
                                                0             0 301 i
 *>  FC00::22/128     ::                       0         32768 i
R22#


```
Соседство установлено, маршруты есть

##  Настроите eBGP IPv6 unicast между Ламас и Триада.
R21
```
R21(config)#int e0/2
R21(config-if)#ipv6 add 2000:301:520::13:13:13:1/64
R21(config-if)#router bgp 301
R21(config-router)#neighbor 2000:301:520::13:13:13:2 remote-as 520
R21(config-router)#address-family ipv6
R21(config-router-af)#neighbor 2000:301:520::13:13:13:2 activate
R21(config-router-af)#
```
R24
```
R24(config)#ipv6 unicast-routing
R24(config)#int lo0
R24(config-if)#ipv6 add FC00::24/128
R24(config-if)#int e0/0
R24(config-if)#ipv6 add 2000:301:520::13:13:13:2/64
R24(config-if)#router bgp 520
R24(config-router)#neighbor 2000:301:520::13:13:13:1 remote-as 301
R24(config-router)#address-family ipv6
R24(config-router-af)#net FC00::24/128
R24(config-router-af)#neighbor 2000:301:520::13:13:13:1 act
R24(config-router-af)#
*Jan 24 16:11:05.013: %BGP-5-ADJCHANGE: neighbor 2000:301:520:0:13:13:13:1 Up
R24(config-router-af)#
R24#sh ip bgp ipv6 unicast
BGP table version is 6, local router ID is 24.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::14/128     2000:301:520:0:13:13:13:1
                                                              0 301 101 1001 i
 *>  FC00::15/128     2000:301:520:0:13:13:13:1
                                                              0 301 1001 i
 *>  FC00::21/128     2000:301:520:0:13:13:13:1
                                                0             0 301 i
 *>  FC00::22/128     2000:301:520:0:13:13:13:1
                                                              0 301 101 i
 *>  FC00::24/128     ::                       0         32768 i
R24#

```

##  Настроите eBGP IPv6 unicast между офисом С.-Петербург и провайдером Триада.
R18
```
R18(config)#ipv6 unicast-routing
R18(config)#int lo0
R18(config-if)#ipv6 add FC00::18/128
R18(config-if)#int e0/2
R18(config-if)#ipv6 add 2000:520:2042::111:111:111:2/64
R18(config-if)#int e0/3
R18(config-if)#ipv6 add 2000:520:2042::111:111:1:2/64
%Ethernet0/3: Error: 2000:520:2042::/64 is overlapping with 2000:520:2042::/64 on Ethernet0/2
R18(config-if)#ipv6 add 2000:2042:520::111:111:1:2/64
R18(config-if)#router bgp 2042
R18(config-router)#neighbor 2000:520:2042::111:111:111:1 remote-as 520
R18(config-router)#neighbor 2000:2042:520::111:111:1:1 remote
R18(config-router)#neighbor 2000:2042:520::111:111:1:1 remote-as 520
R18(config-router)#add ipv6
R18(config-router-af)#neighbor 2000:520:2042::111:111:111:1 act
R18(config-router-af)#neighbor 2000:2042:520::111:111:1:1 act
R18(config-router-af)#net FC00::18/128
R18(config-router-af)#

```
R24
```
R24(config)#ipv6 u
R24(config)#int lo0
R24(config-if)#ipv6 add FC00::24/128
R24(config-if)#int e0/3
R24(config-if)#ipv6 add 2000:520:2042::111:111:111:1/64
R24(config-if)#router bgp 520
R24(config-router)#neighbor 2000:520:2042::111:111:111:2 remote-as 2042
R24(config-router)#add ipv6
R24(config-router-af)#net FC00::24/128
R24(config-router-af)#neighbor 2000:520:2042::111:111:111:2 act
R24(config-router-af)#
*Jan 24 17:51:24.067: %BGP-5-ADJCHANGE: neighbor 2000:520:2042:0:111:111:111:2 Up
R24(config-router-af)#

```
R26
```
R26(config)#ipv6 unicast-routing
R26(config)#int lo0
R26(config-if)#ipv6 add FC00::26/128
R26(config-if)#int e0/3
R26(config-if)#ipv6 add 2000:2042:520::111:111:1:1/64
R26(config-if)#router bgp 520
R26(config-router)#neighbor 2000:2042:520::111:111:1:2 remote-as 2042
R26(config-router)#add ipv6
R26(config-router-af)#neighbor 2000:2042:520::111:111:1:2 act
R26(config-router-af)#
*Jan 24 17:47:37.846: %BGP-5-ADJCHANGE: neighbor 2000:2042:520:0:111:111:1:2 Up
R26(config-router-af)#net FC00::26/128
R26(config-router-af)#

```
Проверка:
```
R18#sh ip bgp ipv6 uni
BGP table version is 8, local router ID is 18.18.18.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::14/128     2000:520:2042:0:111:111:111:1
                                                              0 520 301 101 1001 i
 *>  FC00::15/128     2000:520:2042:0:111:111:111:1
                                                              0 520 301 1001 i
 *>  FC00::18/128     ::                       0         32768 i
 *>  FC00::21/128     2000:520:2042:0:111:111:111:1
                                                              0 520 301 i
 *>  FC00::22/128     2000:520:2042:0:111:111:111:1
                                                              0 520 301 101 i
 *>  FC00::24/128     2000:520:2042:0:111:111:111:1
                                                0             0 520 i
 *>  FC00::26/128     2000:2042:520:0:111:111:1:1
                                                0             0 520 i
R18#
```

##  Организуете IPv6 unicast связность между пограничными роутерами офисов Москва и С.-Петербург.
R14
```
R14#sh ipv6 route
IPv6 Routing Table - default - 8 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
C   2000:101:1001::/64 [0/0]
     via Ethernet0/2, directly connected
L   2000:101:1001:0:100::2/128 [0/0]
     via Ethernet0/2, receive
LC  FC00::14/128 [0/0]
     via Loopback0, receive
B   FC00::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2
B   FC00::21/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2
B   FC00::22/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2
B   FC00::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2
L   FF00::/8 [0/0]
     via Null0, receive
R14#
R14#ping FC00::18 source FC00::14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FC00::18, timeout is 2 seconds:
Packet sent with a source address of FC00::14
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#

```
R15
```
R15#sh ipv6 route
IPv6 Routing Table - default - 8 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
C   2000:301:1001::/64 [0/0]
     via Ethernet0/2, directly connected
L   2000:301:1001:0:200::2/128 [0/0]
     via Ethernet0/2, receive
LC  FC00::15/128 [0/0]
     via Loopback0, receive
B   FC00::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   FC00::21/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   FC00::22/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   FC00::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
L   FF00::/8 [0/0]
     via Null0, receive
R15#
R15#ping FC00::18 source FC00::15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FC00::18, timeout is 2 seconds:
Packet sent with a source address of FC00::15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R15#
```
R18
```

R18#sh ipv6 route
IPv6 Routing Table - default - 12 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
C   2000:520:2042::/64 [0/0]
     via Ethernet0/2, directly connected
L   2000:520:2042:0:111:111:111:2/128 [0/0]
     via Ethernet0/2, receive
C   2000:2042:520::/64 [0/0]
     via Ethernet0/3, directly connected
L   2000:2042:520:0:111:111:1:2/128 [0/0]
     via Ethernet0/3, receive
B   FC00::14/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   FC00::15/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
LC  FC00::18/128 [0/0]
     via Loopback0, receive
B   FC00::21/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   FC00::22/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   FC00::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   FC00::26/128 [20/0]
     via FE80::A8BB:CCFF:FE01:A030, Ethernet0/3
L   FF00::/8 [0/0]
     via Null0, receive
R18#
R18#
R18#ping FC00::14 source FC00::18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FC00::14, timeout is 2 seconds:
Packet sent with a source address of FC00::18
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R18#ping FC00::15 source FC00::18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FC00::15, timeout is 2 seconds:
Packet sent with a source address of FC00::18
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R18#

```
##  Настроите iBGP IPv6 unicast в офисе Москва между маршрутизаторами R14 и R15.

Подняли OSPFV3 на R12-R15, анонсировали лупбэки, подняли iBGP между R14-R15
<details>
<summary>R12</summary>
<pre><code>

interface Loopback0
 ip address 1.1.1.12 255.255.255.255
 ipv6 address FC00::12/128
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/0
 ip address 10.10.10.1 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.10.10.5 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.10.10.9 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::122 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/3
 ip address 10.10.10.13 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::123 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router ospfv3 1
 router-id 12.12.12.12
 !
 address-family ipv6 unicast
 exit-address-family
!
router ospf 1
 router-id 12.12.12.12
 network 1.1.1.12 0.0.0.0 area 10
 network 10.10.10.0 0.0.0.3 area 10
 network 10.10.10.4 0.0.0.3 area 10
 network 10.10.10.8 0.0.0.3 area 0
 network 10.10.10.12 0.0.0.3 area 0
!

</code></pre>
</details>

<details>
<summary>R13</summary>
<pre><code>

interface Loopback0
 ip address 1.1.1.13 255.255.255.255
 ipv6 address FC00::13/128
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/0
 ip address 10.10.10.17 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.10.10.21 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.10.10.25 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::132 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/3
 ip address 10.10.10.29 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::133 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router ospfv3 1
 router-id 13.13.13.13
 !
 address-family ipv6 unicast
 exit-address-family
!
router ospf 1
 router-id 13.13.13.13
 network 1.1.1.13 0.0.0.0 area 10
 network 10.10.10.16 0.0.0.3 area 10
 network 10.10.10.20 0.0.0.3 area 10
 network 10.10.10.24 0.0.0.3 area 0
 network 10.10.10.28 0.0.0.3 area 0
!

</code></pre>
</details>
<details>
<summary>R14</summary>
<pre><code>

!
interface Loopback0
 ip address 1.1.1.14 255.255.255.255
 ipv6 address FC00::14/128
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/0
 ip address 10.10.10.10 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::140 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/1
 ip address 10.10.10.30 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::141 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/2
 ip address 100.0.0.2 255.255.255.252
 ipv6 address 2000:101:1001:0:100::2/64
!
interface Ethernet0/3
 ip address 10.10.10.33 255.255.255.252
 ip ospf 1 area 101
!
router ospfv3 1
 router-id 14.14.14.14
 !
 address-family ipv6 unicast
 exit-address-family
!
router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
 network 1.1.1.14 0.0.0.0 area 0
 network 10.10.10.12 0.0.0.3 area 0
 network 10.10.10.24 0.0.0.3 area 0
 network 10.10.10.32 0.0.0.3 area 101
 default-information originate
!
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 neighbor 1.1.1.15 remote-as 1001
 neighbor 1.1.1.15 update-source Loopback0
 neighbor 2000:101:1001:0:100::1 remote-as 101
 neighbor 100.0.0.1 remote-as 101
 neighbor FC00::15 remote-as 1001
 !
 address-family ipv4
  network 1.1.1.14 mask 255.255.255.255
  redistribute ospf 1
  neighbor 1.1.1.15 activate
  neighbor 1.1.1.15 soft-reconfiguration inbound
  no neighbor 2000:101:1001:0:100::1 activate
  neighbor 100.0.0.1 activate
  neighbor 100.0.0.1 soft-reconfiguration inbound
  neighbor 100.0.0.1 route-map R14_AS_PREPEND out
  neighbor 100.0.0.1 filter-list 1 out
  no neighbor FC00::15 activate
 exit-address-family
 !
 address-family ipv6
  network FC00::14/128
  neighbor 2000:101:1001:0:100::1 activate
  neighbor FC00::15 activate
 exit-address-family
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
no ip http server
no ip http secure-server
!
ipv6 route FC00::15/128 2000:1001::10:10:10:29
!
route-map R14_AS_PREPEND permit 10
 set as-path prepend 1001 1001
!
!

</code></pre>
</details>
<details>
<summary>R15</summary>
<pre><code>


!
interface Loopback0
 ip address 1.1.1.15 255.255.255.255
 ipv6 address FC00::15/128
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/0
 ip address 10.10.10.26 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::150 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/1
 ip address 10.10.10.14 255.255.255.252
 ip ospf 1 area 0
 ipv6 address FE80::151 link-local
 ospfv3 1 ipv6 area 0
!
interface Ethernet0/2
 ip address 200.0.0.2 255.255.255.252
 ipv6 address 2000:301:1001:0:200::2/64
!
interface Ethernet0/3
 ip address 10.10.10.37 255.255.255.252
 ip ospf 1 area 102
!
router ospfv3 1
 router-id 15.15.15.15
 !
 address-family ipv6 unicast
 exit-address-family
!
router ospf 1
 router-id 15.15.15.15
 area 0 filter-list prefix FILTER101 out
 network 1.1.1.15 0.0.0.0 area 0
 network 10.10.10.8 0.0.0.3 area 0
 network 10.10.10.28 0.0.0.3 area 0
 network 10.10.10.36 0.0.0.3 area 102
 default-information originate
!
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 neighbor 1.1.1.14 remote-as 1001
 neighbor 1.1.1.14 update-source Loopback0
 neighbor 2000:301:1001:0:200::1 remote-as 301
 neighbor 200.0.0.1 remote-as 301
 neighbor FC00::14 remote-as 1001
 !
 address-family ipv4
  network 1.1.1.15 mask 255.255.255.255
  redistribute ospf 1
  neighbor 1.1.1.14 activate
  no neighbor 2000:301:1001:0:200::1 activate
  neighbor 200.0.0.1 activate
  neighbor 200.0.0.1 soft-reconfiguration inbound
  neighbor 200.0.0.1 route-map R15_LOC_PREF in
  neighbor 200.0.0.1 filter-list 1 out
  no neighbor FC00::14 activate
 exit-address-family
 !
 address-family ipv6
  network FC00::15/128
  neighbor 2000:301:1001:0:200::1 activate
  neighbor FC00::14 activate
 exit-address-family
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 200.0.0.1
!
!
ip prefix-list FILTER101 seq 5 deny 10.10.10.32/30
ip prefix-list FILTER101 seq 10 permit 0.0.0.0/0 le 32
!
route-map R15_LOC_PREF permit 10
 set local-preference 200
!

</code></pre>
</details>

Проверки:

```
R15#sh ip bgp ipv6 uni
BGP table version is 58, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i FC00::14/128     FC00::14                 0    100      0 i
 *>  FC00::15/128     ::                       0         32768 i
 * i FC00::18/128     2000:101:1001:0:100::1
                                                0    100      0 101 301 520 2042 i
 *>                   2000:301:1001:0:200::1
                                                              0 301 520 2042 i
 * i FC00::21/128     2000:101:1001:0:100::1
                                                0    100      0 101 301 i
 *>                   2000:301:1001:0:200::1
                                                0             0 301 i
 *>  FC00::22/128     2000:301:1001:0:200::1
                                                              0 301 101 i
 * i                  2000:101:1001:0:100::1
                                                0    100      0 101 i
 * i FC00::24/128     2000:101:1001:0:100::1
                                                0    100      0 101 301 520 i
 *>                   2000:301:1001:0:200::1
                                                              0 301 520 i
 * i FC00::26/128     2000:101:1001:0:100::1
                                                0    100      0 101 301 520 i
 *>                   2000:301:1001:0:200::1
                                                              0 301 520 i
R15#
IPv6 Routing Table - default - 12 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
C   2000:301:1001::/64 [0/0]
     via Ethernet0/2, directly connected
L   2000:301:1001:0:200::2/128 [0/0]
     via Ethernet0/2, receive
O   FC00::12/128 [110/10]
     via FE80::123, Ethernet0/1
O   FC00::13/128 [110/10]
     via FE80::132, Ethernet0/0
O   FC00::14/128 [110/20]
     via FE80::123, Ethernet0/1
     via FE80::132, Ethernet0/0
LC  FC00::15/128 [0/0]
     via Loopback0, receive
B   FC00::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   FC00::21/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   FC00::22/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   FC00::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   FC00::26/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
L   FF00::/8 [0/0]
     via Null0, receive
R15#ping FC00::14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FC00::14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#traceroute ipv6 FC00::14
Type escape sequence to abort.
Tracing the route to FC00::14

  1 FC00::12 1 msec
    FC00::13 1 msec
    FC00::12 2 msec
  2 FC00::14 3 msec 3 msec 2 msec


```

##  Настроите iBGP IPv6 unicast в провайдере Триада, с использованием RR.

На роутерах Триады в ISIS аноннсировали IPv6 лупбэки, создали пиргруппу для IPv6 и подняли RR

<details>
<summary>R23</summary>
<pre><code>
interface Loopback0
 ip address 1.1.1.23 255.255.255.255
 ip router isis 1
 ipv6 address FC00::23/128
 ipv6 router isis 1
!
interface Ethernet0/0
 ip address 12.12.12.2 255.255.255.252
!
interface Ethernet0/1
 ip address 10.10.10.69 255.255.255.252
 ip router isis 1
 ipv6 address FE80::231 link-local
 ipv6 router isis 1
 isis circuit-type level-1
!
interface Ethernet0/2
 ip address 10.10.10.73 255.255.255.252
 ip router isis 1
 ipv6 address FE80::232 link-local
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/3
 no ip address
 shutdown
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router isis 1
 net 49.2222.0000.0000.0000.0023.00
!
router bgp 520
 bgp router-id 23.23.23.23
 bgp log-neighbor-changes
 neighbor 1.1.1.24 remote-as 520
 neighbor 1.1.1.24 update-source Loopback0
 neighbor 12.12.12.1 remote-as 101
 neighbor FC00::24 remote-as 520
 neighbor FC00::24 update-source Loopback0
 !
 address-family ipv4
  network 1.1.1.23 mask 255.255.255.255
  neighbor 1.1.1.24 activate
  neighbor 1.1.1.24 next-hop-self
  neighbor 12.12.12.1 activate
  no neighbor FC00::24 activate
 exit-address-family
 !
 address-family ipv6
  neighbor FC00::24 activate
 exit-address-family
!
</code></pre>
</details>
<details>
<summary>R24</summary>
<pre><code>

!
interface Loopback0
 ip address 1.1.1.24 255.255.255.255
 ip router isis 1
 ipv6 address FC00::24/128
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/0
 ip address 13.13.13.2 255.255.255.252
 ipv6 address FE80::240 link-local
 ipv6 address 2000:301:520:0:13:13:13:2/64
!
interface Ethernet0/1
 ip address 10.10.10.77 255.255.255.252
 ip router isis 1
 ipv6 address FE80::240 link-local
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/2
 ip address 10.10.10.74 255.255.255.252
 ip router isis 1
 ipv6 address FE80::242 link-local
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/3
 ip address 111.111.111.1 255.255.255.252
 ipv6 address 2000:520:2042:0:111:111:111:1/64
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router isis 1
 net 49.0024.0000.0000.0000.0024.00
 is-type level-2-only
!
router bgp 520
 bgp router-id 24.24.24.24
 bgp log-neighbor-changes
 neighbor TRIADAPG peer-group
 neighbor TRIADAPG remote-as 520
 neighbor TRIADAPG update-source Loopback0
 neighbor TRIADAV6 peer-group
 neighbor TRIADAV6 remote-as 520
 neighbor TRIADAV6 update-source Loopback0
 neighbor 1.1.1.23 peer-group TRIADAPG
 neighbor 1.1.1.25 peer-group TRIADAPG
 neighbor 1.1.1.26 peer-group TRIADAPG
 neighbor 13.13.13.1 remote-as 301
 neighbor 2000:301:520:0:13:13:13:1 remote-as 301
 neighbor 2000:520:2042:0:111:111:111:2 remote-as 2042
 neighbor 111.111.111.2 remote-as 2042
 neighbor FC00::23 peer-group TRIADAV6
 neighbor FC00::25 peer-group TRIADAV6
 neighbor FC00::26 peer-group TRIADAV6
 !
 address-family ipv4
  network 1.1.1.24 mask 255.255.255.255
  neighbor TRIADAPG route-reflector-client
  neighbor TRIADAPG next-hop-self
  neighbor 1.1.1.23 activate
  neighbor 1.1.1.25 activate
  neighbor 1.1.1.26 activate
  neighbor 13.13.13.1 activate
  no neighbor 2000:301:520:0:13:13:13:1 activate
  no neighbor 2000:520:2042:0:111:111:111:2 activate
  neighbor 111.111.111.2 activate
  neighbor 111.111.111.2 soft-reconfiguration inbound
  no neighbor FC00::23 activate
  no neighbor FC00::25 activate
  no neighbor FC00::26 activate
 exit-address-family
 !
 address-family ipv6
  network FC00::24/128
  neighbor TRIADAV6 route-reflector-client
  neighbor TRIADAV6 next-hop-self
  neighbor 2000:301:520:0:13:13:13:1 activate
  neighbor 2000:520:2042:0:111:111:111:2 activate
  neighbor FC00::23 activate
  neighbor FC00::25 activate
  neighbor FC00::26 activate
 exit-address-family
!


</code></pre>
</details>
<details>
<summary>R25</summary>
<pre><code>
!
interface Loopback0
 ip address 1.1.1.25 255.255.255.255
 ip router isis 1
 ipv6 address FC00::25/128
 ipv6 router isis 1
!
interface Ethernet0/0
 ip address 10.10.10.70 255.255.255.252
 ip router isis 1
 ipv6 address FE80::250 link-local
 ipv6 router isis 1
 isis circuit-type level-1
!
interface Ethernet0/1
 ip address 111.1.1.1 255.255.255.252
!
interface Ethernet0/2
 ip address 10.10.10.81 255.255.255.252
 ip router isis 1
 ipv6 address FE80::252 link-local
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/3
 ip address 15.15.15.1 255.255.255.252
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router isis 1
 net 49.2222.0000.0000.0000.0025.00
!
router bgp 520
 bgp router-id 25.25.25.25
 bgp log-neighbor-changes
 neighbor 1.1.1.24 remote-as 520
 neighbor 1.1.1.24 update-source Loopback0
 neighbor FC00::24 remote-as 520
 neighbor FC00::24 update-source Loopback0
 !
 address-family ipv4
  network 1.1.1.25 mask 255.255.255.255
  neighbor 1.1.1.24 activate
  neighbor 1.1.1.24 next-hop-self
  no neighbor FC00::24 activate
 exit-address-family
 !
 address-family ipv6
  neighbor FC00::24 activate
 exit-address-family
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 192.168.50.0 255.255.255.0 15.15.15.2
ip route 192.168.60.0 255.255.255.0 15.15.15.2
!
!


</code></pre>
</details>
<details>
<summary>R26</summary>
<pre><code>
!
interface Loopback0
 ip address 1.1.1.26 255.255.255.255
 ip router isis 1
 ipv6 address FC00::26/128
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/0
 ip address 10.10.10.78 255.255.255.252
 ip router isis 1
 ipv6 address FE80::260 link-local
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/1
 ip address 16.16.16.1 255.255.255.252
!
interface Ethernet0/2
 ip address 10.10.10.82 255.255.255.252
 ip router isis 1
 ipv6 address FE80::262 link-local
 ipv6 router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/3
 ip address 111.111.1.1 255.255.255.252
 ipv6 address 2000:2042:520:0:111:111:1:1/64
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router isis 1
 net 49.0026.0000.0000.0000.0026.00
 is-type level-2-only
!
router bgp 520
 bgp router-id 26.26.26.26
 bgp log-neighbor-changes
 neighbor 1.1.1.24 remote-as 520
 neighbor 1.1.1.24 update-source Loopback0
 neighbor 2000:2042:520:0:111:111:1:2 remote-as 2042
 neighbor 111.111.1.2 remote-as 2042
 neighbor FC00::24 remote-as 520
 neighbor FC00::24 update-source Loopback0
 !
 address-family ipv4
  network 1.1.1.26 mask 255.255.255.255
  neighbor 1.1.1.24 activate
  neighbor 1.1.1.24 next-hop-self
  no neighbor 2000:2042:520:0:111:111:1:2 activate
  neighbor 111.111.1.2 activate
  neighbor 111.111.1.2 soft-reconfiguration inbound
  no neighbor FC00::24 activate
 exit-address-family
 !
 address-family ipv6
  network FC00::26/128
  neighbor 2000:2042:520:0:111:111:1:2 activate
  neighbor FC00::24 activate
 exit-address-family
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 192.168.50.0 255.255.255.0 16.16.16.2
ip route 192.168.60.0 255.255.255.0 16.16.16.2
!
</code></pre>
</details>

проверки:
```
R24#sh ip bgp ipv6 unicast
BGP table version is 74, local router ID is 24.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  FC00::14/128     2000:301:520:0:13:13:13:1
                                                              0 301 1001 i
 *>  FC00::15/128     2000:301:520:0:13:13:13:1
                                                              0 301 1001 i
 * i FC00::18/128     2000:2042:520:0:111:111:1:2
                                                0    100      0 2042 i
 *>                   2000:520:2042:0:111:111:111:2
                                                0             0 2042 i
 *>  FC00::21/128     2000:301:520:0:13:13:13:1
                                                0             0 301 i
 *>  FC00::22/128     2000:301:520:0:13:13:13:1
                                                              0 301 101 i
 *>  FC00::24/128     ::                       0         32768 i
 r>i FC00::26/128     FC00::26                 0    100      0 i
R24#

```
```
R25#sh ip bgp ipv6 unicast
BGP table version is 23, local router ID is 25.25.25.25
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i FC00::14/128     FC00::24                 0    100      0 301 1001 i
 *>i FC00::15/128     FC00::24                 0    100      0 301 1001 i
 *>i FC00::18/128     FC00::24                 0    100      0 2042 i
 *>i FC00::21/128     FC00::24                 0    100      0 301 i
 *>i FC00::22/128     FC00::24                 0    100      0 301 101 i
 r>i FC00::24/128     FC00::24                 0    100      0 i
 r>i FC00::26/128     FC00::26                 0    100      0 i
R25#
R25#sh ipv6 route
IPv6 Routing Table - default - 10 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   FC00::14/128 [200/0]
     via FC00::24
B   FC00::15/128 [200/0]
     via FC00::24
B   FC00::18/128 [200/0]
     via FC00::24
B   FC00::21/128 [200/0]
     via FC00::24
B   FC00::22/128 [200/0]
     via FC00::24
I1  FC00::23/128 [115/20]
     via FE80::231, Ethernet0/0
I2  FC00::24/128 [115/30]
     via FE80::262, Ethernet0/2
LC  FC00::25/128 [0/0]
     via Loopback0, receive
I2  FC00::26/128 [115/20]
     via FE80::262, Ethernet0/2
L   FF00::/8 [0/0]
     via Null0, receive
R25#

```
