# VxLAN. VPC & VxLAN EVPN route-type 5
Лабараторное задание №7.

## Задание
1. [Настроить VPC между 2 устройствами](#chapter-0)
2. [Настроить Overlay на основе VxLAN EVPN route-type 5](#chapter-1)
![alt-текст](https://github.com/Thor-VR4/CCNP/blob/main/HomeWork/%237%20VPC/VPC.jpg "Стенд №1")

<a id="chapter-0"></a>
## Настроить VPC между 2 устройствами
VPC настраивается между NXOS4 и NXOS5.
Пример настройки на NXOS4:
```
feature lacp
feature vpc

vpc domain 1
  peer-keepalive destination 20.0.0.5 source 20.0.0.4 vrf KEEP
  
interface port-channel1
  description R12
  switchport mode trunk
  vpc 1

interface port-channel5
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link
  
interface Ethernet1/3
  switchport mode trunk
  channel-group 5 mode active

interface Ethernet1/4
  switchport mode trunk
  channel-group 5 mode active

interface Ethernet1/5
  description R12
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  no switchport
  vrf member KEEP
  ip address 20.0.0.4/24
  no shutdown
```
Настройка lacp на R12:
```
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/0
 description N4
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/1
 description N5
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
```
Состояние VPC на NXOS5:
```
NXOS5# show vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1   
Peer status                       : peer adjacency formed ok      
vPC keep-alive status             : peer is alive                 
Configuration consistency status  : success 
Per-vlan consistency status       : success                       
Type-2 consistency status         : success 
vPC role                          : secondary                     
Number of vPCs configured         : 1   
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans    
--    ----   ------ -------------------------------------------------
1     Po5    up     1,44,99                                                     
         

vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
1     Po1           up     success     success               1,44,99          
```
Состояние port-channel на R12:
```
R12#show etherchannel 1 summary 
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Et0/0(P)    Et0/1(P)    
```

<a id="chapter-1"></a>
## Настроить Overlay на основе VxLAN EVPN route-type 5
Таблица адресации:
Устройство | VLAN | VNI | IPv4 
--- | --- | --- | --- 
R9 | 4 | 10004 | 192.168.4.9 
R12 | 44 | 10044 | 192.168.44.12

Пример настройки на NXOS6:
```
vrf context R9
  vni 99000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
	
vlan 99
  vn-segment 99000
  
interface nve1
  member vni 99000 associate-vrf

interface Vlan4
  vrf member R9
  no shutdown
  ip address 192.168.4.1/24
  fabric forwarding mode anycast-gateway
  
interface Vlan99
  description L3VNI
  no shutdown
  vrf member R9
  ip forward
  
router bgp 65000
  vrf R9
    address-family ipv4 unicast
      network 192.168.4.0/24
```  

## Проверка роботоспособности

Доступность R9 c R12:
```
R12#ping 192.168.4.9
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.4.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 85/239/657 ms
```

Таблица маршрутизации на NXOS6 в vrf R9:
```
NXOS6# show ip route vrf R9
IP Route Table for VRF "R9"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

192.168.4.0/24, ubest/mbest: 1/0, attached
    *via 192.168.4.1, Vlan4, [0/0], 06:01:32, direct
192.168.4.1/32, ubest/mbest: 1/0, attached
    *via 192.168.4.1, Vlan4, [0/0], 06:01:32, local
192.168.4.9/32, ubest/mbest: 1/0, attached
    *via 192.168.4.9, Vlan4, [190/0], 05:43:51, hmm
192.168.44.0/24, ubest/mbest: 1/0
    *via 172.17.1.4%default, [200/0], 01:24:09, bgp-65000, internal, tag 65000 (evpn) segid: 99000 tunnelid: 0xac110104 encap: VXLAN
 
192.168.44.12/32, ubest/mbest: 1/0
    *via 172.17.1.4%default, [200/0], 05:29:05, bgp-65000, internal, tag 65000 (evpn) segid: 99000 tunnelid: 0xac110104 encap: VXLAN
```

Таблица BGP L2VPN на NXOS5:
```
NXOS5# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 9810, Local Router ID is 172.16.0.5
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 172.16.0.5:32811    (L2VNI 10044)
*>l[2]:[0]:[0]:[48]:[aabb.cc80.c000]:[0]:[0.0.0.0]/216
                      172.17.1.4                        100      32768 i
*>l[2]:[0]:[0]:[48]:[aabb.cc80.c000]:[32]:[192.168.44.12]/272
                      172.17.1.4                        100      32768 i
*>l[3]:[0]:[32]:[172.17.1.4]/88
                      172.17.1.4                        100      32768 i

Route Distinguisher: 172.16.0.6:3
*>i[5]:[0]:[0]:[24]:[192.168.4.0]/224
                      172.17.0.6                        100          0 i
* i                   172.17.0.6                        100          0 i

Route Distinguisher: 172.16.0.6:32771
* i[2]:[0]:[0]:[48]:[aabb.cc00.9000]:[0]:[0.0.0.0]/216
                      172.17.0.6                        100          0 i
*>i                   172.17.0.6                        100          0 i
* i[2]:[0]:[0]:[48]:[aabb.cc00.9000]:[32]:[192.168.4.9]/272
                      172.17.0.6                        100          0 i
*>i                   172.17.0.6                        100          0 i
* i[3]:[0]:[32]:[172.17.0.6]/88
                      172.17.0.6                        100          0 i
*>i                   172.17.0.6                        100          0 i

Route Distinguisher: 172.16.0.5:4    (L3VNI 99000)
*>l[5]:[0]:[0]:[24]:[192.168.44.0]/224
                      172.17.1.4                        100      32768 i
```