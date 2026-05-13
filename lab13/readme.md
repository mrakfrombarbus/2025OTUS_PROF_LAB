# IPSec over DmVPN

## Цель:
- Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
- Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги


## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

- Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
- Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.
- Все узлы в офисах в лабораторной работе должны иметь IP связность.
- План работы и изменения зафиксированы в документации.
- Дополнительно: Для IPSec использовать CA и сертификаты.

## Топология

![](Topology9.png)

## CA и сертификаты
</code></pre>
</details>
<details>
<summary>R14 (Выбран в качестве CA) </summary>
<pre><code>


```
R14(config)#ip domain name otus.ru
R14(config)#ip http server
R14(config)#$generate rsa general-keys label R14 exportable modulus 2048
The name for the keys will be: R14

% The key modulus size is 2048 bits
% Generating 2048 bit RSA keys, keys will be exportable...
[OK] (elapsed time was 1 seconds)

R14(config)#crypto pki server R14
R14(cs-server)# database level complete
R14(cs-server)# no database archive
R14(cs-server)# issuer-name CN=R14, O=Otus, C=ru
R14(cs-server)#no shutdown
%Some server settings cannot be changed after CA certificate generation.
% Please enter a passphrase to protect the private key
% or type Return to exit
Password:

Re-enter password:

% Certificate Server enabled.
R14(cs-server)#

```
</code></pre>
</details>

Получаем сертификаты со всех необходимых устройств:
</code></pre>
</details>
<details>
<summary>Пример R15</summary>
<pre><code>


```
R15(config)#ip host R14 1.1.1.14
R15(config)#ip host R14.otus.ru 1.1.1.14
R15(config)#crypto pki trustpoint R14
R15(ca-trustpoint)# enrollment url http://R14.otus.ru:80
R15(ca-trustpoint)# serial-number
R15(ca-trustpoint)# subject-name CN=R15, O=Otus, C=RU
R15(ca-trustpoint)#revocation-check crl
R15(ca-trustpoint)#
R15(config)#crypto pki authenticate R14
Certificate has the following attributes:
       Fingerprint MD5: A792544F 9A2D75F5 B89E404F DF8EFD2C
      Fingerprint SHA1: 81788E5E 10EBA99F 904FA74E 543B8375 3A490814

% Do you accept this certificate? [yes/no]: y
Trustpoint CA certificate accepted.
R15(config)#crypto pki enroll R14
%
% Start certificate enrollment ..
% Create a challenge password. You will need to verbally provide this
   password to the CA Administrator in order to revoke your certificate.
   For security reasons your password will not be saved in the configuration.
   Please make a note of it.

Password:
*Apr  2 17:07:35.215:  RSA key size needs to be atleast 768 bits for ssh version 2
*Apr  2 17:07:35.222: %SSH-5-ENABLED: SSH 1.5 has been enabled
*Apr  2 17:07:35.222: %CRYPTO-6-AUTOGEN: Generated new 512 bit key pair
Re-enter password:

% The subject name in the certificate will include: CN=R15, O=Otus, C=RU
% The subject name in the certificate will include: R15
% The serial number in the certificate will be: 67109104
% Include an IP address in the subject name? [no]: n
Request certificate from CA? [yes/no]: y
% Certificate request sent to Certificate Authority
% The 'show crypto pki certificate verbose R14' commandwill show the fingerprint.

R15(config)#
*Apr  2 17:07:52.537: CRYPTO_PKI:  Certificate Request Fingerprint MD5: B9333411 05F61100 22750CC5 413A53AD
*Apr  2 17:07:52.537: CRYPTO_PKI:  Certificate Request Fingerprint SHA1: 093F4427 4A700507 7FBBA3B1 923F68F7 D56A0498
R15(config)#
```
</code></pre>
</details>


Повторяем на всех устройствах и подтверждаем на СА

</code></pre>
</details>
<details>
<summary>R14</summary>
<pre><code>

