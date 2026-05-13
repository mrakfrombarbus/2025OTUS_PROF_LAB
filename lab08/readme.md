# BGP. Фильтрация

## Цель:
Настроить фильтрацию для офисе Москва       
Настроить фильтрацию для офисе С.-Петербург     


Описание/Пошаговая инструкция выполнения домашнего задания:     


Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).       
Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).     
Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.     
Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.     
Все сети в лабораторной работе должны иметь IP связность.       

## Топология
![](Topology8.png)

## Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path). 

<details>
<summary>R21 до фильтрации</summary>
<pre><code>
R21#sh ip bgp neighbors 200.0.0.2 received-routes
BGP table version is 298, local router ID is 21.21.21.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  1.1.1.4/32       200.0.0.2               21             0 1001 ?
 *>  1.1.1.5/32       200.0.0.2               21             0 1001 ?
 *>  1.1.1.12/32      200.0.0.2               11             0 1001 ?
 *>  1.1.1.13/32      200.0.0.2               11             0 1001 ?
 *>  1.1.1.14/32      200.0.0.2               21             0 1001 ?
 *>  1.1.1.15/32      200.0.0.2                0             0 1001 i
 *>  1.1.1.19/32      200.0.0.2               31             0 1001 ?
 *>  1.1.1.20/32      200.0.0.2               11             0 1001 ?
 *>  10.10.10.0/30    200.0.0.2               20             0 1001 ?
 *>  10.10.10.4/30    200.0.0.2               20             0 1001 ?
 *>  10.10.10.8/30    200.0.0.2               20             0 1001 ?
 *>  10.10.10.12/30   200.0.0.2                0             0 1001 ?
 *>  10.10.10.16/30   200.0.0.2               20             0 1001 ?
 *>  10.10.10.20/30   200.0.0.2               20             0 1001 ?
 *>  10.10.10.24/30   200.0.0.2                0             0 1001 ?
 *>  10.10.10.28/30   200.0.0.2               20             0 1001 ?
 *>  10.10.10.32/30   200.0.0.2               30             0 1001 ?
 *>  10.10.10.36/30   200.0.0.2                0             0 1001 ?
 *>  192.168.10.0     200.0.0.2               21             0 1001 ?
 *>  192.168.20.0     200.0.0.2               21             0 1001 ?

Total number of prefixes 20

</code></pre>
</details>

<details>
<summary>R22 до фильтрации</summary>
<pre><code>

```
R22#sh ip bgp neighbors 100.0.0.2 received-routes
BGP table version is 363, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   1.1.1.4/32       100.0.0.2                              0 1001 1001 1001 ?
 *   1.1.1.5/32       100.0.0.2                              0 1001 1001 1001 ?
 *   1.1.1.9/32       100.0.0.2                              0 1001 1001 1001 301 520 2042 ?
 *   1.1.1.10/32      100.0.0.2                              0 1001 1001 1001 301 520 2042 ?
 *   1.1.1.12/32      100.0.0.2                              0 1001 1001 1001 ?
 *   1.1.1.13/32      100.0.0.2                              0 1001 1001 1001 ?
 *   1.1.1.14/32      100.0.0.2                              0 1001 1001 1001 ?
 *   1.1.1.15/32      100.0.0.2                              0 1001 1001 1001 i
 *   1.1.1.16/32      100.0.0.2                              0 1001 1001 1001 301 520 2042 ?
 *   1.1.1.17/32      100.0.0.2                              0 1001 1001 1001 301 520 2042 ?
 *   1.1.1.18/32      100.0.0.2                              0 1001 1001 1001 301 520 2042 i
 *   1.1.1.19/32      100.0.0.2                              0 1001 1001 1001 ?
 *   1.1.1.20/32      100.0.0.2                              0 1001 1001 1001 ?
 *   1.1.1.21/32      100.0.0.2                              0 1001 1001 1001 301 i
 *   1.1.1.23/32      100.0.0.2                              0 1001 1001 1001 301 520 i
 *   1.1.1.24/32      100.0.0.2                              0 1001 1001 1001 301 520 i
 *   1.1.1.25/32      100.0.0.2                              0 1001 1001 1001 301 520 i
 *   1.1.1.26/32      100.0.0.2                              0 1001 1001 1001 301 520 i
 *   1.1.1.32/32      100.0.0.2                              0 1001 1001 1001 301 520 2042 ?
 *   10.10.10.0/30    100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.4/30    100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.8/30    100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.12/30   100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.16/30   100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.20/30   100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.24/30   100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.28/30   100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.32/30   100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.32/27   100.0.0.2                              0 1001 1001 1001 301 520 2042 ?
 *   10.10.10.36/30   100.0.0.2                              0 1001 1001 1001 ?
 *   10.10.10.64/30   100.0.0.2                              0 1001 1001 1001 301 520 2042 ?
 *   192.168.10.0     100.0.0.2                              0 1001 1001 1001 ?
 *   192.168.20.0     100.0.0.2                              0 1001 1001 1001 ?

Total number of prefixes 33
R22#
``` 
</code></pre>
</details>

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
 neighbor 100.0.0.1 filter-list 1 out
