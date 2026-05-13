# iBGP

## Цель:
Настроить iBGP в офисе Москва       
Настроить iBGP в сети провайдера Триада     
Организовать полную IP связанность всех сетей       


## Описание/Пошаговая инструкция выполнения домашнего задания:


Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15.        
Настроите iBGP в провайдере Триада, с использованием RR.        
Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.      
Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.       
Все сети в лабораторной работе должны иметь IP связность.       


## Топология 

![](Topology8.png)

## Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15.     
Прямого линка между R14 и R15 нет, но есть пути через R12 и R13, где развернут OSPF и маршруты до лупбэков уже есть. iBGP соседство можно построить по Lo0


R14
```
R14#sh ip route 1.1.1.15
Routing entry for 1.1.1.15/32
  Known via "ospf 1", distance 110, metric 21, type intra area
  Last update from 10.10.10.9 on Ethernet0/0, 01:38:32 ago
  Routing Descriptor Blocks:
  * 10.10.10.29, from 15.15.15.15, 01:38:32 ago, via Ethernet0/1
      Route metric is 21, traffic share count is 1
    10.10.10.9, from 15.15.15.15, 01:38:32 ago, via Ethernet0/0
      Route metric is 21, traffic share count is 1
R14#
R14#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R14(config)#router bgp 1001
R14(config-router)#nei
R14(config-router)#neighbor 1.1.1.15 remote-as 1001
R14(config-router)#
R14(config-router)#neighbor 1.1.1.15 update-source lo0




```

R15
```
R15#sh ip route 1.1.1.14
Routing entry for 1.1.1.14/32
  Known via "ospf 1", distance 110, metric 21, type intra area
  Last update from 10.10.10.25 on Ethernet0/0, 01:40:17 ago
  Routing Descriptor Blocks:
    10.10.10.25, from 14.14.14.14, 01:40:17 ago, via Ethernet0/0
      Route metric is 21, traffic share count is 1
  * 10.10.10.13, from 14.14.14.14, 01:40:17 ago, via Ethernet0/1
      Route metric is 21, traffic share count is 1
R15#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R15(config)#router bgp 1001
R15(config-router)#neighbor 1.1.1.14 remote-as 1001
R15(config-router)#
R15(config-router)#neighbor 1.1.1.14 update-source lo0

```
```
R14#sh ip bgp sum
BGP router identifier 14.14.14.14, local AS number 1001
BGP table version is 10, main routing table version 10
6 network entries using 840 bytes of memory
9 path entries using 720 bytes of memory
9/6 BGP path/bestpath attribute entries using 1296 bytes of memory
7 BGP AS-PATH entries using 184 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3040 total bytes of memory
BGP activity 13/7 prefixes, 18/9 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.15        4         1001      11      13       10    0    0 00:01:12        4
100.0.0.1       4          101      24      20       10    0    0 00:11:45        4



R15#sh ip bgp sum
BGP router identifier 15.15.15.15, local AS number 1001
BGP table version is 8, main routing table version 8
6 network entries using 840 bytes of memory
7 path entries using 560 bytes of memory
7/6 BGP path/bestpath attribute entries using 1008 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2528 total bytes of memory
BGP activity 11/5 prefixes, 15/8 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.14        4         1001      16      14        8    0    0 00:03:43        2
200.0.0.1       4          301      28      21        8    0    0 00:14:32        4
R15#

```

## Настроите iBGP в провайдере Триада, с использованием RR.  

Пусть R24 будет RR, тогда:

R24
```
R24#sh run | sec bgp
router bgp 520
 bgp router-id 24.24.24.24
 bgp log-neighbor-changes
 network 1.1.1.24 mask 255.255.255.255
 neighbor TRIADAPG peer-group
 neighbor TRIADAPG remote-as 520
 neighbor TRIADAPG update-source Loopback0
 neighbor TRIADAPG route-reflector-client
 neighbor TRIADAPG next-hop-self
 neighbor 1.1.1.23 peer-group TRIADAPG
 neighbor 1.1.1.25 peer-group TRIADAPG
 neighbor 1.1.1.26 peer-group TRIADAPG
 neighbor 13.13.13.1 remote-as 301
 neighbor 111.111.111.2 remote-as 2042
R24#


```