```
R14#sh crypto pki server R14 requests
Enrollment Request Database:

Subordinate CA certificate requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------

RA certificate requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------

Router certificates requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
4      pending    6C63BD69ABD5DC36C46012DAE6BF3ABE serialNumber=67109152+hostname=R18,cn=R18,o=Otus,c=RU
3      pending    3AC83F9D9A5B6CF1DEC45B782AB88A36 serialNumber=67109312+hostname=R28,cn=R28,o=Otus,c=RU
2      pending    FC0B80F760C7BA2BC934EF685BB9727D serialNumber=67109296+hostname=R27,cn=R27,o=Otus,c=RU
1      pending    B933341105F6110022750CC5413A53AD serialNumber=67109104+hostname=R15,cn=R15,o=Otus,c=RU

R14#crypto pki server R14 grant 1
R14#crypto pki server R14 grant all

R14#sh cry pki serv R14 certificates
Serial Issued date              Expire date               Subject Name
1       16:57:14 UTC Apr 2 2026  16:57:14 UTC Apr 1 2029   cn=R14 o=Otus c=ru
2       17:20:26 UTC Apr 2 2026  17:20:26 UTC Apr 2 2027   serialNumber=67109104+hostname=R15 cn=R15 o=Otus c=RU
3       17:20:44 UTC Apr 2 2026  17:20:44 UTC Apr 2 2027   serialNumber=67109152+hostname=R18 cn=R18 o=Otus c=RU
4       17:20:44 UTC Apr 2 2026  17:20:44 UTC Apr 2 2027   serialNumber=67109312+hostname=R28 cn=R28 o=Otus c=RU
5       17:20:44 UTC Apr 2 2026  17:20:44 UTC Apr 2 2027   serialNumber=67109296+hostname=R27 cn=R27 o=Otus c=RU

R14#

```
</code></pre>
</details>

## Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
</code></pre>
</details>
<details>
<summary>R15</summary>
<pre><code>

```
R15#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R15(config)#crypto ikev2 proposal OTUS
IKEv2 proposal MUST either have a set of an encryption algorithm other than aes-gcm, an integrity algorithm and a DH group configured or
 encryption algorithm aes-gcm, a prf algorithm and a DH group configured
R15(config-ikev2-proposal)# encryption aes-cbc-128
R15(config-ikev2-proposal)# integrity sha256
R15(config-ikev2-proposal)# group 24
R15(config-ikev2-proposal)#!
R15(config-ikev2-proposal)#crypto ikev2 policy OTUS
IKEv2 policy MUST have atleast one complete proposal attached
R15(config-ikev2-policy)# proposal OTUS
R15(config-ikev2-policy)#!
R15(config-ikev2-policy)#!
R15(config-ikev2-policy)#crypto ikev2 profile OTUS
IKEv2 profile MUST have:
   1. A local and a remote authentication method.
   2. A match identity or a match certificate or match any statement.
R15(config-ikev2-profile)# match identity remote address 0.0.0.0
R15(config-ikev2-profile)# authentication remote rsa-sig
R15(config-ikev2-profile)# authentication local rsa-sig
R15(config-ikev2-profile)# pki trustpoint R14
R15(config-ikev2-profile)#!
R15(config-ikev2-profile)#!
R15(config-ikev2-profile)#!
R15(config-ikev2-profile)#crypto ipsec transform-set OTUS esp-gcm 256
R15(cfg-crypto-trans)# mode transport
R15(cfg-crypto-trans)#!
R15(cfg-crypto-trans)#crypto ipsec profile OTUS
R15(ipsec-profile)# set transform-set OTUS
R15(ipsec-profile)# set ikev2-profile OTUS
R15(ipsec-profile)#!
R15(ipsec-profile)#
R15#
*Apr  2 18:15:11.756: %SYS-5-CONFIG_I: Configured from console by console
R15#ping 10.15.18.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/12/16 ms
R15#ping 10.15.18.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R15#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R15(config)#int tun 0
R15(config-if)#tun prote ipsec profile OTUS
R15(config-if)#
*Apr  2 18:17:03.194: %CRYPTO-6-ISAKMP_ON_OFF: ISAKMP is ON
R15(config-if)#
*Apr  2 18:17:24.554: %CRYPTO-6-IKMP_NO_ID_CERT_ADDR_MATCH: ID of 111.111.111.2 (type 1) and certificate addr with
*Apr  2 18:17:24.555: %CRYPTO-6-IKMP_NO_ID_CERT_ADDR_MATCH: ID of 111.111.111.2 (type 1) and certificate addr with
R15(config-if)#
R15#ping 10.15.18.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/16/22 ms
R15#ping 10.15.18.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/14/16 ms
R15#

```

