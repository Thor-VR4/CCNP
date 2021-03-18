# VxLAN. L2VNI
Лабараторное задание №6.

## Задание
1. [Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами](#chapter-0)
![alt-текст](https://github.com/Thor-VR4/CCNP/blob/main/HomeWork/%231%20IP/Net.png "Стенд №1")

<a id="chapter-0"></a>
## Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами
Сначала настраиваем BGP peering между Leaf и Spine в AF l2vpn evpn.  
Пример настройки на SPINE:
```
router bgp 65000
  template peer LEAF
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
  neighbor 172.16.0.4
    inherit peer LEAF
  neighbor 172.16.0.5
    inherit peer LEAF
  neighbor 172.16.0.6
    inherit peer LEAF
```
Пример настройки на LEAF:
```
router bgp 65000
  template peer SPINE
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 172.16.0.1
    inherit peer SPINE
  neighbor 172.16.0.2
    inherit peer SPINE
```
Настраиваем evpn и клиентский vlan:
```
evpn
  vni 10004 l2
    rd auto
    route-target import auto
    route-target export auto
fabric forwarding anycast-gateway-mac 0000.0000.1111
interface Vlan4
  no shutdown
  ip address 192.168.0.1/24
  fabric forwarding mode anycast-gateway

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10004
    ingress-replication protocol bgp
```


<a id="chapter-1"></a>
## Проверка роботоспособности

На R9 проверяем L2 связанность с R12:
```
R9#ping 192.168.0.12
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.12, timeout is 2 seconds:
!!!!!
```
Вывод BGP AF l2vpn на NXOS6:
```
NXOS6# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 66, Local Router ID is 172.16.0.6
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 172.16.0.4:32771
* i[2]:[0]:[0]:[48]:[aabb.cc00.c000]:[0]:[0.0.0.0]/216
                      172.17.0.4                        100          0 i
*>i                   172.17.0.4                        100          0 i
* i[2]:[0]:[0]:[48]:[aabb.cc00.c000]:[32]:[192.168.0.12]/248
                      172.17.0.4                        100          0 i
*>i                   172.17.0.4                        100          0 i
*>i[3]:[0]:[32]:[172.17.0.4]/88
                      172.17.0.4                        100          0 i
* i                   172.17.0.4                        100          0 i

Route Distinguisher: 172.16.0.6:32771    (L2VNI 10004)
*>l[2]:[0]:[0]:[48]:[aabb.cc00.9000]:[0]:[0.0.0.0]/216
                      172.17.0.6                        100      32768 i
*>i[2]:[0]:[0]:[48]:[aabb.cc00.c000]:[0]:[0.0.0.0]/216
                      172.17.0.4                        100          0 i
*>l[2]:[0]:[0]:[48]:[aabb.cc00.9000]:[32]:[192.168.0.9]/248
                      172.17.0.6                        100      32768 i
*>i[2]:[0]:[0]:[48]:[aabb.cc00.c000]:[32]:[192.168.0.12]/248
                      172.17.0.4                        100          0 i
*>i[3]:[0]:[32]:[172.17.0.4]/88
                      172.17.0.4                        100          0 i
*>l[3]:[0]:[32]:[172.17.0.6]/88
                      172.17.0.6                        100      32768 i
```
P.S. В записи "i[2]:[0]:[0]:[48]:[aabb.cc00.c000]:[32]:[192.168.0.12]/248" [48] или [32] скорее всего указывают на рамер поля данных за ними.