R23
```
R23(config)#router bgp 520
R23(config-router)#bgp router-id 23.23.23.23
R23(config-router)#network 1.1.1..23 mask 255.255.255.255
R23(config-router)# neighbor 1.1.1.24 remote-as 520
R23(config-router)# neighbor 1.1.1.24 update-source Loopback0
R23(config-router)# neighbor 1.1.1.24 next-hop-self
R23(config-router)#neighbor 12.12.12.1 remote-as 101
```


R25
```
R25(config)#router bgp 520
R25(config-router)#bgp router
R25(config-router)#bgp router-id 25.25.25.25
R25(config-router)# neighbor 1.1.1.24 remote-as 520
R25(config-router)# neighbor 1.1.1.24  update-source Loopback0
R25(config-router)# neighbor 1.1.1.24  next-hop-self
R25(config-router)#network 1.1.1.25 mask 255.255.255.255

```

R26
```
R26(config)#router bgp 520
R26(config-router)#bgp router-id 26.26.26.26
R26(config-router)#network 1.1.1.26 mask 255.255.255.255
R26(config-router)#neighbor 1.1.1.24 remote-as 520
R26(config-router)#neighbor 1.1.1.24 update-source Loopback0
R26(config-router)#neighbor 1.1.1.24 next-hop-self
R26(config-router)#
*Dec 28 18:04:49.272: %BGP-5-ADJCHANGE: neighbor 1.1.1.24 Up
R26(config-router)#
```

Check: Соседство R24:
```
R24#sh ip bgp sum
BGP router identifier 24.24.24.24, local AS number 520
BGP table version is 16, main routing table version 16
9 network entries using 1260 bytes of memory
12 path entries using 960 bytes of memory
8/6 BGP path/bestpath attribute entries using 1152 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3516 total bytes of memory
BGP activity 11/2 prefixes, 14/2 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.23        4          520      11      15       16    0    0 00:04:45        4
1.1.1.25        4          520       6      15       16    0    0 00:01:34        1
1.1.1.26        4          520       7      16       16    0    0 00:03:47        1
13.13.13.1      4          301     389     381       16    0    0 05:38:06        4
111.111.111.2   4         2042     375     382       16    0    0 05:30:59        1
R24#

```


## Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.  


Пишем route-map на R15 на увеличение Local preference. на R14 указываем prepends. Сделаем редистрибьюцию OSPF и EIGRP в BGP