</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>R18</summary>
<pre><code>


```
R18#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R18(config)#
R18(config)#
R18(config)#crypto ikev2 proposal OTUS
IKEv2 proposal MUST either have a set of an encryption algorithm other than aes-gcm, an integrity algorithm and a DH group configured or
 encryption algorithm aes-gcm, a prf algorithm and a DH group configured
R18(config-ikev2-proposal)# encryption aes-cbc-128
R18(config-ikev2-proposal)# integrity sha256
R18(config-ikev2-proposal)# group 24
R18(config-ikev2-proposal)#!
R18(config-ikev2-proposal)#crypto ikev2 policy OTUS
IKEv2 policy MUST have atleast one complete proposal attached
R18(config-ikev2-policy)# proposal OTUS
R18(config-ikev2-policy)#!
R18(config-ikev2-policy)#!
R18(config-ikev2-policy)#crypto ikev2 profile OTUS
IKEv2 profile MUST have:
   1. A local and a remote authentication method.
   2. A match identity or a match certificate or match any statement.
R18(config-ikev2-profile)# match identity remote address 0.0.0.0
R18(config-ikev2-profile)# authentication remote rsa-sig
R18(config-ikev2-profile)# authentication local rsa-sig
R18(config-ikev2-profile)# pki trustpoint R14
R18(config-ikev2-profile)#!
R18(config-ikev2-profile)#!
R18(config-ikev2-profile)#!
R18(config-ikev2-profile)#crypto ipsec transform-set OTUS esp-gcm 256
R18(cfg-crypto-trans)# mode transport
R18(cfg-crypto-trans)#!
R18(cfg-crypto-trans)#crypto ipsec profile OTUS
R18(ipsec-profile)# set transform-set OTUS
R18(ipsec-profile)# set ikev2-profile OTUS
R18(ipsec-profile)#!
R18(ipsec-profile)#
R18#
Apr  2 18:15:09.371: %SYS-5-CONFIG_I: Configured from console by console
R18#ping 10.15.18.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R18#ping 10.15.18.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/15/17 ms
R18#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R18(config)#int tun 0
R18(config-if)#tun protec ipsec prof OTUS
R18(config-if)#
Apr  2 18:17:24.419: %CRYPTO-6-ISAKMP_ON_OFF: ISAKMP is ON
Apr  2 18:17:24.668: %CRYPTO-6-IKMP_NO_ID_CERT_ADDR_MATCH: ID of 200.0.0.2 (type 1) and certificate addr with
Apr  2 18:17:24.668: %CRYPTO-6-IKMP_NO_ID_CERT_ADDR_MATCH: ID of 200.0.0.2 (type 1) and certificate addr with
R18(config-if)#
R18#ping 10.15.18.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 15/15/16 ms
R18#ping 10.15.18.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.15.18.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 14/16/21 ms
R18#

```
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>Проверка</summary>
<pre><code>


```
R15#sh crypto ipsec sa

interface: Tunnel0
    Crypto map tag: Tunnel0-head-0, local addr 200.0.0.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (200.0.0.2/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (111.111.111.2/255.255.255.255/47/0)
   current_peer 111.111.111.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 10, #pkts encrypt: 10, #pkts digest: 10
    #pkts decaps: 10, #pkts decrypt: 10, #pkts verify: 10
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 200.0.0.2, remote crypto endpt.: 111.111.111.2
     plaintext mtu 1466, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/2
     current outbound spi: 0x4AF1F066(1257369702)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0xDBDAE5A4(3688555940)
        transform: esp-gcm 256 ,
        in use settings ={Transport, }
        conn id: 1, flow_id: SW:1, sibling_flags 80000000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4161743/3491)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x4AF1F066(1257369702)
        transform: esp-gcm 256 ,
        in use settings ={Transport, }
        conn id: 2, flow_id: SW:2, sibling_flags 80000000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4161743/3491)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:
R15#

```
</code></pre>
</details>