R14#sh run | sec acc
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*


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
 neighbor 200.0.0.1 filter-list 1 out
R15#sh run | sec acc
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
R15#

```

<details>
<summary>R21 после фильтрации</summary>
<pre><code>
```
R21#sh ip bgp neighbors 200.0.0.2 received-routes
BGP table version is 316, local router ID is 21.21.21.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  1.1.1.4/32       200.0.0.2               21             0 1001 ?
 *>  1.1.1.5/32       200.0.0.2               21             0 1001 ?
 *>  1.1.1.12/32      200.0.0.2               11             0 1001 ?
 *>  1.1.1.13/32      200.0.0.2               11             0 1001 ?
 *>  1.1.1.14/32      200.0.0.2               21             0 1001 ?
 *>  1.1.1.15/32      200.0.0.2                0             0 1001 i
 *>  1.1.1.19/32      200.0.0.2               31             0 1001 ?
 *>  1.1.1.20/32      200.0.0.2               11             0 1001 ?
 *>  10.10.10.0/30    200.0.0.2               20             0 1001 ?
 *>  10.10.10.4/30    200.0.0.2               20             0 1001 ?
 *>  10.10.10.8/30    200.0.0.2               20             0 1001 ?
 *>  10.10.10.12/30   200.0.0.2                0             0 1001 ?
 *>  10.10.10.16/30   200.0.0.2               20             0 1001 ?
 *>  10.10.10.20/30   200.0.0.2               20             0 1001 ?
 *>  10.10.10.24/30   200.0.0.2                0             0 1001 ?
 *>  10.10.10.28/30   200.0.0.2               20             0 1001 ?
 *>  10.10.10.32/30   200.0.0.2               30             0 1001 ?
 *>  10.10.10.36/30   200.0.0.2                0             0 1001 ?
 *>  192.168.10.0     200.0.0.2               21             0 1001 ?
 *>  192.168.20.0     200.0.0.2               21             0 1001 ?

Total number of prefixes 20


</code></pre>
</details>

<details>
<summary>R22 после фильтрации</summary>
<pre><code>

```
R22#sh ip bgp neighbors 100.0.0.2 received-routes
BGP table version is 424, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   1.1.1.4/32       100.0.0.2               21             0 1001 1001 1001 ?
 *   1.1.1.5/32       100.0.0.2               21             0 1001 1001 1001 ?
 *   1.1.1.12/32      100.0.0.2               11             0 1001 1001 1001 ?
 *   1.1.1.13/32      100.0.0.2               11             0 1001 1001 1001 ?
 *   1.1.1.14/32      100.0.0.2                0             0 1001 1001 1001 i
 *   1.1.1.15/32      100.0.0.2               21             0 1001 1001 1001 ?
 *   1.1.1.19/32      100.0.0.2               11             0 1001 1001 1001 ?
 *   1.1.1.20/32      100.0.0.2               31             0 1001 1001 1001 ?
 *   10.10.10.0/30    100.0.0.2               20             0 1001 1001 1001 ?
 *   10.10.10.4/30    100.0.0.2               20             0 1001 1001 1001 ?
 *   10.10.10.8/30    100.0.0.2                0             0 1001 1001 1001 ?
 *   10.10.10.12/30   100.0.0.2               20             0 1001 1001 1001 ?
 *   10.10.10.16/30   100.0.0.2               20             0 1001 1001 1001 ?
 *   10.10.10.20/30   100.0.0.2               20             0 1001 1001 1001 ?
 *   10.10.10.24/30   100.0.0.2               20             0 1001 1001 1001 ?
 *   10.10.10.28/30   100.0.0.2                0             0 1001 1001 1001 ?
 *   10.10.10.32/30   100.0.0.2                0             0 1001 1001 1001 ?
 *   10.10.10.36/30   100.0.0.2               30             0 1001 1001 1001 ?
 *   192.168.10.0     100.0.0.2               21             0 1001 1001 1001 ?
 *   192.168.20.0     100.0.0.2               21             0 1001 1001 1001 ?

Total number of prefixes 20

```

</code></pre>
</details>
Число получаемых R22 маршрутов от R14 снизилось с 33 до 20.

## Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).  

<details>
<summary>R24 до фильтрации</summary>
<pre><code>