В итоге конфиги:
R14
```
R14#sh run | sec bgp
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 network 1.1.1.14 mask 255.255.255.255
 redistribute ospf 1
 neighbor 1.1.1.15 remote-as 1001
 neighbor 1.1.1.15 update-source Loopback0
 neighbor 1.1.1.15 soft-reconfiguration inbound
 neighbor 100.0.0.1 remote-as 101
 neighbor 100.0.0.1 route-map R14_AS_PREPEND out
R14#sh ip bgp
BGP table version is 318, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 1.1.1.4/32       10.10.10.13             21    100      0 ?
 *>                   10.10.10.9              21         32768 ?
 * i 1.1.1.5/32       10.10.10.13             21    100      0 ?
 *>                   10.10.10.9              21         32768 ?
 *   1.1.1.9/32       100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 *   1.1.1.10/32      100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 * i 1.1.1.12/32      10.10.10.13             11    100      0 ?
 *>                   10.10.10.9              11         32768 ?
 * i 1.1.1.13/32      10.10.10.25             11    100      0 ?
 *>                   10.10.10.29             11         32768 ?
 * i 1.1.1.14/32      10.10.10.13             21    100      0 ?
 *>                   0.0.0.0                  0         32768 i
 * i 1.1.1.15/32      1.1.1.15                 0    100      0 i
 *>                   10.10.10.9              21         32768 ?
 *   1.1.1.16/32      100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 *   1.1.1.17/32      100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 *   1.1.1.18/32      100.0.0.1                              0 101 520 2042 i
 *>i                  200.0.0.1                0    200      0 301 520 2042 i
 * i 1.1.1.19/32      10.10.10.13             31    100      0 ?
 *>                   10.10.10.34             11         32768 ?
 * i 1.1.1.20/32      10.10.10.38             11    100      0 ?
 *>                   10.10.10.9              31         32768 ?
 *   1.1.1.21/32      100.0.0.1                              0 101 301 i
 *>i                  200.0.0.1                0    200      0 301 i
 *>i 1.1.1.22/32      200.0.0.1                0    200      0 301 101 i
 *                    100.0.0.1                0             0 101 i
 *   1.1.1.23/32      100.0.0.1                              0 101 520 i
 *>i                  200.0.0.1                0    200      0 301 520 i
 *   1.1.1.24/32      100.0.0.1                              0 101 520 i
 *>i                  200.0.0.1                0    200      0 301 520 i
 *   1.1.1.25/32      100.0.0.1                              0 101 520 i
 *>i                  200.0.0.1                0    200      0 301 520 i
 *   1.1.1.26/32      100.0.0.1                              0 101 520 i
 *>i                  200.0.0.1                0    200      0 301 520 i
 *   1.1.1.32/32      100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 * i 10.10.10.0/30    10.10.10.13             20    100      0 ?
 *>                   10.10.10.9              20         32768 ?
 * i 10.10.10.4/30    10.10.10.13             20    100      0 ?
 *>                   10.10.10.9              20         32768 ?
 * i 10.10.10.8/30    10.10.10.13             20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.10.12/30   1.1.1.15                 0    100      0 ?
 *>                   10.10.10.9              20         32768 ?
 * i 10.10.10.16/30   10.10.10.25             20    100      0 ?
 *>                   10.10.10.29             20         32768 ?
 * i 10.10.10.20/30   10.10.10.25             20    100      0 ?
 *>                   10.10.10.29             20         32768 ?
 * i 10.10.10.24/30   1.1.1.15                 0    100      0 ?
 *>                   10.10.10.29             20         32768 ?
 * i 10.10.10.28/30   10.10.10.25             20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.10.32/30   10.10.10.13             30    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 *   10.10.10.32/27   100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 * i 10.10.10.36/30   1.1.1.15                 0    100      0 ?
 *>                   10.10.10.9              30         32768 ?
 *   10.10.10.44/30   100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 *   10.10.10.60/30   100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 *   10.10.10.64/30   100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 * i 192.168.10.0     10.10.10.13             21    100      0 ?
 *>                   10.10.10.9              21         32768 ?
 * i 192.168.20.0     10.10.10.13             21    100      0 ?
 *>                   10.10.10.9              21         32768 ?
 *   192.168.30.0     100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
 *   192.168.40.0     100.0.0.1                              0 101 520 2042 ?
 *>i                  200.0.0.1                0    200      0 301 520 2042 ?
R14#

```