## $Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.
</code></pre>
</details>
<details>
<summary>Конфиг</summary>
<pre><code>

```
!
crypto ikev2 proposal OTUS-DMVPN
 encryption aes-cbc-128
 integrity sha256
 group 24
!
crypto ikev2 policy OTUS-DMVPN
 proposal OTUS-DMVPN
!
!
crypto ikev2 profile OTUS-DMVPN
 match identity remote address 0.0.0.0 
 authentication remote rsa-sig
 authentication local rsa-sig
 pki trustpoint R14
!
!
!
crypto ipsec transform-set OTUS-DMVPN esp-gcm 256 
 mode transport
!
crypto ipsec profile OTUS-DMVPN
 set transform-set OTUS-DMVPN 
 set ikev2-profile OTUS-DMVPN
!
 
interface Tunnel13
 tunnel protection ipsec profile OTUS-DMVPN

```
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>Проверка</summary>
<pre><code>

Sending 5, 100-byte ICMP Echos to 1.1.1.27, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 14/16/19 ms
R28#ping 1.1.1.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.15, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/14/17 ms
R28#ping 10.66.6.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.66.6.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 14/16/20 ms
R28#ping 10.66.6.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.66.6.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 13/16/23 ms
R28#sh cry
R28#sh crypto ipsec sa

interface: Tunnel13
    Crypto map tag: Tunnel13-head-0, local addr 16.16.16.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (16.16.16.2/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (111.1.1.2/255.255.255.255/47/0)
   current_peer 111.1.1.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 12, #pkts encrypt: 12, #pkts digest: 12
    #pkts decaps: 12, #pkts decrypt: 12, #pkts verify: 12
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 16.16.16.2, remote crypto endpt.: 111.1.1.2
     plaintext mtu 1466, path mtu 1500, ip mtu 1500, ip mtu idb (none)
     current outbound spi: 0x69DE51D3(1776177619)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x91BF7455(2445243477)
        transform: esp-gcm 256 ,
        in use settings ={Transport, }
        conn id: 8, flow_id: SW:8, sibling_flags 80000000, crypto map: Tunnel13-head-0
        sa timing: remaining key lifetime (k/sec): (4357783/3459)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x69DE51D3(1776177619)
        transform: esp-gcm 256 ,
        in use settings ={Transport, }
        conn id: 7, flow_id: SW:7, sibling_flags 80000000, crypto map: Tunnel13-head-0
        sa timing: remaining key lifetime (k/sec): (4357783/3459)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (16.16.16.2/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (200.0.0.2/255.255.255.255/47/0)
   current_peer 200.0.0.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 71, #pkts encrypt: 71, #pkts digest: 71
    #pkts decaps: 72, #pkts decrypt: 72, #pkts verify: 72
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 16.16.16.2, remote crypto endpt.: 200.0.0.2
     plaintext mtu 1466, path mtu 1500, ip mtu 1500, ip mtu idb (none)
     current outbound spi: 0xDCFBE023(3707494435)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x746319CF(1952651727)
        transform: esp-gcm 256 ,
        in use settings ={Transport, }
        conn id: 2, flow_id: SW:2, sibling_flags 80000000, crypto map: Tunnel13-head-0
        sa timing: remaining key lifetime (k/sec): (4350620/3405)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0xDCFBE023(3707494435)
        transform: esp-gcm 256 ,
        in use settings ={Transport, }
        conn id: 1, flow_id: SW:1, sibling_flags 80000000, crypto map: Tunnel13-head-0
        sa timing: remaining key lifetime (k/sec): (4350620/3405)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:
R28#

</code></pre>
</details>