R24#sh ip bgp neighbors 111.111.111.2 received-routes
BGP table version is 479, local router ID is 24.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  1.1.1.9/32       111.111.111.2      1536640             0 2042 ?
 *>  1.1.1.10/32      111.111.111.2      1536640             0 2042 ?
 *>  1.1.1.16/32      111.111.111.2      1024640             0 2042 ?
 *>  1.1.1.17/32      111.111.111.2      1024640             0 2042 ?
 *>  1.1.1.18/32      111.111.111.2            0             0 2042 i
 *>  1.1.1.32/32      111.111.111.2      1536640             0 2042 ?
 *>  10.10.10.32/27   111.111.111.2      1536000             0 2042 ?
 *>  10.10.10.44/30   111.111.111.2            0             0 2042 ?
 *>  10.10.10.60/30   111.111.111.2            0             0 2042 ?
 *>  10.10.10.64/30   111.111.111.2      1536000             0 2042 ?
 *>  192.168.30.0     111.111.111.2      1541120             0 2042 ?
 *>  192.168.40.0     111.111.111.2      1541120             0 2042 ?

Total number of prefixes 12
R24#


</code></pre>
</details>


prefix-list на R18
```
R18(config-router)#do sh run | sec pre
ip prefix-list SPBFILT seq 10 permit 10.10.10.40/29
ip prefix-list SPBFILT seq 11 permit 10.10.10.32/27
ip prefix-list SPBFILT seq 12 permit 10.10.10.64/30
ip prefix-list SPBFILT seq 20 permit 192.168.30.0/24
ip prefix-list SPBFILT seq 21 permit 192.168.40.0/24
ip prefix-list SPBFILT seq 30 permit 1.1.1.9/32
ip prefix-list SPBFILT seq 31 permit 1.1.1.10/32
ip prefix-list SPBFILT seq 32 permit 1.1.1.16/32
ip prefix-list SPBFILT seq 33 permit 1.1.1.17/32
ip prefix-list SPBFILT seq 34 permit 1.1.1.18/32
ip prefix-list SPBFILT seq 35 permit 1.1.1.32/32
ip prefix-list SPBFILT seq 100 deny 0.0.0.0/0 le 32
R18(config)#do sh run | sec bgp
router bgp 2042
 bgp router-id 18.18.18.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 1.1.1.18 mask 255.255.255.255
 redistribute eigrp 1
 neighbor 111.111.1.1 remote-as 520
 neighbor 111.111.1.1 prefix-list SPBFILT out
 neighbor 111.111.111.1 remote-as 520
 neighbor 111.111.111.1 prefix-list SPBFILT out
 maximum-paths 2
```
<details>
<summary>R24 после фильтрации</summary>
<pre><code>

R24#sh ip bgp neighbors 111.111.111.2 received-routes
BGP table version is 513, local router ID is 24.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  1.1.1.9/32       111.111.111.2      1536640             0 2042 ?
 *>  1.1.1.10/32      111.111.111.2      1536640             0 2042 ?
 *>  1.1.1.16/32      111.111.111.2      1024640             0 2042 ?
 *>  1.1.1.17/32      111.111.111.2      1024640             0 2042 ?
 *>  1.1.1.18/32      111.111.111.2            0             0 2042 i
 *>  1.1.1.32/32      111.111.111.2      1536640             0 2042 ?
 *>  10.10.10.32/27   111.111.111.2      1536000             0 2042 ?
 *>  10.10.10.64/30   111.111.111.2      1536000             0 2042 ?
 *>  192.168.30.0     111.111.111.2      1541120             0 2042 ?
 *>  192.168.40.0     111.111.111.2      1541120             0 2042 ?

Total number of prefixes 10

</code></pre>
</details>

Количество маршрутов снизилось с 12 до 10.

## Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию. 

<details>
<summary>R14 до фильтрации</summary>
<pre><code>

```
R14#sh ip bgp neighbors 100.0.0.1 received-routes
BGP table version is 163, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   1.1.1.9/32       100.0.0.1                              0 101 520 2042 ?
 *   1.1.1.10/32      100.0.0.1                              0 101 520 2042 ?
 *   1.1.1.16/32      100.0.0.1                              0 101 520 2042 ?
 *   1.1.1.17/32      100.0.0.1                              0 101 520 2042 ?
 *   1.1.1.18/32      100.0.0.1                              0 101 520 2042 i
 *   1.1.1.21/32      100.0.0.1                              0 101 301 i
 *   1.1.1.22/32      100.0.0.1                0             0 101 i
 *   1.1.1.23/32      100.0.0.1                              0 101 520 i
 *   1.1.1.24/32      100.0.0.1                              0 101 520 i
 *   1.1.1.25/32      100.0.0.1                              0 101 520 i
 *   1.1.1.26/32      100.0.0.1                              0 101 520 i
 *   1.1.1.32/32      100.0.0.1                              0 101 520 2042 ?
 *   10.10.10.32/27   100.0.0.1                              0 101 520 2042 ?
 *   10.10.10.64/30   100.0.0.1                              0 101 520 2042 ?
 *   192.168.30.0     100.0.0.1                              0 101 520 2042 ?
 *   192.168.40.0     100.0.0.1                              0 101 520 2042 ?

Total number of prefixes 16
```