R15
```
R15#sh run | sec bgp
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 network 1.1.1.15 mask 255.255.255.255
 redistribute ospf 1
 neighbor 1.1.1.14 remote-as 1001
 neighbor 1.1.1.14 update-source Loopback0
 neighbor 200.0.0.1 remote-as 301
 neighbor 200.0.0.1 route-map R15_LOC_PREF in
R15#sh ip bgp
BGP table version is 224, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 1.1.1.4/32       10.10.10.9              21    100      0 ?
 *>                   10.10.10.13             21         32768 ?
 * i 1.1.1.5/32       10.10.10.9              21    100      0 ?
 *>                   10.10.10.13             21         32768 ?
 *>  1.1.1.9/32       200.0.0.1                     200      0 301 520 2042 ?
 *>  1.1.1.10/32      200.0.0.1                     200      0 301 520 2042 ?
 * i 1.1.1.12/32      10.10.10.9              11    100      0 ?
 *>                   10.10.10.13             11         32768 ?
 * i 1.1.1.13/32      10.10.10.29             11    100      0 ?
 *>                   10.10.10.25             11         32768 ?
 * i 1.1.1.14/32      1.1.1.14                 0    100      0 i
 *>                   10.10.10.13             21         32768 ?
 * i 1.1.1.15/32      10.10.10.9              21    100      0 ?
 *>                   0.0.0.0                  0         32768 i
 *>  1.1.1.16/32      200.0.0.1                     200      0 301 520 2042 ?
 *>  1.1.1.17/32      200.0.0.1                     200      0 301 520 2042 ?
 *>  1.1.1.18/32      200.0.0.1                     200      0 301 520 2042 i
 * i 1.1.1.19/32      10.10.10.34             11    100      0 ?
 *>                   10.10.10.13             31         32768 ?
 * i 1.1.1.20/32      10.10.10.9              31    100      0 ?
 *>                   10.10.10.38             11         32768 ?
 *>  1.1.1.21/32      200.0.0.1                0    200      0 301 i
 *>  1.1.1.22/32      200.0.0.1                     200      0 301 101 i
 *>  1.1.1.23/32      200.0.0.1                     200      0 301 520 i
 *>  1.1.1.24/32      200.0.0.1                     200      0 301 520 i
 *>  1.1.1.25/32      200.0.0.1                     200      0 301 520 i
 *>  1.1.1.26/32      200.0.0.1                     200      0 301 520 i
 *>  1.1.1.32/32      200.0.0.1                     200      0 301 520 2042 ?
 * i 10.10.10.0/30    10.10.10.9              20    100      0 ?
 *>                   10.10.10.13             20         32768 ?
 * i 10.10.10.4/30    10.10.10.9              20    100      0 ?
 *>                   10.10.10.13             20         32768 ?
 * i 10.10.10.8/30    1.1.1.14                 0    100      0 ?
 *>                   10.10.10.13             20         32768 ?
 * i 10.10.10.12/30   10.10.10.9              20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.10.16/30   10.10.10.29             20    100      0 ?
 *>                   10.10.10.25             20         32768 ?
 * i 10.10.10.20/30   10.10.10.29             20    100      0 ?
 *>                   10.10.10.25             20         32768 ?
 * i 10.10.10.24/30   10.10.10.29             20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.10.28/30   1.1.1.14                 0    100      0 ?
 *>                   10.10.10.25             20         32768 ?
 * i 10.10.10.32/30   1.1.1.14                 0    100      0 ?
 *>                   10.10.10.13             30         32768 ?
 *>  10.10.10.32/27   200.0.0.1                     200      0 301 520 2042 ?
 * i 10.10.10.36/30   10.10.10.9              30    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 *>  10.10.10.44/30   200.0.0.1                     200      0 301 520 2042 ?
 *>  10.10.10.60/30   200.0.0.1                     200      0 301 520 2042 ?
 *>  10.10.10.64/30   200.0.0.1                     200      0 301 520 2042 ?
 * i 192.168.10.0     10.10.10.9              21    100      0 ?
 *>                   10.10.10.13             21         32768 ?
 * i 192.168.20.0     10.10.10.9              21    100      0 ?
 *>                   10.10.10.13             21         32768 ?
 *>  192.168.30.0     200.0.0.1                     200      0 301 520 2042 ?
 *>  192.168.40.0     200.0.0.1                     200      0 301 520 2042 ?
R15#
R15#

```

R22 (AS prepends)
```
R22#sh ip bgp
BGP table version is 335, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   1.1.1.4/32       100.0.0.2               21             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   1.1.1.5/32       100.0.0.2               21             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   1.1.1.9/32       100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   1.1.1.10/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   1.1.1.12/32      100.0.0.2               11             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   1.1.1.13/32      100.0.0.2               11             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   1.1.1.14/32      12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *                    100.0.0.2                0             0 1001 1001 1001 i
 *   1.1.1.15/32      100.0.0.2               21             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 i
 *>                   11.11.11.2                             0 301 1001 i
 *   1.1.1.16/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   1.1.1.17/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   1.1.1.18/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 i
 *>                   12.12.12.2                             0 520 2042 i
 *                    11.11.11.2                             0 301 520 2042 i
 *   1.1.1.19/32      12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *                    100.0.0.2               11             0 1001 1001 1001 ?
 *   1.1.1.20/32      100.0.0.2               31             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   1.1.1.21/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 i
 *                    12.12.12.2                             0 520 301 i
 *>                   11.11.11.2               0             0 301 i
 *>  1.1.1.22/32      0.0.0.0                  0         32768 i
 *   1.1.1.23/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 i
 *                    11.11.11.2                             0 301 520 i
 *>                   12.12.12.2               0             0 520 i
 *   1.1.1.24/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 i
 *>                   12.12.12.2                             0 520 i
 *                    11.11.11.2                             0 301 520 i
 *   1.1.1.25/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 i
 *>                   12.12.12.2                             0 520 i
 *                    11.11.11.2                             0 301 520 i
 *   1.1.1.26/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 i
 *>                   12.12.12.2                             0 520 i
 *                    11.11.11.2                             0 301 520 i
 *   1.1.1.32/32      100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   10.10.10.0/30    100.0.0.2               20             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   10.10.10.4/30    100.0.0.2               20             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   10.10.10.8/30    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *                    100.0.0.2                0             0 1001 1001 1001 ?
 *   10.10.10.12/30   100.0.0.2               20             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   10.10.10.16/30   100.0.0.2               20             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   10.10.10.20/30   100.0.0.2               20             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   10.10.10.24/30   100.0.0.2               20             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   10.10.10.28/30   12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *                    100.0.0.2                0             0 1001 1001 1001 ?
 *   10.10.10.32/30   12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *                    100.0.0.2                0             0 1001 1001 1001 ?
 *   10.10.10.32/27   100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   10.10.10.36/30   100.0.0.2               30             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   10.10.10.44/30   100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   10.10.10.60/30   100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   10.10.10.64/30   100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   192.168.10.0     100.0.0.2               21             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   192.168.20.0     100.0.0.2               21             0 1001 1001 1001 ?
 *                    12.12.12.2                             0 520 301 1001 ?
 *>                   11.11.11.2                             0 301 1001 ?
 *   192.168.30.0     100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?
 *   192.168.40.0     100.0.0.2                              0 1001 1001 1001 30                                                       1 520 2042 ?
 *>                   12.12.12.2                             0 520 2042 ?
 *                    11.11.11.2                             0 301 520 2042 ?

```

Трассировка с хоста из москвы в СПБ
```

VPCS> sh ip

NAME        : VPCS[1]
IP/MASK     : 192.168.10.10/24
GATEWAY     : 192.168.10.1
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> trace 192.168.30.10
trace to 192.168.30.10, 8 hops max, press Ctrl+C to stop
 1   192.168.10.3   3.891 ms  2.502 ms  3.500 ms
 2   10.10.10.5   6.635 ms  0.940 ms  1.313 ms
 3   10.10.10.14   2.234 ms  2.662 ms  2.314 ms
 4   200.0.0.1   1.803 ms  7.782 ms  2.295 ms
 5   13.13.13.2   1.221 ms  1.064 ms  0.934 ms
 6   111.111.111.2   9.219 ms  3.663 ms  10.242 ms
 7   10.10.10.61   2.628 ms  8.768 ms  6.761 ms
 8   10.10.10.66   8.156 ms  3.115 ms  7.243 ms

VPCS>

```
Трассировка из СПБ в Москву
```

VPCS> sh ip

NAME        : VPCS[1]
IP/MASK     : 192.168.40.10/24
GATEWAY     : 192.168.40.1
DNS         :
MAC         : 00:50:79:66:68:0b
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> trace 192.168.20.10
trace to 192.168.20.10, 8 hops max, press Ctrl+C to stop
 1   192.168.40.3   0.976 ms  1.098 ms  0.237 ms
 2   10.10.10.65   1.214 ms  1.878 ms  1.803 ms
 3   10.10.10.62   3.117 ms  2.578 ms  3.072 ms
 4   111.111.111.1   1.845 ms  4.137 ms  1.730 ms
 5   13.13.13.1   4.985 ms  5.106 ms  7.109 ms
 6   200.0.0.2   7.223 ms  5.476 ms  8.603 ms
 7   10.10.10.13   4.656 ms  5.041 ms  6.468 ms
 8   10.10.10.2   5.951 ms  4.212 ms  2.982 ms

VPCS>

```
с R18 до SW5:
```

R18#trace 1.1.1.5 source 1.1.1.18
Type escape sequence to abort.
Tracing the route to 1.1.1.5
VRF info: (vrf in name/id, vrf out name/id)
  1 111.111.111.1 2 msec 2 msec 1 msec
  2 13.13.13.1 2 msec 3 msec 3 msec
  3 200.0.0.2 3 msec 2 msec 1 msec
  4 10.10.10.13 [AS 1001] 2 msec 5 msec 3 msec
  5 10.10.10.6 [AS 1001] 4 msec 5 msec *
R18#
```

Трафик ходит через ЛАмас - 200.0.0.0/30

## Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.       

R18
```
R18(config)#router bgp 2042
R18(config-router)#bgp bestpath as-path multipath-relax
R18(config-router)#maximum-paths 2
R18(config-router)#

```