</code></pre>
</details>

На R22 распространяем маршрут по умолчанию и вешаем запрещающий route-map на выход в сторону R14
```
R22#sh run | sec bgp
router bgp 101
 bgp router-id 22.22.22.22
 bgp log-neighbor-changes
 network 1.1.1.22 mask 255.255.255.255
 neighbor 11.11.11.2 remote-as 301
 neighbor 12.12.12.2 remote-as 520
 neighbor 100.0.0.2 remote-as 1001
 neighbor 100.0.0.2 default-originate
 neighbor 100.0.0.2 soft-reconfiguration inbound
 neighbor 100.0.0.2 route-map REJECTALL out
R22#sh run | sec route-map
 neighbor 100.0.0.2 route-map REJECTALL out
route-map REJECTALL deny 10
R22#
```

<details>
<summary>R14 после фильтрации</summary>
<pre><code>

```
R14#sh ip bgp neighbors  100.0.0.1 received-routes
BGP table version is 170, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          100.0.0.1                              0 101 i

Total number of prefixes 1
R14#
```

</code></pre>
</details>

## Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.     

<details>
<summary>R15 до фильтрации</summary>
<pre><code>

```
R15#sh ip bgp neighbors 200.0.0.1 received-routes
BGP table version is 134, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   1.1.1.9/32       200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.10/32      200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.16/32      200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.17/32      200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.18/32      200.0.0.1                              0 301 520 2042 i
 *   1.1.1.21/32      200.0.0.1                0             0 301 i
 *   1.1.1.22/32      200.0.0.1                              0 301 101 i
 *   1.1.1.23/32      200.0.0.1                              0 301 520 i
 *   1.1.1.24/32      200.0.0.1                              0 301 520 i
 *   1.1.1.25/32      200.0.0.1                              0 301 520 i
 *   1.1.1.26/32      200.0.0.1                              0 301 520 i
 *   1.1.1.32/32      200.0.0.1                              0 301 520 2042 ?
 *   10.10.10.32/27   200.0.0.1                              0 301 520 2042 ?
 *   10.10.10.64/30   200.0.0.1                              0 301 520 2042 ?
 *   192.168.30.0     200.0.0.1                              0 301 520 2042 ?
 *   192.168.40.0     200.0.0.1                              0 301 520 2042 ?

Total number of prefixes 16

```

</code></pre>
</details>

Аналогично, ананосируем маршрут по умолчанию и пишем route-map, но в нее закидываем as-path access list с маской _2042$

```
R21(config)#ip as-path access-list 1 permit _2042$
R21(config)#route-map SPBTOMSK permit 10
R21(config-route-map)#match as-path 1
R21(config-route-map)#ex
R21(config)#router bgp 301
R21(config-router)#neighbor 200.0.0.2 route-map SPBTOMSK out
R21#sh run | sec bgp
router bgp 301
 bgp router-id 21.21.21.21
 bgp log-neighbor-changes
 network 1.1.1.21 mask 255.255.255.255
 neighbor 11.11.11.1 remote-as 101
 neighbor 13.13.13.2 remote-as 520
 neighbor 200.0.0.2 remote-as 1001
 neighbor 200.0.0.2 default-originate
 neighbor 200.0.0.2 soft-reconfiguration inbound
 neighbor 200.0.0.2 route-map SPBTOMSK out
R21#sh run | sec route-map
 neighbor 200.0.0.2 route-map SPBTOMSK out
route-map SPBTOMSK permit 10
 match as-path 1
R21#
R21#sh run | sec acc
ip as-path access-list 1 permit _2042$
R21#

```




<details>
<summary>R15 после фильтрации</summary>
<pre><code>

```
R15#sh ip bgp neighbors 200.0.0.1 received-routes
BGP table version is 231, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   0.0.0.0          200.0.0.1                              0 301 i
 *   1.1.1.9/32       200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.10/32      200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.16/32      200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.17/32      200.0.0.1                              0 301 520 2042 ?
 *   1.1.1.18/32      200.0.0.1                              0 301 520 2042 i
 *   1.1.1.32/32      200.0.0.1                              0 301 520 2042 ?
 *   10.10.10.32/27   200.0.0.1                              0 301 520 2042 ?
 *   10.10.10.64/30   200.0.0.1                              0 301 520 2042 ?
 *   192.168.30.0     200.0.0.1                              0 301 520 2042 ?
 *   192.168.40.0     200.0.0.1                              0 301 520 2042 ?

Total number of prefixes 11
R15#


```

</code></pre>
</details>
